# Auditoría de seguridad de la API (13 de noviembre de 2026)

Método: un arnés de caja negra contra la API real (uvicorn y PostgreSQL 16 aislado) con tres identidades (despachador, repartidor A y repartidor B) y datos de dos repartidores. Se probó cada operación con cada identidad, ataques entre repartidores (IDOR), CSRF, sesiones falsificadas, límites de tasa, estados del negocio y 6 328 peticiones hostiles generadas. Las evidencias son de ese corte; no se probó el sitio publicado.

## Lo que ya resistía

| Área | Resultado |
|---|---|
| Autorización | 54 operaciones: ninguna accesible sin sesión. Un repartidor recibe 403 en todo lo del despachador |
| IDOR | El repartidor A no lee, cambia, cierra ni mueve la ruta, el pedido ni la ubicación del repartidor B (404 o 403) |
| Campos privilegiados | El alta de pedidos ignora `status`, `routeId`, `trackingToken` e `id` enviados por el cliente |
| CSRF y origen | Sin token, con token de otra sesión, origen ajeno, `null`, sufijo malicioso y `Sec-Fetch-Site: cross-site`: 403 |
| Sesiones | Cookie y Bearer falsificados o vacíos: 401. `X-Session-Role` inválido: 400 |
| Estados | Una ruta cancelada no resucita; sus pedidos vuelven a pendientes. El repartidor no cambia direcciones ni reabre entregas |
| Ubicación | Fuera de Pasto, con fecha futura o antigua: 400. Hay límite de frecuencia (429) |
| Entradas hostiles | 6 328 peticiones (SQL, rutas, plantillas, tipos y tamaños extremos) con tres identidades: ningún 5xx |
| Exposición | Seguimiento por guía UUID4 sin teléfono, nombre ni id del repartidor; `/docs` y `/openapi.json` cerrados; alias `/api` legado apagado; cuerpo de 2 MB: 413 |
| Dependencias de ejecución | `pip-audit` y `pnpm audit --prod` sin hallazgos (tras actualizar `certifi`) |

## Hallazgos corregidos

| # | Hallazgo | Gravedad | Prueba roja | Corrección |
|---|---|---|---|---|
| H1 | El límite de login se aplicaba por correo: quien conociera el correo del despacho lo mantenía fuera (30 intentos y 429 de 60 s para el despachador legítimo) | Alta (disponibilidad) | `test_h1_un_atacante_no_bloquea_el_login_del_despachador` | Límite por correo y cliente (10 por minuto) más un tope amplio por correo (120 cada 10 minutos) contra la fuerza bruta repartida |
| H2 | `X-Forwarded-For` se tomaba por su primer valor, que el cliente inventa: 80 solicitudes con valores distintos creaban 20 y bloqueaban el cupo global para los clientes reales | Media | `test_h2_x_forwarded_for_falsificado_no_evade_el_limite_por_cliente` | La identidad del cliente usa el último salto, que añade la infraestructura (`app/services/client_identity.py`) |
| H3 | El backend aceptaba llamadas directas aunque solo debe hablar con Vercel | Media | `test_h3_con_candado_de_origen_solo_pasa_el_proxy_autorizado` | Candado de origen: con `ORIGIN_AUTH_SECRET` solo pasan las peticiones con `X-Origin-Auth` correcto (comparación en tiempo constante). Con el candado, el primer valor de `X-Forwarded-For` es fiable porque Vercel lo escribe con la IP real |
| H4 | `certifi` 2026.7.22 con aviso PYSEC-2026-4320 | Media | `pip-audit` | `uv.lock` actualizado a 2026.10.23 |
| H5 | Happy DOM con dos avisos altos (SSRF) en las pruebas | Baja (solo desarrollo) | `pnpm audit` | Actualizado a 21.0.4 |

La matriz de seguridad ya no marca como públicas las operaciones de inteligencia (agrupaciones y explicación), que exigen sesión de despacho.

## Cómo activar el candado de origen

1. Generar un secreto de 32 bytes o más. Ya existe en Vercel (`ORIGIN_AUTH_SECRET`, sensible, producción) y en la Container App de Azure (secreto `origin-auth-secret` y variable `ORIGIN_AUTH_SECRET`).
2. El middleware del frontend (`src/middleware.ts`) reenvía `/api/*` al backend con la cabecera `X-Origin-Auth`. Sin el secreto (desarrollo local), responde el rewrite de `next.config.ts`.
3. En AWS hay que añadir el mismo valor a la task definition antes de volver a usarlo. Si el backend no tiene el secreto, no hay candado y todo sigue funcionando.
4. Orden seguro: primero el secreto en Vercel y redeploy, después en el backend. Si solo el backend lo tiene, todo recibe 403.
5. `/api/v1/health` queda exento para el balanceador.

## Riesgos aceptados y pendientes

| Pendiente | Detalle |
|---|---|
| Dependencias de desarrollo | Vite 7.3.6 con avisos del servidor de desarrollo y `launch-editor`; el parche 7.3.7 todavía no entra por la política de antigüedad de pnpm. No llegan a producción |
| Limitadores en memoria | Tráfico, geocodificación y chat usan contadores por proceso. Con una sola tarea es correcto; con réplicas habría que moverlos a PostgreSQL (`retry_after_shared`) |
| PostgreSQL de Azure | La regla `0.0.0.0` («Servicios de Azure») permite conectar desde cualquier suscripción de Azure; solo protegen la contraseña y TLS verificado. Para producción: red privada o restringir a las IP de salida de la app |
| Registro de contenedores de Azure | Usuario administrador activo (el entorno express no admite identidad administrada) |
| Credenciales | Las claves de Groq, TomTom y TextBee no se rotaron; AWS se usa con el usuario raíz; la RDS de AWS no tiene protección contra borrado |
| Seguimiento temporal | Sigue pendiente conectar las sesiones temporales de seguimiento (el acceso es por guía UUID) |
| Bloqueo del despacho | Un atacante distribuido (muchos clientes) sigue pudiendo llegar al tope por correo; requiere CAPTCHA o alertas para cerrarlo del todo |

## Pruebas nuevas

- `tests/test_abuse_resistance.py` (7 pruebas): bloqueo por cliente, tope por correo, `X-Forwarded-For` falsificado, candado de origen y confianza en el primer salto con candado.
- `tests/unit/originProxy.test.ts` (5 pruebas): el proxy añade el secreto, reemplaza el del cliente, no lo expone y no afecta a las páginas.
- Las pruebas e2e se actualizaron al diseño de hoja y ya no afirman la interfaz de avisos al cliente, que se retiró.
