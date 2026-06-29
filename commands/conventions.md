---
description: Extrae las convenciones técnicas del proyecto (cómo se hace cada cosa acá) cruzadas con la guía oficial de la tecnología, vía el subagente conventions-analyst. Genera docs/conventions.md.
argument-hint: "[carpeta | .]"
---

Usá el subagente `conventions-analyst` para extraer las convenciones técnicas del
proyecto, siguiendo la skill `extracting-conventions`.

Parámetro: $ARGUMENTS — carpeta a analizar. Si es `.` o no se indica, analizá la
raíz del proyecto completa.

Flujo obligatorio:
1. Ejecutá la Fase 1 (discovery): detectá stack + **versión exacta** y mostrá qué
   catálogo se seleccionó y las categorías a cubrir.
2. PARÁ ahí y esperá confirmación del usuario (incluida la versión detectada, por
   si hay que ajustar qué guía oficial traer).
3. Tras la confirmación, ejecutá la extracción completa: guía oficial (cacheada) +
   patrones del proyecto + delta, por categoría.

Guardá el documento en `docs/conventions.md` (es un entregable del proyecto; lo
consume el subagente `fix-dev` y, más adelante, el rework). Es complementario al
`docs/business-logic.xml`: uno dice QUÉ hace el negocio, este dice CÓMO se construye.
