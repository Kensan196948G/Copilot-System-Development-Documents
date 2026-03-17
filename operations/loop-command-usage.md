# Loop Command Usage

## Overview

The `copilot-loops` CLI provides commands to manage and interact with the Monitor, Build, and Verify loops. This reference covers all available commands, flags, and usage examples.

---

## Global Flags

These flags are available on all commands:

| Flag               | Short | Default             | Description                                   |
|--------------------|-------|---------------------|-----------------------------------------------|
| `--config`         | `-c`  | `loop-config.yaml`  | Path to the loop configuration file           |
| `--log-level`      | `-l`  | `info`              | Log verbosity: `debug`, `info`, `warn`, `error`|
| `--log-file`       |       | `~/.copilot-loops/logs/` | Directory for log files                |
| `--no-color`       |       | `false`             | Disable colored output                        |
| `--json`           |       | `false`             | Output in JSON format (for scripting)          |

---

## `start` — Start All Loops

Start the Monitor, Build, and Verify loops together.

```bash
npx copilot-loops start [flags]
```

### Flags

| Flag         | Default | Description                                                  |
|--------------|---------|--------------------------------------------------------------|
| `--daemon`   | `false` | Run in background as a daemon process                        |
| `--dry-run`  | `false` | Parse config and connect to APIs, but do not execute tasks  |
| `--once`     | `false` | Run a single Monitor cycle, then exit                        |
| `--task`     |         | Process a specific task ID only (useful for debugging)       |

### Examples

```bash
# Start all loops interactively
npx copilot-loops start

# Start in daemon mode
npx copilot-loops start --daemon

# Dry-run to validate configuration
npx copilot-loops start --dry-run

# Process only one specific task
npx copilot-loops start --task TASK-42
```

---

## `monitor` — Start Monitor Loop Only

```bash
npx copilot-loops monitor [flags]
```

### Flags

| Flag             | Description                                              |
|------------------|----------------------------------------------------------|
| `--interval`     | Override the polling interval in seconds                 |
| `--once`         | Run a single Monitor cycle, then exit                    |
| `--idle`         | Start in idle monitoring mode (no active task dispatch)  |

### Examples

```bash
# Start the Monitor Loop
npx copilot-loops monitor

# Run one cycle and print the task plan
npx copilot-loops monitor --once --log-level debug

# Monitor with a faster polling interval
npx copilot-loops monitor --interval 30
```

---

## `build` — Start Build Loop Only

```bash
npx copilot-loops build [flags]
```

### Flags

| Flag             | Description                                              |
|------------------|----------------------------------------------------------|
| `--task`         | Process a specific task ID (requires context package)    |
| `--context-file` | Path to a JSON context package file (skip Monitor Loop)  |
| `--no-commit`    | Apply file changes but do not commit to Git              |
| `--no-docker`    | Run builds on the host instead of in Docker              |

### Examples

```bash
# Start the Build Loop in listener mode
npx copilot-loops build

# Run the Build Loop manually on a specific task context
npx copilot-loops build --task TASK-42 --context-file /tmp/task-42-context.json

# Build without Docker (for local development)
npx copilot-loops build --no-docker
```

---

## `verify` — Start Verify Loop Only

```bash
npx copilot-loops verify [flags]
```

### Flags

| Flag             | Description                                              |
|------------------|----------------------------------------------------------|
| `--commit`       | Verify a specific commit SHA                             |
| `--task`         | Verify a specific task's latest build artifact           |
| `--skip-tests`   | Skip test execution (run lint and security only)         |
| `--skip-security`| Skip security scan                                       |
| `--no-docker`    | Run verification on the host instead of in Docker        |

### Examples

```bash
# Start the Verify Loop in listener mode
npx copilot-loops verify

# Verify a specific commit
npx copilot-loops verify --commit a1b2c3d

# Quick verify without running the full test suite
npx copilot-loops verify --task TASK-42 --skip-tests
```

---

## `status` — Check System Status

```bash
npx copilot-loops status [flags]
```

### Flags

| Flag      | Description                                    |
|-----------|------------------------------------------------|
| `--watch` | Continuously refresh status (like `watch`)     |

### Examples

```bash
# Check current loop status
npx copilot-loops status

# Watch status in real time
npx copilot-loops status --watch
```

