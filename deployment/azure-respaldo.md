# Backend en Azure y AWS en pausa

**Estado al 20 de noviembre de 2026: Azure está ENCENDIDO** (imagen `api-3d94372`, esquema Alembic `0013_order_priority`, `DEMO_MODE=true` y `AI_TEXT_PHRASING_ENABLED=true`, `PYTHON_API_URL` de Vercel apuntando a `ca-rutas-pasto-api.mangobush-7ec3063e.canadacentral.azurecontainerapps.io`). El servicio de AWS sigue en 0 tareas. Las secciones «Cómo está apagado» y «Apagar Azure» describen cómo volver al estado apagado del 31 de octubre.

Migrar desde el contenedor (necesita un pty, por eso `script`):

```bash
script -q /dev/null az containerapp exec -g rg-rutas-pasto -n ca-rutas-pasto-api --command 'sh -c "cd /app && alembic upgrade head"'
```

## Recursos en Azure

Suscripción «Azure for Students». La política solo permite las regiones `mexicocentral`, `chilecentral`, `brazilsouth`, `canadacentral` y `eastus`. PostgreSQL Flexible Server no está disponible en `eastus` para esta suscripción; por eso todo está en `canadacentral`.

| Recurso | Nombre | Notas |
|---|---|---|
| Grupo de recursos | `rg-rutas-pasto` | Etiqueta `app=rutas-pasto` |
| PostgreSQL Flexible Server 16 | `rutaspgb96abe` | Burstable B1ms, 32 GB, base `rutas_pasto`, usuario `rutasadmin`, acceso público solo desde servicios de Azure |
| Azure Container Registry | `rutaspastofb2dfd` | Básico, usuario administrador activo (el entorno express no admite identidad administrada para extraer imágenes) |
| Entorno de Container Apps | `cae-rutas-pasto` | Entorno express |
| Container App | `ca-rutas-pasto-api` | Ejecuta `api-55f5a13` (linux/amd64), puerto 8000, 0,5 vCPU y 1 GiB. En ACR ya está `api-b70063d` (optimización exacta, límites por cliente y candado de origen); se activa con `az containerapp update --image` al encender. La app ya tiene el secreto y la variable `ORIGIN_AUTH_SECRET`, el mismo valor que Vercel |

La contraseña de la base y los secretos viven como secretos de la Container App. No están en Git.

## Qué se migró y qué no

- **Esquema:** Alembic aplicado hasta `0012_traffic_observations` en una base nueva.
- **Datos:** ninguno. La base de Azure está vacía; las cuentas, pedidos y ubicaciones siguen solo en RDS. Antes de usar Azure hay que crear la cuenta de despacho (`scripts/create_dispatcher_account.py`) y, si se quiere una demo, sembrar datos (`scripts/seed_demo.py`, `scripts/seed_simulated_day.py`).
- **Configuración:** se copió la de la task definition de ECS. `DATABASE_*` apunta a Azure con `DATABASE_SSL_MODE=verify-full` y los certificados del sistema. `AUTH_SECRET` es nuevo.
- **Secretos vacíos en AWS** (no se crearon en Azure): `ORS_API_KEY`, `NOTIFICATION_SMTP_HOST`, `NOTIFICATION_SMTP_USERNAME`, `NOTIFICATION_SMTP_PASSWORD`, `NOTIFICATION_FROM_EMAIL`.
- Las claves de Groq, TomTom y TextBee se copiaron tal cual (siguen sin rotar).

## Cómo está apagado

| Pieza | Apagado | Límite |
|---|---|---|
| Container App | Detenida con la acción `stop` de ARM (estado `Stopped`, 0 réplicas) y con el ingreso deshabilitado | El entorno express no escala a cero ni admite desactivar revisiones; `az containerapp stop` no existe en la CLI, se usa `az rest` |
| PostgreSQL | `az postgres flexible-server stop` | **Azure lo reinicia solo a los 7 días** (ya ocurrió el 9 de noviembre y se volvió a detener). Hay que volver a detenerlo o se cobra el cómputo |
| ACR | Sigue activo | Cuesta unos centavos al día; contiene la imagen |
| AWS ECS | Autoescalado en 0/0 y `desiredCount 0` | La RDS de AWS **sigue encendida** y con todos los datos |

