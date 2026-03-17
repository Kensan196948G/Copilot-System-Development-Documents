# Example: End-to-End Autonomous Development Workflow

## Overview

This example walks through a complete autonomous development session spanning an overnight run. It covers five tasks processed by the Triple Loop system, including one that required multiple retries, one that was escalated to human review, and three that completed successfully on the first attempt.

---

## Session Configuration

```yaml
# loop-config.yaml used for this session
monitor:
  interval_seconds: 120
  max_context_files: 20
  task_queue:
    source: github_issues
    label_filter: "autonomous"
  escalation_retry_limit: 3

build:
  max_retries: 5
  timeout_minutes: 15
  build_isolation: docker

verify:
  test_command: "npm test"
  coverage_command: "npm run test:coverage"
  lint_command: "npm run lint"
  coverage_thresholds:
    lines: 80
    branches: 75
  create_pull_request_on_pass: true
```

**Session start:** 2024-01-15 18:00:00 UTC (after working hours)  
**Session end:** 2024-01-16 06:30:00 UTC (next morning)  
**Tasks in queue:** 5

---

## Task Queue at Session Start

| Task     | Title                                        | Priority | Estimate |
|----------|----------------------------------------------|----------|----------|
| TASK-42  | Add JWT expiry refresh logic                 | High     | Small    |
| TASK-43  | Add pagination to /users endpoint            | High     | Small    |
| TASK-44  | Migrate user table to new schema             | Medium   | Large    |
| TASK-45  | Add rate limiting to /auth/login             | Medium   | Small    |
| TASK-46  | Update API documentation for v2 endpoints    | Low      | Medium   |

---

## Task 1: TASK-42 — JWT Refresh Logic ✅

**Start:** 18:00:05  
**Complete:** 18:02:15  
**Attempts:** 1  
**Outcome:** PR #47 opened

All quality gates passed on the first attempt. See [Example: Monitor Loop Session](example-monitor-session.md) for a detailed walkthrough of this task.

---

## Task 2: TASK-43 — Pagination for /users Endpoint ✅

**Start:** 18:02:20  
**Complete:** 18:09:45  
**Attempts:** 2  
**Outcome:** PR #48 opened

### Attempt 1 (18:02:20) — Build Failure

The Build Loop generated pagination logic but introduced a TypeScript type error:

```
Error: Property 'total' does not exist on type 'UserQueryResult'
  at src/users/userService.ts:89:32
```

**Monitor Loop response:** Enriched the context package with the `UserQueryResult` type definition from `src/types/users.d.ts` (missed in initial context gathering).

### Attempt 2 (18:04:10) — Success

With the type definition included, Copilot generated the correct implementation. Build and all verify gates passed.

**Changes:**
- `src/users/userService.ts`: Added `findWithPagination()` using cursor-based pagination
- `src/users/userController.ts`: Added `page`, `pageSize`, `cursor` query params
- `src/types/users.d.ts`: Added `PaginatedResult<T>` generic type
- `src/users/__tests__/userService.test.ts`: 6 new test cases

---

## Task 3: TASK-44 — Database Schema Migration ⚠️ (Escalated)

**Start:** 18:10:00  
**Escalated:** 18:52:30  
**Attempts:** 3  
**Outcome:** Issue labeled `needs-human`; assigned to @db-team

### Attempt 1 (18:10:00) — Verify Failure

The Build Loop generated the migration but tests revealed data integrity issues:

```
FAIL src/users/__tests__/migration.test.ts
  ✕ migrated users retain their role assignments (245ms)
    Expected: [{ userId: "u1", roles: ["admin", "viewer"] }]
    Received: [{ userId: "u1", roles: [] }]
```

**Rework context:** The Monitor Loop added the test failure details and the current migration file.

### Attempt 2 (18:22:00) — Verify Failure (Different Error)

The fix for role assignments introduced a foreign key constraint violation in PostgreSQL:

```
ERROR: insert or update on table "user_roles" violates foreign key constraint
"user_roles_role_id_fkey" DETAIL: Key (role_id)=(3) is not present in table "roles"
```

**Rework context:** Monitor Loop added the database schema ERD reference file.

### Attempt 3 (18:35:00) — Verify Failure (Third Time)

A different approach to the migration resolved the FK constraint but introduced a regression in the role permission caching layer.

### Escalation Decision

After 3 consecutive verify failures, the Monitor Loop triggered the escalation policy:

