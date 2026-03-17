# templates/ — コピーしてすぐ使えるテンプレート集

このフォルダには、新しいプロジェクトに Copilot CLI + Triple Loop 15H を導入する際に
**コピーしてすぐ使えるテンプレート**を収録しています。

## ファイル一覧

| ファイル | 用途 | コピー先 |
|---------|------|---------|
| `AGENTS.md` | Copilot CLIへのプロジェクト規約指示 | プロジェクトルート |
| `TASKS.md` | タスク管理・優先度キュー | プロジェクトルート |
| `CLAUDE.md` | Claude/Copilot起動時の詳細設定 | プロジェクトルート |

## 使い方

```bash
# 新規プロジェクトへのセットアップ（1コマンド）
cp templates/AGENTS.md templates/TASKS.md templates/CLAUDE.md /path/to/your-project/

# プロジェクト情報をカスタマイズ
code /path/to/your-project/AGENTS.md
```

## カスタムエージェントのコピー

`.github/agents/` フォルダ内のエージェント定義も合わせてコピーしてください:

```bash
mkdir -p /path/to/your-project/.github/agents/
cp ../.github/agents/*.agent.md /path/to/your-project/.github/agents/
```

## 関連ドキュメント

- [チーム導入ガイド](../operations/team-onboarding-guide.md) — 30分クイックスタート手順
- [カスタムエージェント定義](../.github/agents/) — 5種のエージェント定義
- [best-practices/](../best-practices/) — ベストプラクティス集
