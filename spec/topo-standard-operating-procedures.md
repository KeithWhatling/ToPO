---
title: TOPO — Standard Operating Procedures
version: "1.0"
last_updated: 2026-03-21
---

# TOPO map — Standard Operating Procedures

These SOPs define **when, how, and by whom** the TOPO map is updated.
Map maintenance is not optional — it is part of the definition of done
for any component work.

---

## SOP-1: Project Initialisation

**When:** A new project is created (via `/init-project` or manually).

**Steps:**
1. Create `docs/solution-map.topo` from the TOPO template
2. Set the solution name, owner, lifecycle, and platform(s) in the header
3. Write the Solution Intent section (1 paragraph + design principles)
4. Complete the Platform Profile table mapping abstract types to concrete implementations
5. Add section 14 (Architectural Rules) with project-specific constraints
6. Commit with message: `docs: initialise TOPO map`

**Owner:** Lead developer or solution architect.

---

## SOP-2: New Data Entity

**When:** A data entity (table, collection, document) is designed and approved.

**Steps:**
1. Add a `[DE]` block in section 4 (Data Entities), ordered by role
2. Set the YAML front matter: kind, id, role, lifecycle, schema_ref
3. Write the intent (one sentence: why this entity exists)
4. List logic-driving fields as `[DF]` entries — only fields that trigger logic
5. List registered server logic, client logic, and UI controls (may be empty initially)
6. Add `[DR]` relationship entries
7. Complete the `depends_on` and `depended_on_by` tables
8. Update related entities' relationship lists and dependency tables
9. Run `/topo-validate` to check consistency

**Owner:** Data modelling agent/role.

**Triggered by:** Schema design approval.

---

## SOP-3: New Server Logic

**When:** A server-side component (plugin, function, service, handler) is created.

**Steps:**
1. Add a `[SL]` block in section 5 (Server Logic), ordered by subtype
2. Set YAML front matter: kind, id, subtype (config-driven or bespoke), lifecycle
3. If bespoke: set fit_gap_ref linking to the justification document
4. Write the intent (one sentence)
5. Set error-behaviour: `throw`, `log-and-continue`, or `circuit-breaker`
6. Complete `depends_on` table — list every entity, shared function, and integration boundary this component reads from or calls
7. Complete `depended_on_by` table — list every entity this component writes to or serves
8. Update the parent `[DE]` block's server-logic list
9. Update every component in `depends_on` — add this component to their `depended_on_by`
10. If this component enforces a rule also enforced client-side: add a parity contract note and update section 12
11. Add platform-specific registration details in the footer
12. Run `/topo-validate` to check consistency

**Owner:** Plugin/function development agent/role.

**Triggered by:** Plugin creation, `/new-plugin` command, or equivalent.

---

## SOP-4: New Client Logic

**When:** A client-side component (form script, React component, etc.) is created.

**Steps:**
1. Add a `[CL]` block in section 6 (Client Logic)
2. Set YAML front matter: kind, id, lifecycle
3. Write the intent
4. Document the trigger (event, field, form)
5. Document what it writes (field values, notifications, visibility changes)
6. **Set parity** — this is mandatory:
   - If server-side equivalent exists: `parity: [SL] ComponentName`
   - If server-side is not yet built: `parity: [SL] ComponentName — TODO: not yet built`
   - If intentionally no server equivalent: `parity: (NONE — client-only because {reason})`
7. Complete dependency tables; update all referenced components
8. Update the parent `[DE]` block's client-logic list
9. Add or update the parity contract table in section 12
10. Add platform-specific details in the footer

**Owner:** Client development agent/role.

**Triggered by:** Form script creation, component development.

---

## SOP-5: New API

**When:** A new API endpoint or contract is created.

**Steps:**
1. Add an `[API]` block in section 7 (APIs)
2. Set YAML front matter: kind, id, lifecycle, protocol
3. Write the intent (what it does, who calls it)
4. Document request and response parameters with types
5. Set backing-logic reference to the `[SL]` component that implements it
6. Complete dependency tables; update the backing `[SL]` component's `depended_on_by`
7. If called by flows or external systems: update those components' dependency tables

**Owner:** API design agent/role.

---

## SOP-6: New Shared Function

**When:** A reusable function is created that is called by multiple components.

**Steps:**
1. Add an `[SF]` block in section 8 (Shared Functions)
2. Set YAML front matter: kind, id, language, location
3. Write the intent
4. Document args (name and type) and return value (type and description)
5. Complete dependency tables
6. Note: `depended_on_by` may be empty initially — it gets populated as callers are created

**Owner:** Development agent/role.

---

## SOP-7: New Background Process

**When:** An async, scheduled, or event-driven process is created.

