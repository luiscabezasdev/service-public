# Plan de trabajo — Sistema de Gestión de Arrendamiento (Full-Stack)

Este documento explica **qué se construye en cada fase y por qué en ese
orden**. El estado de avance (checkboxes) vive en el
[ROADMAP](../ROADMAP.md), y qué estudiar en cada fase está en el
[plan de aprendizaje](./plan-de-aprendizaje.md).

**Stack:** Backend Node.js + Express + TypeScript · PostgreSQL en Supabase
(Prisma) · Web React + TypeScript + MUI · Android Kotlin + Jetpack
Compose. Los clientes (web y Android) consumen la **misma API**.

**Objetivo central:** el administrador paga los recibos de servicios por
anticipado y cobra a cada inquilino **todos los servicios del mes en un
solo pago**, con un reporte claro que se envía como imagen (ver
[reglas de negocio](./reglas-de-negocio.md), sección 3).

Cada fase termina con: código funcionando + Pull Request con el resumen
de lo aprendido + visto bueno antes de seguir.

---

## Bloque A — Fundamentos del backend

### Fase 0 — Diseño del contrato de la API
**Qué se hace:** antes de escribir código se define en un documento el
esquema de PostgreSQL (`usuarios`, `apartamentos`, `inquilinos`,
`registro_personas`, `mantenimientos`, `grupos_servicio`,
`configuracion_reparto`, `facturas`, `repartos`, `cobros_mensuales`) y la
contrato de la API en OpenAPI (`docs/api/openapi.yaml`, rutas bajo
`/api/v1`). Se crea el proyecto en Supabase, se obtiene la cadena de
conexión y se crea el rol de base de datos con mínimo privilegio.
**Concepto nuevo:** diseño de API REST, OpenAPI, modelado relacional
(tablas, claves foráneas, relaciones 1:N y N:M, restricciones de
integridad).
**Resultado:** un documento que sirve de referencia para las 3 piezas.

### Fase 1 — Backend: esqueleto + conexión a la base de datos
**Qué se hace:** proyecto Node/Express/TypeScript con la estructura por
módulos de la [arquitectura](./arquitectura.md), conexión a Supabase con
Prisma, migraciones que crean las tablas de la Fase 0. Base de
operación: validación de variables de entorno, logs estructurados,
manejador central de errores y `/api/v1/health`. Base de seguridad
(`helmet`, CORS, `.env`) y de calidad (TypeScript estricto, ESLint,
Prettier, hooks de Git, CI en GitHub Actions y Dependabot).
**Concepto nuevo:** ORM, migraciones, variables de entorno, arquitectura
por capas, git hooks, integración continua.
**Resultado:** servidor corriendo localmente que lee y escribe en una base
de datos real.

### Fase 2 — Backend: autenticación
**Qué se hace:** endpoint de login, hash de contraseña con `bcrypt`, JWT
con expiración, middleware que protege las demás rutas, rate limiting.
**Concepto nuevo:** hashing, JWT, middleware.
**Resultado:** solo con usuario y contraseña correctos se accede a la API.

### Fase 3 — Backend: CRUD de Apartamentos, Inquilinos, Mantenimientos
**Qué se hace:** endpoints protegidos para crear, leer, actualizar y
borrar estas entidades, con validación de entrada (`zod`) y manejo
centralizado de errores.
**Concepto nuevo:** validación en el servidor, códigos de estado HTTP.
**Resultado:** API completa para la gestión básica del edificio.

---

## Bloque B — Primer cliente funcionando (web)

### Fase 3.5 — Diseño de UI/UX
**Qué se hace:** wireframes del Dashboard, Apartamentos y Cobros sobre el
[plan de UI](./plan-frontend-ui.md), antes de programar pantallas.
**Resultado:** mapa de pantallas validado.

### Fase 4 — Web: login + consumo de la API
**Qué se hace:** proyecto React + TypeScript + MUI, pantalla de login,
manejo del token, rutas protegidas.
**Concepto nuevo:** componentes, estado, efectos, peticiones HTTP desde
React, React Router.
**Resultado:** inicio de sesión desde el navegador.

