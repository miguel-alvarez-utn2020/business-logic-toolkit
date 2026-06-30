# Eval business-logic — 01: cobertura y precisión sobre rosfar

- **Qué prueba:** que la extracción de lógica de negocio cubra el manifiesto y que sus
  reglas existan de verdad en el código (precisión / trazabilidad).
- **Criticidad:** alta (es el contexto del que dependen `/fix` y el rework).

## Setup
- App: `rosfar-backoffice` (Angular 18, catálogo `catalog-angular`). Read-only.

## Input
- Ejecutar la extracción (Fase 1+2+3) sobre `.`, formato xml.

## Expected

### Objetivos (verificables)
- [ ] Selecciona el catálogo `catalog-angular` (frontend, sin Nest).
- [ ] Cubre las secciones core (entities, business-rules, user-flows) y las opcionales
      activadas (roles, integrations, realtime).
- [ ] **Toda `<rule>` tiene `<source>`** (archivo → función). Ninguna sin source.
- [ ] Detecta entidades clave: user, role, shipping, return, medication-request,
      pdf-staging-row, notification.

### De juicio (spot-check directo, read-only — NO requiere orquestar otra tool)
- [ ] Verificá vos mismo leyendo el código: las 4 reglas sample existen donde el doc
      declara su `source` (BR-01 → auth.guard; BR-04 → module.guard ROUTE_REQUIRED_PERMISSIONS;
      BR-19 → tablero anti-spam; BR-27 → file-upload validateFile). Confirmá leyendo cada archivo.
- [ ] No hay reglas inventadas que el código no tenga (sin alucinaciones) en el sample.
- [ ] Coverage report presente con % y gaps explícitos.

## Runs / umbral
- 2 corridas (la extracción es más determinista). **2/2** objetivos; las 4 reglas
  sample confirmadas en el código.
