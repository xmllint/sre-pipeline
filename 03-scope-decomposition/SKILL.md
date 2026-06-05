---
name: scope-decomposition
description: Take a solution architecture and decompose it into independently deliverable epics with acceptance criteria, dependencies, and effort estimates.
version: 1.1.0
author: Hermes Agent + Joe
license: MIT
---

# Scope Decomposition

## Overview

Takes a completed Solution Architecture and decomposes it into epics —
independently deliverable work areas with acceptance criteria, SLO
linkage, dependencies, and effort estimates. Each epic maps to one or
more architecture components, and every acceptance criterion traces
back to an SLO, a failure mode mitigation, or a deployment requirement
from the architecture.

## When to Use

- A Solution Architecture exists and needs epic-level decomposition
- You need to estimate effort and sequence work across multiple sprints
- You want each epic independently deliverable (mergeable, testable,
  demoable without waiting for other epics)

**Don't use when:**
- No Solution Architecture exists — run solution architecture first
- The scope is a single component with no decomposition needed
- You're doing task-level breakdown — that's the next phase

## Setup (first use)

Ask the user:

1. **Sprint cadence** — how long are sprints? How many teams?
2. **Effort scale** — what does S/M/L/XL mean in this org?
3. **Priority definitions** — what's P0 vs P1 vs P2?
4. **Tracking system** — Jira? Linear? Something else? (affects epic
   field names but not the decomposition itself)

## Input

Solution Architecture document. The skill reads:
- **Component Topology** → which components become epics
- **Observability Strategy** → acceptance criteria for metrics, logs, traces, dashboards
- **Failure Mode Analysis** → acceptance criteria for resilience
- **Deployment & Lifecycle** → acceptance criteria for delivery
- **SLIs & Success Criteria** (from Problem Brief) → SLO linkage per epic
- **ADRs** → assumptions to validate per epic

## Output Schema

```markdown
# Scope Decomposition: <brief title>

## Epic 1: <title>
- **Description:** <what, why, scope — 2-3 sentences>
- **Components:** <which architecture components this covers>
- **Acceptance Criteria:**
  - [ ] <testable, binary pass/fail criterion>
  - [ ] <criterion>
- **SLO Linkage:** <which SLI/SLO this epic protects>
- **Dependencies:** <other epics or external teams, what blocks this>
- **Blocks:** <which epics depend on this>
- **Estimated effort:** <S/M/L/XL>
- **Priority:** <P0=blocks everything, P1=core deliverable, P2=nice-to-have>

## Epic Dependency Graph
<ASCII diagram showing epic ordering>

## Epic Sequencing
<Suggested sprint/phase ordering with rationale>
```

## Epic Design Rules

1. **Independent delivery:** Each epic should produce something mergeable,
   testable, and demoable. No epic should require another epic to be
   complete before its own acceptance criteria can be validated.
2. **SLO traceability:** Every epic touching a production path must
   list which SLO it protects or enables. Epics that don't touch
   production paths (testing, docs) are exempt.
3. **Acceptance criteria are tests:** Every criterion must be a binary
   pass/fail statement. "The API is fast" → "API P99 latency < 100ms
   under load test with 10 concurrent requests."
4. **Dependency graph must be a DAG:** No circular dependencies between
   epics. If two epics depend on each other, they're one epic.
5. **Effort sanity check:** If an epic is XL, split it. If an epic is S
   but has 10 acceptance criteria, it's not S — it's under-scoped.
6. **Priority reflects blast radius:** P0 = its absence blocks other
   epics from being testable. P1 = core deliverable but other work can
   proceed. P2 = quality-of-life, docs, polish.

## Engineering Guardrails

- **Infrastructure before code.** The platform, IAM, databases, and
  deployment config must exist before services can deploy. Make
  infrastructure an explicit epic with its own acceptance criteria.
- **Observability is not an afterthought.** The observability epic has
  acceptance criteria traced to every alert in the observability strategy
  table, plus metrics instrumentation, log statements, and dashboard
  configuration. Without it, services are invisible in production.
- **Backfill and recovery are features.** If the architecture has retry,
  DLQ, or state machine states, those need acceptance criteria. "Queue
  stall triggers an alert" is an acceptance criterion, not an
  implementation detail.

## Common Pitfalls

1. **Epic = component.** Not every component is an epic. Components that
   are tightly coupled (API Service + database schema) belong in one
   epic. Components that can be built and tested in isolation are
   separate epics.
2. **Vague acceptance criteria.** "Report is generated weekly" is not
   testable. "Scheduler fires, worker processes a message with mock
   sources, output is created with expected content" is.
3. **Sequencing everything serially.** If Epic B only needs Epic A's
   interface (not its implementation), Epic B can start with a mock.
   Parallelism is how teams ship faster.
4. **No SLO linkage.** If an epic touches the API, it has SLO
   implications. State them. If you can't trace an epic to an SLO,
   ask whether it's necessary.
5. **Forgetting the glue.** IAM roles, deployment configs, DNS records,
   queue configuration — these are infrastructure epics, not
   afterthoughts.

## Verification Checklist

- [ ] Every architecture component is covered by at least one epic
- [ ] Every epic has binary pass/fail acceptance criteria
- [ ] Dependency graph is acyclic
- [ ] P0 epics are true blockers (nothing else testable without them)
- [ ] At least one epic covers infrastructure/platform
- [ ] At least one epic covers monitoring/observability (metrics, logs, traces, dashboards, alerts)
- [ ] At least one epic covers testing (integration, e2e)
- [ ] Every SLO is traced to at least one epic
- [ ] Effort estimates are consistent (no XL epics, no under-scoped S epics with 10 criteria)
- [ ] Epics are independently deliverable (mock-able interfaces where needed for parallelism)
