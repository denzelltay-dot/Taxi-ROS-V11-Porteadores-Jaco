# Auditoría V8 — Porteadores Jacó

## Verificaciones locales
- `node --check app.js`: PASS.
- Sin `localStorage`/`sessionStorage` en `app.js`.
- Sin escrituras directas (`insert/update/upsert/delete`) desde el frontend; las operaciones sensibles usan RPC.
- Sin generador QR ni CSS del generador dentro de la PWA.
- Sin marca/demo antigua en el código operativo.
- Service Worker actualizado a `porteadores-jaco-v8-shell`.
- `config.js` mantiene placeholders intencionalmente hasta que exista el proyecto Supabase real.

## Hardening añadido en V8
- Función `expire_driver_offers_job()` para expiración autónoma mediante Supabase Cron.
- La función de cron no es ejecutable por `public`, `anon` ni `authenticated`.
- El job sugerido corre cada minuto; la respuesta del conductor también valida expiración, por lo que la seguridad funcional no depende del minuto exacto del cron.
- Auditoría registra `source=cron` y `actor_user_id=NULL` para eventos generados por automatización.

## Pendientes externos antes de LIVE
1. Proyecto Supabase exclusivo.
2. Aplicar `schema.sql` y, si se desea, `optional_cron.sql`.
3. Habilitar Anonymous Sign-Ins y protección antiabuso.
4. Crear cuentas staff y conductores.
5. Colocar URL y publishable key reales en `config.js`.
6. Publicar en HTTPS.
7. Activar/verificar Cron si se requiere expiración autónoma.
8. Pruebas reales en teléfonos y GPS.
9. Crear QR externo definitivo apuntando a la URL HTTPS real.

## V9 — cierre para demostración comercial
- Se conserva la arquitectura y el flujo operativo; no se añade QR dentro de la PWA.
- Se conserva la política sin localStorage/sessionStorage y sin escrituras directas desde el frontend.
- La PWA incorpora un modo DEMOSTRACIÓN en memoria cuando Supabase aún no está configurado: permite enseñar mapa, ubicación, solicitud, asignación simulada y llegada sin fingir conexión con producción ni persistir datos del demo.
- El modo producción sigue dependiendo de Supabase + HTTPS y no se declara LIVE hasta conectar y probar esa infraestructura.
- La fachada usa el logo real de Porteadores Jacó incluido en assets.
- Verificaciones locales: `node --check app.js` PASS; `node --check sw.js` PASS; `unzip -t` PASS; búsqueda de localStorage/sessionStorage y escrituras Data API directas en app.js PASS (sin coincidencias).
- Pendiente externo obligatorio para operación real: crear/configurar instancia Supabase exclusiva, ejecutar schema.sql, crear usuarios/roles/conductores, configurar config.js, habilitar Anonymous Sign-Ins, revisar anti-abuso/CAPTCHA, HTTPS, tiles/mapa y probar GPS real en Android.
