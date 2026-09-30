# Seguridad y Buenas Prácticas de Desarrollo

Este documento aplica el **OWASP API Security Top 10 (2023)** y el
**OWASP Top 10:2025** (aplicaciones web) directamente a este proyecto.
No es una lista genérica — cada punto dice qué significa en tu stack
(Node/Express/TypeScript/Prisma/PostgreSQL en el backend, React en la
web, Kotlin en Android).

**Nota aparte:** existe también un "OWASP Top 10 para aplicaciones LLM",
que trata riesgos de sistemas que usan modelos de lenguaje (inyección de
prompts, fuga de datos por IA, etc.). Ese documento **no aplica a este
proyecto** — no tiene ningún componente de IA. Se menciona aquí solo para
que quede explícito por qué no se usa como referencia.

Como tu backend es ante todo una **API REST** consumida por dos clientes
(web y Android), el **API Security Top 10** es la referencia más directa;
el Top 10 web general aplica sobre todo a la parte de `web/`.

---

## OWASP API Security Top 10 (2023) — aplicado al backend

### API1 — Broken Object Level Authorization
**Qué es:** un endpoint recibe un ID (ej. `/apartamentos/5`) y no verifica
si quien pide ese recurso tiene permiso sobre él.
**En este proyecto:** hoy hay un solo administrador, así que el riesgo es
bajo — pero se implementa igual desde el día uno, porque es la base para
si algún día se agregan más usuarios (ej. un segundo administrador).
**Práctica:** todo endpoint que reciba un ID de recurso valida contra el
usuario autenticado del token, nunca confía en que "si llegó el ID, es
válido". Se hace en el middleware/servicio, no repetido a mano en cada ruta.

### API2 — Broken Authentication
**En este proyecto:** el login (Fase 2 del backend).
**Prácticas:**
- Contraseña con `bcrypt` (nunca texto plano ni hash débil como MD5/SHA1)
- JWT con expiración corta (ej. 1-2 horas) + mecanismo de refresh token
  si se necesita sesión larga, en vez de un token que nunca expira
- Límite de intentos de login (rate limiting) para frenar fuerza bruta
- El secreto de firma del JWT (`JWT_SECRET`) vive en variable de entorno,
  nunca en el código

### API3 — Broken Object Property Level Authorization
**En este proyecto:** que un endpoint no devuelva u obligue a aceptar más
campos de los que debería. Ej.: el formulario de "reportar personas"
(que usa el inquilino, según la idea original) no debería poder tocar el
campo `canon` del apartamento aunque alguien manipule el request.
**Práctica:** cada endpoint define explícitamente qué campos acepta de
entrada (no un `...body` genérico hacia Prisma) y qué campos devuelve.

### API4 — Unrestricted Resource Consumption
**En este proyecto:** con solo 6 apartamentos el volumen es bajo, pero se
protege igual la API contra abuso (ej. un script que golpee el login mil
veces, o pida reportes en bucle).
**Práctica:** rate limiting general en Express (`express-rate-limit`),
paginación en endpoints que devuelven listas (aunque hoy sean pocas filas).

### API5 — Broken Function Level Authorization
**En este proyecto:** como hay un solo rol (administrador), hoy no aplica
en forma compleja — pero se deja la estructura lista (middleware de auth
que se pueda extender a roles) para cuando/si se agregue otro tipo de
usuario.

### API6 — Unrestricted Access to Sensitive Business Flows
**En este proyecto:** el flujo de registrar una factura o modificar el
canon de un apto son "flujos sensibles" — no deberían poder ejecutarse
sin autenticación ni de forma automatizada sin control.
**Práctica:** todos los endpoints de escritura (POST/PUT/DELETE) exigen
JWT válido, sin excepción.

### API7 — Server Side Request Forgery (SSRF)
**En este proyecto:** aplica si en el futuro el backend llega a
descargar contenido desde una URL que el usuario proporcione (por
ejemplo, si se agrega una integración externa). Hoy no hay ese caso, pero
queda documentado para cuando se toque la Fase 12 (adjuntar comprobante),
si esa subida involucra URLs externas.

