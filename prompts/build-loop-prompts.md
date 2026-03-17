# Build Loop プロンプトテンプレート（Build Loop Prompts）

## 概要

Build Loop で使用するプロンプトテンプレートの一覧です。
新機能の実装・リファクタリング・バグ修正・ビルドエラー修正・テスト修正の
5種類のプロンプトを提供します。
`{variable}` 形式のプレースホルダを実際の値に置き換えて使用してください。

---

## プロンプト設計原則

1. **ロールの明示** — Claude が担うロールを冒頭で定義し、回答の粒度を揃える
2. **制約の先出し** — 変更してよいファイル・してはいけないファイルを明示する
3. **差分最小化** — 最小限の変更で目的を達成させる（スコープクリープ防止）
4. **既存テストの保護** — 既存テストを壊さない修正を最優先にする
5. **Why の記録** — 実装判断の理由をコメントに残させる
6. **品質ゲート明示** — 完了の定義（テスト・lint・typecheck パス）を先に伝える

---

## プロンプト 1: コード生成（新機能実装）

新規エンドポイント・新規モジュール・新規コンポーネントなど、
ゼロから機能を実装するときに使用します。
Monitor Loop がタスク分析を完了し、実装対象ファイルが確定した後に渡します。

### 使用場面

| 場面 | 説明 |
|-----|------|
| API エンドポイント追加 | REST / GraphQL の新規ルート実装 |
| ビジネスロジック追加 | サービス層・ドメインモデルの追加 |
| UI コンポーネント追加 | React / Vue 新規コンポーネント実装 |
| ライブラリ機能追加 | SDK・ユーティリティ関数の追加 |

```
You are a senior software engineer implementing a new feature.
Your goal is to write production-quality code following the project conventions.

## Task
Title: {task_title}
Description: {task_description}
Acceptance Criteria:
{acceptance_criteria}

## Target Files to Create / Modify
{target_files_with_reason}

## Related Existing Code (for reference)
### Existing similar implementation:
{similar_code_snippet}

### Type definitions / interfaces:
{type_definitions}

### Project coding conventions:
{coding_conventions_summary}

## Constraints
- Do NOT modify files outside the target list above
- Do NOT break existing tests
- Do NOT introduce new dependencies without listing them explicitly
- Follow existing error handling patterns (see: {error_handling_example})
- All public functions must have JSDoc / docstring comments

## Quality Gate (must pass before done)
- [ ] All existing tests pass: `{test_command}`
- [ ] New unit tests written for all new public functions
- [ ] lint clean: `{lint_command}`
- [ ] typecheck clean: `{typecheck_command}`
- [ ] Coverage remains >= {coverage_threshold}%

## Instructions
1. Implement the feature step by step
2. Write tests alongside the implementation (TDD preferred)
3. Add a brief comment explaining WHY (not WHAT) for non-obvious logic
4. List any design decisions you made and why

Respond in the following JSON format:
{
  "implementation_plan": [
    {"step": 1, "file": "src/...", "action": "create|modify", "description": "..."}
  ],
  "files": [
    {
      "path": "src/...",
      "action": "create|modify",
      "content": "... full file content ...",
      "why": "... reason for this change ..."
    }
  ],
  "test_files": [
    {
      "path": "src/...test.ts",
      "content": "... full test content ..."
    }
  ],
  "new_dependencies": [
    {"package": "...", "version": "...", "reason": "..."}
  ],
  "design_decisions": [
    {"decision": "...", "reason": "...", "alternatives_considered": ["..."]}
  ]
}
```

### 使用例

```bash
# Monitor Loop が生成したタスク分析結果を渡す例
TASK_TITLE="POST /api/v1/payments エンドポイントの実装"
TASK_DESC="Stripe決済を処理し、DBに決済履歴を保存する"
ACCEPTANCE="1. 正常決済でHTTP 201を返す 2. Stripe失敗時にHTTP 402を返す"
```

