---
name: verify-business-logic
description: >
  Verifica que las reglas de negocio de un documento extraído existen
  realmente en el código. Por cada regla, va al source declarado y confirma
  si el archivo, la función y la lógica coinciden. Úsese para validar o
  auditar un documento de lógica de negocio contra el código fuente.
argument-hint: "ruta al documento extraído (xml o md)"
allowed-tools: Read, Grep, Glob
---
## What this skill does
Given an extracted business-logic document, verify each business rule against
the real source code. You do NOT re-extract or discover new logic; you only
check what the document claims.

## Steps

1. Read the document at the given path and locate every `<rule>` (or, in MD,
   each rule line). For each one, capture: its id, its statement, and its
   declared `source` (file → function/guard).

2. For each rule, verify three things against the codebase:
   a. FILE — does the file in `source` exist?
   b. SYMBOL — does the function/guard/method named in `source` exist in it?
   c. LOGIC — does that code actually do what the statement claims?

3. Assign a verdict per rule:
   - CONFIRMED   — file, symbol and logic all match.
   - BAD_SOURCE  — the rule is real but the source points to the wrong place.
   - NOT_FOUND   — no code supports the statement (possible hallucination).

4. Output a report: a table with id, verdict, and a short note. End with a
   summary line (how many confirmed / bad_source / not_found out of total).

## Boundaries
- Read-only. Never modify code or the document.
- Do NOT invent rules or look for missing ones — this pass only checks the
  rules already in the document (forward pass only).
- If a verdict is uncertain, mark it for human review rather than guessing
  CONFIRMED.
