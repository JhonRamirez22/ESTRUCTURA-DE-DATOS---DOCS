# Modernización: estado verificable

## Base reproducible — 18 de octubre de 2026

- GitHub/local partían de `d0b2450`; AWS ejecuta la imagen
  `sha256:d994c1bbf6dbc16d965011b2d85897d4b17bdc0ed91c37d5a3651ecc01c3eb20`.
- Commit `6499c69`: recupera de esa imagen TLS/asyncpg, geocodificación,
  asignación por tiempos viales, observaciones TomTom y fecha de optimización.
- Conserva la secuencia real de producción: `0007_shared_chat_rate_limit` →
  `0008_route_optimized_at` → `0009_email_password_accounts`.
- No se modificó producción, no se copiaron secretos y no se reescribieron commits.
  El índice local previo está preservado; sus eliminaciones preparadas no forman
  parte de estos commits.

## Sesiones y versión HTTP

- `/api/v1/` es canónico. Los alias `/api/` ejecutan los mismos handlers y
  permisos, devuelven `X-API-Deprecated: true` y registran `legacy_api_used`
  sin URL, query ni cuerpo. Retirarlos solo tras 30 días sin ese evento.
- Cookie opaca `HttpOnly`, `SameSite=Strict`, `Secure` en producción;
  SHA-256 almacenado, 8 horas absolutas y 30 minutos de inactividad.
- La cuenta, rol y repartidor se consultan en PostgreSQL en cada petición.
  Logout revoca la sesión; cambiar contraseña de repartidor revoca todas sus
  sesiones. Las cookies firmadas anteriores dejan de autorizar: volver a ingresar.
- Escrituras de personal requieren origen exacto y `X-CSRF-Token`, ligado a la
  sesión mediante HMAC. El cliente guarda solo esa prueba en memoria, nunca el
  token de sesión ni en `localStorage` ni en JavaScript.
- Login por correo limita fallos en PostgreSQL, compartidos entre réplicas.
  Auditoría registra acción, resultado, cuenta y requestId, sin credenciales/PII.
- Límite de cuerpo 64 KiB, incluso con transferencia fragmentada; requestId
  validado/generado y cabeceras `nosniff`, `no-referrer`, `DENY`, `private/no-store`.
- Uvicorn no confía en proxies arbitrarios y no registra URLs con guías.
  httpx/httpcore no registran URLs de proveedores que contienen claves.
- `APP_ALLOWED_ORIGINS` es una lista JSON de orígenes exactos. Configurar el
  dominio público efectivo antes del despliegue; no permitir `*`.

### Migración y pruebas

`0010_revocable_sessions` agrega sesiones y auditoría; no borra tablas de negocio.
Se probó en PostgreSQL local (`docker compose`, puerto 5433), no en RDS.
La reversión de aplicación conserva estas tablas: no ejecutar downgrade destructivo.
El despliegue requiere confirmación para aplicar la migración a producción.

