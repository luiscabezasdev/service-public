# Dónde vive la base de datos — recomendación y cómo asegurarla

## Recomendación: Supabase (PostgreSQL administrado)

Para este proyecto, **Supabase** es la mejor opción entre las alternativas
razonables (Supabase, Neon, Railway, Render), por estas razones concretas:

1. **Plan gratuito suficiente y perdonavidas para aprender**: 500 MB de
   base de datos (de sobra para 6 apartamentos, facturas e historial), y
   el proyecto se pausa solo tras **una semana sin uso** — no cada 5
   minutos de inactividad como Neon, lo cual es más cómodo cuando
   programas en ratos libres entre trabajo y universidad.
2. **PostgreSQL real, sin sorpresas**: es Postgres estándar, así que
   Prisma se conecta exactamente igual que a cualquier otro Postgres — no
   hay que aprender nada especial del proveedor para la parte de datos.
3. **Storage integrado — resuelve la Fase 12 sin buscar otro servicio**:
   cuando llegues a "adjuntar comprobante" (PDF/imagen), Supabase ya trae
   un servicio de almacenamiento de archivos incluido en el mismo
   proyecto. Evitas contratar un tercer servicio (tipo AWS S3) solo para
   guardar comprobantes.
4. **Seguridad de base sin configuración extra**: conexión siempre por
   SSL, y soporta *Row Level Security* nativo de PostgreSQL si en el
   futuro se necesita (hoy no es crítico con un solo usuario, pero la
   base de datos ya lo soporta si se necesita más adelante).

**Alternativa a considerar más adelante:** Neon, si en algún momento
quieres aprender sobre *branching* de bases de datos (crear una copia
aislada de la base para probar una migración sin tocar producción) — es
una función interesante para aprender, pero no esencial para este
proyecto ahora.

**Para el backend (`backend/`, Fase 14):** Railway o Render — cualquiera
de los dos tiene plan gratuito/económico razonable para desplegar la API
Node/Express. Supabase es la base de datos; Railway/Render es donde corre
el código del servidor.

Fuentes consultadas (septiembre 2026):
- [Supabase vs Neon vs Railway (2026): Which PostgreSQL for SaaS? - DEV Community](https://dev.to/ilshadyx/supabase-vs-neon-vs-railway-2026-which-postgresql-for-saas-3h7a)
- [The Best PostgreSQL Hosting for Developers in 2026 - Railway](https://blog.railway.com/p/best-postgresql-hosting-2026)
- [Neon vs Supabase 2026: Benchmarks, Pricing & Verdict](https://designrevision.com/blog/supabase-vs-neon)

## Cómo se asegura la conexión (aplica desde la Fase 1)

1. **La cadena de conexión (`DATABASE_URL`) nunca se commitea.** Vive en
   `.env` (ya está en `.gitignore`), y en producción se configura como
   variable de entorno en el panel de Railway/Render — nunca escrita en
   el código.
2. **Conexión siempre con SSL forzado.** Supabase lo exige por defecto;
   se verifica que la cadena de conexión use `sslmode=require` (o el modo
   que indique Supabase en su panel).
3. **Usar el "connection pooler" de Supabase (PgBouncer), no la conexión
   directa**, para el backend en producción — evita agotar las conexiones
   simultáneas disponibles del plan gratuito. Supabase da dos cadenas de
   conexión distintas en su panel (directa y con pooler); se usa la del
   pooler desde `backend/`.
4. **Rol de base de datos con mínimo privilegio para la aplicación.** El
   backend no usa el usuario `postgres` (administrador total), sino un rol
   propio que solo puede leer y escribir las tablas de la aplicación. Las
   migraciones de Prisma usan una conexión separada (`DIRECT_URL`). Ver
   [arquitectura.md](./arquitectura.md), sección 5.
5. **Contraseña de la base de datos fuerte y única** (generada, no
   reusada de otra cuenta), guardada en un gestor de contraseñas, no en
   texto plano en ningún archivo del repo ni en notas sueltas.
6. **Una base de datos por entorno cuando se pueda** — idealmente un
   proyecto de Supabase para desarrollo y otro para producción, para que
   una prueba local nunca toque datos reales de los inquilinos. Para el
   tamaño de este proyecto, puede posponerse hasta la Fase 14 (despliegue)
   sin problema — durante el desarrollo (Fases 1-13) se trabaja con datos
   de ejemplo en el proyecto gratuito.
7. **Backups**: el plan gratuito de Supabase no incluye point-in-time
   recovery (eso es de planes pagos), así que mientras estés en el plan
   gratuito, exporta un respaldo manual (`pg_dump`) de vez en cuando,
   sobre todo antes de una migración de esquema grande — es buen hábito
   incluso cuando se llegue a tener backups automáticos.

## Cuándo se decide esto en el plan

Esta decisión se toma **antes de la Fase 1** (el ROADMAP ya lo referencia
ahí) — se crea el proyecto en Supabase, se obtiene la cadena de conexión,
y esa es la base de datos contra la que corren las migraciones de Prisma
desde el primer commit de código del backend.
