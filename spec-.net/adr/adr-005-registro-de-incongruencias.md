# ADR-005 · Registro de incongruencias del spec

- **Estado:** propuesta — pendiente de decisión del propietario del spec
- **Contexto:** al leer el spec en el orden del `README` (constitución → spec → plan/arquitectura → datos/contrato → tareas → ADR)
  aparecieron contradicciones entre documentos, vacíos y ambigüedades. El artículo X dice que un documento roto se corrige, pero
  corregirlo no le corresponde a quien implementa: cambiar el spec por cuenta propia sería decidir por el propietario.
- **Decisión:** **no se modifica ningún documento existente.** Cada incongruencia se registra aquí con su evidencia (archivo y
  sección) y su impacto, para que el propietario la resuelva. Mientras no se decida, la implementación no asume en silencio:
  cualquier supuesto que tome se declara en el código o en el `README` del repositorio correspondiente.
- **Alternativas descartadas:** corregir los documentos directamente (sustituye la decisión del propietario); ignorar las
  incongruencias y elegir en silencio (deja supuestos invisibles, contrario al artículo XII).
- **Consecuencias:** el spec queda tal como se entregó; este registro es la única fuente de las discrepancias conocidas.
- **Cómo se verifica:** cada fila cita el lugar exacto del spec donde se puede comprobar.

Tipos: **Contradicción** (dos documentos dicen cosas incompatibles) · **Vacío** (algo se usa o se exige, pero no se define) ·
**Ambigüedad** (admite más de una lectura razonable).

---

## 1. Incongruencias internas del spec

