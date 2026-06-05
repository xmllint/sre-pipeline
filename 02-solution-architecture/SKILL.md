---
name: solution-architecture
description: Take a problem brief and produce a solution architecture document — component topology, tradeoff analysis, monitoring strategy, failure mode analysis, and ADRs. Composable with downstream planning skills.
version: 1.2.0
author: Hermes Agent + Joe
license: MIT
---

# Solution Architecture

## Overview

Takes a completed Problem Brief and produces a **Solution Architecture**
document: component topology, data flow, tradeoff analysis, chosen design
with rationale, and a monitoring strategy. Downstream skills (scope
decomposition → task breakdown → output formatting) consume this without
revisiting architectural decisions.

## When to Use

- A Problem Brief exists and needs architectural design before epics
- A component topology is implicit in the problem but hasn't been
  diagrammed or stress-tested
- Tradeoffs between approaches need structured analysis (not gut feel)
- You need monitoring + failure-mode analysis built into the design
  phase, not bolted on later

**Don't use when:**
- No Problem Brief exists — run problem clarification first
- The solution is a trivial config change with no topology decisions
- The problem is already a fully-designed solution with diagrams and
  monitoring — feed directly to scope decomposition

## Setup (first use)

Before designing, ask the user:

1. **Tech stack** — what infrastructure, languages, databases, CI/CD
   are available? What's the fixed tooling?
2. **Observability stack** — what metrics, logging, tracing, and
   dashboarding exists? What's the alerting pipeline? Where do
   dashboards live?
3. **Deployment model** — how do services ship? What orchestration
   exists? What's the promotion path?
4. **Engineering standards** — any existing principles or conventions
   the design must conform to? (12-Factor? SOLID? Internal playbooks?)
5. **Prior art** — what internal libraries, patterns, or services
   already exist that this should leverage?

Record these as design constraints.

## Input

Problem Brief (fixed Markdown schema with sections: Context, Goals, SLIs
& Success Criteria, Incident Response Impact, Constraints, Out of Scope,
Stakeholders).

The skill parses these sections and uses them as design constraints:
- **Goals** → what the architecture must enable
- **SLIs** → observability requirements the design must support
- **Constraints** → hard boundaries
- **Out of Scope** → what NOT to design for

## Interactive Mode (default)

The skill runs a structured design interview:

1. **Load** the Problem Brief, extract all constraints and goals
2. **Pre-audit** the brief against engineering principles BEFORE
   proposing topology. This surfaces constraints and violations early
   so the first topology draft doesn't need patching. Principles that
   don't apply (e.g., circuit breaker for a weekly cron) are marked
   N/A with justification; the audit is a radar, not a gate.
3. **Propose** a component topology with data flow, informed by the
   pre-audit findings
4. **Present 2-3 architectural tradeoffs** specific to this problem
5. **Ask clarifying questions** about the highest-impact unresolved
   tradeoff. Max 3 rounds of questions.
6. **Resolve** each tradeoff with rationale
7. **Final audit** against principles — catch anything the pre-audit
   missed or that emerged during tradeoff resolution
8. **Emit** the Solution Architecture document

### Tradeoff Catalogue (fixed)

The skill selects applicable tradeoffs from this catalogue. It does
NOT invent new categories — this keeps design reviews comparable.

| Tradeoff                  | Question                                              |
|---------------------------|-------------------------------------------------------|
| Push vs poll              | Do data sources push events or do we poll them?       |
| Synchronous vs async      | Does the operation block the caller, or is it async?  |
| Queue topology            | Single queue, DLQ, or priority lanes?                 |
| Monolith vs fan-out       | One engine or per-source workers?                     |
| State management          | Database, in-memory, or event-sourced?                |
| API surface               | REST, GraphQL, or gRPC?                               |
| Scheduling model          | Cron, workflow engine, or external trigger?           |
| Idempotency strategy      | At-least-once with dedup, or exactly-once?            |
| Failure isolation         | One source failure blocks all, or per-source fencing?  |
| Deployment granularity    | Monorepo vs per-component deployable?                 |
| Secrets management        | How are credentials stored and rotated?               |
| Data retention            | How long do we keep data? Raw vs processed?           |
| Rollback strategy         | Blue/green, canary, or redeploy?                      |
| Observability depth       | Metrics-only, or logs + traces + dashboards too?      |

### Engineering Principles Audit

Applied to every design. References established engineering principles.
The audit is a radar, not a gate — principles that don't apply to the
problem context should be marked N/A with a one-line justification.