**Steps:**
1. Add a `[BP]` block in section 9 (Background Processes)
2. Set YAML front matter: kind, id, lifecycle
3. Write the intent — state what it does and emphasise it is for orchestration/scheduling, not business logic (if that's a project rule)
4. Document the trigger
5. Complete dependency tables; update trigger entity and any APIs called
6. Add platform-specific details in the footer

**Owner:** Integration/automation agent/role.

---

## SOP-8: New UI Control

**When:** A custom UI control is created.

**Steps:**
1. Add a `[UC]` block in section 10 (UI Controls)
2. Set YAML front matter: kind, id, lifecycle
3. Write the intent (what it renders and why a custom control is needed)
4. Document bound-to field and any config sources
5. Complete dependency tables
6. Update the parent `[DE]` block's ui-controls list

**Owner:** UI development agent/role.

---

## SOP-9: New Config Pattern

**When:** A new configuration-driven pattern is designed.

**Steps:**
1. Add a `[CFG]` block in section 3 (Config Patterns)
2. Add a `[DE]` block for the config entity (role: config)
3. Add a `[SL]` block for the generic handler (subtype: config-driven)
4. Set applies-to with the initial target entities
5. Link catalogue-ref to the configuration catalogue
6. Complete all dependency tables across all three new blocks
7. Run `/topo-validate` to check consistency

**Owner:** Config design agent/role.

**Triggered by:** Pattern identified 3+ times.

---

## SOP-10: New Integration Boundary

**When:** The solution connects to a new external system.

**Steps:**
1. Add an `[IB]` block in section 11 (Integration Boundaries)
2. Set YAML front matter: kind, id, lifecycle, direction (inbound/outbound/bidirectional), owner
3. Write the intent
4. Complete all three layers:
   - **Contract:** protocol, auth, endpoint, payload, SLA
   - **Boundary:** trust level, data classification, failure mode
   - **Operational:** rate limits, circuit breaker, monitoring
5. Complete dependency tables — external systems have no `depends_on` within the solution
6. Ensure a `[SF]` shared function wraps the integration call (never call external systems directly from server logic)

**Owner:** Integration architect/role.

---

## SOP-11: Adding a Dependency

**When:** An existing component starts consuming another component.

**Steps:**
1. Add a row to the consumer's `depends_on` table
2. Add a row to the provider's `depended_on_by` table
3. Choose the correct relationship type from the vocabulary
4. Write a one-sentence justification (why column) and impact statement

**Owner:** Whoever is making the code change.

---

## SOP-12: Removing a Component

**When:** A component is removed from the solution.

**Steps:**
1. **Before removing:** Check the component's `depended_on_by` table for impact
2. Present the impact analysis to the reviewer/approver
3. On approval: remove the component block
4. Remove ALL references to this component from every other block:
   - `depends_on` tables
   - `depended_on_by` tables
   - Parent entity server-logic/client-logic/ui-controls lists
   - Parity contract table (section 12)
5. Run `/topo-validate` to catch any orphaned references
6. Regenerate the Mermaid dependency graph

**Owner:** Whoever is removing the component.

---

## SOP-13: Periodic Map Review

**When:** Weekly, or before any major milestone (sprint review, UAT, go-live).

**Steps:**
1. Run `/topo-validate`
2. Review all `TODO` entries in the parity contracts table
3. Review all `draft` lifecycle components — are they still planned or abandoned?
4. Review all `deprecated` components — have dependents migrated?
5. Verify the Mermaid graph matches your mental model of the architecture
6. Update `last_updated` in the document header

**Owner:** Solution architect or lead developer.

---

## SOP-14: New Platform Profile

**When:** The solution needs to deploy to an additional platform.

**Steps:**
1. Add the platform name to the header's `platforms` list
2. Add a new Platform Profile table in section 2
3. For each existing component: add a platform-specific footer if the component
   exists on the new platform
4. Review parity contracts — do any need new platform-specific rows?
5. Review integration boundaries — do protocols or auth methods change per platform?

**Owner:** Platform/infrastructure architect.

---

## Quick Reference: "What do I update?"

| I just... | Update these in the map |
|---|---|
| Created a new entity | SOP-2 |
| Created a server-side component | SOP-3 |
| Created a client-side component | SOP-4 |
| Created an API | SOP-5 |
| Created a shared function | SOP-6 |
| Created a background process | SOP-7 |
| Created a UI control | SOP-8 |
| Created a config pattern | SOP-9 |
| Connected to an external system | SOP-10 |
| Made component A call component B | SOP-11 |
| Removed a component | SOP-12 |
| It's sprint review time | SOP-13 |
| Adding AWS alongside Dataverse | SOP-14 |
