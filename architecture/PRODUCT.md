# Rutas Pasto

<!-- impeccable:product-schema 1 -->

## Platform

web

## Users

Despachadores que registran pedidos, asignan repartidores y supervisan rutas.
Repartidores que consultan sus entregas y comparten GPS desde el teléfono.
Clientes que consultan exclusivamente su envío mediante una guía privada.

## Product Purpose

Organizar entregas dentro de Pasto mediante tiempos de calles, optimización
del orden de paradas y seguimiento del avance.

## Operating Context

Despacho desde escritorio; reparto y seguimiento desde móvil. La interfaz
operativa usa sesiones privadas con correo y contraseña. Los estados de GPS,
pedidos y rutas cambian durante la jornada.

## Capabilities and Constraints

Next.js 15 y TypeScript presentan datos de FastAPI y PostgreSQL. MapLibre
representa geometría vial OSRM/ORS. Nearest Neighbor y 2-opt ordenan las
paradas; TomTom aporta observaciones parciales de tráfico cuando está disponible.
La IA es opcional. No inventar tráfico, ETA, pedidos, credenciales ni métricas.
Conservar permisos, contratos HTTP, controles de GPS y funciones existentes.

## Brand Commitments

Nombre Rutas Pasto; español directo. Mapa oscuro y rutas verdes, confirmados
por el usuario. El usuario solicita reemplazar la UI genérica con una interfaz
elegante y controles cuidados, delegando las decisiones dentro del PLAN.

## Evidence on Hand

PLAN.md, README.md, código y pruebas E2E de despacho, pedidos, seguimiento,
reparto y autenticación. Datos de pruebas etiquetados como sintéticos.

## Product Principles

- El mapa y la ruta activa conducen la operación.
- Cada rol recibe únicamente los datos y acciones autorizados.
- La estimación comunica su alcance real.
- El diseño facilita tareas frecuentes en escritorio y móvil.
