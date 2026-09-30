# ADR-0005: Cobro mensual inmutable

- **Estado:** Aceptado
- **Fecha:** 2026-09-28

## Contexto
El cobro mensual se envía al inquilino como imagen. Si después se
modifica en el sistema, lo guardado dejaría de coincidir con lo que el
inquilino recibió, y se pierde la trazabilidad del dinero.

## Decisión
Estados `GENERADO → ENVIADO → PAGADO`. A partir de `ENVIADO`, los valores
de un cobro no se editan. Un error se corrige registrando un **ajuste**
(positivo o negativo) que se suma al cobro del mes siguiente. Confirmado
por el administrador (ver [reglas de negocio](../reglas-de-negocio.md),
sección 3).

## Alternativas consideradas
- **Editar el cobro y reenviarlo:** más simple, pero se pierde el
  historial de lo que realmente se cobró.

## Consecuencias
- **Positivas:** trazabilidad completa; lo guardado siempre coincide con
  lo enviado; auditoría sin complejidad extra.
- **Negativas:** una corrección no se ve hasta el mes siguiente.
- **Cuándo revisarla:** si aparece la necesidad de anular un cobro
  completo; en ese caso se agregaría un estado `ANULADO`, sin permitir
  la edición.
