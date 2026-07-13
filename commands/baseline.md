---
description: Genera el baseline de specs OpenSpec del proyecto a partir de docs/business-logic.xml, vía el subagente baseline-generator. Escribe openspec/specs/<capability>/spec.md.
argument-hint: "[--trace]"
---

Usá el subagente `baseline-generator` para generar el baseline de specs de OpenSpec,
siguiendo la skill `generating-baseline-specs`.

Parámetros recibidos: $ARGUMENTS
- `--trace` (opcional) = agregar trazabilidad a las reglas de negocio dentro de cada
  spec (comentarios `<!-- BR-XX -->` que no rompen la validación). Sin el flag, specs limpias.

Precondiciones (verificar ANTES de arrancar):
- Debe existir `docs/business-logic.xml` verificado. Si no existe, avisá que hay que correr
  `/business-logic` (y `/validate-business-logic`) primero. El baseline es downstream de eso.
- Debe existir `openspec/` con schema `spec-driven`. Si no, avisá que hay que correr `openspec init`.

Flujo obligatorio:
1. Ejecutá la Fase 1 (descubrimiento) y mostrá el MANIFIESTO: mapa de capabilities propuesto
   (nombre kebab-case + qué flows/BR cubre + nº estimado de requirements), nombres de capability
   potencialmente bloqueados por el filtro de seguridad (credentials/secrets/keys/tokens…) con su
   alternativa, y qué comportamiento del XML quedaría sin mapear.
2. PARÁ ahí y esperá la confirmación del usuario (incluido el mapa de capabilities, por si hay que
   renombrar o reagrupar).
3. Solo tras la confirmación, generá las specs (una `spec.md` por capability) y validalas con
   `openspec validate --specs` hasta que pasen todas.

Guardá las specs en `openspec/specs/<capability>/spec.md` (es un entregable del proyecto).
Es el puente entre el análisis (`docs/business-logic.xml`) y el flujo SDD de OpenSpec: convierte
el QUÉ hace la app en specs vivas contra las que se harán los cambios.
