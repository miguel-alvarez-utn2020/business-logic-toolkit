# business-logic-toolkit

Plugin de [Claude Code](https://claude.com/claude-code) para **entender, documentar y mantener** apps ya desarrolladas — sin inventar nada: todo apunta al archivo y función donde vive.

Hace tres cosas:
1. **Extrae la lógica de negocio** — *qué hace* la app (entidades, reglas, flujos).
2. **Extrae las convenciones técnicas** — *cómo está construida* (arquitectura, estado, estilos…), cruzadas con la guía oficial del framework.
3. **Guía arreglos de bugs** (`/fix`) con una entrevista estructurada, pensada para que hasta alguien no técnico (ej. un QA) pueda dirigir un fix con precisión.

**Stacks soportados hoy:** Angular (+ NestJS), Angular frontend, y React Native / Expo. Para otro stack se agrega un "catálogo" de detección (ver más abajo).

---

## 📖 Guía de comandos (qué hace cada uno)

### `/business-logic [carpeta] [xml|md]`
**Qué hace:** analiza el código y extrae *qué hace la app* — entidades, reglas de negocio, flujos de usuario, roles e integraciones. Cada regla queda trazada a su `source` (archivo → función).
**Cuándo usarlo:** al empezar a trabajar sobre una app que no conocés, para documentarla o antes de escalarla/arreglarla.
**Produce:** `docs/business-logic.xml` (o `.md`).
**Ejemplo:** `/business-logic .` (todo el proyecto) · `/business-logic src/app md` (una carpeta, en Markdown).
**Cómo trabaja:** en 3 fases — *descubrimiento* (detecta el stack y muestra un manifiesto, **para y espera tu OK**) → *extracción* → *cobertura* (reporta qué cubrió y qué no).

### `/validate-business-logic [ruta-doc]`
**Qué hace:** audita un documento ya extraído contra el código real. Por cada regla confirma si existe donde dice.
**Cuándo usarlo:** para verificar que el doc de lógica de negocio es fiel (o auditar uno viejo).
**Produce:** un reporte con cada regla marcada **CONFIRMED** / **BAD_SOURCE** (existe pero el source apunta mal) / **NOT_FOUND**.
**Ejemplo:** `/validate-business-logic docs/business-logic.xml`

### `/conventions [carpeta]`
**Qué hace:** extrae *cómo se construye* en este proyecto (arquitectura, estado, datos/API, componentes, forms, estilos, tooling) y lo **cruza con la guía oficial** del framework para la versión detectada, marcando el **delta** (alineado / desviación / sin patrón).
**Cuándo usarlo:** para que cualquier cambio futuro siga el estilo del equipo, no el que improvise la IA. Es el insumo que hace que `/fix` respete las convenciones.
**Produce:** `docs/conventions.md`.
**Ejemplo:** `/conventions .`
**Regla de uso (queda escrita en el doc):** para un cambio, seguí la convención del proyecto; donde no hay patrón, caé en lo oficial; las "desviaciones" son cómo está hoy — no las refactorices en un fix.

### `/fix [descripción opcional del bug]`
**Qué hace:** conduce una **entrevista guiada** para arreglar un bug. Te hace preguntas simples (de opción múltiple) sobre lo que *observaste*; la parte técnica (dónde está en el código) la resuelve solo. Va mostrando un **% de claridad**; al llegar al umbral arma un **Fix-Brief** (resumen preciso del arreglo) y, tras tu confirmación, delega la ejecución al subagente `fix-dev`.
**Cuándo usarlo:** para arreglar bugs sin escribir un "prompt suelto" — con guía y alta precisión. Ideal para gente con poca experiencia técnica.
**Produce:** un `docs/fixes/<nombre>.md` (el Fix-Brief) + el cambio aplicado.
**Requisito:** conviene tener antes `docs/business-logic.xml` (le da contexto). Si no existe, `/fix` lo avisa.
**Cómo cuida la calidad:** no aplica cambios "a ciegas" — si lo que reportás no se reproduce en el código, **frena** en vez de inventar un arreglo; y cita las reglas/convenciones **leyéndolas** del código, no de memoria.

---

## 🔷 Flujo recomendado (para un proyecto nuevo)

```
1. /business-logic .      → entender QUÉ hace la app        (docs/business-logic.xml)
2. /conventions .         → entender CÓMO está construida   (docs/conventions.md)
3. /fix                   → arreglar bugs con esos contextos ya cargados
   /validate-business-logic docs/business-logic.xml  → auditar el doc cuando haga falta
```

Los pasos 1 y 2 se corren **una vez** por proyecto (quedan en `docs/`). Después `/fix` los usa como contexto.

---

## 🧩 Qué incluye

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

---

## Instalación

Corré **una línea a la vez** (no las pegues juntas):

```
/plugin marketplace add https://github.com/miguel-alvarez-utn2020/business-logic-toolkit.git
```
```
/plugin install business-logic-toolkit@dev-toolkit-plugins
```

Reiniciá la sesión. Verificá con `/plugin` que aparezca **`business-logic-toolkit`** y anotá la **versión** (debe ser la última publicada).

**¿Viene desactualizado?** El marketplace cachea. Forzá refresco:
```
/plugin marketplace update dev-toolkit-plugins
```
Si aún así trae viejo, `remove` + `add` de nuevo (limpia la caché) y reiniciá.

> Nota: `business-logic-toolkit` es el **plugin**; `dev-toolkit-plugins` es el **marketplace** (la parte después del `@`). Si lo agregaste hace mucho, tu marketplace local puede llamarse distinto — mirá `/plugin` y usá ese nombre en el `@`.

---

## Stacks y catálogos

La extracción es **agnóstica en su salida** (describe el negocio/convenciones, no la tecnología), pero la **detección** es por stack, vía "catálogos". Hoy hay:

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