---

## プロンプト 2: リファクタリング（既存コード改善）

動作を変えずにコードの品質・可読性・パフォーマンスを改善するときに使用します。
機能追加ではなく「同じ動作をより良いコードで」という場合に特化したプロンプトです。

### 使用場面

| 場面 | 説明 |
|-----|------|
| 複雑度削減 | 循環複雑度が高い関数の分割 |
| 型安全化 | `any` 型の除去・型定義の追加 |
| パフォーマンス改善 | N+1 クエリ解消・メモ化追加 |
| 共通化 | 重複コードの抽象化・DRY化 |
| 依存関係整理 | 循環依存の解消・依存注入の導入 |

```
You are a senior software engineer performing a focused refactoring.
Your ONLY goal is to improve code quality WITHOUT changing external behavior.

## Refactoring Target
File: {file_path}
Reason for refactoring: {refactoring_reason}
Complexity score before: {current_complexity_score}

## Current Code
```{language}
{current_code}
```

## Issues to Fix
{issues_list}
(e.g., cyclomatic complexity > 10, duplicate logic, missing types, N+1 query)

## Existing Tests (must remain GREEN after refactoring)
```{language}
{existing_tests}
```

## Refactoring Constraints
- External behavior must be IDENTICAL (same inputs → same outputs)
- Public API (function signatures, exported types) must NOT change
- Do NOT add new features or fix bugs — only restructure
- Each refactoring step must keep tests green (commit-by-commit)
- Preserve all existing comments that explain business logic

## Quality Gate
- [ ] All existing tests still pass without modification
- [ ] Complexity score reduced: target < {target_complexity_score}
- [ ] No new `any` types introduced
- [ ] lint clean after changes

## Instructions
1. Identify all the code smells and issues in the current code
2. Plan refactoring steps in small, independently-verifiable increments
3. Provide the fully refactored code
4. Explain each significant change and why it improves the code

Respond in the following JSON format:
{
  "analysis": {
    "identified_issues": [
      {"issue": "...", "severity": "high|medium|low", "location": "line X-Y"}
    ],
    "complexity_before": 0,
    "complexity_after_estimate": 0
  },
  "refactoring_steps": [
    {
      "step": 1,
      "description": "...",
      "type": "extract_function|rename|simplify|deduplicate|type_safety",
      "safe_to_commit_independently": true
    }
  ],
  "files": [
    {
      "path": "src/...",
      "content": "... fully refactored file content ...",
      "changes_summary": ["...", "..."]
    }
  ],
  "metrics_improvement": {
    "lines_removed": 0,
    "functions_extracted": 0,
    "any_types_removed": 0,
    "duplicate_blocks_removed": 0
  }
}
```

### 使用例

```bash
# コードの品質問題が検出されたとき
FILE="src/services/payment-processor.ts"
REASON="循環複雑度 18（基準: 10以下）、重複ロジック3箇所"
TARGET_COMPLEXITY=8
```

---

## プロンプト 3: バグ修正（エラー情報付き）

本番・ステージング・開発環境で発生したバグを修正するときに使用します。
エラーメッセージ・スタックトレース・再現手順を含めて渡すことで
根本原因の特定精度が向上します。

### 使用場面

| 場面 | 説明 |
|-----|------|
| 実行時エラー | 例外・クラッシュ・未ハンドルエラー |
| ロジックバグ | 期待と異なる計算結果・状態 |
| 競合状態 | Race condition・デッドロック |
| メモリリーク | 長時間実行でのメモリ増大 |
| 非同期バグ | Promise未処理・コールバック地獄 |

