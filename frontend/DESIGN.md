---
name: Rutas Pasto
description: Operación oscura con referencia Linear y controles React entregados.
colors:
  background: "#08090b"
  foreground: "#f0f1f3"
  surface: "#121316"
  surface-muted: "#191b20"
  line: "#2a2c34"
  primary: "#c6c4f8"
  primary-foreground: "#161421"
  accent-strong: "#e2e0ff"
  secondary: "#24262d"
  ink-soft: "#a4a7b1"
  input: "#343640"
  ring: "#b9b6f5"
  destructive: "#efa4b1"
typography:
  display:
    fontFamily: '"Public Sans", ui-sans-serif, sans-serif'
    fontSize: "clamp(38px, 4.8vw, 64px)"
    fontWeight: 500
    lineHeight: 1.08
    letterSpacing: "-0.04em"
  headline:
    fontSize: "30px"
    fontWeight: 500
    lineHeight: 1.2
    letterSpacing: "-0.035em"
  title:
    fontSize: "24px"
    fontWeight: 500
    letterSpacing: "-0.03em"
  body:
    fontSize: "14px"
    lineHeight: 1.85
  label:
    fontSize: "13px"
    fontWeight: 500
    lineHeight: 1.3
rounded:
  control: "8px"
  panel: "14px"
  login: "16px"
  dock: "18px"
spacing:
  sm: "8px"
  md: "12px"
  lg: "24px"
  section: "32px"
components:
  button-primary:
    backgroundColor: "{colors.primary}"
    textColor: "{colors.primary-foreground}"
    typography: "{typography.label}"
    rounded: "{rounded.control}"
    padding: "10px 16px"
  button-secondary:
    backgroundColor: "{colors.secondary}"
    rounded: "{rounded.control}"
    padding: "10px 16px"
  input:
    textColor: "{colors.foreground}"
    rounded: "{rounded.control}"
    padding: "12px"
  navigation:
    padding: "9px 14px"
  panel:
    backgroundColor: "{colors.surface}"
    rounded: "{rounded.panel}"
  dock:
    rounded: "{rounded.dock}"
    padding: "8px 12px"
---

# Design System: Rutas Pasto

## Overview

**Creative North Star: "Operación oscura · referencia Linear"**

Grafito y lavanda; mapa y paradas conducen el trabajo. Reemplazo autorizado del sistema anterior y atlas rechazado. Fuentes: `globals.css`, `ui/{button,dock,interactive-login}.tsx`, `AppShell.tsx`; contexto: PRODUCT y docs de dirección/frontend.

**Key Characteristics:** navegación horizontal, bordes discretos, superficies tonales, dock por rol. Sin comp raster aprobado. Capturas: `.impeccable/review/linear/`, no inspeccionadas aquí.

## Colors

### Primary

Lavanda para acción y foco; accent-strong aclara hover. Rosa para errores; familias semánticas operativas remapeadas en @theme inline.

### Neutral

Background sostiene la página; surface agrupa contenido; line separa; ink-soft reduce jerarquía.

**The Mapa protegido Rule.** Conservar mapa oscuro y rutas verdes; lavanda pertenece a controles.

## Typography

Public Sans autoalojada en `/fonts/public-sans-variable.woff2`; mono para identificadores. Roles extraídos de portada, títulos y botones.

Portada: 17ch, móvil 38px. Títulos: móvil 26px. Relato: 52px/1.1, móvil 32px. Explicaciones: 38–65ch. Metadatos: 11–12px.

## Layout

Encabezado sticky: máximo 1600px, mínimo 72px, padding 12px 32px. Bajo 768px: marca/cuenta arriba, navegación en segunda fila; mínimo 64px. Dashboard: 28px 32px, móvil 24px 18px.

Portada: 1280px, 1.2fr/0.8fr. Acceso: 1100px, 1.1fr/0.9fr. Gap 96px; móvil apila.

Escritorio: dock fijo, panel mínimo 64px; shell reserva 112px; bottom max(16px, env(safe-area-inset-bottom)). Móvil: dock estático debajo del contenido, padding 24px 16px 0; shell reserva max(24px, env(safe-area-inset-bottom)). No trasladar el flotante al formulario móvil.

## Elevation & Depth

Paneles/login sin sombra; profundidad tonal. Sombra dock en sidecar. Controles: 160ms; flecha: 3px/200ms. Dock: 44–68px, distancia 150px; estático táctil/movimiento reducido.

## Shapes

Radios por función; navegación 7px, item dock 11px. Mapa existente, SVG/Lucide y fuente local; sin raster. Manifest/icon grafito/lavanda confirmado por usuario.

## Components

Button entregado: default, secondary, outline, destructive, ghost, link; tamaños default/sm/lg/icon. Mínimo 44px; lg 48px. Disabled: opacidad 0.5. Foco general: ring 2px, separación 4px.

Campos: mínimo 46px; foco separado 2px. Login: correo/contraseña, visibilidad, carga, alerta y bloqueo duplicado; rol en servidor.

Navegación/dock: enlaces por rol, aria-current y etiqueta al hover/foco. Seis snippets CSS independientes; dock estático, sin runtime React ni datos ficticios.

## Do's and Don'ts

- **Do** reutilizar tokens, controles entregados y destinos autorizados.
- **Do** mantener estados textuales y nombres accesibles.
- **Don't** introducir raster ajeno, pagos o métricas inventadas.
- **Don't** tratar atlas o capturas E2E como aprobación visual.

No canonizado ni reparado: transición de color del enlace permanece bajo movimiento reducido por sobrescritura CSS; ancho 430px no fijado.

Resultados de ejecución: 67 Vitest, 315 pytest, type/lint ambos y build Node 22 pasan. Suite E2E final: 14 pasan directamente y uno pasa al reintentar tras cierre de Chrome. Subset final: 7/7 sin retry. Detector anterior contrastó el sistema sustituido: avisos de diseño revisados, sin supresiones. El documentador no ejecutó pruebas adicionales.

