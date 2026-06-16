# Detection Catalog — Angular + NestJS

Defines how Phase 1 (Discovery) detects and counts each section for this
stack. Selected by the Phase 1 router when `angular.json` + `nest-cli.json`
are present.

## Rules for using this catalog
- Glob counts are hard denominators, EXCEPT where a note marks a Grep pattern
  as authoritative (see the entity detection note).
- Activate an optional section only if its pattern matches at least once.
- Names of instances come from filenames OR from the class name when detected
  by content (e.g. `@Entity() class Invoice` → entity "Invoice").
- Extend by ADDING rows. Never let the skill invent sections outside this list.

## Detection methods
- **Glob** (hard count): matches files by name. The count is an exact,
  deterministic denominator — use it for Phase 3 coverage validation.
- **Grep** (signal): searches file *contents* for a pattern (e.g. a
  decorator). Activates a section; its count is approximate, NOT a hard
  denominator.

## Entity detection note (overrides the general method rule)
For entities in NestJS, the `@Entity(` decorator is an unambiguous marker:
a file containing it IS an entity regardless of its filename. Therefore:
- The `@Entity(` **Grep** count is AUTHORITATIVE — it is the hard denominator.
- The `**/*.entity.ts` **Glob** is secondary. Use it only to cross-check.
- If Grep count > Glob count, the extra files are off-convention entities:
  list them and include them. Never report fewer entities than the Grep count.

## Core sections (always active)
| Section        | Stack   | Method | Pattern                                        |
|----------------|---------|--------|------------------------------------------------|
| overview       | —       | —      | synthesized by the model, never counted        |
| entities       | Nest    | Grep   | `@Entity(`  (authoritative — see entity note)  |
| entities       | Nest    | Glob   | `**/*.entity.ts`, `**/*.schema.ts`  (secondary) |
| entities       | Angular | Glob   | `**/*.model.ts`, `**/models/**/*.ts`           |
| business-rules | Nest    | Glob   | `**/*.service.ts`                              |
| business-rules | both    | Glob   | `**/*.guard.ts`, `**/*.validator.ts`           |
| user-flows     | Nest    | Glob   | `**/*.controller.ts`                           |
| user-flows     | Angular | Grep   | `**/*.routes.ts`, `RouterModule`               |

## Optional sections (activate only if its pattern matches ≥1)
| Section         | Method | Pattern                                            |
|-----------------|--------|----------------------------------------------------|
| roles           | Grep   | `@Roles(`, `CanActivate`, `casl` in package.json   |
| integrations    | Grep   | SDKs in package.json (stripe, twilio, aws-sdk,     |
|                 |        | @sendgrid…), `**/*.client.ts`                      |
| state-machines  | Glob   | `**/*.machine.ts`, `xstate` in package.json        |
| scheduled-jobs  | Grep   | `@Cron(`, `@Interval(`, `@Timeout(`                |
| domain-events   | Grep   | `@OnEvent(`, `@nestjs/cqrs`, `**/*.event.ts`       |
| realtime        | Grep   | `@WebSocketGateway(`, `**/*.gateway.ts`            |
| queues          | Grep   | `@Processor(`, `bullmq` / `@nestjs/bull`           |
