# Despliegue en AWS de Rutas Pasto

Inventario de lo que está desplegado en AWS para este proyecto, verificado el
5 de octubre de 2026 con la API de AWS (MCP) sobre la cuenta `762911134095`,
región `us-east-1`. Este documento asocia cada recurso en la nube con el código
del repositorio que lo define o lo consume.

Documentos relacionados: [arquitectura.md](../architecture/arquitectura.md),
[aws-cdk.md](./aws-cdk.md),
[../../../PLAN.md](../../../PLAN.md) y [../../../AGENTS.md](../../../AGENTS.md).

## Cuenta y región

Cuenta AWS `762911134095` (17 regiones habilitadas); todo lo relevante del
proyecto vive en `us-east-1`. La identidad usada para la inspección fue la raíz
de la cuenta. No hay presupuestos, alarmas de coste ni organizaciones
vinculados al proyecto.

## CloudFormation RutasPastoAws-dev

Pila CDK `RutasPastoAws-dev` en `UPDATE_COMPLETE`, creada el
2026-10-04T23:58:27Z y actualizada por última vez el 2026-10-05T02:57:54Z.
Existen cinco revisiones `DELETE_COMPLETE` anteriores con el mismo nombre, es
decir, la pila se destruyó y recreó varias veces antes de la versión vigente.

La pila tiene 58 recursos. Además existe la pila estándar `CDKToolkit` del
bootstrap de CDK (`cdk-hnb659fds`).

Salidas (`Outputs`) relevantes:

| Salida | Valor |
| --- | --- |
| `BackendUrl` | `ru-f8da1e81fc2f411eb7eb48aa8676c444.ecs.us-east-1.on.aws` |
| `EcsClusterName` | `rutas-pasto-dev` |
| `DatabaseEndpoint` | `rutaspastoaws-dev-postgresdatabase0a8a7373-v9wqpv10dg9b.c8d2mksimgvn.us-east-1.rds.amazonaws.com` |
| `DatabaseBootstrapTaskDefinitionArn` | `task-definition/rutas-pasto-dev-database-bootstrap:5` |
| `DatabaseMigrationTaskDefinitionArn` | `task-definition/rutas-pasto-dev-database-migrate:5` |
| `IntegrationSecretName` | `rutas-pasto/dev/integrations` |
| `DispatcherCodeSecretName` | `rutas-pasto/dev/dispatcher-access-code` |
| `BackendSecurityGroupId` | `sg-0291b789c99b6c3a8` |
| `PrivateSubnetIds` | `subnet-0f3d2a42a97466702,subnet-05157ec9324d1c505` |
| `BackendInitiallyEnabled` | `true` |

La descripción de `BackendUrl` indica que el endpoint debe configurarse como
`PYTHON_API_URL` en Vercel, lo que confirma que el frontend no corre en esta
cuenta.

## ECS Cluster rutas-pasto-dev

Clúster ECS `rutas-pasto-dev` con un único servicio:
`rutas-pasto-dev-api-public` (`ACTIVE`, `desiredCount=1`, `runningCount=1`,
`launchType=FARGATE`, `platformVersion=LATEST`, `propagateTags=SERVICE`,
`enableExecuteCommand=false`, `healthCheckGracePeriodSeconds=0`).

El servicio está declarado como `AWS::ECS::ExpressGatewayService`, es decir,
ECS Express Mode: no hay Application Load Balancer ni Service Connect en la
pila. El despliegue primario se creó el 2026-10-05T19:47:51Z con
`rolloutState=COMPLETED`.

La tarea actual (`service:rutas-pasto-dev-api-public`) está `RUNNING` desde el
2026-10-05T19:49:24Z con `cpu=512`, `memory=1024` y contenedor `Main` en estado
`RUNNING` (`healthStatus=UNKNOWN`).

## Endpoint HTTPS del backend

`https://ru-f8da1e81fc2f411eb7eb48aa8676c444.ecs.us-east-1.on.aws/api/health`
responde `200` con `{"status":"ok","checks":{"database":"ok"}}`, lo que confirma
que la API y la conexión a RDS están vivas en producción.

La raíz `/`, `/docs` y `/openapi.json` devuelven `404` de forma esperada:
`app.main` desactiva `docs_url`, `redoc_url` y `openapi_url`, y todos los
routers se registran bajo el prefijo `/api`. `GET /api/traffic` responde `422`
porque exige `lat` y `lon`.

## Task definition del backend

