---
name: topo-validate
description: >
  Validate the TOPO map for consistency — check bidirectional dependencies,
  orphan references, missing parity contracts, lifecycle hygiene, and regenerate
  the Mermaid dependency graph. Run periodically and as part of /check-solution.
arguments: (none)
---

# /topo-validate

Run a full consistency check on `docs/solution-map.topo` and regenerate the dependency graph.

## Step 1: Load the Map

Read `docs/solution-map.topo`. If it does not exist, tell the user to run `/init-project`
or create it from the template.

## Step 2: Parse All Component Blocks

Extract every TOPO block and build an in-memory index:
- Component ID (tag + name, e.g. `[SL] ValidateItem`)
- All `depends_on` table entries
- All `depended_on_by` table entries
- For `[CL]` blocks: the `parity` value
- For `[DE]` blocks: the `server-logic`, `client-logic`, `ui-controls` lists
- For all blocks: the `lifecycle` value

## Step 3: Bidirectional Dependency Check

For every `depends_on` entry pointing to target `[X] Name`:
- Verify `[X] Name` exists in the map
- Verify `[X] Name` has a `depended_on_by` entry referencing this component

For every `depended_on_by` entry pointing to source `[X] Name`:
- Verify `[X] Name` exists in the map
- Verify `[X] Name` has a `depends_on` entry referencing this component

Report: `DEPENDENCY MISMATCH: [SL] ValidateItem depends_on [DE] contoso_itemtype, but [DE] contoso_itemtype does not list [SL] ValidateItem in depended_on_by`

## Step 4: Orphan Reference Check

Any component referenced in `depends_on`, `depended_on_by`, server-logic lists,
client-logic lists, ui-controls lists, parity references, or backing-logic
references that has no block in the map.

Report: `ORPHAN REFERENCE: [SF] InventoryApiClient referenced by [SL] ValidateItem but has no [SF] block`

## Step 5: Entity Cross-Reference Check

For every `[DE]` block:
- Every component listed in `server-logic` must have a corresponding `[SL]` block
- Every component listed in `client-logic` must have a corresponding `[CL]` block
- Every component listed in `ui-controls` must have a corresponding `[UC]` block

Report: `MISSING BLOCK: [SL] ValidateItem listed in [DE] contoso_listlineitem server-logic but no [SL] block exists`

## Step 6: Parity Check

For every `[CL]` block:
- `parity` must be populated
- Valid values: a `[SL]` reference, `(NONE — reason)`, or a `[SL]` reference with `TODO`
- Empty or missing parity: flag as warning

Cross-check against section 12 (Parity Contracts table):
- Every `[CL]` with a `[SL]` parity reference should appear in the table
- Every row in the table should reference components that exist in the map

Report: `PARITY WARNING: [CL] HighlightHealthStatus has no parity specified`
Report: `PARITY TABLE MISMATCH: [CL] EnforceQuantityLimit references [SL] ConfigurableValidator in parity but no row exists in Parity Contracts table`

## Step 7: Lifecycle Hygiene Check

- Components with `lifecycle: deprecated` should have no new `depended_on_by` entries
  from `active` components (warn if found)
- Components with `lifecycle: draft` for more than 30 days: flag for review
- Components with `lifecycle: archived` should have zero `depended_on_by` entries
  (error if found)

## Step 8: Integration Boundary Completeness Check

For every `[IB]` block:
- Contract section must have: Protocol, Auth, Endpoint
- Boundary section must have: Trust, Failure mode
- Every `[IB]` should be referenced by at least one `[SF]` (integration calls
  should be wrapped in shared functions, not called directly from server logic)

Report: `INTEGRATION WARNING: [IB] ExternalInventoryAPI is not referenced by any [SF] — integration calls should be wrapped in shared functions`

## Step 9: Regenerate Mermaid Dependency Graph

Build a `graph TD` from all dependency table entries:

1. Create nodes using the naming convention from the language spec (section 9.2)
2. Create subgraphs by component type
3. Create solid edges for runtime dependencies with relationship type labels
4. Create dotted edges for parity relationships
5. Replace section 13 content with the new diagram

## Step 10: Report Summary

Print a summary table:

| Check | Result | Issues |
|---|---|---|
| Bidirectional dependencies | PASS / FAIL | count |
| Orphan references | PASS / FAIL | count |
| Entity cross-references | PASS / FAIL | count |
| Parity coverage | PASS / WARN | count |
| Lifecycle hygiene | PASS / WARN | count |
| Integration boundaries | PASS / WARN | count |
| Mermaid graph | REGENERATED | node count, edge count |

Then list individual issues grouped by severity:
1. **ERRORS** (must fix): dependency mismatches, orphan references, missing blocks, archived with dependents
2. **WARNINGS** (should fix): parity gaps, lifecycle staleness, unwrapped integrations

## Step 11: Offer Fixes

For each error, suggest the specific edit. Ask the user whether to apply all
fixes automatically.
