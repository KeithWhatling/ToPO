---
title: TOPO — Framework Integration Guide
version: "1.0"
last_updated: 2026-03-21
---

# TOPO Framework Integration Guide

This guide describes how to embed TOPO into an existing
development framework. It uses the Claude Code Dataverse Framework as a
concrete example, but the pattern applies to any framework with agents,
commands, and rules.

---

## 1. Files to Add

```
{framework-root}/
  skills/
    topo/
      SKILL.md                              # TOPO skill (agent instructions)
  commands/
    validate-map.md                         # Consistency check command
    update-map.md                           # Interactive map update command
  rules/
    topo-maintenance.md             # Mandatory update rule
  docs/
    topo-language-spec.md                    # Full language specification
    topo-standard-operating-procedures.md    # SOPs for all update scenarios
    topo-template.md                # Blank template for new projects
    topo.md                         # Per-project map (created by init)
```

---

## 2. Hooks into Existing Framework Components

For each existing agent, command, or skill that creates or modifies solution
components, add a **TOPO map obligation** section. The specific additions
below use the Dataverse framework as an example; adapt the agent/command names
to your framework.

### 2.1 Project Initialisation Command

Add after initial scaffolding:

```markdown
## Create TOPO map

1. Copy `templates/solution-map.topo` to `docs/solution-map.topo`
2. Set solution name from project arguments
3. Set platform to match the project's target platform
4. Complete the Platform Profile table
5. Remind user: keep the map updated as you build
```

### 2.2 Data Modelling Agent/Skill

Add as a post-approval step:

```markdown
## TOPO map Obligation

After schema design is approved:
1. Read `docs/solution-map.topo`
2. Add `[DE]` blocks following SOP-2
3. Add `[DF]` entries for logic-driving fields
4. Add `[DR]` entries for relationships
5. Update related entities' dependency tables
```

### 2.3 Server Logic Agent/Skill (e.g. Plugin Development)

Add as a final step:

```markdown
## TOPO map Obligation

After component creation:
1. Add `[SL]` block following SOP-3
2. Update parent `[DE]` server-logic list
3. Update depended_on_by on all dependency targets
4. If parity-relevant: update section 12
5. Add platform-specific registration footer
```

### 2.4 Client Logic Agent/Skill (e.g. Form Scripting)

Add as a final step:

```markdown
## TOPO map Obligation

After component creation:
1. Add `[CL]` block following SOP-4
2. Update parent `[DE]` client-logic list
3. SET PARITY — mandatory, no exceptions
4. Update section 12 parity contracts table
```

### 2.5 Config Design Agent/Skill

Add as a final step:

```markdown
## TOPO map Obligation

After config pattern design:
1. Add `[CFG]` block following SOP-9
2. Add `[DE]` block for config entity (role: config)
3. Add `[SL]` block for generic handler (subtype: config-driven)
4. Complete all dependency tables across all three blocks
```

### 2.6 Parity Checking Agent

Add as a verification step:

```markdown
## TOPO map Parity Verification

1. Read `docs/solution-map.topo`
2. For every `[CL]` block, verify parity is populated
3. If parity is empty and the component validates or transforms data: flag as violation
4. Cross-check section 12 parity contracts table for completeness
```

### 2.7 Architecture Review Agent

Add as a pre-approval gate:

```markdown
## TOPO map Consistency Gate

Before approving any component:
1. Run `/topo-validate` checks
2. No orphan references allowed
3. All bidirectional dependencies must be paired
4. All client logic must have parity specified
5. Map consistency is part of definition of done
```

### 2.8 Solution Check Command

Add as an additional check step:

```markdown
## Validate TOPO map

Run `/topo-validate` as part of the solution check.
Include map validation results in the overall report.
```

---

## 3. Main Project File Updates

Add to the main project instructions file (e.g. `CLAUDE.md`):

### Commands Section

```markdown
| `/topo-validate` | Check TOPO map consistency and regenerate dependency graph |
| `/topo-update <action> <type> <name>` | Add, modify, or remove a component from the TOPO map |
```