Task definition `rutas-pasto-dev-api:3` (Fargate, `awsvpc`, 512 CPU / 1024 MiB)
con un contenedor `Main`:

- Imagen: `762911134095.dkr.ecr.us-east-1.amazonaws.com/cdk-hnb659fds-container-assets-762911134095-us-east-1:773141ca…`
  (repositorio de *container assets* del bootstrap de CDK, no un repositorio
  propio del proyecto).
- Puerto `8000/tcp` (`name=http`), que coincide con el `EXPOSE 8000` y el
  `CMD uvicorn app.main:app --port 8000` de `backend/Dockerfile`.
- Logs a `/aws/ecs/rutas-pasto/dev/backend` con prefijo de flujo `fastapi`.

La versión activa `rutas-pasto-dev-rutas-pasto-dev-api-public:8` añade sobre la
anterior `AI_TRAFFIC_PROVIDER=groq` y
`AI_ALLOW_ROUTE_LOCATION_SHARING=true`.

## Variables de entorno y secretos del contenedor

Variables visibles: `DATABASE_HOST/PORT/NAME/USERNAME`,
`DATABASE_SSL_MODE=verify-full`, `DATABASE_SSL_ROOT_CERT=/etc/ssl/certs/rds-global-bundle.pem`,
`ROUTING_PROVIDER=osrm`, `OSRM_API_URL=https://router.project-osrm.org`,
`TOMTOM_TRAFFIC_TIMEOUT_MS=3500`, `AI_TRAFFIC_TIMEOUT_MS=5000`,
`CUSTOMER_CHAT_API_URL=https://api.groq.com/openai/v1/chat/completions`,
`CUSTOMER_CHAT_MODEL=openai/gpt-oss-20b`,
`CUSTOMER_NOTIFICATION_WORKER_ENABLED=true`,
`CUSTOMER_NOTIFICATION_POLL_SECONDS=30`, `TEXTBEE_API_URL=https://api.textbee.dev/api/v1`,
`AI_ALLOW_LOCATION_DATA_SHARING=true`, `NOTIFICATION_SMTP_PORT=587`.

Secretos inyectados desde Secrets Manager: `DATABASE_PASSWORD`,
`AUTH_SECRET`, `DISPATCHER_ACCESS_CODE` y los campos de
`rutas-pasto/dev/integrations` (`CUSTOMER_CHAT_API_KEY`, `TOMTOM_API_KEY`,
`ORS_API_KEY`, `AI_TRAFFIC_API_URL/KEY/MODEL`, `NOTIFICATION_SMTP_*`,
`TEXTBEE_API_KEY`).

Secretos creados por la pila: `rutas-pasto/dev/auth-secret`,
`rutas-pasto/dev/database-admin`, `rutas-pasto/dev/database-app-password`,
`rutas-pasto/dev/dispatcher-access-code` y `rutas-pasto/dev/integrations`.

## RDS PostgreSQL rutas_pasto

Instancia `rutaspastoaws-dev-postgresdatabase0a8a7373-v9wqpv10dg9b`:
PostgreSQL `16.15`, clase `db.t3.micro`, 20 GB, `available`, **no pública**,
sin Multi-AZ. Base de datos `rutas_pasto`, usuario de aplicación `rutas_app` y
usuario administrador `rutas_admin`.

Se conecta con `verify-full` y el bundle CA global de RDS descargado en la
imagen (`backend/Dockerfile` instala `rds-global-bundle.pem`).

## VPC y red privada

VPC `vpc-032e464082e4f4481` con cuatro subredes (dos privadas en
`subnet-0f3d2a42a97466702`/`subnet-05157ec9324d1c505`, que son las que exporta
`PrivateSubnetIds`), tabla de rutas con NAT Gateway (`nat-0117450430bc5d9a0`,
EIP `52.70.22.37`) e Internet Gateway. El RDS vive en el grupo de subredes
`rutaspastoaws-dev-postgresdatabasesubnetgroup…` y no acepta conexiones desde
Internet.

Grupos de seguridad: `sg-0291b789c99b6c3a8` (backend, exportado como
`BackendSecurityGroupId`) y `sg-0913033e412188011` con una regla de ingreso.

## Tareas puntuales de base de datos

Dos task definitions Fargate de 256/512 usadas por
`deploy/aws/scripts/run-database-task.sh`:

- `rutas-pasto-dev-database-bootstrap:5`, contenedor `BootstrapDatabaseRole`,
  comando `python -m scripts.bootstrap_database_role`, logs en
  `/aws/ecs/rutas-pasto/dev/db-bootstrap`.