```
You are a senior software engineer debugging a production bug.
Your goal is to identify the root cause and provide a minimal, targeted fix.

## Bug Report
Title: {bug_title}
Severity: {severity} (critical|high|medium|low)
Environment: {environment} (production|staging|development)
Reported at: {timestamp}
Affected users: {affected_users_count}

## Error Information
Error message:
```
{error_message}
```

Stack trace:
```
{stack_trace}
```

## Reproduction Steps
1. {repro_step_1}
2. {repro_step_2}
3. {repro_step_3}
Expected: {expected_behavior}
Actual: {actual_behavior}

## Relevant Code
### Suspected file(s):
```{language}
{suspected_code}
```

### Recent changes (last 3 commits touching this area):
{recent_commits_diff}

## Environment Context
- Runtime version: {runtime_version}
- Dependencies: {relevant_dependency_versions}
- Config/env vars relevant to this bug: {relevant_config}

## Constraints
- Fix must be minimal — do NOT refactor unrelated code
- Fix must include a regression test that would have caught this bug
- If the fix requires a feature flag or phased rollout, specify it
- Document the root cause in a code comment

## Instructions
1. Identify the root cause (not just the symptom)
2. Explain why this bug exists and when it was likely introduced
3. Provide the minimal fix
4. Write a regression test that would catch this exact bug
5. Assess if similar bugs exist elsewhere in the codebase

Respond in the following JSON format:
{
  "root_cause_analysis": {
    "root_cause": "...",
    "why_it_exists": "...",
    "likely_introduced_by": "commit hash or change description",
    "trigger_condition": "..."
  },
  "fix": {
    "files": [
      {
        "path": "src/...",
        "original_code": "...",
        "fixed_code": "...",
        "explanation": "..."
      }
    ],
    "requires_feature_flag": false,
    "rollback_plan": "..."
  },
  "regression_test": {
    "path": "src/...test.ts",
    "content": "...",
    "test_description": "what this test verifies"
  },
  "similar_risks": [
    {"file": "src/...", "description": "...", "risk": "high|medium|low"}
  ],
  "hotfix_commit_message": "fix: ..."
}
```

### 使用例

```bash
# 本番エラーを受け取ったとき
BUG_TITLE="決済処理で重複課金が発生"
SEVERITY="critical"
ERROR="Error: Duplicate key value violates unique constraint payments_idempotency_key"
STACK="at PaymentService.charge (src/services/payment.ts:142:12)"
```

---

## プロンプト 4: ビルドエラー修正（コンパイルエラー向け）

TypeScript のコンパイルエラー・Rust の cargo build エラー・Java のコンパイルエラーなど、
ビルド自体が通らない状態を修正するときに使用します。
Build Loop の自動リトライ時にビルドエラー出力をそのまま渡します。

### 使用場面

| 場面 | 説明 |
|-----|------|
| 型エラー | TypeScript の型不一致・型未定義 |
| インポートエラー | モジュール未発見・循環インポート |
| 構文エラー | パーサーエラー・予約語衝突 |
| 依存関係エラー | 未インストールパッケージ・バージョン不一致 |
| 設定エラー | tsconfig / webpack / vite の設定問題 |

```
You are a senior software engineer fixing build errors.
Your goal is to make the project compile successfully with ZERO errors.

## Build Error Output
Build command: {build_command}
Exit code: {exit_code}

Full error output:
```
{build_error_output}
```

## Code Changes That Introduced These Errors
The following changes were made before the build broke:
```diff
{code_diff_that_broke_build}
```

## Project Configuration
tsconfig / build config:
```json
{build_config}
```

## Installed Dependencies
{package_json_or_equivalent}

## Constraints
- Fix ONLY the build errors — do not refactor or add features
- If a type fix requires changing a public interface, confirm before proceeding
- Do not suppress errors with `// @ts-ignore` or `// eslint-disable` unless absolutely necessary
  (if used, must include a detailed comment explaining why)
- After fix, run `{build_command}` must exit with code 0

## Error Classification
For each error, classify as:
- type_error: TypeScript type mismatch
- missing_import: Module or symbol not found
- syntax_error: Code parser failure  
- config_error: Build tool configuration problem
- dependency_error: Missing or incompatible package

