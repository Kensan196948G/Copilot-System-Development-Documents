# 実例: /fleet コマンド並列実行ウォークスルー

> **目的**: `/fleet` コマンドを使った並列マルチエージェント実行の実際の流れを示します。
> **想定シナリオ**: 新機能（ユーザープロフィール更新API）の実装を並列で完成させる。

---

## セットアップ（実行前の状態）

```bash
$ cd my-project
$ cat TASKS.md
```

```markdown
## 🔴 Pending

### [TASK-010] ユーザープロフィール更新API実装
- **優先度**: High
- **見積もり**: Lサイズ
- **説明**:
  PATCH /api/v1/users/:id/profile エンドポイントを追加
  - フィールド: name, bio, avatarUrl
  - バリデーション: name必須(最大50文字), bio任意(最大500文字)
  - avatarUrlはS3にアップロード済みのURLのみ許可
  - 受け入れ基準: Jest 80%以上、E2Eテスト追加
  - 参考: src/api/users/get-user.ts
```

---

## 実行手順

### Step 1: Copilot CLI を起動してAutopilotをON

```bash
$ copilot --allow-all
```

```
✓ Trusted directory: /home/user/my-project
> [Shift+Tab]  # Autopilotモードに切り替え
⚡ Autopilot mode enabled
```

### Step 2: /fleet でタスク分解・並列実行を指示

```
> /fleet TASK-010（ユーザープロフィール更新API）を実装してください。
    Use @backend-agent でAPIエンドポイントとバリデーションを実装し、
    Use @test-writer でJestテスト（カバレッジ≥80%）とE2Eテストを生成し、
    Use @docs-agent でOpenAPI仕様を更新してください。
    Use Claude Opus 4.6 でバリデーションロジックを実装してください。
```

---

## 実行ログ（実際の出力例）

```
🚀 /fleet: タスクを3つのサブタスクに分解します

  📋 サブタスク分解:
  ├── [1] API実装      → @backend-agent  (Claude Opus 4.6)
  ├── [2] テスト生成   → @test-writer    (GPT-5.3-Codex)     ← タスク[1]完了後に開始
  └── [3] API仕様更新  → @docs-agent     (Claude Sonnet 4.6) ← タスク[1]完了後に開始

  ⚡ Subagent [1] @backend-agent 起動中...
```

### Subagent 1: @backend-agent の実行

```
[Subagent 1 / backend-agent]
  📖 src/api/users/get-user.ts を参照して既存パターンを確認
  📖 src/middleware/auth.ts を確認（認証ミドルウェア）
  📖 src/validators/user.validator.ts を確認

  ✏️  src/api/users/update-profile.ts を作成
  ✏️  src/validators/profile.validator.ts を作成（Zodスキーマ）
  ✏️  src/api/users/index.ts を更新（ルート追加）

  🔨 !npm run typecheck
  ✓ TypeScript: エラーなし

  🔨 !npm run lint
  ✓ ESLint: エラーなし

  ✅ Subagent [1] 完了 (2m 34s)
     作成: 2ファイル / 変更: 1ファイル
```

### Subagent 2 & 3: 並列実行開始

```
⚡ Subagent [1] 完了 → Subagent [2] [3] を並列起動

[Subagent 2 / test-writer]              [Subagent 3 / docs-agent]
  📖 update-profile.ts を解析              📖 docs/api-spec.yaml を読み込み
  📝 テストケース設計:                     ✏️  PATCH /users/:id/profile 追加
    ✓ 正常系: 全フィールド更新               - request/response スキーマ
    ✓ 正常系: 一部フィールドのみ              - バリデーションエラー例
    ✓ 異常系: name が空                    ✓ OpenAPI 3.0 形式に準拠
    ✓ 異常系: name が51文字               ✅ Subagent [3] 完了 (1m 12s)
    ✓ 異常系: 未認証リクエスト
    ✓ 異常系: 他ユーザーのプロフィール更新
  ✏️  src/api/users/update-profile.test.ts 作成
  ✏️  e2e/tests/user-profile.spec.ts 作成

  🔨 !npm test src/api/users/update-profile.test.ts
  ✓ 18 tests passed (0 failed)

  🔨 !npm run test:coverage
  ✓ update-profile.ts: 87.5% (Lines), 83.3% (Branches)

  ✅ Subagent [2] 完了 (3m 18s)
```

### 統合・最終確認

```
🔗 全サブエージェント完了 → 統合チェック

  🔨 !npm test          → ✓ 156 tests passed, 0 failed
  🔨 !npm run lint      → ✓ No errors
  🔨 !npm run typecheck → ✓ No errors

  📊 実行サマリー:
  ┌────────────────┬──────────┬─────────────────────────────┐
  │ エージェント    │ 所要時間  │ 成果物                       │
  ├────────────────┼──────────┼─────────────────────────────┤
  │ backend-agent  │ 2m 34s   │ 2作成, 1変更                 │
  │ test-writer    │ 3m 18s   │ 18テスト, カバレッジ87.5%     │
  │ docs-agent     │ 1m 12s   │ OpenAPI仕様更新              │
  └────────────────┴──────────┴─────────────────────────────┘
  合計所要時間: 3m 52s（逐次実行なら約7m）

  💾 コミット作成中...
  ✓ feat: ユーザープロフィール更新APIを追加 (#TASK-010)
```

---

## 失敗時のリカバリー例

### テスト失敗の自動修正

```
[Subagent 2 / test-writer]
  🔨 !npm test
  ✗ 1 test failed:
    ✗ "avatarUrl が無効なURL形式の場合、400を返すこと"
      Expected: 400
      Received: 200

  🔍 原因分析: profile.validator.ts のURL検証が不完全
  ✏️  src/validators/profile.validator.ts を修正
       - z.string().url() → z.string().url().regex(/^https:\/\/s3\.amazonaws\.com/)

  🔨 !npm test  (2回目)
  ✓ 18 tests passed (0 failed) ← 自動修正成功
```

### 停止してエスカレーションが必要な例

```
[Subagent 1 / backend-agent]
  ⚠️  設計判断が必要なため停止します

  ## 停止理由
  avatarUrl の保存方式について判断が必要です

  ## 問題の詳細
  現在の実装はS3 URLを直接DBに保存しますが、
  src/api/avatars/upload.ts を見ると、アップロード処理が
  未実装であることがわかりました。

  ## 推奨アクション
  以下のいずれかを選択してください:
  A. アバターアップロードAPIも同時に実装する（スコープ拡大）
  B. avatarUrl フィールドを今回は除外する（スコープ維持）
  C. avatarUrl はバリデーションなしで受け入れる（暫定実装）
```

---

## プレミアムリクエスト消費量

この実行例での消費量目安:

| エージェント | モデル | 消費リクエスト数 |
|------------|-------|--------------|
| backend-agent | Claude Opus 4.6 | 約8〜12 |
| test-writer | GPT-5.3-Codex | 約6〜8 |
| docs-agent | Claude Sonnet 4.6 | 約3〜5 |
| **合計** | | **約17〜25** |

> **参考**: 通常のシングルエージェント実行の場合、同タスクで約10〜15リクエスト消費。  
> `/fleet` 使用で約1.5〜2倍のリクエストが消費される代わりに、時間は約半分になります。
