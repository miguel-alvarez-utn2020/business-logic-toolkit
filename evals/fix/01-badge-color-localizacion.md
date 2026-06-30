# Eval /fix — 01: localización de bug visual simple (badge de estado)

- **Qué prueba:** localización básica + propuesta de fix mínimo, sin BR.
- **Criticidad:** media (caso feliz base).

## Setup
- App: `rosfar-backoffice`. Read-only.

## Input (guion del QA)
- Comando: `/fix` en frío.
- Área: "Envíos y Devoluciones". Síntoma: "el estado **Entregado** se muestra con
  color naranja (el de Retrasado); debería verse verde". Pasa siempre.

> NOTA (modo read-only): este eval corre sobre código LIMPIO (sin bug sembrado). El
> mapeo estado→color ya es correcto (`'entregado' → 'delivered'`). Por eso NO se
> espera "proponer un cambio": se evalúa la **localización** y que reconozca cuál es
> el valor correcto. Que concluya "ya está correcto / no reproducible" es PASS, no fail.

## Expected

### Objetivos
- [ ] Localiza el mapeo estado→color en `src/app/core/components/badges/badge-status.component.ts`,
      mapa `STATUS_VARIANTS`, clave `'entregado'` (ahí se decide el color).
- [ ] Identifica que la variante correcta para "Entregado" es la verde (`'delivered'`).

### De juicio
- [ ] Reconoce que es UI sin BR asociada (no inventa una regla de negocio).
- [ ] Como el código limpio ya es correcto, aplica el gate y concluye "ya correcto /
      no reproducible" SIN inventar un defecto. (Si en otra corrida hubiera bug, debería
      proponer `'delivered'`.)

## Runs / umbral
- 3 corridas. **2/3** objetivos; **2/3** de juicio.
