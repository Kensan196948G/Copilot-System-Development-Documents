# Verify Loop Prompts

## Overview

This document provides the canonical prompt templates used by the Verify Loop when analyzing quality gate failures and generating rework instructions for the Build Loop. These prompts turn raw test output, lint errors, and security findings into actionable repair guidance.

---

## Prompt Engineering Principles for the Verify Loop

The Verify Loop prompts follow these principles:

1. **Evidence-based reasoning** – always include actual test output, not just summaries
2. **Specific line references** – point to exact file and line numbers
3. **Actionable output** – every prompt should produce instructions the Build Loop can act on directly
4. **Root cause focus** – ask for root cause, not just symptom description
5. **Prioritized feedback** – rank issues by severity so the Build Loop fixes the most critical issues first

---

## Prompt 1: Test Failure Analysis

Used when one or more tests fail. The goal is to analyze the failing tests and produce a repair plan.

```
You are an expert software engineer analyzing failing unit tests. Your goal is to
diagnose the root cause of each failure and provide actionable fix instructions.

## Task Context
Task: {task_title} ({task_id})
Commit: {commit_sha}

## Failing Tests

{failing_tests_block}
<!-- Format:
TEST: src/auth/__tests__/token.test.ts > "refreshIfExpiring returns null when token is not expiring"
STATUS: FAIL
ERROR:
  Expected: null
  Received: { refreshToken: "eyJ...", expiresIn: 3600 }

  at Object.<anonymous> (src/auth/__tests__/token.test.ts:67:32)
-->

## Modified Source Files
```{language}
// src/auth/token.ts
{modified_file_content}
```

## Instructions
For each failing test:
1. Identify the root cause of the failure
2. Identify the specific code change that caused the failure (file and line number)
3. Provide a corrected implementation for the affected code

Respond in JSON format:
{
  "failures": [
    {
      "test": "refreshIfExpiring returns null when token is not expiring",
      "root_cause": "The function always returns a refresh token instead of checking the expiry window",
      "affected_file": "src/auth/token.ts",
      "affected_lines": "23-31",
      "fix": {
        "description": "Add a check: only call refresh() if token.exp - Date.now() < refreshWindowMs",
        "code": "if (tokenPayload.exp * 1000 - Date.now() >= refreshWindowMs) {\n  return null;\n}"
      }
    }
  ],
  "summary": "The expiry check logic is inverted. The condition should return null when NOT expiring.",
  "priority": "high"
}
```

---

## Prompt 2: Coverage Gap Analysis

Used when the code coverage gate fails. Identifies which code paths are untested and suggests test cases.

```
You are an expert software engineer analyzing a code coverage report. The
following code changes did not meet the required coverage threshold.

## Coverage Report
Threshold: {coverage_threshold}%
Actual: {actual_coverage}%
Status: FAIL

## Uncovered Lines
{uncovered_lines_block}
<!-- Format:
File: src/auth/token.ts
  Lines 45-52: Error handling branch when JWT secret is missing
  Lines 78-83: Fallback path when refresh token store is unavailable
-->

## Source Code
```{language}
// src/auth/token.ts (full content)
{source_file_content}
```

## Existing Tests
```{language}
// src/auth/__tests__/token.test.ts (full content)
{test_file_content}
```

## Instructions
For each uncovered code path:
1. Explain what scenario that code path handles
2. Write a specific test case that exercises that path
3. Include both the test input and expected output

Respond in JSON format:
{
  "coverage_gaps": [
    {
      "file": "src/auth/token.ts",
      "lines": "45-52",
      "scenario": "JWT secret environment variable is not set",
      "test_case": {
        "description": "throws ConfigurationError when JWT_SECRET is not set",
        "setup": "delete process.env.JWT_SECRET",
        "assertion": "expect(() => createToken(payload)).toThrow(ConfigurationError)",
        "teardown": "process.env.JWT_SECRET = originalSecret"
      }
    }
  ],
  "estimated_coverage_gain": "8.5%",
  "will_meet_threshold": true
}
```

---

## Prompt 3: Lint Error Analysis

Used when the linter reports new violations introduced by the Build Loop's changes.

```
You are an expert software engineer fixing lint violations. The following new
lint errors were introduced by recent code changes.

## Task Context
Task: {task_title} ({task_id})
Linter: {linter_name}

## New Lint Violations
{lint_violations_block}
<!-- Format:
src/auth/token.ts:47:5  error  Unexpected console statement  no-console
src/auth/token.ts:89:1  error  Function 'refreshToken' has complexity of 15 (max 10)  complexity
src/auth/token.ts:103:7 warning Missing return type annotation  @typescript-eslint/explicit-function-return-type
-->

## Affected Code
```{language}
// src/auth/token.ts (relevant sections)
{code_sections_with_violations}
```

## Instructions
For each lint violation:
1. Explain why this rule exists (the engineering rationale)
2. Provide a corrected version of the code that satisfies the rule
3. Note if fixing this violation requires changes to multiple locations

Respond in JSON format:
{
  "fixes": [
    {
      "file": "src/auth/token.ts",
      "line": 47,
      "rule": "no-console",
      "explanation": "Use the logger utility instead of console.log for structured logging",
      "original": "console.log(`Token refreshed for user ${userId}`);",
      "corrected": "logger.info('Token refreshed', { userId });",
      "additional_change": "Add: import { logger } from '../lib/logger';"
    }
  ],
  "refactoring_needed": true,
  "refactoring_note": "The refreshToken function at line 89 exceeds complexity limit. Extract inner conditional blocks into named helper functions."
}
```

