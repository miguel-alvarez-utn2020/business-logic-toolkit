# Modelo de Claridad — Gate del /fix

El % de claridad NO es una opinión del modelo: se **deriva** de cuántas
dimensiones quedaron resueltas y ancladas al código. Esto es lo que hace el
gate honesto y escalable a otros flujos (`/feature`, `/refactor`).

## Quién resuelve cada dimensión

- **Humano**: dimensiones de comportamiento observable. Se preguntan en opción
  múltiple.
- **Agente**: dimensiones técnicas. Las resuelve buscando en el código y en el
  doc de lógica de negocio. NUNCA se le preguntan al humano.

## Dimensiones base (máximo 90%)

| # | Dimensión              | Resuelve | Peso | Resuelta cuando... |
|---|------------------------|----------|------|--------------------|
| 1 | Esperado vs actual     | Humano   | 22   | Está claro qué hace hoy y qué debería hacer. |
| 2 | Localización           | Agente   | 22   | Hay un `source` candidato (archivo→función) con confianza, mapeado a una BR/flujo del doc cuando aplica. |
| 3 | Reproducción           | Humano   | 18   | Se sabe cómo disparar el bug: dónde, con qué pasos, si es siempre o intermitente. |
| 4 | Criterio de aceptación | Humano   | 18   | Está definido cómo sabremos que quedó arreglado (observable). |
| 5 | Restricciones          | Ambos    | 10   | Está claro qué NO debe cambiar / qué no romper. |

Total base = **90**. Con las 5 base plenamente resueltas, la claridad ronda el
85–90% → umbral "listo".

## Dimensiones de afinado (suman hasta 10% → techo 100%)

Solo se trabajan en la Fase 3 (afinar). Aportan precisión extra para bugs
riesgosos.

| # | Dimensión              | Resuelve | Peso | Resuelta cuando... |
|---|------------------------|----------|------|--------------------|
| 6 | Casos borde / datos    | Humano   | 5    | Se cubrieron variaciones de datos y estados raros relevantes. |
| 7 | Riesgo de regresión    | Ambos    | 5    | Se identificaron flujos relacionados que el fix podría afectar. |

## Scoring

Cada dimensión se puntúa: **0 (sin resolver)**, **parcial (50% del peso)**,
**resuelta (100% del peso)**. Claridad% = suma de los pesos obtenidos.

- Umbral "listo" ≈ **85%** → se cruza con casi todas las base resueltas.
- "Precisión máxima" ≈ **95%+** → base completa + afinado.

## Formato del tablero (mostrar tras cada respuesta)

```
Claridad: 84%   [████████░░]

  ✅ Esperado vs actual      (22/22)
  ✅ Localización            (22/22)  → order.service.ts → checkout()  [BR-01]
  ✅ Reproducción            (18/18)
  🟡 Criterio de aceptación  (9/18)   ← falta definir cómo validamos el arreglo
  ✅ Restricciones           (10/10)
  ⬜ (afinado) Casos borde   (0/5)
  ⬜ (afinado) Regresión     (0/5)
```

Leyenda: ✅ resuelta · 🟡 parcial · ⬜ sin resolver.

## Banco de preguntas

Adaptá la redacción al bug concreto; estas son guías. Preferí opción múltiple.

### Comportamiento (Dim. 1 — Esperado vs actual)
- "¿Qué hace la app hoy cuando ocurre el problema?" (opciones según síntoma)
- "¿Qué debería hacer en cambio?"

### Reproducción (Dim. 3)
- "¿Dónde lo viste?" (pantalla / sección — ofrecé las del doc de flujos)
- "¿Qué pasos hacés justo antes de que falle?"
- "¿Pasa siempre o a veces?" (Siempre / A veces / Una sola vez)

### Criterio de aceptación (Dim. 4)
- "¿Cómo sabríamos que quedó bien arreglado?" (resultado observable esperado)

### Restricciones (Dim. 5)
- "¿Hay algo que funcione bien hoy y que NO querés que cambie?"

### Afinado — solo Fase 3
- Casos borde (Dim. 6): "¿Pasa también con [otro dato/estado]?", variaciones.
- Regresión (Dim. 7): "¿Este botón/pantalla se usa en otros lugares que conozcas?"
  (el agente complementa rastreando usos en el código).

## Localización (Dim. 2 — la resuelve el agente, no se pregunta)

Para subir esta dimensión:
1. Tomá el síntoma + la pantalla/flujo que dio el humano.
2. Cruzá con el doc de lógica de negocio: ¿qué `BR-id` / flujo corresponde?
3. Seguí el `source` de esa regla y/o buscá en el código (Grep/Glob/Read) la
   función responsable.
4. Marcala "resuelta" solo si tenés un candidato con confianza alta; "parcial"
   si tenés la zona pero no la línea; 0 si no encontraste nada (decilo, no
   inventes).
