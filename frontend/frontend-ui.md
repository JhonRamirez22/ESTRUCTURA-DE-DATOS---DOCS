# Frontend y controles

- Referencia fijada por el usuario: https://linear.app/. Base oscura, jerarquía limpia, controles discretos. Conservar MapLibre oscuro y geometría verde.
- Next.js 15, TypeScript y Tailwind 4 ya estaban configurados. `components.json` registra una estructura compatible con shadcn; no se vuelve a inicializar el proyecto.
- Código de interfaz: `frontend/src`; componentes reutilizables: `frontend/src/components/ui`; estilos: `frontend/src/app/globals.css`. `@/components/ui` resuelve ahí mediante el alias de `frontend/tsconfig.json`.
- Dock adaptado del ejemplo aportado por el usuario: enlaces nativos por rol, etiqueta con foco, magnificación desde clientX, sin tabuladores duplicados ni role=button sin acción. Estático en táctil y movimiento reducido. En escritorio flota con espacio reservado al final; en móvil permanece en flujo para no cubrir campos ni acciones.
- El checkout se adaptó a `interactive-login.tsx`, como autorizó el usuario: correo/contraseña, mostrar/ocultar, carga, error y bloqueo de envíos duplicados. No hay carrito, productos, pagos ni roles seleccionables.
- Se conservó el contrato `/api/auth/login`; permisos y sesiones siguen en FastAPI. Sin cambios de base de datos.
- Dependencias añadidas: framer-motion, lucide-react, Radix Slot, class-variance-authority, clsx y tailwind-merge. NumberFlow no se añadió porque no existe un contador monetario en el login. Sin imágenes de stock ajenas al producto.
- En Vitest Motion salta animaciones por las limitaciones de WAAPI en Happy DOM; Playwright verifica magnificación, foco, destinos y movimiento reducido con navegador real.