| Principle               | Check                                                    |
|-------------------------|----------------------------------------------------------|
| Config in env (12F-III) | All config via env vars; no hardcoded credentials         |
| Backing services (12F-IV)| DBs/queues accessed via URL in config; swappable          |
| Disposability (12F-IX)  | Fast startup, graceful SIGTERM drain, idempotent jobs     |
| Logs as streams (12F-XI)| Structured JSON to stdout; correlation IDs                |
| Dev/prod parity (12F-X) | Same backing service types in all environments            |
| SRP (SOLID)             | Each component has one reason to change                   |
| OCP (SOLID)             | Extensible via parameters/interfaces, not forking          |
| DIP (SOLID)             | Depend on abstractions (interfaces, template names)       |
| CAP tradeoff            | Explicit CP vs AP choice for each data store               |
| Postel's Law            | API accepts liberally, validates strictly                 |
| Unix Philosophy         | Components do one thing; compose via pipes/events          |
| Blast Radius            | Failure contained to cell/shard/canary cohort              |
| Single Writer (DB/Svc)  | One service owns the DB; all access via its API             |
| Circuit Breaker         | Every external API call wrapped in per-dependency breaker   |
| Idempotency             | Retried operations safe to repeat; idempotency keys where needed |
| Retry/Backoff/Jitter    | Retry policy has explicit backoff + jitter; not blind retry |
| Resource Governance     | CPU/memory limits set; DB connection pools bounded; no unbounded growth |
| Deployment Safety       | Manifests validated in CI; deployment strategy appropriate    |
| Fix the Service         | Problem solved in service, not by changing shared config    |
| Platform Portability    | Platform dependencies explicit; migration cost estimated         |
| Provider Volatility     | External API changes localised; provider swap = one component    |
| Deprecation Readiness   | Cleanup steps documented; data migration/deletion planned        |

Violations are NOT blocked — they're flagged with rationale required.

When applying specific principle sets (resilience, platform hygiene,
lifecycle), ask "which of these apply to this context?" before adding
them to the design. Not every principle applies to every problem.
N/A with justification is a valid audit outcome.

## Output Schema

```markdown
# Solution Architecture: <brief title>

## Problem Summary
<1 paragraph restating the Problem Brief's Context + Goals>

## Component Topology
<ASCII diagram showing components, data flow direction, protocols>

### Components

| Component              | Tech              | Responsibility                       |
|------------------------|-------------------|---------------------------------------|
| <name>                 | <stack element>   | <1-sentence role>                     |

### Data Flow

1. <step 1: source → destination, protocol, payload shape>
2. <step 2>
...

## Tradeoff Analysis

### Tradeoff 1: <name>
- **Decision:** <chosen option>
- **Alternatives considered:** <other options + why rejected>
- **Rationale:** <why this choice for this problem>

### Tradeoff 2: <name>
...

## Chosen Approach

### Topology Diagram
<Refined ASCII diagram with annotations>

### Key Design Decisions
- <Decision 1 with rationale>
- <Decision 2 with rationale>

### Component Details

#### <Component Name>
- **Purpose:** <what it does>
- **Inputs:** <data sources, API calls, queue messages>
- **Outputs:** <data destinations, API responses, queue messages>
- **Failure modes:** <what breaks + blast radius>
- **Observability:** <metrics, logs, traces, dashboard location>

### API Design (if applicable)
- **Endpoints:** <method, path, purpose>
- **Auth:** <mechanism>
- **Rate limiting:** <if needed>

## Observability Strategy

Every component must be visible in production. This table covers the
full observability surface: metrics, logs, traces, dashboards, and alerts.

| Layer          | Metrics                | Logs                       | Traces              | Dashboard          | Alert Condition              | Severity |
|----------------|------------------------|----------------------------|---------------------|--------------------|------------------------------|----------|
| API            | success_rate, p99_lat  | request log (level=info)   | inbound span        | API Dashboard      | success_rate < threshold     | page     |
| API            | error_rate             | error log (level=error)    | error span          | API Dashboard      | error_rate > threshold       | page     |
| Queue          | depth, age             | enqueue/dequeue events     | producer → consumer | Queue Dashboard    | oldest_message_age > limit   | page     |
| Scheduler      | last_tick_age          | tick log (level=info)      | tick span           | Scheduler Dashboard| last_tick_age > 2× interval  | notify   |
| Data sources   | source_errors          | fetch error (level=warn)   | fetch span          | Sources Dashboard  | error_rate > threshold       | notify   |
| Engine         | build_duration         | build log (level=info)     | build span          | Engine Dashboard   | duration > expected × 2     | notify   |

## Failure Mode Analysis

| Component           | Failure                    | Impact                     | Mitigation                      |
|---------------------|----------------------------|----------------------------|----------------------------------|
| <each component>    | <specific failure>         | <blast radius>             | <design choice or recovery>      |

## Complexity Budget

| New components     | Count |
|--------------------|-------|
| Workflows/jobs     | N     |
| Deployables        | N     |
| Database tables    | N     |
| Services           | N     |
| Queue topics       | N     |
| **Total**          | **N** |

<Flag if total > 5 with justification or simplification suggestion>

## Deployment & Lifecycle

- **Deployment:** How each component ships, repo structure, promotion path
- **CI/CD:** Pipeline stages, test gates, IaC parity (dev/prod)
- **Rollback:** How to revert each component
- **Deprecation:** How to decommission (data cleanup, DNS, queue drain)

## Open Questions for Next Phase

- <Questions the scope decomposition will need to answer>

## Third-Party Dependency Catalog

Every external dependency the design relies on. Scannable at a glance
for future reviewers assessing vendor risk and coupling.

| Dependency          | Type          | Version/Pinned? | Coupling | If It Disappears...                           |
|---------------------|---------------|-----------------|----------|------------------------------------------------|
| <name>              | API/Lib/SaaS  | v1.2.3 / latest | Tight/Loose | <replacement path or migration cost>           |

- **Tight coupling:** The service can't function without it. Replacement
  requires code changes.
- **Loose coupling:** Behind an interface. Replacement is a new
  implementation, no architectural changes.

## Architectural Decision Records / Assumption Log

Every design bets on things that might not hold. ADRs capture these bets
as lightweight notes — not gates, not blockers, just surfacing what
we're assuming so the next person knows what breaks if the world changes.

Format per decision:

```markdown
### ADR-00N: <decision title>

