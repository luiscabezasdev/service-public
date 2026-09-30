# Arquitectura del Sistema

Este documento define **cómo está construido el sistema y por qué**, para
que siga siendo seguro, fácil de mantener y capaz de crecer sin tener que
reescribirse. Es la referencia técnica de todas las fases del
[ROADMAP](../ROADMAP.md). Las decisiones importantes quedan registradas
en [docs/adr/](./adr/README.md).

---

## 1. Atributos de calidad, en orden de prioridad

Cuando dos objetivos chocan, gana el que está más arriba:

| # | Atributo | Qué significa en este proyecto |
|---|---|---|
| 1 | **Seguridad** | Los datos de inquilinos y dinero solo los ve el administrador autenticado; ningún secreto sale del servidor. |
| 2 | **Corrección del dinero** | Un reparto siempre suma exactamente el valor de la factura; un cobro enviado nunca cambia. |
| 3 | **Mantenibilidad** | Cualquier cambio (una regla de reparto nueva, una pantalla nueva) toca pocos archivos y está cubierto por pruebas. |
| 4 | **Escalabilidad** | El sistema puede crecer (más edificios, más administradores) con migraciones, no con una reescritura. |
| 5 | **Rendimiento** | Suficiente para el uso real. No se optimiza sin una medición que lo justifique. |

**Sostenibilidad** atraviesa las cinco: el sistema debe poder operarlo y
entenderlo una sola persona, con planes gratuitos o de bajo costo, y
cualquier decisión debe estar documentada para que en un año siga siendo
comprensible.

---

## 2. Vista general

```
 ┌──────────────┐     ┌──────────────┐
 │  Web (React) │     │   Android    │     Clientes: solo presentación.
 │   Vercel     │     │  (Compose)   │     Sin reglas de negocio.
 └──────┬───────┘     └──────┬───────┘
        │   HTTPS + JWT      │
        └─────────┬──────────┘
                  ▼
        ┌───────────────────┐
        │  API REST /api/v1 │               Única fuente de verdad de
        │ Node + Express    │               las reglas de negocio.
        │ Railway / Render  │               Sin estado (stateless).
        └────────┬──────────┘
                 │  SSL, connection pooler
        ┌────────┴──────────────────┐
        ▼                           ▼
 ┌──────────────┐          ┌──────────────────┐
 │  PostgreSQL  │          │ Supabase Storage │
 │  (Supabase)  │          │  (comprobantes)  │
 └──────────────┘          └──────────────────┘
```

---

## 3. Principios

1. **Las reglas de negocio viven solo en el backend.** Web y Android
   muestran y envían datos; nunca calculan un reparto ni deciden un
   estado de pago. Si el cálculo estuviera en los clientes, habría dos
   implementaciones que podrían dar resultados distintos.
2. **El dominio no depende de la infraestructura.** El motor de reparto
   es TypeScript puro: no importa Express, Prisma ni nada externo. Recibe
   datos y devuelve resultados. Por eso se puede probar sin base de datos
   y sobrevive a un cambio de framework.
3. **Las dependencias apuntan hacia adentro:** rutas → controladores →
   servicios → repositorios. Nunca al revés, y el dominio no conoce a
   nadie.
4. **Contrato primero.** La API se describe en OpenAPI antes de
   programarla; backend, web y Android se ajustan a ese contrato.
5. **API sin estado.** Toda la información de sesión viaja en el JWT; el
   servidor no guarda sesiones en memoria. Esto permite correr varias
   instancias si algún día hace falta.
6. **Configuración fuera del código** y validada al arrancar: si falta
   una variable de entorno, el servidor no inicia (falla rápido, con un
   mensaje claro).
7. **Simple primero (YAGNI).** No se construye infraestructura para un
   problema que todavía no existe. Ver la sección 9 sobre lo que
   deliberadamente no se hace.

---

## 4. Estructura del backend

Organización **por módulo de negocio** (no por tipo de archivo), con las
mismas capas dentro de cada módulo:

