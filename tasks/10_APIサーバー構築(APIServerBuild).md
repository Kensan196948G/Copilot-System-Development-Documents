# 10 API サーバー構築（API Server Build）

## 概要

REST API サーバーを Controller / Service / Repository の 3 層構造で構築。
OpenAPI 仕様書・認証・バリデーション・Docker 環境まで一括構築します。

---

## Claude Code 起動コマンド

```bash
claude --dangerously-skip-permissions
```

---

## プロンプト指示

```
以下の仕様で REST API サーバーを構築してください。

【API 概要】
- サービス名: [SERVICE_NAME]
- フレームワーク: [Express / Fastify / FastAPI / Gin / Hono など]
- データベース: [PostgreSQL / MySQL / MongoDB / SQLite]
- ORM: [Prisma / Drizzle / TypeORM / SQLAlchemy / GORM]
- 認証方式: [JWT / OAuth2 / API Key / Session]
- デプロイ先: [Vercel / Railway / AWS / GCP / Docker]

【エンドポイント設計（DevAPI / Architect Agent が担当）】
[エンドポイント一覧を箇条書きで記載]
例:
  GET    /api/v1/users           - ユーザー一覧取得（ページネーション付き）
  POST   /api/v1/users           - ユーザー作成
  GET    /api/v1/users/:id       - ユーザー詳細取得
  PUT    /api/v1/users/:id       - ユーザー更新
  DELETE /api/v1/users/:id       - ユーザー削除（Soft Delete）
  POST   /api/v1/auth/login      - ログイン（JWT 発行）
  POST   /api/v1/auth/refresh    - トークンリフレッシュ

【実装要件】
- Controller / Service / Repository の 3 層アーキテクチャ
- バリデーション（Zod / Pydantic / go-playground/validator）を実装する
- エラーレスポンスを統一する（RFC 7807 Problem Details 形式）
- レート制限を実装する（IP ベース: 100 req/min）
- ヘルスチェックエンドポイント（GET /health）を実装する
- 構造化ロギング（pino / loguru / zap）を設定する
- OpenAPI 3.0 仕様書を自動生成する
- CORS・Helmet などセキュリティヘッダーを設定する

【実行してほしいこと】
1. 設計を Agent Teams で議論して承認を得る
2. プロジェクト構造を作成する
3. 各エンドポイントを実装する
4. 統合テストを作成する（Supertest / httpx / httptest）
5. Docker / docker-compose.yml を作成する（DB 含む）
6. OpenAPI 仕様書（openapi.yaml）を生成する
7. git commit（feat: API サーバーの初期実装）する

【エラーレスポンス統一形式】
{
  "type": "https://example.com/errors/not-found",
  "title": "Resource Not Found",
  "status": 404,
  "detail": "User with id 123 was not found",
  "instance": "/api/v1/users/123"
}
```

---

## 使用場面

| シナリオ | 説明 |
|---------|------|
| マイクロサービスの新規構築 | 単機能 API の高速構築 |
| モバイルアプリ向けバックエンド | BFF パターンでの API 構築 |
| 社内ツールの API 化 | スプレッドシート → API 化など |
| 既存 API のリプレイス | レガシー API の現代化 |

---

## プロジェクト構造例（Fastify + TypeScript）

```
src/
├── routes/          ← ルーティング定義
│   └── users/
│       ├── index.ts
│       └── schema.ts    ← Zod スキーマ
├── services/        ← ビジネスロジック
│   └── UserService.ts
├── repositories/    ← DB アクセス
│   └── UserRepository.ts
├── middlewares/     ← 認証・ロギング
├── errors/          ← カスタムエラークラス
└── app.ts           ← アプリケーション設定
```

---

## Docker Compose テンプレート

```yaml
services:
  app:
    build: .
    ports: ["3000:3000"]
    environment:
      DATABASE_URL: postgresql://user:pass@db:5432/mydb
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
```

---

## ポイント

- エンドポイント一覧を事前に整理しておくと実装精度が大幅に向上する
- RFC 7807 形式のエラーレスポンスで API クライアントの実装が容易になる
- Docker Compose で DB を含む開発環境を一括起動できる
- Triple Loop の Build Loop で機能を段階的に追加していく運用が推奨