| # | Tipo | Dónde | Qué dice | Por qué no cuadra | Impacto |
|---|---|---|---|---|---|
| INC-01 | Contradicción | [ADR-002](adr-002-concurrencia-optimista.md); `plan.md` D-04 vs. `data-model.md` §3 | El libro lleva un **testigo de versión** | La tabla `book` de §3 no tiene columna de versión, y `data-model.md` dice ser «el único sitio donde vive el modelo de datos» | No se puede implementar la concurrencia optimista (RNF-2, T-13) sin salirse del modelo documentado |
| INC-02 | Contradicción | `data-model.md` (encabezado) vs. `constitution.md` art. X y `data-model.md` §10 | El encabezado dice «si algo contradice a este documento, **gana este documento**» | El artículo X y el propio §10 dicen «**gana el motor**: el documento está roto y se corrige el documento» | No queda claro qué prevalece ante una discrepancia entre documento y base |
| INC-03 | Vacío | `api-contract.md` §3.4 | «Para pintar la portada se usa `GET /books/{id}/cover`» | Ese endpoint no aparece en la tabla de rutas de §3.4 (solo `PUT` y `DELETE` sobre `/cover`); no tiene rol definido | Sin el endpoint, el frontend no puede mostrar portadas; T-16 («recorre todas las rutas») no lo cubre |
| INC-04 | Contradicción | `spec.md` CA-10.4 y `arquitectura-panoramica.md` idea 4 vs. `spec.md` §2, `data-model.md` §1 y §9 | El reporte no cambia «aunque luego se renombre el libro **o su categoría**» | Las categorías son cinco, fijas, sembradas y «sin mantenimiento»; no hay ningún endpoint para renombrarlas | Parte del criterio de aceptación no se puede producir ni probar por la API |
| INC-05 | Contradicción | `spec.md` CA-10.1 vs. `api-contract.md` §3.7 y [ADR-004](adr-004-reporte-agregado-y-congelado.md) | CA-10.1: una fila por **libro y nombre de categoría** congelado | El contrato y el ADR-004 agrupan por `book_id + book_title + category_name`: el título también es clave de agrupación | Si un libro se renombra dentro del rango, CA-10.1 da una fila y el contrato da dos |
| INC-06 | Vacío | `tasks.md` T-21 vs. `api-contract.md` §3.6 y `data-model.md` §9 | El sembrador crea **3 préstamos vencidos usando solo la API** | `POST /loans` no recibe `loaned_at`: el servidor usa el instante actual y el vencimiento es a 14 días. Por la API no se puede crear un préstamo ya vencido | La evidencia de T-21 («`/loans/overdue` devuelve 3 líneas») no se puede cumplir con las reglas tal como están |
| INC-07 | Ambigüedad | `spec.md` CA-03.5 y `api-contract.md` §2 vs. `api-contract.md` §3.6 | Un libro con **préstamos abiertos** no se da de baja | En el contrato, `open` es un estado específico, distinto de `overdue`. Leído al pie de la letra, un libro con un préstamo **vencido** sí se podría dar de baja | Riesgo de dar de baja libros que siguen prestados |
| INC-08 | Contradicción | `api-contract.md` §1 vs. §3.7 | Fechas: «ISO 8601 en UTC, con `Z`» | El reporte usa `from`/`to` como fecha sin hora (`2026-10-01`), «ambos inclusive», mientras que `loaned_at` es un instante (`timestamptz`) | No se define cómo se interpreta el límite `to` (¿hasta las 23:59:59 UTC? ¿`< to + 1 día`?); hay riesgo de perder el último día |
| INC-09 | Contradicción | `spec.md` RNF-1 vs. `tasks.md` T-21 | La búsqueda se mide «con 5 000 libros» usando «el sembrador de `tool`» | T-21 define el sembrador con «≥ 40 libros» | No hay forma documentada de producir el volumen que exige la medición |
| INC-10 | Contradicción | [ADR-001](adr-001-propiedad-del-esquema.md) y el `README` de `infra` vs. `tasks.md` T-20 | `infra` «**solo** crea el motor vacío, el usuario y el volumen» | T-20 pide que `docker compose up` levante «motor vacío, **API y frontend**» | El alcance del repositorio `infra` no es consistente |
| INC-11 | Vacío | [ADR-001](adr-001-propiedad-del-esquema.md), `data-model.md` §0, `tasks.md` T-17 y T-20, `spec.md` RNF-5 | Todo vive en el esquema `library`; el motor arranca vacío; el sistema «levanta con un solo comando» | No se dice quién crea el esquema `library` (infra o la migración) ni quién ejecuta las migraciones al arrancar | T-17 («se puede iniciar sesión como admin») depende de un paso no asignado |
| INC-12 | Ambigüedad | [ADR-002](adr-002-concurrencia-optimista.md) | Ante un conflicto de versión responde 409 «(o reintenta una vez si el ejemplar sigue disponible)» | Deja el comportamiento como opcional y no dice quién decide | Dos implementaciones válidas se comportan distinto ante el mismo conflicto |
| INC-13 | Vacío | `api-contract.md` §1 y `spec.md` CA-04.1 | `size` tiene un máximo de 50 | No se dice si `size > 50` responde 422 o se recorta a 50, ni cuál es el valor por defecto | Respuesta indefinida ante una entrada fuera de rango |
| INC-14 | Vacío | `spec.md` CA-06.4 vs. `api-contract.md` §1 | Responde 422 «con el conteo actual» | El cuerpo de error único (`code`, `message`, `details`) no tiene un campo para el conteo; no se dice si va en `message` o en `details` | La prueba de contrato no puede verificar el conteo |
| INC-15 | Vacío | `api-contract.md` §3.6 | `GET /loans/overdue` devuelve `items` | No se dice si está paginado (`page`, `size`, `total`) como los demás listados | Forma de respuesta indefinida |
| INC-16 | Contradicción | `tasks.md` T-06 vs. `architecture.md` §3 | T-06 (dominio) exige como evidencia «rol `admin` por API rechazado» | El agregado `User` acepta `admin` y `librarian`; rechazar `admin` es una regla de la API/aplicación (CA-02.3), no del dominio | La evidencia de T-06 está en la capa equivocada |
| INC-17 | Contradicción | `architecture.md` §3 y §4 (versión Python) vs. `constitution.md` art. XI | `Loan.AddItem`, `Book.Withdraw`, `Lend(memberId, bookIds)` | En el spec de Python los nombres siguen la convención de .NET (PascalCase, camelCase) y no la de Python | Menor: nombres que no corresponden al stack que dice describir |
| INC-18 | Vacío | `spec.md` CA-02.3 vs. `api-contract.md` §2 | Pedir el rol `admin` responde 422 | No se dice qué `code` lleva (¿`VALIDATION_ERROR`?) | Código de error indefinido |
| INC-19 | Vacío | `api-contract.md` §1 (códigos de estado) vs. §2 | 401 aplica a «sin token, token inválido o vencido, credenciales inválidas» | El único `code` con 401 es `INVALID_CREDENTIALS`, con origen CA-01.2 (inicio de sesión). No hay código para «sin token» ni «token vencido» | Se responde `INVALID_CREDENTIALS` por ser el único 401 definido |
| INC-20 | Vacío | `spec.md` RNF-4 vs. `api-contract.md` §1 y §2 | «Todo error HTTP usa el mismo cuerpo» | No hay `code` ni estado definido para un error interno inesperado (500) | Un 500 no puede cumplir el cuerpo único sin inventar un código |
| INC-21 | Vacío | `api-contract.md` §1 y `data-model.md` §5 vs. `constitution.md` art. II y IV, `architecture.md` §2 | Los identificadores son UUID v4 generados por la aplicación | Ningún puerto ni componente del núcleo está asignado a generarlos (la lista de puertos no lo incluye) | No está definido dónde se generan los identificadores |
| INC-22 | Ambigüedad | `api-contract.md` §4 vs. `spec.md` CA-01.1 | El token lleva «`sub` y `role`» | La vigencia de 8 horas exige además un vencimiento (`exp`) dentro del token, que el contrato no menciona | La lista de datos del token está incompleta |
| INC-23 | Contradicción | `data-model.md` §4 vs. §0 | Las claves primarias se llaman `PK_*` | §0 exige `snake_case` en minúsculas y solo define el patrón de nombre `ck_{tabla}_{regla}`; no define nombres para FK ni índices | Convención de nombres inconsistente e incompleta |
| INC-24 | Vacío | `plan.md` §2 vs. `arquitectura-panoramica.md` | El frontend llama a la API por HTTP `/api/v1` | La configuración por entorno no incluye ninguna variable para que el frontend conozca la URL de la API, ni los puertos de los servicios | El frontend no tiene forma documentada de ubicar la API |
| INC-25 | Ambigüedad | `plan.md` D-09 de `spec-python` vs. de `spec-.net` | Python: Argon2id. .NET: BCrypt (factor 12) | El `README` dice que ambas versiones describen el mismo producto, pero el reto (Laravel) no dice cuál de los dos algoritmos aplica | Elección de algoritmo de hash sin criterio documentado |
| INC-26 | Vacío | `api-contract.md` §1 (códigos de estado) | Lista 200/201/204, 401, 403, 404, 409 y 422 | No define 405 (método no permitido, p. ej. `DELETE /members/{id}`, CA-05.3) | Esa respuesta no puede usar el cuerpo único sin inventar un código |
| INC-27 | Vacío | `data-model.md` §6 Q4 vs. `architecture.md` §2 | Q4: «Listar categorías», frecuencia alta | La lista de puertos no tiene repositorio ni lector de categorías | No está asignado qué puerto atiende `GET /categories` |
| INC-28 | Vacío | `spec.md` CA-05.2 vs. CA-04.1 | Socios: «se lista y se busca, paginado» | Los libros tienen orden definido (por título); los socios no | Paginación de socios sin orden documentado |
| INC-29 | Vacío | `api-contract.md` §3.4 y §3.5 | Define la petición de `POST /books`, `PUT /books/{id}` y `POST /members` | No define el cuerpo de su respuesta (solo «Libro (respuesta)» como forma general) | Respuesta de creación y edición sin contrato explícito |
| INC-30 | Vacío | `api-contract.md` §1 (paginación) | `?page=1&size=20`, `size` máximo 50 | No dice si 20 es el valor por defecto o solo un ejemplo | Tamaño de página por defecto indefinido |
| INC-31 | Vacío | `spec.md` HU-06 vs. `api-contract.md` §2 y §3.6 | `POST /loans` recibe `member_id` y `book_id` | No define la respuesta cuando el socio o un libro no existe o está dado de baja | Se responde 404 `NOT_FOUND` por ser el código genérico de recurso inexistente |
| INC-32 | Vacío | `spec.md` CA-06.1 vs. `api-contract.md` §2 | Un préstamo trae de 1 a 3 líneas | Ningún `code` está asociado a CA-06.1 (0 líneas o más de 3) | Se responde `VALIDATION_ERROR` por ser una regla de entrada |
| INC-33 | Vacío | `spec.md` CA-09.1 y CA-09.2 vs. `api-contract.md` §3.6 | Vencidos con `days_late`, ordenados por más días de retraso | No define cómo se cuentan los días (períodos completos de 24 h o días de calendario) ni el orden en caso de empate | Dos implementaciones pueden dar `days_late` y orden distintos |
| INC-34 | Vacío | `spec.md` CA-08.2 vs. `api-contract.md` §3.6 | `GET /loans` filtra por `from`, `to`, `status`, `member_id` | No define si `from`/`to` son fechas o instantes, si se puede enviar uno solo, ni la forma de cada elemento de la lista | Filtro y respuesta del listado sin contrato explícito |