```
backend/
├── prisma/
│   ├── schema.prisma          # modelo de datos
│   └── migrations/            # historial de cambios del esquema
├── src/
│   ├── config/
│   │   └── env.ts             # lee y valida variables de entorno (zod)
│   ├── shared/
│   │   ├── errors/            # clases de error y manejador central
│   │   ├── middleware/        # auth, rate limit, requestId, validación
│   │   ├── logger.ts          # logs estructurados (pino)
│   │   └── money.ts           # aritmética exacta de dinero (decimal.js)
│   ├── domain/
│   │   └── reparto/           # motor de reparto: TypeScript puro
│   │       ├── estrategia.ts  # interfaz común
│   │       ├── por-cabeza.ts
│   │       ├── individual.ts
│   │       ├── diferencia-medidor.ts
│   │       ├── igualitario.ts
│   │       ├── manual-formula.ts
│   │       └── *.test.ts      # pruebas junto al código
│   ├── modules/
│   │   ├── auth/
│   │   ├── apartamentos/
│   │   ├── inquilinos/
│   │   ├── mantenimientos/
│   │   ├── facturas/
│   │   ├── cobros/
│   │   └── reportes/
│   ├── app.ts                 # arma Express: middleware y rutas
│   └── server.ts              # arranca el servidor
└── tests/
    └── integration/           # pruebas de endpoints con base de datos de prueba
```

Cada módulo tiene los mismos archivos:

| Archivo | Responsabilidad | Puede usar |
|---|---|---|
| `*.routes.ts` | Define URL, método y middleware | controller |
| `*.controller.ts` | Traduce HTTP ↔ servicio (lee request, arma response) | service |
| `*.schemas.ts` | Esquemas zod de entrada y salida | — |
| `*.service.ts` | Reglas de negocio y orquestación | repository, domain, otros services |
| `*.repository.ts` | Único lugar que habla con Prisma | Prisma |

**Reglas que mantienen esto ordenado:**
- Un módulo no consulta las tablas de otro módulo: le pide los datos a
  su servicio. Así, si cambia la tabla de inquilinos, solo cambia el
  módulo de inquilinos.
- Los controladores no tienen lógica de negocio; los servicios no saben
  nada de HTTP (no reciben `req` ni `res`).
- **Por qué por módulo y no por tipo:** al trabajar en "cobros" todo lo
  necesario está en una carpeta. Con carpetas por tipo (`controllers/`,
  `services/`…) un cambio pequeño obliga a saltar entre cinco carpetas.

---

## 5. Modelo de datos e integridad

La base de datos es la última línea de defensa: aunque el código tenga un
error, las restricciones impiden guardar datos inválidos.

| Regla | Cómo se aplica |
|---|---|
| **Dinero exacto, con centavos** | Valores de dinero en `NUMERIC(14,2)` y valores unitarios o porcentajes en `NUMERIC(14,6)`; cálculos con `decimal.js`, nunca con `number` ni punto flotante. Se calcula con toda la precisión y se redondea una sola vez al final ([ADR-0009](./adr/0009-dinero-decimal-exacto.md)). |
| **El cobro es completo** | Si al redondear a centavos queda una diferencia (ej. $10.000 / 3), se asigna en el mismo reparto, centavo por centavo, con el método del mayor residuo. La suma de los repartos es siempre exactamente el valor de la factura y nada queda pendiente para otro mes. Una prueba lo verifica para cada estrategia. |
| **Valores válidos** | Restricciones `CHECK` (valores ≥ 0, personas ≥ 0) y `NOT NULL` donde aplique. |
| **Sin cobros duplicados** | Restricción `UNIQUE (apartamento_id, mes_cobro)` en cobros mensuales: generar dos veces el mismo mes es imposible. |
| **Relaciones consistentes** | Claves foráneas en todas las relaciones; no se puede borrar un apartamento con facturas o cobros asociados. |
| **Cobro inmutable** | Estados `GENERADO → ENVIADO → PAGADO`. Desde `ENVIADO` no se editan valores; las correcciones son **ajustes** (filas nuevas) en el cobro del mes siguiente ([ADR-0005](./adr/0005-cobro-inmutable.md)). |
| **Historial, no sobrescritura** | Personas por apartamento, inquilinos anteriores y ajustes se guardan como historial. Los inquilinos que se van se marcan inactivos, no se borran. |
| **Operaciones atómicas** | Generar los cobros de un mes es una transacción: se crean todos o ninguno. |
| **Auditoría básica** | Todas las tablas tienen `created_at` y `updated_at`; las tablas de dinero registran también qué usuario hizo el cambio. |
| **Fechas** | Se guardan en UTC (`timestamptz`) y se muestran en hora de Colombia. `mes_cobro` se guarda como el primer día del mes. |
| **Índices** | En todas las claves foráneas y en `mes_cobro`, que es el filtro más usado. |

