# Monitor Loop

## Overview

The Monitor Loop is the observational backbone of the Triple Loop Architecture. It continuously watches the repository state, task backlog, and outputs from the Build and Verify loops to maintain a current, accurate picture of what the autonomous system needs to do next.

Think of the Monitor Loop as the **product manager and scrum master** of the AI team: it reads requirements, tracks progress, detects blockers, and feeds context-enriched work packages to the Build Loop.

---

## Responsibilities

| Responsibility           | Description                                                   |
|--------------------------|---------------------------------------------------------------|
| Task ingestion           | Read new tasks from the task queue (Issues, YAML manifest)    |
| Context gathering        | Identify and read relevant source files for each task         |
| State reconciliation     | Compare expected state vs. actual state of the codebase       |
| Loop feedback processing | Consume Build and Verify loop outputs and adjust the task plan|
| Escalation management    | Detect when human intervention is needed and alert reviewers  |
| Idle monitoring          | Watch for unexpected changes (drift, security events)         |

---

## Monitor Loop Execution Flow

```
┌─────────────────────────────────────────────────────────┐
│                   Monitor Loop Cycle                    │
│                                                         │
│  1. Read task queue                                     │
│  2. Select next highest-priority task                   │
│  3. Gather context (files, issues, recent commits)      │
│  4. Build context package                               │
│  5. Dispatch context package to Build Loop              │
│  6. Wait for Build Loop status (timeout: 15 min)        │
│  7. Process Build Loop result:                          │
│     ├── success → wait for Verify Loop result           │
│     ├── failure → enrich context with error; re-dispatch│
│     └── timeout → escalate to human reviewer            │
│  8. Process Verify Loop result:                         │
│     ├── pass → mark task complete; proceed to next task │
│     ├── fail → add regression notes; re-queue task      │
│     └── partial → log coverage gaps; continue           │
│  9. Sleep until next cycle interval                     │
└─────────────────────────────────────────────────────────┘
```

---

## Context Package Format

The Monitor Loop produces a structured context package that the Build Loop consumes:

```json
{
  "task_id": "TASK-1042",
  "title": "Add JWT expiry refresh logic to auth service",
  "description": "When a JWT token is within 5 minutes of expiry, automatically issue a refresh token.",
  "priority": "high",
  "context_files": [
    "src/auth/token.ts",
    "src/auth/middleware.ts",
    "src/auth/__tests__/token.test.ts",
    "docs/auth-design.md"
  ],
  "related_issues": ["#102", "#98"],
  "recent_commits": [
    {
      "sha": "a1b2c3d",
      "message": "Fix token parsing edge case",
      "files_changed": ["src/auth/token.ts"]
    }
  ],
  "constraints": [
    "Must not break existing token validation tests",
    "Must work with RS256 and HS256 algorithms",
    "Refresh window configurable via environment variable"
  ],
  "previous_errors": [],
  "attempt_number": 1
}
```

On retries, `previous_errors` is populated with structured error output from the previous Build Loop attempt.

---

## Context Gathering Strategy

The Monitor Loop uses the following heuristics to select context files:

1. **Direct references**: Files mentioned in the task description or related issues
2. **Git blame**: Files most recently modified by changes related to the task keyword
3. **Import graph**: Files that import or are imported by the primary file being modified
4. **Test files**: Test files co-located with the primary implementation file
5. **Documentation**: Markdown files in `docs/` that match the task domain

Context is truncated to the most relevant sections if file count or token size would exceed the Copilot CLI context window.

---

## Escalation Triggers

The Monitor Loop escalates to a human reviewer under the following conditions:

| Trigger                          | Action                                               |
|----------------------------------|------------------------------------------------------|
| Build fails > 3 consecutive times | Post comment on GitHub Issue; label as `needs-human` |
| Verify fails with unknown error  | Open a new bug issue with full error context         |
| Conflicting requirements detected| Flag issue as `blocked`; request clarification       |
| Security vulnerability found     | Immediately pause loop; alert via Slack/email        |
| Task older than 48 hours         | Escalate with full progress summary                  |

---

## Idle Monitoring Mode

When all tasks are complete or the queue is empty, the Monitor Loop enters **idle monitoring mode**:

- Polls GitHub Issues every 15 minutes for new tasks
- Watches for unexpected file changes (e.g., manual edits that may conflict with autonomous changes)
- Monitors CI pipeline for unexpected failures in the main branch
- Reports drift if the production state no longer matches the expected state

---

## Configuration

```yaml
monitor:
  interval_seconds: 120          # Active mode: check every 2 minutes
  idle_interval_seconds: 900     # Idle mode: check every 15 minutes
  max_context_files: 20          # Max files per context package
  max_context_tokens: 8000       # Max tokens in context window
  escalation_retry_limit: 3      # Escalate after N consecutive failures
  task_timeout_hours: 48         # Auto-escalate tasks older than N hours
  task_queue:
    source: github_issues        # or: yaml_manifest, json_api
    label_filter: "autonomous"   # only process issues with this label
    priority_field: "milestone"  # use milestone as priority signal
```

---

## Related Documents

- [Triple Loop Architecture](../architecture/triple-loop-architecture.md)
- [Build Loop](build-loop.md)
- [Verify Loop](verify-loop.md)
- [Monitor Loop Prompts](../prompts/monitor-loop-prompts.md)
