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
