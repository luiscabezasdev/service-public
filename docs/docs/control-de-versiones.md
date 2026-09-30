# Control de Versiones — Cómo se trabaja con Git en este proyecto

Este documento define cómo se hacen commits, ramas, Pull Requests y
versiones. El objetivo es que el historial de Git cuente la historia del
proyecto con claridad, igual que en un equipo profesional, aunque hoy
trabajes solo.

---

## 1. Flujo de ramas: una rama por fase + Pull Request

```
main  ────●────────────●────────────●──────────►   (siempre funciona)
           \          /  \          /
feat/fase-1 ●──●──●──●    \        /
                    feat/fase-2 ●──●──●
```

- **`main` siempre debe estar en un estado que funciona.** Nunca se
  programa directo sobre `main`.
- **Cada fase del ROADMAP tiene su propia rama**, creada desde `main`:
  ```
  git switch main
  git pull
  git switch -c feat/fase-1-esqueleto-backend
  ```
- Mientras trabajas en la fase, haces commits pequeños en esa rama (ver
  sección 2).
- Al terminar la fase, abres un **Pull Request (PR)** en GitHub hacia
  `main`, lo revisas tú mismo con la plantilla de PR (sección 5) y lo
  fusionas (*merge*).
- Después del merge, borras la rama y vuelves a `main` para la siguiente
  fase.

**¿Por qué hacer PRs si trabajas solo?** El PR te obliga a revisar todo
lo que cambió antes de integrarlo, que es cuando se detectan un `.env`
colado, un `console.log` olvidado o un test roto. Además queda en GitHub
como registro de cada fase terminada, lo cual suma en el portafolio.

### Nombres de ramas

Formato: `tipo/descripcion-corta-en-minusculas`

| Prefijo | Cuándo | Ejemplo |
|---|---|---|
| `feat/` | Fase nueva o funcionalidad | `feat/fase-2-autenticacion` |
| `fix/` | Corregir un error | `fix/calculo-energia-pareja` |
| `docs/` | Solo documentación | `docs/contrato-api` |
| `refactor/` | Reorganizar código sin cambiar comportamiento | `refactor/servicios-reparto` |
| `chore/` | Configuración, dependencias, herramientas | `chore/configurar-eslint` |

Sin tildes, sin espacios, sin mayúsculas.

---

## 2. Commits: Conventional Commits

