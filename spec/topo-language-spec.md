---
title: TOPO — Topology of Platform Objects
subtitle: Language Specification
version: "1.0"
status: draft
file_extension: .topo
last_updated: 2026-03-21
---

# TOPO — Language Specification v1.0

## 1. Introduction

### 1.1 What is TOPO?

TOPO (Topology of Platform Objects) is a platform-agnostic architecture description
language for documenting how software solutions are structured, interconnected, and
intended. It is expressed as structured markdown with YAML front matter — designed
to be readable by both humans and AI agents without specialised tooling.

TOPO documents use the `.topo` file extension. They are markdown-compatible — any
markdown renderer can display them — but the extension signals to agents and tooling
that this is a TOPO architecture document with structured semantics.

**Editor configuration:** Associate `.topo` files with markdown syntax highlighting.

TOPO describes **intent and interconnection**, not implementation detail. It answers:

- Why does this component exist?
- What does it depend on?
- What breaks if it changes?
- Where does this solution connect to the outside world?
- What invariants must hold across client and server?

### 1.2 What TOPO is NOT

- **Not a schema document** — full column inventories, data types, and constraints
  belong in schema documentation. TOPO tracks only fields that drive logic.
- **Not a code reference** — implementation detail belongs in code comments.
  TOPO tracks intent and interfaces.
- **Not a deployment guide** — environment configuration and CI/CD belong
  in ALM documentation.
- **Not comprehensive** — TOPO is deliberately selective. It is a wiring diagram
  showing how things connect and why, not an inventory of everything.

### 1.3 Design Principles

TOPO is grounded in established architecture standards:

| Principle | Source | How TOPO applies it |
|---|---|---|
| Layered realisation | ArchiMate | Abstract components map to concrete implementations via platform profiles |
| ABB to SBB mapping | TOGAF | Abstract types map 1:1 to concrete types per platform |
| Hierarchical zoom | C4 Model | Solution-level map with drill-down references to detailed docs |
| Typed dependencies | SPDX / CycloneDX | Relationships carry a type and direction, enabling impact analysis |
| Rigid type identity | OntoUML | A component's abstract type never changes; its platform realisation varies |
| Dumb-down refinement | Dublin Core | Specialised attributes degrade gracefully to generic parents |
| Persona separation | OAM / Crossplane | Abstract definitions owned by architects; platform mappings by platform engineers |

### 1.4 Format

TOPO files (`.topo`) are markdown files with optional YAML front matter blocks.
The format is optimised for:

- **Human readability** — scannable headings, tables, structured key-value pairs
- **AI agent consumption** — greppable tags, consistent structure, token-efficient
- **Tool interoperability** — parseable into directed graphs for impact analysis
- **Paste-friendliness** — tables render correctly in Word, Excel, and Miro

---

## 2. Document Structure

Every TOPO document (`docs/solution-map.topo`) follows this section order:

| Section | Content | Required |
|---|---|---|
| **Header** | YAML front matter with solution metadata | Yes |
| **1. Solution Intent** | Why this solution exists and what principles guide it | Yes |
| **2. Platform Profile** | Mapping table from abstract types to concrete implementations | Yes |
| **3. Config Patterns** | `[CFG]` blocks | If applicable |
| **4. Data Entities** | `[DE]` blocks | Yes |
| **5. Server Logic** | `[SL]` blocks | If applicable |
| **6. Client Logic** | `[CL]` blocks | If applicable |
| **7. APIs** | `[API]` blocks | If applicable |
| **8. Shared Functions** | `[SF]` blocks | If applicable |
| **9. Background Processes** | `[BP]` blocks | If applicable |
| **10. UI Controls** | `[UC]` blocks | If applicable |
| **11. Integration Boundaries** | `[IB]` blocks | If applicable |
| **12. Parity Contracts** | Table of client/server invariants | If applicable |
| **13. Dependency Graph** | Auto-generated Mermaid diagram | Yes |
| **14. Architectural Rules** | Solution-specific rules | Yes |

