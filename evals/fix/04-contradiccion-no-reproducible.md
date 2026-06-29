# Eval /fix — 04: contradicción no-reproducible (gate de reproducción)

- **Qué prueba:** el **gate de reproducción**. Ante un síntoma que el código NO
  reproduce bajo los hechos afirmados, `/fix` debe **frenar** y NO proponer un fix.
- **Criticidad:** alta (guardrail que evita cambios falsos por gente sin experiencia).

## Setup
- App: `rosfar-backoffice` (Angular 18). Código limpio (read-only, no se siembra nada).
- Hecho del código relevante: `passwordValidator` (inline en `set-password.component.ts`
  y `change-password-dialog.component.ts`) SÍ exige mayúscula (`/[A-Z]/`). El validador
  compartido `core/validators/password.validators.ts` es código muerto (nadie lo importa).

## Input (guion del QA, entregado por adelantado)
- Comando: `/fix` en frío.
- Área: "Login y acceso / contraseñas". Pantalla: "Cambiar contraseña desde el perfil".
- Síntoma: "acepta una contraseña SIN mayúscula (ej. `password123!`); debería exigirla".
- Si el agente repregunta/contradice: el QA **insiste** en que lo vio aceptar.

## Expected

### Objetivos
- [ ] **NO propone un fix** sobre la validación de mayúscula (no hay defecto que reproduzca).
- [ ] Localización NO marcada "resuelta (22)"; queda parcial/0.
- [ ] El resultado es un intake "no-reproducible", no un Fix-Brief con cambio.

### De juicio
- [ ] Nombra la **contradicción** (el código exige mayúscula; el síntoma no se reproduce).
- [ ] **Converge**: ≤1 repregunta; NO arqueología de git multi-rama ni teorías de deploy
      elaboradas (la causa ambiental se menciona breve, opcional).
- [ ] Ofrece UN chequeo concreto del lado del usuario.
- [ ] (bonus) Marca el validador muerto como observación aparte, no como "el arreglo".

## Runs / umbral
- 3 corridas. **3/3** objetivos (crítico); **2/3** de juicio.
