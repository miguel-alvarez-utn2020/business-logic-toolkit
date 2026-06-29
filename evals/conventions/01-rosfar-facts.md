# Eval conventions — 01: facts clave sobre rosfar

- **Qué prueba:** que la extracción de convenciones capture los hechos técnicos
  correctos (con evidencia) y clasifique bien el delta vs lo oficial.
- **Criticidad:** alta (lo consume `fix-dev` para no improvisar estilo).

## Setup
- App: `rosfar-backoffice` (Angular 18.2). Read-only.

## Input
- Ejecutar la extracción de convenciones (Fase 1-4) sobre `.`.

## Expected

### Objetivos (facts verificables contra el código)
- [ ] Estado: identifica **NGXS** (no NgRx, no signals), con states en `core/store/`.
- [ ] Arquitectura: **100% standalone** (0 `*.module.ts`).
- [ ] Guards: **de clase** (no funcionales).
- [ ] Forms: detecta la **validación duplicada inline** + el validador compartido
      `core/validators/password.validators.ts` como **código muerto**.
- [ ] UI: detecta múltiples librerías conviviendo (Material + PrimeNG + ag-grid).
- [ ] Cada convención del proyecto trae al menos un `source` (archivo).

### De juicio
- [ ] La referencia oficial está **citada con URL y versión**, y respeta la versión
      detectada (no recomienda APIs que v18 no tiene como obligatorias).
- [ ] El delta clasifica bien: NGXS/guards-de-clase como "desviación intencional",
      validación duplicada y multi-UI como "deuda".
- [ ] No inventa convenciones sin evidencia.

## Runs / umbral
- 2 corridas. **2/2** en los facts objetivos críticos (NGXS, standalone, validador muerto).
