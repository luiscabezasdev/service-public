# Plan de Frontend / UI — Web y Android

Este documento cubre lo que faltaba: qué pantallas existen, cómo se
navega entre ellas, y con qué sistema visual se construyen. Aplica tanto
a `web/` (React + MUI) como a `android/` (Compose + Material 3) — ambas
comparten el mismo lenguaje visual (Material Design), así que el diseño
se piensa una sola vez y se traduce a los dos.

## 1. Librería de componentes

- **Web:** [Material UI (MUI)](https://mui.com/) — tablas, formularios,
  navegación y diálogos con buen diseño ya resuelto. Se aprende su API
  (props de componentes) en vez de escribir CSS desde cero.
- **Android:** Jetpack Compose ya trae **Material 3** por defecto — no
  hay librería adicional que instalar, solo usar sus componentes
  (`Scaffold`, `Card`, `OutlinedTextField`, etc.) con el mismo sistema de
  colores que se define abajo.

## 2. Sistema de diseño (colores y estilo)

Paleta simple, de panel administrativo, con significado semántico claro
(que un color siempre signifique lo mismo en toda la app):

| Token | Color | Uso |
|---|---|---|
| `primary` | Azul petróleo `#0F4C5C` | Barra superior, botones principales, enlaces |
| `secondary` | Verde `#2E7D32` | Estados "pagado" / "al día" |
| `warning` | Ámbar `#ED6C02` | Facturas próximas a vencer |
| `error` | Rojo `#C62828` | Facturas vencidas / sin pagar |
| `background` | Gris muy claro `#F5F6F8` | Fondo general |
| `surface` | Blanco `#FFFFFF` | Tarjetas, tablas |

Tipografía: fuente del sistema por defecto de MUI (Roboto) — es la misma
que usa Material 3 en Android, así que texto y jerarquía visual (títulos,
subtítulos, cuerpo) se ven consistentes entre web y app.

## 3. Mapa de pantallas y navegación

```
Login
  └─ (autenticado) → Dashboard
        ├─ Apartamentos
        │     └─ Detalle de Apartamento
        │           ├─ Datos del inquilino / contrato / canon
        │           └─ Mantenimientos del apto
        ├─ Registro de Factura
        │     └─ Vista de reparto calculado (por apto)
        ├─ Reportes Mensuales
        │     └─ Filtro por mes / servicio / apartamento
        └─ Cobros a Inquilinos
              ├─ Generar cobros del mes (uno por apto)
              ├─ Vista del cobro → exportar imagen para compartir
              └─ Marcar cobro como pagado
```

Navegación principal: un menú lateral (drawer) fijo en web con los 5
destinos (Dashboard, Apartamentos, Facturas, Reportes, Cobros); en
Android, una barra de navegación inferior con los mismos 5 destinos —
mismo mapa, cada plataforma con su patrón de navegación nativo.

## 4. Detalle de cada pantalla

### Dashboard
Vista de aterrizaje tras el login. Responde de un vistazo las dos
preguntas del flujo del dinero (ver reglas de negocio, sección 3):
- **¿Qué debo pagar yo?** Facturas por vencer que aún no se han pagado al
  proveedor (en ámbar o rojo según la fecha).
- **¿Qué me deben?** Cobros mensuales pendientes por apartamento y el
  **saldo adelantado** (lo que el administrador ya pagó y los inquilinos
  aún no han devuelto).
- Accesos directos a "Registrar factura" y "Generar cobros del mes".

### Apartamentos
Lista de los 6 apartamentos (tarjeta o fila por apto: número, inquilino
actual, estado del cobro mensual). Click entra al detalle.

**Detalle de Apartamento:** datos del inquilino, fecha de inicio de
contrato, valor del canon, historial de personas reportadas, e historial
de mantenimientos (con opción de agregar uno nuevo).

### Registro de Factura
Formulario: selecciona servicio (agua/gas/energía/aseo/internet-parabólica),
periodo facturado, fecha de vencimiento, **mes de cobro**, valor total
del recibo, y los datos específicos que pida esa estrategia de reparto
(ej. kilovatios para energía). Al guardar, muestra de inmediato cuánto le
toca a cada apartamento involucrado. Desde la factura se marca "pagada al
proveedor" y se adjunta el comprobante.

### Reportes Mensuales
Tabla filtrable por mes y/o servicio: total facturado, lo pagado a
proveedores, lo cobrado a inquilinos, lo pendiente y el saldo adelantado.
Pensada para que el administrador vea de un vistazo la salud financiera
del mes.

### Cobros a Inquilinos
Es la pantalla que cumple el objetivo principal del sistema: cobrar
**todos los servicios del mes en un solo pago** por apartamento.
1. **Generar cobros del mes:** el sistema agrupa los repartos de todas
   las facturas asignadas a ese mes de cobro y crea un cobro por
   apartamento con su total.
2. **Vista del cobro:** el resumen visual (como el reporte actual del
   Apto 301): cada servicio con su periodo facturado, cómo se calculó
   (personas, kilovatios, etc.), su valor, y el **total a pagar**
   destacado. Botón para exportar como imagen y compartir (WhatsApp u
   otro medio).
3. **Marcar como pagado** cuando el inquilino paga, con la fecha.

La imagen debe entenderse sin explicación adicional: el inquilino solo
necesita ver qué se le cobra, por qué, y cuánto paga en total.

## 5. Cuándo se construye esto en el plan

Este diseño se usa como referencia visual desde la **Fase 4** en adelante
(cuando arranca el cliente web) — no es una fase nueva de código, sino el
documento que guía cómo se ven las pantallas que ya estaban planeadas en
las Fases 4, 5, 8 y 13 del [ROADMAP](../ROADMAP.md). Antes de programar la
primera pantalla (Fase 4, login), vale la pena armar un wireframe rápido
(a mano o en una herramienta simple) del Dashboard y Apartamentos para
verificar que el mapa de navegación tiene sentido contigo antes de
codificarlo.
