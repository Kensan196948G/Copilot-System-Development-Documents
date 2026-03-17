# Changelog

このリポジトリの全変更履歴を記録します。  
フォーマットは [Keep a Changelog](https://keepachangelog.com/ja/1.0.0/) に準拠し、  
バージョニングは [Semantic Versioning](https://semver.org/lang/ja/) を採用しています。

> **凡例**  
> - `Added` — 新機能・新ファイルの追加  
> - `Changed` — 既存機能・既存ファイルの変更  
> - `Deprecated` — 近い将来削除予定の機能  
> - `Removed` — 削除された機能・ファイル  
> - `Fixed` — バグ修正  
> - `Security` — セキュリティ関連の変更

---

## [Unreleased]

---

## [1.3.0] - 2026-03-17

### Added
- `QUICK-REFERENCE.md` — Triple Loop チートシート（334行）
- `CHANGELOG.md` — Keep a Changelog形式の変更履歴（v1.0〜v1.3）
- `prompts/build-loop-prompts.md` — Build Loop用プロンプト5種（635行）
- `templates/CLAUDE.md` — ~/.claude/CLAUDE.md統合設定（617行）
- `.github/workflows/ci.yml` — Markdown品質CI（markdownlint・リンクチェック・統計）
- `.github/workflows/weekly-improvement.yml` — 週次統計レポート自動生成
- `.markdownlint.json` / `.mlc-config.json` — CI設定ファイル
- `operations/troubleshooting-guide.md` — 17エラーパターン診断ガイド（1,078行）
- `architecture/claudeos-loop-spec.md` — ClaudeOS Loop完全仕様書（1,062行）
- `best-practices/agent-quality-framework.md` — 品質評価スコアカード（847行）
- `examples/real-world-project-example.md` — SaaS実践例・コスト実績（728行）
- `.github/agents/devops-agent.agent.md` — DevOps専門エージェント定義
- `.github/agents/qa-automation-agent.agent.md` — QA自動化専門エージェント定義

### Changed
- `README.md` — 新規ファイル反映・リポジトリ構造図更新・CI/CD表追加
- `architecture/triple-loop-architecture.md` — Mermaid図3種追加（状態遷移・シーケンス・ガント）
- `architecture/agent-teams-system.md` — Mermaid図3種追加（グラフ・シーケンス・マップ）
- `architecture/autonomous-development-architecture.md` — State Manager JSON Schema・Feedback Bus Protocol・Mermaidシーケンス図追加
- `.github/agents/devops-agent.agent.md` — terraform権限をwildcardから明示的権限へ修正（セキュリティ強化）

### Fixed
- `.github/workflows/ci.yml` — サードパーティAction SHA ピン適用（サプライチェーンセキュリティ）
- `.github/workflows/weekly-improvement.yml` — Actions Expression を環境変数経由に変更（インジェクション対策）
- `CHANGELOG.md` — プレースホルダーURLを実際のリポジトリURLに修正

---

## [1.2.0] - 2026-03-17

> **コミット**: `c1d7bbc` (セッションサマリーをマージ), `f94a755` (次の5ステップを全実装)  
> **概要**: ドキュメントを「読むだけ」から「すぐ使える」実用ツールキットへ進化。  
> カスタムエージェント定義・コピペテンプレート・運用ガイド強化・実例拡充を一括実装。

### Added

#### セッションサマリー（コミット `c1d7bbc`）
- **`SESSION-SUMMARY.md`** — 2026-03-17 セッションの全作業記録を追加
  - Phase 1〜3 の作業詳細（背景・実施内容・成果物）
  - 最終リポジトリ構造のツリー図
  - 次のステップ（今後の発展方向）5項目
  - コミット数・変更行数のサマリー（5コミット、40ファイル以上、+6,400行以上）

#### カスタムエージェント定義（コミット `f94a755` — Step 1）
- **`.github/agents/backend-agent.agent.md`** — バックエンド専任エージェント定義
  - 推奨モデル: Claude Opus 4.6
  - 担当領域: REST API設計・実装、データベース設計・マイグレーション、JWT/OAuth2認証実装
  - 自動実行ルール: `src/api/**`, `src/db/**`, `migrations/**` の変更に反応
  - 品質ゲート: OpenAPI仕様自動生成、DBスキーマ整合性チェック、認証フロー検証
- **`.github/agents/frontend-agent.agent.md`** — フロントエンド専任エージェント定義
  - 推奨モデル: Claude Sonnet 4.6
  - 担当領域: Reactコンポーネント実装、UIアニメーション、WCAG 2.1 AA準拠アクセシビリティ
  - 自動実行ルール: `src/components/**`, `src/pages/**`, `src/styles/**` の変更に反応
  - 品質ゲート: Storybook自動生成、Lighthouse スコア90+維持、axe-core アクセシビリティ検証
- **`.github/agents/test-writer.agent.md`** — テスト設計・生成専任エージェント定義
  - 推奨モデル: GPT-5.3-Codex
  - 担当領域: ユニットテスト・統合テスト・E2Eテスト生成、カバレッジ分析
  - 自動実行ルール: 新規実装ファイルの追加・変更に反応して対応テスト自動生成
  - 品質ゲート: カバレッジ80%以上維持、境界値テスト必須、スナップショットテスト管理
- **`.github/agents/security-agent.agent.md`** — セキュリティレビュー専任エージェント定義
  - 推奨モデル: Claude Opus 4.6
  - 担当領域: OWASP Top 10 診断、依存関係脆弱性スキャン、シークレット漏洩検出
  - 動作モード: レビュー専任（コード自動変更は行わない安全設計）
  - 品質ゲート: CVSSスコア7.0以上は必ずブロック、Secretlint通過必須
- **`.github/agents/docs-agent.agent.md`** — ドキュメント生成専任エージェント定義
  - 推奨モデル: Claude Sonnet 4.6
  - 担当領域: README自動更新、OpenAPI仕様生成、Mermaidアーキテクチャ図生成、JSDoc/docstring補完
  - 自動実行ルール: API仕様変更・新機能追加時に関連ドキュメント自動更新
  - 品質ゲート: 全公開API仕様の記述率100%、リンク切れチェック

#### コピペ即使用テンプレート（コミット `f94a755` — Step 2）
- **`templates/AGENTS.md`** — プロジェクトルートへのコピー用エージェント規約テンプレート
  - Triple Loop アーキテクチャ設定（Monitor/Build/Verify 各ループの設定値）
  - プロジェクト固有ルール記述欄（コーディング規約・禁止操作・必須レビュー項目）
  - エージェント間メッセージフォーマット定義
  - 停止条件・エスカレーション条件の設定
- **`templates/TASKS.md`** — プロジェクトルートへのコピー用タスク管理テンプレート
  - 優先度別キュー構造（High / Medium / Low）
  - タスク状態管理（Pending / InProgress / Done / Blocked）
  - 各タスクの記述フォーマット（目的・受け入れ条件・関連ファイル）
  - Blocked タスクの依存関係記述欄
- **`templates/CLAUDE.md`** — プロジェクトルートへのコピー用 Claude 設定テンプレート
  - 自律実行ルール設定（許可コマンド・禁止コマンド）
  - 停止条件の詳細設定（本番環境操作・機密情報処理の自動停止）
  - ループサイクル設定（Monitor 30分 / Build 240分 / Verify 90分）
  - コンテキスト管理設定（圧縮タイミング・保存先）
- **`templates/README.md`** — テンプレートディレクトリの使い方説明ファイル
  - 各テンプレートのコピー先・用途一覧
  - カスタマイズが必要なプレースホルダー一覧
  - よくある設定パターン（個人開発・チーム開発・OSS向け）

#### 実例の充実（コミット `f94a755` — Step 3）
- **`examples/example-fleet-execution.md`** — `/fleet` 並列実行ウォークスルー
  - 実ログ形式での /fleet コマンド使用例（バックエンド + フロントエンド同時実装）
  - エージェント間の依存関係解決フロー
  - 並列実行時のコンフリクト回避パターン
  - 所要時間・コスト内訳の実測データ
- **`examples/example-failure-recovery.md`** — 失敗時のリカバリー手順集
  - 7パターンのリカバリー手順を網羅
    1. コンテキストウィンドウ溢れ（Context Overflow）からの復旧
    2. 無限ループ（Build Loop 停止不能）への対処
    3. テスト失敗連鎖（Verify Loop ブロック）の解消
    4. 依存関係競合（package-lock.json 破損）の修復
    5. 誤ったファイル削除からの Git 復元
    6. APIレート制限によるセッション中断からの再開
    7. 環境変数未設定による実行時エラーの診断・修正

#### 運用ガイドの強化（コミット `f94a755` — Step 4, 5）
- **`operations/cost-optimization-guide.md`** — プレミアムリクエスト管理・コスト最適化ガイド
  - モデル別コスト倍率表（Claude Opus / Sonnet / Haiku / GPT-5.3-Codex 比較）
  - Triple Loop 15H の消費量シミュレーション（タスク規模別3パターン）
  - 5つのコスト削減パターン
    1. Monitor Loop に Haiku を使用する「軽量Monitor戦略」
    2. キャッシュ活用による重複リクエスト削減
    3. Verify Loop の選択的実行（変更範囲連動スコープ最適化）
    4. /fleet よりも /delegate を使う段階的並列化
    5. 月次予算アラート設定と自動停止ルールの組み合わせ
  - プレミアムリクエスト上限の組織別管理ガイド
- **`operations/team-onboarding-guide.md`** — チームオンボーディング完全ガイド
  - 30分クイックスタート（個人開発者 → チーム移行チェックリスト）
  - 組織共通エージェント設定（`.github-private/agents/` によるプライベート管理）
  - GitHub Actions 週次自動化統合例（cronによる夜間Triple Loop実行）
  - 新メンバー向け Day 1 / Week 1 プログラム
    - Day 1: 環境構築・初回Monitor Loop実行・ドキュメント読了
    - Week 1: 既存タスクへの参加・カスタムエージェントカスタマイズ・初回15Hセッション実施

### Changed
- **`README.md`** — ドキュメント一覧テーブルを最新化
  - `templates/` ディレクトリを新規セクションとして追加（★新規マーク付き）
  - `.github/agents/` カスタムエージェント一覧セクションを追加（推奨モデル列含む）
  - `operations/` テーブルに `cost-optimization-guide.md` / `team-onboarding-guide.md` を追加
  - `examples/` テーブルに `example-fleet-execution.md` / `example-failure-recovery.md` を追加

### Fixed
- なし（本バージョンは新機能追加のみ）

---

## [1.1.0] - 2026-03-17

> **コミット**: `6d6bfc4` (Triple Loop 15H & 自律型DevOps 能力サマリーを追加),  
>              `1f1e51f` (Copilot CLI最大能力分析・Triple Loop 15H実現可能性レポートを追加)  
> **概要**: GitHub Copilot CLI 最新版（2026年2月 GA）の能力を公式ドキュメントで徹底調査・分析。  
> Triple Loop 15H の完全実現可能性を実証し、自律型 DevOps の実装ロードマップを策定。

### Added

#### Copilot CLI 最大能力分析（コミット `1f1e51f`）
- **`best-practices/copilot-maximum-capabilities.md`** — Copilot CLI 最大能力の詳細分析レポート（503行）
  - **Chapter 1: コマンド能力の全体マップ**
    - `/loop` — タイムボックス制御ループ（最大999分対応・入れ子サポート）
    - `/fleet` — 複数エージェント並列実行（最大8エージェント同時稼働）
    - `/delegate` — サブタスク委任チェーン（最大3段階ネスト）
    - `/autopilot` — 完全自律モード（人間承認なし実行・条件付き有効化）
    - MCP（Model Context Protocol）— 外部ツール統合プロトコル
  - **Chapter 2: AI Agent Teams の構成パターン**
    - 役割分担モデル（Planner / Executor / Reviewer / Reporter）
    - エージェント間通信プロトコル（JSON メッセージフォーマット）
    - 失敗伝播の防止設計（エラー分離・独立リトライ）
  - **Chapter 3: 各ループの技術的上限**
    - Monitor Loop: コンテキスト収集の最大スコープ・圧縮率
    - Build Loop: 1セッションあたりの最大実装行数（実測 2,000〜5,000行）
    - Verify Loop: 自動テスト実行・PR作成・マージまでの完全自動化
  - **Chapter 4: MCP 統合による外部サービス連携**
    - GitHub API 統合（Issue/PR/Milestone の自動操作）
    - Slack MCP（進捗通知・承認フロー）
    - AWS/GCP MCP（インフラ操作・デプロイ自動化）
    - Jira/Linear MCP（タスク管理システム連携）
  - **Chapter 5: 2026年ロードマップ**
    - Q2 2026: マルチリポジトリ横断エージェント
    - Q3 2026: 自然言語 → インフラ設計自動化
    - Q4 2026: 組織レベルの自律開発オーケストレーション

#### Triple Loop 15H 実現可能性サマリー（コミット `6d6bfc4`）
- **`best-practices/copilot-capability-summary.md`** — Triple Loop 15H 対応表・自律型 DevOps 能力サマリー
  - **Triple Loop 15H 実現可能性マトリクス**

    | 能力カテゴリ | 実現可能性 | 根拠 |
    |------------|-----------|------|
    | Triple Loop 15H（Monitor/Build/Verify） | ✅ 完全実現可能 | `/loop 900m` + CLAUDE.md 設定で実証済み |
    | AI Agent Teams（/fleet + カスタムエージェント） | ✅ 完全実現可能 | `.github/agents/` + `/fleet` コマンドで実証済み |
    | 自律型 DevOps（Autopilot + /delegate + MCP） | ✅ 完全実現可能 | `/autopilot` + MCP設定で実証済み |
    | ゼロ人間介入 8H セッション | ✅ 完全実現可能 | 停止条件設定 + エラー自動リカバリーで実証済み |

  - **完全自律開発の究極形（理論上の上限）**
    - 24時間連続稼働: Monitor(2h) → Build(6h) → Verify(2h) × 3サイクル
    - 複数プロジェクト並列管理: /fleet で最大8プロジェクト同時進行
    - コスト: 月20万円〜50万円（Opus使用比率による）の試算
  - **自律型 DevOps パイプライン全体図（Mermaid）**
    - コードコミット → Monitor Loop → Build Loop → Verify Loop → 自動マージ → デプロイの全自動フロー
  - **組織別推奨設定プロファイル**
    - スタートアップ向け: コスト優先（Sonnet中心・Opus最小化）
    - エンタープライズ向け: 品質優先（Opus中心・全ゲート有効）
    - OSS向け: コミュニティ参加型（レビューゲート有効・Autopilot制限）

### Changed
- **`README.md`** — `best-practices/` セクションに2ファイルを追加
  - `copilot-maximum-capabilities.md` の説明を追加（503行・詳細能力分析・ロードマップ）
  - `copilot-capability-summary.md` の説明を追加（Triple Loop 15H対応表・完全自律開発究極形）

### Fixed
- なし（本バージョンは新機能追加のみ）

---

## [1.0.0] - 2026-03-17

> **コミット**: `2eb8c69` (ルートドキュメントを参考に全ドキュメントを再作成・再構成),  
>              `c1caffb` (Merge initial documentation structure),  
>              `664f6a4` (Add complete documentation structure for Copilot autonomous development system)  
> **概要**: ルートフォルダに散在していた日本語ドキュメント17件を参考に全ファイルを再作成・再構成。  
> タスクプロンプト15種・ループリファレンス・運用ガイド・アーキテクチャ設計など、  
> 自律型 Copilot 開発システムのドキュメント体系を一から整備。

### Added

#### プロジェクト起動・初期構造（コミット `664f6a4`, `c1caffb`）
- **`README.md`** — プロジェクト概要・全ドキュメントナビゲーション
  - 1行説明: "CopilotCLI（Claude Code）による自律型ソフトウェア開発システムのドキュメント集"
  - クイックスタート（CLAUDE.md 配置 → TASKS.md 作成 → claude 起動 → /loop 実行の4ステップ）
  - 7フォルダ全体のナビゲーションテーブル（ファイル名・内容説明付き）
  - 運用モード比較表（2サイクル15H / 1サイクル8H / Monitor のみ / 単発タスク）

#### アーキテクチャドキュメント（コミット `664f6a4`）
- **`architecture/triple-loop-architecture.md`** — Triple Loop の全体設計
  - Triple Loop の概念図（Mermaid: Monitor → Build → Verify → サイクル繰り返し）
  - 各ループの責務・入力・出力・時間配分の定義
  - ループ間連携プロトコル（前ループの出力が次ループの入力になる設計）
  - 状態機械の定義（IDLE / MONITORING / BUILDING / VERIFYING / COMPLETED / FAILED）
  - 15時間サイクルのタイムライン（Monitor 2h → Build 6h → Verify 2h × 2サイクル）
  - コンテキスト保持戦略（ループをまたぐ情報の構造化・圧縮・引き継ぎ）
- **`architecture/autonomous-development-architecture.md`** — 自律開発システム全体アーキテクチャ
  - システム全体構成図（Mermaid: CLAUDE.md → CopilotCLI → Triple Loop → Git/CI/CD）
  - コンポーネント一覧と責務（CLAUDE.md / TASKS.md / Agent Teams / MCP Servers）
  - ループ実行エンジンの内部設計
  - 外部システム連携ポイント（GitHub / Slack / AWS / Jira）
  - 障害耐性設計（エラー分類・リトライ戦略・エスカレーション条件）
  - セキュリティ境界（許可操作 / 要確認操作 / 絶対禁止操作の3層定義）
- **`architecture/agent-teams-system.md`** — Agent Teams の設計・メッセージプロトコル
  - 8つの Agent チームの役割定義
    - PlannerAgent: タスク分解・依存関係グラフ生成
    - BackendAgent: API・DB・認証実装
    - FrontendAgent: UI・コンポーネント実装
    - TestAgent: テスト生成・カバレッジ維持
    - SecurityAgent: 脆弱性診断・セキュリティゲート
    - DocsAgent: ドキュメント生成・API仕様更新
    - ReviewAgent: コードレビュー・PR承認
    - ReporterAgent: 進捗サマリー・ステークホルダー通知
  - エージェント間メッセージフォーマット（JSON Schema 定義）
  - タスク委任チェーン（PlannerAgent → 各専門 Agent → ReviewAgent の標準フロー）
  - 並列実行時の競合解決プロトコル

#### ループリファレンス（コミット `664f6a4`）
- **`loops/monitor-loop.md`** — Monitor Loop の詳細リファレンス
  - 実行フロー（コンテキスト収集 → タスク分析 → 優先度整理 → Build への引き継ぎ）
  - コンテキスト収集スコープ（Git履歴・テスト結果・CI/CDステータス・依存関係）
  - タスク分解アルゴリズム（依存関係グラフ・並列化可能タスクの特定）
  - ヘルスチェック項目（テストカバレッジ・技術的負債・セキュリティスキャン）
  - 出力フォーマット（Build Loop への引き継ぎ JSON 仕様）
  - 推奨実行時間: 30分（コンテキスト収集20分 + 分析・出力10分）
- **`loops/build-loop.md`** — Build Loop の詳細リファレンス
  - 5段階ステップの定義
    1. タスクキューの読み込み・実行順決定
    2. コード生成・実装（TDD アプローチ推奨）
    3. ユニットテスト自動追加
    4. リントチェック・自動修正
    5. コミット・プッシュ・PR ドラフト作成
  - リトライロジック（最大3回・バックオフ戦略・失敗パターン分類）
  - コンテキスト管理（180分超でのコンテキスト圧縮タイミング）
  - タスク完了判定基準（受け入れ条件チェックリスト）
  - 推奨実行時間: 240分（通常タスク1件）〜 360分（複雑タスク1件）
- **`loops/verify-loop.md`** — Verify Loop の詳細リファレンス
  - 品質ゲート一覧（5段階: コード品質 → テスト → セキュリティ → パフォーマンス → ドキュメント）
  - テスト実行戦略（ユニット → 統合 → E2E → ビジュアルリグレッション）
  - セキュリティスキャン手順（Secretlint → npm audit → SAST → DAST）
  - PR 作成テンプレート（変更サマリー・テスト結果・スクリーンショット）
  - 品質ゲート不合格時のフィードバックループ（Build Loopへの差し戻し）
  - 推奨実行時間: 90分（検証60分 + PR作成・レビュー準備30分）

#### 運用ガイド（コミット `2eb8c69`）
- **`operations/00_フル自律開発起動(FullAutoStart).md`** — 全プロンプト・コピペ元マスターファイル
  - CLAUDE.md に貼り付ける完全プロンプト（▼コピー開始〜▲コピー終了 マーカー付き）
  - `/loop` コマンドの完全実行形（2サイクル15H版・1サイクル8H版）
  - 自律実行ルール・停止条件の完全版
  - Monitor / Build / Verify 各ループの詳細プロンプト
- **`operations/00_利用ガイド(UsageGuide).md`** — 2ファイル構成の説明・起動方法3種
  - CLAUDE.md との組み合わせ使い方
  - 起動方法3種（フル自律 / 半自律 / 単発タスク）の使い分けガイド
  - よくある設定ミスと対処法
- **`operations/copilot-start-guide.md`** — CLAUDE.md 配置から放置までの起動ガイド
  - Step-by-Step セットアップ手順（所要時間付き）
  - CLAUDE.md の最小構成と推奨構成の違い
  - `--dangerously-skip-permissions` フラグの用途と注意事項
  - セッション開始後の確認ポイント（正常起動の兆候・異常の兆候）
- **`operations/loop-command-usage.md`** — `/loop` コマンド完全リファレンス
  - コマンド構文: `/loop [時間]m [指示文]`
  - 時間指定パターン（15m / 30m / 90m / 240m / 450m / 900m）
  - 入れ子ループの設定方法（外ループ ÷ 内ループ のタイムボックス分割）
  - /loop と /fleet の使い分け判断基準
  - よくあるエラーと対処法（タイムアウト / コンテキスト溢れ / タスクキュー空）
- **`operations/autonomous-development-workflow.md`** — タスク準備から最終処理までの全ワークフロー
  - タスク準備フェーズ（TASKS.md 記述ガイド・受け入れ条件の書き方）
  - Monitor Loop フェーズ（実行開始 → 出力確認 → Build 引き継ぎ）
  - Build Loop フェーズ（タスク実行 → 中間チェックポイント → 完了確認）
  - Verify Loop フェーズ（品質ゲート → PR作成 → レビュー依頼）
  - 最終処理フェーズ（セッションサマリー → 次サイクル準備 → コスト確認）

#### プロンプトテンプレート（コミット `2eb8c69`）
- **`prompts/monitor-loop-prompts.md`** — Monitor Loop 用プロンプトテンプレート集
  - コンテキスト分析プロンプト（JSON出力形式・スコープ指定可能）
  - タスク分解プロンプト（依存関係グラフ付き・並列化候補特定）
  - ヘルスチェックプロンプト（テストカバレッジ・リント・セキュリティ統合）
  - 優先度整理プロンプト（MoSCoW法・リスク評価・コスト見積もり）
  - Build 引き継ぎプロンプト（構造化ハンドオフ・コンテキスト圧縮）
- **`prompts/verify-loop-prompts.md`** — Verify Loop 用プロンプトテンプレート集
  - テスト分析プロンプト（失敗テストのパターン分類・修正方針生成）
  - カバレッジ向上プロンプト（未カバー関数の特定・テスト追加指示）
  - セキュリティスキャンプロンプト（OWASP Top 10 チェックリスト連動）
  - PR サマリー生成プロンプト（変更内容・影響範囲・レビューポイント）
  - 品質レポートプロンプト（ステークホルダー向け非技術者向けサマリー）

#### ベストプラクティス（コミット `2eb8c69`）
- **`best-practices/autonomous-session-best-practices.md`** — 長時間自律セッションのベストプラクティス
  - タスク定義のベストプラクティス
    - 良い例 / 悪い例の対比（具体的コード例付き）
    - 受け入れ条件の書き方（GIVEN-WHEN-THEN 形式）
    - タスク粒度の基準（1 Build Loop = 1タスクが原則）
  - コンテキスト管理のベストプラクティス
    - 180分でのコンテキスト強制圧縮手順
    - 重要情報のピン留め方法
    - セッション間の引き継ぎ文書フォーマット
  - 週次運用スケジュール例
    - 月曜: Monitor Loop 単独実行（週次コンテキスト収集）
    - 火〜木: 2サイクル15H本番稼働
    - 金曜: Verify Loop + PR レビュー + セッションサマリー
  - アンチパターン集（7パターン）
    1. タスクを曖昧に書く（「APIを改善する」など）
    2. 停止条件を設定しない
    3. コンテキストをリセットせずにセッションを続ける
    4. Opus を全タスクに使う（コスト爆発）
    5. Verify Loop をスキップして直接マージする
    6. TASKS.md を更新せずにタスクを追加する
    7. 並列実行で同一ファイルを複数エージェントが編集する

#### 実例集（コミット `2eb8c69`）
- **`examples/example-monitor-session.md`** — Monitor Loop の実行例
  - 実際のコマンド入出力ログ形式
  - コンテキスト収集〜タスク優先度整理〜Build 引き継ぎの全ステップ
  - 典型的な Monitor 出力 JSON の解説
- **`examples/example-end-to-end-workflow.md`** — 5タスクの一夜セッション全記録
  - 21:00 セッション開始 〜 翌09:00 完了までのタイムライン
  - Monitor Loop の実行ログ（タスク発見・分解・優先度付け）
  - Build Loop 2サイクルの実装ログ（コード生成・テスト追加・PR作成）
  - Verify Loop の品質チェックログ（テスト結果・セキュリティスキャン）
  - コスト内訳: Opus 3件 + Sonnet 2件 = 合計約 $42

#### タスクプロンプト（コミット `2eb8c69`）
- **`tasks/README.md`** — タスクプロンプト一覧・使い方ガイド
  - 15種類のプロンプトの用途マトリクス
  - プレースホルダー一覧（`[PROJECT_NAME]` / `[LANGUAGE]` / `[FRAMEWORK]` 等）
  - TASKS.md への組み込み方法
- **`tasks/01_新規プロジェクト初期化(NewProjectInit).md`** — プロジェクトスケルトン構築
  - ディレクトリ構造の自動生成
  - 依存関係の初期インストール（package.json / requirements.txt / go.mod 対応）
  - CI/CD 初期設定（GitHub Actions ワークフロー基本形）
  - README / CHANGELOG / CONTRIBUTING 自動生成
- **`tasks/02_バグ修正(BugFix).md`** — バグ調査・修正・テスト追加
  - 再現手順の特定・最小再現コードの生成
  - 根本原因分析（スタックトレース読み込み・関連コード検索）
  - 修正コードの生成・既存テストへの非デグレード確認
  - 修正を証明するリグレッションテストの追加
- **`tasks/03_コードレビュー(CodeReview).md`** — PR・コードの多角的レビュー
  - 可読性・保守性・パフォーマンス・セキュリティの4軸評価
  - レビューコメントの自動生成（GitHub PR コメント形式）
  - 修正提案の優先度付け（Must / Should / Could）
- **`tasks/04_リファクタリング(Refactoring).md`** — 技術的負債の段階的解消
  - Strangler Fig パターン・Extract Method・依存性逆転の適用
  - リファクタリング前後の品質指標比較（循環的複雑度・結合度）
  - テストを壊さない安全なリファクタリング手順
- **`tasks/05_テスト自動化(TestAutomation).md`** — テスト生成・カバレッジ向上
  - カバレッジレポートの分析・未テスト関数の特定
  - ユニット / 統合 / E2E テストの3層自動生成
  - テストデータ管理（ファクトリーパターン・フィクスチャ設計）
- **`tasks/06_CI_CD構築(CICDSetup).md`** — GitHub Actions パイプライン構築
  - ブランチ戦略に合わせたワークフロー設計（trunk-based / GitFlow 対応）
  - キャッシュ戦略（依存関係・Docker レイヤー・ビルド成果物）
  - 並列ジョブ設計（lint / test / build / security scan の並列化）
  - デプロイ環境（staging / production）の分離設定
- **`tasks/07_セキュリティ診断(SecurityAudit).md`** — 脆弱性スキャン・修正
  - OWASP Top 10 チェックリスト自動診断
  - 依存関係の CVE スキャン（npm audit / Safety / govulncheck）
  - Secretlint によるシークレット漏洩検出
  - 修正優先度マトリクス（CVSS スコア × 露出度）
- **`tasks/08_ドキュメント生成(DocGeneration).md`** — README・API仕様書・Mermaid図
  - README 自動生成（クイックスタート・API リファレンス・アーキテクチャ図）
  - OpenAPI 3.1 仕様書の自動生成（コードベースから逆引き）
  - Mermaid アーキテクチャ図・ER 図・シーケンス図の自動生成
  - JSDoc / docstring の補完（公開 API 全件）
- **`tasks/09_パフォーマンス最適化(PerformanceOpt).md`** — ボトルネック特定・最適化
  - プロファイリングレポートの分析・ホットスポット特定
  - N+1 クエリ検出・インデックス最適化提案
  - フロントエンド Core Web Vitals の改善（LCP / FID / CLS）
  - キャッシュ戦略の設計（Redis / CDN / Service Worker）
- **`tasks/10_APIサーバー構築(APIServerBuild).md`** — REST API 設計・実装・Docker
  - OpenAPI ファースト設計（仕様書 → 実装の順序）
  - 認証・認可の実装（JWT / OAuth2 / API Key の選択基準）
  - Docker / docker-compose による開発環境整備
  - API バージョニング戦略（URL / Header / Content-Type 方式比較）
- **`tasks/11_フロントエンド開発(FrontendDev).md`** — UI コンポーネント・アクセシビリティ
  - Atomic Design に基づくコンポーネント設計
  - WCAG 2.1 AA 準拠アクセシビリティ実装
  - Storybook によるコンポーネントカタログ構築
  - パフォーマンスバジェット設定（Bundle Size / Lighthouse Score）
- **`tasks/12_データベース設計(DatabaseDesign).md`** — ER 図・マイグレーション・インデックス
  - Mermaid ER 図の自動生成・正規化チェック
  - マイグレーションファイルの自動生成（rollback 付き）
  - インデックス設計（クエリパターン分析ベース）
  - パーティショニング戦略（大規模テーブル対応）
- **`tasks/13_レガシーコード移行(LegacyMigration).md`** — Strangler Fig パターンでの移行
  - レガシーコードの依存関係マッピング
  - 段階的移行計画の自動生成（リスク評価・工数見積もり付き）
  - 移行中の並行稼働戦略（Feature Flag / Anti-corruption Layer）
  - 移行完了の定義・検証手順
- **`tasks/14_依存関係更新(DependencyUpdate).md`** — 安全な依存関係の最新化
  - 依存関係ツリーの分析・破壊的変更の特定
  - 段階的アップデート計画（パッチ → マイナー → メジャーの順序）
  - 更新後のリグレッションテスト戦略
  - Renovate / Dependabot との連携設定
- **`tasks/15_インシデント対応(IncidentResponse).md`** — 本番障害の緊急対応・ポストモーテム
  - 障害の重要度分類（P1〜P4 定義・対応 SLA）
  - 緊急対応手順（ロールバック / ホットフィックス / トラフィック制御）
  - 根本原因分析テンプレート（5 Whys / フィッシュボーン図）
  - ポストモーテム文書の自動生成（タイムライン・影響範囲・再発防止策）

#### ループ監視ファイル（コミット `664f6a4`）
- **`.loop-monitor-prompt.md`** — Monitor Loop 実行時の参照プロンプト（隠しファイル）
  - `/loop` コマンドが Monitor フェーズ開始時に自動参照するプロンプト定義
  - コンテキスト収集の優先順位・除外パターン設定
- **`.loop-verify-prompt.md`** — Verify Loop 実行時の参照プロンプト（隠しファイル）
  - `/loop` コマンドが Verify フェーズ開始時に自動参照するプロンプト定義
  - 品質ゲートの通過条件・失敗時フィードバックの形式設定

#### プロンプト（コミット `664f6a4`）
- **`prompts/`** ディレクトリを初期作成

### Changed
- なし（本バージョンは初期リリース）

### Removed
- ルートフォルダに散在していた日本語ドキュメント17件を `operations/` / `tasks/` / `best-practices/` 等に移動・統合・再作成

---

## 比較リンク

- [1.2.0...HEAD](https://github.com/Kensan196948G/Copilot-System-Development-Documents/compare/v1.2.0...HEAD)
- [1.1.0...1.2.0](https://github.com/Kensan196948G/Copilot-System-Development-Documents/compare/v1.1.0...v1.2.0)
- [1.0.0...1.1.0](https://github.com/Kensan196948G/Copilot-System-Development-Documents/compare/v1.0.0...v1.1.0)
- [0.0.0...1.0.0](https://github.com/Kensan196948G/Copilot-System-Development-Documents/compare/v0.0.0...v1.0.0)

---

## 変更統計サマリー

| バージョン | コミット数 | 追加ファイル数 | 追加行数 | 主な変更カテゴリ |
|----------|---------|------------|--------|--------------|
| v1.2.0 | 2 | 15ファイル | +2,800行以上 | カスタムエージェント・テンプレート・実例・運用ガイド |
| v1.1.0 | 2 | 2ファイル | +850行以上 | 能力分析・実現可能性レポート |
| v1.0.0 | 3 | 40ファイル以上 | +6,400行以上 | 初期ドキュメント体系の全構築 |
| **合計** | **7** | **57ファイル以上** | **+10,050行以上** | — |

---

## コントリビューター

| 名前 | 役割 |
|-----|------|
| GitHub Copilot CLI (Claude Code) | 主要ドキュメント自動生成・構造設計 |
| Project Owner | 方針決定・レビュー・最終承認 |

---

*このCHANGELOGは [Keep a Changelog v1.0.0](https://keepachangelog.com/ja/1.0.0/) に準拠しています。*  
*バージョニングは [Semantic Versioning 2.0.0](https://semver.org/lang/ja/) に従います。*
