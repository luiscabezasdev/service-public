# Plan de Aprendizaje

Este proyecto es el vehículo; el objetivo es formarse como desarrollador
de software. Este plan define **qué aprender en cada fase, dónde
aprenderlo (documentación oficial) y cómo saber que ya se domina**, para
desarrollar el criterio técnico que se espera de un desarrollador
profesional: no solo hacer que algo funcione, sino poder explicar por qué
se hizo así y qué alternativas se descartaron.

Documentos relacionados: [ROADMAP](../ROADMAP.md) (qué se construye y el
avance) · [plan de trabajo](./plan-de-trabajo.md) (por qué en ese orden).

---

## 1. Método de estudio (para cada concepto nuevo)

1. **Leer la documentación oficial primero.** Los tutoriales y videos
   sirven de apoyo, pero la documentación oficial es la fuente que un
   desarrollador profesional consulta a diario. Acostumbrarse a leerla es
   parte del aprendizaje.
2. **Hacer un experimento mínimo aislado.** Antes de meter el concepto en
   el proyecto, probarlo en un archivo o proyecto de prueba de 20 líneas
   (ej. firmar y verificar un JWT en un script suelto).
3. **Aplicarlo al proyecto**, en la rama de la fase.
4. **Explicarlo con tus propias palabras** en la sección "¿Qué aprendí?"
   del Pull Request de la fase. Si no puedes explicarlo sin mirar el
   código, todavía no lo dominas.
5. **Responder las preguntas de criterio** de la fase (sección 3). Son el
   tipo de preguntas que aparecen en entrevistas técnicas.

**Regla sobre la IA (incluido Claude):** no se commitea código que no
puedas explicar línea por línea. Usar IA es una práctica profesional
normal; entregar código que no entiendes no lo es. Cuando Claude proponga
algo, pregunta "¿por qué así y no de otra forma?" hasta que tengas la
respuesta.

### Cómo leer documentación técnica

