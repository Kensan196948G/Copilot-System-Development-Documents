# Copilot Start Guide

## Overview

This guide walks you through setting up and starting an autonomous development session using Copilot CLI and the Triple Loop system. By the end, you will have a running Monitor Loop that can pick up tasks from your issue tracker and orchestrate Build and Verify loops without manual intervention.

---

## Prerequisites

Before you begin, ensure the following tools are installed and configured:

| Tool              | Version  | Purpose                                     |
|-------------------|----------|---------------------------------------------|
| GitHub CLI (`gh`) | ≥ 2.40   | Authenticating with GitHub and managing PRs |
| Copilot CLI extension | latest | AI-powered code generation                |
| Node.js           | ≥ 20.0   | Running the loop orchestrator (JS/TS)       |
| Docker            | ≥ 24.0   | Isolated build and verify environments      |
| Git               | ≥ 2.40   | Version control operations                  |

### Install Copilot CLI Extension

```bash
gh extension install github/gh-copilot
gh copilot --version    # verify installation
```

### Authenticate with GitHub

```bash
gh auth login
gh auth status          # verify authentication
```

---

## Repository Setup

### 1. Clone Your Project Repository

```bash
git clone https://github.com/your-org/your-project.git
cd your-project
```

### 2. Initialize the Autonomous Configuration

Create a `loop-config.yaml` in the project root:

```yaml
monitor:
  interval_seconds: 120
  max_context_files: 20
  task_queue:
    source: github_issues
    label_filter: "autonomous"
    priority_field: "milestone"
  escalation_retry_limit: 3

build:
  max_retries: 5
  timeout_minutes: 15
  build_isolation: docker
  branch_prefix: "autonomous/"

verify:
  test_command: "npm test"
  coverage_command: "npm run test:coverage"
  lint_command: "npm run lint"
  coverage_thresholds:
    lines: 80
    branches: 75
  create_pull_request_on_pass: true
```

### 3. Label Your GitHub Issues

Apply the `autonomous` label to any issue you want the system to process. The system will only pick up issues with this label by default.

```bash
gh label create autonomous --color "#0075ca" --description "Ready for autonomous processing"
gh issue edit 42 --add-label autonomous
```

---

## Starting the Loops

### Start All Loops Together (Recommended)

The easiest way to start is using the provided start script:

```bash
npx copilot-loops start --config loop-config.yaml
```

This starts the Monitor, Build, and Verify loops as background processes and streams logs to stdout.

### Start Individual Loops

You can also start each loop separately for more control:

```bash
# Terminal 1: Monitor Loop
npx copilot-loops monitor --config loop-config.yaml

# Terminal 2: Build Loop
npx copilot-loops build --config loop-config.yaml

# Terminal 3: Verify Loop
npx copilot-loops verify --config loop-config.yaml
```

### Run as a Background Service (Daemon Mode)

For long-running autonomous sessions, run in daemon mode:

```bash
npx copilot-loops start --config loop-config.yaml --daemon
npx copilot-loops status    # check running status
npx copilot-loops logs      # stream logs
npx copilot-loops stop      # graceful shutdown
```

---

## Verifying the System is Running

### Check Loop Status

```bash
npx copilot-loops status
```

Expected output:

```
✓ Monitor Loop  RUNNING  (cycle: 47, last_check: 30s ago)
✓ Build Loop    IDLE     (awaiting tasks)
✓ Verify Loop   IDLE     (awaiting artifacts)
```

### Check the Task Queue

```bash
npx copilot-loops queue list
```

### View Active Task Progress

```bash
npx copilot-loops task status TASK-42
```

---

## First Autonomous Run

### 1. Create a Test Issue

Create a simple GitHub Issue to test the system:

```bash
gh issue create \
  --title "Add hello world function to utils" \
  --body "Add a function named helloWorld() to src/utils/greeting.ts that returns the string 'Hello, World!'. Include a corresponding unit test." \
  --label autonomous
```

### 2. Watch the Monitor Loop Pick It Up

Within the next Monitor Loop cycle (up to 2 minutes), you should see:

```
[Monitor] New task detected: TASK-1 "Add hello world function to utils"
[Monitor] Gathering context for TASK-1...
[Monitor] Context package ready: 3 files, 1200 tokens
[Monitor] Dispatching TASK-1 to Build Loop
```

### 3. Watch the Build Loop Execute

```
[Build] Received TASK-1 (attempt 1/5)
[Build] Constructing Copilot prompt...
[Build] Invoking Copilot CLI...
[Build] Applying changes to src/utils/greeting.ts
[Build] Running: npm run build
[Build] Build succeeded
[Build] Committed: autonomous/TASK-1 (commit: a1b2c3d)
[Build] Notifying Verify Loop...
```

### 4. Watch the Verify Loop Validate

```
[Verify] Received artifact for TASK-1 (commit: a1b2c3d)
[Verify] Running: npm test
[Verify] Tests: 143 passed, 0 failed
[Verify] Coverage: 82.1% (threshold: 80%) ✓
[Verify] Lint: 0 new violations ✓
[Verify] Security: 0 new CVEs ✓
[Verify] All gates passed. Task TASK-1 is merge-ready.
[Verify] Creating Pull Request...
[Verify] PR opened: #15 "autonomous: Add hello world function to utils"
```

---

## Stopping the System

```bash
# Graceful stop (wait for current task to complete)
npx copilot-loops stop

# Immediate stop (interrupt current task)
npx copilot-loops stop --force
```

---

## Troubleshooting

### Common Issues

| Issue                                    | Solution                                                       |
|------------------------------------------|----------------------------------------------------------------|
| Copilot CLI returns empty response       | Check `gh auth status`; re-authenticate if token expired       |
| Build Loop retries exhausted             | Inspect `~/.copilot-loops/logs/build.log` for error details    |
| Monitor Loop not picking up issues       | Verify the `autonomous` label is applied to issues             |
| Docker build fails                       | Ensure Docker daemon is running: `docker ps`                   |
| Coverage gate always fails               | Lower threshold in `loop-config.yaml` or add more tests first  |

---

## Related Documents

- [Loop Command Usage](loop-command-usage.md)
- [Autonomous Development Workflow](autonomous-development-workflow.md)
- [Triple Loop Architecture](../architecture/triple-loop-architecture.md)
