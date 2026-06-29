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
| 2 | Localización (en código) | Agente | 22   | Hay un `source` (archivo→función) con confianza ALTA que **reproduce el síntoma reportado**, mapeado a una BR/flujo cuando aplica. Se puntúa por confianza del CÓDIGO, NO por si coincide con la pantalla que nombró el humano (ver nota abajo). |
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

### Apertura (agnóstica del área — solo si no hay síntoma inicial claro)
- "¿En qué parte de la app viste el problema?" — opciones = áreas/flujos del
  documento de negocio (envíos, devoluciones, coordinación, usuarios/roles,
  login…) + "Otra". NO asumas el área; dejá que el usuario la elija.
- Recién con esa respuesta pasás a las preguntas de comportamiento.

### Comportamiento (Dim. 1 — Esperado vs actual)
- "¿Qué hace la app hoy cuando ocurre el problema?" (opciones según síntoma)
- "¿Qué debería hacer en cambio?"
- Si hay un mensaje de error visible: "¿Qué dice el mensaje, textual?" — capturalo
  temprano: el texto del error suele ubicar el defecto mejor que la pantalla.

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
1. Tomá el síntoma observable (sobre todo el **mensaje de error textual**, que es
   más diagnóstico que la pantalla) + el área que dio el humano como INDICIO.
2. Cruzá con el doc de lógica de negocio: ¿qué `BR-id` / flujo corresponde?
3. Seguí el `source` de esa regla y/o buscá en el código (Grep/Glob/Read) el
   defecto que **reproduce el síntoma**.
4. Puntuá por confianza del CÓDIGO (sujeto al gate de reproducción de abajo):
   - **resuelta (22)**: encontraste un defecto verificable que reproduce el
     síntoma reportado **bajo los hechos que el usuario afirmó**, con
     archivo→función concretos.
   - **parcial (11)**: tenés la zona pero no la línea/causa exacta, hay más de un
     candidato sin discriminar, o el candidato solo reproduce el síntoma si se
     asume algo que el usuario NO confirmó.
   - **0**: no encontraste nada (decilo, no inventes).

### Gate de reproducción (OBLIGATORIO antes de marcar resuelta)
Antes de puntuar Localización como "resuelta", verificá explícitamente:
**¿este defecto reproduce el síntoma reportado USANDO SOLO los hechos que el
usuario afirmó?**

- **Sí** → resuelta (22). Avanzá.
- **Solo si asumo un hecho extra que el usuario no dijo** → NO está resuelta
  (máx. parcial). NO inventes ese hecho para que calce.
- **Requiere asumir algo que CONTRADICE una respuesta del usuario** → STOP. Es
  una contradicción, no una localización (ver abajo).

**Chequeo de contradicción.** Si los hechos que afirmó el usuario son incompatibles
con lo que hace el código candidato (ej.: el usuario dice "el nombre termina en
.pdf y lo rechaza", pero el código ACEPTA todo lo que termina en .pdf), entonces
ese código NO explica el síntoma. NO lo "arregles" igual basándote en una hipótesis
alternativa (ej. "los archivos vienen sin extensión") que el usuario no afirmó o
que contradice lo que dijo. En cambio:
1. Surfacéale la contradicción al usuario en lenguaje claro.
2. Pedile el dato que la resuelve (re-confirmar el síntoma / un ejemplo concreto),
   o reconocé que no podés localizar el defecto con lo que hay.
3. Localización queda **parcial/0**. NO procedas a un fix a ciegas: un cambio que
   por tu propio análisis no resuelve lo reportado NO es un fix.

**Convergencia ante contradicción (NO te vayas a la madriguera).** Cuando el código
contradice el reporte, el objetivo es cerrar rápido y claro, no investigar de más:
- Parás en: *"contra este código, esto no se reproduce; el defecto que describís no
  está acá"* + **un (1) chequeo concreto** que lo confirme del lado del usuario
  (ej. hard refresh con caché desactivada y mirar si dispara request).
- La causa ambiental (build viejo cacheado, deploy desfasado del repo, dato real
  distinto) se menciona **breve, como posibilidad** — NO la conviertas en una
  investigación: nada de arqueología de git multi-rama, blame extendido ni teorías
  de deploy, salvo que el usuario confirme el escenario y pida cavar.
- Tope: como mucho **una** pregunta de desempate más. Si sigue sin cerrar, entregá
  el intake "no-reproducible" con el chequeo sugerido y listo. No insistas en loop.

Hallazgo lateral legítimo: si de paso descubrís una fragilidad real distinta del
síntoma reportado (p.ej. código muerto, validación duplicada), anotala como
**observación / posible `/fix` aparte** — pero NO la presentes como "el arreglo de
lo que reportaste" si no reproduce el síntoma.

### Área reportada ≠ localización en código (regla clave)
El área/pantalla que nombró el humano es un **indicio falible**, NO parte del
score. Es normal y esperable que un reporte no técnico atribuya mal la pantalla
mientras describe bien el síntoma.

- Si encontrás **un defecto certero y verificable que reproduce el síntoma
  central**, la Localización es **resuelta (22)** AUNQUE el defecto NO esté en la
  pantalla que nombró el humano. El mismatch de pantalla es una **observación**
  que se anota en el Fix-Brief, no un motivo para bajar el score ni para quedarse
  trabado.
- **Regla de convergencia (anti-loop):** cuando un único defecto de alta
  confianza explica el síntoma central, **re-preguntá la pantalla UNA sola vez**
  como máximo. Si el humano insiste en su pantalla, NO sigas investigándola en
  círculos ni inventes un defecto que el código no tiene. Presentá:
  *"El defecto real que reproduce lo que viste está en X; la pantalla que nombraste
  (Y) no tiene este defecto, pero X explica el síntoma. Procedemos con X."* y
  avanzá al gate.
- Si NO hay ningún defecto que reproduzca el síntoma en NINGÚN lado, ahí sí
  Localización queda parcial/0 y lo decís con honestidad (no fabriques una causa).
