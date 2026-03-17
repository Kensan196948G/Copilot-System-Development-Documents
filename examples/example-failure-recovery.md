# 実例: 失敗時のリカバリー手順集

> **目的**: Copilot CLI 使用時によくある失敗パターンと、その回復手順を示します。
> **対象**: Triple Loop 15H 運用中に発生する典型的な問題への対処法。

---

## 1. テスト失敗ループから抜け出す

### 症状
```
🔄 3回連続でテストが失敗している
  1回目: npm test → 3 failed
  2回目: 修正後 npm test → 2 failed（別のテストが壊れた）
  3回目: さらに修正 → 5 failed（デグレが拡大）
```

### 原因の確認
```bash
# 最初に失敗したテストだけ実行してデバッグ
!npm test -- --testNamePattern="失敗しているテスト名" --verbose

# git で変更前の状態を確認
!git diff HEAD~1 src/対象ファイル.ts
```

### 回復手順
```
1. 変更をいったん退避
   !git stash

2. テストが全PASSの状態を確認
   !npm test  # ← 全PASSになるはず

3. 変更を少しずつ適用
   !git stash pop
   実装を小さなステップに分割して再実装してください。
   各ステップで !npm test を実行して確認してください。
```

### Copilot への指示例
```
テストが3回連続で失敗しました。
一度 !git stash で変更を退避し、
テストが全PASSの状態から再出発してください。
TASK-XXX の実装を以下の小さなステップに分割して進めてください:
  Step1: [最小限の変更]
  Step2: [次の変更]
各ステップで !npm test を実行して確認してください。
```

---

## 2. コンテキストウィンドウが枯渇したとき

### 症状
```
⚠️  Context usage: 94% — approaching limit
    The conversation history will be automatically compacted soon.
```

### 確認コマンド
```
/context  # トークン使用量の可視化
/usage    # セッション統計の確認
```

### 回復手順
```
/compact  # 会話履歴を要約・圧縮（推奨）
```

### セッション引き継ぎが必要な場合
```
/share    # 現在のセッションをMarkdownファイルに保存
/resume   # 別セッションで再開する場合
```

### 作業の引き継ぎ指示例
```
コンテキストが不足してきました。
以下の内容をWORKING.mdに記録してから /compact してください:
1. 完了したタスク（TASK-XXX, TASK-YYY）
2. 現在進行中のタスク（TASK-ZZZ）とその進捗状況
3. 発見した問題・注意事項
4. 次のステップ
```

---

## 3. /delegate（クラウドエージェント）が進まないとき

### 症状
```
/delegate でCopilot coding agentに委譲したが、
PRが作成されない、または想定と異なるPRが作成された
```

### 確認手順
```bash
# GitHubでエージェントの状態を確認
gh run list --limit 5

# またはCLI内で確認
/tasks  # バックグラウンドタスクの状態確認
```

### ローカルに引き戻す場合
```
/resume  # 委譲セッションをローカルに戻す
```

### 再試行する場合
```
前回の /delegate の指示が不明確でした。
以下の指示で再度試みてください:

[より具体的な指示を記載]
  - 実装するファイル: src/...
  - 使用するテストコマンド: npm test
  - コミットメッセージ形式: feat: [説明]
  - PRタイトル形式: feat: [説明] (#Issue番号)
```

---

## 4. 権限エラーが発生したとき

### 症状
```
⚠️  Permission denied: tool 'shell(rm ...)' requires approval
    または
⚠️  Directory not in allowed list: /path/to/dir
```

### 解決方法

**方法A: 個別承認**
```
# CLIの確認プロンプトで "Yes" を選択
# または "Yes, and approve for this session" を選択
```

**方法B: ディレクトリを追加**
```
/add-dir /path/to/directory   # 新しいディレクトリを許可リストに追加
/list-dirs                     # 現在の許可ディレクトリを確認
```

**方法C: 全権限を付与（信頼環境のみ）**
```
/allow-all
# または起動時に
copilot --allow-all
```

---

## 5. モデルが変わって動作が変化したとき

### 症状
```
以前は正しく動作していたが、
/model を変更後に動作が変わった
```

### 確認・切り替え
```
/model          # 現在のモデルを確認
/model          # リストから元のモデルを選択
```

### モデル別の特性

| モデル | 特性 | 向いているタスク |
|-------|------|---------------|
| Claude Opus 4.6 | 高品質・低速 | 設計・複雑なリファクタリング |
| Claude Sonnet 4.6 | バランス型 | 一般的な実装（デフォルト） |
| GPT-5.3-Codex | コード特化 | コード生成・変換 |
| Claude Haiku 4.5 | 高速・低コスト | 単純タスク・サブエージェント |

---

## 6. セッションを完全にリセットしたいとき

### 手順
```bash
# 1. 現在の作業を保存
!git stash  # または !git commit -m "wip: [作業内容]"

# 2. セッション履歴をクリア
/clear

# 3. 必要に応じてCLIを再起動
/restart

# 4. 作業を復元
!git stash pop  # stashした場合
```

---

## 7. 緊急停止が必要なとき

### Copilot を即座に止める

| 操作 | 効果 |
|------|------|
| `Esc` | 現在の操作をキャンセル |
| `Ctrl+C` | 入力クリア（2回で終了） |
| `Ctrl+D` | CLIをシャットダウン |

### 変更を取り消す

```bash
# 直前の変更を取り消す
!git checkout -- .  # 未コミットの変更を全部取り消す

# 特定のファイルだけ取り消す
!git checkout -- src/path/to/file.ts

# 直前のコミットを取り消す（変更は保持）
!git reset HEAD~1

# 直前のコミットを完全に取り消す
!git reset --hard HEAD~1
```

---

## まとめ: トラブル別クイックリファレンス

| 症状 | 対処 |
|------|------|
| テストが失敗ループ | `!git stash` → ステップ分割で再実装 |
| コンテキスト枯渇 | `/compact` → WORKING.mdに状態を記録 |
| /delegate が進まない | `/tasks` で確認 → `/resume` でローカルに戻す |
| 権限エラー | `/allow-all` or `/add-dir` |
| 動作がおかしい | `/model` でモデル確認・変更 |
| 完全リセット | `/clear` → `/restart` |
| 緊急停止 | `Esc` → `!git checkout -- .` |