### Sample Output

```
╔═══════════════════════════════════════════════════════════╗
║             Copilot Loop System Status                    ║
╠═══════════════╦═══════════╦══════════╦════════════════════╣
║ Loop          ║ Status    ║ Cycle    ║ Last Activity      ║
╠═══════════════╬═══════════╬══════════╬════════════════════╣
║ Monitor Loop  ║ RUNNING   ║ 47       ║ 23s ago            ║
║ Build Loop    ║ BUILDING  ║ -        ║ TASK-42 attempt 2  ║
║ Verify Loop   ║ IDLE      ║ -        ║ TASK-41 passed     ║
╚═══════════════╩═══════════╩══════════╩════════════════════╝

Active task: TASK-42 "Implement JWT refresh logic" (attempt 2/5)
Queue depth: 3 tasks pending
```

---

## `queue` — Manage the Task Queue

```bash
npx copilot-loops queue <subcommand> [flags]
```

### Subcommands

| Subcommand | Description                                          |
|------------|------------------------------------------------------|
| `list`     | List all tasks in the queue with their status        |
| `add`      | Manually add a task to the queue                     |
| `remove`   | Remove a task from the queue                         |
| `pause`    | Pause a task (prevent it from being dispatched)      |
| `resume`   | Resume a paused task                                 |

### Examples

```bash
# List the task queue
npx copilot-loops queue list

# Add a task manually (bypassing GitHub Issues)
npx copilot-loops queue add \
  --title "Fix null pointer in auth module" \
  --description "src/auth/token.ts line 47 throws NPE when token is undefined" \
  --priority high

# Pause a task
npx copilot-loops queue pause TASK-44

# Remove a task
npx copilot-loops queue remove TASK-44
```

---

## `task` — Inspect and Manage Individual Tasks

```bash
npx copilot-loops task <subcommand> <task-id> [flags]
```

### Subcommands

| Subcommand | Description                                              |
|------------|----------------------------------------------------------|
| `status`   | Show detailed status of a task                           |
| `logs`     | Show all agent logs for a task                           |
| `context`  | Show the context package that was sent to the Build Loop |
| `retry`    | Force a task to retry from the current state             |
| `reset`    | Reset a task to `pending` (start from scratch)           |
| `escalate` | Manually escalate a task to human review                 |

### Examples

```bash
# Check task status
npx copilot-loops task status TASK-42

# View agent logs for a task
npx copilot-loops task logs TASK-42

# Retry a failed task
npx copilot-loops task retry TASK-42

# Escalate a stuck task to human review
npx copilot-loops task escalate TASK-42 --reason "Build errors require architectural decision"
```

---

## `stop` — Stop All Loops

```bash
npx copilot-loops stop [flags]
```

### Flags

| Flag      | Description                                              |
|-----------|----------------------------------------------------------|
| `--force` | Kill immediately without waiting for current task        |
| `--loop`  | Stop only a specific loop: `monitor`, `build`, `verify`  |

### Examples

```bash
# Graceful stop
npx copilot-loops stop

# Force stop all loops immediately
npx copilot-loops stop --force

# Stop only the Build Loop
npx copilot-loops stop --loop build
```

---

## `logs` — Stream Logs

```bash
npx copilot-loops logs [flags]
```

### Flags

| Flag       | Description                                              |
|------------|----------------------------------------------------------|
| `--loop`   | Stream logs from a specific loop only                    |
| `--task`   | Stream logs for a specific task only                     |
| `--follow` | Continuously stream new log lines (default: true)        |
| `--since`  | Show logs since a timestamp (e.g., `10m`, `1h`, `2024-01-15T10:00:00Z`) |

### Examples

```bash
# Stream all logs
npx copilot-loops logs

# Stream only Build Loop logs
npx copilot-loops logs --loop build

# Stream logs for a specific task
npx copilot-loops logs --task TASK-42

# Show logs from the last 10 minutes
npx copilot-loops logs --since 10m
```

---

## Related Documents

- [Copilot Start Guide](copilot-start-guide.md)
- [Autonomous Development Workflow](autonomous-development-workflow.md)
- [Triple Loop Architecture](../architecture/triple-loop-architecture.md)
