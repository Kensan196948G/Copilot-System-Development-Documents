# Autonomous Development Workflow

## Overview

This document describes the end-to-end workflow for running autonomous software development sessions with the Triple Loop system. It covers the full lifecycle from task creation to merged Pull Request, including decision points, human-in-the-loop checkpoints, and post-session cleanup.

---

## Workflow Summary

```
1. Task Preparation
   └── Create labeled GitHub Issues

2. Session Start
   └── Start loop orchestrator

3. Monitor Phase
   └── Monitor Loop selects highest-priority task
   └── Gathers context, builds context package

4. Build Phase
   └── Build Loop generates code via Copilot CLI
   └── Runs build; retries with error context if needed

5. Verify Phase
   └── Verify Loop runs tests, lint, security, coverage
   └── All gates pass → PR created
   └── Any gate fails → rework context sent back to Monitor

6. Human Review
   └── Engineer reviews auto-generated PR
   └── Approves and merges (or requests changes)

7. Session End
   └── All tasks complete or escalated
   └── Loop system shuts down gracefully
```

---

## Phase 1: Task Preparation

### Creating Effective Tasks

The quality of autonomous output is directly proportional to the quality of the task description. Use this template for GitHub Issues:

```markdown
## Task Description
[What needs to be done, in 2–4 sentences]

## Acceptance Criteria
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## Constraints
- Must not break existing behavior in [module]
- Must follow [coding standard or convention]
- Must include unit tests

## Context Files
<!-- Optional: list specific files the agent should focus on -->
- src/[relevant-file].ts
- src/[another-file].ts

## Related Issues
<!-- Optional: link related issues for context -->
- Depends on #42
- Related to #39
```

### Issue Labeling

Label your issue with the `autonomous` label to add it to the work queue. The system processes issues in priority order based on:

1. Milestone (earlier milestone = higher priority)
2. Issue creation date (older = higher priority)
3. Explicit `priority:high` label

---

## Phase 2: Session Start

Start a development session before the team's working hours so the system can make progress autonomously:

```bash
# Ensure your configuration is up to date
npx copilot-loops start --dry-run

# Start the session
npx copilot-loops start --daemon

# Verify the system is running
npx copilot-loops status
```

**Recommended session schedule:**
- Start: before work (e.g., 06:00)
- Human review window: start of day (e.g., 09:00)
- Evening run: after hours (e.g., 18:00–06:00 overnight)

---

## Phase 3: Monitor Phase

The Monitor Loop runs every 2 minutes (configurable) and performs the following steps:

1. **Poll task queue** – read all open issues labeled `autonomous`
2. **Select task** – pick the highest-priority task that is not already in progress
3. **Gather context** – identify relevant files using import graphs, git history, and issue references
4. **Build context package** – assemble a structured JSON package with task details, code snippets, constraints, and retry history
5. **Dispatch to Build Loop** – send the context package

### Monitoring Loop Output (Example)

```
[2024-01-15 06:02:00] [Monitor] Cycle 1 starting
[2024-01-15 06:02:01] [Monitor] Found 3 tasks in queue
[2024-01-15 06:02:01] [Monitor] Selected: TASK-42 "Add JWT expiry refresh logic" (priority: high)
[2024-01-15 06:02:03] [Monitor] Context gathered: 5 files, 3800 tokens
[2024-01-15 06:02:03] [Monitor] Dispatching to Build Loop
```

---

## Phase 4: Build Phase

The Build Loop receives the context package and generates code:

1. **Construct prompt** – assemble a detailed Copilot prompt with task context
2. **Invoke Copilot CLI** – call `gh copilot suggest` with the constructed prompt
3. **Apply changes** – write Copilot's output to the appropriate files
4. **Run build** – execute the project's build command
5. **Evaluate result**:
   - Success → commit and notify Verify Loop
   - Failure → parse error, enrich prompt, retry (up to `max_retries`)

### What Good Build Loop Output Looks Like

```
[2024-01-15 06:02:05] [Build] TASK-42 attempt 1/5
[2024-01-15 06:02:05] [Build] Prompt constructed (2100 tokens)
[2024-01-15 06:02:18] [Build] Copilot response received (450 tokens)
[2024-01-15 06:02:18] [Build] Applying changes to src/auth/token.ts
[2024-01-15 06:02:19] [Build] Running: npm run build
[2024-01-15 06:02:28] [Build] Build succeeded
[2024-01-15 06:02:29] [Build] Committed: autonomous/TASK-42 (a1b2c3d)
[2024-01-15 06:02:29] [Build] Notifying Verify Loop
```

