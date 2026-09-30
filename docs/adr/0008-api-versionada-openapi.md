# ADR-0008: API versionada con contrato OpenAPI

- **Estado:** Aceptado
- **Fecha:** 2026-09-29

## Contexto
Dos clientes (web y Android) consumen la misma API. Una app Android
instalada puede quedar desactualizada y seguir llamando a la API vieja.

## Decisión
- Todas las rutas bajo `/api/v1`.
- El contrato se describe en OpenAPI (`docs/api/openapi.yaml`) en la
  Fase 0, antes de programar, y se actualiza en el mismo PR que cambie
  la API.

## Alternativas consideradas
- **Sin versión en la URL:** cualquier cambio incompatible rompería las
  apps instaladas.
- **Documentar la API solo en Markdown:** se desactualiza con facilidad y
  no permite generar documentación ni validar el contrato con
  herramientas.

## Consecuencias
- **Positivas:** una sola fuente de verdad para las tres piezas;
  documentación navegable; los cambios incompatibles se controlan.
- **Negativas:** mantener el archivo OpenAPI al día exige disciplina (se
  incluye en la plantilla de PR).
- **Cuándo revisarla:** si se necesita un cambio incompatible, se crea
  `/api/v2` en un ADR nuevo.
