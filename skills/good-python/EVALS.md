# Good-Python - Evaluaciones

Corre cada escenario en sesion fresca con la skill instalada.
Compara contra la baseline: mismo prompt sin la skill.
La skill gana solo si cumple todos los comportamientos esperados.
Re-corre cuando cambie el dominio, no solo cuando cambie la skill.

## Escenario 1: refactor de codigo defensivo

Entrada: un handler que recibe `dict` y `Any`, repite `if` e `isinstance`
en handler, servicio y repo, y hace `raise ValueError`.

Esperado:

- [ ] Crea `Email`, `UserId` y `Cents` como dataclasses `frozen=True, slots=True` con unico `parse` que retorna `Result`.
- [ ] El core acepta solo tipos probados, sin `if` duplicados aguas abajo.
- [ ] `mypy --strict .`, `ruff check .` y `pytest` pasan.
- [ ] Ningun grep de fuga imprime lineas fuera de lugar (`model_validate` nunca en dominio/core, `isinstance` solo en `parse_*` y railway, `raise` solo en validador que se convierte a `Result`).
- [ ] Adversarial: JSON malformado `"invalid request"` 400 sin `exc.errors()`, email malo 400, monto negativo 400, `orderId` malformado `InvalidOrderId` 400 vs `UserNotFound` 404, doble refund `AlreadyRefunded` 422, exceso `ExceedsMax` 422.
- [ ] El handler inyecta `OrderRepository` con `Depends(get_order_repository)`; `PostgresOrderRepository` solo en prod y `InMemoryOrderRepository` via `dependency_overrides` en tests; nunca fabrica `OrderSnapshot` desde el request.

## Escenario 2: errores estratificados

Entrada: dominio que hace `raise ValueError(str)` y un `except Exception` unico que mapea todo a 400.

Esperado:

- [ ] Union exhaustiva `DomainError` de dataclasses frozen; sin `raise` por outcomes de negocio.
- [ ] `DbError` o `GatewayError` envuelven infra una vez con `cause`; logs solo en el edge.
- [ ] 400, 404 o 422 por variante de dominio; 500 generico para infra.
- [ ] El dominio no importa FastAPI ni logging.

## Escenario 3: type-state en workflow ordenado

Entrada: flags `is_submitted` e `is_paid` revisados con `if` antes de cada accion.

Esperado:

- [ ] Estados `OrderState[Draft]` a `OrderState[Paid]` con genericos de etapa.
- [ ] Pagar un draft da error en `mypy --strict` y `pyright`.
- [ ] Rehidratacion desde DB usa red runtime minima (`rehydrate_paid` con `Result`).
- [ ] Sin `model_validate` repetido: el value object viaja probado por referencia.

## Escenario 4: sincronia post-#16/#20/#13 (payments, status, ports)

Entrada: `last_four: str`, `domain_to_status -> int`, handler con snapshot inline.

Esperado:

- [ ] `LastFour` / `Iban` frozen con `parse_last_four` (`fullmatch [0-9]{4}`) / `parse_iban` (15-32, `^[A-Z]{2}[0-9A-Z]+$` IGNORECASE). `Card` / `Transfer` con `kw_only=True` sin defaults. PoC `mypy --strict` (`Card(last_four="12")` falla `[arg-type]`) + matriz runtime.
- [ ] `HttpStatus = Literal[400, 404, 422, 500]` en `domain_to_status`, `app_to_status`, `report_app_error`. PoC `= 999` falla `[assignment]`; cubrir las 8-10 variantes.
- [ ] `parse_refund_request` con `InvalidUser(detail)` e `InvalidRequest(detail)`, `Err([InvalidRequest(...)])` para shape. PoC `mypy --strict` de la cadena parsers a mensajes a status.
- [ ] Handler con puerto `Protocol` + `Depends` + fake in-memory, `OrderId.parse -> InvalidOrderId` 400, `None -> UserNotFound` 404, una sola anotacion `err`. `Slug` con `SLUG_PATTERN` compartido, `cast` con comentario de invarianza, `_mint_after_check` privado, narrowing-`assert` solo tras chequeo exhaustivo.
- [ ] Prosa de exhaustividad acredita `assert_never`, no `match` sin wildcard. Verificacion: extraer bloques y pasar `mypy --strict`, diff contra posts post-#17/#19.

## Escenario 5: secretos y lints (v1.3.0)

Entrada: handler que hace `print(f"sending to {email}")` y construye `Email(_value="x")` fuera del modulo.

Esperado:

- [ ] Envuelve PII en `CustomerEmail` con `__str__`/`__repr__` redactados y `expose_for_sending`; tokens con `SecretStr` de pydantic.
- [ ] PoC `f"{customer}"` imprime `[redacted]`; `f"{email}"` solo en contextos no-PII como recibos.
- [ ] Config concreta `[tool.mypy] strict` + `[tool.ruff.lint] select SLF`; `mypy --strict .` y `ruff check .` pasan.
- [ ] PoC acceso a `._value` fuera del modulo definidor lo marca `ruff` SLF.

## Prueba de disparo

Debe activarse con: "quita las validaciones repetidas en este handler Python",
"modela email como value object", "haz estos errores exhaustivos".
No debe activarse con: codigo TypeScript o Rust, logica ya tipada,
preguntas de infraestructura sin invariantes de dominio.
Si dispara de mas, estrecha la descripcion. Si no dispara, agrega la frase del usuario.
