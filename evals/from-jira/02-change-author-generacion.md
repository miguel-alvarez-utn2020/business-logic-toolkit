# Eval from-jira — 02: change-author (generación desde un Brief, smoke objetivo)

- **Qué prueba:** que el subagente `change-author`, dado un Change-Brief confirmado, genere un change de
  OpenSpec **válido**, **grounded en código real**, y **reporte ambigüedades** en vez de resolverlas solo.
- **Criticidad:** media. La correctitud de formato la garantiza `openspec validate` (auto-chequeo), así que
  este eval es liviano y mayormente objetivo. Análogo al chequeo semi-manual del apply de `fix-dev`.
- **Modo:** muta el repo (crea `openspec/changes/<id>/`). Correr en worktree/rama descartable + limpiar.

## Setup
- App: `gob-cba-cidi-app-mobile`. OpenSpec inicializado. CLI `openspec` ≥ 1.5.
- Baseline de specs presente en `openspec/specs/` (incluida `authentication`).

## Input (Change-Brief confirmado, autocontenido)
- change-id: `boton-cerrar-sesion-color-error`
- jira: CIDI-7446 (https://ayigroup.atlassian.net/browse/CIDI-7446)
- why: identificar el logout como acción destructiva.
- what: el ítem "Cerrar sesión" usa el token `error` del theme vía `useTheme()`, en claro y oscuro.
- scenarios: (1) ítem en color error; (2) color desde token, no hardcode; (3) consistente claro/oscuro;
  (4) los demás ítems no cambian.
- non-goals: otros botones; paleta del theme; flujo de logout.
- capability objetivo: `authentication` (modificada — ADDED requirement de presentación).
- adaptación al stack: color por `useTheme()`/token, sin hardcode.

Pedir: generar el change desde este brief (invocar `change-author`).

## Expected

### Objetivos (verificables)
- [ ] `openspec validate boton-cerrar-sesion-color-error` → **valid**.
- [ ] `openspec status --change ...` → **4/4 artefactos** (proposal, specs, design, tasks).
- [ ] El spec delta usa `## ADDED Requirements` (NO `## Requirements`), `### Requirement:` y
      `#### Scenario:` con 4 hashtags; los 4 CA quedan como scenarios (conteo coincide).
- [ ] `proposal.md` incluye la línea de trazabilidad **Jira: CIDI-7446 (url)**.
- [ ] `tasks.md` usa checkboxes `- [ ] N.M` con prefijos, e incluye gates (`yarn typecheck`/`yarn lint`).

### De juicio (spot-check directo, read-only sobre los artefactos)
- [ ] **Grounding real:** el design apunta a archivos reales — `MenuItemListing.tsx` (color del título del
      ítem) y `src/core/theme/colors.ts` (token `error`) con referencia file:line.
- [ ] **Ambigüedad reportada:** detecta y REPORTA que "Cerrar sesión" existe en 2 lugares
      (`MoreScreen`/`MenuItemListing` y `ProfileMenuSheet`), sin resolverlo en silencio.
- [ ] **Convención sobre issue:** el color se toma del token vía `useTheme()`, no hardcode.
- [ ] No inventa comportamiento fuera del brief (respeta los non-goals).

## Runs / umbral
- 2 corridas. **2/2** en objetivos (validate + 4/4 + formato). Grounding y reporte de ambigüedad presentes en ambas.

## Limpieza
- `rm -rf openspec/changes/boton-cerrar-sesion-color-error` (o `git checkout`/descartar la rama) al terminar.