---

## Prompt 4: Security Finding Analysis

Used when the security scanner detects new vulnerabilities. This is a critical prompt — security findings always require careful, accurate analysis.

```
You are an expert application security engineer analyzing new security findings
in a code change. Your goal is to identify the risk, assess its severity, and
provide a secure fix.

## Task Context
Task: {task_title} ({task_id})
Scanner: {security_tool_name}

## Security Findings
{security_findings_block}
<!-- Format:
FINDING: Potential hardcoded secret
Severity: HIGH
File: src/auth/token.ts
Line: 12
Code: const defaultSecret = "dev-secret-key-12345";
Rule: detect-hardcoded-credentials
-->

## Affected Code
```{language}
{affected_code_with_context}
```

## Instructions
For each finding:
1. Confirm whether this is a true positive or false positive
2. If true positive: explain the specific risk and provide a secure remediation
3. If false positive: explain why and suggest a suppression comment

IMPORTANT: Never suggest workarounds that reduce security. Prefer proper fixes.

Respond in JSON format:
{
  "findings": [
    {
      "file": "src/auth/token.ts",
      "line": 12,
      "true_positive": true,
      "risk": "Hardcoded default secret could be used in production if JWT_SECRET env var is not set",
      "cvss_estimate": "HIGH",
      "remediation": {
        "description": "Remove the default value; throw a clear ConfigurationError if JWT_SECRET is not set",
        "code": "const secret = process.env.JWT_SECRET;\nif (!secret) {\n  throw new ConfigurationError('JWT_SECRET environment variable is required');\n}"
      }
    }
  ],
  "overall_risk": "HIGH",
  "requires_immediate_human_review": true
}
```

---

## Prompt 5: Overall Quality Report Summary

Used to generate a human-readable summary of the Verify Loop's quality report for the PR description.

```
You are a senior software engineer writing a pull request description for
code changes generated by an autonomous AI agent.

## Task
Title: {task_title}
Description: {task_description}

## Changes Made
{changed_files_summary}

## Quality Gate Results
Tests: {tests_passed} passed, {tests_failed} failed, {tests_skipped} skipped
Coverage: {coverage_actual}% (threshold: {coverage_threshold}%)
Lint: {lint_violations} new violations
Security: {security_findings} new findings

## Instructions
Write a clear, professional pull request description that:
1. Summarizes what was changed and why (2–3 sentences)
2. Lists the specific files changed with one-line explanations
3. Notes any design decisions or trade-offs made
4. Summarizes the quality gate results
5. Flags any areas that need particular reviewer attention

Format the output as markdown suitable for a GitHub PR description.
Do NOT include any headers for the title — the title is set separately.
```

---

## Prompt Variables Reference

| Variable                    | Source                           | Description                                     |
|-----------------------------|----------------------------------|-------------------------------------------------|
| `{task_id}`                 | State Manager                    | Task identifier (e.g., TASK-42)                 |
| `{task_title}`              | GitHub Issue                     | Task title                                      |
| `{commit_sha}`              | Git                              | SHA of the commit being verified                |
| `{failing_tests_block}`     | Test runner output parser        | Structured list of failing test names and errors|
| `{modified_file_content}`   | Git diff                         | Content of files modified by the Build Loop     |
| `{coverage_threshold}`      | `loop-config.yaml`               | Required coverage percentage                    |
| `{actual_coverage}`         | Coverage tool output             | Measured coverage percentage                    |
| `{uncovered_lines_block}`   | Coverage report parser           | List of uncovered file/line ranges              |
| `{source_file_content}`     | Repository file reader           | Full content of the file being analyzed         |
| `{test_file_content}`       | Repository file reader           | Full content of the test file                   |
| `{lint_violations_block}`   | Linter output parser             | Structured list of new lint violations          |
| `{security_findings_block}` | Security scanner output parser   | Structured list of new security findings        |
| `{linter_name}`             | `loop-config.yaml`               | Name of the linting tool in use                 |
| `{security_tool_name}`      | `loop-config.yaml`               | Name of the security scanning tool in use       |
| `{changed_files_summary}`   | Git diff stats                   | Summary of files added/modified/deleted         |

---

## Related Documents

- [Verify Loop](../loops/verify-loop.md)
- [Monitor Loop Prompts](monitor-loop-prompts.md)
- [Best Practices for Autonomous Sessions](../best-practices/autonomous-session-best-practices.md)
