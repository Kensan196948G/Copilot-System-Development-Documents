# チーム導入・オンボーディングガイド

> **目的**: 個人利用からチーム・組織での Copilot CLI 活用へ拡張するための設定・運用手順を提供します。
> **対象読者**: 開発チームリード・エンジニアリングマネージャー

---

## 1. 30分クイックスタート（個人〜チーム移行）

### Step 1: インストール（5分）

```bash
# macOS / Linux
curl -fsSL https://gh.io/copilot-install | bash

# Windows（PowerShell v6+）
winget install GitHub.Copilot

# npm（全OS共通）
npm install -g @github/copilot
```

### Step 2: 認証（2分）

```bash
copilot
# → /login を実行
# → GitHub.com でデバイス認証
# → Copilot サブスクリプションを確認
```

### Step 3: プロジェクト設定（10分）

```bash
cd your-project

# テンプレートをコピー
cp /path/to/Copilot-System-Development-Documents/templates/AGENTS.md ./AGENTS.md
cp /path/to/Copilot-System-Development-Documents/templates/TASKS.md ./TASKS.md
cp /path/to/Copilot-System-Development-Documents/templates/CLAUDE.md ./CLAUDE.md

# プロジェクト情報を編集
code AGENTS.md  # 技術スタック・規約をカスタマイズ
code TASKS.md   # 最初のタスクを追加
```

### Step 4: カスタムエージェントの配置（8分）

```bash
mkdir -p .github/agents

# カスタムエージェントをコピー
cp /path/to/agents/backend-agent.agent.md .github/agents/
cp /path/to/agents/frontend-agent.agent.md .github/agents/
cp /path/to/agents/test-writer.agent.md .github/agents/
cp /path/to/agents/security-agent.agent.md .github/agents/
cp /path/to/agents/docs-agent.agent.md .github/agents/

# プロジェクトに合わせてカスタマイズ
code .github/agents/backend-agent.agent.md
```

### Step 5: 動作確認（5分）

```bash
copilot --allow-all
```

```
# CLIで確認コマンドを実行
/agent      # カスタムエージェント一覧を確認
/skills     # 利用可能なスキル一覧
/model      # 現在のモデルを確認

# テスト実行
Use @backend-agent でこのプロジェクトの技術スタックを説明してください
```

---

## 2. 組織・チームレベルの設定

### 組織共通エージェントの配置

```
組織の .github-private リポジトリ:
  .github-private/
    agents/
      backend-agent.agent.md      ← 組織標準のバックエンドエージェント
      security-agent.agent.md     ← セキュリティポリシー準拠
      docs-agent.agent.md         ← ドキュメント標準準拠
```

**設定方法**:
```bash
# .github-private リポジトリをクローン
git clone https://github.com/YOUR_ORG/.github-private
cd .github-private

# agents/ ディレクトリを作成
mkdir -p agents/

# エージェント定義を配置
cp /path/to/agents/*.agent.md agents/

git add -A
git commit -m "chore: 組織共通Copilotエージェントを追加"
git push
```

### 優先度（エージェントの上書き）

```
システム内蔵エージェント
    ↑ 上書きされる
組織レベル (.github-private/agents/)
    ↑ 上書きされる
リポジトリレベル (.github/agents/)
    ↑ 上書きされる
ユーザーレベル (~/.copilot/agents/)
```

---

## 3. 権限管理・セキュリティポリシー

### 環境別の権限設定推奨

| 環境 | 推奨設定 | コマンド |
|------|---------|---------|
| 開発ローカル | 適宜承認 | デフォルト |
| CI/CD パイプライン | 全権限付与 | `copilot --allow-all` |
| 本番環境 | **使用禁止** | — |
| セキュリティレビュー | 読み取り専用 | `--allow-tool=read` のみ |

### セキュリティポリシー推奨事項

```markdown
# チームのCopilot CLI 利用ポリシー

1. 本番DBへの接続禁止
   → AGENTS.md に明記: "本番環境への接続禁止"

2. シークレット管理
   → .gitignore で .env をignore
   → AGENTS.md でハードコード禁止を明記

3. 破壊的操作の制限
   → `--allow-all` は開発環境限定
   → CI/CDでは最小権限で実行

4. コードレビュー必須
   → /delegate 後のPRは必ず人間がレビュー
   → @security-agent によるチェックを義務化

5. 監査ログ
   → セッションを /share で保存・記録
   → 重要変更はコミットメッセージに経緯を記載
```