**Mínimo privilegio en la base de datos:** el backend no se conecta con
el usuario administrador `postgres` de Supabase, sino con un rol propio
que solo puede leer y escribir las tablas de la aplicación (sin permiso
para borrar tablas ni cambiar el esquema). Las migraciones usan una
conexión separada. Si alguien comprometiera la API, el daño posible es
menor.

---

## 6. Diseño de la API

- **Versionada desde el inicio:** todas las rutas bajo `/api/v1`. Si
  algún día hay un cambio incompatible, se crea `/api/v2` sin romper una
  versión de la app Android que alguien no haya actualizado.
- **Contrato en OpenAPI** (`docs/api/openapi.yaml`, se escribe en la
  Fase 0): una sola descripción de la API para las tres piezas. Además
  genera documentación navegable.
- **Formato de error único** en toda la API:
  ```json
  { "error": { "code": "COBRO_YA_ENVIADO", "message": "El cobro ya fue enviado y no se puede modificar", "requestId": "a1b2c3" } }
  ```
  Nunca incluye *stack traces* ni detalles internos. El `requestId`
  permite encontrar el error exacto en los logs.
- **Códigos HTTP correctos:** 201 al crear, 400 datos inválidos, 401 sin
  sesión, 403 sin permiso, 404 no existe, 409 conflicto (ej. cobro ya
  generado), 500 error interno.
- **Paginación** en todo listado (`?page=1&limit=20`), aunque hoy haya
  pocos registros.
- **Salud del servicio:** `GET /api/v1/health` responde si la API y la
  base de datos están disponibles (lo usan Railway/Render para saber si
  reiniciar el servicio).

---

## 7. Seguridad por capas (defensa en profundidad)

Ninguna capa se considera suficiente sola. Detalle completo en
[seguridad-y-buenas-practicas.md](./seguridad-y-buenas-practicas.md).

| Capa | Controles |
|---|---|
| Transporte | HTTPS obligatorio, CORS limitado al dominio de la web |
| Borde de la API | `helmet`, rate limiting general y estricto en login, límite de tamaño del body |
| Autenticación | bcrypt, JWT con expiración corta, secreto fuerte en variable de entorno |
| Entrada | Validación con zod en cada endpoint; solo se aceptan los campos definidos |
| Lógica | Autorización en el servicio, no en el cliente; estados de cobro controlados |
| Datos | Prisma (consultas parametrizadas), rol de BD con mínimo privilegio, restricciones en el esquema |
| Archivos | Tipo y tamaño validados en el servidor; bucket privado con enlaces temporales |
| Secretos | Solo en variables de entorno; gitleaks + push protection |
| Registro | Logs sin contraseñas, tokens ni datos personales completos |
| Dependencias | `npm audit` en CI y Dependabot |

---

## 8. Clientes

**Web (`web/`)**, organizada también por funcionalidad:
```
web/src/
├── api/          # cliente HTTP único: agrega el token y maneja errores
├── features/     # auth/, apartamentos/, facturas/, cobros/, reportes/
├── components/   # componentes reutilizables (tabla, formulario, layout)
├── routes/       # rutas y protección de rutas
└── theme/        # tema de MUI (colores del plan de UI)
```

