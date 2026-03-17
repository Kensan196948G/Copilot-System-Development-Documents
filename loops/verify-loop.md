# Verify Loop

## Overview

The Verify Loop is the quality assurance engine of the Triple Loop Architecture. After the Build Loop produces a successful build and commits code changes, the Verify Loop takes over: it runs automated tests, static analysis, security scans, and coverage checks to determine whether the changes are safe to merge.

The Verify Loop treats quality gates as **pass/fail contracts**. If a gate fails, it produces a structured report that the Monitor Loop uses to re-plan and re-queue the task for another Build Loop iteration.

---

## Responsibilities

| Responsibility           | Description                                                      |
|--------------------------|------------------------------------------------------------------|
| Test execution           | Run unit, integration, and end-to-end test suites                |
| Coverage measurement     | Check that new code meets minimum coverage thresholds            |
| Static analysis          | Run linters and type checkers                                    |
| Security scanning        | Detect vulnerabilities, secrets, and insecure patterns           |
| Regression detection     | Compare results against previous baseline to detect regressions  |
| Quality report production| Produce a structured report for Monitor Loop consumption         |
| Merge readiness          | Signal whether the commit is ready to open a Pull Request        |

---

## Verify Loop Execution Flow

```
┌──────────────────────────────────────────────────────────────────┐
│                      Verify Loop Cycle                           │
│                                                                  │
│  1. Receive commit SHA and task ID from Build Loop               │
│  2. Checkout commit in isolated verification environment         │
│  3. Install dependencies                                         │
│  4. Run test suite → collect results                             │
│  5. Run coverage tool → check threshold                          │
│  6. Run linter → check for new violations                        │
│  7. Run security scanner → check for new vulnerabilities         │
│  8. Compare against baseline (previous passing commit)           │
│  9. Produce quality report                                       │
│ 10. Evaluate all gates:                                          │
│     ├── all pass → notify Monitor Loop: "merge ready"            │
│     └── any fail → notify Monitor Loop: "rework required"        │
│         (attach structured failure details)                      │
└──────────────────────────────────────────────────────────────────┘
```

---

## Quality Gates

Each gate is evaluated independently. All gates must pass for the commit to be considered merge-ready.

### Gate 1: Test Suite

| Check              | Pass Condition                         |
|--------------------|----------------------------------------|
| All tests pass     | Zero failing tests                     |
| No new skipped     | No increase in skipped test count      |
| Flaky test timeout | Tests complete within timeout limit    |

### Gate 2: Code Coverage

| Check              | Pass Condition                         |
|--------------------|----------------------------------------|
| Line coverage      | ≥ configured threshold (default: 80%) |
| Branch coverage    | ≥ configured threshold (default: 75%) |
| New code coverage  | New code has ≥ 90% line coverage       |

### Gate 3: Static Analysis

| Check              | Pass Condition                          |
|--------------------|-----------------------------------------|
| Lint errors        | Zero new lint errors introduced         |
| Type errors        | Zero TypeScript / mypy / Go vet errors  |
| Code complexity    | No new functions exceeding cyclomatic complexity threshold |

### Gate 4: Security

| Check                     | Pass Condition                         |
|---------------------------|----------------------------------------|
| Dependency vulnerabilities| No new HIGH or CRITICAL CVEs           |
| Secret detection          | No hardcoded secrets or API keys       |
| SAST findings             | No new HIGH or CRITICAL SAST findings  |

---

## Quality Report Format

The Verify Loop produces a structured quality report:

```json
{
  "task_id": "TASK-1042",
  "commit_sha": "f3a2b1c",
  "timestamp": "2024-01-15T11:45:00Z",
  "overall_status": "fail",
  "gates": {
    "tests": {
      "status": "pass",
      "passed": 142,
      "failed": 0,
      "skipped": 3,
      "duration_seconds": 47
    },
    "coverage": {
      "status": "fail",
      "line_coverage": 76.2,
      "threshold": 80.0,
      "missing_coverage": [
        "src/auth/token.ts:lines 102-118 (error handling branch)"
      ]
    },
    "lint": {
      "status": "pass",
      "new_violations": 0
    },
    "security": {
      "status": "pass",
      "new_vulnerabilities": 0
    }
  },
  "merge_ready": false,
  "rework_notes": [
    "Coverage gate failed: line coverage 76.2% is below threshold 80%.",
    "Add tests for error handling branches in src/auth/token.ts lines 102-118."
  ]
}
```

---

## Regression Detection

The Verify Loop maintains a baseline from the last passing commit. On each run it computes deltas:

- **New failures**: tests that previously passed but now fail → critical regression
- **Newly skipped**: tests that were previously running but are now skipped → investigate
- **Coverage drop**: coverage decreased since last passing commit → fail gate
- **New lint violations**: violations introduced by the current commit → fail gate

Regressions are given highest priority in the `rework_notes` returned to the Monitor Loop.

---

## Verification Environment

Like the Build Loop, verification runs in an isolated container:

```yaml
verify:
  docker_image: "node:20-alpine"
  working_dir: /workspace
  env_vars:
    - NODE_ENV=test
    - CI=true
  timeout_seconds: 600
  tools:
    test_runner: "jest"              # or: pytest, go test, mvn test
    coverage_tool: "jest --coverage" # or: coverage.py, go cover
    lint_tool: "eslint"              # or: pylint, golangci-lint
    security_tool: "npm audit"       # or: safety, gosec
```

---

## Verify Loop Outputs and Actions

| Outcome          | Verify Loop Action                                               |
|------------------|------------------------------------------------------------------|
| All gates pass   | Mark task `merge_ready`; create or update Pull Request           |
| Test failures    | Feed failing test names + output into Monitor Loop context       |
| Coverage failure | Identify uncovered lines; add coverage task to Monitor queue     |
| Lint failure     | Include specific lint errors in rework context                   |
| Security finding | Pause all loops; escalate to human reviewer immediately          |

---

## Configuration

```yaml
verify:
  test_command: "npm test"
  coverage_command: "npm run test:coverage"
  lint_command: "npm run lint"
  security_command: "npm audit --audit-level=high"
  coverage_thresholds:
    lines: 80
    branches: 75
    new_code_lines: 90
  max_test_duration_seconds: 300
  fail_on_new_lint_violations: true
  fail_on_high_cve: true
  create_pull_request_on_pass: true
  pull_request_reviewers:
    - "@team/platform-engineers"
```

---

## Related Documents

- [Triple Loop Architecture](../architecture/triple-loop-architecture.md)
- [Monitor Loop](monitor-loop.md)
- [Build Loop](build-loop.md)
- [Verify Loop Prompts](../prompts/verify-loop-prompts.md)