### Fase 5 — Web: CRUD de Apartamentos/Inquilinos/Mantenimientos
**Qué se hace:** pantallas que consumen los endpoints de la Fase 3.
**Resultado:** primer flujo completo de punta a punta: login web →
gestionar apartamentos → guardado en PostgreSQL.

*(Punto de control: algo demostrable completo, aunque falte el dinero y
Android.)*

---

## Bloque C — El motor de reparto y el cobro consolidado

### Fase 6 — Backend: motor de reparto (patrón Strategy)
**Qué se hace:** una interfaz común con 5 implementaciones (por cabeza,
individual, diferencia de medidor, igualitaria, manual/fórmula) y la
suma del cobro mensual por apartamento. **Se prueba con tests unitarios
(Jest)** usando valores con la misma forma de un recibo real, pero
inventados, antes de exponerlo en la API.
**Concepto nuevo:** interfaces, polimorfismo, patrón Strategy, testing
unitario, manejo de dinero sin errores de redondeo.
**Resultado:** dado un recibo, el backend calcula correctamente cuánto le
toca a cada apartamento y el total del cobro mensual.

### Fase 7 — Backend: facturas, cobros mensuales y reportes
**Qué se hace:** registrar factura (con mes de cobro y pago al proveedor),
generar los cobros mensuales por apartamento, marcar cobro pagado, y
reportes (pagado a proveedores, cobrado a inquilinos, saldo adelantado).
**Concepto nuevo:** transacciones de base de datos (generar los 6 cobros
de un mes completo o ninguno).
**Resultado:** API completa del flujo del dinero.

### Fase 8 — Web: facturas, cobros y reportes
**Qué se hace:** formulario de factura, vista del reparto calculado,
pantalla **Cobros a Inquilinos** (generar, ver, marcar pagado) y
reportes mensuales.
**Resultado:** el flujo del dinero funciona completo desde la web.

---

## Bloque D — Android como segundo cliente

### Fase 9 — Android: login + consumo de la API
**Qué se hace:** Retrofit para llamar a la misma API, pantalla de login,
token guardado de forma segura (DataStore).
**Concepto nuevo:** Retrofit, corrutinas, DataStore.
**Resultado:** inicio de sesión desde el celular.

### Fase 10 — Android: pantallas equivalentes a la web
**Qué se hace:** apartamentos, mantenimientos, facturas, cobros y
reportes, reutilizando lo aprendido en Compose en IU Digital Radio.
**Resultado:** app Android a la par de la web.

---

## Bloque E — Funcionalidades finales

### Fase 11 — Alertas
(1) Facturas por vencer que aún no se han pagado al proveedor y (2)
cobros mensuales que el inquilino no ha pagado. Job programado en el
backend o notificación local en Android.

### Fase 12 — Adjuntar comprobante
Subida de PDF o imagen del pago al proveedor, guardado en Supabase
Storage y asociado a la factura.
**Concepto nuevo:** subida de archivos, validación de tipo y tamaño.

### Fase 13 — Exportar y compartir el cobro como imagen
La vista del cobro mensual (existente desde la Fase 8) se exporta como
imagen para enviarla al inquilino.

### Fase 14 — Despliegue
Backend en Railway o Render, base de datos de producción en un proyecto
de Supabase separado del de desarrollo, web en Vercel. Guía de operación
y prueba real de restauración de un respaldo.
**Concepto nuevo:** variables de entorno en producción, CORS, HTTPS,
despliegue continuo, operación (respaldos y rotación de secretos).

### Fase 15 — Pulido (opcional)
Validaciones finas, manejo de errores, UX.

---

## Por qué este orden

El backend va primero (Bloque A) porque es la única pieza de la que
dependen las otras dos. Luego se conecta un solo cliente (web, Bloque B)
para validar que el contrato de la API funciona de punta a punta antes de
duplicar ese esfuerzo en Android. El motor de reparto (Bloque C) se aísla
y se prueba con tests antes de ponerle interfaz, para no mezclar errores
de cálculo con errores de pantalla: es la lógica que maneja el dinero de
los inquilinos. Android (Bloque D) llega cuando el contrato de la API ya
está estable, para no cambiar API y cliente al mismo tiempo. El
despliegue va al final porque no tiene sentido publicar algo que todavía
cambia de forma cada semana.
