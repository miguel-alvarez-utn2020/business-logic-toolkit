# Esquema del Fix-Brief

El Fix-Brief es el entregable de la entrevista y el ÚNICO contexto que recibe el
subagente `fix-dev`. Tiene que ser autocontenido: si el brief es claro, el dev no
necesita volver a preguntar nada.

Se guarda en `docs/fixes/` como markdown legible (un QA tiene que poder leerlo).

## Estructura

```markdown
# Fix-Brief: <título corto del bug>

- **Reportado por:** <rol/nombre>   ·   **Claridad:** <NN>%   ·   **Fecha:** YYYY-MM-DD

## Síntoma (actual)
Qué hace la app hoy, en lenguaje observable. Sin jerga técnica.

## Esperado
Qué debería hacer en cambio.

## Reproducción
1. Paso...
2. Paso...
- Frecuencia: Siempre | A veces | Una vez

## Regla de negocio afectada
- <BR-id> — <enunciado>   ·   source: <archivo → función>
  (omitir si el bug no mapea a una regla; entonces dejar solo Localización)

## Localización (candidata)
- `archivo:línea` → `función()`   ·   confianza: alta | media | baja
- Notas del agente sobre por qué cree que es acá.

## Criterio de aceptación
- [ ] Condición observable 1 que prueba que está arreglado
- [ ] Condición observable 2

## Restricciones (no romper)
- Flujo / comportamiento que debe seguir funcionando igual.

## Fuera de scope
- Lo que explícitamente NO se toca en este fix.

## Afinado (solo si se completó la Fase 3)
- Casos borde a contemplar: ...
- Riesgo de regresión: flujos relacionados ...
```

## Reglas
- Todo lo técnico (BR-id, source, localización) lo completa el AGENTE; el humano
  nunca lo redacta.
- El criterio de aceptación debe ser **observable** (lo que vería un QA), no una
  afirmación sobre el código.
- Si una sección no aplica, omitila — no la dejes vacía.
- Trazabilidad: cuando exista regla de negocio, el `BR-id` y `source` se copian
  textualmente del documento de lógica de negocio en `docs/`.
