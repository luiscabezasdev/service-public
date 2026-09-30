# ADR-0004: Backend como monolito modular

- **Estado:** Aceptado
- **Fecha:** 2026-09-29

## Contexto
El backend tiene varias áreas de negocio (autenticación, apartamentos,
facturas, cobros, reportes). Debe ser fácil de mantener por una persona
y poder crecer sin reescribirse.

## Decisión
Un solo servicio desplegable (monolito), organizado internamente por
módulos de negocio (`src/modules/<modulo>`), cada uno con capas
rutas → controlador → servicio → repositorio. Los módulos se comunican
solo a través de sus servicios. Detalle en
[arquitectura.md](../arquitectura.md), sección 4.

## Alternativas consideradas
- **Microservicios:** despliegues, redes y datos distribuidos multiplican
  la complejidad y los puntos de falla, sin beneficio para un equipo de
  una persona.
- **Carpetas por tipo** (`controllers/`, `services/`…): funciona en
  proyectos pequeños, pero un cambio en una funcionalidad obliga a tocar
  muchas carpetas y los límites entre áreas se pierden.

## Consecuencias
- **Positivas:** un solo despliegue y una sola base de datos; límites
  claros entre módulos; un módulo se puede extraer más adelante si hace
  falta.
- **Negativas:** exige disciplina para no saltarse los límites (un
  módulo leyendo tablas de otro).
- **Cuándo revisarla:** si una parte necesita escalar o desplegarse de
  forma independiente del resto.
