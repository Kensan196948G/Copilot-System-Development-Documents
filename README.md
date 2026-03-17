# Copilot System Development Documents

CopilotCLI（Claude Code）による自律型ソフトウェア開発システムのドキュメント集。

Triple Loop アーキテクチャ、Agent Teams、Monitor/Build/Verify ループ、
AI 自動開発のベストプラクティスを体系的に整理。

---

## リポジトリ構造

```
.
├── README.md
├── CHANGELOG.md           ← 変更履歴（Keep a Changelog形式）★新規
├── QUICK-REFERENCE.md     ← Triple Loop チートシート（1ページ）★新規
├── .github/
│   ├── agents/            ← カスタムエージェント定義（7種）★拡充
│   └── workflows/         ← CI/CD ワークフロー（自動品質チェック）★新規
├── templates/             ← コピーしてすぐ使えるテンプレート集
├── architecture/          ← システム設計・アーキテクチャ（Mermaid図付き）
├── loops/                 ← 各ループの詳細リファレンス
├── operations/            ← 起動ガイド・コマンドリファレンス・運用ガイド
├── prompts/               ← AI プロンプトテンプレート（Monitor/Build/Verify）
├── best-practices/        ← 長時間自律セッションのベストプラクティス
├── examples/              ← 実セッションのウォークスルー
└── tasks/                 ← タスク別プロンプトテンプレート（15種類）
```

---

## クイックスタート

### 1. CLAUDE.md の配置

```bash
mkdir -p ~/.claude
# operations/00_フル自律開発起動(FullAutoStart).md の
# 「▼ここからコピー〜▲ここまでコピー」を ~/.claude/CLAUDE.md に貼り付け
```

### 2. TASKS.md の作成

```markdown
# Project Tasks
## Priority: High
- [ ] 実装したいタスクを書く
```

### 3. Claude Code 起動

```bash
cd /path/to/your/project
claude --dangerously-skip-permissions
```

### 4. /loop コマンドで自律開発開始

```
/loop 900m 以下の3重ループを2サイクル実行してください。（以下略 — operations/00_フル自律開発起動参照）
```

---

## ドキュメント一覧

### 🏗 アーキテクチャ（architecture/）

| ファイル | 内容 |
|---------|------|
| [triple-loop-architecture.md](architecture/triple-loop-architecture.md) | Triple Loop の全体設計・状態機械・Mermaid図 |
| [autonomous-development-architecture.md](architecture/autonomous-development-architecture.md) | 自律開発システム全体アーキテクチャ・State Manager JSON仕様・Feedback Bus |
| [agent-teams-system.md](architecture/agent-teams-system.md) | 8つの Agent チームの役割・メッセージプロトコル・Mermaid図 |
| [claudeos-loop-spec.md](architecture/claudeos-loop-spec.md) | ClaudeOS Auto-Mode Loop完全仕様・安定判定ロジック ★新規 |

### 🔄 ループリファレンス（loops/）

| ファイル | 内容 |
|---------|------|
| [monitor-loop.md](loops/monitor-loop.md) | Monitor Loop の実行フロー・コンテキスト収集 |
| [build-loop.md](loops/build-loop.md) | Build Loop の5段階ステップ・リトライロジック |
| [verify-loop.md](loops/verify-loop.md) | Verify Loop の品質ゲート・テスト・セキュリティ |

### ⚙️ 運用ガイド（operations/）

