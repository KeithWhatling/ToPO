---
title: TOPO — Topological Platform Ontology
version: "1.0"
status: draft
last_updated: 2026-03-21
---

# TOPO — Topological Platform Ontology

> **Note:** This file is the entry point for projects that have ToPO installed.
> The file paths below describe the **recommended installation layout** in your
> project. For the repository structure of ToPO itself, see the [README](README.md).

TOPO is a platform-agnostic architecture description language for tracking how
software solutions are structured, interconnected, and intended. It is expressed
as structured markdown (`.topo` files) — readable by humans, parseable by AI agents,
and compatible with any markdown renderer.

TOPO answers five questions about any solution:

1. Why does each component exist?
2. What does it depend on?
3. What breaks if it changes?
4. Where does it connect to external systems?
5. What invariants must hold across client and server?

TOPO is **not** a schema document, code reference, or deployment guide. It is a
wiring diagram — showing how things connect and why. Everything else lives in
your existing docs and code.

---

## Who is this for?

**Developers and AI agents** — read the `.topo` file to understand the solution's
architecture before making changes. Check `depended_on_by` tables before modifying
anything.

**Solution architects** — design the initial TOPO map, define platform profiles,
set architectural rules. Run `/topo-validate` before approvals.

**Delivery leads** — use the parity contracts table and dependency graph to track
completeness and risk.

---

## Quick Start

### 1. Install TOPO into your project

Copy the TOPO files into your project. The file tree below shows where everything goes.
Adapt the paths to match your project structure (e.g. if you use `.claude/` for
Claude Code, put skills/commands/rules there; if you use a different agent framework,
adapt accordingly).

```
your-project/
  docs/
    topo-language-spec.md           # Full language specification (reference)
    topo-standard-operating-procedures.md  # 14 SOPs for all update scenarios
    solution-map.topo               # YOUR solution's TOPO map (created in step 2)
  skills/
    topo/
      SKILL.md                      # Agent skill (tells AI how to maintain the map)
  commands/
    topo-validate.md                # /topo-validate command definition
    topo-update.md                  # /topo-update command definition
  rules/
    topo-maintenance.md             # Mandatory update rule
  templates/
    solution-map.topo               # Blank template (copy to create new maps)
  TOPO.md                           # This file (entry point)
```

### 2. Create your solution map

Copy `templates/solution-map.topo` to `docs/solution-map.topo` and fill in:

1. **Header** — set your solution name, owner, lifecycle, and platform(s)
2. **Section 1 (Solution Intent)** — one paragraph on what this solution does and why
3. **Section 2 (Platform Profile)** — map abstract types to your concrete implementations
4. **Section 14 (Architectural Rules)** — add project-specific constraints

That's your starting point. The map grows as you build components.

### 3. Start building

Every time you create a component (data entity, server logic, client logic, API,
shared function, background process, UI control, config pattern, or integration
boundary), update the TOPO map following the relevant SOP in
`docs/topo-standard-operating-procedures.md`.

The short version:
- Add a block for the new component in the correct section
- Fill in the YAML front matter, intent, and type-specific attributes
- Complete the `depends_on` and `depended_on_by` tables
- Update BOTH sides of every dependency (bidirectional rule)
- For client logic: set parity (mandatory — no exceptions)
- Add platform-specific details in a labelled footer

### 4. Validate regularly

Run `/topo-validate` (or follow the steps in `commands/topo-validate.md` manually) to:
- Check bidirectional dependency consistency
- Find orphan references
- Verify parity coverage
- Check lifecycle hygiene
- Regenerate the Mermaid dependency graph

---

## File Reference

| File | What it does | When to read it |
|---|---|---|
| `TOPO.md` | This file — entry point and install guide | First time setup |
| `docs/topo-language-spec.md` | Full language specification — component types, relationships, block formats, platform profiles, conventions | When writing or reviewing TOPO blocks |
| `docs/topo-standard-operating-procedures.md` | 14 step-by-step SOPs for every update scenario | When adding, modifying, or removing components |
| `docs/solution-map.topo` | Your solution's TOPO map — the actual architecture doc | Constantly — this is the living document |
| `skills/topo/SKILL.md` | Agent instructions — quick reference tables and update contracts | AI agents read this automatically |
| `commands/topo-validate.md` | `/topo-validate` command — consistency checks | When running validation |
| `commands/topo-update.md` | `/topo-update` command — interactive add/modify/remove | When using the command |
| `rules/topo-maintenance.md` | Mandatory update rule — fires on all component operations | Agents read this automatically |
| `templates/solution-map.topo` | Blank template for new projects | When creating a new solution map |

