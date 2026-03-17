# Build Loop

## Overview

The Build Loop is the code generation and compilation engine of the Triple Loop Architecture. When the Monitor Loop dispatches a context package, the Build Loop takes over: it generates or modifies source code using Copilot CLI, attempts to compile or run the project, and iterates until the build succeeds or the retry limit is reached.

---

## Responsibilities

| Responsibility           | Description                                                      |
|--------------------------|------------------------------------------------------------------|
| Code generation          | Use Copilot CLI to write new or modified source code             |
| Build execution          | Run the project's build system (npm, Maven, Go build, etc.)      |
| Error diagnosis          | Parse build errors and feed them back to Copilot for fixing      |
| Artifact production      | Output compiled artifacts and changed files to the artifact store|
| Retry management         | Track attempt count; enrich prompt on each retry with error info |
| Branch management        | Commit changes to the feature branch after each successful build |

---

## Build Loop Execution Flow

```
┌──────────────────────────────────────────────────────────────────┐
│                      Build Loop Cycle                            │
│                                                                  │
│  1. Receive context package from Monitor Loop                    │
│  2. Construct Copilot prompt (see Prompt Engineering below)      │
│  3. Invoke Copilot CLI → receive code changes                    │
│  4. Apply code changes to working directory                      │
│  5. Run build command (e.g., npm run build)                      │
│  6. Check build result:                                          │
│     ├── success → commit to branch; notify Monitor + Verify      │
│     └── failure:                                                 │
│         ├── attempt_number < max_retries:                        │
│         │   └── append error to context; go to step 2           │
│         └── attempt_number >= max_retries:                       │
│             └── report failure to Monitor Loop; halt             │
└──────────────────────────────────────────────────────────────────┘
```

---

## Prompt Construction

The Build Loop constructs Copilot prompts dynamically based on the task context and current attempt number.

### First Attempt Prompt Structure

```
You are an expert software engineer working on the following task:

TASK: {task_title}
DESCRIPTION: {task_description}

CONSTRAINTS:
{constraints}

RELEVANT FILES:
{context_files_content}

RELATED ISSUES:
{related_issues_summaries}

Please implement the required changes. For each file you modify:
1. Show the complete updated file
2. Explain what you changed and why
3. Note any edge cases you handled

Focus on correctness and test coverage. Do not break existing tests.
```

### Retry Attempt Prompt Structure

On retry, the previous error is prepended:

```
The previous implementation attempt produced the following build error:

ERROR:
{build_error_output}

FAILED FILE: {file_with_error}
LINE: {error_line}

Please diagnose the root cause of this error and provide a corrected
implementation. Show the complete corrected file.

[original prompt context follows...]
```

---

## Build Commands by Ecosystem

The Build Loop detects the project's build system and runs the appropriate command:

| Ecosystem    | Detection File            | Build Command           |
|--------------|---------------------------|-------------------------|
| Node.js      | `package.json`            | `npm run build`         |
| Python       | `setup.py` / `pyproject.toml` | `pip install -e . && python -m py_compile` |
| Go           | `go.mod`                  | `go build ./...`        |
| Java/Maven   | `pom.xml`                 | `mvn compile`           |
| Java/Gradle  | `build.gradle`            | `./gradlew build`       |
| .NET         | `*.csproj`                | `dotnet build`          |
| Rust         | `Cargo.toml`              | `cargo build`           |

---

## Error Classification

The Build Loop classifies build errors to determine the best retry strategy:

| Error Class         | Examples                                   | Retry Strategy                            |
|---------------------|--------------------------------------------|-------------------------------------------|
| `syntax_error`      | Missing semicolon, unclosed brace          | Provide exact error location; ask for fix |
| `type_error`        | Wrong type, missing property               | Include type definitions in context       |
| `import_error`      | Module not found, wrong import path        | Include package.json / go.mod in context  |
| `logic_error`       | Test assertion fails                       | Include failing test + expected output    |
| `dependency_error`  | Missing or incompatible package            | Run install command; log dependency tree  |
| `environment_error` | Missing env var, wrong Node version        | Flag for human review; do not retry       |

---

## Artifact Management

After a successful build, the Build Loop:

1. **Stages changes** – `git add` all modified files
2. **Creates a commit** – with a structured message: `[autonomous] TASK-{id}: {title}`
3. **Pushes to feature branch** – named `autonomous/TASK-{id}`
4. **Registers artifacts** – logs file paths and build hash in the State Manager
5. **Notifies Verify Loop** – dispatches a verification request with the commit SHA

---

## Build Isolation

All builds run inside isolated Docker containers to prevent:
- Environment drift between builds
- Interference between parallel build tasks
- Accidental modification of the host system

```yaml
build:
  docker_image: "node:20-alpine"    # or per-project override
  working_dir: /workspace
  env_vars:
    - NODE_ENV=test
    - CI=true
  mount_read_only:
    - ./src:/workspace/src:ro       # source files (agent writes to copy)
  timeout_seconds: 600
```

---

## Configuration

```yaml
build:
  max_retries: 5
  timeout_minutes: 15
  commit_message_prefix: "[autonomous]"
  branch_prefix: "autonomous/"
  build_isolation: docker
  error_context_lines: 50          # lines of build log to include in retry prompt
  always_run_install: true         # run npm install / pip install before each build
```

---

## Related Documents

- [Triple Loop Architecture](../architecture/triple-loop-architecture.md)
- [Monitor Loop](monitor-loop.md)
- [Verify Loop](verify-loop.md)
