# Monitor Loop Prompts

## Overview

This document provides the canonical prompt templates used by the Monitor Loop when gathering context and building task packages for the Build Loop. These prompts are designed for Copilot CLI and follow structured prompt engineering principles for AI-driven development.

---

## Prompt Engineering Principles for the Monitor Loop

The Monitor Loop prompts follow these core principles:

1. **Role specification** – always define what role Copilot should take
2. **Explicit output format** – request structured JSON output for machine parsing
3. **Bounded scope** – constrain the task to avoid hallucination or scope creep
4. **Context grounding** – include actual code snippets and issue text, not abstractions
5. **Failure reasoning** – on retries, explicitly describe what went wrong

---

## Prompt 1: Initial Context Analysis

Used when the Monitor Loop first picks up a new task. The goal is to understand which files and code regions are most relevant.

```
You are an expert software engineer performing codebase analysis. Your task is to
identify the most relevant source files and code sections for implementing the
following change.

## Task
Title: {task_title}
Description: {task_description}

## Repository Structure
{repository_file_tree}

## Recent Commits (last 10)
{recent_commits}

## Related Issues
{related_issue_summaries}

## Instructions
1. Identify the 5–10 files most relevant to this task.
2. For each file, explain in one sentence WHY it is relevant.
3. Identify any tests that should be updated or created.
4. Identify any documentation that may need updating.
5. List any potential risks or breaking changes.

Respond in the following JSON format:
{
  "relevant_files": [
    { "path": "src/auth/token.ts", "reason": "Contains the token generation logic that needs to be extended" }
  ],
  "test_files": [
    { "path": "src/auth/__tests__/token.test.ts", "reason": "Unit tests for the token module" }
  ],
  "documentation_files": [
    { "path": "docs/auth.md", "reason": "Documents the authentication flow" }
  ],
  "risks": [
    "Changing token validation logic may affect all routes using the auth middleware"
  ],
  "suggested_approach": "Brief description of the recommended implementation approach"
}
```

---

## Prompt 2: Context Enrichment (Retry)

Used when the Build Loop has failed at least once. Adds error context to help the Monitor Loop update the context package.

```
You are an expert software engineer analyzing a failed code generation attempt.
The autonomous Build Loop attempted to implement the following task but failed.

## Task
Title: {task_title}
Description: {task_description}

## Previous Build Error (Attempt {attempt_number})
File: {error_file}
Line: {error_line}
Error: {error_message}

Full build output:
```
{build_log_excerpt}
```

## Current File Content
```{language}
{current_file_content}
```

## Instructions
1. Diagnose the root cause of the build error.
2. Identify which additional files or context would help fix this error.
3. Suggest a corrected implementation approach.
4. List specific constraints the next Build Loop attempt should follow.

Respond in JSON format:
{
  "root_cause": "Description of why the error occurred",
  "additional_context_files": [
    { "path": "types/auth.d.ts", "reason": "Contains the JwtOptions type definition" }
  ],
  "corrected_approach": "Description of the corrected implementation",
  "constraints_for_next_attempt": [
    "Use the JwtOptions interface from types/auth.d.ts",
    "Do not import directly from jsonwebtoken; use the wrapper in src/lib/jwt.ts"
  ]
}
```

---

## Prompt 3: Task Decomposition

Used for large tasks that exceed the Build Loop's single-prompt capacity. The Monitor Loop asks Copilot to break the task into sequential subtasks.

```
You are a senior software architect planning the implementation of a complex
feature. Your goal is to decompose this task into a series of smaller, independent
subtasks that can be executed sequentially by an autonomous agent.

## Task
Title: {task_title}
Description: {task_description}

## Constraints
{constraints}

## Repository Context
{relevant_files_summary}

## Decomposition Rules
- Each subtask must be independently buildable and testable
- Subtasks must be listed in dependency order (no subtask should depend on a later one)
- Each subtask should take at most 2–3 files to complete
- Include a verification step for each subtask

Respond in JSON format:
{
  "subtasks": [
    {
      "id": 1,
      "title": "Add JwtRefreshOptions interface to types/auth.d.ts",
      "description": "Define the TypeScript interface for refresh token configuration",
      "files_to_modify": ["types/auth.d.ts"],
      "verification": "TypeScript compilation succeeds; interface is exported",
      "depends_on": []
    },
    {
      "id": 2,
      "title": "Implement refreshIfExpiring() in src/auth/token.ts",
      "description": "Add the core refresh logic using the new JwtRefreshOptions interface",
      "files_to_modify": ["src/auth/token.ts"],
      "verification": "Unit tests pass; function returns correct refresh token",
      "depends_on": [1]
    }
  ]
}
```

