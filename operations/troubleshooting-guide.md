# トラブルシューティングガイド（Triple Loop システム）

## 概要

このガイドは、Triple Loop 自律開発システムの運用中に発生しうるエラー・問題の
診断・解決手順をまとめたものです。

**対象読者:** 運用担当者・開発者  
**対象システム:** Triple Loop 15H（Monitor / Build / Verify + Agent Teams + Git/PR + MCP）

---

## 目次

1. [Monitor Loop のエラー](#1-monitor-loop-のエラー)
2. [Build Loop のエラー](#2-build-loop-のエラー)
3. [Verify Loop のエラー](#3-verify-loop-のエラー)
4. [Agent Teams のエラー](#4-agent-teams-のエラー)
5. [Git / PR 操作のエラー](#5-git--pr-操作のエラー)
6. [MCP / 外部連携のエラー](#6-mcp--外部連携のエラー)
7. [ログ確認・診断コマンド集](#7-ログ確認診断コマンド集)

---

## 1. Monitor Loop のエラー

### 1-1. コンテキスト収集失敗

#### 症状

```
[Monitor] ERROR: Failed to collect context package
[Monitor] context_files: 0 / 20 (expected ≥ 5)
.loop-monitor-report.md が空または未生成
```

#### 原因

| 原因 | 確認方法 |
|------|---------|
| リポジトリが初期化されていない | `git status` でエラーが出る |
| 対象ディレクトリへのアクセス権限がない | `ls -la` で確認 |
| TASKS.md が存在しない・空である | `cat TASKS.md` |
| git コマンドがパスに存在しない | `which git` |
| コンテキスト収集対象ファイルが0件 | `git ls-files | wc -l` |

#### 解決手順

1. リポジトリの状態を確認する
   ```bash
   git status
   git log --oneline -5
   ```

2. TASKS.md の存在と内容を確認する
   ```bash
   ls -la TASKS.md
   cat TASKS.md
   ```

3. TASKS.md が存在しない場合は作成する
   ```bash
   cat > TASKS.md << 'EOF'
   # タスクリスト
   - [ ] 最初のタスクをここに記述する
   EOF
   ```

4. ファイルのアクセス権限を修正する
   ```bash
   chmod 644 TASKS.md
   chmod 755 .
   ```

5. Monitor Loop を再起動する
   ```bash
   # Claude セッションで実行
   > /loop 30m [モニターループ再起動コマンド]
   ```

#### 予防策

- プロジェクト開始前に必ず `git init` と `TASKS.md` 作成を完了させる
- 定期的に `git status` でリポジトリの健全性を確認する
- CI にリポジトリ整合性チェックを組み込む

---

### 1-2. タスクキュー空（処理対象タスクなし）

#### 症状

```
[Monitor] WARNING: Task queue is empty. No actionable items found.
[Monitor] All tasks in TASKS.md are marked as completed [x]
.loop-alert.md が生成され "no_tasks" アラートが記録される
```

#### 原因

- TASKS.md のすべてのタスクが `[x]` 完了済みになっている
- TASKS.md に未完了タスクはあるが、依存関係が未解決で着手不可能
- TASKS.md が誤って上書きされた

#### 解決手順

1. TASKS.md の現状を確認する
   ```bash
   cat TASKS.md
   grep -c '\- \[ \]' TASKS.md   # 未完了タスク数を確認
   grep -c '\- \[x\]' TASKS.md   # 完了済みタスク数を確認
   ```

2. 未完了タスクが本当に存在しない場合は、新しいタスクを追加する
   ```markdown
   - [ ] 次のスプリントのタスクをここに追加する
   ```

3. 依存関係ブロックの場合は、ブロック要因を TASKS.md に明記して再キューイングする
   ```markdown
   - [ ] タスクB（依存: タスクA完了後）
   ```

4. git log で誤上書きがないか確認する
   ```bash
   git log --oneline TASKS.md
   git diff HEAD~1 TASKS.md
   ```

#### 予防策

- 1サイクル分のタスクが常にキューに入るよう、次スプリントのタスクを事前準備する
- TASKS.md を git 管理し、削除・上書き変更を追跡する
- `min_task_threshold: 1` を loop-config.yaml に設定してアラートを早期に出す

---

### 1-3. エスカレーション多発（連続ブロック）

#### 症状

```
[Monitor] ESCALATION: 3 consecutive blocks detected for TASK-007
.loop-alert.md: {"type": "escalation", "task_id": "TASK-007", "count": 3}
Monitor Loop が human_review 状態でスタックする
```

#### 原因

- タスクの要件が不明確・矛盾している
- 設計上の前提が実装と乖離している
- 外部システムへの依存が未解決（APIキー未設定など）
- AGENTS.md の設計方針がコードベースと整合していない

#### 解決手順

1. `.loop-alert.md` の内容を確認して原因を特定する
   ```bash
   cat .loop-alert.md
   ```

2. 問題のタスクを TASKS.md でより詳細に記述し直す
   ```markdown
   - [ ] JWT 認証を実装する
     - POST /api/v1/auth/login（メール + パスワード → JWT 返却）
     - 使用ライブラリ: jsonwebtoken@9.x
     - 参考: src/auth/existing-pattern.ts
   ```

3. AGENTS.md に設計判断を追記して文脈を補強する
   ```markdown
   ## 追加設計判断
   - JWT の有効期限: アクセストークン 1h / リフレッシュ 7d
   ```

4. 外部依存がある場合は環境変数・設定ファイルを確認する
   ```bash
   cat .env.example
   env | grep -i api
   ```

5. Monitor Loop を再起動する（新しいコンテキストで再試行）

#### 予防策

- タスク記述は「最低限の記述」ではなく「推奨の記述」形式を使う（[ワークフローガイド参照](autonomous-development-workflow.md)）
- 外部 API キー・環境変数はセッション開始前に検証する
- 週次で AGENTS.md とコードベースの整合性をレビューする

---

## 2. Build Loop のエラー

### 2-1. ビルド失敗（コンパイルエラー）

#### 症状

```
[Build] ERROR: Build failed after attempt 1
[Build] npm run build → exit code 1
TypeScript: 12 errors in 3 files
.loop-build-handoff.md に "build_failed" が記録される
```

#### 原因

| 原因 | 確認方法 |
|------|---------|
| TypeScript 型エラー | `npx tsc --noEmit 2>&1` |
| 存在しないモジュールの import | `npm run build 2>&1 | grep "Cannot find"` |
| 循環 import | `npx madge --circular src/` |
| Node.js バージョン不一致 | `node --version` と `.nvmrc` を比較 |
| 依存パッケージ未インストール | `npm install` を再実行 |

#### 解決手順

1. ビルドエラーの全文を確認する
   ```bash
   npm run build 2>&1 | tee /tmp/build-error.log
   cat /tmp/build-error.log
   ```

2. TypeScript エラーの場合は個別に修正する
   ```bash
   npx tsc --noEmit 2>&1 | head -50
   ```

3. 依存パッケージを再インストールする
   ```bash
   rm -rf node_modules package-lock.json
   npm install
   npm run build
   ```

4. Node.js バージョンを確認・切り替える
   ```bash
   cat .nvmrc          # 期待バージョン確認
   nvm use             # バージョン切り替え
   node --version      # 現在のバージョン確認
   ```

5. Build Loop の自動修復（最大 15 回）が有効であることを確認する
   ```yaml
   # loop-config.yaml
   build:
     max_retries: 15
   ```

#### 予防策

- `pre-commit` フックで `tsc --noEmit` を実行する
- CI パイプラインに型チェックステップを追加する
- `.nvmrc` または `engines` フィールドで Node.js バージョンを固定する

---

### 2-2. ビルドタイムアウト

#### 症状

```
[Build] ERROR: Build timed out after 15 minutes
[Build] Process killed: npm run build (PID: 12345)
以降のフェーズがすべてスキップされる
```

#### 原因

- ビルド対象ファイルが多すぎる（大規模プロジェクト）
- `node_modules` が壊れており依存解決に時間がかかる
- ループ内でファイル監視が競合している（watch モードの誤起動）
- システムリソース（CPU/メモリ）の不足

#### 解決手順

1. システムリソースを確認する
   ```bash
   free -h               # メモリ確認
   df -h                 # ディスク確認
   nproc                 # CPU コア数確認
   top -bn1 | head -20   # CPU/メモリ使用率確認
   ```

2. ビルドプロセスの重複起動を確認・停止する
   ```bash
   ps aux | grep -E "node|npm|tsc"
   kill -9 <PID>  # 重複プロセスを停止
   ```

3. タイムアウト値を延長する（一時的措置）
   ```yaml
   # loop-config.yaml
   build:
     timeout_minutes: 30  # 15 → 30 に変更
   ```

4. インクリメンタルビルドを有効にする
   ```json
   // tsconfig.json
   {
     "compilerOptions": {
       "incremental": true,
       "tsBuildInfoFile": ".tsbuildinfo"
     }
   }
   ```

5. `node_modules` を再構築する
   ```bash
   rm -rf node_modules .tsbuildinfo
   npm ci
   ```

#### 予防策

- `tsconfig.json` で `incremental: true` を設定する
- CI では `npm ci` を使い `node_modules` をキャッシュする
- 大規模プロジェクトでは `turbo` や `nx` のビルドキャッシュを活用する

---

### 2-3. リトライ上限超過

#### 症状

```
[Build] FATAL: Max retries exceeded (15/15). Escalating to human.
[Build] Last error: Cannot resolve dependency 'some-package'
.loop-alert.md: {"type": "build_fatal", "retries": 15, "last_error": "..."}
```

#### 原因

- エラーが自動修復できない根本問題（設計的な問題）
- 必要なパッケージが npm registry に存在しない
- 環境固有の問題（OS・ランタイムの互換性）
- タスクの要件と既存コードが根本的に矛盾している

#### 解決手順

1. `.loop-alert.md` と最後のビルドログを確認する
   ```bash
   cat .loop-alert.md
   cat .loop-build-handoff.md
   ```

2. エラーを手動で再現して根本原因を特定する
   ```bash
   npm run build 2>&1
   ```

3. パッケージの問題の場合は代替パッケージを調査する
   ```bash
   npm search <alternative-package>
   ```

4. TASKS.md に調査結果と制約条件を追記して再実行する

5. どうしても解決不能な場合はタスクを保留マークにする
   ```markdown
   - [~] 問題のタスク（保留: 理由を記載）
   ```

#### 予防策

- 使用パッケージは事前に `npm install` で動作確認してから TASKS.md に記載する
- AGENTS.md に使用可能・不可能なパッケージのリストを管理する
- `max_retries` は 15 で固定し、超過時に必ず人間レビューを挟む運用にする

---

## 3. Verify Loop のエラー

### 3-1. テスト失敗

#### 症状

```
[Verify] FAIL: 7 tests failed, 43 passed
[Verify] Coverage: 62% (threshold: 80%)
.loop-verify-report.md に失敗テスト一覧が記録される
```

#### 原因

| 原因 | 確認方法 |
|------|---------|
| 実装とテストの期待値が不一致 | 失敗テストの assertion を確認 |
| モック・スタブの設定不備 | `jest.mock()` 呼び出しを確認 |
| 非同期処理の待機漏れ | `await` / `done()` の使用を確認 |
| テスト間の状態汚染 | `beforeEach` でのクリーンアップを確認 |
| 環境変数の未設定 | `process.env` の参照箇所を確認 |

#### 解決手順

1. 失敗テストを個別に実行して詳細を確認する
   ```bash
   npx jest --testNamePattern="失敗しているテスト名" --verbose
   ```

2. テスト環境の変数を確認する
   ```bash
   cat .env.test
   cat jest.config.js
   ```

3. テスト間の状態汚染を調査する
   ```bash
   # テストを単独実行 vs 全体実行で結果が変わるか確認
   npx jest path/to/failing.test.ts
   npx jest --runInBand  # シリアル実行で競合チェック
   ```

4. 自動修復フロー（最大 15 回）が機能しているか確認する
   ```bash
   tail -f .loop-verify-report.md
   ```

5. 自動修復後も解決しない場合は Build Loop に差し戻す
   - `.loop-alert.md` を確認し、修正指示を TASKS.md に追記する

#### 予防策

- `beforeEach` / `afterEach` で必ずテスト状態をリセットする
- テスト用環境変数は `.env.test` に集約し git 管理する（シークレットは除く）
- テストは純粋関数として設計し、外部依存はモック化する

---

### 3-2. カバレッジ不足

#### 症状

```
[Verify] WARNING: Coverage below threshold
  Lines:      62.3% (threshold: 80%)
  Branches:   55.1% (threshold: 70%)
  Functions:  70.2% (threshold: 80%)
Verify Loop が品質ゲートを通過できない
```

#### 原因

- テストが書かれていないコードパスが多い
- エラーハンドリングのコードパスがテストされていない
- Build Loop がテスト実装をスキップした
- カバレッジの計算から除外すべきファイルが含まれている

#### 解決手順

1. カバレッジレポートで未カバーの箇所を特定する
   ```bash
   npx jest --coverage --coverageReporters="text-summary" 2>&1
   open coverage/lcov-report/index.html  # HTML レポートを確認
   ```

2. 未カバーのコードパスを特定する
   ```bash
   npx jest --coverage --coverageReporters="text" 2>&1 | grep "Uncovered"
   ```

3. 不要なファイルをカバレッジ計算から除外する
   ```json
   // jest.config.js
   {
     "coveragePathIgnorePatterns": [
       "/node_modules/",
       "/dist/",
       "*.d.ts",
       "*.config.*"
     ]
   }
   ```

4. Build Loop にカバレッジ改善タスクを追加する
   ```markdown
   - [ ] src/auth/token.ts のカバレッジを 80% 以上に改善する
     - エラーハンドリングのテストケースを追加
     - 境界値テストを追加
   ```

#### 予防策

- `loop-config.yaml` の `coverage_threshold` をプロジェクト初期から現実的な値に設定する
- カバレッジレポートを PR コメントに自動投稿する CI ステップを追加する
- エラーパスのテスト（異常系）を必須要件として TASKS.md テンプレートに含める

---

### 3-3. セキュリティスキャン失敗

#### 症状

```
[Verify] SECURITY: High severity vulnerabilities detected
  npm audit: 3 high, 1 critical
  Semgrep: 2 findings (CWE-79: XSS, CWE-89: SQL Injection)
.loop-verify-report.md に CVE 番号が記録される
```

#### 原因

- 依存パッケージに既知の脆弱性が含まれている（CVE）
- コードに SQL インジェクションや XSS などのセキュリティ問題が含まれている
- シークレット・APIキーがソースコードにハードコードされている
- 古いバージョンのパッケージを使用している

#### 解決手順

1. npm audit の詳細を確認する
   ```bash
   npm audit --json | jq '.vulnerabilities | to_entries[] | select(.value.severity == "high" or .value.severity == "critical")'
   ```

2. 修正可能な脆弱性を自動修正する
   ```bash
   npm audit fix
   npm audit fix --force  # 重大な場合（破壊的変更を含む可能性あり）
   ```

3. Semgrep の指摘箇所を確認する
   ```bash
   cat .loop-verify-report.md | grep -A5 "semgrep"
   ```

4. ハードコードされたシークレットを環境変数に移行する
   ```bash
   # シークレットスキャン
   git grep -n "api_key\|password\|secret\|token" -- "*.ts" "*.js"
   ```

5. Critical / High 脆弱性が残る場合はループを中断してエスカレーションする

#### 予防策

- `npm audit` を CI の必須チェックに組み込む
- `pre-commit` フックで `gitleaks` や `detect-secrets` を実行する
- `.env` ファイルは必ず `.gitignore` に含める
- 依存パッケージは月次で `npm outdated` を確認して更新する

---

## 4. Agent Teams のエラー

### 4-1. エージェント応答なし（タイムアウト）

#### 症状

```
[AgentTeams] ERROR: Agent 'codegen-agent' did not respond within 120s
[AgentTeams] Orchestrator waiting for response from TASK-042...
セッションがハングアップした状態になる
```

#### 原因

- API レート制限に達している
- エージェントが無限ループに入っている
- ネットワーク接続の問題
- プロンプトが複雑すぎて処理時間が超過している

#### 解決手順

1. 現在実行中のプロセスを確認する
   ```bash
   ps aux | grep claude
   ```

2. API のレート制限状況を確認する（ログに `rate_limit` が含まれるか）
   ```bash
   grep -i "rate_limit\|429\|too many" .loop-*.md
   ```

3. セッションを安全に中断して再起動する
   ```bash
   # Ctrl+C でセッションを停止
   # 数分待ってから再起動
   claude --dangerously-skip-permissions
   ```

4. タイムアウト値を調整する
   ```yaml
   # loop-config.yaml
   agents:
     response_timeout_seconds: 180  # 120 → 180
   ```

5. 問題のタスクを小さく分割して再試行する

#### 予防策

- API レート制限のバッファを考慮してエージェント並列数を設定する
- タスクは 1 エージェントが 2 時間以内に完了できる粒度に分割する
- `max_concurrent_agents` を設定してレート制限超過を防ぐ

---

### 4-2. コンテキスト超過（Context Window Overflow）

#### 症状

```
[AgentTeams] ERROR: Context window exceeded for codegen-agent
  Current context: 198,000 tokens
  Limit: 200,000 tokens
コード生成が途中で打ち切られる、または不完全な出力が生成される
```

#### 原因

- コンテキストパッケージに含まれるファイルが多すぎる
- 長大な git diff が丸ごとコンテキストに含まれている
- ループが長時間実行されてコンテキストが蓄積された
- 大きなファイル（生成コードや SQL ダンプ等）が含まれている

#### 解決手順

1. コンテキストパッケージのサイズを確認する
   ```bash
   wc -c .loop-monitor-report.md  # バイトサイズ確認
   wc -l .loop-monitor-report.md  # 行数確認
   ```

2. 不要な大ファイルをコンテキストから除外する
   ```yaml
   # loop-config.yaml
   monitor:
     max_context_files: 10   # 20 → 10 に削減
     max_file_size_kb: 50    # 50KB 以上のファイルを除外
     exclude_patterns:
       - "*.lock"
       - "*.sql"
       - "dist/**"
       - "coverage/**"
   ```

3. セッションを再起動してコンテキストをリセットする（定期的に推奨）

4. タスクをより小さい単位に分割して、1 回の呼び出しのコンテキストを削減する

5. サマリーモードを有効にして過去ログを圧縮する

#### 予防策

- `max_context_files` は 20 以下に保つ
- `.loopignore` ファイルを作成して除外パターンを定義する
- 長時間実行セッションでは 4〜6 時間ごとにセッションを再起動する
- `dist/`, `coverage/`, `*.lock` は常にコンテキスト除外に設定する

---

### 4-3. モデル制限・機能制限エラー

#### 症状

```
[AgentTeams] ERROR: Model refused to execute tool: bash
[AgentTeams] Permission denied: file write to /etc/hosts
エージェントがタスクを実行せずにスキップする
```

#### 原因

- `--dangerously-skip-permissions` オプションなしで起動している
- セキュリティポリシーによる特定ツールの実行制限
- 実行しようとしているコマンドがモデルの安全制限に抵触する
- モデルのバージョン変更により以前使えた操作が制限された

#### 解決手順

1. 起動オプションを確認する
   ```bash
   # 正しい起動コマンド
   claude --dangerously-skip-permissions
   ```

2. 実行しようとした操作の必要性を評価する
   - `/etc/hosts` などのシステムファイル変更は自律モードでは行わない
   - 代替手段（Docker コンテナ内での実行など）を検討する

3. TASKS.md から制限に抵触するタスクを除外または再記述する

4. 必要な操作は人間が手動で実行してから Loop を再起動する

#### 予防策

- 自律モードでは `/etc`, `/usr`, `/sys` への書き込みを行うタスクを記載しない
- セキュリティ上の制限操作は TASKS.md に「手動対応必要」と明記する
- モデルの制限事項は定期的に公式ドキュメントで確認する

---

## 5. Git / PR 操作のエラー

### 5-1. マージコンフリクト

#### 症状

```
[Git] ERROR: Merge conflict in src/auth/token.ts
Auto-merging src/auth/token.ts
CONFLICT (content): Merge conflict in src/auth/token.ts
git push が失敗する
```

#### 原因

- 別のブランチ / 並列実行の Loop が同じファイルを編集した
- リモートの main ブランチが Loop の実行中に更新された
- WorkTree 間でのファイル競合

#### 解決手順

1. コンフリクトの全容を確認する
   ```bash
   git status
   git diff --name-only --diff-filter=U   # コンフリクトファイル一覧
   ```

2. コンフリクトを手動で解決する
   ```bash
   # コンフリクトファイルを編集して <<<< ==== >>>> マーカーを解消
   vim src/auth/token.ts

   # 解決後
   git add src/auth/token.ts
   git commit -m "fix: merge conflict in token.ts を解消"
   ```

3. または、Loop のブランチを最新の main にリベースする
   ```bash
   git fetch origin
   git rebase origin/main
   # コンフリクトを解消しながら rebase を進める
   git rebase --continue
   ```

4. コンフリクト解消後に Loop を再開する

#### 予防策

- 並列 Loop 実行時はタスクのファイル影響範囲を分離する（Domain 分割）
- Loop 開始前に最新の main を取得する（`git pull --rebase`）
- Git WorkTree を使ってブランチを分離する（[Git WorkTree ガイド参照](../architecture/claudeos-loop-spec.md)）

---

### 5-2. プッシュ失敗

#### 症状

```
[Git] ERROR: Failed to push to origin/main
error: failed to push some refs to 'https://github.com/...'
hint: Updates were rejected because the remote contains work that you do not have locally.
```

#### 原因

- リモートブランチが先行してコミットされている（non-fast-forward）
- プッシュ権限がない（認証エラー）
- ブランチ保護ルールに違反している（直接 main へのプッシュが禁止）

#### 解決手順

1. エラーの詳細を確認する
   ```bash
   git push origin main 2>&1
   ```

2. 認証エラーの場合は認証情報を確認する
   ```bash
   git remote -v
   gh auth status     # GitHub CLI の認証状態
   ```

3. non-fast-forward の場合はリベースしてから再プッシュする
   ```bash
   git fetch origin
   git rebase origin/main
   git push origin main
   ```

4. ブランチ保護の場合は PR 経由でマージする
   ```bash
   git push origin HEAD:feature/auto-loop-YYYYMMDD
   gh pr create --title "Auto Loop 実装" --body "自律開発ループによる自動生成"
   gh pr merge --merge
   ```

#### 予防策

- `main` ブランチへの直接プッシュを禁止し、PR フローを必須にする
- Loop の設定で `push_strategy: feature_branch` を使用する
- `gh auth status` でセッション開始前に認証状態を確認する

---

### 5-3. PR マージブロック

#### 症状

```
[Git] ERROR: PR #42 cannot be merged
  - Required status checks failed: ci/build (failed)
  - Required reviews: 0/1 (need 1 approval)
  - Branch is not up to date with main
gh pr merge が exit code 1 で終了する
```

#### 原因

- CI チェックが失敗している
- ブランチ保護ルールで承認レビューが必要
- ベースブランチが更新されてブランチが古くなっている
- コンフリクトが残っている

#### 解決手順

1. PR の状態を詳細に確認する
   ```bash
   gh pr status
   gh pr checks <PR番号>
   ```

2. CI 失敗の場合はログを確認して修正する
   ```bash
   gh run list --limit 5
   gh run view <run-id> --log
   ```

3. ブランチを最新の main に更新する
   ```bash
   git fetch origin
   git merge origin/main   # または git rebase origin/main
   git push origin HEAD
   ```

4. 承認が必要な場合はエスカレーションして人間のレビューを依頼する
   ```bash
   gh pr review --request <reviewer-username>
   # .loop-alert.md にエスカレーション記録
   ```

5. すべての条件が満たされたら再度マージを試みる
   ```bash
   gh pr merge <PR番号> --merge --admin  # 管理者権限がある場合
   ```

#### 予防策

- Loop の完了前に CI が通過していることを Verify Loop で確認する
- 自動承認が可能なリポジトリでは `gh pr merge --auto` を活用する
- ブランチ保護ルールと Loop の設定を整合させる

---

## 6. MCP / 外部連携のエラー

### 6-1. MCP サーバー接続タイムアウト

#### 症状

```
[MCP] ERROR: Connection timeout to mcp-server://filesystem
  Timeout: 30s exceeded
  Retries: 3/3
外部ツール（ファイルシステム・ブラウザ等）が使用不能になる
```

#### 原因

- MCP サーバープロセスが停止している
- ネットワーク設定の問題（ファイアウォール、プロキシ）
- MCP サーバーの過負荷
- Claude の設定ファイルで MCP サーバーのパスが誤っている

#### 解決手順

1. MCP サーバーの状態を確認する
   ```bash
   # claude_desktop_config.json または mcp 設定を確認
   cat ~/.claude/config.json | jq '.mcpServers'
   ```

2. MCP サーバープロセスを再起動する
   ```bash
   # filesystem MCP の例
   pkill -f "mcp-server-filesystem"
   # Claude を再起動すると MCP サーバーも自動起動される
   ```

3. MCP サーバーのログを確認する
   ```bash
   # macOS の場合
   cat ~/Library/Logs/Claude/mcp-*.log 2>/dev/null
   # Linux の場合
   journalctl -u claude-mcp --since "1 hour ago"
   ```

4. 設定ファイルのパスが正しいか確認する
   ```json
   {
     "mcpServers": {
       "filesystem": {
         "command": "npx",
         "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/project"]
       }
     }
   }
   ```

5. MCP なしで動作するタスクに切り替えて作業を継続する

#### 予防策

- MCP サーバーの `health_check` エンドポイントを Loop 開始前に確認する
- MCP 依存処理には必ずフォールバック（MCP なし代替処理）を用意する
- `mcp_retry_policy` を設定してサーバー再起動を自動化する

---

### 6-2. 外部 API 認証失敗

#### 症状

```
[MCP] ERROR: Authentication failed for GitHub API
  Status: 401 Unauthorized
  Message: "Bad credentials"
gh コマンド / API 呼び出しがすべて失敗する
```

#### 原因

- トークンが期限切れになっている
- トークンに必要なスコープ（権限）がない
- 環境変数 `GITHUB_TOKEN` が未設定または誤った値
- IP アドレス制限により認証が拒否されている

#### 解決手順

1. 認証状態を確認する
   ```bash
   gh auth status
   echo $GITHUB_TOKEN | cut -c1-10  # 先頭10文字だけ確認（フルトークン表示禁止）
   ```

2. トークンを再認証する
   ```bash
   gh auth login
   # ブラウザ認証または PAT 入力
   ```

3. 必要なスコープを確認する
   ```bash
   gh auth status  # 付与されているスコープを確認
   # 必要なスコープ: repo, workflow, read:org
   ```

4. 環境変数を再設定する
   ```bash
   export GITHUB_TOKEN="$(gh auth token)"
   ```

5. API が正常に呼び出せるか確認する
   ```bash
   gh api /user | jq '.login'
   ```

#### 予防策

- PAT（Personal Access Token）の有効期限を Loop 実行期間を超えて設定する（最低 30 日）
- セッション開始前に `gh auth status` を必ず確認する
- `GITHUB_TOKEN` の設定は `.env` ではなくシステムのシークレット管理（1Password CLI 等）から取得する
- **重要: トークンをソースコードや TASKS.md に記載しない**

---

## 7. ログ確認・診断コマンド集

### Loop 状態の確認

```bash
# 現在の Loop 状態を一覧表示
ls -la .loop-*.md

# Monitor Loop の最新レポートを確認
cat .loop-monitor-report.md

# Build Loop のハンドオフ情報を確認
cat .loop-build-handoff.md

# Verify Loop の検証結果を確認
cat .loop-verify-report.md

# アラート・エスカレーション情報を確認
cat .loop-alert.md
```

### Git 状態の確認

```bash
# 現在のブランチとコミット履歴
git log --oneline --graph --all -20

# 未コミットの変更
git status
git diff --stat

# リモートとの差分
git fetch origin
git log HEAD..origin/main --oneline

# WorkTree の状態
git worktree list
```

### リソースとプロセスの確認

```bash
# システムリソース
free -h && df -h && uptime

# Node.js / npm 関連プロセス
ps aux | grep -E "node|npm|claude" | grep -v grep

# ポート使用状況
ss -tlnp | grep -E "3000|8080|443"
```

### ビルド・テスト診断

```bash
# ビルドエラーの完全ログ
npm run build 2>&1 | tee /tmp/build-debug.log

# テスト失敗の詳細
npx jest --verbose --no-coverage 2>&1 | tee /tmp/test-debug.log

# TypeScript エラー
npx tsc --noEmit 2>&1 | head -100

# セキュリティスキャン
npm audit --json | jq '.metadata'
```

### ループ再起動の標準手順

問題が解決できない場合の安全な再起動手順：

```bash
# 1. 現在の状態を保存
git add -A
git commit -m "wip: ループ再起動前の状態を保存"

# 2. ループ関連の一時ファイルをクリーンアップ
rm -f .loop-monitor-report.md .loop-build-handoff.md .loop-verify-report.md .loop-alert.md

# 3. 環境を確認
gh auth status
node --version
npm --version

# 4. セッションを再起動
claude --dangerously-skip-permissions
```

---

## 関連ドキュメント

- [自律開発ワークフロー](autonomous-development-workflow.md)
- [ループコマンドリファレンス](loop-command-usage.md)
- [Triple Loop アーキテクチャ](../architecture/triple-loop-architecture.md)
- [ClaudeOS ループ仕様書](../architecture/claudeos-loop-spec.md)
- [Agent Teams システム](../architecture/agent-teams-system.md)
- [ベストプラクティス](../best-practices/autonomous-session-best-practices.md)