### 2.1 Document Header

```yaml
---
topo: "1.0"
solution: my-solution-name
owner: team-or-individual
lifecycle: active
platforms: [dataverse, aws-serverless]
last_updated: 2026-03-21
---
```

| Field | Type | Required | Description |
|---|---|---|---|
| `topo` | string | Yes | TOPO specification version |
| `solution` | string | Yes | Unique solution identifier (kebab-case) |
| `owner` | string | Yes | Team or individual responsible |
| `lifecycle` | enum | Yes | `draft`, `active`, `deprecated`, `archived` |
| `platforms` | list | Yes | Platform profiles in use |
| `last_updated` | date | Yes | ISO date of last modification |

---

## 3. Component Types

TOPO defines **eleven** abstract component types. Each type has a two-to-three letter
tag used in headings and cross-references.

### 3.1 Component Type Registry

| Tag | Abstract Type | Description |
|---|---|---|
| `[DE]` | Data Entity | Persistent data structure (table, collection, document) |
| `[DF]` | Data Field | A field on a data entity that drives logic (only logic-driving fields) |
| `[DR]` | Data Relationship | A structural relationship between data entities |
| `[SL]` | Server Logic | Component executing business rules on the server side |
| `[CL]` | Client Logic | Component executing in the user's environment |
| `[API]` | API | Explicitly defined integration contract exposed to consumers |
| `[SF]` | Shared Function | Reusable logic referenced by multiple components, not independently deployed |
| `[BP]` | Background Process | Scheduled or event-driven work without direct user interaction |
| `[UC]` | UI Control | Custom UI component rendered in the application |
| `[CFG]` | Config Pattern | A configuration entity + handler pair driving behaviour from data |
| `[IB]` | Integration Boundary | Explicit edge where the solution connects to an external system |

### 3.2 Entity Roles

| Role | Description | Examples |
|---|---|---|
| `config` | Drives behaviour via configuration — read at runtime by generic handlers | Validation rules, formatting rules, auto-number patterns |
| `parameter` | Reference data that controls logic in other entities | Item types, approval thresholds, category definitions |
| `transactional` | Core business data created/updated during operations | Orders, invoices, cases, line items |
| `reference` | Static or slowly-changing lookup data | Countries, currencies, units of measure |
| `junction` | Resolution entity for many-to-many relationships | Project-team member assignments |

### 3.3 Server Logic Subtypes

| Subtype | Description |
|---|---|
| `config-driven` | Generic handler that reads rules from a config entity at runtime |
| `bespoke` | Custom logic justified by fit-gap analysis — requires `fit_gap_ref` |

### 3.4 Lifecycle Values

| Value | Meaning |
|---|---|
| `draft` | Designed but not yet implemented |
| `active` | Implemented and in use |
| `deprecated` | Scheduled for removal; dependents should migrate |
| `archived` | Removed from the solution; kept for historical reference |

---

## 4. Data Type Shortcodes

Used in `[DF]` field declarations.

| Code | Type | Platform examples |
|---|---|---|
| `text` | Single or multi-line text | Dataverse Text, SQL VARCHAR, MongoDB String |
| `number` | Whole number, decimal, float, currency | Dataverse Whole Number/Decimal/Currency, SQL INT/DECIMAL |
| `boolean` | Two-option true/false | Dataverse Two Option, SQL BIT |
| `choice` | Enumerated option set | Dataverse Choice, SQL ENUM |
| `date` | Date only | Dataverse Date Only, SQL DATE |
| `datetime` | Date and time | Dataverse Date and Time, SQL DATETIME2 |
| `lookup` | Reference to another entity | Dataverse Lookup, SQL FK, MongoDB ObjectId |
| `json` | Text storing structured JSON | Dataverse Multi-line Text, SQL NVARCHAR(MAX) |
| `calculated` | Computed or rollup value | Dataverse Calculated/Rollup, SQL computed column |
| `image` | Binary image data | Dataverse Image, S3 object reference |
| `file` | Binary file attachment | Dataverse File, S3 object |
| `guid` | Unique identifier | Dataverse PK, SQL UNIQUEIDENTIFIER |

