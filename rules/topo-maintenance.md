---
paths:
  - "src/**/*"
  - "docs/**/*.md"
  - "config/**/*"
---

# TOPO Maintenance Rule

AFTER creating, modifying, or removing ANY solution component, you MUST update
`docs/solution-map.topo` following the SOPs in `docs/topo-standard-operating-procedures.md`.

This rule is platform-agnostic. It applies regardless of technology stack.

## Quick Checklist

1. Does `docs/solution-map.topo` exist?
   - No + new project: create from template (SOP-1)
   - No + existing project: warn and offer to create

2. Did you just create a component?
   - Data entity > SOP-2: add `[DE]` block
   - Server logic > SOP-3: add `[SL]` block, update parent `[DE]`, set parity if applicable
   - Client logic > SOP-4: add `[CL]` block, update parent `[DE]`, SET PARITY (mandatory)
   - API > SOP-5: add `[API]` block, update backing `[SL]`
   - Shared function > SOP-6: add `[SF]` block
   - Background process > SOP-7: add `[BP]` block, update trigger entity
   - UI control > SOP-8: add `[UC]` block, update parent `[DE]`
   - Config pattern > SOP-9: add `[CFG]`, `[DE]` (config), `[SL]` (handler)
   - Integration boundary > SOP-10: add `[IB]` with all three layers

3. Did you add a dependency (A now reads/calls/triggers B)?
   - SOP-11: add B to A's `depends_on`, add A to B's `depended_on_by`

4. Did you remove a dependency?
   - Remove from BOTH sides

5. Did you add a logic-driving field?
   - Add `[DF]` entry under parent `[DE]` key-fields

## Definition of Done

A component change is not complete until:
- The TOPO map block exists and is accurate
- All dependency tables are updated on both sides
- Parity is set on all client logic components
- The Mermaid graph reflects the current state
