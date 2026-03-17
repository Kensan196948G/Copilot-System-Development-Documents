# 実世界プロジェクト例 — SaaS ダッシュボード開発での Triple Loop 実践

> **このドキュメントの目的**: 架空の SaaS プロジェクト「DataPulse」を題材に、
> Triple Loop アーキテクチャの実際の動作・ログ・コスト・トラブルシュートを具体的に示します。
> 実際のプロジェクトへの適用時のイメージを掴むためのリファレンスとして利用してください。

---

## 1. プロジェクト概要

### DataPulse — SaaS 分析ダッシュボード

| 項目 | 内容 |
|------|------|
| **プロジェクト名** | DataPulse |
| **種別** | B2B SaaS（分析ダッシュボード） |
| **フロントエンド** | React 18 + TypeScript + Vite + TailwindCSS |
| **バックエンド** | Node.js 20 + Express + Prisma ORM |
| **データベース** | PostgreSQL 15 |
| **インフラ** | Docker Compose（開発）/ AWS ECS（本番） |
| **テスト** | Vitest + React Testing Library + Playwright（E2E） |
| **CI/CD** | GitHub Actions |
| **チーム規模** | 開発者 3名 + PM 1名 |
| **Triple Loop 運用開始** | 2025年1月（プロジェクト立ち上げ2ヶ月後） |

### 当時の課題

Triple Loop 導入前は以下の問題を抱えていた:

- **テストカバレッジ 32%** — バグが本番で頻繁に発生
- **技術的負債が急増** — 2ヶ月で 87件の TODO コメントが蓄積
- **ドキュメント皆無** — 新メンバーのオンボーディングに1週間かかっていた
- **リリース頻度 月1回** — 手動テストに2日かかっていたため

### Triple Loop 導入の目標 KPI

| KPI | 導入前 | 目標（3ヶ月後） | 実績（3ヶ月後） |
|-----|--------|--------------|--------------|
| テストカバレッジ | 32% | 75% | **82%** ✅ |
| リリース頻度 | 月1回 | 週1回 | **週2回** ✅ |
| バグ発生率（本番） | 4.2件/週 | 1件/週以下 | **0.8件/週** ✅ |
| ドキュメント整備率 | 12% | 60% | **71%** ✅ |
| 1スプリント完了タスク数 | 8タスク | 15タスク | **18タスク** ✅ |

---

## 2. AGENTS.md 実装例

実際のプロジェクトルートに配置した `AGENTS.md` の内容。

