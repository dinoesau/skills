# Rusty - Ejemplos before/after

Cada ejemplo muestra el patron defensivo y su reemplazo type-driven.
Copia el lado derecho como punto de partida.
Todos los snippets compilan con las definiciones de [REFERENCE.md](./REFERENCE.md).

## 1. Validar en todos lados vs parsear una vez

Before: mismos tres guardias copiados en cada funcion.
After: firmas que prueban sus precondiciones.

```rust
// Before: paranoia repetida.
pub fn send_receipt(user_id: String, email: String, amount_cents: i32) -> Result<(), String> {
    if user_id.trim().is_empty() {
        return Err("invalid user_id".to_string());
    }
    if !email.contains('@') {
        return Err("invalid email".to_string());
    }
    if amount_cents <= 0 {
        return Err("invalid amount".to_string());
    }
    Ok(())
}

// After: el tipo ya probo todo.
pub fn send_receipt_typed(user_id: UserId, email: Email, amount: Cents) -> Result<(), DomainError> {
    let _ = (user_id, email, amount);
    Ok(())
}
```

## 2. `is_valid` booleano vs smart constructor

Before: la respuesta se tira y el tipo sigue debil.
After: el valor sale certificado del borde.

```rust
// Before: el compilador no aprende nada.
pub fn notify_raw(raw_email: String) {
    if is_valid_email(&raw_email) {
        println!("sending to {raw_email}");
    }
}

// After: downstream ya no rechequea el @.
pub fn notify_typed(email: Email) {
    println!("sending to {}", email.as_str());
}
```

## 3. Division parcial vs total

Before: explota con cero.
After: el borde queda explicito en la firma.

```rust
// Before.
pub fn refund_share_partial(amount: u64, parts: u64) -> u64 {
    amount / parts
}

// After: vive en `src/domain/cents.rs` con `Cents` para que `from_raw` siga privado.
pub fn refund_share_total(amount: Cents, parts: u64) -> Result<Cents, SplitError> {
    if parts == 0 {
        return Err(SplitError::EmptyParts);
    }
    let raw = amount.value();
    if raw % parts != 0 {
        return Err(SplitError::NotDivisible { amount: raw, parts });
    }
    Ok(Cents::from_raw(raw / parts))
}
```

## 4. Piramide de `if` vs railway con `?`

Before: niveles anidados por cada parseo.
After: happy path lineal con riel de error tipado.

```rust
// After: version recomendada.
pub fn build_order_clean(
    raw_order_id: &str,
    raw_user_id: &str,
    raw_email: &str,
    raw_amount: i64,
) -> Result<Order, OrderError> {
    let id = OrderId::parse(raw_order_id).map_err(OrderError::OrderId)?;
    let user_id = UserId::parse(raw_user_id).map_err(OrderError::UserId)?;
    let email = Email::parse(raw_email).map_err(OrderError::Email)?;
    let amount = Cents::parse(raw_amount).map_err(OrderError::Money)?;
    Ok(Order {
        id,
        user_id,
        email,
        amount,
        method: PaymentMethod::Cash,
        already_refunded: false,
    })
}
```

## 5. `String` de error vs `thiserror` exhaustivo

Before: imposible clasificar por programa.
After: cada variante mapea a un status distinto.

```rust
// Before.
pub fn process(raw: String) -> Result<String, String> {
    Err("something failed".to_string())
}

// After: mapeo unico via `refund_status_code`, usado por `IntoResponse`.
pub fn classify_failure(err: &DomainError) -> axum::http::StatusCode {
    crate::domain::error::refund_status_code(err)
}
```

## 6. Booleanos de estado vs type-state

Before: el orden se puede olvidar.
After: el orden malo no compila.

```rust
// Before: flag manual.
pub struct OrderFlag {
    pub paid: bool,
}

// After: transicion por movimiento.
// Lifecycle usa StagedOrder, nunca el Order del core.
let order_id = OrderId::parse("ord_1").expect("fixture is valid");
let email = Email::parse("user@example.com").expect("fixture is valid");
let draft = StagedOrder::<Draft>::new(order_id, email, Cents::parse(5000).expect("fixture is valid"));
let submitted = draft.submit();
let paid = submitted.pay();
println!("{}", paid.receipt());
```

