# 06 CI/CD 構築（CI/CD Setup）

## 概要

GitHub Actions を中心に、PR 時のチェック・main マージ時のステージングデプロイ・
タグ push 時の本番デプロイのパイプラインを構築します。

---

## Claude Code 起動コマンド

```bash
claude --dangerously-skip-permissions
```

---

## プロンプト指示

```
このプロジェクトに CI/CD パイプラインを構築してください。

【環境情報】
- CI プラットフォーム: [GitHub Actions / GitLab CI / CircleCI など]
- デプロイ先: [Vercel / AWS ECS / GCP Cloud Run / Heroku / VPS など]
- 技術スタック: [Node.js 20 / Python 3.12 / Go 1.22 など]
- コンテナ利用: [Docker あり / なし]

【パイプライン要件（Ops / QA Agent が設計）】
1. PR 作成・更新時に実行（高速フィードバック）
   - lint / format チェック
   - typecheck
   - ユニットテスト（並列実行）
   - セキュリティスキャン（npm audit / trivy）
   - ビルド確認
2. main マージ時に実行（ステージング CI）
   - 上記すべて + 統合テスト
   - Docker イメージビルド & プッシュ
   - ステージング環境へ自動デプロイ
   - Smoke test（デプロイ後の動作確認）
3. タグ push 時に実行（本番リリース・要ユーザー確認）
   - Production デプロイ（Approval Gate 付き）
   - リリースノート自動生成（git-cliff）
   - Slack / Teams への通知

【実行してほしいこと】
1. 現在のプロジェクト構成を分析する
2. 最適なパイプライン設計を提示して承認を得る
3. CI 設定ファイルを生成する（.github/workflows/*.yml）
4. キャッシュ戦略を設定してビルド時間を最適化する
   - Node.js: node_modules / .npm キャッシュ
   - Docker: BuildKit レイヤーキャッシュ
5. Secrets の設定手順を README / docs/deployment.md に記載する
6. ジョブを並列実行できる設計にする
7. テスト結果・カバレッジを PR コメントに自動投稿する
8. サンプル PR を作成して CI が正常に動くことを確認する

【完了基準】
- PR 作成で自動的に CI が実行される
- main へのマージでステージングが更新される
- CI 実行時間が 5 分以内（PR チェック）
- 失敗時のエラーが明確でデバッグしやすい
```

---

## 使用場面

| シナリオ | 説明 |
|---------|------|
| 新規プロジェクトへの CI 導入 | 0 から CI/CD 環境を構築 |
| 既存 CI の高速化 | キャッシュ戦略・並列化で時間短縮 |
| デプロイフローの自動化 | 手動デプロイをパイプラインに移行 |
| マルチ環境管理 | dev / staging / production の環境分離 |

---

## GitHub Actions ワークフロー構成例

```yaml
# .github/workflows/ci.yml（PR チェック）
name: CI
on: [pull_request]
jobs:
  lint-typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci
      - run: npm run lint && npm run typecheck

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci && npm test -- --coverage
      - uses: codecov/codecov-action@v4  # カバレッジ PR コメント
```

---

## 重要: Secrets の設定

> ⚠️ **Secrets の設定はユーザーが手動で実施してください**（セキュリティ上 AI に委任しない）

設定場所: GitHub → Settings → Secrets and variables → Actions

| Secret 名 | 用途 |
|-----------|------|
| `DEPLOY_KEY` | デプロイ用 SSH キー |
| `DOCKER_HUB_TOKEN` | Docker Hub 認証 |
| `SLACK_WEBHOOK_URL` | Slack 通知 |

---

## ポイント

- Secrets の設定のみユーザーが実施（セキュリティポリシー）
- `concurrency` 設定で同ブランチへの並走実行をキャンセル（コスト削減）
- `needs` でジョブ依存関係を管理し、無駄な実行を避ける
- Triple Loop の DevOps Agent が CI の改善提案を自動生成する
