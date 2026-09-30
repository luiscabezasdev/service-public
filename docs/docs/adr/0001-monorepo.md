# ADR-0001: Un solo repositorio (monorepo)

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto
El sistema tiene tres piezas (backend, web, Android) que comparten un
contrato de API. Lo desarrolla una sola persona.

## Decisión
Las tres piezas viven en un solo repositorio, en `backend/`, `web/` y
`android/`, con documentación común en `docs/`.

## Alternativas consideradas
- **Tres repositorios separados:** común en empresas con equipos por
  servicio, pero obliga a coordinar versiones entre repos y fragmenta la
  historia del proyecto. Sin beneficio para un equipo de una persona.

## Consecuencias
- **Positivas:** un cambio de API y la actualización de sus clientes van
  en el mismo PR; un solo ROADMAP, CHANGELOG e historial.
- **Negativas:** el CI debe distinguir qué pieza cambió para no ejecutar
  todo siempre.
- **Cuándo revisarla:** si equipos distintos mantienen piezas distintas o
  necesitan permisos separados.