Se usa el estándar **Conventional Commits**
(https://www.conventionalcommits.org/es/), el formato más usado en la
industria.

### Formato

```
tipo(alcance): descripción corta en imperativo

Cuerpo opcional: explica el POR QUÉ del cambio, no el qué
(el qué ya se ve en el diff).

Refs: Fase 2
```

### Tipos

| Tipo | Uso |
|---|---|
| `feat` | Nueva funcionalidad (un endpoint, una pantalla, una estrategia de reparto) |
| `fix` | Corrección de un error |
| `docs` | Solo documentación |
| `test` | Agregar o corregir tests |
| `refactor` | Cambio de código que no altera el comportamiento |
| `style` | Formato (espacios, comas), sin cambio de lógica |
| `chore` | Configuración, dependencias, scripts |
| `perf` | Mejora de rendimiento |
| `ci` | Cambios en GitHub Actions u otra automatización |

### Alcances (scopes) de este proyecto

`backend`, `web`, `android`, `db` (esquema/migraciones), `docs`, `repo`
(configuración general del monorepo).

### Ejemplos buenos y malos

| ✅ Bueno | ❌ Malo | Por qué |
|---|---|---|
| `feat(backend): agrega endpoint de login con JWT` | `login` | No dice qué ni dónde |
| `fix(backend): corrige reparto de energía cuando el medidor interno supera al externo` | `arreglos varios` | "Varios" = varios commits mezclados |
| `test(backend): agrega casos de reparto por cabeza con recibo real de agosto` | `tests` | Sin contexto |
| `feat(db): agrega tabla reparto_apartamento con estado de pago` | `cambios en la bd` | No sirve para buscar en el historial |
| `chore(backend): instala helmet y express-rate-limit` | `update` | No dice qué se actualizó |

### Reglas

1. **Descripción en imperativo y minúsculas:** "agrega", "corrige",
   "elimina" (no "agregado" ni "agregué").
2. **Máximo ~72 caracteres** en la primera línea.
3. **Un commit = un cambio lógico.** Si en la descripción necesitas usar
   "y", probablemente son dos commits.
4. **Cada commit debe dejar el proyecto funcionando** (que compile y
   pasen los tests). Así cualquier punto del historial sirve para volver
   atrás si algo se rompe.
5. **No se hacen commits por hacer.** Se hace commit cuando se completa
   un cambio con sentido, no para cumplir una racha diaria ni para llenar
   la gráfica de contribuciones de GitHub. Commits artificiales ("arregla
   espacio", "actualiza README" sin cambio real) ensucian el historial y
   restan ante quien lo revisa con criterio. Lo que se mide es el avance
   real del proyecto, no la cantidad de commits por día.
6. **Push al cerrar cada sesión de trabajo**, aunque la fase no esté
   terminada: la rama queda respaldada en GitHub y el avance es visible.
7. **Cambio que rompe compatibilidad** (ej. cambiar la forma de un
   endpoint que ya usan web o Android): se marca con `!` y se explica:
   ```
   feat(backend)!: cambia formato de respuesta de /facturas

   BREAKING CHANGE: el campo `valor` ahora es `valorTotal`.
   web y android deben actualizarse en el mismo PR.
   ```
   En un monorepo, el cambio del backend y la actualización de los
   clientes van en el mismo PR. Esa es una de las ventajas del monorepo.

### Revisar antes de cada commit

```
git status              # ¿qué archivos cambiaron? ¿hay algo que no debería estar?
git diff                # revisar línea por línea lo que vas a commitear
git add ruta/archivo    # agregar archivos específicos, NO "git add ." a ciegas
git diff --staged       # confirmar lo que quedó preparado
git commit
```

`git add .` solo se usa después de revisar `git status` y confirmar que
todo lo listado debe entrar.

---

## 3. Versiones y etiquetas (Semantic Versioning)

Cada vez que se cierra un **bloque** del ROADMAP se crea una versión con
etiqueta (tag), siguiendo **SemVer** (`MAYOR.MENOR.PARCHE`):

| Hito | Versión |
|---|---|
| Bloque A cerrado (backend base con auth y CRUD) | `v0.1.0` |
| Bloque B cerrado (web con login y CRUD) | `v0.2.0` |
| Bloque C cerrado (motor de reparto y facturas) | `v0.3.0` |
| Bloque D cerrado (Android) | `v0.4.0` |
| Bloque E + despliegue en producción | `v1.0.0` |
| Corrección de un error después de una versión | `v0.3.1`, etc. |

Se usa `0.x` mientras el sistema no esté en uso real. `1.0.0` indica que
ya está desplegado y lo usas con datos reales.

```
git switch main
git pull
git tag -a v0.1.0 -m "Bloque A: backend con autenticación y CRUD"
git push origin v0.1.0
```

Después, en GitHub → *Releases* → *Draft a new release*, se elige el tag
y se pega la sección correspondiente del `CHANGELOG.md`.

---

## 4. CHANGELOG

El archivo [`CHANGELOG.md`](../CHANGELOG.md) en la raíz registra, por
versión, qué se agregó, cambió o corrigió, siguiendo el formato de
https://keepachangelog.com/es-ES/. Se actualiza **en el PR que cierra
cada fase**, no al final de todo, porque después ya no se recuerda qué
cambió.

---

## 5. Pull Requests

La plantilla está en `.github/pull_request_template.md`, y GitHub la
carga automáticamente al abrir un PR. Antes de hacer merge, todos los
puntos del checklist deben estar marcados, en especial los de seguridad.

**Tipo de merge recomendado:** *Squash and merge* para ramas con muchos
commits pequeños de prueba, o *Create a merge commit* si los commits de
la rama ya están limpios y quieres conservarlos. Elige uno y úsalo
siempre, por consistencia.

---

## 6. Configuración del repositorio en GitHub

Se hace una sola vez, al crear el repositorio:

**Settings → Branches → Add branch protection rule** (para `main`):
- ✅ Require a pull request before merging (no permite push directo a
  `main`, ni siquiera tuyo por accidente)
- ✅ Require status checks to pass (se activa en la Fase 1, cuando se
  configura el CI con GitHub Actions)

**Settings → Code security:**
- ✅ Secret scanning + **Push protection** (bloquea un push que contenga
  claves)
- ✅ Dependabot alerts + Dependabot security updates

---

## 7. Protecciones automáticas en tu máquina (a partir de la Fase 1)

Cuando exista `backend/package.json` se configuran *git hooks* con
**husky**. Son scripts que Git ejecuta solo antes de cada commit:

- **gitleaks** (o similar): revisa que el commit no contenga claves ni
  secretos. Es la primera barrera; *push protection* de GitHub es la
  segunda.
- **lint-staged + ESLint/Prettier**: formatea y revisa solo los archivos
  que vas a commitear.
- **commitlint**: rechaza el commit si el mensaje no cumple Conventional
  Commits.

Así las reglas de este documento no dependen de acordarse de ellas:
Git las aplica solo. Se configuran como parte de la Fase 1, explicando
qué hace cada herramienta según la constitución del proyecto.

---

## 8. Flujo completo de una fase (resumen)

```
1. git switch main && git pull
2. git switch -c feat/fase-N-descripcion
3. Trabajar → commits pequeños con Conventional Commits, solo cuando hay
   un cambio lógico completo; git push al cerrar cada sesión
4. Actualizar CHANGELOG.md y marcar [x] la fase en ROADMAP.md
5. git push -u origin feat/fase-N-descripcion
6. Abrir PR en GitHub → revisar checklist → merge
7. git switch main && git pull && git branch -d feat/fase-N-descripcion
8. Si cierra un bloque: crear tag de versión + Release en GitHub
```

---

## 9. Qué NUNCA entra al repositorio

Ver [seguridad-y-buenas-practicas.md](./seguridad-y-buenas-practicas.md),
sección "Repositorio público". En resumen: `.env`, contraseñas, claves de
API, cadenas de conexión, datos reales de inquilinos, recibos reales y
cualquier dato que identifique el edificio.
