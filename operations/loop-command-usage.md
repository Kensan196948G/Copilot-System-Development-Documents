# ループコマンドリファレンス（Loop Command Usage）

## 概要

Claude Code の `/loop` コマンドと `triple-loop-15h.sh` スクリプトの
完全なリファレンスです。全コマンド・フラグ・使用例を網羅します。

---

## /loop コマンドの基本

`/loop` は Claude Code セッション内で使用するコマンドです。
時間制限付きの自律実行ループを起動します。

```
/loop [時間(m)] [実行指示]
```

| 引数 | 説明 | 例 |
|-----|------|---|
| 時間(m) | 実行制限時間（分） | `900m`（15時間）|
| 実行指示 | Claude への詳細な指示文 | 下記テンプレート参照 |

---

## プリセットコマンド

### 🔄 2サイクル15H（推奨・週5日運用の標準）

```
/loop 900m 以下の3重ループを2サイクル実行してください。（中略）
```

→ 詳細は [起動ガイド](copilot-start-guide.md) を参照

**特徴:**
- 実装タスク: 2タスク/日
- 総稼働: 15時間（Monitor×2 + Build×2 + Verify×2 + CD + 最終処理）
- 週5日 × 15H = 75H/週（安全圏内）

---

### 🔄 1サイクル8H（軽量・リミット節約）

```
/loop 450m 以下の3重ループを1サイクルとして実行してください。（中略）
```

**特徴:**
- 実装タスク: 1タスク/日
- 総稼働: 8時間
- 週5日 × 8H = 40H/週（余裕あり）

---

### 🔍 Monitor 単体（現状把握のみ）

```
/loop 30m 本ファイル内の「Monitor Loop 指示」に従い、プロジェクトの現状把握を実行してください。コード変更は一切行わないこと。結果を .loop-monitor-report.md に出力。異常検知時は .loop-alert.md も出力。
```

**出力ファイル:**
- `.loop-monitor-report.md` — 現状分析レポート
- `.loop-alert.md` — 異常検知時のみ（セキュリティ脆弱性・テスト失敗・ビルドエラー）

---

### 🔨 Build 単体（実装のみ）

```
/loop 120m 本ファイル内の「Build Loop 指示」に従い、TASKS.mdから優先度順にタスクを選択し5段階開発ステップを実行してください。各Step完了ごとにmainへcommit。全操作を自動承認で続行。完了時に .loop-build-handoff.md を出力。
```

**Build の5段階ステップ:**
1. 要件分析・設計（AGENTS.md に記録）
2. コア実装
3. テスト実装
4. lint / typecheck 修正
5. ドキュメント更新 + commit

**出力ファイル:**
- `.loop-build-handoff.md` — Verify Loop への引き継ぎ文書

---

### 🧪 Verify 単体（レビュー・テストのみ）

```
/loop 240m 本ファイル内の「Verify Loop 指示」に従い、.loop-build-handoff.mdを読み込みBuild成果物をレビュー・テスト・デバッグ修正してください。修正はmainへcommit。完了時に .loop-verify-report.md を出力。
```

**Verify の実施内容:**
- コードレビュー（品質・設計・セキュリティ）
- テスト実行（ユニット・統合）
- カバレッジ確認（80%以上）
- lint / typecheck 確認
- デバッグ修正（最大15回）

**出力ファイル:**
- `.loop-verify-report.md` — レビュー・テスト結果レポート

---

## 運用モード別比較

| モード | コマンド | 稼働時間 | タスク数/日 | 週消費（5日） |
|-------|---------|---------|------------|-------------|
| 2サイクル15H | `/loop 900m` | 15H | 2タスク | 75H |
| 1サイクル8H | `/loop 450m` | 8H | 1タスク | 40H |
| Monitor のみ | `/loop 30m` | 30分 | 0（調査のみ）| - |
| Build のみ | `/loop 120m` | 2H | 0.5タスク | - |
| Verify のみ | `/loop 240m` | 4H | 0.5タスク | - |

---

## loop-config.yaml（詳細設定）

プロジェクトルートに配置して挙動をカスタマイズできます：

```yaml
monitor:
  interval_seconds: 120       # ポーリング間隔
  max_context_files: 20       # コンテキストに含める最大ファイル数
  task_queue:
    source: tasks_md          # TASKS.md から読み込む
    priority_field: "Priority: High/Medium/Low"
  escalation_retry_limit: 3   # エスカレーション前の最大リトライ回数

build:
  max_retries: 15             # CI 修復 AI の最大リトライ数
  timeout_minutes: 120        # 1タスクのタイムアウト
  build_isolation: docker     # 隔離実行（docker / host）
  branch_prefix: "feature/"
  commit_style: conventional  # Conventional Commits 規約

verify:
  test_command: "npm test"
  coverage_command: "npm run test:coverage"
  lint_command: "npm run lint"
  typecheck_command: "npm run typecheck"
  coverage_thresholds:
    lines: 80
    branches: 75
  create_pull_request_on_pass: true
  auto_merge: false           # PR の自動マージは無効（ユーザー確認推奨）

session:
  daily_limit_hours: 15       # 日次リミット
  weekly_limit_hours: 75      # 週次リミット（警告のみ）
  report_output_dir: "docs/"  # 作業日報の出力先
```

---

## ループ間の引き継ぎファイル

```
Monitor Loop
  ↓ .loop-monitor-report.md（状況・タスク優先度・コンテキスト）
Build Loop
  ↓ .loop-build-handoff.md（実装内容・コミット・変更ファイル）
Verify Loop
  ↓ .loop-verify-report.md（レビュー結果・テスト結果・マージ可否）
最終処理
  ↓ docs/[日付]_自律開発作業報告.md（2サイクル分のサマリー）
```

---

## CI 修復 AI の動作

Build Loop に内蔵された CI 修復 AI は、ビルド・テスト失敗時に自動で修復します：

```
ビルド失敗
  └─ エラー解析
  └─ 修正実装（試行 1/15）
  └─ ビルド再実行
    ├─ 成功 → Verify Loop へ
    └─ 失敗 → 試行 2/15 ...
        └─ 15回で解決不可 → TASKS.md に「要手動対応」記録
                          → 次のタスクに進む
```

---

## ログの確認

```bash
# セッション中のリアルタイムログ
# → Claude Code のターミナル出力を参照

# 引き継ぎファイルの確認
cat .loop-monitor-report.md
cat .loop-build-handoff.md
cat .loop-verify-report.md

# 作業日報の確認
ls docs/
cat "docs/$(date +%Y-%m-%d)_自律開発作業報告.md"

# Git ログで実装内容を確認
git log --oneline --since="15 hours ago"
```

---

## 週次運用スケジュール

```
【週5日（月〜金） — 推奨】
  月〜金: /loop 900m（15H）→ 2タスク/日 → 週10タスク
  土日: 休止（リミット回復）
  週消費: 75H

【週6日（月〜土） — 積極運用】
  月〜土: /loop 900m（15H）→ 2タスク/日 → 週12タスク
  日: 休止
  週消費: 90H ⚠️ 金・土はリミット残量確認後に開始
```

---

## 関連ドキュメント

- [起動ガイド](copilot-start-guide.md)
- [自律開発ワークフロー](autonomous-development-workflow.md)
- [Monitor Loop リファレンス](../loops/monitor-loop.md)
- [Build Loop リファレンス](../loops/build-loop.md)
- [Verify Loop リファレンス](../loops/verify-loop.md)