### API8 — Security Misconfiguration
**Prácticas:**
- CORS configurado explícitamente (solo el dominio de `web/` permitido,
  no `*`)
- Headers de seguridad HTTP con `helmet` en Express
- Nunca correr en producción con mensajes de error detallados expuestos
  al cliente (stack traces solo en logs del servidor, no en la respuesta)
- Variables de entorno (`.env`) fuera de git (ya está en `.gitignore`)

### API9 — Improper Inventory Management
**Práctica:** el contrato de la API (Fase 0) se mantiene como documento
vivo — cada endpoint nuevo se agrega ahí. Evita tener rutas "fantasma"
que nadie recuerda que existen o que quedaron de pruebas.

### API10 — Unsafe Consumption of APIs
**En este proyecto:** aplica si el backend llega a consumir una API de
terceros (ej. para enviar notificaciones). Se trata su respuesta como
dato no confiable (se valida, no se asume que siempre viene bien
formada).

---

## OWASP Top 10:2025 — aplicado a `web/` (y en general)

### A01 — Broken Access Control
Cubierto arriba (API1, API5). En `web/`: las rutas protegidas (Dashboard,
Apartamentos, etc.) verifican el token en el cliente **solo como UX**
(redirigir si no hay sesión) — la seguridad real siempre vive en el
backend, nunca se confía en el cliente.

### A02 — Security Misconfiguration
Cubierto arriba (API8).

### A03 — Software Supply Chain Failures
**Práctica:** revisar `npm audit` periódicamente en `backend/` y `web/`,
mantener dependencias actualizadas, no instalar paquetes sin verificar
que tengan mantenimiento activo y buena reputación.

### A04 — Cryptographic Failures
**Prácticas:**
- HTTPS obligatorio en producción (Fase 14, despliegue)
- Contraseñas nunca en texto plano (ver API2)
- Conexión a la base de datos siempre por SSL (`sslmode=require`),
  cadena de conexión nunca commiteada — ver
  [hosting-base-datos.md](./hosting-base-datos.md) para el detalle
  completo de dónde vive la base de datos y cómo se protege esa conexión
- Datos sensibles (si se llega a guardar algo como número de documento
  de un inquilino) considerados para cifrado en reposo si el hosting lo
  permite

### A05 — Injection
**En este proyecto:** además del ORM, el backend se conecta con un rol
de base de datos de mínimo privilegio (ver
[arquitectura.md](./arquitectura.md), sección 5), así que incluso una
consulta mal construida no podría borrar tablas. El uso de **Prisma como ORM** (definido desde la
Fase 1) previene inyección SQL por diseño, porque las consultas se arman
de forma parametrizada — nunca se concatena texto del usuario directo en
una consulta SQL cruda.
**Práctica adicional:** validar y sanear toda entrada de usuario en el
backend con una librería de validación (ej. `zod`), no solo confiar en la
validación del formulario en `web/` o `android/` — la validación del
cliente es para UX, la del servidor es la que protege de verdad.

### A06 — Insecure Design
**En este proyecto:** ya se está aplicando desde el inicio — diseñar el
contrato de la API (Fase 0) y las reglas de negocio antes de programar,
en vez de improvisar sobre la marcha, es en sí mismo una práctica de
diseño seguro.

### A07 — Authentication Failures
Cubierto arriba (API2). En Android: el token JWT se guarda con
**DataStore cifrado** (no en `SharedPreferences` plano), como ya estaba
planeado en la Fase 9.

### A08 — Software or Data Integrity Failures
**Práctica:** verificar integridad de dependencias (`package-lock.json`
siempre commiteado, nunca `--force` sin revisar), y en el flujo de CI/CD
(cuando exista) no desplegar sin que pasen los tests del motor de
reparto (Fase 6).

### A09 — Security Logging and Alerting Failures
**Práctica:** loggear intentos de login fallidos, errores 500, y accesos
denegados — sin loggear contraseñas ni tokens completos. Útil también
para depurar durante el aprendizaje, no solo para seguridad.

