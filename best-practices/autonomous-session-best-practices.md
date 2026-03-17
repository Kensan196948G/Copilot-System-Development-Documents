# Best Practices for Autonomous Development Sessions

## Overview

Running long autonomous development sessions requires discipline in how tasks are defined, how the system is configured, and how humans interact with the output. This guide distills the most important practices for getting reliable, high-quality results from the Copilot Triple Loop system.

---

## 1. Task Definition Quality

The single most important factor in autonomous development quality is **how well tasks are written**. The AI agent cannot infer intent, resolve ambiguous requirements, or make architectural decisions without clear guidance.

### ✅ Do

- **Be specific about scope**: Name the exact files, functions, or modules to be changed.
- **Include acceptance criteria**: Use a checklist that can be mechanically verified.
- **State constraints explicitly**: "Must not change the public API", "Must maintain backwards compatibility with Node 18".
- **Reference related code**: Link to existing functions that the new code should follow as a pattern.
- **Specify the test requirement**: "Include unit tests for all new public functions".

### ❌ Don't

- Write vague tasks: "Improve the auth module" → the agent will guess at scope.
- Conflate multiple independent changes in one task → decompose instead.
- Rely on implicit knowledge: "Fix it the way we always do it" → the agent has no memory of past conventions unless they are in the codebase.
- Set unrealistic scope: a single task should be completable in ≤ 5 file changes.

### Example: Good vs. Bad Task Descriptions

**Bad:**
> Fix the authentication issue.

**Good:**
> When a valid user presents an expired JWT token to any authenticated route, the API should return HTTP 401 with body `{"error": "token_expired", "refresh_url": "/auth/refresh"}` instead of the current generic 500 response. Modify `src/auth/middleware.ts` and add a test to `src/auth/__tests__/middleware.test.ts`. Do not change any other error response formats.

---

## 2. Context Management

Autonomous agents are limited by their context window. The Monitor Loop handles context selection, but you can improve it by:

- **Keeping files focused**: large files with many concerns make context selection harder. Prefer smaller, single-responsibility modules.
- **Writing good inline documentation**: the Monitor Loop's context-gathering prompts use docstrings and comments to determine relevance.
- **Maintaining `ARCHITECTURE.md` or similar docs**: if a high-level design document exists, the Monitor Loop will include it in context for architectural tasks.
- **Linking issues**: reference related issues in GitHub Issue bodies so the Monitor Loop can fetch their context.

---

## 3. Configuration Tuning

### Start Conservative

When starting with a new codebase, begin with conservative settings:

```yaml
build:
  max_retries: 3        # lower until you understand failure patterns
  timeout_minutes: 10

verify:
  coverage_thresholds:
    lines: 70           # lower threshold until baseline is measured
```

### Increase Gradually

As the system proves reliable on your codebase, increase quality thresholds and task complexity.

### Recommended Settings by Project Maturity

| Maturity     | `max_retries` | Coverage Threshold | Tasks per Session |
|--------------|---------------|--------------------|--------------------|
| New codebase | 3             | 60%                | 2–3               |
| Established  | 5             | 80%                | 5–10              |
| Mature       | 5             | 90%                | 10–20             |

---

## 4. Human-in-the-Loop Checkpoints

Fully autonomous operation is the goal, but certain decisions must involve humans.

### Always Require Human Review For:

- **Security-related changes**: authentication, authorization, cryptography, input validation
- **Database schema changes**: migrations are difficult to roll back
- **API contract changes**: adding/removing/renaming public endpoints
- **Dependency upgrades**: especially major version upgrades
- **Performance-critical code**: code on hot paths or with O(n²) complexity risk
- **Deletion of files or large code blocks**: irreversible and easy to do incorrectly

### Use Automatic Merge Only For:

- Documentation updates
- Dependency patch-level upgrades with no test regressions
- Test additions that do not modify existing tests
- Formatting and lint auto-fixes

---

## 5. Overnight Session Strategy

The most productive use of autonomous development is running sessions overnight, so engineers return to reviewed PRs each morning.

