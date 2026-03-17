# Verify Loop プロンプトテンプレート（Verify Loop Prompts）

## 概要

Verify Loop で使用するプロンプトテンプレートの一覧です。
コードレビュー・テスト分析・lint・セキュリティ・PR サマリーの
5種類のプロンプトを提供します。

---

## プロンプト 1: テスト失敗分析

テストが失敗したとき、原因分析と修正方針を生成します。

```
あなたはシニアソフトウェアエンジニアです。
以下のテスト失敗を分析して修正方針を提示してください。

## タスクの背景
タスク: {task_title}
変更内容の概要: {change_summary}

## テスト実行結果
フレームワーク: {test_framework}
実行コマンド: {test_command}

失敗したテスト:
{failed_test_output}

## 変更されたコード（差分）
{code_diff}

## 指示
1. 各テスト失敗の根本原因を特定する（「実装のバグ」か「テストの想定が変わった」かを区別する）
2. 修正アプローチを提示する
3. 修正コードのサンプルを提示する
4. 追加すべきテストケースがあれば提案する

以下の JSON 形式で回答してください:
{
  "failures": [
    {
      "test_name": "...",
      "root_cause": "implementation_bug|test_expectation_change|missing_mock",
      "explanation": "...",
      "fix_approach": "...",
      "code_fix": "...",
      "confidence": "high|medium|low"
    }
  ],
  "additional_test_cases": [
    {"description": "...", "type": "edge_case|regression|happy_path"}
  ]
}
```

---

## プロンプト 2: カバレッジギャップ分析

カバレッジが基準を下回ったとき、追加テストを生成します。

```
あなたはテスト自動化の専門家です。
以下のカバレッジレポートを分析して、テストを追加すべき箇所を特定してください。

## カバレッジレポート
{coverage_report}

## カバレッジ基準
ライン: {line_threshold}%（現在: {current_line_coverage}%）
ブランチ: {branch_threshold}%（現在: {current_branch_coverage}%）

## 対象ソースコード
{source_code}

## 既存のテストコード
{existing_tests}

## 指示
1. カバレッジが低い理由を分析する（テストなし / 到達困難 / テストの書き方の問題）
2. 追加すべきテストケースを優先度順にリストアップする（基準達成に必要な最小限）
3. 各テストケースのコードを生成する
4. モック・スタブが必要な場合は示す

以下の JSON 形式で回答してください:
{
  "gap_analysis": {
    "uncovered_lines": ["src/auth/middleware.ts:47-52", "..."],
    "uncovered_branches": ["...", "..."],
    "reason": "no_test|unreachable|complex_path"
  },
  "test_cases": [
    {
      "description": "...",
      "priority": "high|medium|low",
      "code": "it('...', async () => { ... });",
      "mocks_required": ["...", "..."]
    }
  ],
  "projected_coverage_after": {
    "lines": 85.2,
    "branches": 78.1
  }
}
```

---

## プロンプト 3: lint / コードスタイルレビュー

lint エラーとコードスタイルの問題を修正します。

```
あなたはコードレビュアーです。
以下の lint エラーとコードスタイルの問題を修正してください。

## lint 実行結果
ツール: {lint_tool}
{lint_output}

## 変更されたコード
{code_diff}

## プロジェクトのスタイルガイド
{style_guide_summary}

## 指示
1. 各 lint エラー・警告の修正コードを生成する
2. 自動修正可能なものは即時修正する
3. アーキテクチャの変更が必要な警告（循環依存など）は修正計画を提示する
4. false positive と判断したものは理由とともに無視リストに追加する

以下の JSON 形式で回答してください:
{
  "fixes": [
    {
      "file": "src/...",
      "line": 42,
      "rule": "no-unused-vars",
      "current": "const unused = ...",
      "fixed": "// 削除または使用",
      "auto_fixable": true
    }
  ],
  "requires_refactoring": [
    {
      "issue": "循環依存: A → B → A",
      "plan": "..."
    }
  ],
  "ignored_rules": [
    {"rule": "...", "reason": "..."}
  ]
}
```

---

## プロンプト 4: セキュリティスキャン

セキュリティの問題を検出・修正します。

```
あなたはセキュリティエンジニアです。
以下のコード変更のセキュリティリスクを分析してください。

## 変更されたコード
{code_diff}

## セキュリティスキャン結果
{security_scan_output}

## チェック観点
- インジェクション脆弱性（SQL / コマンド / LDAP）
- XSS / CSRF リスク
- 認証・認可の抜け漏れ
- 機密情報のハードコード
- 安全でない依存関係
- 安全でない設定（デバッグモード / 広いCORS など）
- レート制限の欠如
- エラーメッセージでの情報漏洩

## 指示
1. 各セキュリティ問題を重大度別に分類する（Critical / High / Medium / Low）
2. 各問題の修正コードを提示する
3. Critical / High は即時修正として処理する
4. 特に注意すべき点を強調する

以下の JSON 形式で回答してください:
{
  "findings": [
    {
      "severity": "critical|high|medium|low",
      "category": "injection|xss|auth|secrets|dependency|config",
      "file": "src/...",
      "line": 23,
      "description": "...",
      "exploit_scenario": "...",
      "fix": "...",
      "fixed_code": "..."
    }
  ],
  "overall_risk": "critical|high|medium|low",
  "requires_immediate_action": true
}
```

---

## プロンプト 5: PR サマリー生成

Verify Loop 完了後、マージ可能な PR の説明文を生成します。

```
あなたはテクニカルライターです。
以下のコード変更から、明確で情報の揃った Pull Request の説明文を生成してください。

## タスク情報
タイトル: {task_title}
説明: {task_description}
関連 Issue: #{issue_number}

## 変更内容
コミット一覧:
{commit_log}

変更ファイル:
{changed_files}

差分の概要:
{diff_summary}

## Verify 結果
テスト: {test_result}（{passed}件パス / {failed}件失敗）
カバレッジ: {coverage}%
lint: {lint_result}
セキュリティ: {security_result}

## 指示
PR の説明文を Markdown で生成してください。
- エンジニアとステークホルダーの両方が理解できる内容
- 変更の「何を」「なぜ」「どのように」を明確に
- テスト・確認方法を具体的に記載

以下のテンプレートで生成してください:

---
## 概要
[2〜3文で変更内容のサマリー]

## 変更内容
- [変更点1]
- [変更点2]

## 変更の理由
[なぜこの変更が必要か]

## テスト内容
- [x] {test_suite_name}: {test_count}件パス
- [x] カバレッジ: {coverage}%（基準: {threshold}%）
- [x] lint / typecheck: クリア
- [x] セキュリティスキャン: クリア

## 動作確認手順
```bash
# ローカルでの確認方法
[確認コマンド]
```

## スクリーンショット（UI 変更がある場合）
[画像または「なし」]

## 関連 Issue
Closes #{issue_number}
---
```

---

## 関連ドキュメント

- [Monitor Loop プロンプト](monitor-loop-prompts.md)
- [Verify Loop リファレンス](../loops/verify-loop.md)
- [コードレビュープロンプト](../tasks/03_コードレビュー(CodeReview).md)
