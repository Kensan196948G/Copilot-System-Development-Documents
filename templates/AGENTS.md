# AGENTS.md — Copilot CLI プロジェクト指示テンプレート

> **使い方**: このファイルをプロジェクトルートにコピーして、プロジェクトに合わせてカスタマイズしてください。  
> `AGENTS.md` はCopilot CLIが自動で読み込む最優先の指示ファイルです。

---

<!-- ========== ここから下をプロジェクトに合わせて編集 ========== -->

# プロジェクト: [プロジェクト名]

## 概要
[プロジェクトの1〜2行説明]

**技術スタック**:
- **言語**: [例: TypeScript 5.x / Node.js 20]
- **フレームワーク**: [例: Express 4.x / React 18]
- **データベース**: [例: PostgreSQL 15 / Redis 7]
- **テスト**: [例: Jest + supertest / Playwright]
- **CI/CD**: [例: GitHub Actions]

---

## ⚠️ 最重要ルール（必ず守る）

1. **シークレット禁止**: 認証情報・APIキーをコードに書かない。環境変数（`.env`）のみ
2. **直接プッシュ禁止**: `main` / `master` ブランチへの直接プッシュ禁止
3. **テスト必須**: 新規コードは必ずテストを追加（カバレッジ ≥ 80%）
4. **セキュリティ確認**: 認証なしエンドポイントの追加は明示的な承認が必要

---

## コーディング規約

### 命名規則
```
変数・関数:  camelCase        (例: getUserById)
クラス:      PascalCase       (例: UserRepository)
定数:        UPPER_SNAKE_CASE (例: MAX_RETRY_COUNT)
ファイル:    kebab-case       (例: user-service.ts)
テスト:      *.test.ts        (例: user-service.test.ts)
```

### 関数・メソッド
- 1関数は20行以内（超える場合は分割）
- 引数は3個以内（超える場合はオブジェクト型に）
- 早期リターンで深いネストを避ける

### エラーハンドリング
```typescript
// ✅ RFC 7807形式
throw new HttpError({
  type: 'https://example.com/errors/not-found',
  title: 'Resource Not Found',
  status: 404,
  detail: `User with ID ${id} does not exist`
});

// ❌ NG
throw new Error('not found');
```

### ログ
```typescript
// ✅ 構造化ログ（winston使用）
logger.info('ユーザー作成成功', { userId, email });
logger.error('DB接続エラー', { error: err.message, stack: err.stack });

// ❌ NG
console.log('user created');
```

---

## テスト要件

| テスト種別 | フレームワーク | カバレッジ基準 |
|-----------|--------------|--------------|
| 単体テスト | Jest / Vitest | ≥ 80% |
| 統合テスト | supertest | 全APIエンドポイント |
| E2Eテスト | Playwright | 主要ユーザーフロー |

```bash
# テスト実行コマンド（品質ゲート）
npm test              # 全テスト実行
npm run test:coverage # カバレッジ計測
npm run lint          # ESLint
npm run typecheck     # TypeScript型チェック
```

---

## コミット規約（Conventional Commits）

```
<type>: <日本語説明>

type:
  feat     - 新機能
  fix      - バグ修正
  docs     - ドキュメント変更
  refactor - リファクタリング
  test     - テスト追加・修正
  chore    - ビルド・設定変更
  security - セキュリティ修正

例:
  feat: ユーザー認証にMFA機能を追加
  fix: ログイン時のセッション固定化バグを修正
  docs: API仕様書にエラーレスポンスの説明を追記
```

---

## Copilot エージェント指定

このプロジェクトで使用するカスタムエージェント:

| エージェント | 役割 | 使用場面 |
|------------|------|---------|
| `@backend-agent` | API・DB・認証実装 | サーバーサイド全般 |
| `@frontend-agent` | UI・UX実装 | フロントエンド全般 |
| `@test-writer` | テスト設計・生成 | テストカバレッジ改善 |
| `@security-agent` | セキュリティレビュー | PR前の最終確認 |
| `@docs-agent` | ドキュメント生成 | README・API仕様更新 |

---

## Triple Loop 15H 設定

```
Monitor Loop: Pendingタスクをチェックし優先度を更新（TASKS.md）
Build Loop:   最高優先度タスクを実装（品質ゲートまで）
Verify Loop:  /review + テスト実行 + セキュリティチェック
```

```bash
# Triple Loop 起動コマンド
copilot --allow-all
# [Shift+Tab → Autopilot ON]
# /fleet TASKS.mdに従ってTriple Loop 15Hを開始してください
```

---

## 禁止事項まとめ

```
❌ console.log（logger.info/warn/errorを使用）
❌ any型（TypeScript使用時）
❌ ハードコードされた認証情報・APIキー
❌ mainブランチへの直接プッシュ
❌ テストなしのコミット（新規機能・バグ修正）
❌ eval() / exec() の使用
❌ 認証なしの新規公開エンドポイント追加
```
