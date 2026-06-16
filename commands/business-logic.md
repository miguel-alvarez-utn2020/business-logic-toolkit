---
description: Extrae la lógica de negocio del proyecto (o de una carpeta) a XML o MD, vía el subagente business-analyst.
argument-hint: "[carpeta | .] [xml | md]"
---

Usá el subagente `business-analyst` para extraer la lógica de negocio,
siguiendo la skill `extracting-business-logic`.

Parámetros recibidos: $ARGUMENTS
- Primer argumento = carpeta a analizar. Si es `.` o no se indica, analizá
  la raíz del proyecto completa.
- Segundo argumento = formato de salida (`xml` o `md`). Si no se indica, usá `xml`.

Flujo obligatorio:
1. Ejecutá la Fase 1 (descubrimiento) y mostrá el MANIFIESTO: secciones
   activadas, conteos por sección, y qué quedó como NO detectado.
2. PARÁ ahí y esperá la confirmación del usuario.
3. Solo tras la confirmación, ejecutá la extracción completa en el formato pedido.

Guardá el documento resultante en `docs/` (es un entregable del proyecto),
no dentro de la skill.
