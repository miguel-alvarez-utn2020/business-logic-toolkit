---
name: extracting-business-logic
description: >
  Analiza una base de código existente para extraer su modelo de negocio:
  entidades, reglas de negocio, flujos de usuario e integraciones, con
  trazabilidad al código fuente. Úsese cuando se pida documentar, mapear o
  entender la lógica de negocio de una app ya desarrollada, o preparar una
  propuesta de mejora.
argument-hint: "formato de salida: xml (default) | md"
allowed-tools: Read, Grep, Glob
---

## Phase 1: Discovery (build the manifest)

Do NOT interpret business logic yet. This phase only inventories what
exists, to define the "denominator" the extraction must cover.

1. Detect the stack and select the catalog. Evaluá en orden; elegí el PRIMERO
   que matchee:
   - `angular.json` + `nest-cli.json` → references/catalog-angular-nest.md
   - `angular.json` sin `nest-cli.json` → references/catalog-angular.md
     (Angular frontend puro: entidades como interfaces/models, sin backend)
   - (other stacks → their own catalog file, when added)
   - If no catalog matches the detected stack, STOP and tell the user which
     stack was detected and that a catalog for it must be added. Do NOT
     guess file patterns.

2. Run the deterministic counts. Apply every Glob/Grep pattern from the
   catalog selected in step 1, and record the exact match count for each.
   These counts are facts, not interpretations.

3. Activate sections. Core sections (overview, entities, business-rules,
   user-flows) are always active. Activate an optional section ONLY if its
   detection pattern returned at least one match.

4. List the instances. For each active section, list the detected items by
   name. Names come from filenames, not from interpretation.

5. Output the manifest before extracting (active sections, per-section
   counts, and explicitly which optional sections were NOT detected).

Example manifest output:
```
Manifest — app: tienda-online
  Active sections: overview, entities, business-rules, user-flows, roles, integrations
  Counts: entities=8, modules=12, controllers=11, guards=3, integrations=1
  Detected entities: order, product, user, cart, payment, address, category, review
  NOT detected: state-machines, scheduled-jobs, domain-events, realtime, queues
```

## Phase 2: Extraction (fill the schema against the manifest)

Now interpret the code. The manifest from Phase 1 is your checklist:
every instance it listed MUST appear in the output. Do not add sections
the manifest did not activate.

Follow the canonical schema in `references/output-schema.md`.

1. For each active section, read the relevant source files and extract the
   fields defined in the schema.

2. Entities: capture name, key fields (name/type/required) and
   relationships to other entities (reuse the ids from the manifest).

### What counts as a business rule
Before extracting, apply this definition. A business rule is a constraint,
validation or authorization the domain enforces and that COULD be violated.

INCLUDE:
   - validations / invariants  ("price must be > 0", "cannot confirm without stock")
   - authorizations            ("only an admin can cancel a paid order")
   - conditional logic gating an action ("X only happens if Y")

EXCLUDE (these are NOT business rules):
   - default values on creation        (e.g. status = 'pending')
   - unconditional state transitions   (e.g. status = 'cancelled' inside cancel())
   - CRUD / persistence / framework plumbing

Heuristic: if removing the line would allow an INVALID domain state, it is a
rule. If the line just sets or initializes state with no guard condition, it is not.

3. Business rules: first apply the "What counts as a business rule"
   definition below to decide what qualifies. Then extract each qualifying
   rule as a clear statement, and for EVERY rule record its `source`
   (file → function/guard/validator). A rule without a source is not
   allowed — if you cannot point to where it lives in the code, do not
   include it.
   ### Rule granularity (one domain constraint = one rule)
   When a single domain constraint is enforced through several collaborating
   services or methods, represent it as ONE rule whose `<source>` lists ALL the
   contributing locations. Do NOT emit one rule per file.

   Heuristic: if several checks must ALL hold to permit ONE domain action, that
   is a single rule about that action — not several rules.

   Example: "an order can only be confirmed if there is stock AND payment
   succeeds", assembled in order.service.ts -> confirm() but delegating to
   cart.service.ts and payment.service.ts, is ONE rule with three sources.

4. User flows: trace each flow as ordered steps and link the entities and
   rules it involves by id (e.g. involves entities="order" rules="BR-01").

5. Cross-reference by id. Reuse the ids so flows, rules and entities point
   to each other.

6. Build the canonical XML first. The XML is the source of truth,
   regardless of the requested format.

7. Render the requested format. If the argument is `md`, convert the XML
   to the human view (one heading per section, each rule as a line with its
   source in parentheses). If `xml` (default), output the XML as-is.

### Boundaries (what NOT to do)
- Do NOT invent rules, entities or relationships not present in the code.
- Do NOT silently skip an instance the manifest listed. If you can't
  extract one, mark it explicitly as incomplete.
- Do NOT modify any file. Read-only analysis.



## Phase 3: Coverage validation (self-check before finishing)

Compare what the manifest expected against what the extraction produced.
This report tells a non-technical reader whether the output is complete
and what, if anything, is missing.

1. Per section, compare manifest count vs extracted count
   (e.g. entities: manifest 8 / extracted 8 → 100%).

2. List gaps explicitly: which specific instances from the manifest are
   missing or were marked incomplete in Phase 2. Name them.

3. Verify every business rule has a `source`. Any rule without one is an
   error to flag (Phase 2 should prevent this — confirm it held).

4. Compute an overall coverage figure. If it is below a sensible floor,
   say so clearly instead of presenting a partial result as complete.

5. Append the coverage report to the output.