---

## For AI Agents

If you are an AI agent (Claude Code, Copilot, Cursor, or similar) and you encounter
a `.topo` file or are asked to work with TOPO:

1. **Read `skills/topo/SKILL.md`** — this is your operating manual. It contains
   quick reference tables for component tags, relationship types, and update contracts.

2. **Read `docs/solution-map.topo`** — this is the current architecture. Before
   creating or modifying any component, check this file to understand the existing
   interconnections.

3. **Follow the update contracts** — after any component operation, update the
   `.topo` file. The contracts in the skill file define exactly what to update
   for each type of change. The bidirectional dependency rule is non-negotiable.

4. **Use the SOPs** — `docs/topo-standard-operating-procedures.md` contains
   step-by-step procedures for every scenario. Follow them.

5. **Validate** — run `/topo-validate` checks after significant changes to catch
   inconsistencies.

### Agent Integration Patterns

**Claude Code:** Place skill, commands, and rules under `.claude/`:
```
.claude/skills/topo/SKILL.md
.claude/commands/topo-validate.md
.claude/commands/topo-update.md
.claude/rules/topo-maintenance.md
```

**Other agent frameworks:** Adapt paths to match your framework's convention for
skills/instructions, commands/actions, and rules/constraints. The file contents
are framework-agnostic — only the paths change.

### Hooking TOPO into existing agents

If you have existing agents or skills that create solution components (data models,
plugins, form scripts, APIs, etc.), add a **TOPO obligation** step at the end of
each. See `docs/topo-integration-guide.md` for specific patches per agent type.

The pattern is always the same:
```
After creating/modifying a component:
1. Read docs/solution-map.topo
2. Add/update the relevant block following the SOP
3. Update dependency tables on BOTH sides
4. Set parity on client logic components
```

---

## Component Types at a Glance

| Tag | Type | What it represents |
|---|---|---|
| `[DE]` | Data Entity | Table, collection, document — persistent data |
| `[DF]` | Data Field | A field that drives logic (not every field — only logic-driving ones) |
| `[DR]` | Data Relationship | Structural link between entities |
| `[SL]` | Server Logic | Business rules executing server-side |
| `[CL]` | Client Logic | Logic executing in the user's environment |
| `[API]` | API | Exposed integration contract |
| `[SF]` | Shared Function | Reusable logic called by multiple components |
| `[BP]` | Background Process | Async or scheduled work |
| `[UC]` | UI Control | Custom UI component |
| `[CFG]` | Config Pattern | Configuration entity + generic handler |
| `[IB]` | Integration Boundary | Edge connecting to an external system |

## Relationship Types at a Glance

| Type | Meaning | Impact if target changes |
|---|---|---|
| `depends_on` | A requires B | B's failure breaks A |
| `serves` | A provides to B | A's interface change may break B |
| `triggers` | A causes B to run | A's changes affect B |
| `flows_to` | Data moves A to B | A's schema may break B |
| `realizes` | A implements B | B's contract change requires A update |
| `configures` | A provides config to B | A's schema affects B |
| `contains` | B is part of A | B's change always impacts A |
| `subscribes_to` | A listens to B | B's event schema affects A |
| `maps_to` | Abstract to concrete | Platform profile mapping |
| `supersedes` | A replaces B | B's dependents should migrate |

---

## Platform Profiles — Examples

TOPO is platform-agnostic. The platform profile (section 2 of your `.topo` file)
maps abstract types to your concrete stack. Here are starter profiles for common
platforms:

### Dataverse / D365 CE

| Abstract Type | Concrete Type | Runtime |
|---|---|---|
| Data Entity | Dataverse Table | Dataverse storage |
| Server Logic | C# Plugin (sandbox) | Plugin Execution Pipeline |
| Client Logic | TypeScript Form Script | Model-driven app |
| API | Custom API (contoso_ prefixed) | Dataverse Web API |
| Shared Function | C# class in plugin assembly | Plugin pipeline |
| Background Process | Power Automate Cloud Flow | Power Automate |
| UI Control | PCF Control (React) | Model-driven app |
| Config Pattern | contoso_Config* table + generic plugin | Plugin reads config at runtime |
| Integration Boundary | External HTTP endpoint | HttpClient from sandbox |

