# Eval baseline — 01: cobertura, formato y mapeo sobre CiDiApp

- **Qué prueba:** que la generación de baseline (a) proponga un mapa de capabilities correcto
  a partir de `business-logic.xml`, (b) produzca specs que **validan** con el CLI de OpenSpec,
  y (c) mapee cada requirement a la BR/flow correcta (trazabilidad), sin inventar comportamiento.
- **Criticidad:** alta (el baseline es la base contra la que se harán todos los cambios SDD).

## Setup
- App: `gob-cba-cidi-app-mobile` (CiDiApp, React Native 0.81, catálogo `catalog-react-native`).
- Precondición: existe `docs/business-logic.xml` verificado (13 entidades, 20 BR, 16 user-flows).
- OpenSpec inicializado (`openspec/`, schema `spec-driven`), CLI `@fission-ai/openspec` ≥1.5.
- **Escritura:** este eval NO es read-only puro (la generación escribe `openspec/specs/`).
  Correr sobre un worktree/carpeta descartable, o limpiar al final (ver Limpieza).

## Input
- Ejecutar `/baseline` (Fase 1 discovery → confirmar mapa → Fase 2 generación → Fase 3 validación).
- Para la parte read-only (ver Variante), pedir SOLO la Fase 1 (manifiesto) sin generar.

## Expected

### Objetivos (verificables sin juicio)
- [ ] `openspec validate --specs` termina con **N passed, 0 failed** (todas las specs válidas).
- [ ] Se crea una carpeta `openspec/specs/<capability>/spec.md` por capability confirmada.
- [ ] Toda spec usa el formato correcto: `## Requirements`, `### Requirement:`, `#### Scenario:`
      (4 hashes), con `- **WHEN**` / `- **THEN**`. NINGUNA usa `## ADDED Requirements` (eso es delta).
- [ ] Cada requirement tiene ≥1 scenario (el conteo de `openspec list --specs` coincide con lo esperado
      → ningún scenario "desaparecido" por `####` mal puesto).
- [ ] Cubre como mínimo las capabilities de dominio: autenticación, biometría, cuenta, documentos,
      credenciales, notificaciones, servicios, búsqueda, chatbot (≈9; agrupación puede variar).

### De juicio (spot-check directo, read-only — NO requiere orquestar otra tool)
- [ ] **Mapeo correcto:** en el sample, cada requirement rastrea a la BR/flow declarada en el XML.
      Confirmá 4 samples leyendo `business-logic.xml`:
      - login CUIL/password ← BR-01 · sesión/refresh ← BR-02 · gating credenciales Nivel 2 ← BR-12 ·
        límite de 3 descargas ← BR-13.
- [ ] **Sin alucinaciones:** no hay requirements de comportamiento que el XML no tenga
      (ej. nada de pagos, i18n, realtime — están como "NO detectado" en el XML).
- [ ] **Filtro de seguridad manejado:** la capability de credenciales NO se llama `credentials`
      (nombre bloqueado por el host); usa una alternativa de dominio (ej. `credenciales`) y el
      manifiesto de Fase 1 avisó del bloqueo ANTES de escribir.
- [ ] **Fidelidad:** si el spec de credenciales redacta la intención del gating por nivel, el reporte
      marca la divergencia con el hallazgo `isLevelOne` (código≠intención) — no lo oculta.

### Del gate (Fase 1)
- [ ] El comando MOSTRÓ el mapa de capabilities y PARÓ esperando confirmación antes de generar
      (no generó specs sin confirmación).

## Variante read-only (paralelizable, sin cleanup)
Correr SOLO la Fase 1: dado `business-logic.xml`, ¿propone el mapa de capabilities correcto,
detecta el nombre bloqueado `credentials`, y lista el comportamiento no mapeado? Se califica el
manifiesto sin escribir nada → corre en paralelo sin colisiones.

## Runs / umbral
- 2 corridas (la generación es relativamente determinista dado el XML).
- **2/2** en objetivos (validate en verde, formato, cobertura de capabilities).
- **2/2** en el gate de Fase 1 (crítico: no generar sin confirmar).
- **2/2** en los 4 samples de mapeo; sin alucinaciones en el sample.

## Limpieza
- Si se corrió la generación: `rm -rf openspec/specs/*` (o `git checkout -- openspec/specs` si estaba
  versionado) al terminar. La variante read-only no requiere limpieza.