```markdown
# DataPulse — AGENTS.md
# Copilot CLI (Claude Code) へのプロジェクト規約指示

## プロジェクト基本情報

project_name: DataPulse
language: TypeScript
runtime: Node.js 20 / React 18
package_manager: pnpm
branch_strategy: main (直接コミット可) / feature/* (PRで保護)
commit_convention: Conventional Commits (feat/fix/docs/refactor/test/chore)

## ディレクトリ構造

src/
  api/          # Express ルーター・コントローラー
  services/     # ビジネスロジック層
  repositories/ # Prisma を使ったDBアクセス層
  middleware/   # 認証・エラーハンドリング
  types/        # 共有型定義
  utils/        # ユーティリティ関数

client/
  src/
    components/   # React コンポーネント（Atomic Design）
    pages/        # ページコンポーネント
    hooks/        # カスタムフック
    stores/       # Zustand ストア
    api/          # API クライアント（fetch ラッパー）

prisma/
  schema.prisma  # DB スキーマ（変更時は必ずマイグレーション作成）
  migrations/    # マイグレーションファイル（手動編集禁止）

## コーディング規約

- TypeScript strict モード必須（any 型禁止）
- 関数コンポーネントのみ（クラスコンポーネント禁止）
- ESLint + Prettier の設定に従う（設定ファイル変更不可）
- API エンドポイントは必ず Zod でバリデーション
- エラーハンドリングは AppError クラスを継承すること
- DB アクセスはリポジトリ層のみ（サービス層からのDB直接アクセス禁止）

## テスト規約

- ユニットテスト: Vitest（新規関数には必須）
- 統合テスト: Supertest（APIエンドポイント追加時に必須）
- カバレッジ閾値: ステートメント 80% / ブランチ 75%
- テストファイル命名: *.test.ts または *.spec.ts

## セキュリティ規約

- シークレット・APIキーは .env のみ（ハードコーディング絶対禁止）
- SQL インジェクション対策: Prisma のパラメータバインドを常に使用
- 認証が必要なエンドポイントには authMiddleware を必ず適用
- CORS は allowedOrigins リストで明示的に管理

## エージェント設定

### backend-agent
model: claude-opus-4.6
focus:
  - src/api/ および src/services/ の実装
  - Prisma スキーマ変更・マイグレーション作成
  - 認証・認可ロジック
constraints:
  - Prisma スキーマを変更したら必ず npx prisma migrate dev を実行
  - 新規エンドポイントには必ず Swagger コメントを追加

### frontend-agent
model: claude-sonnet-4.6
focus:
  - client/src/ 以下の React 実装
  - TailwindCSS によるスタイリング
  - Zustand ストアの設計と実装
constraints:
  - コンポーネントは Storybook ストーリーも同時作成
  - アクセシビリティ: aria-label / role 属性を適切に付与

### test-writer
model: gpt-5.3-codex
focus:
  - Vitest ユニットテスト
  - Supertest 統合テスト
  - Playwright E2E テスト
constraints:
  - テストはモックを最小限に（統合テストを優先）
  - describe/it の命名は日本語可（可読性を優先）

### security-agent
model: claude-opus-4.6
focus:
  - セキュリティレビュー
  - 脆弱性修正
  - 依存関係の脆弱性スキャン
constraints:
  - Critical/High 脆弱性は即時修正必須
  - 修正後は必ずセキュリティテストを追加

### docs-agent
model: claude-sonnet-4.6
focus:
  - OpenAPI / Swagger 仕様書生成
  - README・CHANGELOG 更新
  - JSDoc / 型定義コメント
constraints:
  - API 仕様書は docs/api/ に出力
  - CHANGELOG は Keep a Changelog 形式を維持
```

---

## 3. TASKS.md 実装例

実際に運用中の `TASKS.md`。優先度スコア付き。

```markdown
# DataPulse — TASKS.md
# 最終更新: 2025-03-14 (Monitor Loop 自動更新)

## 🔴 Priority: Critical（即時対応）

- [ ] [TASK-089] JWT リフレッシュトークンのメモリリーク修正
  - 症状: 24時間稼働後にメモリ使用量が 2GB を超過
  - 関連ファイル: src/middleware/auth.ts, src/services/tokenService.ts
  - 優先度スコア: 8.2
  - 見積もり: 3H
  - 依存: なし

## 🟠 Priority: High（今週中）

- [ ] [TASK-091] ダッシュボード KPI ウィジェットの遅延読み込み実装
  - 要件: 初期表示 3秒以内（現在 8.4秒）
  - 関連ファイル: client/src/pages/Dashboard.tsx, client/src/components/widgets/
  - 優先度スコア: 6.1
  - 見積もり: 5H
  - 依存: TASK-089 完了後

- [ ] [TASK-092] チームメンバー招待メール送信機能
  - 要件: SendGrid 連携・招待リンク有効期限 72H
  - 関連ファイル: src/services/invitationService.ts（新規）
  - 優先度スコア: 5.8
  - 見積もり: 4H
  - 依存: なし

- [ ] [TASK-093] API レート制限の実装
  - 要件: 100リクエスト/分/ユーザー、超過時 429 返却
  - 関連ファイル: src/middleware/ （新規ミドルウェア）
  - 優先度スコア: 5.5
  - 見積もり: 2H
  - 依存: なし

## 🟡 Priority: Medium（今月中）

- [ ] [TASK-094] Playwright E2E テストのダッシュボード画面整備
  - 目標: ダッシュボード画面のカバレッジ 70% 以上
  - 優先度スコア: 4.2
  - 見積もり: 6H

- [ ] [TASK-095] OpenAPI 仕様書の自動生成 CI 組み込み
  - 優先度スコア: 3.8
  - 見積もり: 2H

## ✅ 完了（直近10件）

- [x] [TASK-088] Prisma N+1 クエリ問題修正 — 2025-03-13 完了
- [x] [TASK-087] ダッシュボード権限ロールの追加（VIEWER/EDITOR/ADMIN） — 2025-03-12 完了
- [x] [TASK-086] PostgreSQL インデックス最適化（クエリ 340ms → 28ms） — 2025-03-11 完了
- [x] [TASK-085] テストカバレッジ 80% 達成 — 2025-03-10 完了
```

