---
name: problem-clarification
description: Take a fuzzy engineering problem statement and run a structured multi-turn clarification interview to produce a scoped problem brief. Composable with downstream planning skills.
version: 2.1.0
author: Hermes Agent + Joe
license: MIT
---

# Problem Clarification

## Overview

Takes a raw problem statement — a vague alert, an incident postmortem
gap, a reporting request, a resilience concern — and runs a structured
multi-turn clarification interview to produce a **problem brief**: a
scoped document that downstream skills (solution architecture, scope
decomposition, task breakdown) can consume without re-asking the same
questions.

The output uses a fixed Markdown schema so composing skills can parse
it with a known contract.

## When to Use

- An incident review surfaces "we should prevent this class of failure"
  but nobody has scoped it
- A resilience audit flags a gap ("no rate limiting on auth endpoint")
- A reporting request arrives as "we need visibility into X" with no
  metrics defined
- You're about to write epics but the problem statement is a Slack
  thread or incident channel snippet
- A concern spans multiple services/teams and the boundaries are fuzzy

**Don't use when** the problem is already a fully-scoped epic with
acceptance criteria, SLIs, and runbook requirements — feed that directly
to scope decomposition instead.

## Setup (first use)

Ask the user for domain context before beginning the interview:

1. **System landscape** — what services, databases, infrastructure
   are in play? What's the tech stack?
2. **Observability stack** — what monitoring, logging, tracing, and
   dashboarding exists? What's the alerting pipeline?
3. **Operational norms** — who carries the pager? What's the incident
   response workflow? What SLOs exist?
4. **Constraints** — any hard technology boundaries? Compliance
   requirements? Team bandwidth limits?
5. **The problem statement** — as raw as it comes

Record these as domain context. They shape question prioritisation and
the brief's constraint section.

## Interactive Mode (default)

The skill runs a structured interview loop:

1. **Receive** the raw problem statement
2. **Extract** what's already clear (context, apparent goals, mentioned
   constraints, affected services) and present it back as a draft skeleton
3. **Ask targeted questions** in batches of 1-3, prioritized by:
   missing critical context → unclear success criteria/SLIs → unstated
   constraints → incident/on-call impact → scope boundaries →
   stakeholders/teams → data/reporting requirements
4. **Update** the skeleton after each answer batch
5. **Stop** when all sections reach "sufficient" clarity (see thresholds
   below), OR the user signals done, OR after 6 question rounds
6. **Emit** the final problem brief

### Question categories (prioritized)

| Priority | Category               | Example question                                      |
|----------|------------------------|-------------------------------------------------------|
| 1        | System/impact context  | "Which services are affected? What's the blast radius?"|
| 2        | SLIs & observability   | "What SLO does this protect? What metrics/logs/traces prove it works?"|
| 3        | Incident response      | "How is this detected? What runbook step changes?"   |
| 4        | Constraints            | "Any hard tech boundaries? Compliance requirements?"  |
| 5        | Scope boundaries       | "What's explicitly out of scope? One service or many?"|
| 6        | Stakeholders/teams     | "Who carries the pager? Who signs off?"               |
| 7        | Data/reporting         | "What data sources? Databases? Logs? Metrics?"        |
| 8        | Lifecycle              | "Is this a fix-and-forget or an ongoing concern?"     |

### Clarity thresholds

| Section              | Minimum bar                                              |
|----------------------|----------------------------------------------------------|
| Context              | Service(s) identified, failure mode or gap described     |
| Goals                | At least 1 measurable outcome (SLO, MTTR, error rate)    |
| SLIs & Observability | At least 1 SLO-linked condition; observability path identified (metrics/logs/traces/dashboard)|
| Incident Response    | Detection method AND runbook impact stated (even if "none")|
| Constraints          | Tech stack confirmed; at least 1 non-tech constraint      |
| Out of Scope         | At least 1 explicit exclusion                             |
| Stakeholders         | At least 1 team/role named + on-call ownership stated     |

## Problem Brief Schema (mandatory output)

