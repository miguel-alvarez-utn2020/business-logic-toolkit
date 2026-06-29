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
0. **Arranque neutro (primera pregunta).** Si NO hay un síntoma inicial claro en
   $ARGUMENTS, la primera pregunta debe ser ABIERTA y agnóstica del área: NO
   asumas de qué parte de la app habla el usuario. Ofrecé como opciones las
   ÁREAS/flujos detectados en el documento de negocio (p.ej. envíos, devoluciones,
   coordinación, usuarios/roles, login…) más "Otra", y dejá que el usuario elija.
   Recién con esa respuesta empezás a acotar. Si $ARGUMENTS YA trae un síntoma con
   área, no la des por sentada: confirmala con el usuario en la primera pregunta
   en vez de afirmarla ("¿el problema es en …?"), por si el argumento venía sesgado.
1. Mantené un **tablero de claridad** sobre las dimensiones del modelo. Arranca en
   0% (o con lo que aporte $ARGUMENTS).
2. Preguntá de a una cosa por vez. Para todo lo que el humano pueda responder,
   usá **AskUserQuestion en opción múltiple** — un QA elige, no redacta.
3. Las dimensiones marcadas como "resuelve el agente" (localización técnica) las
   subís VOS: buscá en el código (Grep/Glob/Read) y en el doc de negocio el
   `source` candidato. No se las preguntás al humano.
4. **Área reportada ≠ localización en código.** El área/pantalla que nombró el
   humano es un INDICIO falible. Si encontrás un defecto certero que reproduce el
   síntoma central, la Localización cuenta como resuelta AUNQUE esté en otra
   pantalla — el mismatch es una observación para el Fix-Brief, no un bloqueo.
   **Regla de convergencia (anti-loop):** re-preguntá la pantalla UNA sola vez
   como máximo; si el humano insiste, NO investigues en círculos ni inventes un
   defecto inexistente. Presentá el defecto real ("está en X, no en la pantalla Y
   que nombraste, pero X explica el síntoma") y avanzá. (Detalle en
   `references/clarity-model.md` → "Área reportada ≠ localización en código".)
5. Después de cada respuesta, **recomputá la claridad y mostrá el checklist + %**,
   indicando qué dimensiones faltan. Formato en `references/clarity-model.md`.
6. No interrogues por inercia: en cuanto cruzás el umbral "listo", pasá a la Fase 2.

## Fase 2: Gate "listo" + confirmación (~85%)

ANTES de abrir el gate, aplicá el **gate de reproducción** (detalle en
`references/clarity-model.md`): la Localización solo cuenta como resuelta si el
defecto encontrado reproduce el síntoma **bajo los hechos que el usuario afirmó**.
Si para que calce tenés que asumir algo que el usuario no dijo —o que contradice
una de sus respuestas— NO es una localización: surfacéale la contradicción, pedí
el dato que la resuelve, y NO procedas a un fix a ciegas. Un cambio que por tu
propio análisis no resuelve lo reportado NO es un fix (anotalo como observación /
posible `/fix` aparte si es una fragilidad real distinta).
Ante una contradicción, **convergé rápido**: "no se reproduce contra este código"
+ UN chequeo concreto del lado del usuario, mencionando la causa ambiental (build
viejo/cache/deploy) solo como posibilidad breve. NO te vayas a la madriguera de
arqueología de git multi-rama ni teorías de deploy salvo que el usuario lo confirme
y pida cavar. Tope: una pregunta de desempate más; si no cierra, entregá el intake
"no-reproducible" y listo.

Cuando la claridad cruza el umbral "listo" (~85%) y pasó el gate de reproducción:
1. DEJÁ de preguntar solo.
2. Construí y mostrá el **Fix-Brief COMPLETO** (no un borrador), según
   `references/fix-brief-schema.md`. Cada bug debe linkear, cuando exista, la regla
   de negocio afectada por su `BR-id` y su `source` (trazabilidad al doc).
3. Evaluá el **riesgo/complejidad** del bug (toca una regla de negocio crítica,
   afecta datos, varios flujos) y **recomendá** si conviene afinar o no:
   - bug simple y aislado → recomendá proceder (afinar sería ruido).
   - bug riesgoso / toca BR crítica / múltiples flujos → recomendá afinar a 95%+.
4. Ofrecé con AskUserQuestion UNA sola decisión (NO doble confirmación):
   **Proceder (confirmar y aplicar)** | **Afinar a 95%+**.
   Elegir "Proceder" ES la confirmación del Fix-Brief — no vuelvas a pedir
   confirmación después.

## Fase 3: Afinar (opcional)

Si el usuario elige afinar, hacé preguntas de mayor resolución (sección "Afinado"
del banco en `references/clarity-model.md`): casos borde, variaciones de datos,
flujos relacionados, riesgo de regresión. Estas dimensiones suben del ~90 al 100.
Volvé a mostrar el checklist + %, actualizá el Fix-Brief y volvé al gate de la
Fase 2 (misma decisión única).

## Fase 4: Handoff al dev

Solo tras "Proceder":
1. Guardá el Fix-Brief en `docs/fixes/` con un nombre legible (ej.
   `fix-orden-sin-stock.md`).
2. Delegá en el subagente `fix-dev`, pasándole el Fix-Brief completo como contexto.
   Si durante la localización encontraste una implementación análoga que sirve de
   patrón a seguir (o si existe `docs/conventions.md`), pasásela como referencia
   para que el fix imite el estilo del proyecto y no improvise.
3. Mostrá el cambio aplicado (diff/preview) que devuelve el dev.
4. **Decile al usuario DÓNDE verificar** (importante): indicá la pantalla/flujo
   real desde donde se ejercita el código tocado. Si esa pantalla DIFIERE del área
   que el usuario reportó (caso de mismatch), avisalo explícito: *"verificá en
   <pantalla real>, NO en <pantalla que reportaste>, porque el defecto vivía ahí"*.
   Sin esto, el usuario prueba en la pantalla equivocada y cree que el fix falló.
5. NO encadenes verificación automática en esta versión (queda para una iteración
   futura con el subagente QA-Playwright).

## Límites (qué NO hacer)
- No le preguntes detalles técnicos (archivos, funciones, frameworks) al humano.
- No inventes claridad: el % se deriva del modelo, nunca es un número "a sentimiento".
- No toques código durante la entrevista. La ejecución es del subagente `fix-dev`.
- No dispares el fix sin confirmación explícita del Fix-Brief.
