---
name: jira-output-formatting
description: Take structured task data and format it for Jira import — JSON (REST API), CSV (bulk import), and jira-cli commands. Preserves epic-task relationships and engineer-first context.
version: 1.1.0
author: Hermes Agent + Joe
license: MIT
---

# Output Formatting (Jira)

## Overview

Takes a Task Breakdown and formats it for Jira import. Three output
formats: JSON (Jira REST API), CSV (bulk import), and `jira-cli` shell
commands. Preserves epic-task parent/child relationships, all
engineer-first context (Context, Implementation, DoD, Links), and
acceptance criteria.

This is a **serialization layer** — it does not modify task content.
Content quality is the task breakdown's responsibility.

## When to Use

- Task Breakdown is complete
- You need to create epics and tasks in Jira programmatically
- You want dry-run preview before creating issues
- You're importing into a Jira project with specific field mappings

**Don't use when:**
- No Task Breakdown exists — run task breakdown first
- You're editing task content — that's task breakdown
- You need interactive creation — that's `jira-cli` in manual mode

## Setup (first use)

Ask the user:

1. **Jira instance URL** — e.g., `https://company.atlassian.net`
2. **Project key** — e.g., `INC`, `SRE`, `PLAT`
3. **Issue type IDs** — Epic, Task, Story, Bug (instance-specific)
4. **Custom field IDs** — Epic Name, Epic Link, Story Points, Sprint
5. **Priority scheme** — maps to P0/P1/P2
6. **Label convention** — team format (lowercase, hyphens?)
7. **Auth** — classic API token (see `references/jira-api-auth.md`)

Always run pre-flight checks from `references/atlassian-auth-quirks.md`
before generating any output.

## Input

Task Breakdown (tasks with Context, Implementation, DoD, Links,
estimates, dependencies).

The skill also reads:
- Scope Decomposition (epic descriptions, ACs, priorities)
- Jira project configuration (from Setup)

## Output Formats

### 1. JSON (Jira REST API)

Full programmatic import. Each epic is a Jira Epic issue type; each task
is a Task or Story. The `description` field uses Jira's Atlassian Document
Format (ADF) for rich text, or plain text with wiki markup.

```json
{
  "project": "INC",
  "issues": [
    {
      "summary": "Epic: Infrastructure & Platform",
      "description": { "... ADF doc ..." },
      "issuetype": {"name": "Epic"},
      "labels": ["platform"],
      "customfield_10014": "INC-1"
    },
    {
      "summary": "T-01: Provision Database Schema",
      "description": { "... ADF doc with Context, Implementation, DoD, Links ..." },
      "issuetype": {"name": "Task"},
      "parent": {"key": "INC-1"},
      "labels": ["infra", "database"],
      "timetracking": {"originalEstimate": "2d"}
    }
  ]
}
```

### 2. CSV (Bulk Import)

Simpler format for Jira's CSV importer. Columns map to Jira fields.
Preserves epic linkage via Epic Link or Epic Name column.

```
Summary,Issue Type,Epic Name,Description,Labels,Priority,Original Estimate
Epic: Infrastructure & Platform,Epic,,<description>,platform,P0,
T-01: Provision Database Schema,Task,Infrastructure & Platform,<description>,infra|database,P0,2d
```

### 3. jira-cli Shell Commands

For interactive or scripted creation via `jira-cli`. Each command is
self-contained; dependencies (epic keys) are resolved by running epics
first, then substituting the returned keys.

```bash
# Create epic
jira epic create --name "Infrastructure & Platform" \
  --description "$(cat epic-1-desc.txt)" \
  --labels platform

# Create task (after epic key is known: INC-123)
jira issue create --type Task --summary "T-01: Provision Database Schema" \
  --description "$(cat t-01-desc.txt)" \
  --parent INC-123 \
  --labels infra,database \
  --priority High
```

## Field Mappings

These should be configurable per Jira project. Defaults:

| Task Field        | Jira Field            | Notes                          |
|------------------|-----------------------|--------------------------------|
| Task title       | Summary               | "T-0N: <title>"                |
| Context          | Description (top)     | "## Why" section               |
| Implementation   | Description (middle)  | "## How" section               |
| Definition of Done| Description (bottom)  | "## Done" section — checkboxes |
| Links            | Description (bottom)  | "## References" section        |
| Estimate (days)  | Original Estimate     | "2d", "4h" format              |
| Depends on       | Issue Links           | "blocks" / "depends on"        |
| Priority         | Priority              | P0→Highest, P1→High, P2→Medium |
| Epic             | Parent / Epic Link    | Epic Name or Epic Key          |

## Common Pitfalls

1. **Not resolving epic keys before creating tasks.** JSON API creates
   epics and tasks in one payload with parent references by temp ID.
   CSV and jira-cli require epics created first, keys captured, then
   tasks created with parent key.
2. **Plain text in description when Jira expects ADF.** The JSON API
   accepts `description` as a string (plain text) or an ADF document.
   If your Jira instance requires ADF, the description must be
   structured. Check instance config before generating.
3. **Field name collisions.** Jira custom field names vary by instance.
   "Epic Name" might be `customfield_10014` or `customfield_10102`.
   Map fields before generating, not after.
4. **Labels exceeding Jira limits.** Jira has a max label count per
   issue. Our tasks have 2-3 labels — safe, but verify.
5. **CSV encoding.** Jira CSV importer expects UTF-8. Multi-line
   description fields must be properly quoted and escaped.
6. **OAuth token ≠ API token for CLI tools.** Atlassian OAuth tokens
   (with scopes, generated from developer console) are NOT the same as
   API tokens (generated from profile security page). jira-cli and
   Basic auth require the latter. See `references/jira-api-auth.md`.
7. **API access disabled at site level.** If `curl /rest/api/3/myself`
   returns 401 with a valid token, the site admin may have disabled
   API access. See `references/atlassian-auth-quirks.md`.

## Verification Checklist

- [ ] Output format matches target Jira instance (JSON, CSV, or jira-cli)
- [ ] Epic-task parent/child relationships preserved
- [ ] All Context, Implementation, DoD, Links sections in description
- [ ] Estimates in correct format ("2d", "4h")
- [ ] Priority mapping correct (P0→Highest, P1→High, P2→Medium)
- [ ] Labels follow team convention
- [ ] Dry-run mode confirmed (no issues created until explicitly requested)
- [ ] Field mappings verified against Jira instance custom field IDs
- [ ] Pre-flight auth checks passed (see references)
