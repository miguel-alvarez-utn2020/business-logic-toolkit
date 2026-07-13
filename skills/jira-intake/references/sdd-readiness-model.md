# Modelo de Readiness para SDD — Gate del /from-jira

El % de readiness NO es una opinión del modelo: se **deriva** de cuántas dimensiones
de la tarea de Jira quedaron resueltas. Mide si el issue está **lo suficientemente
claro para arrancar el flujo SDD** (propose → specs → design → tasks). Espeja el
modelo de claridad de `/fix`, pero orientado a *iniciar un spec*, no a un bug.

## Diferencia clave con /fix

En `/fix` el humano responde una entrevista en vivo. Acá la fuente primaria es el
**texto del ticket de Jira** (ya escrito). El agente **puntúa el issue tal como está**.
Cuando una dimensión de negocio falta, el agente NO la inventa: la marca como faltante
y la resuelve por una de dos vías (ver "Qué hacer si no llega al umbral").

## Quién resuelve cada dimensión

- **Jira / Humano**: dimensiones de negocio (valor, alcance, criterios de aceptación).
  Salen del ticket; si faltan, se piden al usuario o se recomienda completarlas en Jira.
- **Agente**: dimensiones técnicas (anclaje en el código, encaje con el dominio). Se
  resuelven contra el código y `docs/business-logic.xml`. NUNCA se le preguntan al humano.

## Dimensiones base (máximo 100%)

| # | Dimensión | Resuelve | Peso | Resuelta cuando... |
|---|-----------|----------|------|--------------------|
| 1 | Objetivo / valor (why) | Jira | 20 | Está claro qué problema resuelve y para quién (historia de usuario o motivo). |
| 2 | Alcance (what) | Jira | 20 | Está delimitado qué se construye; idealmente con límites / "fuera de alcance". |
| 3 | **Criterios de aceptación** | Jira | 25 | Existen CA **observables/testables** → mapeables a `#### Scenario` (WHEN/THEN). |
| 4 | Anclaje técnico | Agente | 20 | Se identifica dónde vive (capability existente vs nueva; archivos candidatos) contra el código / business-logic.xml. |
| 5 | Encaje con el dominio | Agente | 15 | El pedido es coherente con lo que la app hace, o se reconoce explícitamente que es feature nuevo. |

Total = **100**. Umbral **"listo para SDD" ≈ 85%**.

Nota de pesos: los **Criterios de aceptación** pesan más (25) porque son el corazón del
spec — cada CA se vuelve un scenario. Un issue sin CA claros produce specs pobres.

## Scoring

Cada dimensión: **0 (sin resolver)**, **parcial (50% del peso)**, **resuelta (100%)**.
Readiness% = suma de pesos obtenidos.

- **≥ 85%** → listo: mostrar el Change-Brief y confirmar (gate normal).
- **< 85%** → no arrancar SDD todavía: surfacear lo que falta (ver abajo).

## Formato del tablero (mostrar tras evaluar el issue)

```
Readiness SDD: 72%   [███████░░░]

  ✅ Objetivo / valor        (20/20)
  ✅ Alcance                 (20/20)
  🟡 Criterios de aceptación (12/25)  ← hay descripción, pero sin CA observables/testables
  ✅ Anclaje técnico         (20/20)  → MenuItemListing.tsx (ítem logout) · colors.error
  ⬜ Encaje con el dominio   (0/15)   ← "contactos" no existe en la app (¿feature nuevo?)
```

Leyenda: ✅ resuelta · 🟡 parcial · ⬜ sin resolver.

## Qué hacer si NO llega al umbral (85%)

No generes el spec con input turbio. Según qué dimensión falte:

- **Criterios de aceptación ausentes/vagos (lo más común):** derivá CA **candidatos**
  a partir de la descripción y presentálos para que el usuario los confirme/complete
  (AskUserQuestion). NO los des por buenos en silencio: los CA definen los scenarios.
  Si el usuario no puede confirmarlos, recomendá completarlos en el ticket de Jira
  (y, con confirmación, se pueden dejar como comentario en el issue).
- **Objetivo / alcance poco claros:** pedí al usuario que aclare el "para qué" o el
  límite, o recomendá afinarlo en Jira.
- **Encaje con el dominio bajo (pedido "fuera de dominio"):** no es un bloqueo duro,
  pero avisá explícito: "esto no existe hoy en la app → sería una capability nueva",
  para que el usuario confirme que es intencional.
- **Anclaje técnico (lo resuelve el agente):** si no podés ubicar dónde vive, decilo
  con honestidad (no inventes). Si es feature nuevo, marcalo como capability nueva.

Regla anti-loop: pedí aclaración de a una dimensión, recomputá y mostrá el tablero.
En cuanto cruzás el 85%, pasá al Change-Brief. No interrogues por inercia.

## Anclaje de citas (NO de memoria)

Igual que en `/fix`: toda referencia a una BR, un `source`, una convención o un archivo
tiene que haber sido **leída** (Read/Grep) antes de citarla. La dimensión "anclaje
técnico" se puntúa por lo que confirmaste en el código, no por lo que dice el ticket.
