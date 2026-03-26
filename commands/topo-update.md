---
name: topo-update
description: >
  Interactively add, modify, or remove a component from the TOPO map.
  Handles all bidirectional dependency updates, entity cross-references,
  parity contract updates, and Mermaid graph regeneration automatically.
arguments: action component_type component_name
---

# /topo-update $ARGUMENTS

Parse arguments as: Action ComponentType ComponentName

- Action: `add` | `modify` | `remove`
- ComponentType: `DE` | `SL` | `CL` | `API` | `SF` | `BP` | `UC` | `CFG` | `IB`
- ComponentName: the identifier or display name

Examples:
- `/topo-update add DE contoso_projecttask`
- `/topo-update add SL ValidateProjectBudget`
- `/topo-update modify CL OnFormLoad`
- `/topo-update remove BP Order - On Create - Notify ERP`
- `/topo-update add IB PaymentGateway`

## Step 1: Load the Map

Read `docs/solution-map.topo`. If missing, instruct user to run `/init-project`.

## Step 2: Route by Action

### ADD

1. Check if a block with this type and name already exists. If yes, ask: did you mean `modify`?

2. Determine the correct SOP from `docs/topo-standard-operating-procedures.md`:
   - DE > SOP-2
   - SL > SOP-3
   - CL > SOP-4
   - API > SOP-5
   - SF > SOP-6
   - BP > SOP-7
   - UC > SOP-8
   - CFG > SOP-9
   - IB > SOP-10

3. Follow the SOP step-by-step. Prompt the user for required attributes.
   Pre-fill what you can infer from context (e.g. if you just created a plugin,
   you know the entity, message, and stage).

4. Create the block in the correct section (section order per language spec).

5. **Automatic cross-updates:**
   - For every `depends_on` entry: find target block, add this component to its `depended_on_by`
   - For `[SL]`, `[CL]`, `[UC]`: update parent `[DE]` block's server-logic/client-logic/ui-controls list
   - For `[CL]`: verify parity is set; if not, prompt for it
   - For `[API]`: update backing `[SL]` block's `depended_on_by`

6. Confirm all changes made (list every block touched).

### MODIFY

1. Find the existing block. If not found, ask: did you mean `add`?

2. Show the current block to the user.

3. Ask what needs to change. Apply changes.

4. **Dependency reconciliation:**
   - Compare old depends_on with new depends_on
   - Removed dependencies: remove this component from old target's depended_on_by
   - Added dependencies: add this component to new target's depended_on_by
   - Same for depended_on_by changes

5. If parity changed on a `[CL]` block: update section 12 parity contracts table.

6. Confirm all changes.

### REMOVE

1. Find the existing block. If not found, report and exit.

2. **Impact analysis first:**
   - List everything in this component's `depended_on_by` — these components will be affected
   - If any dependents exist: show the impact and ask for confirmation
   - Suggest: should these dependents be updated first?

3. On confirmation:
   - Remove the block
   - Remove ALL references from every other block's depends_on, depended_on_by,
     server-logic/client-logic/ui-controls lists, parity references
   - Remove from parity contracts table (section 12) if applicable

4. Report all blocks modified.

## Step 3: Regenerate Mermaid Graph

After any add/modify/remove, regenerate section 13 using the same logic as
`/topo-validate` step 9.

## Step 4: Update Header

Set `last_updated` to today's date.

## Step 5: Confirm

Show summary:
- Blocks added/modified/removed
- Cross-references updated (count)
- Parity table updated (yes/no)
- Mermaid graph regenerated (node count, edge count)
