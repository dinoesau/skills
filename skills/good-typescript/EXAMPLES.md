# Good-TypeScript - Ejemplos before/after

Cada ejemplo muestra el patron defensivo y su reemplazo type-driven.
Copia el lado derecho como punto de partida.
Todos los snippets compilan con las definiciones de [REFERENCE.md](./REFERENCE.md).

## 1. Validar en todos lados vs parsear una vez

Before: mismos tres guardias copiados en cada funcion.
After: firmas que prueban sus precondiciones.

```ts
// Before: paranoia repetida.
export async function sendReceipt(userId: string, amount: number): Promise<void> {
  if (userId.trim() === "") throw new Error("Invalid userId");
  if (!(amount > 0)) throw new Error("Invalid amount");
}

// After: el tipo ya probo todo.
export async function sendReceiptTyped(userId: UserId, amount: Cents): Promise<void> {
  void userId;
  void amount;
}
```

## 2. `isValid` booleano vs smart constructor

Before: la respuesta se tira y el tipo sigue debil.
After: el valor sale certificado del borde.

```ts
// Before: el compilador no aprende nada.
export function notify(rawEmail: string): void {
  if (isValidEmail(rawEmail)) console.log(`sending to ${rawEmail}`);
}

// After: downstream ya no rechequea el @.
export function notifyParsed(email: Email): void {
  console.log(`sending to ${email}`);
}
```

## 3. Division parcial vs total

Before: explota con cero y NaN.
After: el borde queda explicito en la firma.

```ts
// Before.
export function refundSharePartial(amount: number, parts: number): number {
  return amount / parts; // Infinity con cero, NaN silencioso.
}

// After: vive en domain/money.ts para usar mintCentsUnchecked del mismo modulo.
export function refundShareTotal(amount: Cents, parts: number): Result<Cents, SplitError> {
  if (!Number.isInteger(parts) || parts <= 0) return { ok: false, error: { kind: "EmptyParts" } };
  const share = (amount as number) / parts;
  if (!Number.isInteger(share)) return { ok: false, error: { kind: "NotDivisible", amount: amount as number, parts } };
  return { ok: true, value: mintCentsUnchecked(share) };
}
```

## 4. Piramide de `if` vs railway con retorno temprano

Before: niveles anidados por cada parseo.
After: happy path lineal con riel de error tipado.

```ts
// After: version recomendada con union tipada, sin forja.
export function buildOrderClean(rawEmail: unknown, rawAmount: unknown, rawUserId: unknown): Result<OrderShape, BuildOrderError> {
  const email = parseEmail(rawEmail);
  if (!email.ok) return { ok: false, error: { kind: "BadEmail", error: email.error } };
  const amount = parseCents(rawAmount);
  if (!amount.ok) return { ok: false, error: { kind: "BadAmount" } };
  const userId = parseUserId(rawUserId);
  if (!userId.ok) return { ok: false, error: { kind: "BadUserId" } };
  return {
    ok: true,
    value: {
      userId: userId.value, email: email.value, amount: amount.value, method: { kind: "cash" } as const,
    },
  };
}
```

## 5. `throw string` vs `DomainError` exhaustivo

Before: imposible clasificar por programa.
After: cada variante mapea a un status distinto.

```ts
// Before.
export function process(raw: string): string {
  if (raw === "") throw new Error("something failed");
  return raw;
}

// After.
export function classifyFailure(err: DomainError): number {
  return domainToStatus(err); // 400, 404 o 422 segun variante.
}
```

## 6. Booleanos de estado vs type-state

Before: el orden se puede olvidar.
After: el orden malo no compila.

```ts
// Before: flag manual.
export function payIfSubmitted(order: { isSubmitted: boolean; id: string }): string {
  if (!order.isSubmitted) throw new Error("not submitted");
  return `paid ${order.id}`;
}

// After: solo StagedOrder<Submitted> entra.
export function payOrder(order: StagedOrder<Submitted>): StagedOrder<Paid> {
  return { id: order.id, amount: order.amount, stage: "paid", [StageTag]: { stage: "paid" } as Paid };
}

// Politica vs forma: ExceedsMax 422 no es InvalidAmount 400.
export function classifyPolicy(max: Cents): DomainError {
  return { kind: "ExceedsMax", max };
}
```

