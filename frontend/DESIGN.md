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