---

## 4. Monitor Loop 実行ログ

2025-03-14 の Monitor Loop 実行ログ（タイムスタンプ付き実形式）。

```
[2025-03-14 09:00:03] ╔══════════════════════════════════════════╗
[2025-03-14 09:00:03] ║     Monitor Loop 開始 — DataPulse        ║
[2025-03-14 09:00:03] ╚══════════════════════════════════════════╝

[2025-03-14 09:00:05] 📂 コンテキスト収集開始
[2025-03-14 09:00:05]   → TASKS.md を読み込み中...
[2025-03-14 09:00:06]   → git log --oneline -20 を実行
[2025-03-14 09:00:06]   → 最新コミット: abc1234 fix(auth): Prisma N+1 修正 (#088)
[2025-03-14 09:00:07]   → GitHub Issues を確認中（open: 12件）
[2025-03-14 09:00:08]   → .loop-verify-report.md（前回 Verify 結果）を読み込み

[2025-03-14 09:00:09] 🔍 リポジトリ状態スキャン
[2025-03-14 09:00:10]   → npm test -- --coverage を実行
[2025-03-14 09:00:43]   → テスト結果: 247 passed, 0 failed, 2 skipped
[2025-03-14 09:00:43]   → カバレッジ: ステートメント 81.3% / ブランチ 76.8% ✅
[2025-03-14 09:00:44]   → npm run lint を実行
[2025-03-14 09:00:52]   → Lint: エラー 0件 / 警告 3件（既知）✅
[2025-03-14 09:00:53]   → npx tsc --noEmit を実行
[2025-03-14 09:01:02]   → TypeScript: エラー 0件 ✅

[2025-03-14 09:01:03] ⚠️  異常検知スキャン
[2025-03-14 09:01:04]   → メモリ使用量ログ確認: logs/app-2025-03-13.log
[2025-03-14 09:01:05]   → 🚨 検知: メモリ使用量が午後11時に 2.1GB に到達
[2025-03-14 09:01:05]   → 関連コード: src/middleware/auth.ts:87 (tokenCache)
[2025-03-14 09:01:06]   → .loop-alert.md に記録

[2025-03-14 09:01:07] 📊 タスク優先度評価
[2025-03-14 09:01:08]   → TASK-089 (JWT リフレッシュリーク): スコア 8.2 ← 最高優先度
[2025-03-14 09:01:08]   → TASK-091 (ウィジェット遅延読み込み): スコア 6.1
[2025-03-14 09:01:08]   → TASK-092 (招待メール): スコア 5.8
[2025-03-14 09:01:09]   → 選択: TASK-089（Critical + 本番障害の前兆）

[2025-03-14 09:01:10] 📦 コンテキストパッケージ生成
[2025-03-14 09:01:10]   → 関連ファイルを読み込み:
[2025-03-14 09:01:10]     - src/middleware/auth.ts（87行、tokenCache 実装あり）
[2025-03-14 09:01:11]     - src/services/tokenService.ts（234行）
[2025-03-14 09:01:11]     - src/types/auth.ts（型定義）
[2025-03-14 09:01:12]     - tests/middleware/auth.test.ts（既存テスト48件）
[2025-03-14 09:01:13]   → AGENTS.md から制約条件を抽出
[2025-03-14 09:01:14]   → .loop-monitor-report.md に出力完了

[2025-03-14 09:01:15] ✅ Monitor Loop 完了
[2025-03-14 09:01:15]   → 経過時間: 1分12秒
[2025-03-14 09:01:15]   → 選択タスク: TASK-089
[2025-03-14 09:01:15]   → Build Loop へ引き継ぎ
```

**出力された `.loop-alert.md` の内容:**

```markdown
# ⚠️ Alert — 2025-03-14 09:01:05

## 検知した異常

**種別**: メモリリーク（重大度: High）
**発見場所**: logs/app-2025-03-13.log
**症状**: サーバー起動後 24時間でメモリが 2.1GB に増加（正常時: 400MB）

## 調査結果

`src/middleware/auth.ts:87` の `tokenCache` Map が TTL なしで無制限に拡大している。
JWTトークンの検証結果を永続的にキャッシュしているが、
失効したトークンのエントリが削除されないため、時間とともに蓄積する。

## 推奨対処

1. `tokenCache` に TTL（Time-To-Live）機構を追加
2. または `node-lru-cache` などの LRU キャッシュライブラリに置き換え
3. 既存テストにメモリ増加のアサーションを追加

## 関連 Issue

- GitHub Issue #142: "Memory usage keeps growing after 12h"
```

