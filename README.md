# business-logic-toolkit

Plugin de [Claude Code](https://claude.com/claude-code) para **extraer y verificar la lógica de negocio** de una aplicación ya desarrollada: entidades, reglas de negocio, flujos de usuario e integraciones, con trazabilidad al código fuente.

Pensado para documentar, mapear o auditar apps existentes (o preparar una propuesta de mejora sobre ellas) sin inventar nada: cada regla apunta al archivo y función donde vive.

## Qué incluye

| Tipo | Nombre | Para qué |
|------|--------|----------|
| Comando | `/business-logic [carpeta] [xml\|md]` | Extrae la lógica de negocio a un documento (XML por defecto). |
| Comando | `/validate-business-logic [ruta-doc]` | Audita un documento extraído contra el código. |
| Comando | `/conventions [carpeta]` | Extrae las convenciones técnicas cruzadas con la guía oficial del framework. |
| Comando | `/fix` | Entrevista estructurada para arreglar un bug (gate de claridad); delega a `fix-dev`. |
| Comando | `/baseline [--trace]` | Genera el baseline de specs vivas de OpenSpec desde `business-logic.xml`. |
| Comando | `/from-jira <ISSUE-KEY> [--no-writeback]` | Convierte una tarea de Jira en un change de OpenSpec, con gate de readiness antes de generar. |
| Subagente | `business-analyst` | Extracción de lógica de negocio (read-only, basado en evidencia). |
| Subagente | `business-logic-verifier` | Audita cada regla: CONFIRMED / BAD_SOURCE / NOT_FOUND. |
| Subagente | `conventions-analyst` | Extrae convenciones técnicas + delta vs. guía oficial (read-only). |
| Subagente | `fix-dev` | Aplica un fix desde un Fix-Brief confirmado. |
| Subagente | `baseline-generator` | Genera specs vivas de OpenSpec desde el XML y las valida. |
| Subagente | `change-author` | Genera los artefactos de un change de OpenSpec desde un Change-Brief confirmado. |
| Skill | `extracting-business-logic` | Procedimiento de extracción en 3 fases (descubrimiento → extracción → cobertura). |
| Skill | `verify-business-logic` | Procedimiento de verificación regla por regla. |
| Skill | `extracting-conventions` | Extracción de convenciones + catálogos por stack. |
| Skill | `intake-fix` | Entrevista de fix con gate de claridad (Fix-Brief). |
| Skill | `generating-baseline-specs` | Procedimiento del baseline de specs de OpenSpec. |
| Skill | `jira-intake` | Motor de `/from-jira`: fetch + mapeo + gate de readiness para SDD. |

> **Integración con OpenSpec** (`/baseline`, `/from-jira`): requieren el CLI de OpenSpec
> (`@fission-ai/openspec`) instalado y el proyecto inicializado (`openspec/`). `/baseline` es
> downstream de `/business-logic` (deriva specs vivas desde `docs/business-logic.xml`). `/from-jira`
> además necesita el connector de Atlassian (Jira) autorizado, y mide con un gate de readiness si la
> tarea está lista para SDD antes de generar el change.

## Cómo funciona

La extracción corre en 3 fases:

1. **Descubrimiento** — detecta el stack, aplica conteos deterministas (Glob/Grep) y muestra un **manifiesto** (qué secciones se activan y qué quedó fuera). Se detiene a esperar tu confirmación.
2. **Extracción** — completa el esquema canónico (XML) contra el manifiesto. Cada regla de negocio debe tener un `source` al código o no se incluye.
3. **Cobertura** — compara lo esperado vs. lo extraído y reporta gaps.

El resultado se guarda en `docs/` del proyecto analizado. Luego podés auditarlo con `/validate-business-logic`.

> **Stacks soportados:** hoy hay catálogo de detección para **Angular + NestJS**
> (`skills/extracting-business-logic/references/catalog-angular-nest.md`).
> Para otros stacks, agregá un nuevo archivo de catálogo siguiendo ese formato;
> si no hay catálogo, la skill se detiene en vez de adivinar patrones.

## Instalación

### Vía marketplace (git)

```
/plugin marketplace add <git-url-de-este-repo>
/plugin install business-logic-toolkit@dev-toolkit-plugins
```

### Local (desarrollo)

```
/plugin marketplace add /ruta/a/business-logic-toolkit
/plugin install business-logic-toolkit@dev-toolkit-plugins
```

Verificá con `/plugin` que aparezca instalado y reiniciá la sesión si los comandos no figuran todavía.

## Uso

```
/business-logic . xml          # extrae todo el proyecto a docs/business-logic.xml
/business-logic src/app md     # extrae solo una carpeta, en Markdown
/validate-business-logic docs/business-logic.xml
```

## Estructura del plugin

```
business-logic-toolkit/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── commands/
│   ├── business-logic.md
│   ├── validate-business-logic.md
│   ├── conventions.md
│   ├── fix.md
│   ├── baseline.md
│   └── from-jira.md
├── agents/
│   ├── business-analyst.md
│   ├── business-logic-verifier.md
│   ├── conventions-analyst.md
│   ├── fix-dev.md
│   ├── baseline-generator.md
│   └── change-author.md
├── skills/
│   ├── extracting-business-logic/
│   ├── verify-business-logic/
│   ├── extracting-conventions/
│   ├── intake-fix/
│   ├── generating-baseline-specs/
│   └── jira-intake/
└── evals/
    ├── README.md
    ├── business-logic/
    ├── conventions/
    ├── fix/
    ├── baseline/
    └── from-jira/
```

## Proyectos relacionados

| Repo | Qué es |
|------|--------|
| [MCP-PAINT](https://github.com/miguel-alvarez-utn2020/MCP-PAINT) | MCP local que controla Paint de Windows via pyautogui — ejemplo práctico de cómo construir un servidor MCP desde cero con Python + FastMCP, incluyendo auto-calibración por visión artificial (OpenCV) y reproducción de bocetos. |

## Licencia

MIT
