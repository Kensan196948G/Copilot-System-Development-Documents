# タスクプロンプト一覧（Task Prompt Index）

このフォルダには Claude Code で使用するタスク別プロンプトテンプレートが格納されています。
各ファイルをコピーして `[PROJECT_NAME]` などのプレースホルダを書き換えてから使用してください。

---

## 起動方法

```bash
# Claude Code を自動承認モードで起動
claude --dangerously-skip-permissions

# セッション内でプロンプトを貼り付けて実行
```

---

## タスク一覧

| # | ファイル | 用途 | Triple Loop での位置づけ |
|---|---------|------|------------------------|
| 01 | [新規プロジェクト初期化](./01_新規プロジェクト初期化(NewProjectInit).md) | プロジェクトのスケルトン構築 | Build Loop（初回） |
| 02 | [バグ修正](./02_バグ修正(BugFix).md) | バグ調査・修正・テスト追加 | Build Loop / 緊急対応 |
| 03 | [コードレビュー](./03_コードレビュー(CodeReview).md) | PR・コードの多角的レビュー | Verify Loop |
| 04 | [リファクタリング](./04_リファクタリング(Refactoring).md) | 技術的負債の段階的解消 | Build Loop |
| 05 | [テスト自動化](./05_テスト自動化(TestAutomation).md) | テスト生成・カバレッジ向上 | Verify Loop / Build Loop |
| 06 | [CI/CD 構築](./06_CI_CD構築(CICDSetup).md) | パイプライン設計・構築 | DevOps（初回設定） |
| 07 | [セキュリティ診断](./07_セキュリティ診断(SecurityAudit).md) | 脆弱性スキャン・修正 | Verify Loop / 定期保守 |
| 08 | [ドキュメント生成](./08_ドキュメント生成(DocGeneration).md) | README・API仕様書・図の自動生成 | Verify Loop |
| 09 | [パフォーマンス最適化](./09_パフォーマンス最適化(PerformanceOpt).md) | ボトルネック特定・最適化 | Build Loop |
| 10 | [API サーバー構築](./10_APIサーバー構築(APIServerBuild).md) | REST API の設計・実装 | Build Loop |
| 11 | [フロントエンド開発](./11_フロントエンド開発(FrontendDev).md) | UI コンポーネント・画面実装 | Build Loop |
| 12 | [データベース設計](./12_データベース設計(DatabaseDesign).md) | スキーマ設計・マイグレーション | Build Loop（初期） |
| 13 | [レガシーコード移行](./13_レガシーコード移行(LegacyMigration).md) | 段階的な技術スタック移行 | Build Loop（大型） |
| 14 | [依存関係更新](./14_依存関係更新(DependencyUpdate).md) | 安全な依存関係の最新化 | Monitor Loop / 定期保守 |
| 15 | [インシデント対応](./15_インシデント対応(IncidentResponse).md) | 本番障害の緊急対応 | 緊急対応（Loop外） |

---

## Triple Loop との組み合わせ

```
Monitor Loop → 14（依存関係更新）、07（セキュリティ診断）の課題を検知
Build Loop   → 01-05、09-13 のタスクを実装
Verify Loop  → 03（レビュー）、05（テスト）、08（ドキュメント）で品質確認
緊急対応     → 15（インシデント対応）は Loop 外で即時実施
```

---

## カスタマイズのポイント

各プロンプトの `[PLACEHOLDER]` を実際の値に置き換えて使用します。

```
[PROJECT_NAME]     → プロジェクト名
[FILE_PATH]        → 対象ファイルのパス
[STACK]            → 技術スタック
[SERVICE_NAME]     → サービス名
```
