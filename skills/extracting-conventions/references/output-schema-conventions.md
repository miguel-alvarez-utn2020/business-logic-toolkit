# Output Schema — Convenciones del proyecto (docs/conventions.md)

El documento es **markdown** (lo consume `fix-dev`, que lee markdown, y humanos).
Es un reference operativo: "cómo se hace cada cosa acá". Cada categoría combina
tres bloques: convención del proyecto (evidencia), referencia oficial (citada), y
delta.

## Encabezado (siempre)

```markdown
# Convenciones técnicas — <app>

- **Stack:** <framework> <versión exacta>   ·   **Generado:** YYYY-MM-DD
- **Guía oficial de referencia:** <url(s)> (versión <X>)
- **Cómo usar este doc:** para un cambio, seguí la convención del PROYECTO
  (consistencia). Donde diga "sin patrón", caé en lo oficial. Las "desviaciones"
  son cómo está hecho hoy — respetalas en un fix, no las refactorices sin pedir.
```

## Por cada categoría

```markdown
## <Categoría>   ·   delta: alineado | desviación | sin patrón

**En este proyecto:** <patrón dominante, en una o dos frases>.
- Ejemplo: `ruta/al/archivo.ts` → <qué muestra>
- (excepción/inconsistencia, si hay): <...>

**Oficial (<framework> <versión>):** <qué recomienda> — <url>

**Delta:** <alineado / desviación / sin patrón> — <explicación corta. Si es
desviación, decí si parece deuda o decisión intencional, sin corregirla>.
```

## Categorías (incluir solo las que el catálogo activó)
- **Arquitectura** — standalone vs módulos, estructura de carpetas, lazy loading.
- **Estado** — store (NgRx) / signals / BehaviorSubject; cuál domina y cuándo.
- **Datos / API** — services + config de URLs, manejo de errores, realtime/SSE.
- **Componentes** — anatomía, naming, change detection, inputs/outputs, librerías UI.
- **Forms / validación** — reactive vs template, patrón de validators (duplicación?).
- **UI / estilos** — toasts, modales, badges, SCSS/tokens.
- **Routing / guards** — estructura de rutas, guards, resolvers.
- **Tooling** — lint/format, convención de commits, testing.

## Reglas
- Toda convención del proyecto lleva al menos un `source` (archivo → patrón). Sin
  evidencia, no se afirma (se marca "sin patrón claro").
- La referencia oficial SIEMPRE lleva URL y versión. Si no se pudo verificar online,
  aclarar "(no verificado online)".
- El delta es la pieza de más valor: es lo que le dice a un fix "seguí el proyecto"
  y al rework "convergé a lo oficial".
- No incluir categorías que el catálogo no activó ni inventar secciones nuevas.