- `rutas-pasto-dev-database-migrate:5`, contenedor `AlembicMigration`,
  comando `alembic upgrade head`, logs en `/aws/ecs/rutas-pasto/dev/db-migrate`.

Ambas comparten imagen con el backend y reciben `DATABASE_ADMIN_PASSWORD` /
`DATABASE_PASSWORD` desde Secrets Manager.

## Registro de imágenes ECR

No hay repositorio ECR propio de Rutas Pasto: la imagen se publica en el
repositorio de assets `cdk-hnb659fds-container-assets-762911134095-us-east-1`,
que es el mecanismo de *container asset* del bootstrap de CDK.

Los únicos repositorios de la región son ese y
`reproductor-estructuras-datos-api` (otro proyecto).

## Alarmas de CloudWatch

Tres alarmas propias del proyecto, todas en estado sano:

- `rutas-pasto-dev/rutas-pasto-dev-api-public/RollbackAlarm` → `OK`
- `TargetTracking-…/rutas-pasto-dev-api-public-AlarmHigh-…` → `OK`
- `TargetTracking-…/rutas-pasto-dev-api-public-AlarmLow-…` → `ALARM`
  (estado esperado cuando la escala está en el mínimo)

Los grupos de logs `/aws/ecs/rutas-pasto/dev/backend`, `…/db-bootstrap` y
`…/db-migrate` tienen retención de 30 días y 0 bytes almacenados, es decir,
todavía no se ha acumulado historial de ejecución.

## Frontend Next.js no desplegado en AWS

En la pila no hay ALB, ni servicio de frontend, ni salida `ApplicationUrl`, ni
ALB público alguno. No existen hosted zones en Route 53, ni distribuciones de
CloudFront para el proyecto, ni aplicaciones Amplify, ni API Gateway, ni
Lambda, ni DynamoDB, ni EKS, ni App Runner.

El frontend está previsto para Vercel: `vercel.json` define los servicios
`frontend` (Next.js) y `backend`, y `PLAN.md` y `AGENTS.md` fijan Vercel
Services como objetivo de despliegue. La API de Vercel no está conectada como
MCP, por lo que este documento solo afirma que el frontend **no** está en AWS.

## Drift entre el CDK del repositorio y la pila desplegada

La pila desplegada no coincide con `deploy/aws/lib/rutas-pasto-stack.ts` del
repositorio:

- Desplegado: `AWS::ECS::ExpressGatewayService`, task definition
  `rutas-pasto-dev-api`, contenedor `Main`, salida `BackendUrl` en
  `*.ecs.us-east-1.on.aws`.
- Repositorio: `ApplicationLoadBalancer`, `FrontendService`, salida
  `ApplicationUrl`, contenedor `FastApi`, familia
  `rutas-pasto-dev-<entorno>-backend`.

Además, el despliegue usa `TEXTBEE_*` y `AI_TRAFFIC_PROVIDER=groq` (Twilio solo
aparece en `INTEGRATION_SECRET_FIELDS` del CDK) y fija
`AI_ALLOW_LOCATION_DATA_SHARING=true` frente a `false` en el repositorio.
Ninguna de esas cadenas aparece en el historial de git
(`git log -S ExpressGateway` y `-S BackendUrl` no devuelven commits), y no hay
directorio `cdk.out` que documente la síntesis usada.

Conclusión: la infraestructura vigente se sintetizó desde una versión del stack
que no está en este repositorio.

## Otros proyectos en la misma cuenta

No forman parte de este proyecto pero comparten cuenta y región:

- ECS `default/reproductor-estructuras-datos-api` (1 tarea Fargate RUNNING,
  creado el 2026-10-02) con ECR propio y alarmas de autoescalado.
- Aurora PostgreSQL serverless `reproductor-estructuras-datos-prod-instance-1`
  (17.9) y secreto `/reproductor/prod/cookie-secret`.
- EC2 `ritech-web` (t3.small, perfil `ritech-ec2-deploy`), secreto
  `ritech/production` y CloudFront `d3preziqjebh2s.cloudfront.net`
  («RiTech Export website production origin»).

## Verificación de salud

| Comprobación | Resultado |
| --- | --- |
| `GET /api/health` | `200` — `status=ok`, `database=ok` |
| Tarea Fargate del servicio | `RUNNING`, despliegue `COMPLETED` |
| Instancia RDS | `available` |
| RollbackAlarm | `OK` |
| Frontend en AWS | ausente por diseño (Vercel) |
| Historial de logs | vacío (0 bytes) |
