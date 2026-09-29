## ¿Qué hace este PR?

<!-- 1-3 líneas. Ej: "Implementa la Fase 2: login con JWT y rate limiting" -->

**Fase del ROADMAP:** Fase N
**Piezas afectadas:** [ ] backend [ ] web [ ] android [ ] db [ ] docs

## ¿Qué aprendí?

<!-- Conceptos nuevos de esta fase y dónde quedaron en el código -->

## Checklist antes de hacer merge

### Funciona

- [ ] El proyecto compila y arranca sin errores
- [ ] El CI pasa (lint, tipos, pruebas, auditoría, secretos)
- [ ] Hay pruebas automatizadas donde aplica (obligatorias para dinero y autenticación)
- [ ] Probé manualmente el flujo principal de esta fase

### Arquitectura (ver docs/arquitectura.md)

- [ ] El código respeta los módulos y capas (sin lógica de negocio en controladores ni en los clientes)
- [ ] El dinero se maneja con decimales exactos a través de `money.ts` (nunca `number`), y la suma de repartos es igual a la factura
- [ ] Los cambios de esquema son migraciones nuevas (no se editan migraciones aplicadas)
- [ ] Si cambió la API: `docs/api/openapi.yaml` actualizado
- [ ] Si hubo una decisión técnica nueva: ADR agregado en `docs/adr/`

### Seguridad (ver docs/seguridad-y-buenas-practicas.md)

- [ ] No hay `.env`, contraseñas, claves ni cadenas de conexión en el diff
- [ ] No hay datos reales (inquilinos, cánones, recibos, dirección)
- [ ] Toda entrada de usuario nueva se valida en el backend
- [ ] Los endpoints nuevos exigen autenticación (salvo `/login`)
- [ ] Los errores no exponen detalles internos al cliente

### Orden

- [ ] Los commits siguen Conventional Commits
- [ ] El código nuevo tiene comentarios del _por qué_ (constitución)
- [ ] `CHANGELOG.md` actualizado
- [ ] Fase marcada `[x]` en `ROADMAP.md`
- [ ] Si cambió la API: web y android actualizados en este mismo PR