---

## Prompt 4: Escalation Summary

Used when the Monitor Loop prepares an escalation to a human reviewer.

```
You are an expert software engineer preparing a status report for human review.
The autonomous development system has been unable to complete the following task
after {attempt_count} attempts and requires human assistance.

## Task
Title: {task_title}
Description: {task_description}

## Attempt History
{attempt_history}

## Current State of Modified Files
{modified_files_content}

## Instructions
Write a clear, concise escalation report for a human engineer who needs to:
1. Understand what the system was trying to do
2. Understand why it failed
3. Know what action is needed to unblock progress

Format:
## Summary
[2–3 sentences explaining the task and current status]

## What Was Tried
[Bullet list of attempts and outcomes]

## Root Cause
[Your best assessment of why the system cannot proceed autonomously]

## Recommended Human Action
[Specific, actionable steps the human should take]

## Relevant Files
[List of files the human should review]
```

---

## Prompt 5: Idle State Repository Health Check

Used by the Monitor Loop in idle mode to check for repository health issues.

```
You are a senior software engineer performing a repository health check.
Review the following information and identify any issues that should be
addressed proactively.

## Repository Status
- Failing CI checks: {failing_checks}
- Open PRs older than 7 days: {stale_prs}
- Issues labeled 'bug' with no assignee: {unassigned_bugs}
- Last successful deployment: {last_deployment}
- Dependency security advisories: {security_advisories}

## Instructions
Identify the top 3 issues that should be prioritized and explain why.
For each issue, suggest a specific action.

Respond in JSON format:
{
  "health_issues": [
    {
      "priority": 1,
      "type": "security",
      "description": "3 HIGH severity CVEs in npm dependencies",
      "recommended_action": "Run npm audit fix; review breaking changes in lodash upgrade",
      "estimated_effort": "low"
    }
  ],
  "overall_health": "warn",
  "summary": "Repository is functional but has 3 dependency vulnerabilities that should be addressed this sprint"
}
```

---

## Prompt Variables Reference

| Variable                  | Source                              | Description                                   |
|---------------------------|-------------------------------------|-----------------------------------------------|
| `{task_title}`            | GitHub Issue title                  | Short task name                               |
| `{task_description}`      | GitHub Issue body                   | Full task requirements                        |
| `{repository_file_tree}`  | `git ls-files` output               | File listing (truncated to top-level dirs)    |
| `{recent_commits}`        | `git log --oneline -10`             | Last 10 commit messages                       |
| `{related_issue_summaries}`| GitHub API                         | Summaries of linked issues                    |
| `{attempt_number}`        | State Manager                       | Current retry count                           |
| `{error_file}`            | Build Loop error parser             | File where build error occurred               |
| `{error_line}`            | Build Loop error parser             | Line number of error                          |
| `{error_message}`         | Build Loop error parser             | Exact error message from build output         |
| `{build_log_excerpt}`     | Build Loop logs                     | Last 50 lines of build output                 |
| `{current_file_content}`  | Repository file reader              | Current content of the failing file           |
| `{attempt_history}`       | State Manager                       | JSON array of all previous attempt outcomes   |
| `{failing_checks}`        | GitHub Checks API                   | Names of currently failing CI checks          |

---

## Related Documents

- [Monitor Loop](../loops/monitor-loop.md)
- [Verify Loop Prompts](verify-loop-prompts.md)
- [Best Practices for Autonomous Sessions](../best-practices/autonomous-session-best-practices.md)