| ファイル | 内容 |
|---------|------|
| [00_フル自律開発起動(FullAutoStart).md](operations/00_フル自律開発起動(FullAutoStart).md) | 全プロンプト・/loop コマンド・起動手順（コピペ元） |
| [00_利用ガイド(UsageGuide).md](operations/00_利用ガイド(UsageGuide).md) | 2ファイル構成の説明・起動方法3種 |
| [copilot-start-guide.md](operations/copilot-start-guide.md) | 起動ガイド（CLAUDE.md 配置から放置まで） |
| [loop-command-usage.md](operations/loop-command-usage.md) | /loop コマンド完全リファレンス |
| [autonomous-development-workflow.md](operations/autonomous-development-workflow.md) | タスク準備 → Monitor → Build → Verify → 最終処理 |
| [cost-optimization-guide.md](operations/cost-optimization-guide.md) | プレミアムリクエスト管理・コスト最適化パターン |
| [team-onboarding-guide.md](operations/team-onboarding-guide.md) | 30分クイックスタート・組織設定・チーム運用フロー |
| [troubleshooting-guide.md](operations/troubleshooting-guide.md) | 17エラーパターン・診断コマンド集・ループ再起動手順 ★新規 |

### 💬 プロンプトテンプレート（prompts/）

| ファイル | 内容 |
|---------|------|
| [monitor-loop-prompts.md](prompts/monitor-loop-prompts.md) | コンテキスト分析・タスク分解・ヘルスチェック |
| [build-loop-prompts.md](prompts/build-loop-prompts.md) | コード生成・リファクタ・バグ修正・ビルドエラー修正 ★新規 |
| [verify-loop-prompts.md](prompts/verify-loop-prompts.md) | テスト分析・カバレッジ・セキュリティ・PR サマリー |

### ✅ ベストプラクティス（best-practices/）

| ファイル | 内容 |
|---------|------|
| [autonomous-session-best-practices.md](best-practices/autonomous-session-best-practices.md) | タスク定義・コンテキスト管理・週次運用スケジュール |
| [copilot-capability-summary.md](best-practices/copilot-capability-summary.md) | Triple Loop 15H 実現可能性サマリー・自律型DevOps |
| [copilot-maximum-capabilities.md](best-practices/copilot-maximum-capabilities.md) | Copilot CLI 最大能力の詳細分析・ロードマップ |
| [agent-quality-framework.md](best-practices/agent-quality-framework.md) | エージェント出力品質評価スコアカード・自動チェック ★新規 |

### 📘 実例（examples/）

| ファイル | 内容 |
|---------|------|
| [example-monitor-session.md](examples/example-monitor-session.md) | Monitor Loop の実行例 |
| [example-end-to-end-workflow.md](examples/example-end-to-end-workflow.md) | 5タスクの一夜セッション全記録 |
| [example-fleet-execution.md](examples/example-fleet-execution.md) | /fleet 並列実行ウォークスルー（実ログ付き） |
| [example-failure-recovery.md](examples/example-failure-recovery.md) | 失敗時のリカバリー手順集（7パターン） |
| [real-world-project-example.md](examples/real-world-project-example.md) | SaaS プロジェクト実践例・コスト実績・トラブル事例 ★新規 |

### 📁 テンプレート（templates/）★新規

| ファイル | コピー先 | 内容 |
|---------|---------|------|
| [AGENTS.md](templates/AGENTS.md) | プロジェクトルート | Copilot CLIへのプロジェクト規約指示 |
| [TASKS.md](templates/TASKS.md) | プロジェクトルート | タスク管理・優先度キュー |
| [CLAUDE.md](templates/CLAUDE.md) | プロジェクトルート | Claude/Copilot起動時の詳細設定 |

### 🤖 カスタムエージェント定義（.github/agents/）

| ファイル | 役割 | 推奨モデル |
|---------|------|-----------|
| [backend-agent.agent.md](.github/agents/backend-agent.agent.md) | API・DB・認証実装 | Claude Opus 4.6 |
| [frontend-agent.agent.md](.github/agents/frontend-agent.agent.md) | UI・UX実装 | Claude Sonnet 4.6 |
| [test-writer.agent.md](.github/agents/test-writer.agent.md) | テスト設計・生成 | GPT-5.3-Codex |
| [security-agent.agent.md](.github/agents/security-agent.agent.md) | セキュリティレビュー | Claude Opus 4.6 |
| [docs-agent.agent.md](.github/agents/docs-agent.agent.md) | ドキュメント生成 | Claude Sonnet 4.6 |
| [devops-agent.agent.md](.github/agents/devops-agent.agent.md) | インフラ・CI/CD・クラウドデプロイ ★新規 | Claude Opus 4.6 |
| [qa-automation-agent.agent.md](.github/agents/qa-automation-agent.agent.md) | E2E・パフォーマンステスト ★新規 | GPT-5.3-Codex |