### A10 — Mishandling of Exceptional Conditions
**Práctica:** todo endpoint maneja sus errores explícitamente (try/catch
+ middleware de manejo de errores centralizado en Express), en vez de
dejar que una excepción no controlada tumbe el servidor o filtre detalles
internos en la respuesta.

---

## Repositorio público en GitHub — qué es seguro y qué no

**Principio:** el sistema no debe depender de que nadie vea el código.
Lo protegen tres secretos: la contraseña del administrador (guardada con
bcrypt), el `JWT_SECRET` y la cadena de conexión a Supabase. Si esos
secretos nunca llegan al repositorio, el código puede ser público sin
comprometer la seguridad.

### Nunca entra al repositorio
- Archivos `.env` (ni `.env.local`, ni `.env.production`). Solo se
  commitea `.env.example`, con los **nombres** de las variables y valores
  vacíos o ficticios.
- Contraseñas, `JWT_SECRET`, claves de Supabase (en especial la
  `service_role`), cadenas de conexión (`DATABASE_URL`).
- Datos reales: nombres de inquilinos, valores de canon, números de
  contrato o de cuenta de servicios públicos, recibos o capturas reales.
- Información que ubique el edificio: dirección, barrio, fotos.
- Respaldos de la base de datos (`*.sql`, `*.dump`). Ya están en
  `.gitignore`.

### Datos de prueba
Todos los datos de ejemplo (seeds de Prisma, tests del motor de reparto,
capturas para el README) son **inventados**. Para los tests del reparto
se usan valores con la misma *forma* que un recibo real (ej. 78 kWh,
$35.012), sin datos que identifiquen a nadie.

### Si un secreto llega a GitHub por error
1. **Cambiarlo de inmediato** (nueva contraseña de base de datos en
   Supabase, nuevo `JWT_SECRET`). Este es el paso que importa.
2. Borrar el archivo en un commit nuevo **no basta**: el secreto sigue
   en el historial de Git y cualquiera puede verlo. Por eso siempre se
   cambia, aunque el repo sea privado.
3. Revisar en Supabase si hubo accesos sospechosos.

### Barreras automáticas
- `.gitignore` en la raíz (ya existe).
- Hook de pre-commit con gitleaks, desde la Fase 1 (ver
  [control-de-versiones.md](./control-de-versiones.md), sección 7).
- *Secret scanning* + *push protection* activados en GitHub.
- Dependabot para alertas de dependencias vulnerables (A03).

### ¿Público o privado?
Público mientras se desarrolla con datos ficticios: suma al portafolio y
las protecciones de GitHub son gratuitas. Se puede pasar a privado en
cualquier momento desde *Settings → General → Danger Zone* sin perder
historial ni configuración.

---

## Validación de datos — regla general del proyecto

Regla simple que se aplica en las tres piezas, pero en capas distintas:

1. **Cliente (web/Android):** valida para dar feedback inmediato al
   usuario (campo requerido, formato de fecha, número positivo). Es UX,
   no seguridad.
2. **Backend (API):** valida **todo** de nuevo, sin excepción, porque un
   request puede llegar sin pasar por el cliente (ej. con Postman o un
   script). Esta es la validación que realmente protege los datos.
3. **Base de datos (Prisma/PostgreSQL):** restricciones a nivel de
   esquema (`NOT NULL`, tipos, claves foráneas) como última línea de
   defensa.

## Cuándo se aplica esto en el plan

No es una fase aparte — estas prácticas se aplican **dentro** de las
fases ya definidas en el [ROADMAP](../ROADMAP.md):
- Fase 2 (autenticación): API2, A04, A07
- Fase 3 y 7 (CRUD y facturas): API1, API3, A05, validación en 3 capas
- Fase 1 (esqueleto backend): A02, A08 (CORS, helmet, .env, lockfile)
- Fase 9 (login Android): A07 (DataStore cifrado)
- Fase 14 (despliegue): A04 (HTTPS), API8 (config de producción)

Se revisa este documento al empezar cada una de esas fases, como parte
del "resumen de 2-3 líneas antes de escribir código" que ya define la
constitución del proyecto.
