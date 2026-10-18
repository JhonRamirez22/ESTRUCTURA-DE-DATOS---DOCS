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

Los tests de negocio aíslan autenticación mediante una dependencia de prueba.
`test_auth_api.py`, `test_access_sessions.py` y `test_security_postgres.py` verifican
autenticación real y no emplean esa sustitución.

Verificación de este bloque: 332 tests backend, 73 frontend, 6 integraciones
PostgreSQL; Ruff, mypy, TypeScript, ESLint y build pasan. Playwright: 14/15;
queda reproducido el fallo preexistente de estabilidad de la animación del dock
(`dock.spec.ts`); se corrige en el bloque visual, sin forzar clics ni omitir tests.

## Direcciones por texto

`POST /api/v1/direcciones/resolver` requiere despacho, CSRF y límite compartido
de 30 búsquedas/minuto por cuenta. El formulario espera 500 ms, cancela búsquedas
anteriores y descarta respuestas tardías. La alternativa manual permanece disponible.

Primero normaliza abreviaturas y consulta TomTom restringido a Colombia/Pasto.
Si no encuentra candidatos, pide interpretación a la IA autorizada y vuelve a
geocodificar. La IA no aporta coordenadas ni puede cambiar números u orientación.
Los resultados de otros municipios/países o fuera del área se descartan.
Una numeración incompleta nunca se presenta como dirección exacta; el usuario
confirma explícitamente cualquier candidato antes de registrar el pedido.

Para Groq: `AI_TRAFFIC_PROVIDER=groq`, credenciales `CUSTOMER_CHAT_*` y
`AI_ALLOW_LOCATION_DATA_SHARING=true` tras autorizar el envío. Sin autorización,
TomTom y normalización local siguen funcionando sin enviar texto a IA.

Verificado con proveedores simulados: 344 tests backend, 76 frontend y un E2E
específico de búsqueda sin pin/confirmación. Build, tipos y lint pasan.
La disponibilidad/cuota real de proveedores se verificará con credenciales rotadas
antes del lanzamiento; estos tests no consumen APIs externas.

## Bloques restantes del plan aprobado

- Autorización temporal de seguimiento por pedido, límites compartidos restantes,
  paginación y CSP del frontend; revisión de dependencias y rotación de secretos.
- Teselas transparentes de tráfico por proxy autenticado; caché/estado de cobertura,
  cámara independiente del GPS, agrupación y recuperación de capas.
- Interfaz marfil/beige/café, mapa claro y rutas verdes. Propuesta visual local
  presentada; todavía no se sustituyó el diseño de la aplicación.
- Pruebas finales completas y despliegue canario AWS/Vercel, previa confirmación
  de migración y credenciales nuevas. La sesión AWS actual corresponde a root;
  no se utilizará para lanzar cambios antes de preparar un rol limitado.