### 📋 タスクプロンプト（tasks/）

Claude Code に渡すタスク別プロンプトテンプレート。
`[PROJECT_NAME]` などのプレースホルダを書き換えて使用。

| # | ファイル | 用途 |
|---|---------|------|
| 01 | [新規プロジェクト初期化](tasks/01_新規プロジェクト初期化(NewProjectInit).md) | プロジェクトのスケルトン構築 |
| 02 | [バグ修正](tasks/02_バグ修正(BugFix).md) | バグ調査・修正・テスト追加 |
| 03 | [コードレビュー](tasks/03_コードレビュー(CodeReview).md) | PR・コードの多角的レビュー |
| 04 | [リファクタリング](tasks/04_リファクタリング(Refactoring).md) | 技術的負債の段階的解消 |
| 05 | [テスト自動化](tasks/05_テスト自動化(TestAutomation).md) | テスト生成・カバレッジ向上 |
| 06 | [CI/CD 構築](tasks/06_CI_CD構築(CICDSetup).md) | GitHub Actions パイプライン構築 |
| 07 | [セキュリティ診断](tasks/07_セキュリティ診断(SecurityAudit).md) | 脆弱性スキャン・修正 |
| 08 | [ドキュメント生成](tasks/08_ドキュメント生成(DocGeneration).md) | README・API 仕様書・Mermaid 図 |
| 09 | [パフォーマンス最適化](tasks/09_パフォーマンス最適化(PerformanceOpt).md) | ボトルネック特定・最適化 |
| 10 | [API サーバー構築](tasks/10_APIサーバー構築(APIServerBuild).md) | REST API 設計・実装・Docker |
| 11 | [フロントエンド開発](tasks/11_フロントエンド開発(FrontendDev).md) | UI コンポーネント・アクセシビリティ |
| 12 | [データベース設計](tasks/12_データベース設計(DatabaseDesign).md) | ER 図・マイグレーション・インデックス |
| 13 | [レガシーコード移行](tasks/13_レガシーコード移行(LegacyMigration).md) | Strangler Fig パターンでの移行 |
| 14 | [依存関係更新](tasks/14_依存関係更新(DependencyUpdate).md) | 安全な依存関係の最新化 |
| 15 | [インシデント対応](tasks/15_インシデント対応(IncidentResponse).md) | 本番障害の緊急対応・ポストモーテム |

---

## 運用モード

| モード | コマンド | 稼働時間 | 実装タスク数/日 |
|-------|---------|---------|--------------|
| 2サイクル15H（推奨） | `/loop 900m` | 15時間 | 2タスク |
| 1サイクル8H | `/loop 450m` | 8時間 | 1タスク |
| Monitor のみ | `/loop 30m` | 30分 | 0（調査のみ） |
| 単発タスク | `tasks/` テンプレート使用 | 任意 | 任意 |

---

## CI/CD

| ワークフロー | トリガー | 内容 |
|------------|---------|------|
| [ci.yml](.github/workflows/ci.yml) | push/PR | Markdown lint・リンクチェック・統計 |
| [weekly-improvement.yml](.github/workflows/weekly-improvement.yml) | 毎週日曜0時UTC | 週次統計レポートIssue自動生成 |

---

## 変更履歴

詳細は [CHANGELOG.md](CHANGELOG.md) を参照してください。

---

## ライセンス

MIT License
