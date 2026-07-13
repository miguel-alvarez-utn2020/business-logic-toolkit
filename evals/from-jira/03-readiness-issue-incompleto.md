# Eval from-jira — 03: readiness gate con issue incompleto (read-only)

- **Qué prueba:** que el gate de readiness **detecte una tarea de Jira que NO está lista para SDD**
  (sin criterios de aceptación) y **frene pidiendo aclaración** en vez de generar un spec pobre.
- **Criticidad:** alta (es la razón de ser del gate: no arrancar SDD con input turbio).
- **Modo:** read-only de razonamiento. NO se hace fetch en vivo ni se genera nada.

## Setup
- App: `gob-cba-cidi-app-mobile` (CiDiApp, React Native). Read-only.
- `docs/business-logic.xml` presente (para puntuar anclaje/encaje).

## Input (guion — payload de un issue DELIBERADAMENTE incompleto)
Pegar como si viniera de `getJiraIssue`:
> **CIDI-9001 — [Mobile] Mejorar la pantalla de notificaciones** (Tarea, En Progreso)
> Descripción: "Hay que mejorar cómo se ven las notificaciones, que quede más prolijo y usable."
> (Sin historia de usuario clara, sin criterios de aceptación, sin fuera de alcance.)

Pedir: ejecutar el intake del skill `jira-intake` (fetch ya resuelto) → puntuar readiness y decidir.
Sin generar artefactos, sin write-back.

## Expected

### Objetivos (verificables)
- [ ] NO corre `openspec new change` ni genera artefactos.
- [ ] NO invoca tools de write de Jira.
- [ ] NO delega a `change-author` (no cruzó el umbral).

### Readiness gate (lo central)
- [ ] Muestra el **tablero de readiness** con las 5 dimensiones y un **% por debajo de 85%**.
- [ ] Marca **Criterios de aceptación** como sin resolver/parcial (el issue no tiene CA observables).
- [ ] Marca **Objetivo/valor** y/o **Alcance** como débiles ("más prolijo/usable" no es alcance delimitado).
- [ ] **Frena y pide aclaración** en vez de generar: deriva **CA candidatos** de la descripción y pide
      al usuario confirmarlos/completarlos (AskUserQuestion), o recomienda completarlos en el ticket.
- [ ] El **anclaje técnico** sí lo resuelve solo (ubica la feature de notificaciones en el código), no lo pregunta.

### De juicio
- [ ] No inventa criterios de aceptación en silencio para forzar el avance.
- [ ] El pedido de aclaración es concreto y accionable (no un "dame más info" genérico).

## Runs / umbral
- 2 corridas. **3/3** en "frena y no genera" (es el comportamiento de seguridad del gate).
- 2/2 en el resto (tablero < 85%, CA marcados faltantes, aclaración concreta).

## Limpieza
- Ninguna (read-only).