- **Decision:** <what we chose and why>
- **Alternatives considered:** <what we rejected and why>
- **Assumptions:** <what must remain true for this to be the right choice>
- **Consequences if assumption breaks:** <what happens, what we'd do>
- **Fitness check:** <how we'd know this assumption is at risk>
```

Mandatory ADRs for every design:

1. **Platform choice:** Why this infrastructure/stack for this service?
   What would trigger a re-evaluation?
2. **Observability strategy:** Why this depth of observability (metrics
   only? logs? traces?)? What's the cost of not having each pillar?
   Which dashboards will operators use?
3. **Third-party dependencies:** Every external API, library, or service
   the design depends on. For each: version (if pinned), what happens
   if it changes/disappears, how tightly coupled we are.
4. **Data store choice:** Why this database for this data? What access
   patterns drove the choice? What scale assumptions are baked in?

Additional ADRs should be created for any decision that:
- Involves a tradeoff where the alternatives were credible
- Introduces a new dependency (library, service, pattern)
- Has a blast radius that spans multiple teams or services
- Could be questioned 6 months from now ("why did we do it this way?")

ADRs are notes, not gates. An ADR with "assumption: this library
remains maintained" is a flag for future reviewers, not a reason to
reject the design.

## Common Pitfalls

1. **Designing for scale that doesn't exist.** Low-bandwidth weekly
   report doesn't need event sourcing, Kafka, or microservices. Push
   back on over-engineering.
2. **Skipping failure mode analysis.** "It's dev phase" isn't an
   excuse — failure modes surface during dev and shape monitoring
   design. Even a dev service has failure modes.
3. **Observability as an afterthought.** The observability strategy table
   must be filled before the design is complete. Every component
   that doesn't have metrics, logs, and a dashboard is invisible in
   production. Observability is a design-time decision, not an
   implementation detail.
4. **Ignoring prior art.** If internal libraries exist for queue,
   observability, or data patterns, the design MUST use them.
   Don't reinvent.
5. **Over-abstracting the API.** "We'll add a frontend someday" →
   design the API for the ACTUAL consumers first. Future
   extensibility is a secondary concern.
6. **No deployment model.** Deployable structure, repo layout, and
   promotion path are architecture decisions, not implementation
   details. Design them here.
7. **Blast radius by omission.** "One service" can still mean "one
   service that calls 4 external APIs." Trace every external
   dependency.
8. **Database access sprawl.** Multiple components connecting directly
   to the same database creates schema coupling and credential sprawl.
   The API Service pattern — single component with DB access, all others
   call the API — isolates schema changes and reduces credential
   surface area. Apply whenever more than one component reads/writes
   the same data store.
9. **Checklist reflex over relevance reflex.** The principles audit
   table is a radar, not a gate. Not every principle applies to every
   problem. A circuit breaker on a once-weekly cron job would take 5
   weeks to trip — the "fast failure" benefit doesn't materialise at
   that cadence. When asked to "apply the resilience lens," ask which
   principles apply to this context, not "how do I tick all the boxes."
   N/A with a one-line justification is a valid audit outcome.
10. **Auditing after topology, not before.** The principles audit should
    run against the Problem Brief's constraints before designing the
    topology. If a constraint conflicts with a principle, surface it
    early. Don't design first and audit later — that leads to patching
    violations instead of preventing them.

## Verification Checklist

- [ ] Problem Summary references the brief
- [ ] Component topology is a diagram (ASCII) with data flow
- [ ] At least 2 tradeoffs analyzed from the catalogue
- [ ] Every component has failure mode + observability entry (metrics, logs, dashboard)
- [ ] Observability strategy table covers all layers with metrics, logs, traces, dashboard, and alerts
- [ ] Complexity budget counted; flag if > 5 new components
- [ ] Deployment model specified
- [ ] Prior art leveraged where applicable
- [ ] Open Questions section filled with handoff items
- [ ] ADRs created for platform choice, observability strategy, 3rd-party dependencies, and data store choice
- [ ] Every 3rd-party dependency catalogued with version, coupling tightness, and replacement path
- [ ] ADRs are notes/surfacings, not blockers — assumptions documented, not gated
