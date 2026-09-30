# Reglas de negocio — App de Gestión de Arrendamiento (6 Apartamentos)

## 1. Alcance real del proyecto

No es solo "registrar servicios públicos": es un sistema full-stack de
gestión de arrendamiento para un edificio de **6 apartamentos numerados**,
con un único rol de usuario (el arrendador/administrador), que debe poder
entrar **desde la app Android y desde una web**, desde cualquier lugar.
Los inquilinos nunca usan el sistema; solo reciben una imagen del **cobro
mensual consolidado** (por WhatsApp u otro medio) generada por el
administrador — ver sección 3 para el flujo completo del dinero.

Por eso **sí se necesita login con autenticación real** (JWT) y una base
de datos centralizada en la nube (no local) — es lo que permite que el
mismo dato se vea igual desde el celular y desde el navegador. La
privacidad de canon y mantenimientos ahora se resuelve con el login, no
con la ausencia de él.

## 2. Entidades de dominio (visión general, sin código todavía)

- **Usuario (administrador)**: correo/usuario, contraseña (hash con
  bcrypt, nunca en texto plano). Hoy es un solo registro — la tabla existe
  igual desde el principio porque el login la necesita, y porque deja la
  puerta abierta a un segundo administrador en el futuro sin rediseñar nada.
- **Apartamento**: número, inquilino actual, fecha de inicio del contrato,
  valor del canon (privado, solo lo ve el administrador)
- **Mantenimiento**: historial de arreglos por apartamento (privado)
- **Registro de personas**: cantidad de personas viviendo en cada
  apartamento, reportada por el inquilino, puede cambiar en el tiempo
  (se necesita guardar el historial, no solo el valor actual, porque el
  agua y el gas compartido se calculan con la cantidad vigente en cada
  periodo facturado)
- **Factura**: un recibo del proveedor (servicio, periodo facturado,
  valor total, fecha de vencimiento, y si aplica, el grupo de apartamentos
  al que corresponde). Guarda también el **pago al proveedor** que hace el
  administrador: estado (pendiente/pagada), fecha de pago y comprobante.
  Además tiene asignado el **mes de cobro** en que se le cobrará a los
  inquilinos (ver sección 3).
- **Reparto por apartamento** (registro derivado de cada Factura): cuánto
  le corresponde a cada apartamento de esa factura, con los datos que
  explican el cálculo (personas, kilovatios, porcentaje). No tiene estado
  de pago propio: se cobra dentro del Cobro Mensual.
- **Cobro mensual** (cuenta de cobro): un registro por apartamento y por
  mes de cobro que agrupa todos sus repartos de ese mes y su **total a
  pagar**. Es lo que el inquilino paga en **un solo pago**, y es lo que se
  muestra en la imagen que se le envía. Guarda su propio estado
  (pendiente/pagado, fecha de pago del inquilino). El Dashboard y los
  Reportes Mensuales leen de aquí qué inquilinos están al día.

## 3. Flujo del dinero: pago anticipado y cobro consolidado

Este es el propósito central del sistema. El administrador paga cada
recibo al proveedor **antes** de cobrarlo, para evitar retrasos y
recargos. Después le cobra a cada inquilino **todos los servicios del
mes en un solo pago**, en vez de cobrar recibo por recibo (lo cual es
incómodo para ambas partes).

```
1. Llega un recibo del proveedor        → se registra la Factura
2. El administrador lo paga             → Factura: pagada al proveedor (+ comprobante)
3. El sistema calcula el reparto        → Reparto por apartamento
4. Cierre del mes de cobro              → un Cobro Mensual por apartamento
                                           (suma de todos sus repartos del mes)
5. Se genera la imagen del cobro        → se envía al inquilino
6. El inquilino paga una sola vez       → Cobro Mensual: pagado
```

**Consecuencias en el diseño:**

- **Hay dos estados de pago distintos, y no se deben mezclar:** el pago
  del administrador al proveedor (en la Factura) y el pago del inquilino
  al administrador (en el Cobro Mensual). Una factura puede estar pagada
  al proveedor mientras el cobro al inquilino sigue pendiente; esa
  diferencia es justamente el dinero que el administrador ha adelantado.