### Critical Rules Section

```markdown
- ALWAYS update `docs/solution-map.topo` after creating, modifying, or removing
  any component — see `rules/topo-maintenance.md`
- Full TOPO language reference: `docs/topo-language-spec.md`
- SOPs for all update scenarios: `docs/topo-standard-operating-procedures.md`
```

---

## 4. Platform Profile — Dataverse Example

This is a complete platform profile for the Dataverse/D365 CE stack. Copy and
adapt for other platforms.

```markdown
## 2. Platform Profile: Dataverse

| Abstract Type | Concrete Type | Runtime | Notes |
|---|---|---|---|
| Data Entity | Dataverse Table (Entity) | Dataverse storage | Schema in docs/dataverse-schema.md |
| Data Field | Dataverse Column (Attribute) | Dataverse storage | Only tracked when driving logic |
| Data Relationship | Lookup / N:N | Dataverse storage | Cascade behaviour per relationship |
| Server Logic | C# Plugin (.NET 4.6.2, sandbox) | Plugin Execution Pipeline | Registered via Plugin Registration Tool |
| Client Logic | TypeScript Form Script | Model-driven app form | Web resource bound to form events |
| API | Custom API (contoso_ prefixed) | Dataverse Web API | POST /api/data/v9.2/contoso_ApiName |
| Shared Function | C# class/method in plugin assembly | Plugin Execution Pipeline | Compile-time linked; no independent deployment |
| Background Process | Power Automate Cloud Flow | Power Automate runtime | Connector orchestration only |
| UI Control | PCF Control (React + Fluent UI v9) | Model-driven app form | Field-bound or dataset-bound |
| Config Pattern | contoso_Config* table + generic plugin | Plugin reads config at runtime | New rules = rows, not code |
| Integration Boundary | External HTTP endpoint | HttpClient from sandbox plugin | Outbound only; sandboxed |
```

---

## 5. Platform Profile — Other Platforms (Starters)

### Azure Serverless

```markdown
| Abstract Type | Concrete Type | Runtime | Notes |
|---|---|---|---|
| Data Entity | Azure SQL Table / Cosmos DB Collection | Azure SQL / Cosmos DB | Schema in migrations or ARM templates |
| Server Logic | Azure Function (C# / Node.js) | Azure Functions runtime | HTTP or event trigger |
| Client Logic | React Component | Browser | SPA hosted on Static Web Apps |
| API | Azure Function HTTP endpoint | API Management (optional) | OpenAPI spec in /api/swagger |
| Shared Function | NuGet package / npm module | Linked at build | Published to private feed |
| Background Process | Durable Function / Timer Function | Azure Functions runtime | Orchestrations and schedules |
| UI Control | React Component | Browser | Part of SPA; not independently deployed |
| Config Pattern | Azure App Configuration | App Config service | Feature flags and settings |
| Integration Boundary | External HTTP / Service Bus | HttpClient / Service Bus SDK | Managed identity auth |
```

### Java Spring Boot

```markdown
| Abstract Type | Concrete Type | Runtime | Notes |
|---|---|---|---|
| Data Entity | JPA Entity / PostgreSQL Table | PostgreSQL | Flyway migrations |
| Server Logic | Spring Service class | Spring Boot | @Service annotated |
| Client Logic | Thymeleaf / React Component | Browser | SSR or SPA |
| API | REST Controller | Spring MVC | OpenAPI via springdoc |
| Shared Function | Utility class / shared library | JVM classpath | Internal or published JAR |
| Background Process | @Scheduled / Spring Batch Job | Spring Boot | Cron or event-driven |
| UI Control | React Component | Browser | Part of frontend build |
| Config Pattern | application.yml / Spring Cloud Config | Spring Environment | Profile-based |
| Integration Boundary | External REST / Kafka Topic | RestTemplate / KafkaTemplate | Circuit breaker via Resilience4j |
```
