---
name: intake-fix
description: >
  Conduce una entrevista estructurada para arreglar un bug: traduce un reporte
  (posiblemente de alguien no técnico, ej. QA) en un Fix-Brief preciso y trazable,
  midiendo la claridad con un gate por dimensiones. Al confirmar el brief, lo
  entrega al subagente dev. Úsese cuando se quiera arreglar un bug de forma guiada
  y de alta precisión, no con un prompt suelto.
argument-hint: "[descripción breve del bug, opcional]"
allowed-tools: Read, Grep, Glob, AskUserQuestion
---

Este es el motor del comando `/fix`. La entrevista corre en el hilo principal
(necesita AskUserQuestion). El subagente `fix-dev` solo ejecuta al final.

Principio rector: **el agente carga el peso técnico; el humano solo aporta la
visión del problema.** Nunca le pidas a un no-técnico que diga "en qué archivo
está" — esa dimensión la resolvés vos contra el código.

## Fase 0: Cargar contexto

1. Ubicá el documento de lógica de negocio en `docs/` (xml o md). Es el contexto
   que da precisión al fix: contiene entidades, reglas de negocio (con `BR-id` y
   `source`) y flujos.
2. Si NO existe, PARÁ. Recomendá correr `/business-logic` primero y explicá que
   sin ese contexto el fix pierde precisión. No sigas a ciegas.
3. Si existe, cargalo. Lo vas a usar para mapear el síntoma a una regla/flujo y a
   un `source` candidato.

## Fase 1: Entrevista con gate de claridad

El modelo de claridad (dimensiones, pesos, scoring y banco de preguntas) está en
`references/clarity-model.md`. Seguilo.

Reglas de la entrevista:
1. Mantené un **tablero de claridad** sobre las dimensiones del modelo. Arranca en
   0% (o con lo que aporte $ARGUMENTS).
2. Preguntá de a una cosa por vez. Para todo lo que el humano pueda responder,
   usá **AskUserQuestion en opción múltiple** — un QA elige, no redacta.
3. Las dimensiones marcadas como "resuelve el agente" (localización técnica) las
   subís VOS: buscá en el código (Grep/Glob/Read) y en el doc de negocio el
   `source` candidato. No se las preguntás al humano.
4. Después de cada respuesta, **recomputá la claridad y mostrá el checklist + %**,
   indicando qué dimensiones faltan. Formato en `references/clarity-model.md`.
5. No interrogues por inercia: en cuanto cruzás el umbral "listo", pasá a la Fase 2.

## Fase 2: Gate "listo" (~85%)

Cuando la claridad cruza el umbral "listo" (~85%):
1. DEJÁ de preguntar solo.
2. Mostrá el **borrador del Fix-Brief** (esquema en `references/fix-brief-schema.md`).
3. Evaluá el **riesgo/complejidad** del bug (toca una regla de negocio crítica,
   afecta datos, varios flujos) y **recomendá** si conviene afinar o no:
   - bug simple y aislado → recomendá proceder (afinar sería ruido).
   - bug riesgoso / toca BR crítica / múltiples flujos → recomendá afinar a 95%+.
4. Ofrecé con AskUserQuestion: **Proceder** | **Afinar a 95%+**.

## Fase 3: Afinar (opcional)

Si el usuario elige afinar, hacé preguntas de mayor resolución (sección "Afinado"
del banco en `references/clarity-model.md`): casos borde, variaciones de datos,
flujos relacionados, riesgo de regresión. Estas dimensiones suben del ~90 al 100.
Volvé a mostrar el checklist + %. Cuando el usuario quede conforme (o se llegue al
techo), pasá a la Fase 4.

## Fase 4: Fix-Brief final + confirmación

1. Construí el Fix-Brief final según `references/fix-brief-schema.md`. Cada bug
   debe linkear, cuando exista, la regla de negocio afectada por su `BR-id` y su
   `source` (trazabilidad al doc de negocio).
2. Mostralo completo y pedí **confirmación explícita** del usuario.
3. Guardalo en `docs/fixes/` con un nombre legible (ej. `fix-orden-sin-stock.md`).

## Fase 5: Handoff al dev

Solo tras la confirmación:
1. Delegá en el subagente `fix-dev`, pasándole el Fix-Brief completo como contexto.
2. Mostrá el cambio propuesto (diff/preview) que devuelve el dev.
3. NO encadenes verificación automática en esta versión (queda para una iteración
   futura con el subagente QA-Playwright).

## Límites (qué NO hacer)
- No le preguntes detalles técnicos (archivos, funciones, frameworks) al humano.
- No inventes claridad: el % se deriva del modelo, nunca es un número "a sentimiento".
- No toques código durante la entrevista. La ejecución es del subagente `fix-dev`.
- No dispares el fix sin confirmación explícita del Fix-Brief.