- **Mes de cobro ≠ periodo facturado:** cada servicio tiene su propio
  periodo (agua abril–junio, energía 24 jul–24 ago), así que al registrar
  una factura se le asigna en qué mes de cobro entra. El reporte muestra
  el periodo de cada servicio como referencia, pero cobra por mes.
- **Agua (bimestral):** en los meses sin recibo de agua, la fila aparece
  en el cobro con valor $0, como en el reporte actual del Apto 301.
- **Contenido del cobro:** solo servicios públicos. El canon no se
  incluye en este reporte.
- **Saldo adelantado (dato útil para el Dashboard):** la suma de facturas
  pagadas al proveedor cuyos cobros mensuales siguen pendientes. Indica
  cuánto dinero ha puesto el administrador que todavía no le han
  devuelto.
- **Dos tipos de alerta (Fase 11):** (1) factura por vencer que el
  administrador aún no ha pagado al proveedor; (2) cobro mensual que el
  inquilino aún no ha pagado.
- **El cobro mensual se congela al generarse:** una vez enviado al
  inquilino, sus valores no cambian aunque después se corrija algo. Si
  hay un error, se registra un ajuste en el cobro del mes siguiente, para
  que lo que el inquilino recibió siempre coincida con lo guardado.

## 4. Reglas de prorrateo por servicio

| Servicio | Periodicidad | Regla de reparto |
|---|---|---|
| **Agua** | Bimestral | Un recibo para los 6 aptos. Se cobra por cabeza: `valor_apto = (personas_apto / personas_totales) × valor_recibo` |
| **Gas** | Mensual | 4 aptos tienen recibo/medidor individual propio (pagan su monto directo). Los otros 2 comparten un segundo recibo, dividido por cabeza entre esos 2 |
| **Energía** | Mensual | Los 6 aptos están organizados en **3 parejas fijas** (no cambian con el tiempo). En cada pareja: un apto tiene medidor individual y paga su consumo directo; el otro paga la diferencia (`valor_pareja_total − valor_consumo_medidor_individual`) |
| **Aseo** | Mensual | Un recibo único para los 6 aptos, dividido en **partes iguales** (recibo / 6) |
| **Internet / Parabólica** | Mensual | Un recibo único para los 6 aptos. Reparto **configurable por apartamento**: porcentaje fijo manual, o por fórmula (ej. por personas). El administrador decide qué tipo de reparto usa cada apto, y puede cambiarlo |

## 5. Concepto técnico clave: "estrategia de reparto"

En vez de escribir un `if`/`when` distinto por cada servicio (lo cual se
vuelve inmanejable con 5 servicios y reglas distintas), cada servicio se
modela con una **estrategia de reparto** intercambiable:

- `RepartoPorCabeza` (agua, y el gas compartido entre 2)
- `RepartoIndividual` (gas de los 4 aptos con medidor propio)
- `RepartoPorDiferenciaMedidor` (energía, por parejas)
- `RepartoIgualitario` (aseo)
- `RepartoManualOFormula` (internet/parabólica, configurable por apto)

Esto es el patrón de diseño **Strategy**: una interfaz común
(`calcularValorApto(...)`) con una implementación distinta por regla. Así,
agregar una regla nueva en el futuro no obliga a tocar el código existente
— solo se agrega una estrategia más. Es un concepto nuevo que vale la pena
explicar con calma antes de programarlo (según la constitución del proyecto).

## 6. Reglas ya resueltas

- Periodicidad: agua bimestral; gas, energía, aseo e internet/parabólica mensuales
- Los grupos (parejas de energía, par de gas compartido) son fijos, no cambian de apartamentos con el tiempo
- Si un inquilino no reporta el número de personas actualizado a tiempo, se usa el último valor conocido
- El administrador paga los recibos al proveedor por anticipado y luego cobra a cada inquilino todos los servicios del mes en un solo pago (sección 3)
- El cobro consolidado incluye solo servicios públicos, no el canon

