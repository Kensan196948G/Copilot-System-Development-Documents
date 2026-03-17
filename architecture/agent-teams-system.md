# Agent Teams System

## Overview

The Agent Teams System organizes individual Copilot agents into specialized teams, each responsible for a specific area of the development lifecycle. Rather than using a single general-purpose agent for all tasks, this system assigns narrowly scoped agents to roles — improving focus, reliability, and the ability to optimize prompts for each domain.

---

## Team Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                       Orchestrator Agent                        │
│        (coordinates all teams; owned by Task Dispatcher)        │
└──────────────────────────┬──────────────────────────────────────┘
                           │
       ┌───────────────────┼───────────────────┐
       │                   │                   │
       ▼                   ▼                   ▼
┌─────────────┐   ┌─────────────────┐   ┌─────────────────┐
│  Analysis   │   │  Development    │   │    Quality      │
│    Team     │   │     Team        │   │    Team         │
└─────────────┘   └─────────────────┘   └─────────────────┘
```

---

## Teams and Roles

### Orchestrator Agent

The master coordinator. Receives high-level task descriptions and decomposes them into subtasks assigned to each team. Monitors overall progress and handles cross-team dependencies.

**Responsibilities:**
- Task decomposition and prioritization
- Dependency resolution between subtasks
- Progress monitoring and escalation
- Final integration and handoff decisions

---

### Analysis Team

Responsible for understanding what needs to be built or fixed before any code is written.

| Agent          | Role                                                          |
|----------------|---------------------------------------------------------------|
| `context-agent`  | Reads and summarizes relevant code files and documentation  |
| `issue-agent`    | Parses GitHub Issues, PRs, and comments for requirements    |
| `dependency-agent` | Analyzes package dependencies and version constraints    |
| `impact-agent`   | Identifies which parts of the codebase may be affected      |

**Output:** A structured context package (JSON/YAML) passed to the Development Team.

---

### Development Team

Responsible for writing and modifying code to implement the required changes.

| Agent              | Role                                                      |
|--------------------|-----------------------------------------------------------|
| `codegen-agent`    | Generates new source code based on context packages       |
| `refactor-agent`   | Modifies existing code while preserving behavior          |
| `fix-agent`        | Diagnoses and repairs build errors and failed tests       |
| `migration-agent`  | Handles schema migrations, API version upgrades, etc.     |
| `docs-agent`       | Generates or updates inline documentation and READMEs     |

**Output:** Modified or created source files committed to a feature branch.

---

### Quality Team

Responsible for validating that the code produced by the Development Team is correct, safe, and maintainable.

| Agent              | Role                                                       |
|--------------------|------------------------------------------------------------|
| `test-agent`       | Runs the test suite and reports pass/fail/coverage         |
| `lint-agent`       | Runs static analysis and code style checks                 |
| `security-agent`   | Scans for vulnerabilities, secrets, and insecure patterns  |
| `review-agent`     | Performs AI-driven code review, flags design issues        |
| `perf-agent`       | Runs performance benchmarks and flags regressions          |

**Output:** Quality report submitted to the Verify Loop; regressions fed back to Development Team.

---

## Agent Communication Protocol

Agents communicate through structured JSON messages on the Feedback Bus:

```json
{
  "sender": "codegen-agent",
  "receiver": "fix-agent",
  "task_id": "TASK-1042",
  "type": "error_report",
  "payload": {
    "file": "src/auth/token.ts",
    "line": 47,
    "error": "Property 'expiresIn' does not exist on type 'JwtOptions'",
    "build_log": "...(truncated)..."
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

**Message types:**

| Type              | Direction           | Description                                  |
|-------------------|---------------------|----------------------------------------------|
| `task_assignment` | Orchestrator → Team | Assign a new subtask to a team               |
| `context_package` | Analysis → Dev      | Deliver gathered context for code generation |
| `error_report`    | Build → Fix         | Report a build or runtime error              |
| `quality_report`  | Quality → Monitor   | Deliver test/lint/security results           |
| `escalation`      | Any → Human         | Request human review or decision             |
| `status_update`   | Any → Orchestrator  | Report progress on current subtask           |

---

## Team Coordination Patterns

### Sequential Pattern

Used for well-defined tasks with clear dependencies:

```
Analysis → Development → Quality → (merge or retry)
```

### Parallel Pattern

Used when multiple independent subtasks can proceed simultaneously:

```
Analysis ──┬── Development (feature A) ──┬── Quality
           └── Development (feature B) ──┘
```

### Iterative Pattern

Used when quality gates fail and the development team must rework:

```
Development → Quality → [fail] → Development (retry with error context)
                      → [pass] → Merge
```

---

## Scaling the Team

For large codebases or high task throughput, additional agent instances can be spawned horizontally:

- Multiple `codegen-agent` instances working on different modules in parallel
- Dedicated `fix-agent` per failing test suite
- Separate `security-agent` running on every PR regardless of other loops

All agent instances share the same State Manager and Artifact Store to prevent duplication and enable coordination.

---

## Mermaid 図: Agent Teams 全体構造

```mermaid
graph TB
    O[🎯 Orchestrator Agent<br/>タスク分解・調整]

    O --> AT[📊 Analysis Team]
    O --> DT[💻 Development Team]
    O --> QT[✅ Quality Team]
    O --> RT[🚀 Release Team]

    subgraph AT[Analysis Team]
        CA[context-agent<br/>コード読解・要約]
        IA[issue-agent<br/>Issue・PR解析]
        DA[dependency-agent<br/>依存関係分析]
        IPA[impact-agent<br/>影響範囲評価]
    end

    subgraph DT[Development Team]
        CGA[codegen-agent<br/>コード生成]
        RFA[refactor-agent<br/>既存コード改善]
        FXA[fix-agent<br/>エラー修正]
        MA[migration-agent<br/>マイグレーション]
        DOCA[docs-agent<br/>ドキュメント生成]
    end

    subgraph QT[Quality Team]
        TA[test-agent<br/>テスト実行]
        LA[lint-agent<br/>静的解析]
        SA[security-agent<br/>セキュリティ診断]
        RA[review-agent<br/>コードレビュー]
        PA[performance-agent<br/>パフォーマンス測定]
    end

    subgraph RT[Release Team]
        VRA[version-agent<br/>バージョン管理]
        CHA[changelog-agent<br/>変更履歴生成]
        PRA[pr-agent<br/>PR作成・マージ]
    end

    AT --> |コンテキストパッケージ| DT
    DT --> |成果物| QT
    QT --> |承認/否認| RT
    RT --> |完了通知| O
```

## Mermaid 図: Agent 間メッセージフロー

```mermaid
sequenceDiagram
    participant ORC as Orchestrator
    participant CA as context-agent
    participant CGA as codegen-agent
    participant TA as test-agent
    participant PRA as pr-agent

    ORC->>CA: タスク分析依頼 {task_id, repo, priority}
    CA-->>ORC: コンテキストパッケージ {files[], issues[], constraints}
    ORC->>CGA: コード生成依頼 {context_package, target_files}
    CGA-->>ORC: 生成結果 {changed_files[], build_status}
    ORC->>TA: テスト実行依頼 {artifacts, test_command}
    TA-->>ORC: テスト結果 {pass_count, fail_count, coverage}
    alt 全テスト通過
        ORC->>PRA: PR作成依頼 {branch, title, description}
        PRA-->>ORC: PR URL {pr_number, html_url}
    else テスト失敗
        ORC->>CGA: 修正依頼 {error_context, failed_tests}
    end
```

## Mermaid 図: カスタムエージェント対応マップ

```mermaid
graph LR
    subgraph GitHub[.github/agents/]
        BA[backend-agent.agent.md]
        FA[frontend-agent.agent.md]
        TW[test-writer.agent.md]
        SEC[security-agent.agent.md]
        DOC[docs-agent.agent.md]
    end

    subgraph Teams[Agent Teams ロール対応]
        DT_CGA[codegen-agent<br/>fix-agent]
        DT_FA[codegen-agent<br/>UI専門]
        QT_TA[test-agent]
        QT_SA[security-agent]
        DT_DA[docs-agent]
    end

    BA -->|担当| DT_CGA
    FA -->|担当| DT_FA
    TW -->|担当| QT_TA
    SEC -->|担当| QT_SA
    DOC -->|担当| DT_DA
```

---

## Related Documents

- [Autonomous Development Architecture](autonomous-development-architecture.md)
- [Triple Loop Architecture](triple-loop-architecture.md)
- [Autonomous Development Workflow](../operations/autonomous-development-workflow.md)
