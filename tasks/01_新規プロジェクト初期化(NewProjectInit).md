# 01 新規プロジェクト初期化（New Project Init）

## 概要

新規プロジェクトのスケルトン構築から初回コミット・GitHubリポジトリ登録まで、
Claude Code が完全自動で実施します。

---

## Claude Code 起動コマンド

```bash
claude --dangerously-skip-permissions
```

---

## プロンプト指示

```
新規プロジェクトを初期化してください。以下の要件で進めてください。

【プロジェクト概要】
- プロジェクト名: [PROJECT_NAME]
- 技術スタック: [STACK] （例: Next.js + TypeScript + Prisma + PostgreSQL）
- 目的: [PURPOSE]
- チーム規模: [人数 / 個人]

【実行してほしいこと】
1. ディレクトリ構造を設計・作成する
   - src/ レイアウト（feature-based または layer-based）を提案し承認を得る
2. package.json / pyproject.toml など依存関係ファイルを初期化する
3. 開発ツールを設定する
   - ESLint / Prettier / TypeScript (strict mode)
   - Husky + lint-staged（pre-commit hook）
   - commitlint（Conventional Commits 規約）
4. 環境変数ファイルを生成する
   - .env.example（全変数の説明コメント付き）
   - .env.local（gitignore 対象）
5. README.md を生成する
   - プロジェクト概要・アーキテクチャ図（Mermaid）・セットアップ手順
6. git init & 初回コミット（chore: initial project setup）を行う
7. GitHub リポジトリ作成（gh repo create）してリモートを登録し push する
8. TASKS.md と AGENTS.md を生成する（Triple Loop 運用向け）

【品質基準】
- TypeScript strict モードを有効にする
- lint / format / typecheck がすべてパスする状態にする
- 環境変数はすべて .env.example に記載する
- CI（GitHub Actions: lint + test）を .github/workflows/ci.yml に設定する

準備ができたら構成案を提示し、承認後に実装を開始してください。
```

---

## 使用場面

| シナリオ | 説明 |
|---------|------|
| 新規サービス立ち上げ | Web アプリ・API・CLI ツール問わず適用可能 |
| ハッカソン・PoC | 品質基準を最小限にして高速起動 |
| モノレポ構成 | Turborepo / Nx の初期設定にも対応 |
| OSS 公開準備 | LICENSE / CONTRIBUTING.md も同時生成 |

---

## カスタマイズポイント

```
[PROJECT_NAME]  → 実際のプロジェクト名（kebab-case 推奨）
[STACK]         → 使用する技術スタック（具体的に書くと精度向上）
[PURPOSE]       → 一文でサービスの目的を説明
```

### スタック例

| カテゴリ | スタック例 |
|---------|-----------|
| フルスタック Web | Next.js 14 + TypeScript + Prisma + PostgreSQL + Tailwind CSS |
| API サーバー | Fastify + TypeScript + Drizzle + PostgreSQL |
| Python API | FastAPI + Python 3.12 + SQLAlchemy + PostgreSQL |
| CLI ツール | Go 1.22 + Cobra |
| モバイル BFF | Hono + TypeScript + Cloudflare Workers |

---

## Triple Loop との連携

初期化後に Triple Loop 15H 運用を始める場合：

```
# 生成された TASKS.md に最初のタスクを追加
# その後 /loop コマンドで自律開発を開始

/loop 450m 本ファイル内の「Build Loop 指示」に従い、TASKS.mdの最優先タスクを実装してください。
```

---

## ポイント

- `--dangerously-skip-permissions` で全操作（ファイル作成・git・gh）が自動実行される
- 構成案を提示してから実装開始することで意図しない構成を防止できる
- TASKS.md と AGENTS.md を初期化時に作成しておくと Triple Loop がすぐ使える