---

## 5. Build Loop 実行ログ

TASK-089（JWT メモリリーク修正）の Build Loop 実行ログ。

```
[2025-03-14 09:01:20] ╔══════════════════════════════════════════╗
[2025-03-14 09:01:20] ║     Build Loop 開始 — TASK-089           ║
[2025-03-14 09:01:20] ╚══════════════════════════════════════════╝

━━━ Step 1: 要件分析・設計 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[2025-03-14 09:01:21] 📋 タスク分析
[2025-03-14 09:01:22]   → .loop-monitor-report.md を読み込み
[2025-03-14 09:01:23]   → 問題箇所を特定: src/middleware/auth.ts:87
[2025-03-14 09:01:24]   → 現在の実装:
                          const tokenCache = new Map<string, TokenPayload>();
                          // ← TTL なし。エントリが永続する
[2025-03-14 09:01:25]   → 解決方針を決定:
                          選択肢A: lru-cache パッケージ導入（推奨）
                          選択肢B: 手動 TTL 実装
                          → 選択A（メンテナンス性・信頼性が高い）
[2025-03-14 09:01:26]   → AGENTS.md に設計決定を記録

━━━ Step 2: コア実装 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[2025-03-14 09:01:30] 🔨 実装開始
[2025-03-14 09:01:31]   → pnpm add lru-cache を実行
[2025-03-14 09:01:38]   → pnpm add -D @types/lru-cache を実行
[2025-03-14 09:01:43]   → src/middleware/auth.ts を修正

変更内容（抜粋）:
  - import { LRUCache } from 'lru-cache';
  + const tokenCache = new LRUCache<string, TokenPayload>({
  +   max: 10000,           // 最大エントリ数
  +   ttl: 1000 * 60 * 15, // 15分 TTL（JWTの有効期限に合わせる）
  +   allowStale: false,
  + });

[2025-03-14 09:02:15]   → src/services/tokenService.ts を修正
                          キャッシュキーに token の jti クレームを使用するよう変更
[2025-03-14 09:02:48]   → 実装完了

━━━ Step 3: テスト実装 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[2025-03-14 09:02:50] 🧪 テスト追加
[2025-03-14 09:02:51]   → tests/middleware/auth.test.ts を更新
[2025-03-14 09:02:52]   → 追加したテストケース:
                          - "TTL 経過後にキャッシュエントリが削除される"
                          - "max エントリ数到達時に古いエントリが削除される"
                          - "10000エントリ追加後もメモリが上限内に収まる"
[2025-03-14 09:03:45]   → テスト実行: npm test -- tests/middleware/auth.test.ts
[2025-03-14 09:04:02]   → 結果: 51 passed（+3新規）, 0 failed ✅

━━━ Step 4: Lint / 型チェック修正 ━━━━━━━━━━━━━━━━━━━━━━━━━━━
[2025-03-14 09:04:05] 🔍 品質チェック
[2025-03-14 09:04:05]   → npm run lint を実行
[2025-03-14 09:04:13]   → エラー 1件検出:
                          src/middleware/auth.ts:12
                          Unexpected 'any' in type assertion (@typescript-eslint/no-explicit-any)
[2025-03-14 09:04:14]   → 修正: TokenPayload 型を明示
[2025-03-14 09:04:20]   → npm run lint を再実行: エラー 0件 ✅
[2025-03-14 09:04:21]   → npx tsc --noEmit を実行: エラー 0件 ✅

━━━ Step 5: ドキュメント + コミット ━━━━━━━━━━━━━━━━━━━━━━━━━
[2025-03-14 09:04:25] 📝 ドキュメント更新
[2025-03-14 09:04:26]   → JSDoc を auth.ts に追加
[2025-03-14 09:04:32]   → CHANGELOG.md に追記（Fixed セクション）
[2025-03-14 09:04:38]   → git add -A を実行
[2025-03-14 09:04:39]   → git commit:
                          fix(auth): JWT tokenCache を LRU キャッシュに変更しメモリリーク修正 (#089)
                          
                          - lru-cache パッケージを導入（max: 10000, ttl: 15分）
                          - 無制限拡大していた Map を LRUCache に置き換え
                          - TTL・エントリ上限のテスト3件を追加
                          - 修正前: 24H で 2.1GB, 修正後: 安定 400MB 以下（予測）
                          
                          Closes #142
[2025-03-14 09:04:42]   → コミット SHA: def5678

[2025-03-14 09:04:43] ✅ Build Loop 完了
[2025-03-14 09:04:43]   → 経過時間: 3分23秒
[2025-03-14 09:04:43]   → コミット: def5678
[2025-03-14 09:04:43]   → .loop-build-handoff.md に出力
[2025-03-14 09:04:43]   → Verify Loop へ引き継ぎ
```

