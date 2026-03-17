# Example: Monitor Loop Session

## Overview

This example walks through a complete Monitor Loop session, showing actual prompt inputs, Copilot CLI responses, and system decisions. The scenario is a real-world task: implementing JWT token refresh logic in a TypeScript/Node.js API.

---

## Scenario

**Repository:** `my-org/api-service` (TypeScript, Node.js 20, Jest)  
**Task:** TASK-42 "Add JWT expiry refresh logic to auth middleware"  
**Session start:** 2024-01-15 06:00:00 UTC

---

## Step 1: Monitor Loop Cycle Start

```
[2024-01-15 06:00:00] [Monitor] Cycle 1 starting
[2024-01-15 06:00:00] [Monitor] Polling GitHub Issues for label 'autonomous'
[2024-01-15 06:00:01] [Monitor] Found 2 tasks:
  TASK-42: "Add JWT expiry refresh logic" (priority: high, milestone: v2.3)
  TASK-43: "Add pagination to /users endpoint" (priority: medium, milestone: v2.3)
[2024-01-15 06:00:01] [Monitor] Selected TASK-42 (highest priority)
```

---

## Step 2: Context Gathering

The Monitor Loop reads the GitHub Issue and constructs a context analysis prompt.

### Issue Content

```markdown
## Task Description
When a valid JWT token is within 5 minutes of its expiry time, the auth
middleware should automatically issue a refresh token alongside the response.
The original request should still be processed normally.

## Acceptance Criteria
- [ ] Tokens within 5 minutes of expiry trigger a refresh
- [ ] Refresh window is configurable via `JWT_REFRESH_WINDOW_MINUTES` env var
- [ ] Refresh token is added to response header `X-Refresh-Token`
- [ ] Expired tokens (past expiry) still return 401 (no change)
- [ ] All existing auth tests continue to pass
- [ ] New tests cover the refresh window logic

## Constraints
- Must work with both RS256 and HS256 algorithms
- Must not change the public token validation API
- Refresh token must use the same signing key as the original token

## Context Files
- src/auth/token.ts
- src/auth/middleware.ts
```

### Monitor Loop Prompt (Context Analysis)

```
You are an expert software engineer performing codebase analysis...

## Task
Title: Add JWT expiry refresh logic to auth middleware
Description: [issue body above]

## Repository Structure
src/
  auth/
    token.ts
    middleware.ts
    __tests__/
      token.test.ts
      middleware.test.ts
  lib/
    jwt.ts
    logger.ts
  types/
    auth.d.ts

## Recent Commits (last 5)
a1b2c3d Fix token parsing edge case in RS256 mode
b2c3d4e Add rate limiting to /auth/login
c3d4e5f Update jest to 29.7.0
...

[full prompt as per prompts/monitor-loop-prompts.md Prompt 1]
```

### Copilot CLI Response

```json
{
  "relevant_files": [
    { "path": "src/auth/token.ts", "reason": "Core token generation and validation logic to be extended" },
    { "path": "src/auth/middleware.ts", "reason": "Auth middleware pipeline where refresh logic should be integrated" },
    { "path": "src/lib/jwt.ts", "reason": "JWT signing wrapper; refresh token should use this, not jsonwebtoken directly" },
    { "path": "src/types/auth.d.ts", "reason": "TypeScript type definitions for auth objects; may need new RefreshOptions type" }
  ],
  "test_files": [
    { "path": "src/auth/__tests__/token.test.ts", "reason": "Unit tests for token module" },
    { "path": "src/auth/__tests__/middleware.test.ts", "reason": "Integration tests for auth middleware" }
  ],
  "documentation_files": [],
  "risks": [
    "Adding logic to the middleware request pipeline may affect response latency for all authenticated routes",
    "RS256 and HS256 have different signing APIs; the implementation must handle both"
  ],
  "suggested_approach": "Add a refreshIfExpiring(token, windowMs) function to token.ts, then call it in middleware.ts after successful token verification. If it returns a refresh token, set the X-Refresh-Token response header."
}
```

---

## Step 3: Context Package Construction

```
[2024-01-15 06:00:08] [Monitor] Context analysis complete
[2024-01-15 06:00:08] [Monitor] Reading 4 relevant files (3 test files)...
[2024-01-15 06:00:09] [Monitor] Context package assembled:
  - Files: 7
  - Tokens: 4,200
  - Constraints: 5
  - Risks flagged: 2
[2024-01-15 06:00:09] [Monitor] Dispatching to Build Loop...
```

