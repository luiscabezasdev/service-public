# Constitution — Sistema de Gestión de Arrendamiento

## 1. Propósito y contexto del proyecto

Sistema full-stack (backend API, web y Android) para gestionar un edificio
de 6 apartamentos en arriendo: apartamentos, inquilinos, mantenimientos,
canon, y el reparto de servicios públicos (agua, gas, energía, aseo,
internet/parabólica) entre inquilinos según reglas de prorrateo propias
del edificio, con reportes mensuales y generación de un recibo compartible
por apartamento.

Es un proyecto de **aprendizaje**: el objetivo no es solo tener el sistema
funcionando, sino que Luis entienda y pueda explicar cada parte del código,
incluida la seguridad de los datos que maneja (información financiera y
de inquilinos). Se construye en modalidad **pair programming**: Claude
propone código, explica el porqué de cada decisión, y pregunta antes de
avanzar a la siguiente parte. No se avanza en bloque sin que Luis entienda
el paso anterior.

Historia: el proyecto empezó como una app Android local (Room/SQLite) y
evolucionó a un sistema full-stack cuando surgió el requisito de acceso
del administrador desde cualquier lugar, tanto en app como en web. Ver
[reglas-de-negocio.md](./reglas-de-negocio.md) para el detalle completo.

## 2. Stack técnico

- **Backend:** Node.js + Express + TypeScript, PostgreSQL + Prisma (ORM),
  autenticación JWT
- **Web:** React + TypeScript + Material UI (MUI)
- **Android:** Kotlin + Jetpack Compose (Material 3) + Retrofit + DataStore
  (sin base de datos local — todo vía la API)
- **Arquitectura backend:** capas separadas (rutas → controladores →
  servicios → acceso a datos), motor de reparto de servicios como módulo
  independiente con el patrón Strategy
- **Base de datos en la nube:** ver
  [hosting-base-datos.md](./hosting-base-datos.md) para dónde vive y cómo
  se asegura

## 3. Alcance del MVP

Ver el detalle completo por fases en [ROADMAP.md](../ROADMAP.md). En
resumen: autenticación → gestión de apartamentos/inquilinos/mantenimientos
→ motor de reparto de servicios → registro de facturas y reportes →
Android como segundo cliente → alertas, comprobantes, recibo compartible,
despliegue.

No se agregan funciones fuera de lo definido en el ROADMAP sin que Luis
las pida explícitamente. Nada de "mientras tanto agrego X" no solicitado.

## 4. Reglas no negociables de código

- **Comentarios obligatorios**: todo bloque de lógica no trivial debe
  tener un comentario explicando el *por qué*, no solo el *qué*.
- **Un concepto nuevo por vez**: si se introduce algo que Luis no ha
  usado antes (Prisma, JWT, Strategy pattern, Retrofit, etc.), Claude debe:
  - explicarlo brevemente en sus propias palabras antes de escribir código,
  - señalar la documentación oficial correspondiente del
    [plan de aprendizaje](./plan-de-aprendizaje.md) (la documentación
    oficial va primero; videos o tutoriales solo como apoyo),
  - esperar confirmación de que se entendió antes de seguir.
- **Código que no se puede explicar no se commitea**, lo haya escrito Luis
  o lo haya propuesto Claude.
- **Nada de código "mágico"**: si una solución es difícil de explicar
  con lo que Luis sabe hasta ahora, se prefiere la versión más simple
  y explicable, aunque sea menos elegante.
- **La arquitectura se respeta**: el código sigue la estructura y las
  reglas de [arquitectura.md](./arquitectura.md) (módulos, capas, dinero
  con decimales exactos y cobro completo, reglas de negocio solo en el
  backend). Toda decisión
  técnica importante nueva se registra como ADR en [adr/](./adr/README.md)
  antes o junto con el código que la implementa. Una fase está terminada
  solo si cumple la definición de "terminado" de ese documento.
- **Seguridad no es opcional ni se pospone**: las prácticas de
  [seguridad-y-buenas-practicas.md](./seguridad-y-buenas-practicas.md) se
  aplican en la fase que corresponde, no se dejan para "después" — un
  login sin rate limiting o una consulta sin validar no se consideran
  "terminados" aunque funcionen.
- **Nombres en español para dominio, inglés para código técnico**:
  entidades y variables de negocio (`Factura`, `servicio`, `fechaVencimiento`)
  pueden ir en español si eso ayuda a la claridad; nombres de patrones
  técnicos (`Repository`, `Middleware`, `Service`) siguen la convención
  en inglés del framework.

## 5. Flujo de trabajo esperado con Claude

- Antes de escribir código de una función/endpoint/pantalla nueva, Claude
  resume en 2-3 líneas qué se va a hacer y por qué, y espera el visto bueno.
- Claude nunca reemplaza código existente sin explicar qué cambia y por qué.
- Cuando Luis no entiende algo, Claude reexplica con una analogía o
  ejemplo más simple antes de repetir el mismo código.
- Al cerrar cada fase del ROADMAP, se hace un resumen de lo aprendido: qué
  conceptos nuevos se tocaron y dónde quedaron en el código. Ese resumen va
  en la sección "¿Qué aprendí?" del Pull Request de la fase.
- Todo el trabajo con Git (ramas, commits, PRs, versiones) sigue
  [control-de-versiones.md](./control-de-versiones.md).

## 6. Fuera de alcance (por ahora)

- Multi-rol o multi-administrador (hoy hay un solo usuario administrador)
- Publicación en tiendas (Play Store / despliegue público más allá de lo
  necesario para que Luis y, si aplica, los inquilinos reciban recibos)
- Cualquier funcionalidad no listada en el ROADMAP

Estos puntos solo se agregan si Luis decide explícitamente ampliar el
alcance más adelante.
