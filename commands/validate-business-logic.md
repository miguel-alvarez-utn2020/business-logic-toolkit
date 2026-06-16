---
description: Verifica que las reglas de un documento de lógica de negocio existen realmente en el código.
argument-hint: "[ruta-del-documento]"
---

Usá el subagente `business-logic-verifier` para auditar el documento de
lógica de negocio indicado, siguiendo la skill `verify-business-logic`.

Documento a verificar: $ARGUMENTS
- Si no se indica una ruta, buscá el documento en `docs/` (el lugar por
  defecto donde se guardan las extracciones).

Verificá cada regla contra el código y devolvé el reporte: por regla el
veredicto (CONFIRMED / BAD_SOURCE / NOT_FOUND) con una nota breve, y al final
el resumen con los totales.
