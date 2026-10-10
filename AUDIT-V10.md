# Auditoría V10 — Porteadores Jacó (segundo revisor)

Auditoría realizada sobre el paquete `Servicio-Taxi-Porteadores-Jaco-V10-PRO-CLAUDE-AUDIT.zip`,
siguiendo `CLAUDE.md` y `CLAUDE-CODE-TASK.md`. No se reconstruyó la PWA, no se cambió la
arquitectura de Supabase, no se sustituyó `public_token` por Anonymous Auth y no se eliminó
funcionalidad existente.

## Hallazgos y corrección

### 1. CRÍTICO — Flujo demo del cliente roto
`createCustomerRide()` llamaba directamente a `db.rpc('customer_create_ride', ...)` sin
comprobar `demoMode`/`db`. Con `config.js` en modo placeholder (`db === null`, tal como lo
tiene el `config.js` externo del propio paquete), pulsar "QUIERO UN TAXI" lanzaba una excepción
no controlada. La función `startDemoRide()` ya existía en el código pero nunca se invocaba.
**Corrección:** se agregó `if(demoMode||!db){startDemoRide(btn);return;}` antes de la llamada RPC,
después de obtener el GPS y antes de contactar Supabase.

### 2. ALTO — Pantalla de cliente sin estilos
Comparación sistemática (script) de todas las clases usadas en `app.js` contra `styles.css`:
faltaban `.customer-hero`, `.customer-map-wrap`, `.request-sheet`, `.gps-chip`, `.central-chip`,
`.customer-brand-overlay`, `.contact-strip`, `.demo-driver`, `.tracking-sheet`, `.map-topline`
(y modificadores menores `.kpi-main`, `.ride-main`, `.alerts-card` del panel central). La
pantalla principal del cliente —punto de entrada del servicio— se renderizaba como texto sin
diseño. **Corrección:** bloque nuevo en `styles.css` con estilos coherentes con la identidad
visual existente (variables `--yellow`/`--black`/`--radius`/`--shadow` ya definidas).

### 3. MEDIO — Manifest embebido sin `icons`
`index.html` es autocontenido: descomprime CSS/config/JS embebidos (gzip+base64) y también
usa un manifest como *data URI* en vez de cargar `manifest.webmanifest`. Ese data URI no traía
el arreglo `icons`, aunque el archivo externo sí lo tenía — una PWA instalada quedaría sin
íconos. **Corrección:** se regeneró el data URI incluyendo el mismo `icons` que
`manifest.webmanifest`.

### 4. MEDIO — Falta `apple-touch-icon`
Existe `assets/porteadores-apple-touch.png` mencionado en `sw.js` (se precachea), pero
`index.html` no tenía `<link rel="apple-touch-icon">`. En iOS Safari, "Agregar a inicio" no
usaría el ícono de marca. **Corrección:** se agregó el `<link>` correspondiente.

### 5. Observación — payload de config embebido no es placeholder
El `config.js` embebido en `index.html` contiene una URL y `publishable key` de Supabase reales
(distintos del `config.js` externo, que sí trae placeholders). Se confirmó que es la clave
pública/anon (protegida por RLS), no una `service_role`; esto es el uso previsto de Supabase y
no se modificó. Se documenta para que quien opere el proyecto sepa que el paquete YA apunta a
un proyecto Supabase específico a través del HTML, aunque el `config.js` de referencia no lo
refleje.

### 6. Observación — sin cambios de seguridad en el esquema
Se revisó `supabase/schema.sql` completo: todas las funciones `SECURITY DEFINER` fijan
`search_path=public`; `is_staff()`/`is_driver()` están revocadas de `anon`; las RPC de cliente
validan `public_token` junto con `id` (no solo el token); RLS activo en todas las tablas;
privilegios de escritura directa revocados para `anon`/`authenticated`. No se encontró un
defecto de seguridad concreto que justificara un cambio de esquema, por lo que **no se tocó**.

### 7. Observación — redundancia inofensiva en `schema.sql`
`driver_advance_ride` está definido dos veces (idéntico) en el archivo, una vez cerca de la
línea 326 y otra cerca de la 423, ambas como `create or replace function`. No es un error
funcional (la segunda definición simplemente reemplaza a la primera con el mismo cuerpo), pero
es limpieza pendiente si se reescribe el archivo en el futuro. No se tocó por no ser un defecto
real y para minimizar el diff del esquema.

## Verificación realizada
- `node --check app.js`: PASS.
- `node --check sw.js`: PASS.
- `python3 -m json.tool manifest.webmanifest`: PASS (JSON válido).
- Búsqueda de `localStorage`/`sessionStorage`/`signInAnonymously`/`ensureAnon`/`service_role`/
  generador QR en todo el proyecto: sin coincidencias.
- Cobertura de clases CSS usadas en `app.js` vs. definidas en `styles.css`: 78 clases usadas,
  0 faltantes tras la corrección (antes: 13 faltantes, 10 de ellas estructurales).
- Sincronización de payloads embebidos: se descomprimieron los tres bloques `ung('...')` de
  `index.html` y se compararon byte a byte contra `app.js`/`styles.css` actualizados — coinciden
  exactamente. El bloque de configuración embebido se dejó intacto.
- Manifest data URI decodificado y verificado como JSON válido con `icons` presente.
- `unzip -t` sobre el ZIP original de entrada: sin errores.

## No verificado (fuera del alcance de este entorno)
- Ejecución real contra un proyecto Supabase (no hay servidor Postgres/Supabase conectado en
  este entorno).
- Prueba en navegador/dispositivo real (GPS físico, instalación de PWA, service worker en
  condiciones de red reales).
- No se afirma operación LIVE; sigue pendiente toda la infraestructura externa ya descrita en
  `README.md` (proyecto Supabase exclusivo, HTTPS, usuarios reales, etc.).
