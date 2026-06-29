# Detection Catalog — Angular (frontend, sin backend)

Defines how Phase 1 (Discovery) detects and counts each section for an Angular
frontend application WITHOUT a NestJS backend in the same repo. Selected by the
Phase 1 router when `angular.json` is present and `nest-cli.json` is NOT.

## Rules for using this catalog
- Glob counts are hard denominators, EXCEPT where a note marks a Grep pattern
  as authoritative.
- Activate an optional section only if its pattern matches at least once.
- Names of instances come from filenames OR from the interface/class name when
  detected by content (e.g. `interface User` → entity "User").
- Extend by ADDING rows. Never let the skill invent sections outside this list.

## Detection methods
- **Glob** (hard count): matches files by name. The count is an exact,
  deterministic denominator — use it for Phase 3 coverage validation.
- **Grep** (signal): searches file *contents* for a pattern. Activates a
  section; its count is approximate, NOT a hard denominator.

## Entity detection note (frontend)
In a frontend there are no `@Entity` decorators. Domain entities live as
TypeScript **interfaces, types or model classes** (often under `interfaces/` or
`models/`). Therefore:
- The Glob over `**/*.interface.ts`, `**/*.model.ts`, `**/models/**/*.ts` and
  `**/*.dto.ts` is the **candidate denominator**.
- NOT every interface is a domain entity: transport/UI shapes (e.g.
  `api-response.interface.ts`, `*.dto.ts` wrappers, component @Input props) are
  candidates that Phase 2 must FILTER OUT. Report the raw candidate count in the
  manifest, and in Phase 2 keep only the real domain entities, listing which
  candidates were discarded and why.

## Business-rule sources note (frontend)
A frontend's business rules live mainly in:
- **services** (`*.service.ts`) — orchestration, conditional gating, derived state.
- **guards** (`*.guard.ts`) — route authorization.
- **form validators** — custom `ValidatorFn` / `Validators` with domain conditions.
They MAY also live inside components (conditional UI gating). Components are too
many to be a denominator, so they are NOT counted as a section; Phase 2 may still
cite a component file in a rule `source` when that is where the rule actually lives.
Remember the core definition: only real domain constraints are rules, never
framework plumbing or unconditional state writes.

## Core sections (always active)
| Section        | Method | Pattern                                                        |
|----------------|--------|----------------------------------------------------------------|
| overview       | —      | synthesized by the model, never counted                        |
| entities       | Glob   | `**/*.interface.ts`, `**/*.model.ts`, `**/models/**/*.ts`, `**/*.dto.ts`  (candidates — see entity note) |
| business-rules | Glob   | `**/*.service.ts`, `**/*.guard.ts`, `**/*.validator.ts`        |
| user-flows     | Glob   | `**/*.routes.ts`                                               |
| user-flows     | Grep   | `RouterModule`, `provideRouter`  (secondary signal)            |

## Optional sections (activate only if its pattern matches ≥1)
| Section         | Method | Pattern                                                       |
|-----------------|--------|---------------------------------------------------------------|
| roles           | Grep   | `canActivate`, `data: { roles`, `permission`/`role` en guards/services, `casl` en package.json |
| integrations    | Grep   | `HttpClient`, URLs de API en `environment.*`, `**/*.client.ts`, SDKs de terceros en package.json (ag-grid, chart.js, file-upload…) |
| realtime        | Grep   | `EventSource`, `new WebSocket`, `**/*-sse.service.ts`, `**/*.sse.service.ts` |
| state-machines  | Glob   | `**/*.machine.ts`, `xstate` en package.json                   |
| domain-events   | Grep   | `**/*.event.ts`, `EventEmitter` con eventos de dominio        |

## Sections that rarely apply in a frontend
`scheduled-jobs` y `queues` casi nunca aplican en un frontend (no hay cron ni
colas en el browser). Activarlas SOLO si hay evidencia real (p. ej. polling
periódico modelado explícitamente). Ante la duda, no las actives.