---

## 4. チームでの TASKS.md 管理フロー

### 基本フロー

```
開発者A（Issue作成）
    │
    ▼
GitHub Issue #42 作成
    │
    ▼
チームリード（毎朝 or 週次）
    │
    ▼
TASKS.md に追加・優先度設定
    │
    ▼
Copilot CLI（自律実装）
    │
    ├── Monitor Loop: TASKS.md を読んで着手
    ├── Build Loop: 実装・テスト
    └── Verify Loop: /review → PR作成
    │
    ▼
GitHub PR #43 作成（自動）
    │
    ▼
開発者A（レビュー）
    │
    ▼
マージ → Issue クローズ
```

### TASKS.md のブランチ管理

```bash
# 方法A: mainブランチで直接管理（小規模チーム向け）
git checkout main
# TASKS.md を直接編集してコミット

# 方法B: tasks-queue ブランチで管理（大規模チーム向け）
git checkout -b tasks-queue
# TASKS.md を編集・PR経由でmainにマージ
```

### タスク衝突の防止

```markdown
# TASKS.md で「担当エージェント/担当者」を明示

### [TASK-010] ユーザープロフィール更新API
- **担当**: @backend-agent（Copilot自律実行）
- **ステータス**: In Progress
- **セッションID**: sess_abc123  ← Copilotのセッション番号を記録
```

---

## 5. CI/CD パイプラインへの統合

### GitHub Actions での Copilot CLI 活用

```yaml
# .github/workflows/copilot-weekly-tasks.yml
name: Weekly Copilot Task Execution

on:
  schedule:
    - cron: '0 22 * * 0'  # 毎週日曜22時（月曜0時JST）
  workflow_dispatch:       # 手動実行も可能

jobs:
  copilot-tasks:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Install Copilot CLI
        run: curl -fsSL https://gh.io/copilot-install | bash
        
      - name: Run autonomous tasks
        env:
          GH_TOKEN: ${{ secrets.COPILOT_TOKEN }}  # Copilot Requests権限付きPAT
        run: |
          copilot --allow-all --prompt "
            TASKS.mdのPendingタスク（優先度Highのみ）を実装してください。
            品質ゲート（npm test, lint, typecheck）をPASSしてから
            各タスクのPRを作成してください。
          "
```

---

## 6. 新メンバー向けオンボーディング資料

### Day 1: 基本操作（2時間）

```
□ Copilot CLI インストール（上記 Step 1-2）
□ 基本コマンドの練習:
    - 「このプロジェクトの構造を説明してください」
    - @src/api/users.ts を解析してもらう
    - /diff で変更確認の練習

□ 読むべきドキュメント:
    - AGENTS.md（プロジェクト規約）
    - TASKS.md（現在のタスク一覧）
    - operations/copilot-start-guide.md
```

### Day 2-3: 実践（4時間）

```
□ 簡単なタスクを1件自律実行してみる
    - TASKS.md の Low優先度タスクを選ぶ
    - Autopilotは使わず、各ステップを確認しながら進める
    - /review でレビューを体験

□ /fleet を試す
    - 小さなタスクを2サブエージェントで並列実行
    - コスト消費量を /usage で確認
```

### Week 1: 応用（自習）

```
□ カスタムエージェントの作成・カスタマイズ
□ TASKS.md へのタスク追加を練習
□ Triple Loop 15H の1サイクルを完走
□ /delegate でのPR自動作成を体験
```

---

## 7. よくある質問（FAQ）

**Q: Copilotが間違ったコードを書いたら？**
A: `!git checkout -- .` で変更を取り消してください。重要な変更前は `!git stash` で退避を習慣に。

**Q: どのくらいのリクエストを消費するか事前にわかる？**
A: `/usage` で現在の消費量を確認できます。タスクの複雑さとモデルによって変わりますが、1タスク = 20〜50リクエストが目安です。

**Q: 本番環境でCopilotを使っても大丈夫？**
A: **本番環境での `--allow-all` は禁止**です。本番DBへの接続も禁止してください。開発・ステージング環境のみで使用してください。

**Q: セッションを途中で中断したら？**
A: `copilot --continue` で最近のセッションを再開できます。または `/resume` でセッション一覧から選択できます。

**Q: チームメンバーとセッションを共有できる？**
A: `/share` でセッションをMarkdown/GitHub Gistとして共有できます。
