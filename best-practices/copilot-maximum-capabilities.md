# GitHub Copilot CLI 最大能力分析 & ベストプラクティス（2026年版）

> **対象バージョン**: GitHub Copilot CLI v1.x（2026年2月 GA版）  
> **参照元**: [GitHub公式ドキュメント](https://docs.github.com/en/copilot)、GitHub Blog、GitHub Changelog  
> **検証日**: 2026年3月

---

## 1. Triple Loop 15H アーキテクチャ — 実現可能性評価

Triple Loop 15H（Monitor → Build → Verify の3ループ、15時間連続稼働）は、**Copilot CLI 最新版で完全実現可能**です。

```
┌─────────────────────────────────────────────────────────────┐
│                  Triple Loop 15H Architecture               │
│                                                             │
│  [Monitor Loop]  →→→  [Build Loop]  →→→  [Verify Loop]    │
│  継続監視・分析        自律実装              品質検証         │
│  /fleet + MCP         Autopilot            /review + Task   │
│                         Mode               Agent            │
└─────────────────────────────────────────────────────────────┘
```

### 1.1 Monitor Loop（継続監視ループ）

| 機能 | Copilot 対応機能 | 実現度 |
|------|----------------|--------|
| Issue/PR監視 | GitHub MCP Server（内蔵）| ✅ 完全対応 |
| ログ分析 | `explore` エージェント | ✅ 完全対応 |
| CI/CD状態確認 | `/pr` コマンド + GitHub MCP | ✅ 完全対応 |
| 外部メトリクス | カスタム MCP サーバー追加 | ✅ 拡張対応 |
| 定期実行 | `copilot --prompt "..." --allow-all` をcronで | ✅ CLI フラグ経由 |
| バックグラウンド監視 | `/delegate` + クラウドエージェント | ✅ 完全対応 |

**Monitor Loop 起動プロンプト例**:
```
/fleet @explore-agent GitHubのオープンIssueを分析して優先度を判定し、
@task-agent で現在のCIステータスを取得し、
実装すべき次のタスクをTASKS.mdに更新してください
```

### 1.2 Build Loop（自律実装ループ）

| 機能 | Copilot 対応機能 | 実現度 |
|------|----------------|--------|
| 自律的コード実装 | **Autopilot モード**（Shift+Tab） | ✅ 完全対応 |
| 並列実装 | `/fleet` コマンド | ✅ 完全対応 |
| マルチエージェント分業 | カスタムエージェント + /fleet | ✅ 完全対応 |
| モデル最適化 | タスク別モデル指定（`/model`） | ✅ 完全対応 |
| 自動PR作成 | `/delegate` → Copilot coding agent | ✅ 完全対応 |
| 長時間自律実行 | `--allow-all` + Autopilot | ✅ 完全対応 |

**Build Loop 最大構成（/fleet + Autopilot）**:
```
[Shift+Tab でAutopilot ON] →
/fleet TASKS.mdの次のタスクを実装してください。
  Use @backend-agent で APIエンドポイントを実装し、
  Use @frontend-agent で対応するUIを実装し、
  Use Claude Opus 4.6 で複雑なロジックを実装し、
  Use GPT-5.3-Codex でテストコードを生成してください
```

### 1.3 Verify Loop（品質検証ループ）

| 機能 | Copilot 対応機能 | 実現度 |
|------|----------------|--------|
| コードレビュー | `/review` コード・レビューエージェント | ✅ 完全対応 |
| テスト実行 | `task` エージェント（build/test）| ✅ 完全対応 |
| セキュリティチェック | カスタム `security-agent` | ✅ 拡張対応 |
| PR状態確認 | `/pr` コマンド | ✅ 完全対応 |
| diff確認 | `/diff` コマンド | ✅ 完全対応 |
| 品質ゲート判定 | `general-purpose` エージェント | ✅ 完全対応 |

**Verify Loop プロンプト例**:
```
/review このブランチの変更をレビューしてください。
バグ・セキュリティ脆弱性・ロジックエラーのみ報告してください。
その後 !npm test でテストを実行し、失敗があれば自動修正してください。
```

---

## 2. AI Agent Teams — 実現可能性評価

### 2.1 Copilot CLI 組み込みエージェント（4種）

```
┌──────────────────────────────────────────────────────┐
│               Built-in Agent Teams                   │
│                                                      │
│  ┌──────────┐  ┌──────────┐  ┌───────────────────┐  │
│  │ Explore  │  │  Task    │  │  General-Purpose  │  │
│  │ コード解析│  │ コマンド │  │   複雑タスク      │  │
│  │ 質問応答 │  │ 実行専任 │  │   高品質推論      │  │
│  └──────────┘  └──────────┘  └───────────────────┘  │
│                                                      │
│              ┌───────────────┐                       │
│              │  Code-Review  │                       │
│              │  変更レビュー  │                       │
│              │  問題発見専任  │                       │
│              └───────────────┘                       │
└──────────────────────────────────────────────────────┘
```

| エージェント | 役割 | 最適用途 |
|------------|------|---------|
| `explore` | コードベース分析・質問応答 | 「このコードは何をしているか？」「認証ロジックはどこ？」 |
| `task` | テスト・ビルド等コマンド実行 | `npm test`、`docker build`、lint実行 |
| `general-purpose` | 複雑なマルチステップタスク | フィーチャー実装、リファクタリング |
| `code-review` | 変更レビュー（ノイズ最小化） | PR前の最終品質チェック |

### 2.2 カスタムエージェント定義（3階層）

```
優先度（高→低）: システム > リポジトリ > 組織
```

**配置場所**:

| スコープ | パス | 適用範囲 |
|---------|------|---------|
| ユーザーグローバル | `~/.copilot/agents/*.agent.md` | 全プロジェクト |
| リポジトリ固有 | `.github/agents/*.agent.md` | 当該リポジトリのみ |
| 組織共通 | `.github-private/agents/*.agent.md` | 組織全プロジェクト |

**カスタムエージェント定義例**（`.github/agents/backend-agent.agent.md`）:

```markdown
---
name: backend-agent
description: RESTful API実装・DB設計・認証実装の専門エージェント
tools:
  - read
  - write
  - shell(npm:*)
  - shell(docker:*)
model: claude-opus-4.6
---

# Backend Agent 指示

あなたはバックエンド開発の専門家です。

## 専門領域
- Node.js / Express / Fastify APIサーバー
- PostgreSQL / Redis データモデリング
- JWT認証・OAuth2実装
- OpenAPI 3.0仕様書の自動生成

## コーディング規約
- エラーハンドリング: RFC 7807形式
- ログ: 構造化JSON（winston使用）
- テスト: Jest + supertest（カバレッジ80%以上必須）

## 禁止事項
- ハードコードされたシークレット
- console.log（winston使用）
- any型の乱用（TypeScript使用時）
```

### 2.3 /fleet による並列エージェント実行

```
/fleet の実行フロー:

  User Prompt
      │
      ▼
  Main Agent（オーケストレーター）
      │
      ├── Subagent 1 ──→ @backend-agent（API実装）
      ├── Subagent 2 ──→ @frontend-agent（UI実装）  ← 並列実行
      ├── Subagent 3 ──→ @test-writer（テスト生成）
      └── Subagent 4 ──→ @docs-agent（ドキュメント）
              │
              ▼
      結果を統合して報告
```

> **注意**: /fleetはプレミアムリクエストを複数消費します。サブエージェント数 × モデル倍率がコストに影響します。

---

## 3. 自律型 DevOps ワークフロー — 実現可能性評価

### 3.1 完全自律実行モード

| モード | コマンド | 特徴 |
|-------|---------|------|
| Autopilot | `Shift+Tab` | ユーザー承認不要で継続実行 |
| Allow All | `--allow-all` / `--yolo` | 全ツール権限を即付与 |
| Headless | `copilot --prompt "..." --allow-all` | ターミナル非インタラクティブ実行 |
| Cloud Delegate | `/delegate` | GitHub上のCopilot coding agentに委譲 → PR作成 |
| Background | `&` prefix | ローカルを解放してバックグラウンド実行 |

### 3.2 自律型 DevOps パイプライン構成

```
┌──────────────────────────────────────────────────────────────┐
│            Autonomous DevOps with Copilot CLI                │
│                                                              │
│  ① Issue作成                                                  │
│        │                                                      │
│        ▼                                                      │
│  ② Monitor Loop（/fleet + GitHub MCP）                        │
│     - Issue分析・優先度付け                                    │
│     - TASKS.md自動更新                                        │
│        │                                                      │
│        ▼                                                      │
│  ③ Build Loop（Autopilot + /fleet）                           │
│     - カスタムエージェントで並列実装                            │
│     - コミット・ブランチ作成                                    │
│        │                                                      │
│        ▼                                                      │
│  ④ Verify Loop（/review + task agent）                        │
│     - コードレビュー・テスト実行                                │
│     - 品質ゲート判定                                           │
│        │                                                      │
│        ▼                                                      │
│  ⑤ /delegate → Copilot coding agent → PR作成                 │
│        │                                                      │
│        ▼                                                      │
│  ⑥ GitHub Actions（CI/CD）→ デプロイ                          │
└──────────────────────────────────────────────────────────────┘
```

### 3.3 MCP サーバー拡張による DevOps 統合

```bash
# GitHub MCP（内蔵）— Issue/PR/Actions操作
/mcp  # 設定確認

# カスタムMCPサーバー追加例
/mcp add
# → Slack通知、Jira連携、Datadog監視、Terraform実行 等
```

**対応可能な外部統合**:
- Slack / Teams（通知）
- Jira / Linear（Issue同期）
- Datadog / Grafana（メトリクス）
- AWS / Azure / GCP（クラウドリソース）
- Docker / Kubernetes（コンテナ操作）
- Terraform / Pulumi（IaC実行）

---

## 4. プロンプトエンジニアリング ベストプラクティス

### 4.1 指示ファイル階層（優先度順）

```
優先度: 高────────────────────────────────────→低
  CLAUDE.md / GEMINI.md / AGENTS.md（リポジトリルート）
  .github/copilot-instructions.md
  .github/instructions/**/*.instructions.md（パス別）
  $HOME/.copilot/copilot-instructions.md（グローバル）
  カスタムエージェント定義（.github/agents/*.agent.md）
```

### 4.2 AGENTS.md ベストプラクティス

**❌ 悪い例**（曖昧・長すぎ）:
```markdown
このプロジェクトはNode.jsを使っています。
テストを書いてください。
コードは綺麗に書いてください。
セキュリティに気をつけてください。
```

**✅ 良い例**（具体的・構造化・測定可能）:
```markdown
# プロジェクト概要
Node.js 20 + TypeScript 5.x / Express 4.x / PostgreSQL 15 / Redis 7

# コーディング規約（強制）
- TypeScript strict モード必須（any型禁止）
- 関数は20行以内（超える場合は分割）
- エラーハンドリング: RFC 7807形式 (`{ type, title, status, detail }`)

# テスト要件（品質ゲート）
- 新規コード: Jest カバレッジ ≥ 80%
- E2Eテスト: Playwright、main ブランチのAPIエンドポイント全件

# セキュリティ（必須チェック）
- 環境変数のみでシークレット管理（.env.example を参照）
- SQLインジェクション防止: パラメータクエリ必須
- 認証なしエンドポイント追加禁止

# コミット規約
feat/fix/docs/refactor/test/chore: + 日本語説明
例: `feat: ユーザー認証APIにMFA機能を追加`

# 禁止事項
- console.log（winston.info/warn/errorを使用）
- ハードコードされた認証情報
- masterブランチへの直接プッシュ
```

### 4.3 効果的なプロンプト設計パターン

#### パターン1: タスク分解プロンプト（/fleet用）

```
/fleet 以下のタスクを並列実行してください:

1. Use @explore-agent で src/api/ ディレクトリを分析し、
   未テストのエンドポイントリストを出力

2. Use @test-writer で 未テストエンドポイント各3件の単体テストを生成、
   Jest + supertest使用、AAA（Arrange-Act-Assert）パターン

3. Use @docs-agent で OpenAPI 3.0仕様のYAMLを更新、
   新規エンドポイントのスキーマを追加

タスク1完了後にタスク2・3を実行してください。
```

#### パターン2: 品質ゲート付きループ

```
以下の品質基準をすべてPASSするまで繰り返し修正してください:

実装内容: [具体的なタスク説明]

品質基準:
✓ !npm test がすべてPASS
✓ !npm run lint がエラーゼロ  
✓ !npm run typecheck がエラーゼロ
✓ テストカバレッジ ≥ 80%（!npm run coverage で確認）

基準未達の場合: 失敗した項目の原因を分析して自動修正してください。
最大5回まで繰り返し、5回で解決しない場合は問題点をサマリーで報告。
```

#### パターン3: コンテキスト注入プロンプト

```
@TASKS.md の "In Progress" タスクを確認し、
@AGENTS.md の規約に従って実装してください。

実装前に以下を確認:
- @src/api/routes.ts で既存のルーティングパターン
- @docs/api-spec.yaml で既存のAPIスキーマ

実装後:
1. !git diff で変更確認
2. !npm test で回帰テスト
3. TASKS.mdの当該タスクを "Done" に更新
```

#### パターン4: エスカレーション付き自律ループ

```
[Autopilotモードで実行]

TASKS.md の Pendingタスクを上から順に処理してください。

各タスクの処理手順:
1. タスク内容を読んでWORKING.mdに作業計画を記録
2. 実装 → テスト → lint → typecheck
3. すべてPASSしたらコミット（規約: feat/fix + 日本語説明）
4. TASKS.mdで "Done" に更新
5. 次のタスクへ

停止条件（いずれか）:
- 全Pendingタスク完了
- テストが3回連続失敗
- セキュリティ上の懸念が発見された場合
→ 停止理由をWORKING.mdに記録してください
```

### 4.4 モデル選択ベストプラクティス

| タスク種別 | 推奨モデル | 理由 |
|-----------|-----------|------|
| 複雑なアーキテクチャ設計 | `claude-opus-4.6` | 高度な推論・設計能力 |
| 実装・コーディング | `claude-sonnet-4.6`（デフォルト） | バランス型、高速 |
| コード生成・変換 | `gpt-5.3-codex` | コード特化モデル |
| 大規模解析 | `gemini-3-pro-preview` | 大コンテキスト対応 |
| 単純なタスク/subagent | `gpt-5-mini` / `claude-haiku-4.5` | コスト最適化 |

**/fleet内でのモデル使い分け例**:
```
/fleet 以下を実装してください:
- Use Claude Opus 4.6 でシステム全体のアーキテクチャ設計を行い
- Use GPT-5.3-Codex で設計を基にコードを実装し  
- Use Claude Haiku 4.5 でコメントとドキュメントを追記してください
```

---

## 5. Copilot CLI が「できる最大のこと」

### 5.1 自律型フルサイクル開発（Ultimate Mode）

```bash
# Autopilot + Allow-All + /fleet の組み合わせ
# ターミナルから離れた完全自律開発

copilot --allow-all
> [Shift+Tab → Autopilot ON]
> /fleet TASKS.mdの全Pendingタスクを自律的に実装してください。
>   各タスクで品質ゲートを自動チェックし、
>   完成したものはGitHubにPRとして提出してください。
>   問題が発生した場合は詳細ログをWORKING.mdに記録してください。
```

**このモードで自律実行できること**:
- Issue → 設計 → 実装 → テスト → PR作成まで完全自動
- 複数ファイルの同時並列変更
- テスト失敗の自動デバッグ・修正
- CI/CDパイプラインの自動修正
- ドキュメントの自動更新
- セキュリティ脆弱性の自動検出・修正

### 5.2 マルチプロジェクト並列開発

```bash
# 複数ターミナルで異なるリポジトリのCopilotを並列起動
# Terminal 1: フロントエンドリポジトリ
cd frontend-repo && copilot --allow-all

# Terminal 2: バックエンドリポジトリ
cd backend-repo && copilot --allow-all

# Terminal 3: インフラリポジトリ
cd infra-repo && copilot --allow-all
```

### 5.3 Copilot SDK による完全自動化

```javascript
// Copilot SDK（Node.js）でエージェントループを外部制御
import { CopilotAgent } from '@github/copilot-sdk';

const agent = new CopilotAgent({
  repo: 'owner/repo',
  model: 'claude-sonnet-4.6',
  allowAll: true
});

// ループ制御をコードで実装
while (hasPendingTasks()) {
  const task = await getNextTask();
  const result = await agent.execute(task.prompt);
  await updateTaskStatus(task.id, result.success ? 'done' : 'failed');
}
```

### 5.4 能力の限界と注意点

| 制約 | 内容 | 対策 |
|------|------|------|
| プレミアムリクエスト上限 | プラン別の月次上限 | モデル倍率を意識、安価モデルをsubagentに |
| コンテキストウィンドウ | 長大セッションで劣化 | `/compact` で定期圧縮 |
| /fleet のコスト | サブエージェント数分 × 倍率 | タスク分割数を必要最小限に |
| 外部ネットワーク | セキュアな環境では制限 | MCP経由でのみアクセス |
| 破壊的操作 | `rm -rf` 等は慎重に | `--allow-all` は信頼環境のみ |
| セッション状態 | `/delegate` 後は非同期 | `/resume` でローカルに戻せる |

---

## 6. Triple Loop 15H 実装ロードマップ

### フェーズ1: 基盤整備（今すぐ実行可能）

```
✅ AGENTS.md を上記ベストプラクティスで整備
✅ .github/agents/ にカスタムエージェント定義を作成
✅ .github/copilot-instructions.md にプロジェクト規約を記載
✅ TASKS.md の構造化（Pending/In Progress/Done）
```

### フェーズ2: ループ個別最適化

```
Monitor: GitHub MCP + /fleet での定期解析プロンプト整備
Build:   カスタムエージェント + Autopilot での自律実装確認
Verify:  /review + task エージェントでの品質ゲート自動化
```

### フェーズ3: 完全自律化（Ultimate）

```
15H Autonomous Loop:
  Hour 0-5:   Monitor Loop（GitHub状態収集・タスク優先度決定）
  Hour 5-12:  Build Loop（Autopilot + /fleet で並列実装）
  Hour 12-15: Verify Loop（/review + テスト + PR作成）
  → 翌朝: 人間がPRレビューして承認
```

---

## 参考リンク

- [GitHub Copilot CLI 公式ドキュメント](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/use-copilot-cli)
- [/fleet コマンドリファレンス](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/fleet)
- [Autopilot モード](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/autopilot)
- [カスタムエージェント作成](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/create-custom-agents-for-cli)
- [Copilot CLI ベストプラクティス](https://docs.github.com/en/copilot/how-tos/copilot-cli/cli-best-practices)
- [Copilot SDK](https://docs.github.com/en/copilot/how-tos/copilot-sdk)
- [GitHub Copilot CLI GA発表（2026-02-25）](https://github.blog/changelog/2026-02-25-github-copilot-cli-is-now-generally-available/)
