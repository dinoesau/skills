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

## Escenario 3: type-state en workflow ordenado

Entrada: flags `is_submitted` e `is_paid` revisados con `if` antes de cada accion.

Esperado:

- [ ] Estados `Order<Draft>` a `Order<Paid>` con transiciones que mueven `self`.
- [ ] Reusar un valor viejo no compila.
- [ ] Metodos por estado solo existen en el estado correcto (`receipt` solo en `Paid`).

## Prueba de disparo

Debe activarse con: "quita las validaciones repetidas en este handler Rust",
"modela email como tipo", "haz estos errores exhaustivos".
No debe activarse con: codigo Python o TypeScript, logica ya tipada,
preguntas de infraestructura sin invariantes de dominio.
Si dispara de mas, estrecha la descripcion. Si no dispara, agrega la frase del usuario.