---

## 6. Verify Loop 実行ログ

TASK-089 の Verify Loop 実行ログ（品質ゲート全項目）。

```
[2025-03-14 09:04:50] ╔══════════════════════════════════════════╗
[2025-03-14 09:04:50] ║     Verify Loop 開始 — TASK-089          ║
[2025-03-14 09:04:50] ╚══════════════════════════════════════════╝

[2025-03-14 09:04:51]   → コミット SHA: def5678 を検証
[2025-03-14 09:04:51]   → .loop-build-handoff.md を読み込み

━━━ Gate 1: テストスイート ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[2025-03-14 09:04:52]   → npm test -- --run を実行（全テスト）
[2025-03-14 09:05:41]   → 結果サマリー:
                          Tests: 250 passed, 0 failed, 2 skipped
                          Duration: 49.3s
                          
[2025-03-14 09:05:41]   → Gate 1: ✅ PASS（失敗テスト: 0件）

━━━ Gate 2: カバレッジ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[2025-03-14 09:05:42]   → npm test -- --coverage --run を実行
[2025-03-14 09:06:28]   → カバレッジ結果:
                          
                          ファイル                         | Stmts | Branch | Funcs | Lines
                          ─────────────────────────────────|───────|────────|───────|──────
                          src/middleware/auth.ts           | 94.2% |  88.1% | 100%  | 94.2%
                          src/services/tokenService.ts     | 87.3% |  82.4% |  96%  | 87.3%
                          ─────────────────────────────────|───────|────────|───────|──────
                          変更ファイル 合計                | 91.2% |  85.8% |  98%  | 91.2%
                          ─────────────────────────────────|───────|────────|───────|──────
                          プロジェクト全体                 | 81.3% |  76.8% |  89%  | 81.3%

[2025-03-14 09:06:28]   → Gate 2: ✅ PASS（変更ファイル: 91.2% ≥ 80% 閾値）

━━━ Gate 3: Lint / 静的解析 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[2025-03-14 09:06:30]   → npm run lint を実行
[2025-03-14 09:06:38]   → エラー: 0件 / 警告: 3件（変更前と同数・既知）
[2025-03-14 09:06:39]   → Gate 3: ✅ PASS（新規 violation: 0件）

━━━ Gate 4: セキュリティスキャン ━━━━━━━━━━━━━━━━━━━━━━━━━━━
[2025-03-14 09:06:40]   → npx audit を実行
[2025-03-14 09:06:48]   → 結果:
                          Critical: 0件
                          High: 0件
                          Moderate: 1件（既知・対処中・Issue #138）
                          Low: 3件（既知）
[2025-03-14 09:06:49]   → npx trivy fs . --severity CRITICAL,HIGH を実行
[2025-03-14 09:07:12]   → Critical: 0件 / High: 0件 ✅
[2025-03-14 09:07:13]   → Gate 4: ✅ PASS（Critical/High 脆弱性: 0件）

━━━ Gate 5: 型チェック ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[2025-03-14 09:07:14]   → npx tsc --noEmit を実行
[2025-03-14 09:07:22]   → TypeScript エラー: 0件
[2025-03-14 09:07:22]   → Gate 5: ✅ PASS

━━━ Gate 6: リグレッション検知 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[2025-03-14 09:07:23]   → ベースライン（コミット abc1234）との比較
[2025-03-14 09:07:24]   → テスト数: 247 → 250（+3件）✅
[2025-03-14 09:07:24]   → カバレッジ: 81.3% → 81.3%（変化なし）✅
[2025-03-14 09:07:24]   → Lint エラー: 0 → 0（変化なし）✅
[2025-03-14 09:07:24]   → Gate 6: ✅ PASS（リグレッション: 0件）

━━━ 最終判定 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[2025-03-14 09:07:25] 🎉 全品質ゲート PASS — マージ可能
[2025-03-14 09:07:26]   → PR 作成: gh pr create --title "fix(auth): JWT tokenCache メモリリーク修正 (#089)"
[2025-03-14 09:07:31]   → PR URL: https://github.com/datapulse/app/pull/147
[2025-03-14 09:07:32]   → .loop-verify-report.md に結果を出力
[2025-03-14 09:07:33]   → TASKS.md の TASK-089 を ✅ に更新
[2025-03-14 09:07:34]   → Monitor Loop に完了を通知

[2025-03-14 09:07:35] ✅ Verify Loop 完了
[2025-03-14 09:07:35]   → 経過時間: 2分45秒
[2025-03-14 09:07:35]   → ステータス: MERGE_READY
[2025-03-14 09:07:35]   → PR: #147
```

