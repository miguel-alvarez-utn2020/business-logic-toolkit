---
name: extracting-conventions
description: >
  Analiza un proyecto para extraer sus convenciones técnicas (cómo se hace cada
  cosa: arquitectura, estado, datos, componentes, forms, estilos, tooling),
  cruzadas con la guía oficial de la tecnología para la versión detectada, con el
  delta entre ambas. Produce docs/conventions.md, que guía a fix-dev y al rework.
argument-hint: "[carpeta | .]"
allowed-tools: Read, Grep, Glob, WebFetch, WebSearch
---

Genera el documento de convenciones del proyecto. La idea: que cualquier cambio
futuro siga el estilo del equipo (consistencia) y, donde no haya patrón, caiga en
lo idiomático de la tecnología. NO son best-practices genéricas: es lo que ESTE
proyecto hace, cruzado con lo oficial.

## Fase 1: Discovery (stack + versión + muestreo)

1. Detectá el stack y la **versión exacta** (leé `package.json`, `angular.json`,
   etc.). La versión es crítica: las convenciones de Angular 14 ≠ Angular 18.
2. Seleccioná el catálogo de detección: `angular.json` → references/catalog-angular.md.
   Si no hay catálogo para el stack detectado, PARÁ y avisá (no adivines).
3. Por cada categoría del catálogo, listá archivos representativos a muestrear
   (no todos: una muestra suficiente para ver el patrón dominante).

## Fase 2: Guía oficial (traer y cachear)

1. Para la versión detectada, traé la guía oficial de las fuentes del catálogo
   (WebFetch). Extraé los puntos relevantes por categoría (ej. style guide, control
   flow, signals, standalone).
2. Si el WebFetch falla o no hay conectividad, usá tu conocimiento del framework
   PERO citá igual la URL oficial y aclará "(no verificado online)".
3. Vas a **hornear** estos puntos en el documento (con URL + versión + fecha), para
   que quien lea `conventions.md` después NO necesite volver a la web.

## Fase 3: Extracción (proyecto + delta, por categoría)

Seguí el esquema en `references/output-schema-conventions.md`. Para CADA categoría
activa del catálogo:
1. **Convención del proyecto:** identificá el patrón DOMINANTE en el código y
   respaldalo con 1-2 ejemplos reales (`archivo → patrón`). Si hay inconsistencias
   o duplicación, anotalas como excepción.
2. **Referencia oficial:** qué recomienda la tecnología para esa categoría en la
   versión detectada (de la Fase 2), con URL.
3. **Delta:** clasificá la relación:
   - `alineado` — el proyecto sigue lo oficial.
   - `desviación` — el proyecto hace otra cosa (decí cuál y por qué podría ser; NO
     lo corrijas, solo documentá; marcá si parece deuda).
   - `sin patrón` — el proyecto no tiene un patrón establecido (acá, para un cambio
     nuevo, vale caer en lo oficial).

### Límites (qué NO hacer)
- No inventes convenciones que el código no muestra.
- No "arregles" desviaciones: este doc describe, no refactoriza.
- No traigas guías de fuentes no oficiales.

## Fase 4: Coverage (self-check)

1. Por categoría, indicá si quedó cubierta (con evidencia) o no se pudo determinar.
2. Listá explícitamente las categorías sin patrón claro y las desviaciones/deuda
   detectadas.
3. Anotá la versión del stack y la fecha, para saber cuándo el doc podría quedar
   desactualizado.

Guardá el resultado en `docs/conventions.md` (es un entregable del proyecto, lo
consume `fix-dev` y el rework).
