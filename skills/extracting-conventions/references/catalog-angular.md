# Detection Catalog — Convenciones Angular

Define cómo detectar el patrón dominante de cada categoría en un proyecto Angular,
y cuáles son las fuentes oficiales a consultar. Seleccionado cuando hay
`angular.json`. La detección es por stack; el esquema de salida es universal.

## Fuentes oficiales (Fase 2 — traer la versión detectada)
- **Style guide:** https://angular.dev/style-guide
- **Signals:** https://angular.dev/guide/signals
- **Control flow (@if/@for/@switch):** https://angular.dev/guide/templates/control-flow
- **Standalone / componentes:** https://angular.dev/guide/components
- **Forms:** https://angular.dev/guide/forms
- **Routing:** https://angular.dev/guide/routing
> Para versiones < 16, varias de estas APIs no existen (signals, control flow,
> standalone por defecto): traer la guía acorde a la versión y NO recomendar APIs
> que esa versión no tiene. `angular.io` cubre las versiones viejas.

## Categorías y señales de detección

| Categoría | Qué muestrear / Grep | Qué determina el patrón |
|-----------|----------------------|-------------------------|
| Arquitectura | `**/*.routes.ts`, `**/*.module.ts`, `standalone:` en componentes, `loadChildren/loadComponent` | standalone vs NgModules; carpetas core/modules; lazy loading |
| Estado | `@ngrx` en package.json, `**/*.state.ts`/`**/*.store.ts`, `BehaviorSubject`, `signal(`/`computed(` | store vs services-con-subject vs signals; cuál domina |
| Datos / API | `**/*.service.ts`, `HttpClient`, archivo de settings/URLs, `EventSource`/SSE, manejo de error (`catchError`) | patrón de service, centralización de URLs, manejo de error/toast |
| Componentes | `**/*.component.ts`, `ChangeDetectionStrategy`, `@Input/@Output` vs `input()/output()`, librerías UI en package.json | anatomía, OnPush, naming, ag-grid/primeng/material |
| Forms / validación | `ReactiveFormsModule`/`FormBuilder`, `ValidatorFn`, `**/validators/**`, validadores inline en componentes | reactive vs template; dónde viven los validators; duplicación |
| UI / estilos | `ngx-toastr`, `MatDialog`/modales, `**/*.scss`, tokens/variables | toasts, modales, sistema de estilos |
| Routing / guards | `**/*.guard.ts`, `CanActivate`/functional guards, `**/*.resolver.ts` | guards de clase vs funcionales, resolvers |
| Tooling | `.eslintrc*`/`eslint.config.*`, `.prettierrc*`, `commitlint`, `**/*.spec.ts`, husky | lint/format, convención de commits, testing |

## Notas
- **Patrón dominante, no inventario:** si hay 2 formas (ej. guards de clase y
  funcionales), reportá la que predomina y anotá la otra como excepción.
- **Versión manda en lo oficial:** si el proyecto está en Angular 16 con NgModules
  y la versión ya soporta standalone, eso es un `delta = desviación` (no error):
  documentalo, no lo corrijas.
- **Inconsistencias reales = oro:** validación duplicada, mezcla de patrones de
  estado, etc. — anotarlas en la categoría correspondiente como excepción/deuda.
