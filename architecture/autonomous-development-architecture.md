# Autonomous Development Architecture

## Overview

The Autonomous Development Architecture defines how GitHub Copilot CLI is used as the backbone of a fully automated software development pipeline. Rather than serving as a simple code-completion tool, Copilot is orchestrated as an active development agent capable of executing complete development cycles with minimal human intervention.

This architecture separates concerns across three primary dimensions:

1. **Perception** – understanding the current state of the codebase and task backlog
2. **Action** – writing code, running builds, and executing tests
3. **Verification** – validating correctness, performance, and compliance

---

## System Components

### 1. Copilot CLI Agent

The core execution engine. Copilot CLI receives structured prompts and returns code, explanations, or shell commands. In autonomous mode, it is invoked repeatedly within loops until a task reaches a defined completion state.

**Key capabilities:**
- Natural language to code translation
- Context-aware code generation using repository history
- Inline error diagnosis and auto-correction
- Shell command generation and execution planning

### 2. Task Dispatcher

A lightweight orchestrator that reads from the task queue (e.g., GitHub Issues, a JSON manifest, or a YAML task file) and dispatches individual tasks to the appropriate agent loop.

```
Task Queue → Task Dispatcher → [Monitor Loop | Build Loop | Verify Loop]
```

### 3. State Manager

Maintains the current state of each task, tracking:
- Task status (`pending`, `in_progress`, `blocked`, `complete`, `failed`)
- Loop iteration counts
- Artifact paths (generated code, test reports, logs)
- Error history for retry logic

### 4. Artifact Store

A versioned store (typically the repository itself combined with CI artifact storage) that holds:
- Generated source code
- Build outputs
- Test reports
- Coverage data
- Agent decision logs

### 5. Feedback Bus

A lightweight pub/sub or file-based messaging channel that allows loops to signal each other. For example, the Build Loop can notify the Monitor Loop when a build fails so the Monitor can re-queue additional context gathering.

---

## High-Level Data Flow

```
┌─────────────────────────────────────────────────────────┐
│                     Task Queue                          │
│         (GitHub Issues / YAML manifest)                 │
└───────────────────┬─────────────────────────────────────┘
                    │ dispatch
                    ▼
┌─────────────────────────────────────────────────────────┐
│                  Task Dispatcher                        │
└──────┬──────────────────┬──────────────────┬────────────┘
       │                  │                  │
       ▼                  ▼                  ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ Monitor Loop │  │  Build Loop  │  │ Verify Loop  │
│ (observe &   │  │  (generate & │  │ (test &      │
│  plan)       │  │   build)     │  │  validate)   │
└──────┬───────┘  └──────┬───────┘  └──────┬───────┘
       │                 │                  │
       └─────────────────┴──────────────────┘
                         │
                         ▼
              ┌──────────────────┐
              │  State Manager   │
              │  Artifact Store  │
              └──────────────────┘
```

---

## Agent Execution Model

Each agent loop follows a **Sense → Plan → Act → Evaluate** cycle:

| Phase    | Description                                                        |
|----------|--------------------------------------------------------------------|
| Sense    | Read task definition, codebase context, and previous outputs       |
| Plan     | Generate a structured prompt for Copilot describing the next step  |
| Act      | Execute Copilot's response (write files, run commands, call APIs)  |
| Evaluate | Check exit codes, test results, or output assertions; decide next  |

If Evaluate determines the step is incomplete or failed, the loop increments its retry counter and re-enters the Sense phase with enriched context (e.g., the error message from the previous attempt).

---

## Termination Conditions

Each task terminates under one of the following conditions:

| Condition     | Trigger                                                  |
|---------------|----------------------------------------------------------|
| `success`     | All assertions pass, all required artifacts produced     |
| `max_retries` | Loop retry limit exceeded (default: 5)                   |
| `timeout`     | Wall-clock limit reached (default: 30 minutes per task)  |
| `blocked`     | Loop detects a prerequisite task that is not yet complete|
| `human_required` | Loop cannot make progress without human input        |

---

## Security and Safety Boundaries

Autonomous agents operate within strict boundaries:

- **No destructive operations** without explicit confirmation flags
- **Read-only access** to production environments
- **Sandboxed build environments** (Docker containers per build)
- **Audit log** of every Copilot prompt and response
- **Human approval gates** for merging to protected branches

---

## Related Documents

- [Triple Loop Architecture](triple-loop-architecture.md)
- [Agent Teams System](agent-teams-system.md)
- [Autonomous Development Workflow](../operations/autonomous-development-workflow.md)