---

## Phase 5: Verify Phase

The Verify Loop runs quality gates against the build artifact:

1. **Run tests** – full test suite
2. **Check coverage** – compare against threshold
3. **Run linter** – check for new violations
4. **Run security scan** – check for new vulnerabilities
5. **Produce quality report** – structured pass/fail for each gate
6. **Decide next step**:
   - All gates pass → open Pull Request
   - Any gate fails → send rework context to Monitor Loop

### Rework Cycle

When a gate fails, the system does not abandon the task. Instead:

1. Verify Loop produces a detailed failure report
2. Monitor Loop re-queues the task with the failure report appended to context
3. Build Loop receives enriched context including failure details
4. Build Loop attempts to fix the failing tests or lint issues

This cycle can repeat up to `max_retries` times before the task is escalated to human review.

---

## Phase 6: Human Review

When the Verify Loop passes all gates, it automatically opens a Pull Request:

```
autonomous: TASK-42 Add JWT expiry refresh logic

## Summary
Implemented JWT token refresh logic in src/auth/token.ts. When a token
is within 5 minutes of expiry, the middleware now automatically issues
a refresh token using the configured RS256 key.

## Changes
- src/auth/token.ts: Added refreshIfExpiring() function
- src/auth/middleware.ts: Integrated refresh logic into request pipeline
- src/auth/__tests__/token.test.ts: Added 4 test cases for refresh logic

## Quality Gates
- ✓ Tests: 146 passed, 0 failed
- ✓ Coverage: 83.2% (threshold: 80%)
- ✓ Lint: 0 new violations
- ✓ Security: 0 new CVEs

Generated by Copilot Autonomous System | Task: TASK-42 | Attempt: 2
```

### Human Reviewer Checklist

Before approving an autonomously generated PR:

- [ ] Code changes are logically correct and match the task requirements
- [ ] No unexpected files were modified
- [ ] Test cases cover meaningful scenarios, not just happy path
- [ ] No hardcoded values that should be configuration
- [ ] Performance characteristics are acceptable
- [ ] The code is readable and maintainable

---

## Phase 7: Session Management

### During a Session

```bash
# Check what the system is doing
npx copilot-loops status --watch

# See recent log output
npx copilot-loops logs --since 30m

# Check which tasks are complete
npx copilot-loops queue list
```

### Pausing the Session

```bash
# Gracefully pause (completes current task, then stops)
npx copilot-loops stop

# Pause a specific task without stopping the system
npx copilot-loops queue pause TASK-44
```

### Resuming After a Break

```bash
# Check for any tasks that were left in mid-flight
npx copilot-loops queue list

# Resume any paused tasks
npx copilot-loops queue resume TASK-44

# Restart the system
npx copilot-loops start --daemon
```

### Ending a Session

```bash
# Graceful stop
npx copilot-loops stop

# Review all PRs created during the session
gh pr list --state open --label autonomous

# Review any escalated tasks
gh issue list --label needs-human
```

---

## Handling Edge Cases

### Task Stuck in `in_progress`

If a task has been in `in_progress` for more than 30 minutes:

```bash
# Check what's happening
npx copilot-loops task logs TASK-42

# Force reset and retry
npx copilot-loops task reset TASK-42
```

### Build Loop in Infinite Retry Loop

If you see the same error repeatedly:

```bash
# Pause the task
npx copilot-loops queue pause TASK-42

# Review the error context
npx copilot-loops task context TASK-42

# Add more context manually and retry
npx copilot-loops task retry TASK-42 --add-context "See design doc at docs/auth-design.md for RS256 key format requirements"
```

### Escalating to Human Review

```bash
npx copilot-loops task escalate TASK-42 --reason "Requires architectural decision on token storage strategy"
```

---

## Related Documents

- [Copilot Start Guide](copilot-start-guide.md)
- [Loop Command Usage](loop-command-usage.md)
- [Triple Loop Architecture](../architecture/triple-loop-architecture.md)
- [Best Practices for Autonomous Sessions](../best-practices/autonomous-session-best-practices.md)
