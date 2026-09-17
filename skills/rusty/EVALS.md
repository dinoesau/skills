# Rusty - Evaluaciones

Corre cada escenario en sesion fresca con la skill instalada.
Compara contra la baseline: mismo prompt sin la skill.
La skill gana solo si cumple todos los comportamientos esperados.
Re-corre cuando cambie el dominio, no solo cuando cambie la skill.

## Escenario 1: refactor de codigo defensivo

Entrada: una funcion que recibe `user_id: String`, `email: String`, `amount_cents: i32`,
repite tres `if` de validacion y retorna `Result<_, String>`.

Esperado:

- [ ] Crea `Email`, `UserId` y `Cents` con campo privado y unico `parse` que retorna `Result`.
- [ ] El core acepta solo tipos probados, sin `if` duplicados aguas abajo.
- [ ] `cargo check`, `cargo clippy -- -D warnings` y `cargo test` pasan.
- [ ] Ningun grep de fuga imprime lineas en `src/domain` o `src/core`.

## Escenario 2: errores estratificados

Entrada: dominio que retorna `Result<T, String>` y un handler que mapea todo a 400.

Esperado:

- [ ] Enum exhaustivo con `thiserror`; sin `String` ni `anyhow` en el dominio.
- [ ] `AppError` envuelve infra una vez; `anyhow` con `.context()` solo en el edge.
- [ ] 400, 404 o 422 por variante de dominio; 500 generico para infra.
- [ ] Agregar una variante rompe el `match` en compilacion.
- [ ] Adversarial: forma invalida `InvalidOrderId` 400 vs fila faltante `UserNotFound` 404, exceso `ExceedsMax` 422, doble refund `AlreadyRefunded` 422.
- [ ] El handler inyecta `Arc<dyn OrderRepository>` via `State`; `SqlxOrderRepo` solo en prod y `InMemoryOrderRepo` con `oneshot` en tests; nunca fabrica `Order` desde el request.

## Escenario 3: type-state en workflow ordenado

Entrada: flags `is_submitted` e `is_paid` revisados con `if` antes de cada accion.

Esperado:

- [ ] Estados `StagedOrder<Draft>` a `StagedOrder<Paid>` con transiciones que mueven `self`.
- [ ] Reusar un valor viejo no compila.
- [ ] Metodos por estado solo existen en el estado correcto (`receipt` solo en `Paid`).

## Escenario 4: sincronia post-#13/#19 (payments, status, ports)

Entrada: `pub last_four: String`, `refund_status_code -> u16`, handler con `Order` inline.

Esperado:

- [ ] `CardDetails` / `TransferDetails` con campos privados + `parse` por struct y enums `LastFourError` / `IbanError` con `Display` manual + `Error` estilo `MoneyError`. PoC `cargo check` (verbatim compila con campos publicos) + `E0308` contra `&str` cuando el campo es privado.
- [ ] Regla IBAN 15-34 chars, prefijo dos letras, alfanumerico ASCII, conteo por chars, mensaje actualizado, mas `as_str()`. PoC `cargo test` con matriz de bounds (8/8): `"DE12"`, `"X"`, `-`/`!`/espacios rechazan.
- [ ] `refund_status_code -> StatusCode` como mapeo unico usado por `IntoResponse`. Trait `OrderRepository` + `Arc<dyn>` via `State` + adapters sqlx/in-memory. `None -> UserNotFound` 404, `Err -> Database` 500. PoC `cargo check` + `clippy -D warnings`.
- [ ] Verificacion: extraer bloques y pasar `cargo check` + `clippy -D warnings`, diff contra `content/en|es/post/*-stop-validating-everywhere/index.md` post-#17/#19.

## Prueba de disparo

Debe activarse con: "quita las validaciones repetidas en este handler Rust",
"modela email como tipo", "haz estos errores exhaustivos".
No debe activarse con: codigo Python o TypeScript, logica ya tipada,
preguntas de infraestructura sin invariantes de dominio.
Si dispara de mas, estrecha la descripcion. Si no dispara, agrega la frase del usuario.