```markdown
# Problem Brief: <1-line title>

## Context
<2-5 sentences: affected services, failure mode, why this matters now,
 blast radius, what broke or what's at risk>

## Goals
- <Measurable outcome: SLO target, MTTR reduction, error budget recovery>
- <Secondary outcome if applicable>

## SLIs & Observability
- [ ] <Concrete, verifiable condition tied to an SLO or alert>
- [ ] <Verification method: dashboard, load test, chaos experiment, canary>
- [ ] <Target metric: e.g. "p99 latency < 200ms under 2x peak load">
- **Observability path:** <how will we know this works? metrics? logs? traces? which dashboard?>

## Incident Response Impact
- **Detection:** <How will this be detected? New/moved alert? Dashboard?>
- **Runbook change:** <What runbook step is added/changed/removed?>
- **On-call burden:** <Expected change in alert volume, severity, after-hours pages>

## Constraints
- **Tech stack:** <the stack from Setup, with any deviations flagged>
- **Compliance/regulatory:** <SOC2, GDPR, data residency, etc. if applicable>
- **Capacity:** <Team bandwidth, timeline pressure, competing priorities>

## Out of Scope
- <Explicitly excluded work or services>
- <Boundary condition: "only service X, not service Y">

## Stakeholders
| Role              | Who           | Involvement                          |
|-------------------|---------------|--------------------------------------|
| Requester         | <name/team>   | Owns the outcome                     |
| On-call owner     | <name/team>   | Carries the pager for affected svc   |
| Approver          | <name/team>   | Signs off on approach                |
| Consumer          | <name/team>   | Uses the result (dashboard, runbook) |

## Raw Input
<Original problem statement, verbatim — preserved for traceability>

## Clarification Log
- Q1: <question> → A: <answer>
- Q2: <question> → A: <answer>
```

The schema is the **inter-skill contract**. Downstream skills depend on
these section names and the checklist format for success criteria. Don't
deviate without updating all consumers.

## One-Shot Mode

When the user explicitly requests single-pass (no interaction), do one
pass of enrichment without asking questions:

```
User: "One-shot: auth-service circuit breaker keeps tripping under load"
```

Fill every section you can from the statement. Mark unfilled sections
with `[NEEDS CLARIFICATION]`. Skip the Clarification Log. Add a
`## Gaps` section listing what would need a follow-up round.

## Engineering Guardrails

### Observability First
Every problem must answer: how will we know this works in production?
- **Metrics:** What numbers prove success? Where do they live?
- **Logs:** What log lines confirm correct behaviour? At what level?
- **Traces:** What spans connect this to upstream/downstream services?
- **Dashboards:** Which dashboard shows this? Is it linked from the runbook?

A solution without an observability path is incomplete — the brief must
specify how operators will confirm correct behaviour, not just that the
code compiles.

### Production-grade requirement
Every solution must consider:
- **Lifecycle:** Is this a permanent capability or a stopgap? How is it
  maintained, deprecated, or handed off?
- **Sustainability:** Will the on-call team understand this at 3am?
  Does it add toil or reduce it?
- **Complexity fatigue:** Does this add a new component outside the
  established stack? If yes, flag it for explicit justification.

### Blast radius awareness
Every problem scoping must consider:
- Can a failure cascade to other services or customers?
- Does this touch a critical path?
- If yes, the brief MUST include blast radius analysis and SLO linkage.

## Common Pitfalls

1. **Asking too many questions at once.** 1-3 per round max. More than
   3 and the user will answer only the last one.
2. **Premature solution design.** The brief describes WHAT and WHY,
   never HOW. "We need a circuit breaker" is a solution. Push back:
   "What failure mode does the circuit breaker protect against?"
3. **Vague SLIs.** "Make it reliable" → "What SLO does this protect?
   Availability? Latency? Error rate? What's the target?"
4. **Skipping incident response impact.** Every operational change touches
   on-call. Even a reporting project changes what's visible during
   an incident.
5. **Over-clarifying.** Stop at 6 rounds even if thresholds aren't
   all met. Mark gaps and move on.
6. **No lifecycle thinking.** Fix-and-forget doesn't exist. How is
   this maintained? Who owns it in 6 months?

## Verification Checklist

- [ ] Every section from the schema is present in the output
- [ ] At least 1 goal is tied to an SLO or measurable metric
- [ ] Observability path defined for the primary success criterion (which dashboard/metric/log stream?)
- [ ] At least 1 success criterion is a binary pass/fail check
- [ ] Incident Response Impact section filled (even if "none")
- [ ] Tech stack constraint stated and any deviation flagged
- [ ] At least 1 explicit exclusion in Out of Scope
- [ ] Stakeholders table has at least 1 team + on-call owner
- [ ] Raw Input section preserves the original statement verbatim
- [ ] No solution language in Goals or SLIs (WHAT, not HOW)
- [ ] Blast radius considered (explicitly noted even if "single service")
- [ ] Clarification Log records every Q&A round
