---
name: task-breakdown
description: Break epics into concrete, engineer-ready tasks with context, steps, definition of done, and links. Self-contained — no pipeline dependency.
version: 1.1.0
author: Hermes Agent + Joe
license: MIT
---

# Task Breakdown

## Overview

Takes epics and explodes them into concrete, engineer-ready tasks. Every
task includes:

1. **Context** — why this exists, what SLO/outcome it protects, what
   happens if it's not done
2. **Implementation steps** — specific guidance: files, functions,
   patterns, test approach
3. **Definition of Done** — broader than acceptance criteria: tests pass,
   CI green, deployed, metrics/logs/traces visible, dashboard updated,
   runbook updated
4. **Links** — to architecture docs, ADRs, monitoring tables, dependency
   catalogs, relevant standards

The test: an engineer opening this ticket at 9am should know what to do,
why it matters, and how to know they're done — without reading the
architecture doc.

## When to Use

- Epics exist and need sprint-ready task cards
- You need day-level estimates (1-5 days per task)
- You want tasks to be independently assignable and completable
- The task output will feed into a ticketing system (Jira, Linear, etc.)

**Don't use when:**
- No epics exist — define the scope first
- You need acceptance criteria only — epics should already carry those
- You're building Jira import formats — that's a separate formatting step

## Setup (first use)

Before breaking down tasks, ask the user:

1. **Tech stack** — language, framework, infra tools (Terraform? K8s?
   CI system?)
2. **Deployment model** — how do things ship? PR merge → CI → CD?
   Manual approval? Feature flags?
3. **Operational standards** — what does "done" mean here? Dashboard
   update? Runbook entry? Alert configuration? Metrics instrumentation?
   Log statements? Trace spans?
4. **The epics** — summary, acceptance criteria, dependencies

Record these as domain context. Every task card will reference them.

## Input

- Epics with acceptance criteria and dependencies
- Domain context from setup questions (tech stack, deploy model, ops
  standards)
- Any existing architecture docs, ADRs, or monitoring strategy (optional
  but improves task specificity)

## Output Schema

Each task uses this structure:

```markdown
### T-0N: <task title>

**Epic:** <parent epic>
**Estimate:** <1-5 days>
**Depends on:** <task IDs or "none">

**Context**
<1-2 sentences: why this task exists, what SLO or failure mode it
addresses, what breaks if it's skipped>

**Implementation**
1. <Concrete step with file path, function name, or pattern hint>
2. <Concrete step>
3. <Test approach>

**Definition of Done**
- [ ] <Condition beyond AC — e.g. CI green, metrics/logs/traces visible, dashboard updated, deployed>
- [ ] <Condition>

**Links**
- Architecture: <section reference>
- ADR: <ADR-00N if applicable>
- Observability: <metric, log stream, trace span, or dashboard reference>
- Standards: <relevant standard + section>
```

## Task Design Rules

1. **1-5 days per task.** If a task is < 1 day, group it with related
   work. If > 5 days, split it. An engineer should be able to complete
   a task within a sprint week.
2. **Every task is independently verifiable.** No "T-02 depends on T-01
   being done to know if T-02 works." Mock interfaces, test stubs, and
   integration test fixtures make tasks independently testable.
3. **Implementation steps are prescriptive, not vague.** "Set up queue"
   → "In `cmd/consumer/main.go`, initialise internal queue client with
   pinned version, configure DLQ topic, write connection health check."
4. **Definition of Done includes operational readiness.** Tests pass is
   minimum. Metrics visible, dashboard updated, runbook entry exists,
   deployment validated — these are part of done.
5. **Links make context self-contained.** An engineer should not need to
   open another document to understand the task. Key context from
   architecture and ADRs is inlined or summarised.
6. **Dependency graph must be a DAG per epic.** Within an epic, tasks
   may have dependencies on each other. Across epics, task dependencies
   are on epic completion, not individual tasks.

## Task Type Modifiers

Some tasks need additional fields based on their nature:

### Infrastructure tasks
- **Rollback plan:** How to revert if this breaks the cluster or shared
  resources
- **Blast radius:** What else could this affect if misconfigured?

### Migration tasks
- **Existing state:** What's running now that this replaces?
- **Cutover plan:** How to switch from old to new without data loss or
  duplicate processing

### Observability tasks
- **Metrics:** What specific metric is being added? What does it measure?
  Where does it surface (dashboard, alert)?
- **Logs:** What log statements are being added? At what level? What
  structured fields?
- **Traces:** What spans? What service boundaries do they cross?
- **Dashboard:** Which dashboard is updated? Link to it.
- **Alert review:** Before adding a new alert, check it doesn't
  duplicate an existing one. Document which existing alerts this
  supplements or replaces.

## Common Pitfalls

1. **Vague steps that assume knowledge.** "Implement retry logic" assumes
   the engineer knows which library, which backoff strategy, which jitter
   factor. The task card should specify all three.
2. **DoD that's just ACs rephrased.** "Tests pass" is an AC. The real
   DoD is: test passes, CI is green, deployment succeeded, metrics/logs/
   traces are visible in dashboard, runbook is updated.
3. **No rollback plan for infra tasks.** Every infrastructure change
   needs a tested rollback. If the engineer doesn't know how to undo it,
   the task isn't ready.
4. **Forgetting operational handoff.** The task that builds the API
   endpoint should also update the runbook. The task that adds a metric
   should also add it to the dashboard. These are one task, not two.
5. **Links that are just filenames.** "See architecture doc" is useless.
   "Architecture: Section 'Component Details / API Service' — endpoints
   and auth model" is actionable.

## Verification Checklist

- [ ] Every epic has tasks covering all its acceptance criteria
- [ ] Every task has Context, Implementation steps, DoD, and Links
- [ ] No task exceeds 5 days
- [ ] Dependency graph per epic is acyclic
- [ ] Infra tasks have rollback plans
- [ ] Migration tasks have cutover plans
- [ ] Every task's DoD includes operational readiness (not just "code works")
- [ ] Links reference specific sections, not whole documents
- [ ] Tasks are self-contained — an engineer can complete one without
      reading the other 20 tasks first
