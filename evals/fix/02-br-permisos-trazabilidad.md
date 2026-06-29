# Eval /fix — 02: trazabilidad a regla de negocio (permisos de ruta)

- **Qué prueba:** que el `/fix` mapee el síntoma a la BR correcta del doc de negocio
  y use ese contexto para localizar y proponer el valor correcto.
- **Criticidad:** alta (es el diferenciador vs un prompt suelto).

## Setup
- App: `rosfar-backoffice`. Read-only. Precondición: existe `docs/business-logic.xml`.

## Input (guion del QA)
- Comando: `/fix` en frío.
- Área: "Atención al Cliente". Pantalla: "Coordinación (tablero)".
- Síntoma: "un usuario que tiene permiso del Tablero de Coordinación, al entrar a
  Coordinación lo redirige afuera como si no tuviera permiso". SuperAdmin entra ok.

## Expected

### Objetivos
- [ ] Localiza en `src/app/core/guards/module/module.guard.ts`, mapa
      `ROUTE_REQUIRED_PERMISSIONS`, clave `/admin/coordinacion`.
- [ ] Identifica el permiso correcto como `ac_tablero_read` (del doc / contexto).

### De juicio
- [ ] Mapea el bug a **BR-04** (autorización por ruta granular) y cita su `source`.
- [ ] Usa el doc de negocio como fuente del permiso correcto (no adivina).
- [ ] Restricción: no relajar la seguridad (quien no tiene el permiso sigue bloqueado).

## Runs / umbral
- 3 corridas. **2/3** objetivos; **2/3** de juicio (BR-04 correcto = obligatorio).
