# GitHub Copilot CLI 能力サマリー — Triple Loop 15H & 自律型DevOps

> **調査・分析結果まとめ**（2026年3月時点 / Copilot CLI GA版）  
> 詳細分析: [copilot-maximum-capabilities.md](./copilot-maximum-capabilities.md)

---

## Triple Loop 15H — ✅ 完全実現可能

| ループ | 対応 Copilot 機能 |
|--------|-----------------|
| **Monitor** | GitHub MCP（内蔵）+ `/fleet` + `explore` エージェント |
| **Build** | **Autopilot モード**（`Shift+Tab`）+ `/fleet` 並列実行 |
| **Verify** | `/review` + `task` エージェント + `/diff` / `/pr` |

```
[Monitor Loop]──→[Build Loop]──→[Verify Loop]
  GitHub MCP        Autopilot      /review
  /fleet            /fleet         task agent
  explore agent     Custom agents  /diff /pr
      └──────────────────────────────┘
              15H 自律サイクル
```

---

## AI Agent Teams — ✅ 完全実現可能

### 組み込み4種エージェント

| エージェント | 役割 |
|------------|------|
| `explore` | コードベース解析・質問応答 |
| `task` | テスト・ビルド等コマンド実行 |
| `general-purpose` | 複雑なマルチステップタスク |
| `code-review` | 変更レビュー（ノイズ最小化） |

### カスタムエージェント 3階層

```
~/.copilot/agents/*.agent.md          ← ユーザーグローバル（全プロジェクト）
.github/agents/*.agent.md             ← リポジトリ固有
.github-private/agents/*.agent.md     ← 組織共通
```

### `/fleet` による並列オーケストレーション

```
Main Agent（オーケストレーター）
  ├── Subagent 1: @backend-agent   （API実装）   ─┐
  ├── Subagent 2: @frontend-agent  （UI実装）     ├─ 並列実行
  ├── Subagent 3: @test-writer     （テスト生成） ─┘
  └── Subagent 4: @docs-agent      （ドキュメント）
```

---

## 自律型 DevOps — ✅ 完全実現可能

### 3つの自律実行パターン

| パターン | コマンド | 特徴 |
|---------|---------|------|
| **完全自律** | `Autopilot + --allow-all + /fleet` | Issue→実装→PR 完全自動 |
| **クラウド委譲** | `/delegate` | GitHub上のCopilot coding agentに委譲 → PR自動作成 |
| **MCP統合** | `/mcp add` | Slack/Jira/AWS/Terraform と連携 |

### 対応外部統合（MCP拡張）

```
Slack/Teams  ─→ 通知・アラート
Jira/Linear  ─→ Issue同期・ステータス更新
AWS/Azure    ─→ クラウドリソース操作
Terraform    ─→ IaC実行・インフラ変更
Datadog      ─→ メトリクス監視・アラート
Docker/K8s   ─→ コンテナ操作・デプロイ
```

---

## プロンプトエンジニアリング ベストプラクティス

### 1. AGENTS.md — 測定可能な規約を具体的に（曖昧な指示NG）

```markdown
# ❌ 悪い例
コードは綺麗に書いてください。テストを書いてください。

# ✅ 良い例
- TypeScript strict モード必須（any型禁止）
- Jest カバレッジ ≥ 80%（!npm run coverage で確認）
- エラーハンドリング: RFC 7807形式
- コミット規約: feat/fix/docs/refactor: + 日本語説明
```

### 2. カスタムエージェント定義 — `.github/agents/*.agent.md`

```markdown
---
name: backend-agent
description: RESTful API・DB設計・認証実装の専門エージェント
tools: [read, write, shell(npm:*), shell(docker:*)]
model: claude-opus-4.6
---
（モデル・ツール・禁止事項を明記）
```

### 3. 4つのプロンプトパターン

| パターン | 用途 | キーワード |
|---------|------|-----------|
| **タスク分解** | `/fleet` で並列処理 | `Use @agent-name で...` |
| **品質ゲート付きループ** | テスト全通過まで繰り返し | `全PASSするまで繰り返し修正` |
| **コンテキスト注入** | 既存コードを参照して実装 | `@ファイルパス を確認してから...` |
| **エスカレーション** | 停止条件付き自律実行 | `失敗3回で停止してログ記録` |

### 4. モデル使い分け

| タスク種別 | 推奨モデル |
|-----------|-----------|
| アーキテクチャ設計 | `claude-opus-4.6` |
| 実装・コーディング | `claude-sonnet-4.6`（デフォルト） |
| コード変換・生成 | `gpt-5.3-codex` |
| 軽量サブエージェント | `claude-haiku-4.5` / `gpt-5-mini` |

---

## Copilot CLI の「最大のこと」— 完全自律開発（究極形）

```bash
# ① Copilot を全権限モードで起動
copilot --allow-all

# ② Autopilot モードに切り替え
[Shift+Tab → Autopilot ON]

# ③ /fleet で並列自律実装を指示
/fleet TASKS.mdの全Pendingタスクを自律実装し、
  品質ゲート通過後にPRとして提出してください。
  Use @backend-agent でAPI実装、
  Use @test-writer でテスト生成、
  Use Claude Opus 4.6 で複雑なロジックを設計してください。
```

### この組み合わせで自動化できること

```
Issue受領
   │
   ▼
Monitor（GitHub MCP で Issue分析・優先度付け）
   │
   ▼
Build（Autopilot + /fleet で並列実装）
   │  ├─ @backend-agent  → APIコード
   │  ├─ @frontend-agent → UIコード
   │  └─ @test-writer    → テストコード
   ▼
Verify（/review + !npm test → 品質ゲート）
   │
   ▼
PR作成（/delegate → Copilot coding agent）
   │
   ▼
GitHub Actions（CI/CD → 自動デプロイ）

← 人間不在で完結 →
```

---

## 関連ドキュメント

| ドキュメント | 内容 |
|------------|------|
| [copilot-maximum-capabilities.md](./copilot-maximum-capabilities.md) | 詳細な能力分析・実装例・ロードマップ |
| [autonomous-session-best-practices.md](./autonomous-session-best-practices.md) | 自律セッションの運用ノウハウ |
| [../operations/copilot-start-guide.md](../operations/copilot-start-guide.md) | 起動方法・初期設定 |
| [../tasks/README.md](../tasks/README.md) | 15タスクプロンプト一覧 |
| [../prompts/monitor-loop-prompts.md](../prompts/monitor-loop-prompts.md) | Monitor Loop プロンプトテンプレート |
| [../prompts/verify-loop-prompts.md](../prompts/verify-loop-prompts.md) | Verify Loop プロンプトテンプレート |