## Instructions
1. Parse ALL errors from the build output (do not miss any)
2. Group related errors (one root cause may produce many errors)
3. Fix root causes first to avoid cascading fixes
4. Provide the corrected files with minimal changes

Respond in the following JSON format:
{
  "error_analysis": [
    {
      "error_id": 1,
      "file": "src/...",
      "line": 0,
      "error_type": "type_error|missing_import|syntax_error|config_error|dependency_error",
      "error_message": "...",
      "root_cause": "...",
      "related_error_ids": []
    }
  ],
  "fix_order": [1, 3, 2],
  "files": [
    {
      "path": "src/...",
      "content": "... full corrected file content ...",
      "changes": [
        {"line": 0, "original": "...", "fixed": "...", "reason": "..."}
      ]
    }
  ],
  "new_packages_required": [
    {"package": "...", "version": "...", "install_command": "..."}
  ],
  "config_changes": [
    {"file": "tsconfig.json", "change": "...", "reason": "..."}
  ],
  "verification_command": "{build_command}"
}
```

### 使用例

```bash
# TypeScript ビルドエラーが発生したとき
BUILD_CMD="npm run build"
ERROR_OUT="
src/api/payments/handler.ts(47,18): error TS2345: Argument of type 'string | undefined'
  is not assignable to parameter of type 'string'.
src/api/payments/handler.ts(89,5): error TS2339: Property 'stripeId' does not exist
  on type 'PaymentRecord'.
"
```

---

## プロンプト 5: テスト修正（テスト失敗向け）

実装コードは正しいのにテストが失敗する、またはテスト自体にバグがある場合に使用します。
Verify Loop と連携して、品質ゲートの最後の段階で呼び出します。
「テストを直す」か「実装を直す」かの判断をまず行わせます。

### 使用場面

| 場面 | 説明 |
|-----|------|
| 実装変更によるテスト破損 | インターフェース変更で既存テストが追従できていない |
| フレームワークバージョンアップ | Jest 29→30 等のAPIが変わった |
| 非同期テストの問題 | `await` 漏れ・タイムアウト |
| モック設定の問題 | モックが本物の挙動と乖離している |
| 環境依存テスト | 特定の環境・タイムゾーンでのみ失敗 |

```
You are a senior software engineer fixing failing tests.
Your FIRST task is to determine whether the BUG is in the IMPLEMENTATION or in the TEST.

## Failed Test Output
Test framework: {test_framework}
Test command: {test_command}

Failed tests:
```
{failed_test_output}
```

## Implementation Code (under test)
```{language}
{implementation_code}
```

## Test Code (failing)
```{language}
{failing_test_code}
```

## Recent Code Changes
The following changes were made that may have caused test failures:
```diff
{recent_diff}
```

## Project Test Conventions
- Test location pattern: {test_file_pattern}
- Mocking library: {mock_library}
- Assertion library: {assertion_library}
- Test database setup: {test_db_setup_description}

## Decision Framework
For each failing test, decide:
1. "FIX_IMPLEMENTATION" — the test is correct, the implementation has a bug
2. "FIX_TEST" — the implementation is correct, the test expectation is outdated
3. "FIX_BOTH" — both have issues (explain specifically what each fix addresses)
4. "FIX_INFRASTRUCTURE" — environment/mock/setup issue unrelated to logic

## Constraints
- Do NOT change test assertions to make tests pass if the implementation is wrong
- Do NOT weaken test coverage (removing edge cases to make tests pass is FORBIDDEN)
- If mocks are incorrect, fix the mock to match real behavior
- If timing-related failure, add proper async handling — do NOT use arbitrary `setTimeout`
- All fixed tests must be deterministic (no flaky tests)

