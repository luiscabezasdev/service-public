# ADR-0002: Stack tecnológico

- **Estado:** Aceptado
- **Fecha:** 2026-09-27

## Contexto
Se necesita acceso del administrador desde web y Android, desde
cualquier lugar. El proyecto es de aprendizaje y el autor ya trabaja con
TypeScript y tiene experiencia previa en Kotlin/Compose.

## Decisión
- Backend: Node.js + Express + TypeScript, Prisma como ORM.
- Web: React + TypeScript + Material UI.
- Android: Kotlin + Jetpack Compose (Material 3), Retrofit, DataStore.

## Alternativas consideradas
- **Kotlin + Ktor en el backend:** mismo lenguaje que Android, pero menos
  material de aprendizaje y no aprovecha el TypeScript que el autor ya usa.
- **Firebase (sin backend propio):** más rápido, pero se pierde el
  aprendizaje de construir una API y se depende de un proveedor para la
  lógica de negocio.
- **HTML/CSS/JS sin framework en la web:** difícil de mantener con varias
  pantallas y formularios.

## Consecuencias
- **Positivas:** TypeScript en backend y web; Material Design en web y
  Android da una apariencia coherente con un solo diseño.
- **Negativas:** dos lenguajes (TypeScript y Kotlin) y dos clientes que
  mantener.
- **Cuándo revisarla:** si el mantenimiento de dos clientes no es
  sostenible, evaluar dejar solo la web como PWA.
