---
description: Trae una tarea de Jira por su key, evalúa si está lista para SDD (gate de readiness) y arranca un change de OpenSpec. Intake vía skill jira-intake; generación vía subagente change-author. Opcionalmente escribe el avance de vuelta en Jira.
argument-hint: "<ISSUE-KEY> [--no-writeback]"
---

Convertí una tarea de Jira en un change de OpenSpec listo para implementar,
midiendo antes si la tarea está lo suficientemente clara para arrancar SDD.

Parámetros recibidos: $ARGUMENTS
- Primer argumento = **issue key** de Jira (ej. `CIDI-7446`). Obligatorio.
- `--no-writeback` (opcional) = NO ofrecer escribir de vuelta en Jira al terminar.

Usá el skill `jira-intake` como motor del intake. Corre en el hilo principal
(necesita las tools de Jira por MCP y AskUserQuestion) y hace:
0. **Preflight de token de adjuntos (gate inteligente):** chequea si hay token scoped
   para leer adjuntos. Si no, ofrece configurarlo (recomendado) o seguir solo-texto
   (degradado). Con token, lee el contenido de los adjuntos (PDF/mockups) vía `jira-read`
   — clave porque el spec suele estar en el diseño, no en el texto.
1. **Fetch** del issue por MCP (+ adjuntos si hay token).
2. **Mapeo** a un Change-Brief + **gate de readiness para SDD** (modelo en
   `references/sdd-readiness-model.md`): puntúa objetivo, alcance, criterios de
   aceptación, anclaje técnico y encaje con el dominio.
3. Si NO llega al umbral (85%), **surfacea lo que falta** (sobre todo criterios de
   aceptación) y lo resuelve con el usuario o recomendando completar el ticket —
   NO genera el spec con input turbio.
4. Al cruzar el umbral, muestra el Change-Brief y **espera confirmación**.

Tras la confirmación, delegá la generación de artefactos al subagente
`change-author` (corre `openspec new change`, ancla en el código real, genera
proposal/specs/design/tasks y valida con `openspec validate`).

Opcional (con confirmación, porque muta Jira): write-back al issue — transición a
"En Progreso" + comentario con el change-id, la rama `feat/<change-id>` y los
criterios de aceptación acordados.

Es el puente Jira → OpenSpec: el "prompt" pasa a ser la tarea definida en Jira,
pero solo si está lista para SDD.