## 7. Clon en hot path vs vista prestada

Before: aloca solo para validar de nuevo.
After: prueba sin heap y promueve una vez.

```rust
// After: parse prestado en router, owned solo para guardar.
let view = EmailRef::parse("user@example.com")?;
let owned: Email = view.to_owned_email();
```

## 8. Core puro vs handler delgado con puerto

El core no toca IO ni `async`.
El shell parsea, carga via puerto, llama y mapea.
La API de `calculate_refund` no cambia.

```rust
// Core: testeable sin mocks.
let refund = calculate_refund(&order, requested, &policy)?;

// Shell: parsea en el borde, carga persistencia via puerto.
// Forma invalida es InvalidOrderId 400; fila faltante es UserNotFound 404.
// Driver caido es Database 500 via AppError.
// Carga via trait `OrderRepository` con `State(AppState)`, `None -> UserNotFound`, `Err -> Database`.
pub struct AppState {
    pub repo: std::sync::Arc<dyn OrderRepository>,
    pub policy: RefundPolicy,
}

let order_id = OrderId::parse(&raw.order_id).map_err(DomainError::InvalidOrderId)?;
let _request_email = Email::parse(&raw.email).map_err(DomainError::InvalidEmail)?;
let amount = Cents::parse(raw.amount_cents).map_err(DomainError::InvalidAmount)?;
let order = state.repo.find(&order_id).await.map_err(AppError::Database)?.ok_or(DomainError::UserNotFound)?;
let refund = calculate_refund(&order, amount, &state.policy)?;
```

## 9. sqlx hardwireado vs `OrderRepository`

Before: el handler importa sqlx y no se puede testear sin DB.
After: el trait vive en app, `SqlxOrderRepo` va a prod y `InMemoryOrderRepo` a tests con `oneshot`.

```rust
// Before: acoplado.
async fn refund_handler(pool: sqlx::PgPool) { /* SELECT directo aqui */ }

// After: puerto.
pub trait OrderRepository: Send + Sync + 'static {
    async fn find(&self, id: &OrderId) -> Result<Option<Order>, sqlx::Error>;
}
```

## 10. Payment con campos publicos vs privados + `parse`

Before: `pub last_four: String` compila verbatim.
After: campo privado, `E0308` si pasas `&str` donde va `CardDetails`.

```rust
// Before: forjable desde cualquier modulo.
pub struct CardDetailsOld { pub last_four: String }

// After: solo `CardDetails::parse("4242")` crea; `parse("12")` es Err.
// `TransferDetails::parse` exige 15-34 chars, dos letras iniciales, alfanumerico ASCII, conteo por chars.
// Mas `as_str()` para lectura sin forja.
let card = CardDetails::parse("4242")?;
let transfer = TransferDetails::parse("DE89370400440532013000")?;
```

## 11. `Email` logueable vs `CustomerEmail` opaco

Before: cada `%email` es una fuga de PII que compila.
After: el log crudo falla con `E0277`.

```rust
// Before: compila y fuga.
tracing::error!(email = %email, "infrastructure failure");

// After: CustomerEmail no implementa Display; el log crudo no compila (E0277).
// Solo escotillas explicitas:
let customer = CustomerEmail::from(Email::parse("user@example.com").map_err(DomainError::InvalidEmail)?);
tracing::info!(email = customer.redacted(), "sending receipt");
let raw: &str = customer.expose_for_sending(); // solo en el borde de envio
```

## 12. `unwrap` por disciplina vs lints como invariantes

Before: "nunca `unwrap`" exigido por review; `cargo clippy -- -D warnings` pasa con `.unwrap()` en dominio.
After: el header lo convierte en fallo de build.

```rust
// Before: compila con -D warnings.
let email = Email::parse(raw).unwrap();

// After: con #![deny(clippy::unwrap_used)] esto falla clippy.
// Nunca implementes Deref<Target = str>: re-expone str en silencio.
// Version exigida:
let email = Email::parse(raw).map_err(DomainError::InvalidEmail)?;
```
