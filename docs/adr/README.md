# Registro de Decisiones de Arquitectura (ADR)

Un ADR (*Architecture Decision Record*) documenta **una** decisión técnica
importante: el contexto, lo que se decidió, las alternativas descartadas
y sus consecuencias. Más información en https://adr.github.io/.

**Por qué se usan:** dentro de un año nadie recuerda por qué se eligió
Supabase o por qué el dinero se guarda con decimales exactos. Sin el ADR, esa
decisión se vuelve a discutir o, peor, se cambia sin entender por qué se
tomó.

## Reglas

- Un ADR por decisión, numerado en orden (`0009-...`).
- Se escribe **cuando se toma la decisión**, en el mismo PR.
- Un ADR aceptado **no se edita**. Si la decisión cambia, se crea un ADR
  nuevo que lo reemplaza, y el anterior se marca como "Reemplazado por
  ADR-00XX". Así queda la historia completa.
- Plantilla: [plantilla.md](./plantilla.md).

## Índice

| ADR | Decisión | Estado |
|---|---|---|
| [0001](./0001-monorepo.md) | Un solo repositorio (monorepo) para backend, web y Android | Aceptado |
| [0002](./0002-stack-tecnologico.md) | Stack: Node/Express/TypeScript, React, Kotlin/Compose | Aceptado |
| [0003](./0003-postgresql-en-supabase.md) | PostgreSQL administrado en Supabase | Aceptado |
| [0004](./0004-monolito-modular.md) | Backend como monolito modular organizado por módulos de negocio | Aceptado |
| [0005](./0005-cobro-inmutable.md) | Cobro mensual inmutable; correcciones con ajustes | Aceptado |
| [0006](./0006-dinero-en-enteros.md) | Dinero en pesos enteros | Reemplazado por 0009 |
| [0007](./0007-reparto-strategy-dominio-puro.md) | Motor de reparto con patrón Strategy en dominio puro | Aceptado |
| [0008](./0008-api-versionada-openapi.md) | API versionada (`/api/v1`) con contrato OpenAPI | Aceptado |
| [0009](./0009-dinero-decimal-exacto.md) | Dinero con decimales exactos (`NUMERIC` + `decimal.js`) y cobro completo en el mismo mes | Aceptado |
