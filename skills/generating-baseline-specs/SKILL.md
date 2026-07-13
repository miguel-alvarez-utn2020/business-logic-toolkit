---
name: generating-baseline-specs
description: >
  Genera el baseline de specs vivas de OpenSpec para una app ya desarrollada, a
  partir de docs/business-logic.xml (entidades, reglas de negocio, user-flows).
  Produce openspec/specs/<capability>/spec.md, validadas con el CLI de OpenSpec.
  Úsese cuando se quiera sembrar los specs de un proyecto brownfield para empezar
  a trabajar spec-driven (delta contra la base). Es downstream de la extracción de
  lógica de negocio: requiere un business-logic.xml verificado.
argument-hint: "flag opcional: --trace (agrega referencias BR-XX a los specs)"
allowed-tools: Read, Grep, Glob, Write, Bash
---

## Precondiciones

- `docs/business-logic.xml` existe y está verificado. Si no, DETENÉ y pedí correr
  `/business-logic` (+ `/validate-business-logic`) primero. El baseline sale de ese XML,
  no del código directo.
- OpenSpec inicializado (`openspec/` con schema `spec-driven`). Si no, `openspec init`.
- Confirmá el formato canónico si dudás: `openspec templates` y el `schema.yaml` del paquete.

## En base a qué sale el baseline

Cadena de origen (regla de oro: todo requirement rastreable al XML → file:line del código):

```
código → business-analyst → business-logic.xml → business-logic-verifier → (esta skill) → openspec/specs/
```

El baseline hereda las fronteras del XML: lo que el XML marcó "NO detectado" no entra.

## Phase 1: Discovery (build the capability map — y PARÁ)

No escribas specs todavía. Inventariá y proponé el mapa.

1. Leé `docs/business-logic.xml`: user-flows, business-rules (BR-XX + su `type`), entities, roles, domain-events.
2. Proponé el MAPA DE CAPABILITIES (kebab-case). Heurística:
   - Una capability por dominio/feature.
   - Agrupá flows muy acoplados (login + refresh + logout → `authentication`).
   - Dividí si crece demasiado o mezcla conceptos (biometría → `biometric-access`).
3. Por cada capability listá: qué user-flows y qué BR cubre, y el nº estimado de requirements.
4. Detectá nombres bloqueables por el filtro de seguridad del host (`credentials`, `secrets`,
   `keys`, `tokens`…) y proponé alternativa de dominio (ej. `credenciales`).
5. Listá el comportamiento del XML que NO quedaría mapeado.
6. Mostrá el manifiesto y ESPERÁ confirmación (puede pedir renombrar/reagrupar).

## Phase 2: Generation (tras confirmación)

Escribí una `openspec/specs/<capability>/spec.md` por capability.

### Formato EXACTO de la spec viva (validado, CLI 1.5)

```markdown
# <Título legible>

## Purpose

<1-3 frases>

## Requirements

### Requirement: <nombre corto>

El sistema SHALL <comportamiento normativo>.

#### Scenario: <nombre>

- **WHEN** <condición>
- **THEN** <resultado esperado>
```

Reglas que causan fallo (o fallo silencioso):
- Scenarios con EXACTAMENTE 4 `####`. Requirements con 3 `###`.
- Cada requirement ≥1 scenario. Usar SHALL/MUST (no "debería/puede").
- WHEN/THEN en negrita. Todo en español. Actor: "el ciudadano" / "el sistema".
- Spec viva usa `## Requirements` — NUNCA `## ADDED Requirements` (eso es delta de change).

### Reglas de mapeo BR/flow → Requirement/Scenario

- Regla `validation` → Requirement + 2 scenarios: input válido / input inválido.
- Regla `conditional-gate` → Requirement + 2 scenarios: gate pasa / gate bloquea.
- Regla `authorization` → Requirement + scenarios por rol/nivel.
- Condición de la regla → WHEN. Resultado → THEN.
- Cubrí cuando aplique: autenticado, token expirado/refresh, API no disponible, datos inválidos,
  gating por nivel de identidad.
- Si `--trace`: agregá `<!-- BR-XX -->` junto al requirement (no rompe render ni validación).

### Validación (incremental)

- Validá cada spec al escribirla; corregí hasta verde. Para escala, una capability por pasada.
- Fidelidad: describí el comportamiento como ES HOY. Si un hallazgo muestra código≠intención,
  marcá que spec y código divergen hasta el fix.

## Phase 3: Report

- `openspec validate --specs` → debe dar N passed, 0 failed.
- `openspec list --specs` → capabilities + conteo de requirements (cruzá contra lo esperado:
  un scenario que "desaparece" delata un `####` mal puesto).
- Reportá: archivos creados, requirements por capability, comportamiento del XML no mapeado,
  y advertencias de fidelidad.
