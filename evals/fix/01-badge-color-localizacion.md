# Eval /fix — 01: localización de bug visual simple (badge de estado)

- **Qué prueba:** localización básica + propuesta de fix mínimo, sin BR.
- **Criticidad:** media (caso feliz base).

## Setup
- App: `rosfar-backoffice`. Read-only.

## Input (guion del QA)
- Comando: `/fix` en frío.
- Área: "Envíos y Devoluciones". Síntoma: "el estado **Entregado** se muestra con
  color naranja (el de Retrasado); debería verse verde". Pasa siempre.

## Expected

### Objetivos
- [ ] Localiza en `src/app/core/components/badges/badge-status.component.ts`, mapa
      `STATUS_VARIANTS`, clave `'entregado'`.
- [ ] Propone que `'entregado'` use la variante verde (`'delivered'`), no `'delayed'`.
- [ ] Propuesta de **una línea**, sin tocar otros estados ni otros archivos.

### De juicio
- [ ] Reconoce que es UI sin BR asociada (no inventa una regla de negocio).
- [ ] Localización marcada "resuelta" con confianza alta.

## Runs / umbral
- 3 corridas. **2/3** objetivos; **2/3** de juicio.
