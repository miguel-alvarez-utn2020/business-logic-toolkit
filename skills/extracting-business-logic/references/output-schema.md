# Output Schema — Canonical Business Logic (stack-agnostic)

The XML is the single source of truth. The `md` format is rendered FROM
this XML, never extracted separately. This schema is universal: it applies
to any stack (Angular/Nest, React, etc.) because it describes WHAT the
business is, not HOW it is found in the code.

## Schema rules
- Core sections always present: overview, entities, business-rules, user-flows.
- Optional sections appear ONLY if the manifest activated them.
- Every entity, rule and flow has a stable `id`, reused for cross-references.
- Every `<rule>` MUST have a `<source>` (file → function). No source = omit the rule.
- Never introduce tags outside this schema.
- A `<rule>` must be a real domain constraint (validation, authorization or a
  conditional gate). Defaults and unconditional state writes are NOT rules.
## Structure

```xml
<business-logic app="app-name" extracted-at="YYYY-MM-DD">
  <overview>
    <purpose>What the app does and for whom</purpose>
    <domain>The business domain in one sentence</domain>
  </overview>

  <entities>
    <entity id="order" name="Order">
      <description>...</description>
      <fields>
        <field name="status" type="enum" required="true"/>
      </fields>
      <relationships>
        <relationship type="has-many" target="line-item"/>
      </relationships>
    </entity>
  </entities>

  <business-rules>
    <rule id="BR-01" applies-to="order" type="validation">
      <statement>An order cannot be confirmed without available stock</statement>
      <source>order.service.ts → checkout()</source>
    </rule>
  </business-rules>

  <user-flows>
    <flow id="checkout" name="Checkout" actor="customer">
      <steps>
        <step order="1">Customer adds products to the cart</step>
      </steps>
      <involves entities="order,product" rules="BR-01"/>
    </flow>
  </user-flows>

  <!-- Optional sections (include ONLY if activated by the manifest):
       roles, integrations, state-machines, scheduled-jobs,
       domain-events, realtime, queues -->
</business-logic>
```

## Markdown rendering (when format = md)
- One heading per section.
- Each rule as a line: **statement** _(source)_.
- Preserve ids in parentheses so the human version stays traceable.
- Render only the sections present in the XML (don't add empty ones).
