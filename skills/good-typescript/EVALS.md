# Good-TypeScript - Evaluaciones

Corre cada escenario en sesion fresca con la skill instalada.
Compara contra la baseline: mismo prompt sin la skill.
La skill gana solo si cumple todos los comportamientos esperados.
Re-corre cuando cambie el dominio, no solo cuando cambie la skill.

## Escenario 1: refactor de codigo defensivo

Entrada: un handler que recibe `userId: string` y `amount: number`,
repite los mismos `if` en handler y servicio, y hace `throw new Error`.

Esperado:

- [ ] Crea `Email`, `UserId` y `Cents` como brands con unico parser que retorna `Result`.
- [ ] El core acepta solo tipos probados, sin `if` duplicados aguas abajo.
- [ ] `npx tsc --noEmit`, `npx eslint . --max-warnings 0` y `npx vitest run` pasan.
- [ ] Ningun grep de fuga imprime lineas fuera de lugar (`safeParse` solo en parsers de dominio, `as` solo en dominio, `throw` solo en `assert.ts`).

## Escenario 2: errores estratificados

Entrada: dominio que hace `throw new Error(string)` y un `catch` unico que mapea todo a 400.

Esperado:

- [ ] Union exhaustiva `DomainError` con discriminante `kind`; sin `throw` en el dominio.
- [ ] `AppError` envuelve infra una vez con `cause`; logs solo en el edge.
- [ ] 400, 404 o 422 por variante de dominio; 500 generico para infra.
- [ ] Agregar una variante rompe el `switch` en compilacion via `assertNever`.
- [ ] El handler es `createRefundHandler({ repo, policy })`; `PostgresOrderRepository` solo en prod y `InMemoryOrderRepository` en tests HTTP; driver que lanza es `Database` 500 y `null` es `UserNotFound` 404; nunca fabrica `Order` desde el request.

## Escenario 3: type-state en workflow ordenado

Entrada: flags `isSubmitted` e `isPaid` revisados con `if` antes de cada accion.

Esperado:

- [ ] Estados `StagedOrder<Draft>` a `StagedOrder<Paid>` con llave `StageTag` exportada solo por `declaration:true`.
- [ ] Pagar un draft o pedir recibo antes de tiempo da error de tipo (`@ts-expect-error` lo documenta).
- [ ] Sin `safeParse` repetido: el brand viaja probado por referencia.
- [ ] Adversarial: JSON malformado 400 generico sin filtrar `error.message`, email malo 400, monto negativo 400, `orderId` malformado `InvalidOrderId` 400 vs fila faltante `UserNotFound` 404, doble refund `AlreadyRefunded` 422, exceso de politica `ExceedsMax` 422.

## Escenario 4: sincronia post-#15/#13 (payments, status, ports)

Entrada: `PaymentMethod` con `lastFour: string`, `domainToStatus: number`, handler con snapshot inline.

Esperado:

- [ ] `LastFour` / `Iban` como brands en `brand.ts` con `parseLastFour` (`/^[0-9]{4}$/`) y `parseIban` (15-32, `/^[A-Z]{2}[0-9A-Z]+$/i`). PoC `"12"` ya no compila como `LastFour`.
- [ ] `ok<const T>` / `err<const E>` con nota TS 5.0+. PoC literal preservado pasa `tsc --strict` 5.5 + 7.x.
- [ ] `HttpStatus = 400 | 404 | 422 | 500` en `domain/status.ts` como tabla unica. `domainToStatus`, `appToStatus`, `ErrorReport.status: HttpStatus`. PoC `return 999` falla. Sin casts `as` en handler. Sin `makeStringBrand` ni `EmailParser` en el repo.
- [ ] Puerto `OrderRepository` con `find(orderId): Promise<Result<OrderSnapshot, AppError>>`, adapters Postgres e in-memory, handler via factory. `null` mapea a `UserNotFound` 404 y `throw` a `Database` 500. Verificacion: extraer bloques y pasar `tsc --strict`, diff contra `content/en|es/post/*-stop-validating-everywhere/index.md` post-#17/#19.

## Escenario 5: secretos y lints (v1.3.0)

Entrada: handler que interpola `Email` en logs y usa `maybe!` en dominio/core.

Esperado:

- [ ] PII envuelto en clase opaca `CustomerEmail` (`#inner`, `toString`/`toJSON` redactados, `exposeForSending`); `Brand<string, "Email">` guardado para hot paths.
- [ ] PoC `sendEmail(customer)` falla con `TS2345`; `` `sending to ${customer}` `` y `JSON.stringify(customer)` muestran `[redacted]`.
- [ ] `eslint.config.mjs` con `no-non-null-assertion` + `no-restricted-syntax` para `TSAsExpression`; `npx tsc --noEmit`, `npx eslint . --max-warnings 0` y `npx vitest run` pasan.
- [ ] PoC `maybe!` en dominio/core lo marca eslint; `!` solo en fixtures/tests con disables a nivel de archivo.

## Prueba de disparo

Debe activarse con: "quita las validaciones repetidas en este handler TypeScript",
"modela email como branded type", "haz estos errores exhaustivos".
No debe activarse con: codigo Python o Rust, logica ya tipada,
preguntas de infraestructura sin invariantes de dominio.
Si dispara de mas, estrecha la descripcion. Si no dispara, agrega la frase del usuario.
