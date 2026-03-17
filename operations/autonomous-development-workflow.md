# 自律開発ワークフロー（Autonomous Development Workflow）

## 概要

Triple Loop 15H システムを使った自律型ソフトウェア開発の
エンドツーエンドワークフローです。タスク準備からマージ・日報出力までの
全フローを解説します。

---

## ワークフロー全体像

```
┌─────────────────────────────────────────────────────────────┐
│ 事前準備（人間が実施）                                       │
│  1. CLAUDE.md が ~/.claude/ に配置済みであることを確認      │
│  2. TASKS.md を作成してタスクを記載                          │
│  3. git リポジトリが初期化済みであることを確認              │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ セッション開始（人間が実施）                                 │
│  $ claude --dangerously-skip-permissions                    │
│  > /loop 900m [コマンド文面を貼り付け] Enter                │
└──────────────────────┬──────────────────────────────────────┘
                       │
          ┌────────────┴────────────┐
          ▼                         ▼
  ┌───────────────┐         ┌───────────────┐
  │  Cycle 1      │         │  Cycle 2      │
  │  (7時間)      │         │  (7時間)      │
  └───────┬───────┘         └───────┬───────┘
          │                         │
    Monitor (30m)             Monitor (30m)
          │                         │
    Build (2h)                Build (2h)
          │                         │
    Verify (4h)               Verify (4h)
          │                         │
    CD3 (30m)                 最終処理 (30m)
                                    │
                              ┌─────┴──────┐
                              │            │
                         git push      作業日報
                         PR作成/        出力
                         マージ
```

---

## Phase 1: 事前準備

### TASKS.md の書き方（重要）

タスクの記述品質が実装品質に直結します。

**最低限の記述（動きはするが品質は低い）:**
```markdown
- [ ] ユーザー認証を実装する
```

**推奨の記述（高品質な実装が生成される）:**
```markdown
- [ ] JWT を使ったユーザー認証 API を実装する
  - POST /api/v1/auth/login（メール + パスワード → JWT 返却）
  - POST /api/v1/auth/refresh（リフレッシュトークン → 新 JWT）
  - POST /api/v1/auth/logout（トークン無効化）
  - Zod でバリデーション
  - bcrypt でパスワードハッシュ（rounds: 12）
  - Jest で統合テストを作成（80% カバレッジ目標）
  - 参考: src/users/ ディレクトリの既存実装パターンに合わせる
```

### AGENTS.md の初期化（推奨）

```markdown
# AGENTS.md — プロジェクト設計判断・AI 学習記録

## アーキテクチャ方針
- レイヤ構造: Controller → Service → Repository
- エラー処理: RFC 7807 Problem Details 形式
- テスト: Jest + Supertest（統合テスト中心）

## 技術スタック
- Runtime: Node.js 20
- Framework: Fastify
- ORM: Prisma + PostgreSQL
- Auth: JWT（jsonwebtoken）

## 命名規約
- ファイル: kebab-case（user-service.ts）
- クラス: PascalCase（UserService）
- 関数: camelCase（getUserById）
```

---

## Phase 2: Monitor Loop（30分）

Monitor Loop が以下を実施します：

1. **コードベース解析**
   - 最新の git log・diff を確認
   - テスト失敗・lint エラーを検出
   - セキュリティアラートを確認

2. **タスク優先度決定**
   - TASKS.md から未完了タスクを取得
   - 依存関係・優先度で並び替え
   - 最優先タスクを Build Loop に渡す

3. **コンテキストパッケージ生成**
   - 関連ファイル（最大20件）を選定
   - 設計判断・制約条件をまとめる
   - `.loop-monitor-report.md` に出力

**出力物:** `.loop-monitor-report.md`

---

## Phase 3: Build Loop（2時間）

Build Loop が5段階ステップで実装します：

