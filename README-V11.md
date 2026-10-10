# TAXI ROS — V11 sobre base V10

V11 es una evolución de la PWA existente; no es una reconstrucción desde cero.

## Arquitectura
- 3 interfaces conectadas al mismo backend Supabase: Pasajero, Conductor y Central.
- Se conserva `public_token` para el flujo público del pasajero.
- No se usa Anonymous Auth para el pasajero.
- No se usa `localStorage` ni `sessionStorage` para operación de viajes/GPS.
- Se conserva Leaflet/OpenStreetMap y el service worker existente.

## Añadido en V11
- Destino en la solicitud del pasajero.
- Selección explícita de los métodos ya definidos: Efectivo, SINPE Móvil, Tarjeta sin datáfono y PayPal / Wink.
- Registro del método de pago y estado de pago en el viaje.
- Estructura de tarifa previa (`fare_estimate_crc`) sin inventar tarifas: si no existe una tarifa configurada, se muestra que debe confirmarse según la tarifa aplicable.
- Botón de emergencia para pasajero y conductor y centro de emergencias para la central.
- Llamada directa al 911 y registro operacional de la alerta; la PWA no sustituye al 911.
- Estado de verificación del conductor y vencimientos/documentación básicos.
- Bloqueo de activación operativa para conductores no verificados en la migración V11.
- Política de privacidad y términos de servicio dentro de la PWA.
- Consentimiento explícito antes de crear una solicitud.
- Enlace para compartir el seguimiento del viaje.
- Corrección de un bug existente: el conductor intentaba leer elementos `#vehicle` y `#plate` que no existían en su interfaz al activar GPS.

## Migración
Ejecutar `supabase/v11_migration.sql` después del esquema existente. La migración es incremental y usa `ADD COLUMN IF NOT EXISTS`, además de crear la tabla de emergencias y RPC nuevas.

## Pagos
V11 registra la selección y estado operacional del pago. No presenta una integración de tarjeta/PayPal como si estuviera activa. La integración real con el proveedor debe conectarse posteriormente con sus credenciales/SDK/API apropiados; nunca se deben almacenar números completos de tarjeta en Supabase.

## Legal
La interfaz está orientada al marco vigente y preparada estructuralmente para futuros requisitos regulatorios, pero no se presenta como certificación legal ni como cumplimiento automático de un proyecto legislativo en trámite. La revisión jurídica y permisos/registros reales siguen siendo externos a este código.

## Verificación realizada en este entorno
- `node --check app.js`: PASS.
- `node --check sw.js`: PASS.
- JSON de manifest válido.
- Payloads comprimidos de CSS/config/JS dentro de `index.html` regenerados desde los archivos fuente.
- Búsqueda de `localStorage`, `sessionStorage` y `service_role` en el código cliente: sin coincidencias.
- Integridad del ZIP pendiente de ejecutar después del empaquetado final.
- No se afirma LIVE: no se ejecutó una prueba end-to-end contra Supabase en un teléfono real desde este entorno.


## V11.1 — correcciones de auditoría (10 oct 2026)
- Seguimiento público del pasajero actualizado mediante `customer_reconnect_ride` usando el token del viaje, en lugar de lectura directa de `ride_requests` que RLS bloquea para el visitante.
- Autorización de aceptación/rechazo de ofertas restringida al conductor propietario de la oferta; la expiración de ofertas no reactiva conductores con documentos vencidos.
- Asignación y activación bloqueadas si falta verificación, documentos o vigencias; GPS y estados se validan en servidor.
- Reasignación limitada a estados permitidos; se reusa de forma segura la oferta previa al volver a asignar al mismo conductor.
- Alerta del conductor toma coordenadas de su perfil/GPS, no las del pasajero. El mensaje de emergencia aclara que no hay garantía de notificación push y que debe llamarse al 911.
- La central dispone de un flujo para registrar la revisión manual de documentos y sus vencimientos. La central debe inspeccionar los documentos originales; esta PWA no almacena imágenes de documentos.
- Sesión de autenticación persistente para que la central/conductor no pierdan la sesión al recargar la PWA. Los datos del viaje y GPS no se guardan manualmente en localStorage.
- Caché del service worker renovada para evitar servir el shell anterior.

## Límites que siguen pendientes antes de operar con público
- No se ha confirmado/publicado este paquete en el proyecto Vercel correcto.
- No hay prueba end-to-end en un Android real con usuarios autenticados y viaje real.
- No existe envío push/SMS/llamada automática al personal ante una emergencia; se registra en la base de datos y la central debe estar abierta/operativa. El 911 sigue siendo la vía de emergencia.
- No hay procesamiento automático de pagos ni tarifa dinámica confirmada; el método se registra y el importe final debe ser confirmado por la central/conductor según la tarifa aplicable.
- La revisión documental del conductor es manual; no existe carga de documentos ni verificación externa automática.

## Estado de integración (comprobación real)
- Aplicadas en Supabase `Porteadores Jaco` las migraciones `taxi_ros_v11_1_security_workflow_hardening`, `taxi_ros_v11_1_document_review_rpc`, `taxi_ros_v11_1_assignment_retry_safety` y `taxi_ros_v11_1_expired_offer_safety`.
- El proyecto Supabase responde como `ACTIVE_HEALTHY`.
- El asesor de seguridad todavía informa advertencias de funciones `SECURITY DEFINER`: las seis operaciones públicas del pasajero usan un token aleatorio como autorización y varias operaciones autenticadas son necesarias para el flujo. No se revocaron en bloque porque eso rompería la aplicación; requieren pruebas con roles reales.
- Esto no confirma el despliegue Vercel ni reemplaza una prueba end-to-end real.
