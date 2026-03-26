---
name: topo
description: >
  Maintains the TOPO map (Topology of Platform Objects) — a living semantic architecture
  document (.topo file) that tracks every component's intent, dependencies, and
  interconnections. TOPO is platform-agnostic; concrete implementations are captured via
  platform profiles. TRIGGER ON EVERY operation that creates, modifies, or removes a
  solution component: data entities, server logic, client logic, APIs, shared functions,
  background processes, UI controls, config patterns, or integration boundaries. Also
  trigger on /topo-validate and /topo-update commands. This skill is CROSS-CUTTING — it
  is invoked by other skills and agents as a mandatory sub-step.
allowed-tools: Read, Write, Edit, Grep, Glob
---

# TOPO Skill — Topology of Platform Objects

## Purpose

The TOPO map (`docs/solution-map.topo`) is the single source of truth for how
components relate to each other and why they exist. This skill defines how to
read, write, and maintain `.topo` files.

**Full language reference:** `docs/topo-language-spec.md`
**SOPs:** `docs/topo-standard-operating-procedures.md`
**File extension:** `.topo` (markdown-compatible; associate with markdown syntax highlighting)

---

## 1. Quick Reference — Component Tags

| Tag | Type | When to create a block |
|---|---|---|
| `[DE]` | Data Entity | New table/collection/document designed |
| `[DF]` | Data Field | Field added that drives logic |
| `[DR]` | Data Relationship | Relationship between entities |
| `[SL]` | Server Logic | Server-side component created |
| `[CL]` | Client Logic | Client-side component created |
| `[API]` | API | Exposed interface created |
| `[SF]` | Shared Function | Reusable function created |
| `[BP]` | Background Process | Async/scheduled process created |
| `[UC]` | UI Control | Custom UI component created |
| `[CFG]` | Config Pattern | Config entity + handler pair created |
| `[IB]` | Integration Boundary | External system connection created |

## 2. Quick Reference — Relationship Types

| Type | Meaning | Impact |
|---|---|---|
| `depends_on` | A requires B | B's failure breaks A |
| `serves` | A provides to B | A's change may break B |
| `triggers` | A causes B to run | A's changes affect B |
| `flows_to` | Data moves A to B | A's schema may break B |
| `realizes` | A implements B | B's contract change requires A update |
| `configures` | A provides config to B | A's schema affects B |
| `contains` | B is part of A | B's change always impacts A |
| `subscribes_to` | A listens to B | B's event schema affects A |
| `maps_to` | Abstract to concrete | Platform profile mapping |
| `supersedes` | A replaces B | B's dependents should migrate |

## 3. Bidirectional Dependency Rule

**NON-NEGOTIABLE.** Every `depends_on` entry MUST have a corresponding `depended_on_by`
on the target, and vice versa. Always update BOTH sides.

## 4. Update Contracts

| Trigger Event | Map Updates Required |
|---|---|
| New data entity | Add `[DE]` block, `[DR]` entries, update related entities |
| New logic-driving field | Add `[DF]` under parent `[DE]` |
| New server logic | Add `[SL]` block, update `[DE]` list, update `[SF]` depended_on_by |
| New client logic | Add `[CL]` block, update `[DE]` list, SET PARITY |
| New API | Add `[API]` block, update backing `[SL]` depended_on_by |
| New shared function | Add `[SF]` block |
| New background process | Add `[BP]` block, update trigger entity and APIs |
| New UI control | Add `[UC]` block, update `[DE]` list |
| New config pattern | Add `[CFG]` + `[DE]` (config) + `[SL]` (handler) |
| New integration boundary | Add `[IB]` with contract/boundary/operational layers |
| Config row added | Update `[CFG]` applies-to |
| New dependency | Add to BOTH sides |
| Dependency removed | Remove from BOTH sides |
| Component removed | Remove block and ALL cross-references |

## 5. Integration Points

| Role | Obligation |
|---|---|
| Data modelling | Add `[DE]`, `[DF]`, `[DR]` blocks |
| Server logic | Add `[SL]` block |
| Client logic | Add `[CL]` block; set parity |
| Config design | Add `[CFG]`, `[DE]`, `[SL]` |
| Parity checking | Verify every `[CL]` has parity |
| Architecture review | Verify map consistency |
| Solution check | Run `/topo-validate` |
| Project init | Create `docs/solution-map.topo` from template |

## 6. What the Map is NOT

- Not a full schema doc — only logic-driving fields
- Not a code reference — only intent and interfaces
- Not a deployment guide — only architecture
- Not comprehensive — only interconnections

## 7. Anti-Patterns to Reject

- Listing every field (bloat)
- Prose in blocks (not scannable)
- Manual Mermaid edits (will drift)
- Skipping depended_on_by (breaks analysis)
- Platform details in abstract blocks (coupling)
- Freeform relationships (not analysable)
- Components without map update (staleness)