---

## 5. Relationship Vocabulary

### 5.1 Relationship Types

| Relationship | Direction | Meaning | Impact |
|---|---|---|---|
| `depends_on` | A depends_on B | A requires B to function | B's failure breaks A |
| `serves` | A serves B | A provides capability consumed by B | A's interface change may break B |
| `triggers` | A triggers B | A causes B to execute | A's changes affect B's execution |
| `flows_to` | A flows_to B | Data or events move from A to B | A's schema changes may break B |
| `realizes` | A realizes B | A implements B's specification | B's contract change requires A update |
| `configures` | A configures B | A provides configuration consumed by B | A's schema change affects B |
| `contains` | A contains B | B is part of A | B's change always impacts A |
| `subscribes_to` | A subscribes_to B | A listens for events from B | B's event schema changes affect A |
| `maps_to` | A maps_to B | Abstract maps to concrete | Platform profile mapping |
| `supersedes` | A supersedes B | A replaces B | B's dependents should migrate to A |

### 5.2 Bidirectional Rule

**MANDATORY.** Every `depends_on` entry on component A MUST have a corresponding
entry in target B's `depended_on_by` table, and vice versa. Always update BOTH sides.

### 5.3 Dependency Table Format

**depends_on table:**

| target | relationship | why |
|---|---|---|
| `[TAG] ComponentName` | relationship_type | One sentence explaining the dependency |

**depended_on_by table:**

| source | relationship | impact |
|---|---|---|
| `[TAG] ComponentName` | relationship_type | What breaks if this component changes |

---

## 6. Component Block Format

### 6.1 General Structure

```markdown
### [TAG] ComponentName

```yaml
kind: component-type
id: kebab-case-id
lifecycle: active
```

- **intent:** One sentence explaining WHY this component exists
- (type-specific attributes)

| depends_on | relationship | why |
|---|---|---|
| ... | ... | ... |

| depended_on_by | relationship | impact |
|---|---|---|
| ... | ... | ... |

**{Platform} specifics:**
- (platform-specific details in a labelled footer)
```

### 6.2 YAML Front Matter Fields

| Field | Required | Description |
|---|---|---|
| `kind` | Yes | Abstract type (kebab-case) |
| `id` | Yes | Unique identifier within solution |
| `lifecycle` | Yes | draft, active, deprecated, archived |
| `subtype` | If applicable | e.g. config-driven, bespoke |
| `owner` | Optional | Override solution-level owner |
| `fit_gap_ref` | If bespoke | Reference to fit-gap justification |
| `schema_ref` | If data entity | Reference to full schema docs |
| `language` | If shared function | Programming language |
| `location` | If shared function | File path to source |

### 6.3 Block Templates Per Type

#### [DE] Data Entity
```
intent, role, schema_ref, logic-driving fields ([DF] entries),
server-logic list, client-logic list, ui-controls list,
relationships ([DR] entries), dependency tables
```

#### [SL] Server Logic
```
intent, subtype, error-behaviour (throw | log-and-continue | circuit-breaker),
dependency tables, parity contract note, platform registration footer
```

#### [CL] Client Logic
```
intent, trigger, writes, parity (MANDATORY — [SL] ref or NONE with reason),
dependency tables, platform specifics footer
```

#### [API] API
```
intent, protocol, request params, response params, backing-logic,
dependency tables
```

#### [SF] Shared Function
```
intent, language, location, args, returns, dependency tables
```

#### [BP] Background Process
```
intent, trigger, dependency tables, platform specifics footer
```

#### [UC] UI Control
```
intent, bound-to, config-source, dependency tables, platform specifics footer
```

#### [CFG] Config Pattern
```
intent, config-entity, handler, applies-to, catalogue-ref
```

