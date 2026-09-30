# ADR-0003: PostgreSQL administrado en Supabase

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto
Los datos son relacionales (apartamentos, inquilinos, facturas,
repartos, cobros) y deben estar disponibles en la nube para web y
Android, con costo cero o bajo y operación sencilla.

## Decisión
PostgreSQL administrado en Supabase, conectado desde el backend con
Prisma a través del connection pooler, y Supabase Storage para los
comprobantes. Detalle en [hosting-base-datos.md](../hosting-base-datos.md).

## Alternativas consideradas
- **Neon:** muy buena opción, pero el plan gratuito suspende la base tras
  pocos minutos de inactividad y no incluye almacenamiento de archivos.
- **Railway / Render Postgres:** planes gratuitos menos generosos para
  la base de datos.
- **MongoDB:** los datos son claramente relacionales; perderíamos
  claves foráneas y restricciones de integridad.

## Consecuencias
- **Positivas:** PostgreSQL estándar (sin dependencia fuerte del
  proveedor: se puede migrar con `pg_dump`), SSL por defecto,
  almacenamiento integrado.
- **Negativas:** el plan gratuito no incluye recuperación a un punto en
  el tiempo; se compensa con respaldos manuales.
- **Cuándo revisarla:** si se superan los límites del plan gratuito o se
  necesitan respaldos automáticos con datos reales.
