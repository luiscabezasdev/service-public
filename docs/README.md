# Gestión de Arrendamiento

Sistema full-stack de gestión de arrendamiento para un edificio de 6
apartamentos: apartamentos, inquilinos, mantenimientos, y el reparto de servicios públicos (agua, gas, energía, aseo, internet/parabólica) entre inquilinos, con reportes mensuales y generación del recibo para compartir.

Proyecto de aprendizaje de Luis Cabezas — construido en pair programming con Claude, con foco en entender cada concepto nuevo, no solo tener algo funcionando.

## Estructura

```
backend/    Node.js + Express + TypeScript + PostgreSQL (Prisma) + JWT
web/        React + TypeScript
android/    Kotlin + Jetpack Compose
docs/       Constitución del proyecto, reglas de negocio, plan detallado
```

Los tres clientes (web, Android, y cualquiera futuro) consumen la misma
API expuesta por `backend/`.

## Documentación

- [ROADMAP.md](./ROADMAP.md) — plan de trabajo por fases, con checkboxes
  de avance
- [docs/plan-de-trabajo.md](./docs/plan-de-trabajo.md) — el plan
  detallado, con el porqué del orden de cada fase
- [docs/plan-de-aprendizaje.md](./docs/plan-de-aprendizaje.md) — qué
  aprender en cada fase, documentación oficial y preguntas de criterio
- [docs/arquitectura.md](./docs/arquitectura.md) — estructura del
  sistema, integridad de datos, escalabilidad y definición de "terminado"
- [docs/adr/](./docs/adr/README.md) — registro de decisiones de
  arquitectura (ADR)
- [docs/reglas-de-negocio.md](./docs/reglas-de-negocio.md) — reglas de
  prorrateo de cada servicio
- [docs/plan-frontend-ui.md](./docs/plan-frontend-ui.md) — pantallas,
  navegación y sistema de diseño (web y Android)
- [docs/seguridad-y-buenas-practicas.md](./docs/seguridad-y-buenas-practicas.md)
  — OWASP API Security Top 10 y OWASP Top 10:2025 aplicados a este stack
- [docs/hosting-base-datos.md](./docs/hosting-base-datos.md) — dónde vive
  la base de datos (Supabase) y cómo se asegura la conexión
- [docs/control-de-versiones.md](./docs/control-de-versiones.md) — ramas,
  Conventional Commits, Pull Requests y versiones
- [CHANGELOG.md](./CHANGELOG.md) — historial de cambios por versión
- [docs/constitution.md](./docs/constitution.md) — reglas del proyecto
  (cómo se trabaja el código, comentarios obligatorios, etc.)

## Estado

🚧 En construcción — ver [ROADMAP.md](./ROADMAP.md)
