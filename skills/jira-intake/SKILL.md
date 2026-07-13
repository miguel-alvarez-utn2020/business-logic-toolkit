---
name: jira-intake
description: >
  Motor del comando /from-jira. Trae una tarea de Jira por MCP, la mapea a un
  Change-Brief para OpenSpec y mide con un gate de readiness si la tarea está lo
  suficientemente clara para arrancar SDD. Si no llega al umbral, surfacea lo que
  falta (sobre todo criterios de aceptación) y lo resuelve con el usuario o
  recomendando completar el ticket, antes de delegar la generación al subagente
  change-author. Úsese como paso de intake del flujo Jira → OpenSpec.
argument-hint: "<ISSUE-KEY> [--no-writeback]"
allowed-tools: Read, Grep, Glob, AskUserQuestion
---

Este es el motor del comando `/from-jira`. El intake corre en el hilo principal
(necesita las tools de Jira por MCP y AskUserQuestion). El subagente `change-author`
solo ejecuta la generación al final.

Principio rector: **el "prompt" es la tarea de Jira, no un texto suelto.** Antes de
convertirla en spec, verificá que la tarea esté lista para SDD. Un spec generado
sobre un issue turbio es un spec pobre.

## Fase 0: Precondiciones

1. Connector de Atlassian autorizado (tools `getJiraIssue`/`getAccessibleAtlassianResources`).
   Si no están, PARÁ y avisá que hay que conectarlo desde la config de connectors de claude.ai.
2. OpenSpec inicializado (`openspec/` con schema `spec-driven`). Si no, PARÁ y avisá.
3. Ubicá `docs/business-logic.xml` si existe: da el contexto de dominio para puntuar
   "encaje con el dominio" y "anclaje técnico". Si no existe, se puede seguir, pero
   el anclaje se resuelve solo contra el código.

## Fase 1: Fetch (MCP)

- Resolvé el `cloudId` con `getAccessibleAtlassianResources`.
- Traé el issue con `getJiraIssue` (fields: summary, description, status, issuetype,
  priority, labels, components, parent, comment; `responseContentFormat: markdown`).

## Fase 2: Mapeo + gate de readiness

El modelo (dimensiones, pesos, scoring, formato del tablero, qué hacer si no llega)
está en `references/sdd-readiness-model.md`. Seguilo.

1. Derivá el borrador del Change-Brief desde el issue (change-id semántico, why, what,
   scenarios candidatos desde los CA, non-goals, capability objetivo, adaptación al stack).
2. **Puntuá el issue** contra las 5 dimensiones del modelo:
   - Las de negocio (objetivo, alcance, criterios de aceptación) salen del TEXTO del ticket.
   - Las técnicas (anclaje, encaje con dominio) las resolvés VOS contra el código y
     `business-logic.xml` — no se preguntan.
3. **Mostrá el tablero de readiness + %** (formato del modelo), indicando qué falta.

## Fase 3: Resolver o confirmar (umbral 85%)

**Si readiness < 85%:** NO arranques SDD. Resolvé lo que falta según el modelo:
- Criterios de aceptación ausentes/vagos → derivá CA candidatos de la descripción y
  pedí al usuario que los confirme/complete con AskUserQuestion. Si no puede,
  recomendá completarlos en el ticket (opcional: dejar comentario en Jira con confirmación).
- Objetivo/alcance turbios → pedí la aclaración puntual (AskUserQuestion) o recomendá afinar en Jira.
- Encaje bajo (fuera de dominio) → avisá "es capability nueva" y pedí confirmación de que es intencional.
- Recomputá y volvé a mostrar el tablero. Pedí de a una dimensión; no interrogues en loop.

**Cuando readiness ≥ 85%:**
1. Construí el **Change-Brief COMPLETO** (change-id, jira key+url, why, what, scenarios,
   non-goals, capability objetivo, adaptación al stack), incorporando lo aclarado.
2. Mostralo y **esperá confirmación** del usuario (gate de confirmación).

## Fase 4: Handoff a change-author

Solo tras la confirmación:
- Delegá en el subagente `change-author` pasándole el Change-Brief completo.
- El subagente corre `openspec new change`, localiza el código real, genera
  proposal/specs/design/tasks y valida con `openspec validate`. Devuelve resumen +
  ambigüedades. NO reimplementes la generación acá.

## Fase 5: Write-back a Jira (opcional, requiere confirmación)

Si NO se pasó `--no-writeback`, ofrecé (y confirmá antes, porque muta Jira):
- Transicionar el issue a "En Progreso" si no lo está.
- Comentar con: change-id, rama sugerida `feat/<change-id>`, resumen de artefactos,
  y (si se derivaron CA en la Fase 3) los criterios de aceptación acordados.
NUNCA hagas write-back sin confirmación explícita.

## Límites (qué NO hacer)
- No generes el spec con un issue por debajo del umbral: primero resolvé la claridad.
- No inventes criterios de aceptación en silencio: derivalos y confirmalos.
- No toques código durante el intake. La generación es del subagente `change-author`.
- No hagas write-back sin confirmación.
