# ADR-0009: Dinero con decimales exactos y cobro completo

- **Estado:** Aceptado — reemplaza a [ADR-0006](./0006-dinero-en-enteros.md)
- **Fecha:** 2026-09-29

## Contexto
El ADR-0006 guardaba el dinero en pesos enteros. El administrador aclaró
que cada inquilino debe pagar su **valor completo y exacto en el mismo
cobro**, aunque tenga centavos: nada se redondea a pesos enteros ni se
deja para después. Además, algunos valores intermedios (tarifa por
kilovatio, porcentajes de internet/parabólica, valor por persona) tienen
decimales por naturaleza.

## Decisión
1. **Tipo exacto de base de datos, nunca punto flotante:**
   - Valores de dinero (facturas, repartos, cobros, ajustes):
     `NUMERIC(14,2)` en PostgreSQL (`Decimal` en Prisma).
   - Valores unitarios y porcentajes (tarifa por kWh, porcentaje de
     reparto): `NUMERIC(14,6)`, para no perder precisión en el cálculo.
2. **Aritmética exacta en TypeScript** con `decimal.js` (el mismo tipo
   que usa `Decimal` de Prisma), centralizada en `src/shared/money.ts`.
   Nunca se opera dinero con `number`.
3. **Se calcula con toda la precisión y se ajusta a centavos una sola
   vez, al final** (nunca en pasos intermedios).
4. **El cobro es completo en el mismo mes (método del mayor residuo):**
   1. Se calcula la parte exacta de cada apartamento.
   2. Cada parte se trunca a centavos (hacia abajo). La suma queda igual
      o unos centavos por debajo de la factura, nunca por encima.
   3. Los centavos que faltan se asignan, de a uno, a los apartamentos
      con mayor parte decimal descartada; si empatan, al de menor número
      de apartamento.

   Ejemplo: $10.000 / 3 → $3.333,34 + $3.333,33 + $3.333,33 = $10.000,00.
   La suma de los repartos es **siempre exactamente** el valor de la
   factura, y nada queda pendiente para otro mes. (Verificado con los
   casos $10.000 entre 3, $35.012 por personas, $10.148 entre 6 y $8.000
   entre 7.)
5. **Se muestra con dos decimales** en pantallas y en la imagen del
   cobro (formato colombiano: `$3.333,34`).

## Alternativas consideradas
- **Pesos enteros (ADR-0006):** descartado; redondea los valores y no
  refleja el monto exacto que se quiere cobrar.
- **Decimales de punto flotante (`number`/`float`):** descartado; no
  representan exactamente muchos valores (`0.1 + 0.2 = 0.30000000000000004`).
- **Dejar la diferencia de centavos para el mes siguiente:** descartado;
  el cobro debe estar completo en el mismo mes.

## Consecuencias
- **Positivas:** cada inquilino paga exactamente lo que le corresponde;
  el total cobrado es exactamente lo que el administrador pagó; los
  cálculos se pueden probar al centavo.
- **Negativas:** `decimal.js` es más verboso que operar con números
  normales (`a.plus(b)` en vez de `a + b`), y `NUMERIC` es algo más lento
  que un entero (irrelevante para el volumen de este sistema).
- **Pruebas obligatorias:** para cada estrategia de reparto, un test que
  verifique que la suma de los repartos es igual al valor de la factura,
  incluidos casos con divisiones inexactas.
- **Cuándo revisarla:** si se necesita otra moneda o más precisión.

## Referencias
- [PostgreSQL — Tipos numéricos (NUMERIC)](https://www.postgresql.org/docs/current/datatype-numeric.html)
- [decimal.js](https://mikemcl.github.io/decimal.js/)