```
Step 1: 要件分析・設計（15分）
  ├─ TASKS.md のタスク詳細を解析
  ├─ 関連ファイルの現状を確認
  ├─ 実装方針を AGENTS.md に記録
  └─ コミット: なし（設計フェーズ）

Step 2: コア実装（45分）
  ├─ メイン機能を実装
  ├─ 既存コードのパターンに従う
  └─ コミット: feat: [機能名] の実装

Step 3: テスト実装（30分）
  ├─ ユニットテスト / 統合テストを作成
  ├─ カバレッジ 80% 以上を目標
  └─ コミット: test: [機能名] のテストを追加

Step 4: lint / typecheck 修正（15分）
  ├─ ESLint / tsc のエラーをすべて修正
  ├─ コードフォーマット適用
  └─ コミット: fix: lint / typecheck エラーを修正

Step 5: ドキュメント更新（15分）
  ├─ JSDoc コメントを追加
  ├─ README / API ドキュメントを更新
  └─ コミット: docs: [機能名] のドキュメントを追加
```

**CI 修復 AI:** ビルド・テスト失敗時に最大15回自動修復

**出力物:** `.loop-build-handoff.md` + main ブランチへのコミット

---

## Phase 4: Verify Loop（4時間）

Verify Loop が多角的な品質検証を実施します：

| 検証項目 | ツール | 基準 |
|---------|-------|------|
| コードレビュー | Agent Teams（QA/Architect/Security） | Critical/High 0件 |
| ユニットテスト | Jest / Vitest / pytest | 全件パス |
| 統合テスト | Supertest / httpx | 全件パス |
| カバレッジ | coverage | ライン 80% 以上 |
| lint | ESLint / flake8 | エラー 0件 |
| typecheck | tsc / mypy | エラー 0件 |
| セキュリティ | npm audit / Semgrep | High 0件 |

**自動修正フロー:**
```
検証失敗
  └─ 問題分析 + 修正実装
  └─ 再検証（最大15回）
    ├─ 成功 → PR 作成準備
    └─ 15回で解決不可 → TASKS.md に記録 + エスカレーション
```

**出力物:** `.loop-verify-report.md`

---

## Phase 5: 最終処理（30分）

2サイクル完了後（または15時間経過時）に実行：

```bash
# 自動実行される処理
git push origin main                      # 全コミットをプッシュ
gh pr create --title "..." --body "..."   # PR 作成
gh pr merge --merge                       # PR マージ（設定による）

# 作業日報の出力
docs/[YYYY-MM-DD]_自律開発作業報告.md
```

**作業日報の構成:**
```markdown
# [YYYY-MM-DD] 自律開発作業報告

## サマリー
- 稼働時間: 15時間（2サイクル）
- 完了タスク: X件
- コミット数: X件

## Cycle 1 実施内容
### Monitor Loop 結果
### Build Loop 実装内容
### Verify Loop レビュー結果

## Cycle 2 実施内容
...

## 次回への申し送り
- 未完了タスク
- 技術的負債・注意事項
```

---

## 人間のレビューポイント

Triple Loop が自動化する範囲でも、以下のポイントは人間が確認することを推奨：

| タイミング | 確認内容 | 所要時間 |
|-----------|---------|---------|
| セッション開始前 | TASKS.md の内容・優先度 | 5分 |
| セッション終了後 | 作業日報・git log | 10分 |
| PR マージ前 | コードレビュー（任意） | 15〜30分 |
| 週次 | AGENTS.md の設計判断 | 10分 |

---

## エスカレーション対応

Claude がエスカレーション（人間への判断要求）を出した場合：

```
.loop-alert.md が生成される
  ↓
内容を確認して判断する
  ├─ 実装方針の指示が必要 → TASKS.md に詳細を追記して再実行
  ├─ 設計変更が必要 → AGENTS.md を更新して再実行
  └─ 手動対応が必要 → 対応後に TASKS.md を更新
```

---

## 関連ドキュメント

- [起動ガイド](copilot-start-guide.md)
- [ループコマンドリファレンス](loop-command-usage.md)
- [Triple Loop アーキテクチャ](../architecture/triple-loop-architecture.md)
- [Agent Teams システム](../architecture/agent-teams-system.md)
- [ベストプラクティス](../best-practices/autonomous-session-best-practices.md)
