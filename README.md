# business-logic-toolkit

Plugin de [Claude Code](https://claude.com/claude-code) para **entender, documentar y mantener** apps ya desarrolladas — sin inventar nada: todo apunta al archivo y función donde vive.

Hace cuatro cosas:
1. **Extrae la lógica de negocio** — *qué hace* la app (entidades, reglas, flujos).
2. **Extrae las convenciones técnicas** — *cómo está construida* (arquitectura, estado, estilos…), cruzadas con la guía oficial del framework.
3. **Guía arreglos de bugs** (`/fix`) con una entrevista estructurada, pensada para que hasta alguien no técnico (ej. un QA) pueda dirigir un fix con precisión.
4. **Conecta con OpenSpec (SDD)** — genera el *baseline* de specs vivas desde la lógica extraída (`/baseline`) y convierte tareas de Jira en *changes* de OpenSpec con un gate de readiness (`/from-jira`).

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

### `/baseline [--trace]`
**Qué hace:** genera el *baseline* de specs vivas de OpenSpec a partir de `docs/business-logic.xml` — mapea flujos y reglas a *capabilities* y escribe una `spec.md` por capability, validada con el CLI de OpenSpec.
**Cuándo usarlo:** en una app existente, después de `/business-logic`, para sembrar `openspec/specs/` y empezar a trabajar spec-driven (cada cambio futuro es un *delta* contra esa base).
**Produce:** `openspec/specs/<capability>/spec.md` (una por capability).
**Requisito:** CLI de OpenSpec (`@fission-ai/openspec`) instalado + proyecto inicializado (`openspec init`), y `docs/business-logic.xml` ya verificado.
**Cómo trabaja:** *descubrimiento* (propone el mapa de capabilities y **para y espera tu OK**) → *generación* → *validación* (`openspec validate`). Con `--trace` agrega la referencia a las reglas (BR) dentro de cada spec.

### `/from-jira <ISSUE-KEY> [--no-writeback]`
**Qué hace:** convierte una tarea de Jira en un *change* de OpenSpec. Trae el issue por MCP, lo mapea a un *Change-Brief* y mide con un **gate de readiness** si la tarea está lista para SDD; si le falta info (sobre todo criterios de aceptación), **frena y pide aclaración** en vez de generar un spec pobre. Al confirmar, delega la generación al subagente `change-author`.
**Cuándo usarlo:** para que el "prompt" de trabajo sea la tarea definida en Jira, no un texto suelto.
**Produce:** un change en `openspec/changes/<change-id>/` (proposal/specs/design/tasks). Opcional (con confirmación): comenta y transiciona el issue en Jira (*write-back*).
**Requisito:** CLI de OpenSpec + connector de Atlassian (Jira) autorizado (desde la config de connectors de claude.ai). Para **leer el contenido de los adjuntos** del ticket (PDF de marca, mockups), además un token de API scoped (ver skill `jira-read`); si no está, un preflight ofrece configurarlo o seguir solo-texto (degradado).
**Cómo cuida la calidad:** el gate puntúa objetivo, alcance, criterios de aceptación, anclaje técnico y encaje con el dominio (umbral 85%); no genera con input turbio ni inventa criterios en silencio. Cuando hay token, resuelve dimensiones también desde los adjuntos de diseño (el spec suele vivir en el PDF/mockup, no en el texto). Después del change, seguís con el flujo nativo de OpenSpec (`/opsx:apply` → `/opsx:archive`).

### `jira-read` (skill) — leer una tarea de Jira con sus adjuntos
**Qué hace:** trae una issue por key o URL `/browse/` **incluido el contenido de sus adjuntos** (imágenes, PDF, .md, .txt), no solo la metadata. Sirve standalone ("leé la tarea X") y es la capa que usa `/from-jira` para los adjuntos.
**Cuándo usarlo:** para leer/entender una tarea que tiene diseño o specs adjuntos.
**Requisito:** token de API scoped (`read:jira-work` + `read:attachment:jira`) guardado en env (`JIRA_EMAIL`/`JIRA_API_TOKEN`/`JIRA_SITE`) — setup en `skills/jira-read/references/jira-token-setup.md`. Gate duro: sin token válido, frena y guía el setup.
**Por qué el token:** el MCP de Atlassian trae metadata del adjunto pero no el binario; el contenido real exige token propio pegándole al dominio del sitio.

---

## 🔷 Flujo recomendado (para un proyecto nuevo)

```
1. /business-logic .      → entender QUÉ hace la app        (docs/business-logic.xml)
2. /conventions .         → entender CÓMO está construida   (docs/conventions.md)
3. /fix                   → arreglar bugs con esos contextos ya cargados
   /validate-business-logic docs/business-logic.xml  → auditar el doc cuando haga falta
```

Los pasos 1 y 2 se corren **una vez** por proyecto (quedan en `docs/`). Después `/fix` los usa como contexto.

### Extensión con OpenSpec (spec-driven)

Si además vas a trabajar con OpenSpec, sobre los pasos 1-2:

```
4. /baseline .            → sembrar specs vivas desde business-logic.xml   (openspec/specs/)
5. /from-jira <ISSUE-KEY> → convertir una tarea de Jira en un change de OpenSpec
   → seguís con /opsx:apply y /opsx:archive (flujo nativo de OpenSpec)
```

Requiere el CLI de OpenSpec instalado y el proyecto inicializado (`openspec init`).
`/from-jira` además necesita el connector de Atlassian (Jira) autorizado.

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
| Skill | `jira-read` | Lee una tarea de Jira por key/URL **incluido el contenido de sus adjuntos** (PDF/imágenes/.md); requiere token scoped. |

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
│   ├── jira-intake/
│   └── jira-read/
└── evals/
    ├── README.md
    ├── business-logic/
    ├── conventions/
    ├── fix/
    ├── baseline/
    ├── from-jira/
    └── jira-read/
```

## Proyectos relacionados

| Repo | Qué es |
|------|--------|
| [MCP-PAINT](https://github.com/miguel-alvarez-utn2020/MCP-PAINT) | MCP local que controla Paint de Windows via pyautogui — ejemplo práctico de cómo construir un servidor MCP desde cero con Python + FastMCP, incluyendo auto-calibración por visión artificial (OpenCV) y reproducción de bocetos. |

## Licencia

MIT