## Encender Azure

```bash
az postgres flexible-server start -g rg-rutas-pasto -n rutaspgb96abe
ID=$(az containerapp show -g rg-rutas-pasto -n ca-rutas-pasto-api --query id -o tsv)
az rest --method post --url "https://management.azure.com${ID}/start?api-version=2025-01-01"
az containerapp ingress enable -g rg-rutas-pasto -n ca-rutas-pasto-api --type external --target-port 8000
az containerapp update -g rg-rutas-pasto -n ca-rutas-pasto-api --min-replicas 1 --max-replicas 1
az containerapp show -g rg-rutas-pasto -n ca-rutas-pasto-api --query properties.configuration.ingress.fqdn -o tsv
```

Después, comprobar `https://<fqdn>/api/v1/health` y cambiar `PYTHON_API_URL` en Vercel al nuevo dominio. El valor se lee al compilar, así que hay que volver a desplegar el frontend. `APP_ALLOWED_ORIGINS` ya contiene el dominio de Vercel.

## Apagar Azure

```bash
az containerapp ingress disable -g rg-rutas-pasto -n ca-rutas-pasto-api
ID=$(az containerapp show -g rg-rutas-pasto -n ca-rutas-pasto-api --query id -o tsv)
az rest --method post --url "https://management.azure.com${ID}/stop?api-version=2025-01-01"
az postgres flexible-server stop -g rg-rutas-pasto -n rutaspgb96abe
```

## Volver a AWS

```bash
aws application-autoscaling register-scalable-target --service-namespace ecs --scalable-dimension ecs:service:DesiredCount --resource-id service/rutas-pasto-dev/rutas-pasto-dev-api-public --min-capacity 1 --max-capacity 1 --region us-east-1
aws ecs update-service --cluster rutas-pasto-dev --service rutas-pasto-dev-api-public --desired-count 1 --region us-east-1
```

La tarea vuelve con la task definition `:35` y los datos de RDS. `PYTHON_API_URL` en Vercel apunta a AWS, así que no hay que tocarlo si no se cambió.

## Actualizar la imagen en Azure

Construir en local para `linux/amd64` (ACR Tasks no está permitido en esta suscripción), subir a ACR y actualizar la app:

```bash
export PATH="/Applications/Docker.app/Contents/Resources/bin:$PATH"
az acr login -n rutaspastofb2dfd
docker buildx build --platform linux/amd64 -t rutaspastofb2dfd.azurecr.io/rutas-pasto-api:api-<sha> --push .
az containerapp update -g rg-rutas-pasto -n ca-rutas-pasto-api --image rutaspastofb2dfd.azurecr.io/rutas-pasto-api:api-<sha>
```

## Aviso al modificar la app mientras está detenida

Cualquier `az containerapp update` o `secret set` sobre una app detenida crea una revisión nueva y **arranca una réplica** aunque el estado siga diciendo `Stopped`. Después de cambiar algo, deja `--min-replicas 0` y repite `start` y luego `stop` con la acción de ARM hasta que `az containerapp replica list` devuelva 0. La acción `stop` responde 400 si ya está detenida; es normal.

## Interruptores de entorno añadidos el 20 de noviembre de 2026

| Variable | Efecto | Estado en Azure |
|---|---|---|
| `DEMO_MODE` | Muestra «Renovar GPS demo» y habilita `POST /api/v1/demo/renovar-gps` (renueva solo a los repartidores «Demo ·») | `true` (entorno de demostración) |
| `AI_TEXT_PHRASING_ENABLED` | Groq redacta avisos con cifras y alias, sin ubicaciones, nombres ni direcciones (usa `CUSTOMER_CHAT_*`) | `true` |
| `TIME_WINDOWS_ENABLED`, `AI_SUGGESTIONS_ENABLED`, `JOINT_ASSIGNMENT_ENABLED` | Ventanas horarias, avisos en contexto y reparto conjunto | valor por defecto (`true`) |
| `TRAFFIC_HISTORY_ENABLED` | Guarda y usa el historial de tráfico para el ETA histórico | apagado |

Con una operación real, apagar `DEMO_MODE` (`az containerapp update ... --set-env-vars DEMO_MODE=false`).
