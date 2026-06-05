# SRE Pipeline

A composable 5-phase pipeline for turning fuzzy problem statements into
engineer-ready Jira tasks. Each phase is a standalone skill — use them
together for full end-to-end planning, or pick individual phases.

## Pipeline

```
01 Problem Clarification     →   structured problem brief
02 Solution Architecture      →   component topology, tradeoffs, monitoring, ADRs
03 Scope Decomposition        →   independently deliverable epics with ACs
04 Task Breakdown             →   concrete tasks with context, steps, DoD, links
05 Output Formatting (Jira)   →   JSON / CSV / jira-cli import
```

Each phase produces a document that the next phase consumes. Phases
can also be used standalone — feed your own problem brief into 02, your
own epics into 04, etc.

## Skills

| # | Skill | Directory |
|---|-------|-----------|
| 1 | Problem Clarification | [`01-problem-clarification/`](01-problem-clarification/) |
| 2 | Solution Architecture | [`02-solution-architecture/`](02-solution-architecture/) |
| 3 | Scope Decomposition | [`03-scope-decomposition/`](03-scope-decomposition/) |
| 4 | Task Breakdown | [`04-task-breakdown/`](04-task-breakdown/) |
| 5 | Output Formatting (Jira) | [`05-jira-output-formatting/`](05-jira-output-formatting/) |

## Design

- **Composable.** Each skill works standalone. No hard pipeline coupling.
- **Self-contained.** Every task card includes enough context that an
  engineer can work from the ticket alone.
- **Setup-driven.** Each skill asks for domain context upfront (tech
  stack, deployment model, ops standards) rather than baking in
  assumptions.
- **Engineer-first.** Tasks are written for the person opening the ticket
  at 9am, not for the architect who designed the system.

## Usage

Start at Phase 1 with a raw problem statement:

> "The auth service keeps tripping its circuit breaker under peak load
> and we don't know why"

Or jump in at any phase with your own artifacts:

> "Here's a problem brief — give me a solution architecture"

> "Here's an architecture doc — decompose it into epics"

## AI Agent Integration

These skills were authored for use with AI coding agents (Claude Code,
Hermes Agent, etc.). Each SKILL.md contains:
- Trigger conditions (when to use / when not to)
- Input/output schemas
- Interactive interview flows
- Verification checklists
- Common pitfalls

## License

MIT