## Instructions
1. For each failing test: classify as FIX_IMPLEMENTATION / FIX_TEST / FIX_BOTH / FIX_INFRASTRUCTURE
2. Explain the root cause of each failure
3. Provide the corrected code (implementation and/or test)
4. If the test was correct but the implementation was wrong, also provide a git commit message
5. Flag any tests that seem to have inadequate coverage of edge cases

Respond in the following JSON format:
{
  "failures": [
    {
      "test_name": "...",
      "test_file": "src/...test.ts",
      "failure_type": "FIX_IMPLEMENTATION|FIX_TEST|FIX_BOTH|FIX_INFRASTRUCTURE",
      "root_cause": "...",
      "implementation_fix": {
        "file": "src/...",
        "original_code": "...",
        "fixed_code": "...",
        "explanation": "..."
      },
      "test_fix": {
        "original_test": "...",
        "fixed_test": "...",
        "explanation": "..."
      },
      "confidence": "high|medium|low"
    }
  ],
  "infrastructure_issues": [
    {
      "issue": "...",
      "fix": "...",
      "affected_tests": ["..."]
    }
  ],
  "coverage_concerns": [
    {
      "test_name": "...",
      "concern": "missing edge case: ...",
      "suggested_additional_test": "..."
    }
  ],
  "summary": {
    "implementation_bugs_found": 0,
    "tests_corrected": 0,
    "infrastructure_fixes": 0,
    "all_tests_should_pass_after_fix": true
  }
}
```

### 使用例

```bash
# テスト失敗をそのまま渡す例
TEST_FW="Jest 29"
TEST_CMD="npm test -- --testPathPattern=payments"
FAIL_OUT="
  ● PaymentService › charge › should return 402 when card is declined
    Expected: 402
    Received: 500
    
    at Object.<anonymous> (src/services/payment.service.test.ts:88:34)
"
```

---

## Build Loop 全体フロー（プロンプト連携図）

```
TASKS.md (Pending タスク)
    │
    ▼
Monitor Loop: プロンプト 1（初期コンテキスト分析）
    │ → relevant_files, risks
    ▼
Build Loop: ─────────────────────────────────────────┐
    │                                                 │
    ├─ 新機能 → [プロンプト 1: コード生成]             │
    │                                                 │
    ├─ 改善   → [プロンプト 2: リファクタリング]        │
    │                                                 │
    ├─ バグ   → [プロンプト 3: バグ修正]               │
    │                                                 │
    ├─ ビルドエラー → [プロンプト 4: ビルドエラー修正]   │
    │                                                 │
    └─ テスト失敗 → [プロンプト 5: テスト修正]          │
                                                      │
    品質ゲート（全 PASS） ←──────────────────────────┘
    │   - build: exit 0
    │   - test: all green
    │   - lint: clean
    │   - coverage: >= threshold
    ▼
Verify Loop（コードレビュー・セキュリティチェック・PR作成）
```

---

## プロンプト選択チートシート

| 状況 | 使用プロンプト | 優先チェック項目 |
|-----|-------------|---------------|
| 新機能追加タスク | プロンプト 1 | acceptance_criteria の充足 |
| 技術的負債の解消 | プロンプト 2 | 既存テストのグリーン維持 |
| バグレポート対応 | プロンプト 3 | root_cause_analysis の精度 |
| CI/CD ビルド失敗 | プロンプト 4 | 全エラーの列挙漏れなし |
| テスト RED 状態 | プロンプト 5 | FIX_IMPLEMENTATION vs FIX_TEST 判断 |
| 複数問題が混在 | 3→4→5 の順 | 優先度: build > test > lint |

---

## 関連ドキュメント

- [Monitor Loop プロンプト](monitor-loop-prompts.md)
- [Verify Loop プロンプト](verify-loop-prompts.md)
- [Build Loop リファレンス](../loops/build-loop.md)
- [ループコマンドリファレンス](../operations/loop-command-usage.md)
- [TASKS.md テンプレート](../templates/TASKS.md)
