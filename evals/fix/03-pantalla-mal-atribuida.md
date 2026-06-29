# Eval /fix — 03: pantalla mal-atribuida (convergencia, área ≠ localización)

- **Qué prueba:** que el agente trate el área reportada como indicio falible, encuentre
  el defecto real aunque esté en otra pantalla, converja sin loop, y avise dónde verificar.
- **Criticidad:** alta (reportes no técnicos suelen errar la pantalla).

## Setup
- App: `rosfar-backoffice`. Read-only.
- Hecho del código: `validateFile()` (en `core/services/file-upload/file-upload.service.ts`,
  usado solo al subir conformidad de **Devoluciones**) valida tipo por MIME. El flujo de
  **Importar PDF (OCR)** usa otro componente (`multi-upload-document`) que valida por extensión.

## Input (guion del QA)
- Comando: `/fix` en frío.
- Área: "Atención al Cliente". Pantalla reportada: "Importar PDF (OCR)".
- Síntoma: "al subir un comprobante PDF lo rechaza como 'tipo no permitido'". (La pantalla
  reportada NO es donde está el defecto de MIME.)

## Expected

### Objetivos
- [ ] Distingue que el defecto de validación por MIME vive en el flujo de Devoluciones
      (`validateFile`), no en el de Importar PDF que el usuario nombró.
- [ ] NO inventa un defecto en el flujo de OCR que el código no tiene.

### De juicio
- [ ] **Converge**: ≤1 repregunta de pantalla; no loopea 3+ veces.
- [ ] Trata el área reportada como indicio, no como verdad; marca el mismatch como observación.
- [ ] Indica **dónde verificar** el fix (la pantalla real, avisando que difiere de la reportada).

## Runs / umbral
- 3 corridas. **2/3** objetivos; **2/3** de juicio.