---

## 7. コスト実績

3ヶ月間の Triple Loop 運用コスト実績データ。

### 2025年3月の1週間（代表値）

| 日付 | サイクル数 | リクエスト数 | モデル内訳 | 処理タスク数 |
|------|----------|-----------|----------|-----------|
| 3/10（月）| 2サイクル 15H | 89 | Sonnet:71, Opus:18 | 2タスク |
| 3/11（火）| 2サイクル 15H | 102 | Sonnet:84, Opus:18 | 2タスク |
| 3/12（水）| 1サイクル 8H | 47 | Sonnet:41, Opus:6 | 1タスク |
| 3/13（木）| 2サイクル 15H | 94 | Sonnet:79, Opus:15 | 2タスク |
| 3/14（金）| 2サイクル 15H | 98 | Sonnet:83, Opus:15 | 2タスク |
| **週合計** | **9サイクル** | **430** | **S:358, O:72** | **9タスク** |

### モデル別リクエスト比率（3月実績）

| モデル | リクエスト数 | 割合 | コスト換算比率 |
|-------|-----------|-----|-------------|
| Claude Sonnet 4.6 | 1,432 | 79% | 1,432相当 |
| Claude Opus 4.6 | 291 | 16% | 2,910相当（10×） |
| Claude Haiku 4.5 | 89 | 5% | 22相当（0.25×） |
| **合計** | **1,812** | **100%** | **4,364相当** |

> **解説**: 実リクエスト数は 1,812 だが、コスト換算では 4,364相当。
> Opus を設計タスクに集中させることで、全タスクを Opus で処理した場合の
> **18,120相当（10,000リクエスト節約）** から大幅削減できている。

### /fleet 導入前後の比較（同種タスク・5件ずつ測定）

| 指標 | /fleet 非使用 | /fleet 使用（3エージェント） | 差分 |
|------|------------|--------------------------|------|
| 1タスク処理時間 | 平均 4.2H | 平均 2.8H | **-33%** ⚡ |
| リクエスト消費 | 平均 31件 | 平均 48件 | +55% 増加 |
| コード品質スコア | 7.8/10 | 8.4/10 | **+7.7%** ✅ |
| テストカバレッジ | 78% | 84% | **+6pt** ✅ |

> **判断**: 時間効率 > コスト増加 の場面では `/fleet` が有効。
> コスト制約が厳しい週は シングルエージェントに切り替えて運用した。

---

## 8. トラブル事例と解決

実際に発生した問題3件とその対処法。

---

### 🔴 トラブル事例 1: Build Loop が無限リトライループに陥った

**発生日**: 2025年2月3日

**症状**:
```
[2025-02-03 14:23:11] ⚠️ Build Step 2 失敗 — リトライ 1/3
[2025-02-03 14:38:44] ⚠️ Build Step 2 失敗 — リトライ 2/3
[2025-02-03 14:54:22] ⚠️ Build Step 2 失敗 — リトライ 3/3
[2025-02-03 15:10:01] ⚠️ Build Step 2 失敗 — リトライ 4/3 ← 上限を超えて継続
[2025-02-03 16:45:33] ⚠️ Build Step 2 失敗 — リトライ 12/3 ← 2時間後も継続
```