**Android (`android/`)**, siguiendo la
[guía oficial de arquitectura](https://developer.android.com/topic/architecture):
capa de UI (pantallas Compose + ViewModel) y capa de datos (repositorios
+ Retrofit), con flujo de datos unidireccional.

**En ambos:** el token se guarda de forma segura, toda llamada pasa por
un único cliente HTTP (un solo lugar para manejar el 401 y cerrar
sesión), y la pantalla siempre tiene estado de carga, error y vacío.

---

## 9. Escalabilidad realista

**Lo que ya queda preparado sin costo extra:**
- API sin estado → se pueden correr varias instancias detrás de un
  balanceador.
- Connection pooler de Supabase → soporta muchas conexiones concurrentes.
- Paginación e índices → las consultas no se degradan al crecer los datos.
- Módulos independientes → un módulo se puede extraer si algún día lo
  justifica.

**Caminos de crecimiento (documentados, no implementados):**

| Si pasa esto… | …se hace esto |
|---|---|
| Administras un segundo edificio | Tabla `edificios` + `edificio_id` en apartamentos, con una migración. Como todo acceso a datos pasa por los repositorios, el filtro por edificio se agrega en un solo lugar por módulo. |
| Otra persona administra contigo | Roles en `usuarios` + autorización por rol en el middleware (ya previsto en API5 de seguridad). |
| Las alertas o reportes se vuelven pesados | Pasar ese trabajo a un proceso en segundo plano (cola de tareas). |
| Mucho tráfico de lectura | Caché para reportes. |

**Lo que deliberadamente NO se hace ahora** (y cuándo sí tendría sentido):
- **Microservicios:** solo con varios equipos trabajando en partes
  distintas. Para una persona, un monolito modular es más seguro y
  mantenible.
- **Kubernetes / contenedores orquestados:** solo con tráfico que un
  servicio administrado no soporte.
- **Redis / caché:** solo cuando una medición muestre consultas lentas.
- **Event sourcing:** el historial y los ajustes cubren la auditoría
  necesaria sin esa complejidad.

Construir esto antes de necesitarlo haría el sistema más difícil de
mantener y más caro, sin ningún beneficio real.

---

## 10. Mantenibilidad

- **TypeScript en modo estricto** (`"strict": true`) en backend y web.
- **ESLint + Prettier** con reglas compartidas; se ejecutan solos en cada
  commit (ver [control-de-versiones.md](./control-de-versiones.md)).
- **Pirámide de pruebas:**
  - Muchas pruebas unitarias en `domain/reparto/`: es donde está el
    dinero. Meta: 100% de las reglas de reparto cubiertas, incluidos los
    casos límite.
  - Pruebas de integración para los endpoints críticos (login, generar
    cobros, marcar pagado) contra una base de datos de prueba.
  - Pocas pruebas de punta a punta, solo del flujo principal.
- **Integración continua (GitHub Actions)** en cada Pull Request: lint →
  verificación de tipos → pruebas → `npm audit` → gitleaks. Si algo
  falla, no se puede hacer merge.
- **Dependabot** revisa dependencias semanalmente (se configura en la
  Fase 1, cuando existan los `package.json`).
- **ADRs:** cada decisión técnica importante se registra en
  [docs/adr/](./adr/README.md) con su contexto y alternativas. Evita
  repetir discusiones y explica el sistema a quien llegue después
  (incluido tú mismo en un año).

---

## 11. Operación sostenible

- **Entornos separados:** desarrollo y producción con proyectos de
  Supabase distintos y variables de entorno distintas.
- **Migraciones solo hacia adelante:** nunca se edita una migración ya
  aplicada; los cambios se hacen con una migración nueva.
- **Logs estructurados** (JSON con `requestId`, nivel, módulo), revisables
  en el panel de Railway/Render.
- **Respaldos:** mientras se use el plan gratuito, `pg_dump` manual
  mensual y antes de cada migración importante. Se prueba al menos una
  vez restaurar un respaldo; un respaldo que nunca se ha restaurado no
  está comprobado.
- **Guía de operación mínima** (se escribe en la Fase 14): cómo desplegar,
  cómo restaurar un respaldo, cómo rotar secretos, y qué hacer si se
  filtra una clave.
- **Costos:** vigilar los límites de los planes gratuitos (almacenamiento
  de Supabase, horas de Railway/Render) antes de que se conviertan en
  una sorpresa.

---

## 12. Definición de "terminado" para cada fase

Una fase se cierra solo si cumple todo esto:

- [ ] Funciona y fue probada manualmente en el flujo principal.
- [ ] Tiene pruebas automatizadas donde aplique (obligatorias para
      dinero y autenticación).
- [ ] El CI pasa (lint, tipos, pruebas, auditoría, secretos).
- [ ] Respeta la estructura de este documento (capas y módulos).
- [ ] Cumple los controles de seguridad que le corresponden.
- [ ] El contrato OpenAPI está actualizado si cambió la API.
- [ ] Si hubo una decisión técnica nueva, tiene su ADR.
- [ ] CHANGELOG, ROADMAP y "¿Qué aprendí?" del PR al día.