### Azure Serverless

| Abstract Type | Concrete Type | Runtime |
|---|---|---|
| Data Entity | Cosmos DB Collection / SQL Table | Azure data services |
| Server Logic | Azure Function | Functions runtime |
| Client Logic | React Component | Browser (Static Web Apps) |
| API | Function HTTP endpoint | API Management |
| Shared Function | NuGet / npm package | Build-time linked |
| Background Process | Durable / Timer Function | Functions runtime |
| UI Control | React Component | Browser |
| Config Pattern | Azure App Configuration | App Config service |
| Integration Boundary | External HTTP / Service Bus | Managed identity |

### Java Spring Boot

| Abstract Type | Concrete Type | Runtime |
|---|---|---|
| Data Entity | JPA Entity / PostgreSQL Table | PostgreSQL |
| Server Logic | Spring @Service class | Spring Boot |
| Client Logic | React / Thymeleaf Component | Browser |
| API | @RestController | Spring MVC |
| Shared Function | Utility class / shared JAR | JVM classpath |
| Background Process | @Scheduled / Spring Batch | Spring Boot |
| UI Control | React Component | Browser |
| Config Pattern | application.yml / Spring Cloud Config | Spring Environment |
| Integration Boundary | External REST / Kafka | Resilience4j circuit breaker |

### .NET Web API

| Abstract Type | Concrete Type | Runtime |
|---|---|---|
| Data Entity | EF Core Entity / SQL Table | SQL Server / PostgreSQL |
| Server Logic | Service class | ASP.NET Core |
| Client Logic | Blazor / React Component | Browser |
| API | Controller / Minimal API endpoint | ASP.NET Core |
| Shared Function | Class library (NuGet) | .NET runtime |
| Background Process | Hosted Service / Hangfire Job | ASP.NET Core |
| UI Control | Blazor Component | Browser |
| Config Pattern | appsettings.json / Azure App Config | Configuration providers |
| Integration Boundary | External HTTP / Azure Service Bus | Polly circuit breaker |

---

## The .topo File Extension

TOPO documents use the `.topo` extension. They are valid markdown — any markdown
renderer can display them — but the extension signals that this file follows the
TOPO specification.

**Editor setup:** Associate `.topo` with your markdown syntax highlighter.

| Editor | How to associate |
|---|---|
| VS Code | Add `"files.associations": { "*.topo": "markdown" }` to settings.json |
| JetBrains IDEs | Settings > Editor > File Types > Markdown > Add `*.topo` |
| Vim/Neovim | `autocmd BufRead,BufNewFile *.topo set filetype=markdown` |
| GitHub | Add to `.gitattributes`: `*.topo linguist-language=Markdown` |

---

## FAQ

**Can I use TOPO without Claude Code / AI agents?**
Yes. TOPO is a documentation standard, not an AI feature. The `.topo` file is
just structured markdown. The skills, commands, and rules are instructions that
*can* be consumed by agents but work equally as checklists for humans.

**Can I have multiple `.topo` files in one project?**
The primary map is `docs/solution-map.topo`. For very large solutions you could
split into domain-specific maps (e.g. `docs/crm-map.topo`, `docs/finance-map.topo`)
but you'd need to manage cross-map references manually. Start with one.

**What if my project spans multiple platforms?**
List all platforms in the header (`platforms: [dataverse, azure-functions, react]`)
and include a platform profile table for each. Components indicate which platform(s)
they exist on via their platform-specific footer sections.

**How is TOPO different from C4 / ArchiMate / UML?**
TOPO operates at the same abstraction level as C4's Component diagram but adds:
typed bidirectional dependencies (from SPDX/CycloneDX), platform profiles
(from TOGAF ABB/SBB), parity contracts, and integration boundary modelling. It's
expressed as markdown, not a visual-first notation — making it native to AI agents
and version control.

**Can I generate diagrams from a `.topo` file?**
The Mermaid dependency graph in section 13 is auto-generated from the dependency
tables. The `/topo-validate` command regenerates it. An interactive HTML viewer
is on the roadmap.

---

## Licence and Attribution

TOPO was designed by Keith Whatling, developed in collaboration with
Claude (Anthropic). It draws on concepts from ArchiMate (The Open Group), TOGAF
(The Open Group), C4 Model (Simon Brown), OntoUML, Dublin Core, SPDX, CycloneDX,
and OAM.

TOPO is free to use, adapt, and distribute.