**根本原因**:
- PostgreSQL マイグレーションが途中で失敗した後、ロック状態に
- 同じマイグレーションを繰り返し適用しようとしてエラーが続いた
- AGENTS.md の「マイグレーション失敗時は手動確認を要求する」ルールが未記載だったため AI が自律で突破しようとし続けた

**対処手順**:
```bash
# 1. Claude Code セッションを強制終了（Ctrl+C）
# 2. DB の状態を確認
npx prisma migrate status

# 3. 失敗したマイグレーションをロールバック
npx prisma migrate resolve --rolled-back 20250203_add_org_table

# 4. マイグレーションファイルを修正（カラム名のタイポを修正）
vi prisma/migrations/20250203_add_org_table/migration.sql

# 5. 再適用
npx prisma migrate deploy

# 6. /loop を再実行（TASK から再開）
/loop 120m Build Loop のみ実行。TASK-073 を継続してください。
```

**再発防止**: `AGENTS.md` に以下を追記
```markdown
## エラー処理の例外ルール
- Prisma マイグレーションが 2回以上失敗した場合は即座に停止し .loop-alert.md に記録
- データベース関連エラーは自律リトライ禁止（データ破損のリスクあり）
```

---

### 🟠 トラブル事例 2: テストが環境依存でローカル Pass / CI Fail

**発生日**: 2025年2月19日

**症状**:
```bash
# ローカル（Monitor ログ）
[2025-02-19 10:45:12] ✅ Gate 1: PASS（250 passed, 0 failed）

# GitHub Actions CI
FAIL tests/integration/dashboard.test.ts
  ● Dashboard API › GET /api/v1/dashboard
    Error: connect ECONNREFUSED 127.0.0.1:5432
    at TCPConnectWrap.afterConnect
```

**根本原因**:
- テストが `DATABASE_URL=postgresql://localhost:5432/datapulse_test` をハードコードしていた
- GitHub Actions では PostgreSQL サービスコンテナが `localhost` ではなく `postgres` ホスト名で起動
- Verify Loop がローカル環境で Pass 判定 → CI で初めて発覚するパターン

**対処手順**:
```bash
# 1. テストの DB URL を環境変数から読むよう修正
# tests/helpers/db.ts
const dbUrl = process.env.DATABASE_URL ?? 'postgresql://localhost:5432/datapulse_test';

# 2. .github/workflows/ci.yml に環境変数を追加
env:
  DATABASE_URL: postgresql://postgres:postgres@postgres:5432/datapulse_test

# 3. GitHub Actions の PostgreSQL サービスコンテナ設定を確認・修正
services:
  postgres:
    image: postgres:15
    env:
      POSTGRES_PASSWORD: postgres
    options: >-
      --health-cmd pg_isready
      --health-interval 10s
```

**再発防止**: AGENTS.md に以下を追記
```markdown
## テスト規約（追加）
- 接続情報（DB URL、APIエンドポイント）は必ず環境変数から読む
- ハードコードされた localhost / ポート番号は Lint エラーとして検出する
- Verify Loop 完了後、CI の緑を確認してから TASKS.md を完了にマーク
```

---

### 🟡 トラブル事例 3: /fleet でサブエージェントが同じファイルを競合編集

**発生日**: 2025年3月5日

**症状**:
```
[2025-03-05 15:12:44] /fleet 実行開始（3エージェント）
[2025-03-05 15:14:33] backend-agent: src/types/dashboard.ts を編集
[2025-03-05 15:14:41] frontend-agent: src/types/dashboard.ts を編集
[2025-03-05 15:15:02] ❌ 競合エラー: src/types/dashboard.ts にコンフリクト発生
                       <<<<<<< backend-agent
                       export interface DashboardConfig {
                         refreshInterval: number;
                       =======
                       export interface DashboardConfig {
                         refreshIntervalMs: number; // Renamed for clarity
                       >>>>>>> frontend-agent
```

**根本原因**:
- `/fleet` 指示で共有型定義ファイルへのアクセス範囲を明示しなかった
- backend-agent と frontend-agent が同じ型を独立して「改善」しようとした
- サブエージェント間の排他制御がなかった

