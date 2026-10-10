# Servicio de Taxi Porteadores Jacó — V8 producción

Versión endurecida del sistema operativo de Central / Conductor / Cliente.

# Servicio de Taxi Porteadores Jacó — PWA REAL / V8

## Estado
Paquete preparado para operación real, **pendiente únicamente de infraestructura externa**: proyecto Supabase exclusivo, credenciales públicas de ese proyecto y publicación HTTPS. No se simula LIVE mientras esas piezas no estén conectadas.

## Flujo público
1. QR externo abre la PWA (el QR NO se genera dentro de la PWA).
2. Cliente pulsa **QUIERO UN TAXI**.
3. Se solicita GPS del teléfono.
4. El cliente público no usa Anonymous Auth; la continuidad de la solicitud usa `public_token`.
5. `customer_create_ride` crea la solicitud y devuelve un token público de alta entropía.
6. El token queda en la URL para continuidad sin `localStorage`.
7. Central ve solicitudes y conductores con GPS reciente.
8. Central asigna el conductor más cercano elegible; la asignación es atómica.
9. El conductor recibe una oferta de 30 s y acepta/rechaza desde su panel.
10. El cliente recibe posición del conductor mediante una consulta segura que expone solo el conductor asignado.
11. Conductor: en camino → llegó → viaje iniciado → finalizado.
12. La tarifa final se registra al terminar; no se calcula ni muestra un precio inventado por adelantado.

## Seguridad aplicada
- Sin `localStorage` ni `sessionStorage` para operación de viajes/GPS.
- Persistencia de sesión Supabase desactivada en frontend (`persistSession:false`).
- Sin escrituras directas del frontend a tablas operativas; las mutaciones usan RPC.
- RPC sensibles con `SECURITY DEFINER`, `search_path=public`, autorización por rol/propietario/conductor.
- Revocados privilegios Data API de insert/update/delete para `authenticated`.
- RLS activado en todas las tablas operativas.
- Control de concurrencia con filas bloqueadas y transiciones de estado.
- Oferta con expiración de 30 s.
- Índice único para impedir más de un viaje activo por cliente y más de un viaje operativo simultáneo por conductor.
- GPS stale > 30 s no se considera elegible para nueva asignación.
- El cliente no puede consultar la tabla completa de conductores: obtiene solo el conductor asignado mediante RPC.
- Tarifa final limitada a ₡1,000,000 y validada en servidor.
- Eventos operativos registrados en `ride_events`.
- Token público de 18 bytes aleatorios codificados base64url.

## Limitaciones reales que NO se deben ocultar
- Un navegador móvil no garantiza GPS continuo cuando la PWA queda completamente en segundo plano o la pantalla se bloquea. Para seguimiento continuo de conductores a nivel Uber, una app nativa/servicio de ubicación en segundo plano sería la solución futura.
- Leaflet + OpenStreetMap sirve para piloto. Para tráfico alto/operación comercial debe revisarse el proveedor de tiles, límites de uso, atribución y SLA.
- Anonymous Sign-In público debe protegerse contra abuso (CAPTCHA/Turnstile u otra medida compatible) antes de abrir el QR al público.
- La expiración automática de ofertas puede reforzarse con un scheduler/Edge Function. Mientras no exista, la central expira ofertas al actualizar y el conductor valida la expiración al responder.

## Configuración antes de LIVE
1. Crear un **proyecto Supabase exclusivo para Porteadores Jacó** y confirmar el costo/plan antes de usarlo.
2. Mantener Anonymous Sign-Ins fuera del flujo público; la PWA usa RPC tokenizadas con `public_token`.
3. Ejecutar/validar únicamente migraciones necesarias; no reemplazar el esquema existente a ciegas.
4. Crear usuarios Auth para staff y conductores y registrar sus UUID en `user_roles`/`drivers` de forma controlada.
5. Copiar la URL del proyecto y la **publishable key** a `config.js`.
6. Nunca poner `service_role`/secret key en `config.js` ni en el navegador.
7. Publicar el contenido por HTTPS.
8. Crear el QR externo apuntando a la URL HTTPS real, con parámetros opcionales `partner`, `location` y `campaign`.
9. Verificar en teléfonos reales: permiso GPS, cliente, central, conductor, asignación, rechazo/expiración, llegada, inicio, finalización, reconexión por URL y pérdida/reconexión de red.
10. Revisar políticas/uso de mapas antes de abrir tráfico público.

## Rutas
- Cliente: `/`
- Central: `/#central`
- Taxista: `/#driver`

## Verificación interna realizada sobre este paquete
- `node --check app.js`: PASS.
- `node --check sw.js`: PASS.
- No hay `localStorage`/`sessionStorage` en `app.js`.
- No hay `.insert()`, `.update()`, `.upsert()` ni `.delete()` directos en `app.js`.
- Todas las mutaciones operativas usadas por la PWA pasan por RPC.
- No hay generador QR ni librería QR en la PWA.
- No hay marca demo/genérica de la versión original.
- Service worker actualizado a `v8` con estrategia network-first para núcleo de la PWA.
- Se generaron iconos PWA desde el logo limpio de Porteadores Jacó.
- La sintaxis SQL fue revisada estructuralmente; este entorno no dispone de un servidor PostgreSQL/Supabase conectado, por lo que **no se afirma una ejecución real de SQL**.


## Automatización de expiración

`supabase/optional_cron.sql` incluye una función interna para que Supabase Cron expire ofertas pendientes aunque el panel Central esté cerrado. La activación del job se hace en el proyecto Supabase real después de habilitar `pg_cron`; no se expone ninguna credencial al frontend.

## Estado de producción

El paquete queda preparado para conexión, pero no se marca LIVE mientras `config.js` siga con placeholders, no exista el proyecto Supabase definitivo y no exista una URL HTTPS publicada y verificada.

## Demostración comercial antes de conectar Supabase
Si `config.js` conserva placeholders, la PWA entra automáticamente en modo DEMOSTRACIÓN. En este paquete queda configurado el proyecto Supabase de Porteadores Jacó. Este modo es deliberadamente en memoria: permite enseñar el recorrido cliente → GPS/mapa → solicitud → taxi acercándose → llegada sin simular una conexión backend real ni guardar datos en el teléfono. Al configurar Supabase, el modo demo desaparece y el flujo real usa las RPC/RLS descritas en este documento.
