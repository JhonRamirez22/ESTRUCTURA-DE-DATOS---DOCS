# Despliegue local preparado para AWS

La carpeta `deploy/aws/` contiene infraestructura como código; no ejecuta despliegues por
sí sola. El diseño mantiene tres componentes separados: Next.js (ECS/Fargate),
FastAPI (ECS/Fargate) y PostgreSQL (RDS). El navegador usa un único dominio:
el ALB envía `/api/*` a FastAPI y las demás rutas al frontend. La base de datos
queda en subredes privadas aisladas y no acepta conexiones desde Internet.

## Archivos

- `../../frontend/Dockerfile`: imagen de frontend Next.js con salida standalone.
- `../../backend/Dockerfile`: imagen FastAPI, Alembic y tarea de aprovisionamiento.
- `../../deploy/aws/lib/rutas-pasto-stack.ts`: VPC, RDS, secretos, tareas/servicios ECS, ALB,
  controles de red y salidas necesarias para las tareas de base de datos.
- `../../deploy/aws/scripts/run-database-task.sh`: ejecuta y espera una tarea Fargate puntual;
  sirve para inicializar el usuario de aplicación y correr Alembic.
- `../../deploy/aws/cdk.json`, `../../deploy/aws/package.json`, `../../deploy/aws/pnpm-lock.yaml`: configuración y dependencias
  fijadas para sintetizar la infraestructura localmente.

La API y la UI se construyen como imágenes independientes. El frontend no recibe
credenciales ni se conecta a PostgreSQL. FastAPI lee las credenciales como
secretos ECS/Secrets Manager y verifica el certificado TLS de RDS. Las claves
opcionales (TomTom, Groq, ORS, SMTP y Twilio) se dejan vacías intencionalmente;
se añaden después al secreto `rutas-pasto/<entorno>/integrations`, nunca a Git ni
a argumentos de build.

## Preparación local

Requisitos: Node.js 22, pnpm 10, Docker con Buildx, AWS CLI autenticado, una
cuenta/región AWS y permisos para CDK, CloudFormation, IAM, ECS, ECR, EC2, RDS,
Secrets Manager, ACM y CloudWatch Logs. La región se elige al preparar el
despliegue; no está codificada en el proyecto.

Desde la raíz del repositorio:

```bash
pnpm --dir deploy/aws install --frozen-lockfile
pnpm --dir deploy/aws typecheck
pnpm --dir deploy/aws synth
```

`synth` solo genera la plantilla local en `deploy/aws/cdk.out`; no crea ni
modifica recursos AWS. Los comandos `cdk bootstrap` y `cdk deploy` son pasos
futuros que requieren autorización explícita y no se ejecutan como parte de la
preparación local.

## Secuencia de despliegue manual, cuando se autorice

1. Revisar costo y capacidad. El entorno `dev` usa una NAT y una instancia RDS
   pequeña; `prod` usa dos NAT, RDS Multi-AZ, backups más largos y protección
