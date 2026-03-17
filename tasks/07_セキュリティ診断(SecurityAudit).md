# 07 セキュリティ診断（Security Audit）

## 概要

依存関係の脆弱性・静的解析・機密情報漏洩・OWASP Top 10 のチェックを実施し、
Critical / High の脆弱性を即時修正します。

---

## Claude Code 起動コマンド

```bash
claude --dangerously-skip-permissions
```

---

## プロンプト指示

```
このプロジェクトのセキュリティ診断を実施し、脆弱性を修正してください。

【診断スコープ】
- 対象ディレクトリ: [PATH or 全体]
- 診断種別:
  - [ ] 依存関係の脆弱性（npm audit / pip-audit / govulncheck）
  - [ ] 静的解析（ESLint security plugin / Bandit / Semgrep）
  - [ ] 機密情報の検出（gitleaks / trufflehog）
  - [ ] OWASP Top 10 チェック
  - [ ] Docker イメージスキャン（trivy）
  - [ ] 認証・認可フローの検証

【実行してほしいこと（Security / Architect Agent が担当）】
1. 上記の診断ツールをすべて実行する
2. 検出結果を重大度別（Critical / High / Medium / Low）に整理する
3. Critical / High の脆弱性を即時修正する
   - 依存関係: バージョンアップまたは代替ライブラリへの移行
   - コード: 安全な実装に修正
4. Medium / Low は修正計画を提示する（次スプリント対応）
5. 機密情報がコードに含まれている場合:
   - 即時除去する
   - git の履歴からも削除する（git filter-branch / BFG）
   - .env.example に環境変数の定義を整備する
6. セキュリティ設定レポートを SECURITY.md に出力する
7. 修正後の確認
   - 再度診断ツールを実行して Critical / High が 0 件であることを確認する
   - git commit（security: 脆弱性修正）する

【修正方針】
- セキュリティ修正は機能変更と分離してコミットする
- 本番適用前に必ずユーザーが確認する
```

---

## 使用場面

| シナリオ | 説明 |
|---------|------|
| リリース前のセキュリティチェック | リリース前の必須ゲート |
| 定期的なセキュリティ保守 | 月次・四半期ごとの診断 |
| 依存関係の脆弱性アラート対応 | Dependabot アラートへの対応 |
| SOC2 / ISO27001 準備 | セキュリティ要件の充足確認 |

---

## 診断ツールのインストールと実行

```bash
# 依存関係の脆弱性（Node.js）
npm audit --audit-level=high

# 依存関係の脆弱性（Python）
pip install pip-audit && pip-audit

# 機密情報の検出
brew install gitleaks
gitleaks detect --source=. --verbose

# SAST（静的解析）
npx semgrep --config=p/security-audit src/

# Docker イメージスキャン
trivy image [IMAGE_NAME]:[TAG]

# OWASP ZAP（動的解析）
docker run -t owasp/zap2docker-stable zap-baseline.py -t http://localhost:3000
```

---

## OWASP Top 10 チェックリスト（2021）

| # | カテゴリ | 主なチェック項目 |
|---|---------|----------------|
| A01 | アクセス制御の不備 | 認可チェックの漏れ |
| A02 | 暗号化の失敗 | 平文での機密情報保存 |
| A03 | インジェクション | SQL / コマンドインジェクション |
| A04 | 安全でない設計 | 脅威モデリングの欠如 |
| A05 | セキュリティの設定ミス | デフォルト設定・デバッグ情報の露出 |
| A06 | 脆弱で古いコンポーネント | 未更新の依存関係 |
| A07 | 識別と認証の失敗 | 弱いパスワード・セッション管理 |
| A08 | ソフトウェアとデータの完全性の失敗 | CI/CD パイプラインの保護 |
| A09 | セキュリティログと監視の失敗 | 不十分なログ記録 |
| A10 | SSRF | 外部リクエストの検証不足 |

---

## ⚠️ 重要な注意事項

- **gitleaks で機密情報が検出された場合、git history の書き換えはユーザー確認必須**
- 修正後は必ず本番環境への影響範囲を確認してからデプロイする
- SECURITY.md に脆弱性の報告方法を記載することを推奨
