# ADR-0007: Motor de reparto con patrón Strategy en dominio puro

- **Estado:** Aceptado
- **Fecha:** 2026-09-27

## Contexto
Hay cinco reglas distintas para repartir servicios entre apartamentos y
pueden aparecer más. Es la lógica que maneja el dinero de los inquilinos,
así que debe ser exacta y fácil de probar.

## Decisión
Una interfaz común de estrategia de reparto con una implementación por
regla (por cabeza, individual, diferencia de medidor, igualitaria,
manual/fórmula), ubicada en `src/domain/reparto/` como TypeScript puro:
sin dependencias de Express, Prisma ni base de datos.

## Alternativas consideradas
- **Un `switch`/`if` por servicio dentro del servicio de facturas:**
  crece sin control con cada regla nueva y mezcla cálculo con acceso a
  datos, lo que hace difícil probarlo.

## Consecuencias
- **Positivas:** agregar una regla es agregar un archivo; las pruebas no
  necesitan base de datos y corren en milisegundos.
- **Negativas:** más archivos y una abstracción que hay que entender
  antes de programarla.
- **Cuándo revisarla:** no se espera; es el núcleo del sistema.
