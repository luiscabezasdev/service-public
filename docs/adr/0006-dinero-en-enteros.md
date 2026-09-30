# ADR-0006: Dinero en pesos enteros

- **Estado:** Reemplazado por [ADR-0009](./0009-dinero-decimal-exacto.md)
  (el cobro debe ser exacto, con centavos si hace falta)
- **Fecha:** 2026-09-29

## Contexto
Los números decimales de punto flotante no representan exactamente
muchos valores (`0.1 + 0.2 = 0.30000000000000004`). Con repartos entre
varios apartamentos, esos errores se acumulan y la suma deja de cuadrar
con la factura.

## Decisión
Todos los valores de dinero se guardan y calculan como **pesos enteros**
(tipo entero en la base de datos y `number` entero en TypeScript, con
validación). Las divisiones se redondean con una regla fija y el residuo
se asigna de forma determinista, de modo que la suma de los repartos sea
siempre exactamente el valor de la factura. Toda operación de dinero pasa
por `src/shared/money.ts`.

## Alternativas consideradas
- **Decimales de punto flotante:** descartado por los errores de
  precisión.
- **Tipo `Decimal` de PostgreSQL/Prisma:** exacto, pero más complejo de
  manejar en TypeScript, y el peso colombiano no usa centavos en la
  práctica.

## Consecuencias
- **Positivas:** cálculos exactos y fáciles de probar.
- **Negativas:** si algún día se manejaran centavos u otra moneda, habría
  que migrar a centavos enteros o a `Decimal`.
- **Cuándo revisarla:** si se necesita manejar fracciones de peso.