**対処手順**:
```bash
# 1. コンフリクトを手動解決
git checkout src/types/dashboard.ts  # 元に戻す
# refreshIntervalMs に統一することを決定
# 手動で型定義を修正

# 2. /fleet を再実行（担当ファイルを明示分割）
/fleet
  Use @backend-agent で src/api/ と src/services/ のみを担当し、
  Use @frontend-agent で client/src/ のみを担当し、
  Use @docs-agent で docs/ のみを担当してください。
  src/types/ の変更が必要な場合は backend-agent のみが担当すること。
```

**再発防止**: `/fleet` 使用ガイドラインを `AGENTS.md` に追記
```markdown
## /fleet 使用ガイドライン
- 各エージェントの担当ディレクトリを明示すること（重複禁止）
- 共有型定義（src/types/）は backend-agent のみが変更可能
- /fleet 後は必ず git status で競合がないことを確認してから Verify Loop を実行
```

---

## 9. プロジェクト完了後の振り返り

Triple Loop 運用 3ヶ月（2025年1月〜3月）の振り返り。

### KPI 達成状況

| KPI | 導入前 | 目標 | 実績 | 達成 |
|-----|--------|------|------|------|
| テストカバレッジ | 32% | 75% | **82%** | ✅ 超過達成 |
| リリース頻度 | 月1回 | 週1回 | **週2回** | ✅ 2倍達成 |
| バグ発生率（本番） | 4.2件/週 | 1件/週以下 | **0.8件/週** | ✅ 達成 |
| ドキュメント整備率 | 12% | 60% | **71%** | ✅ 超過達成 |
| 1スプリント完了タスク数 | 8タスク | 15タスク | **18タスク** | ✅ 超過達成 |
| 開発者の残業時間/週 | 平均 12H | 5H以下 | **平均 3H** | ✅ 大幅改善 |

### よかった点（Keep）

1. **Monitor Loop の異常検知が機能した**
   - トラブル事例1のメモリリーク（TASK-089）は Monitor の自動検知で発見。
   - 手動モニタリングでは発見が1週間以上遅れていたと推定。

2. **TASKS.md の優先度スコアが意思決定を効率化**
   - PM との「次は何をやるか」議論がほぼ不要になった。
   - スコアの根拠が明文化されているため、ステークホルダーへの説明コストが減少。

3. **Verify Loop の品質ゲートでリグレッション 0件**
   - 3ヶ月間、マージ後のバグ（リグレッション）は 0件。
   - 「動かすための修正で他が壊れる」サイクルが完全に止まった。

### 改善すべき点（Improve）

1. **AGENTS.md の初期設定に時間がかかった**
   - プロジェクト固有のルール（マイグレーション禁止事項など）は
     実際にトラブルが起きてから追記するパターンが多かった。
   - **改善策**: 新プロジェクト開始時に「AGENTS.md レビューセッション（1H）」を
     チームで実施し、過去のトラブルパターンを先に記載しておく。

2. **/fleet のコスト増加が計画より大きかった**
   - 当初予測: Sonnet 換算 +30% 増、実績: +55% 増
   - 特に最初の1ヶ月は並列エージェントの使いすぎでリクエスト上限に近づいた。
   - **改善策**: 週初めに `/usage` で残リクエスト数を確認し、
     残量が少ない週は `/fleet` を禁止する運用ルールを追加。

3. **E2E テストの Playwright 設定に時間がかかった**
   - Playwright の CI 設定（ブラウザインストール・タイムアウト調整）は
     AI が苦手なため手動設定が必要だった。
   - **改善策**: Playwright 設定ファイルは手動で整備してから
     Triple Loop に渡すテンプレートを `templates/` に用意する。

### 次の3ヶ月の計画

| 目標 | 具体的アクション |
|------|--------------|
| E2E テストカバレッジ 60% 達成 | Playwright タスクを TASKS.md に常時5件キープ |
| /fleet コスト最適化 | Haiku の活用比率を 5% → 15% に引き上げ |
| オンボーディング時間短縮 | Triple Loop 入門ドキュメントを `docs/onboarding/` に整備 |
| マルチリポジトリ対応 | frontend/backend を別リポジトリに分割して /fleet で並列管理 |

---

> **このドキュメントは架空のプロジェクト「DataPulse」を基に作成したリファレンス例です。**
> 実際のプロジェクトでは AGENTS.md・TASKS.md の内容を自プロジェクトに合わせて
> カスタマイズしてください。テンプレートは `templates/` を参照。