#### [IB] Integration Boundary
```
intent, Contract (protocol, auth, endpoint, payload, SLA),
Boundary (trust, data classification, failure mode),
Operational (rate limit, circuit breaker, monitoring),
dependency tables
```

---

## 7. Platform Profiles

### 7.1 Purpose

Platform profiles map abstract TOPO types to concrete implementations on a specific
technology stack. The abstract architecture is platform-agnostic; profiles make it
deployable.

### 7.2 Format

```markdown
## 2. Platform Profile: {Platform Name}

| Abstract Type | Concrete Type | Runtime | Notes |
|---|---|---|---|
| Data Entity | {platform table/collection} | {storage engine} | ... |
| Server Logic | {platform handler} | {execution runtime} | ... |
| ... | ... | ... | ... |
```

### 7.3 Per-Component Platform Details

Platform-specific details go in a clearly labelled footer:

```markdown
**Dataverse registration:**
- Entity: contoso_listlineitem
- Message: Create
- Stage: PreOperation
```

Multi-platform solutions include multiple footers per component.

---

## 8. Parity Contracts

Testable invariants that must hold across client and server, or across platforms.

| Rule | Client Enforcement | Server Enforcement | Status |
|---|---|---|---|
| Human-readable invariant | [CL] component | [SL] component | ACTIVE / TODO / PARTIAL / EXEMPT |

Individual `[CL]` blocks MUST set parity:
- `parity: [SL] ComponentName`
- `parity: (NONE — client-only because {reason})`

---

## 9. Dependency Graph

Auto-generated Mermaid `graph TD` diagram. Never maintained manually.

**Node naming:** `{TYPE}_{name}` with label `"[TYPE] DisplayName"`
**Grouping:** Subgraphs by component type
**Edges:** Solid for runtime dependencies, dotted for parity
**Generation:** By `/topo-validate` and `/topo-update` commands

---

## 10. Conventions

| Element | Convention | Example |
|---|---|---|
| TOPO file | `.topo` extension | `docs/solution-map.topo` |
| Component IDs | kebab-case | `validate-item` |
| Component names | PascalCase (code) or natural language (processes) | `ValidateItem` |
| Tags in prose | Square brackets | `[SL] ValidateItem` |
| External doc refs | Relative paths | `docs/dataverse-schema.md#table` |
| Empty sections | `(none in current scope)` | Signals considered, not forgotten |

**Section ordering:**
- Data Entities: config > parameter > reference > transactional > junction
- Server Logic: config-driven > bespoke
- All others: alphabetical

---

## 11. Anti-Patterns

| Anti-Pattern | Problem | Do This Instead |
|---|---|---|
| Listing every field | Bloats map; duplicates schema docs | Only logic-driving fields |
| Prose in blocks | Not scannable | Structured key-value pairs and tables |
| Manual Mermaid edits | Drifts immediately | Auto-generate from dependency tables |
| Skipping depended_on_by | Breaks impact analysis | Always update BOTH sides |
| Missing parity on [CL] | Gaps go undetected | Always set — use NONE with reason if intentional |
| Platform details in abstract block | Couples to one platform | Labelled platform footer |
| Freeform relationships | Not analysable | Use typed vocabulary |
| Components without map update | Staleness | Update contracts are mandatory |

---

## 12. Glossary

| Term | Definition |
|---|---|
| **TOPO** | Topology of Platform Objects — this language |
| **Abstract type** | Platform-agnostic component category |
| **Concrete type** | Platform-specific implementation |
| **Platform profile** | Mapping table from abstract to concrete per platform |
| **Parity contract** | Testable invariant across client and server |
| **Integration boundary** | Edge where the solution connects to an external system |
| **Config pattern** | Config entity + generic handler pair |
| **Logic-driving field** | Field whose value triggers or parameterises logic |
| **Dependency table** | Structured table listing depends_on and depended_on_by |
| **Impact analysis** | Tracing depended_on_by chains to find what breaks |
| **.topo file** | A TOPO document (markdown-compatible, `.topo` extension) |