## 2. Riesgos al traducir el spec a React + Laravel

No son errores del spec: son puntos en los que el comportamiento por defecto de Laravel choca con lo especificado. Se registran
para que no se cuelen sin darse cuenta.

| # | Lo que exige el spec | Lo que hace Laravel por defecto |
|---|---|---|
| TR-01 | Exactamente seis tablas (`data-model.md` §2) | Trae migraciones de `users`, `password_reset_tokens`, `sessions`, `cache`, `jobs` |
| TR-02 | Tablas en singular y en el esquema `library` (§0) | Nombres en plural (`users`, `books`) en el esquema `public` |
| TR-03 | Sin `created_at`/`updated_at` (§3, §8) | `$table->timestamps()` y modelos con marcas de tiempo activadas |
| TR-04 | Ninguna columna con `DEFAULT` (§3, T-02) | Algunos tipos y helpers de migración agregan valores por defecto |
| TR-05 | Columna `password_hash`; ids UUID (§3) | Columna `password`; ids autoincrementales |
| TR-06 | JWT de 8 horas con `sub` y `role` (`api-contract.md` §4) | Sanctum emite tokens opacos guardados en tabla |
| TR-07 | Cuerpo de error único `{code, message, details}` (§1) | Validación responde `{message, errors}`; otras excepciones tienen formatos propios |
| TR-08 | `GET /loans/overdue` y `GET /loans/{id}` conviven (§3.6) | Si `/loans/{id}` se registra primero, captura `overdue` como id |
| TR-09 | El sembrador usa la API, nunca SQL (`data-model.md` §9) | La costumbre es `php artisan db:seed`, que escribe directo en la base |
| TR-10 | Pruebas de arquitectura (artículos I a III) con `import-linter` / NetArchTest | Requiere un equivalente en PHP (por ejemplo, pruebas de arquitectura de Pest o Deptrac) |
| TR-11 | Exactamente seis tablas (`data-model.md` §2) | Laravel necesita su tabla de control `migrations` para registrar las migraciones aplicadas |
| TR-12 | Variable `DATABASE_URL` (`plan.md` §2) | Laravel lee `DB_URL` y `DB_HOST`/`DB_DATABASE`/… |
| TR-13 | Solo los endpoints de `api-contract.md` §3 | Laravel trae la ruta `/up`, una página de bienvenida (`/`) y archivos de frontend (Vite) |