```
[2024-01-15 18:52:30] [Monitor] TASK-44 has failed 3 consecutive verify cycles
[2024-01-15 18:52:31] [Monitor] Escalating to human review
[2024-01-15 18:52:32] [Monitor] GitHub Issue #44 labeled: needs-human
[2024-01-15 18:52:33] [Monitor] Comment posted on Issue #44:

  ⚠️ Autonomous agent escalation after 3 failed attempts.
  
  Root cause assessment: The migration involves complex inter-table 
  dependencies between user_roles, roles, and the permission cache. 
  Each fix resolves one layer but breaks another.
  
  Recommended human action:
  1. Review the 3 migration attempts in branch autonomous/TASK-44
  2. Consult the DB team for the correct migration order
  3. Consider whether the cache invalidation logic needs to be updated 
     separately as TASK-44b
  
  Relevant files: 
  - migrations/20240115_user_schema_v2.sql
  - src/permissions/cache.ts
  - src/users/__tests__/migration.test.ts
```

---

## Task 4: TASK-45 — Rate Limiting for /auth/login ✅

**Start:** 18:52:40  
**Complete:** 19:15:20  
**Attempts:** 2  
**Outcome:** PR #49 opened

### Attempt 1 — Build Failure

Copilot attempted to use `express-rate-limit@7` but the project was pinned to `express-rate-limit@6` in package.json. The import syntax changed between versions.

**Monitor Loop response:** Added `package.json` to context; added constraint "Use only packages already listed in package.json".

### Attempt 2 — Success

Rate limiting implemented using the existing v6 API. All tests passed.

**Changes:**
- `src/auth/loginController.ts`: Added rate limiting middleware (15 requests per 15 minutes per IP)
- `src/middleware/rateLimiter.ts`: New file with reusable rate limiter factory
- `src/auth/__tests__/loginController.test.ts`: 3 new tests for rate limit behavior

---

## Task 5: TASK-46 — API Documentation for v2 Endpoints ✅

**Start:** 19:16:00  
**Complete:** 19:45:00  
**Attempts:** 1  
**Outcome:** PR #50 opened

Documentation-only task. Copilot generated updated OpenAPI 3.0 specs and README sections for all v2 endpoints on the first attempt.

**Changes:**
- `docs/api/openapi.yaml`: Updated 12 endpoint definitions
- `docs/README.md`: Added v2 endpoint reference section
- `docs/migration-guide.md`: New file explaining v1 → v2 differences

---

## Session Summary

| Task    | Status        | Attempts | Duration  | PR     |
|---------|---------------|----------|-----------|--------|
| TASK-42 | ✅ Merged      | 1        | 2 min     | #47    |
| TASK-43 | ✅ Ready       | 2        | 8 min     | #48    |
| TASK-44 | ⚠️ Escalated  | 3        | 43 min    | -      |
| TASK-45 | ✅ Ready       | 2        | 23 min    | #49    |
| TASK-46 | ✅ Ready       | 1        | 29 min    | #50    |

**Session duration:** 12.5 hours (18:00 → 06:30)  
**Tasks completed autonomously:** 4/5 (80%)  
**PRs opened:** 4  
**Human time required (next morning):** ~45 minutes to review and merge 4 PRs

---

## What the Engineer Did the Next Morning

```bash
# Review session summary
npx copilot-loops status

# Review PRs
gh pr list --label autonomous

# Review escalated issues
gh issue list --label needs-human

# Merge approved PRs
gh pr merge 47 --squash
gh pr merge 48 --squash
# Review TASK-44 escalation → assign to DB team
gh issue edit 44 --add-assignee "@db-team"
gh pr merge 49 --squash
gh pr merge 50 --squash
```

Total human time: **~40 minutes** to review 4 PRs and handle 1 escalation, compared to approximately **6–8 hours** of focused development work had it been done manually.

---

## Lessons Learned from This Session

1. **Type definitions matter** – TASK-43 required a second attempt because `users.d.ts` was not in the initial context. Adding type definition files to the Monitor Loop's context gathering heuristics would prevent this.

2. **Complex migrations need pre-planning** – TASK-44 is an example of a task that was too large and interdependent for fully autonomous handling. Breaking it into subtasks (schema change, data migration, cache invalidation) would have enabled autonomous completion.

3. **Package version pinning** – TASK-45 failed on the first attempt due to a version mismatch. A dedicated `dependency-agent` that pre-validates package versions before code generation would eliminate this class of failure.

---

## Related Documents

- [Example: Monitor Loop Session](example-monitor-session.md)
- [Autonomous Development Workflow](../operations/autonomous-development-workflow.md)
- [Best Practices for Autonomous Sessions](../best-practices/autonomous-session-best-practices.md)
