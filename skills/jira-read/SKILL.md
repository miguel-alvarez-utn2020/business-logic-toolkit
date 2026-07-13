---
name: jira-read
description: >
  Lee una tarea de Jira a partir de su key (ej. "CIDI-7449") o su URL
  (https://<sitio>/browse/CIDI-7449): trae la issue completa (descripción, comentarios,
  estado) vía el MCP de Atlassian, y además descarga y lee el CONTENIDO de sus adjuntos
  — imágenes, PDF, .md, .txt — usando un token de API scoped propio. Se activa cuando el
  usuario pide "leer/ver la tarea <KEY>", pega una key tipo LETRAS-NÚMEROS, o pega un link
  /browse/ de Jira. No es específica de ningún repo.
allowed-tools: Read, Bash
metadata:
  version: "1.1"
---

Trae una issue de Jira **con el contenido de sus adjuntos leído**, no solo su metadata.
Aplica a cualquier proyecto/sitio de Atlassian Cloud.

**Disparador:** el usuario escribe "podés ver la tarea CIDI-7449", pega una key `[A-Z]+-\d+`,
o pega una URL `https://<sitio>/browse/<KEY>`. Si vino como URL, extraé la key.

## Por qué hace falta un token propio (además del connector)

Las tools MCP de Atlassian (`getJiraIssue`, etc.) traen la **metadata** del adjunto (nombre,
tamaño, mimeType, `content` URL) pero **no descargan el binario**. El campo `content` apunta al
gateway `api.atlassian.com/ex/jira/.../attachment/content/{id}`, que es **solo OAuth 3LO** — no
funciona con Basic Auth, y `WebFetch` da 403. Para el contenido real hace falta un token scoped
propio pegándole al **dominio del sitio**.

## Fase 0: Preflight de token (GATE DURO)

Los adjuntos son la razón de ser de este skill → sin token válido, **PARÁ**.
Corré el preflight de 2 niveles de `references/jira-token-setup.md`:
1. **Presencia** de `JIRA_EMAIL` / `JIRA_API_TOKEN` / `JIRA_SITE` (sin imprimir el valor).
2. **Validez**: `curl` a `/rest/api/3/myself` → 200.

Si falta o da 401/403 → mostrá el setup de `references/jira-token-setup.md` y **no ejecutes**.
Nunca pidas el token por chat.

## Pasos (tras preflight OK)

1. **Resolver cloudId**: usar el hostname `$JIRA_SITE` directamente como `cloudId` en las tools
   de Atlassian (solo llamar `getAccessibleAtlassianResources` si falla).

2. **Traer la issue** con `getJiraIssue`:
   ```
   cloudId: "$JIRA_SITE"
   issueIdOrKey: "<KEY>"
   fields: ["summary","description","status","issuetype","priority","labels",
            "assignee","reporter","created","updated","comment","attachment"]
   ```
   Presentar un resumen: título, tipo, prioridad, estado, asignado/reportado, descripción y comentarios.

3. **Filtrar adjuntos leíbles** de `fields.attachment[]` por mimeType/extensión:
   - Imágenes: `image/*` (png, jpg, jpeg, gif, webp, svg)
   - Documentos: `application/pdf`, `.md`, `.txt`
   - Otros (`.docx`, `.xlsx`, `.zip`…): avisar explícitamente que NO se procesan (no en silencio).

4. **Descargar cada adjunto filtrado** al scratchpad de la sesión (nunca al repo):
   ```bash
   curl -s -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
     "https://$JIRA_SITE/rest/api/3/attachment/content/<ID>" -o "<scratchpad>/<filename>"
   ```
   (Windows/PowerShell: ver `references/jira-token-setup.md`.)

5. **Leer cada archivo** con `Read`:
   - Imágenes y PDFs: `Read` los procesa directo (incluso PDFs de varias páginas; no asumir que necesita poppler).
   - `.md`/`.txt`: como texto plano.
   - **PNGs con transparencia** (logos blancos con alpha) que se ven vacíos: componer sobre fondo
     oscuro antes de leer. macOS: `sips` o ImageMagick (`convert in.png -background '#14285A' -flatten out.png`).
     Windows: `System.Drawing` (ver `references/jira-token-setup.md`).

6. **Presentar** el resumen de la issue + un resumen del CONTENIDO de cada adjunto leído
   (no solo "lo descargué").

## Guardrails
- Adjuntos siempre al scratchpad de la sesión, nunca dentro de un repo.
- Nunca imprimir el valor del token ni pedirlo por chat — solo indicar cómo generarlo/guardarlo.
- 401/403 en la descarga → no reintentar a ciegas: avisar (token expirado / sin scopes) y remitir al setup.
- Para specs exactas (colores, medidas) de un PDF/diseño, aclarar que la lectura visual no reemplaza
  confirmar con el archivo vectorial o con quien mandó el asset.
