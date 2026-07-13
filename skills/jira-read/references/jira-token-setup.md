# Setup del token de Jira + preflight

El MCP de Atlassian (connector OAuth de claude.ai) trae el **texto** del issue, pero
**no descarga el binario de los adjuntos**. Para leer el contenido real de un adjunto
(PDF de diseño, mockup PNG, .md) hace falta un **token de API scoped propio** y pegarle
directo al dominio del sitio (no al gateway `api.atlassian.com`).

## 1. Crear el token (una vez)

En https://id.atlassian.com/manage-profile/security/api-tokens (NO en `apps-and-sites`):
- **Nombre**: p.ej. `claude-code-jira-attachments`
- **Scopes (solo lectura):** `read:jira-work` (ver issues/comentarios) + `read:attachment:jira` (bajar adjuntos)
- Sin escritura ni Confluence. Expira ≤ 1 año.

## 2. Guardarlo (nunca en un repo; nunca impreso)

Se guardan 3 variables. **El valor del token vive solo en el entorno del usuario, jamás en un archivo del repo ni en la memoria del agente.**

### macOS / Linux (zsh/bash) — recomendado en este entorno
Agregar al profile (`~/.zshrc` o `~/.bashrc`) y reiniciar la terminal:
```bash
export JIRA_EMAIL="tu-email@dominio.com"
export JIRA_SITE="tuorg.atlassian.net"
export JIRA_API_TOKEN="<token scoped>"
```
*(Opción más segura: macOS Keychain — `security add-generic-password -a "$USER" -s JIRA_API_TOKEN -w "<token>"` y leerlo en runtime con `security find-generic-password -s JIRA_API_TOKEN -w`.)*

### Windows (PowerShell)
```powershell
[Environment]::SetEnvironmentVariable("JIRA_EMAIL", "<email>", "User")
[Environment]::SetEnvironmentVariable("JIRA_SITE", "tuorg.atlassian.net", "User")
[Environment]::SetEnvironmentVariable("JIRA_API_TOKEN", "<token>", "User")
```

## 3. Preflight — chequeo en 2 niveles (lo corre el skill antes de ejecutar)

### Nivel 1 — Presencia (nunca imprime el valor)
```bash
if [ -n "$JIRA_EMAIL" ] && [ -n "$JIRA_API_TOKEN" ] && [ -n "$JIRA_SITE" ]; then echo OK; else echo MISSING; fi
```

### Nivel 2 — Validez (confirma que el token funciona / no expiró / tiene scopes)
```bash
curl -s -o /dev/null -w "%{http_code}" -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  "https://$JIRA_SITE/rest/api/3/myself"
# 200 → válido · 401/403 → expirado o sin scopes → volver al paso 1-2
```

### Interpretación
| Resultado | Acción |
|---|---|
| Nivel 1 = MISSING | Falta configurar → mostrar este setup y **PARAR** (no ejecutar). |
| Nivel 1 OK, Nivel 2 = 401/403 | Token expirado o sin scopes → remitir al setup y **PARAR**. |
| Nivel 2 = 200 | ✅ "token disponible y válido" → ejecutar. |

## 4. Descargar un adjunto (con el token, al scratchpad)

```bash
curl -s -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  "https://$JIRA_SITE/rest/api/3/attachment/content/<ID>" -o "<scratchpad>/<filename>"
```
Clave: usar el **dominio del sitio** (`https://$JIRA_SITE/...`), NO el gateway `api.atlassian.com`
(ese es solo OAuth 3LO). El `<ID>` sale de `fields.attachment[].id` de `getJiraIssue`.
Guardar SIEMPRE en el scratchpad de la sesión, nunca en el repo.

## Guardrails
- Nunca imprimir el valor del token ni pedirlo por chat — solo indicar cómo generarlo/guardarlo.
- Adjuntos al scratchpad de la sesión, nunca dentro de un repo.
- 401/403 → no reintentar a ciegas: avisar (token expirado / sin scopes) y remitir al setup.