## 7. `safeParse` repetido vs parse-once

Before: el hot path paga el parseo tres veces.
After: el brand viaja probado sin revalidar.

```ts
// Before: redundante.
export function chargeTwice(rawAmount: unknown): void {
  const first = AmountSchema.safeParse(rawAmount);
  if (!first.success) return;
  const second = AmountSchema.safeParse(first.data);
  if (!second.success) return;
}

// After: una sola prueba.
export function chargeOnce(rawAmount: unknown): void {
  const parsed = parseCents(rawAmount);
  if (!parsed.ok) return;
  applyCharge(parsed.value);
}
```

<## 8. Driver hardwireado vs `OrderRepository` (variante nullable local)

Before: el handler importa el driver y fabrica el balance.
After: la factory recibe el puerto y carga persistencia real.

```ts
// Before: acoplado y sin 404 real.
const refund = calculateRefund(
  { orderId: orderId.value, balance: mintCentsUnchecked(10_000), alreadyRefunded: false },
  amount.value, { maxCents: mintCentsUnchecked(500_000) },
);

// After: puerto con 404 y 500 tipados, variante nullable.
// Canonico en REFERENCE usa Result<OrderSnapshot, AppError>; esta variante usa null para codebases pequenos.
export interface OrderRepositoryNullable {
  find(orderId: OrderId): Promise<OrderSnapshot | null>;
}

export function createRefundHandler(deps: { repo: OrderRepositoryNullable; policy: RefundPolicy }): Hono {
  const app = new Hono();
  app.post("/refund", async (c) => {
    // parse borde aqui, luego:
    // const order = await deps.repo.find(orderId.value).catch((cause) => { throw { kind: "Database", cause }; });
    // if (order === null) return c.json({ error: "user not found" }, 404);
    // return calculateRefund(order, amount.value, deps.policy);
  });
  return app;
}

// Prod usa PostgresOrderRepository, tests usan InMemoryOrderRepository con seed.
```

## 9. Payload stringly vs brands `LastFour` / `Iban`

Before: cualquier string pasa como metodo de pago.
After: solo valores parseados llegan al core.

```ts
// Before: stringly, `"12"` compila.
export type PaymentMethodOld =
  | { readonly kind: "card"; readonly lastFour: string }
  | { readonly kind: "transfer"; readonly iban: string }
  | { readonly kind: "cash" };

// After: `parseLastFour("12")` retorna Err, `parseLastFour("4242")` retorna Ok.
// `parseIban` exige 15-32 + `/^[A-Z]{2}[0-9A-Z]+$/i`.
export function payWith(method: PaymentMethod): number {
  return feeFor(method); // exhaustivo via assertNever
}
```

## 10. Status `number` vs `HttpStatus` cerrado

Before: `domainToStatus` retorna `number`, `return 999` compila.
After: tabla unica `HttpStatus = 400 | 404 | 422 | 500`, `return 999` falla en `tsc --strict`.

```ts
// After: importar de `domain/status.ts`, sin casts `as` en el handler.
import { domainToStatus, type HttpStatus } from "./domain/status.js";
export function statusFor(e: DomainError): HttpStatus {
  return domainToStatus(e);
}
```

## 11. Snapshot inline vs puerto `OrderRepository` (canonico Result)

Before: handler fabrica `{ balance: mintCentsUnchecked(10_000) }` inline.
After: carga via puerto, `Result` distingue `UserNotFound` 404 y `Database` 500.

```ts
// After: factory con adapters Postgres e in-memory.
export function createRefundHandler(deps: AppDeps): Hono {
  void deps; // repo + policy inyectados, core puro sin cambios
  throw new Error("ver REFERENCE.md#9-arquitectura-functional-core-imperative-shell");
}
```