Casi toda la documentación seria se organiza en cuatro tipos de
contenido (modelo [Diátaxis](https://diataxis.fr/)). Saber distinguirlos
ahorra mucho tiempo:

| Tipo | Para qué sirve | Cuándo usarlo |
|---|---|---|
| **Tutorial** | Aprender haciendo, paso a paso | La primera vez con una tecnología |
| **Guía práctica (how-to)** | Resolver una tarea concreta | "¿Cómo conecto Prisma a Supabase?" |
| **Referencia (API)** | Consultar detalles exactos | "¿Qué parámetros recibe esta función?" |
| **Explicación** | Entender el porqué | "¿Por qué existen las migraciones?" |

### Ritmo

Sesiones cortas y constantes funcionan mejor que sesiones largas y
esporádicas, sobre todo con trabajo y universidad en paralelo. No hay
fechas fijas: una fase se cierra cuando funciona, está explicada en el
PR y las preguntas de criterio tienen respuesta, no cuando se acaba un
plazo.

---

## 2. Nivel 0 — Fundamentos transversales (antes de la Fase 0)

Son la base de todo el proyecto. No hace falta dominarlos a fondo antes
de empezar, pero sí tener lo básico, y se profundizan sobre la marcha.

| Tema | Documentación oficial | Qué leer primero |
|---|---|---|
| Git | [Pro Git (español)](https://git-scm.com/book/es/v2) | Capítulos 1-3 (fundamentos y ramas) |
| GitHub | [GitHub Docs (español)](https://docs.github.com/es) | Pull requests, protección de ramas |
| Mensajes de commit | [Conventional Commits](https://www.conventionalcommits.org/es/v1.0.0/) | La especificación completa (es corta) |
| HTTP y APIs | [MDN — HTTP](https://developer.mozilla.org/es/docs/Web/HTTP) | Métodos, códigos de estado, cabeceras |
| JavaScript | [MDN — Guía de JavaScript](https://developer.mozilla.org/es/docs/Web/JavaScript/Guide) | Funciones, objetos, promesas, async/await |
| TypeScript | [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html) | "The Basics", "Everyday Types", "Narrowing" |

**Sabes que dominas este nivel cuando puedes:**
- Crear una rama, hacer commits, subirla y abrir un PR sin consultar.
- Explicar la diferencia entre `GET`, `POST`, `PUT` y `DELETE`, y qué
  significan los códigos 200, 201, 400, 401, 403, 404 y 500.
- Explicar qué es una promesa y por qué se usa `await`.
- Explicar qué aporta TypeScript frente a JavaScript.

---

## 3. Aprendizaje por fase

Cada fase lista: conceptos, documentación, y **preguntas de criterio**
que debes poder responder al cerrarla.

### Bloque A — Backend

#### Fase 0 — Contrato de la API y modelo de datos
- **Conceptos:** diseño de API REST, modelado relacional, claves
  primarias y foráneas, relaciones 1:N y N:M, normalización básica.
- **Documentación:**
  - [Tutorial de PostgreSQL](https://www.postgresql.org/docs/current/tutorial.html)
    (capítulos 2 y 3)
  - [OWASP — REST Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html)
  - [Aprender OpenAPI (OpenAPI Initiative)](https://learn.openapis.org/)
  - [PostgreSQL — Roles y privilegios](https://www.postgresql.org/docs/current/user-manag.html)
  - [Supabase Docs](https://supabase.com/docs) (crear proyecto, conexión)
  - [Architecture Decision Records](https://adr.github.io/) y los ADR del
    proyecto en [docs/adr/](./adr/README.md)
- **Criterio:**
  - ¿Por qué la aplicación no se conecta con el usuario `postgres`?
  - ¿Qué restricción de la base de datos impide generar dos veces el
    cobro del mismo mes?
  - ¿Por qué `Reparto` y `CobroMensual` son tablas separadas y no una sola?
  - ¿Por qué el estado de pago al proveedor y el del inquilino no pueden
    ser el mismo campo?
  - ¿Qué pasaría si guardas solo el número de personas actual en vez del
    historial?

#### Fase 1 — Esqueleto del backend
- **Conceptos:** Node.js y su modelo asíncrono, Express (rutas y
  middleware), ORM, migraciones, variables de entorno, git hooks, linters.
- **Documentación:**
  - [Node.js — Learn](https://nodejs.org/learn)
  - [Express — Guía de rutas (español)](https://expressjs.com/es/5x/guide/routing/)
  - [Prisma Docs](https://www.prisma.io/docs) (empezar por la
    introducción y "Getting started" con PostgreSQL)
  - [Prisma con Supabase](https://supabase.com/docs/guides/database/prisma)
  - [The Twelve-Factor App](https://12factor.net/) (disponible en español,
    ver sobre todo "III. Configuración")
  - [Pino — logs estructurados](https://getpino.io/)
  - [GitHub Actions (español)](https://docs.github.com/es/actions)
  - [Configurar Dependabot](https://docs.github.com/es/code-security/dependabot/dependabot-version-updates/configuring-dependabot-version-updates)
  - [Arquitectura del proyecto](./arquitectura.md), secciones 3 y 4
- **Criterio:**
  - ¿Por qué el motor de reparto no puede importar Prisma ni Express?
  - ¿Qué ganas organizando por módulos de negocio y no por tipo de archivo?
  - ¿Qué debe pasar si el servidor arranca sin `JWT_SECRET`?
  - ¿Por qué la configuración va en variables de entorno y no en el código?
  - ¿Qué problema resuelven las migraciones frente a crear tablas a mano?
  - ¿Qué es un middleware y en qué orden se ejecutan?

#### Fase 2 — Autenticación
- **Conceptos:** hashing de contraseñas, JWT, expiración, rate limiting.
- **Documentación:**
  - [Introducción a JWT](https://jwt.io/introduction)
  - [OWASP — Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
  - [OWASP — Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
  - [Express — Buenas prácticas de seguridad](https://expressjs.com/en/advanced/best-practice-security.html)
- **Criterio:**
  - ¿Por qué bcrypt y no SHA-256 para contraseñas?
  - ¿Qué información NO debe ir dentro de un JWT y por qué?
  - Si alguien roba un token, ¿qué lo limita? ¿Qué harías para revocarlo?
  - ¿Por qué el mensaje de error de login no debe decir si falló el
    usuario o la contraseña?

#### Fase 3 — CRUD con validación
- **Conceptos:** validación en el servidor, manejo centralizado de
  errores, arquitectura por capas (rutas → controladores → servicios).
- **Documentación:**
  - [Zod](https://zod.dev)
  - [OWASP — Node.js Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Nodejs_Security_Cheat_Sheet.html)
- **Criterio:**
  - Si el formulario web ya valida, ¿por qué validar otra vez en el backend?
  - ¿Por qué no pasar el `body` del request directo a Prisma?
  - ¿Qué ganas separando controlador y servicio?

### Bloque B — Web

#### Fase 3.5 — Diseño de UI/UX
- **Documentación:** [Material Design 3](https://m3.material.io) (color,
  tipografía, componentes)
- **Criterio:** ¿Por qué diseñar el flujo de pantallas antes de programar?

#### Fases 4 y 5 — React
- **Conceptos:** componentes, props, estado, efectos, formularios,
  rutas protegidas, consumo de APIs, manejo de estados de carga y error.
- **Documentación:**
  - [React — Aprende (español)](https://es.react.dev/learn), en especial
    [Pensar en React](https://es.react.dev/learn/thinking-in-react)
  - [Vite — Guía](https://vite.dev/guide/)
  - [React Router](https://reactrouter.com)
  - [Material UI — Getting started](https://mui.com/material-ui/getting-started/)
- **Criterio:**
  - ¿Dónde guardas el token en el navegador y qué riesgo tiene cada opción?
  - ¿Por qué la protección de rutas en el frontend no es seguridad real?
  - ¿Qué debe ver el usuario mientras carga y cuando la API falla?

### Bloque C — Motor de reparto y cobro consolidado

#### Fase 6 — Patrón Strategy y tests
- **Conceptos:** interfaces, polimorfismo, patrón Strategy, testing
  unitario, casos límite, **manejo de dinero**.
- **Documentación:**
  - [Patrón Strategy — Refactoring.Guru (español)](https://refactoring.guru/es/design-patterns/strategy)
  - [TypeScript — Object Types e Interfaces](https://www.typescriptlang.org/docs/handbook/2/objects.html)
  - [Jest — Getting Started](https://jestjs.io/docs/getting-started)
  - [decimal.js](https://mikemcl.github.io/decimal.js/) (aritmética
    decimal exacta, sección de modos de redondeo)
  - [PostgreSQL — Tipos numéricos (NUMERIC)](https://www.postgresql.org/docs/current/datatype-numeric.html)
  - [ADR-0009 del proyecto](./adr/0009-dinero-decimal-exacto.md)
- **Criterio:**
  - ¿Por qué nunca usar números de punto flotante (`0.1 + 0.2`) para
    dinero? ¿Qué diferencia hay entre `NUMERIC` y `float`?
  - ¿Por qué se redondea una sola vez al final y no en cada paso del
    cálculo?
  - Si divides $10.000 entre 3 apartamentos quedan $3.333,33 × 3 =
    $9.999,99. ¿A quién se le asigna el centavo que falta y por qué esa
    regla es justa y repetible? La suma debe ser exactamente la factura.
  - ¿Qué casos límite probarías? (0 personas, medidor interno mayor que
    el externo, recibo en $0)
  - ¿Qué tendrías que cambiar para agregar un sexto tipo de reparto?

#### Fase 7 — Facturas, cobros y reportes
- **Conceptos:** transacciones de base de datos, consultas agregadas
  (`SUM`, `GROUP BY`), consistencia de datos.
- **Documentación:**
  - [Prisma — Transacciones](https://www.prisma.io/docs/orm/fundamentals/transactions)
  - [PostgreSQL — Funciones de agregación](https://www.postgresql.org/docs/current/tutorial-agg.html)
- **Criterio:**
  - Si al generar los 6 cobros del mes falla el cuarto, ¿qué debe pasar
    con los tres primeros?
  - ¿Por qué un cobro ya enviado al inquilino no debería cambiar después?

#### Fase 8 — Pantallas del flujo del dinero
- Refuerza el Bloque B. **Criterio:** ¿la pantalla de cobro se entiende
  sin explicación, desde el punto de vista del inquilino?

### Bloque D — Android

#### Fases 9 y 10 — Kotlin, Compose y consumo de la API
- **Conceptos:** Kotlin, corrutinas, arquitectura de apps (capas UI y
  datos, ViewModel, flujo de datos unidireccional), Retrofit, DataStore.
- **Documentación:**
  - [Kotlin Docs](https://kotlinlang.org/docs/home.html) y
    [Corrutinas](https://kotlinlang.org/docs/coroutines-overview.html)
  - [Android Basics with Compose](https://developer.android.com/courses/android-basics-compose/course)
    (curso oficial gratuito)
  - [Guía de arquitectura de apps](https://developer.android.com/topic/architecture)
  - [Retrofit](https://square.github.io/retrofit/)
  - [DataStore](https://developer.android.com/topic/libraries/architecture/datastore)
- **Criterio:**
  - ¿Por qué el ViewModel no debe conocer la UI?
  - ¿Qué pasa con una petición en curso si el usuario gira la pantalla?
  - ¿Por qué ya no se necesita Room en esta arquitectura?

### Bloque E — Funcionalidades finales y producción

#### Fases 11-13 — Alertas, archivos y exportación
- **Documentación:**
  - [Supabase Storage](https://supabase.com/docs/guides/storage)
  - [OWASP — File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)
- **Criterio:**
  - ¿Por qué validar el tipo y tamaño de un archivo en el servidor aunque
    el cliente ya lo filtre?
  - ¿Quién debe poder descargar un comprobante y cómo lo garantizas?

#### Fase 14 — Despliegue
- **Conceptos:** entornos (desarrollo y producción), CI/CD, HTTPS, CORS.
- **Documentación:**
  - [Railway Docs](https://docs.railway.com) o [Render Docs](https://render.com/docs)
  - [Vercel Docs](https://vercel.com/docs)
  - [Arquitectura del proyecto](./arquitectura.md), sección 11 (operación)
- **Criterio:**
  - Si hoy se borrara la base de datos de producción, ¿cuánto se pierde y
    cuánto tardas en restaurarla?
  - ¿Por qué desarrollo y producción usan bases de datos separadas?
  - ¿Qué debe pasar automáticamente antes de que un cambio llegue a
    producción?

---

## 4. Competencias transversales del desarrollador

Se practican en todas las fases, no en una sola:

| Competencia | Cómo se practica en este proyecto | Referencia |
|---|---|---|
| **Leer errores** | Leer el mensaje y el *stack trace* completos antes de buscar la solución o preguntar | — |
| **Depurar** | Usar el depurador de VS Code o Android Studio (puntos de interrupción) en vez de solo `console.log` | [VS Code Docs](https://code.visualstudio.com/docs) |
| **Revisar código** | Revisar tu propio PR como si fuera de otra persona, con la plantilla | [Google — Guía de code review](https://google.github.io/eng-practices/) |
| **Probar** | Tests del motor de reparto; probar casos límite, no solo el caso feliz | [Jest](https://jestjs.io/docs/getting-started) |
| **Seguridad** | Aplicar el documento de seguridad en cada fase | [OWASP Cheat Sheets](https://cheatsheetseries.owasp.org/) |
| **Documentar** | README, ROADMAP, CHANGELOG y "¿Qué aprendí?" al día | [Keep a Changelog](https://keepachangelog.com/es-ES/1.1.0/) |
| **Versionar** | Ramas, commits y releases según la guía | [control-de-versiones.md](./control-de-versiones.md) |
| **Estimar y planear** | Antes de cada fase, anotar cuánto crees que tardará; al cerrar, comparar con lo real | — |

---

## 5. Lecturas para el criterio a largo plazo (opcionales)

No son requisito para el proyecto, pero forman el criterio de un
desarrollador con experiencia. Se pueden leer por capítulos en paralelo:

- **The Pragmatic Programmer** (Hunt y Thomas): hábitos y mentalidad
  profesional. El más recomendable para empezar.
- **Refactoring** (Martin Fowler): cómo mejorar código existente sin
  romperlo.
- **Designing Data-Intensive Applications** (Martin Kleppmann): para más
  adelante, cuando interese entender bases de datos y sistemas a fondo.

---

## 6. Evidencia del aprendizaje (portafolio)

Al terminar, el repositorio debe demostrar por sí solo lo aprendido:

- Un historial de commits y PRs limpio que muestre la evolución del
  proyecto fase por fase.
- Releases (`v0.1.0` … `v1.0.0`) con su CHANGELOG.
- Tests del motor de reparto que corren en CI.
- Documentación de decisiones: reglas de negocio, seguridad y hosting.
- Un README con capturas (con datos ficticios) y cómo ejecutar el proyecto.

Ese conjunto es lo que un reclutador o un líder técnico revisa para
evaluar a un desarrollador junior: no solo que funcione, sino cómo se
trabajó.