The context package JSON is written to the state store:

```json
{
  "task_id": "TASK-42",
  "title": "Add JWT expiry refresh logic to auth middleware",
  "priority": "high",
  "context_files": [
    "src/auth/token.ts",
    "src/auth/middleware.ts",
    "src/lib/jwt.ts",
    "src/types/auth.d.ts",
    "src/auth/__tests__/token.test.ts",
    "src/auth/__tests__/middleware.test.ts"
  ],
  "suggested_approach": "Add a refreshIfExpiring(token, windowMs) function...",
  "constraints": [
    "Must work with both RS256 and HS256",
    "Must not change the public token validation API",
    "Refresh token uses same signing key as original token",
    "Refresh window configurable via JWT_REFRESH_WINDOW_MINUTES env var",
    "Add X-Refresh-Token response header"
  ],
  "risks": [
    "RS256 and HS256 have different signing APIs",
    "Pipeline change may affect latency"
  ],
  "attempt_number": 1,
  "previous_errors": []
}
```

---

## Step 4: Build Loop Response (Success)

```
[2024-01-15 06:00:10] [Build] Received context package for TASK-42 (attempt 1/5)
[2024-01-15 06:00:10] [Build] Constructing Copilot prompt (4,800 tokens)
[2024-01-15 06:00:25] [Build] Copilot response received (680 tokens)
[2024-01-15 06:00:25] [Build] Applying changes:
  - MODIFIED: src/auth/token.ts (+42 lines)
  - MODIFIED: src/auth/middleware.ts (+8 lines)
  - MODIFIED: src/types/auth.d.ts (+12 lines)
  - MODIFIED: src/auth/__tests__/token.test.ts (+35 lines)
[2024-01-15 06:00:26] [Build] Running: npm run build
[2024-01-15 06:00:38] [Build] Build succeeded (12.3s)
[2024-01-15 06:00:39] [Build] Committed: autonomous/TASK-42 (commit: f3a2b1c)
[2024-01-15 06:00:40] [Build] Notifying Verify Loop
```

---

## Step 5: Verify Loop Result

```
[2024-01-15 06:00:41] [Verify] Received artifact: TASK-42 commit f3a2b1c
[2024-01-15 06:00:42] [Verify] Checking out commit in Docker container
[2024-01-15 06:00:48] [Verify] Running: npm test
[2024-01-15 06:01:35] [Verify] Tests: 148 passed, 0 failed (47s)
[2024-01-15 06:01:36] [Verify] Running coverage check
[2024-01-15 06:01:40] [Verify] Coverage: 83.4% (threshold: 80%) ✓
[2024-01-15 06:01:41] [Verify] Running: npm run lint
[2024-01-15 06:01:44] [Verify] Lint: 0 new violations ✓
[2024-01-15 06:01:45] [Verify] Running: npm audit --audit-level=high
[2024-01-15 06:01:48] [Verify] Security: 0 new CVEs ✓
[2024-01-15 06:01:48] [Verify] All gates PASSED
[2024-01-15 06:01:49] [Verify] Creating Pull Request...
[2024-01-15 06:01:52] [Verify] PR #47 opened: "autonomous: Add JWT expiry refresh logic"
[2024-01-15 06:01:52] [Verify] Task TASK-42 marked: merge_ready
```

---

## Step 6: Monitor Loop Picks Up Next Task

```
[2024-01-15 06:02:00] [Monitor] Cycle 2 starting
[2024-01-15 06:02:00] [Monitor] TASK-42 is complete (merge_ready)
[2024-01-15 06:02:01] [Monitor] Selecting next task: TASK-43 "Add pagination to /users endpoint"
[2024-01-15 06:02:02] [Monitor] Gathering context...
```

---

## Session Summary

After 62 minutes of autonomous operation, the system:

- Completed TASK-42 on the first attempt (no retries needed)
- All quality gates passed
- Generated PR #47 with 4 files changed, 97 lines added
- Moved to TASK-43 without human intervention

**Total elapsed time:** 2 minutes (from task dispatch to PR creation)  
**Human time required:** ~10 minutes to review and merge PR #47

---

## Related Documents

- [Monitor Loop](../loops/monitor-loop.md)
- [Build Loop](../loops/build-loop.md)
- [Verify Loop](../loops/verify-loop.md)
- [Example: End-to-End Workflow](example-end-to-end-workflow.md)