### Pre-Session Checklist (End of Day)

- [ ] Review and label any new GitHub Issues with `autonomous`
- [ ] Ensure all acceptance criteria are clearly written
- [ ] Confirm the test suite is passing on the main branch (clean baseline)
- [ ] Run `npx copilot-loops start --dry-run` to verify configuration
- [ ] Set escalation recipients in `loop-config.yaml`
- [ ] Start the session: `npx copilot-loops start --daemon`

### Post-Session Checklist (Start of Day)

- [ ] Run `npx copilot-loops status` to see what was completed
- [ ] Review all auto-generated PRs: `gh pr list --label autonomous`
- [ ] Review any escalated issues: `gh issue list --label needs-human`
- [ ] Review session logs for unexpected patterns: `npx copilot-loops logs --since 12h`
- [ ] Merge or request changes on ready PRs
- [ ] Reset any failed tasks as needed
- [ ] Optionally start a new session for the next batch

---

## 6. Monitoring System Health

### Watch for These Warning Signs

| Signal                                   | Likely Cause                                   | Action                                    |
|------------------------------------------|------------------------------------------------|-------------------------------------------|
| Same task retries > 3 times              | Ambiguous requirements or missing context      | Rewrite the issue with more specifics     |
| Build Loop always fails on first attempt | Context packages missing critical files        | Review Monitor Loop context selection     |
| Coverage always below threshold          | Test style not compatible with coverage tool   | Adjust coverage tool configuration        |
| All tasks escalate to human              | Quality thresholds too strict for codebase     | Lower thresholds; build baseline coverage |
| PRs contain many unrelated file changes  | Context files not scoped correctly             | Add `context_files` hints to issue body   |
| Copilot CLI rate limit errors            | Too many parallel tasks                        | Reduce parallelism in config              |

### Log Review Commands

```bash
# Check for error patterns
npx copilot-loops logs --since 24h | grep ERROR

# Count build failures by task
npx copilot-loops logs --since 24h | grep "Build failed" | cut -d' ' -f4 | sort | uniq -c

# Review escalations
npx copilot-loops logs --since 24h | grep "ESCALATION"
```

---

## 7. Security Practices

### Protect Sensitive Branches

Configure branch protection rules so that the autonomous system can only merge to non-protected branches:

```yaml
build:
  branch_prefix: "autonomous/"     # all autonomous branches namespaced
  # NEVER allow direct push to main or release branches
```

### Review Copilot Output for Security Issues

Before merging any autonomously generated PR, verify:

- No hardcoded credentials, tokens, or secrets
- No new direct SQL string construction (use parameterized queries)
- No user input passed directly to shell commands
- No new `eval()` calls or equivalent

### Audit Logging

The system logs all Copilot prompts and responses. Retain these logs for at least 30 days:

```yaml
logging:
  audit_log_path: ~/.copilot-loops/audit/
  retention_days: 30
  include_full_prompts: true
  include_full_responses: true
```

---

## 8. Iterative Improvement

Treat the autonomous system itself as a product you are continuously improving.

### Weekly Review

- Which tasks consistently fail? → Improve task template for that type of work
- Which prompts produce low-quality output? → Refine prompt templates
- What types of bugs appear in auto-generated PRs? → Add those checks to the Verify Loop
- What is the average task completion rate? → Track as a KPI

### Feedback Loop

When you reject a PR or request changes, capture that feedback:

```bash
# After rejecting a PR with reason
npx copilot-loops task feedback TASK-42 \
  --outcome rejected \
  --reason "Generated function does not handle the case where userId is null"
```

This feedback is stored in the State Manager and used to improve future context packages for similar tasks.

---

## Related Documents

- [Autonomous Development Workflow](../operations/autonomous-development-workflow.md)
- [Triple Loop Architecture](../architecture/triple-loop-architecture.md)
- [Monitor Loop Prompts](../prompts/monitor-loop-prompts.md)
- [Verify Loop Prompts](../prompts/verify-loop-prompts.md)
