# 14 依存関係更新（Dependency Update）

## 概要

依存関係を優先度順（CVE → EOL → マイナー → メジャー）に安全に更新。
各更新後にテストを実行して回帰がないことを確認します。

---

## Claude Code 起動コマンド

```bash
claude --dangerously-skip-permissions
```

---

## プロンプト指示

```
このプロジェクトの依存関係を安全に最新化してください。

【対象】
- パッケージマネージャー: [npm / yarn / pnpm / pip / cargo / go]
- 更新スコープ:
  - [ ] セキュリティパッチのみ（最安全）
  - [ ] マイナーバージョンアップ（比較的安全）
  - [ ] メジャーバージョンアップ（破壊的変更の可能性あり）
- 除外パッケージ（理由付き）: [PACKAGE_NAME（理由）]

【実行してほしいこと（Ops / Security Agent が担当）】
1. 現在の依存関係を一覧化し、更新可能なパッケージを確認する
2. 優先度順に整理する:
   🔴 CVE が報告されているパッケージ（即時対応）
   🟠 メンテナンス終了 (EOL) のパッケージ（早急に対応）
   🟡 マイナーアップデートのパッケージ（定期対応）
   🟢 メジャーアップデートのパッケージ（計画的に対応）
3. 更新によって破壊的変更がないかを確認する
   - 各パッケージの CHANGELOG / Migration Guide を確認する
   - TypeScript 型変更がないか確認する
4. 段階的に更新を適用する（1パッケージずつ or カテゴリごと）
5. 各更新後にテストを実行して回帰がないことを確認する
6. メジャーバージョンアップが必要な場合:
   - 影響範囲を調査してコードを修正する
   - 修正内容を説明するコメントを追加する
7. package-lock.json / requirements.txt をコミットする
8. git commit（chore: 依存関係を更新）する

【注意事項】
- 一度に全部更新しない（問題の特定が困難になる）
- テストのない箇所のメジャーアップは慎重に（手動確認を推奨）
```

---

## 使用場面

| シナリオ | 説明 |
|---------|------|
| 定期的な依存関係メンテナンス | 月次・四半期ごとの更新 |
| Dependabot アラートへの対応 | 緊急セキュリティパッチ |
| Node.js / Python のバージョンアップ | ランタイムのバージョンアップに合わせた依存更新 |
| フレームワークメジャーアップ | React 18→19 などの大型アップグレード |

---

## 依存関係確認コマンド

```bash
# Node.js: 更新可能なパッケージ一覧
npm outdated

# セキュリティ脆弱性チェック
npm audit

# インタラクティブ更新（ncdu 的な UI）
npx npm-check-updates -i

# Python: 更新可能なパッケージ一覧
pip list --outdated

# セキュリティチェック
pip-audit

# Go
go list -u -m all
```

---

## メジャーアップデート前の調査手順

```bash
# 1. CHANGELOG を確認
open https://github.com/[owner]/[repo]/blob/main/CHANGELOG.md

# 2. Breaking Changes のみを抽出
npm info [package] dist-tags
npm info [package] versions --json

# 3. 型変更を確認（TypeScript）
npx tsd [package]

# 4. マイグレーションガイドを確認
# （各フレームワークの公式ドキュメント参照）
```

---

## 自動化（Dependabot 設定例）

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    groups:
      # minor・patch を一括 PR にまとめる
      minor-and-patch:
        update-types: ["minor", "patch"]
    # メジャーアップは個別 PR
    ignore:
      - dependency-name: "*"
        update-types: ["version-update:semver-major"]
```

---

## ポイント

- 「一度に全部更新しない」という制約が回帰リスクを劇的に下げる
- Dependabot で minor/patch は自動 PR、major は手動で対応が効率的
- EOL パッケージは CVE と同じ優先度で対応する（将来のリスク）
- Triple Loop の Monitor Loop がセキュリティアラートを自動検知する
