# TASKS.md — タスク管理テンプレート

> **使い方**: このファイルをプロジェクトルートにコピーして使用してください。  
> Copilot CLIは自動的にこのファイルを読み取り、タスクの優先順位付けと進捗管理に活用します。

---

## 📊 ダッシュボード

| ステータス | 件数 |
|-----------|------|
| 🔴 Pending | 0 |
| 🟡 In Progress | 0 |
| ✅ Done | 0 |
| ⛔ Blocked | 0 |

---

## 🔴 Pending（未着手）

<!-- タスクを以下の形式で追加してください -->

### [TASK-001] タスク名をここに記入
- **優先度**: High / Medium / Low
- **見積もり**: Mサイズ（S: 1-2h / M: 2-4h / L: 4-8h / XL: 8h+）
- **担当エージェント**: `@backend-agent` / `@frontend-agent` / `@test-writer` / 未定
- **関連Issue**: #000
- **説明**:
  ここにタスクの詳細を記述します。
  - 実装すべき機能
  - 受け入れ基準
  - 参考ファイル: `src/path/to/file.ts`
- **依存タスク**: なし / TASK-XXX 完了後

---

## 🟡 In Progress（進行中）

<!-- Copilotが作業開始時にここに移動します -->

---

## ✅ Done（完了）

<!-- 完了タスクはここに記録されます -->

---

## ⛔ Blocked（ブロック中）

<!-- 依存関係・外部要因で進められないタスクはここに -->

---

## 📝 タスク記入ガイド

### 良いタスク定義の例
```
### [TASK-002] ユーザー認証APIにリフレッシュトークン機能を追加
- **優先度**: High
- **見積もり**: Mサイズ
- **担当エージェント**: @backend-agent
- **関連Issue**: #42
- **説明**:
  現在のJWT認証にリフレッシュトークンを追加する。
  - POST /api/v1/auth/refresh エンドポイントを追加
  - Redisにリフレッシュトークンを保存（TTL: 7日）
  - 既存の /api/v1/auth/login レスポンスに refresh_token フィールドを追加
  - 受け入れ基準: !npm test が全PASSすること、カバレッジ80%以上
  - 参考: src/api/auth/login.ts, src/middleware/auth.ts
```

### 悪いタスク定義の例（これはNG）
```
### [TASK-003] 認証改善
- 認証をよくする
```

---

## 🤖 Copilot への指示

このファイルを使ってCopilotに作業を依頼する場合:

```
# Pendingの最高優先度タスクを1件実装してください
TASKS.mdのPendingタスクのうち最も優先度が高いものを実装してください。
AGENTS.mdの規約に従い、完了後にこのファイルのステータスを更新してください。

# 全Pendingタスクを自律実装（Autopilot推奨）
[Shift+Tab → Autopilot ON]
/fleet TASKS.mdの全Pendingタスクを優先度順に並列実装してください。
各タスクの品質ゲート（テスト・lint・typecheck）をPASSしてからDoneに移動してください。
```
