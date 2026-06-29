# Eval /fix — 05: el fix respeta las convenciones del proyecto

- **Qué prueba:** que ante un cambio que toca estado/datos, la propuesta siga las
  convenciones reales del proyecto (NGXS, no signals/BehaviorSubject; service+AppSettings)
  en vez de improvisar el estilo de la IA.
- **Criticidad:** alta (clave para fixes grandes hechos por gente sin experiencia).

## Setup
- App: `rosfar-backoffice`. Read-only. Precondición ideal: existe `docs/conventions.md`.

## Input (guion del QA)
- Comando: `/fix` en frío.
- Área: "Atención al Cliente". Síntoma: "el contador del tablero de coordinación no se
  actualiza / queda viejo; debería reflejar el estado nuevo". (Un cambio que toca estado.)

## Expected

### Objetivos
- [ ] La propuesta usa el patrón de estado del proyecto: **NGXS** (tríada
      `*.state.ts`/`@Action`/`patchState` + `select`), NO signals ni `BehaviorSubject` ad-hoc.
- [ ] Si toca datos, propone vía service + `AppSettings`, consistente con el código existente.

### De juicio
- [ ] Cita o respeta `docs/conventions.md` (o el patrón vecino) explícitamente.
- [ ] No introduce librerías/patrones nuevos; reusa lo existente.
- [ ] Si la causa raíz no es clara, aplica el gate de reproducción (no inventa).

## Runs / umbral
- 3 corridas. **2/3** objetivos; **2/3** de juicio.
