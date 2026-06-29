---
description: Guía un fix con una entrevista estructurada (gate de claridad) y, al confirmar el Fix-Brief, lo delega al subagente fix-dev. Pensado para que alguien no técnico (ej. QA) pueda dirigir el arreglo.
argument-hint: "[descripción breve del bug, opcional]"
---

Conducí un fix guiado siguiendo la skill `intake-fix`. El objetivo es convertir
un reporte de bug (posiblemente de alguien no técnico) en un Fix-Brief preciso, y
recién entonces delegar la ejecución al subagente `fix-dev`.

Punto de partida (opcional): $ARGUMENTS — si viene, usalo como síntoma inicial
del bug; si no, la primera pregunta de la entrevista lo recoge.

IMPORTANTE — esta entrevista corre en el hilo principal (vos), NO en un subagente,
porque necesita hacer preguntas interactivas al usuario con AskUserQuestion. El
subagente entra solo en el paso final, para ejecutar el fix ya especificado.

Flujo obligatorio (detallado en la skill `intake-fix`):
1. Cargá el contexto: ubicá el documento de lógica de negocio en `docs/`. Si no
   existe, PARÁ y recomendá correr `/business-logic` primero (es el contexto que
   da precisión al fix).
2. Entrevistá al usuario. Preguntale SOLO lo observable (comportamiento, pasos,
   qué esperaba), de a poco y en opción múltiple cuando se pueda. La localización
   técnica la resolvés vos contra el código. Mostrá el checklist de claridad y el
   % después de cada respuesta.
3. Al llegar al umbral "listo" (~85%), PARÁ de preguntar solo. Presentá el
   Fix-Brief COMPLETO y ofrecé UNA sola decisión: proceder (confirmar y aplicar) |
   afinar a 95%+. Recomendá si afinar vale la pena según el riesgo. "Proceder" ES
   la confirmación — no vuelvas a pedir confirmación aparte.
4. Solo tras "proceder", delegá en el subagente `fix-dev` pasándole el Fix-Brief.
   Mostrá el cambio aplicado (diff/preview) y decile al usuario DÓNDE verificar
   (avisá si la pantalla real difiere de la que reportó). NO encadenes verificación
   automática en esta versión.

Guardá el Fix-Brief en `docs/fixes/` (es un entregable trazable del proyecto).
