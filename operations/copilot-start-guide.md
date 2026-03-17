# Copilot Code 起動ガイド（Copilot Start Guide）

## 概要

このガイドでは Claude Code（旧 Copilot Code）を使った自律型開発の起動手順を説明します。
Triple Loop 15H (2-Cycle) システムを中心に、3つの起動方法と運用パターンを網羅します。

---

## 配置構成（CLAUDE.md 統合方式）

```
~/.claude/
└── CLAUDE.md    ← 全プロンプト統合済（Claude Code が自動読み込み）
                    - Triple Loop 15H (2-Cycle) システムプロンプト本体
                    - Monitor Loop 指示
                    - Build Loop 指示（5段階開発ステップ）
                    - Verify Loop 指示
                    - Agent Teams 構成（8 Agent）
                    - 実行権限マトリクス
                    - /loop 起動コマンド一覧

プロジェクト/
├── TASKS.md     ← タスク一覧（Claude が自動更新）
├── AGENTS.md    ← 設計判断・学習記録（Claude が自動生成）
└── docs/        ← 作業日報の出力先
```

---

## 事前準備（必須）

### 1. CLAUDE.md の配置

```bash
# ~/.claude/ ディレクトリが存在しない場合
mkdir -p ~/.claude

# CLAUDE.md を配置（Triple Loop システムプロンプトを含む）
# → 00_フル自律開発起動(FullAutoStart).md の
#   「▼ここからコピー〜▲ここまでコピー」セクションを貼り付け
vim ~/.claude/CLAUDE.md
```

### 2. TASKS.md の作成

```markdown
# Project Tasks

## Priority: High
- [ ] [タスク1の説明]
- [ ] [タスク2の説明]

## Priority: Medium
- [ ] [タスク3の説明]

## Completed
```

### 3. Git リポジトリの確認

```bash
git status       # git init 済みであることを確認
git remote -v    # リモートが登録されていることを確認
```

---

## 起動方法 A: /loop コマンド（推奨・最も簡単）

### 手順

```
Step 1: ターミナルを開く
Step 2: プロジェクトフォルダに移動
        $ cd /path/to/your/project
Step 3: Claude Code を自動承認モードで起動
        $ claude --dangerously-skip-permissions
Step 4: TASKS.md の確認 or 作成
Step 5: /loop コマンドを貼り付けて Enter
Step 6: 放置（終了まで人間は何もしなくてOK）
```

### /loop コマンド一覧

**2サイクル15時間（推奨・2タスク/日）:**
```
/loop 900m 以下の3重ループを2サイクル実行してください。1サイクル=7時間、サイクル間CD=30分、最終処理=30分、合計15時間で2サイクル完走すること。全操作を自動承認で続行。

【サイクル構成（×2回繰り返し）】
1. 🔍 Monitor（30分）→ 本ファイル内の「Monitor Loop 指示」に従い実行
2. ⏸ 10分待機
3. 🔨 Build（2時間）→ 本ファイル内の「Build Loop 指示」に従い5段階開発ステップを実行。各Step完了ごとにmainへcommit
4. ⏸ 20分待機（commit時間含む）
5. 🧪 Verify（4時間）→ 本ファイル内の「Verify Loop 指示」に従いレビュー・テスト・デバッグ修正を実行。修正はmainへcommit
6. ⏸ CD3 30分待機（サイクル間クールダウン、2サイクル目完了後は不要）
7. 🏁 最終処理（30分）→ push / PR作成 / PRマージ / 作業日報出力（2サイクル目の後のみ）

【終了条件】
- 15時間経過 or 2サイクル完走
- 最終処理で git push → PR作成 → PRマージ を実行
- 停止直前に docs/[YYYY-MM-DD]_自律開発作業報告.md（日本語Markdown）を出力
- リミットに到達した場合はその時点で最終処理に移行し、残タスクは TASKS.md に記録

【参照ファイル】TASKS.md / AGENTS.md
```

**1サイクル8時間（軽量運用・1タスク/日）:**
```
/loop 450m 以下の3重ループを1サイクルとして実行してください。1サイクル=7時間30分、8時間内で1サイクル完走すること。全操作を自動承認で続行。

【サイクル構成】
1. 🔍 Monitor（30分）→ 本ファイル内の「Monitor Loop 指示」に従い実行
2. ⏸ 10分待機
3. 🔨 Build（2時間）→ 本ファイル内の「Build Loop 指示」に従い5段階開発ステップを実行
4. ⏸ 20分待機
5. 🧪 Verify（4時間）→ 本ファイル内の「Verify Loop 指示」に従いレビュー・テスト・デバッグ修正を実行
6. �� 最終処理（30分）→ push / PR作成 / PRマージ / 作業日報出力

【参照ファイル】TASKS.md / AGENTS.md
```

**Monitor のみ（現状把握だけ）:**
```
/loop 30m 本ファイル内の「Monitor Loop 指示」に従い、プロジェクトの現状把握を実行してください。コード変更は一切行わないこと。結果を .loop-monitor-report.md に出力。
```

**Build のみ（実装だけ）:**
```
/loop 120m 本ファイル内の「Build Loop 指示」に従い、TASKS.mdから優先度順にタスクを選択し5段階開発ステップを実行してください。各Step完了ごとにmainへcommit。全操作を自動承認で続行。
```

**Verify のみ（レビュー・テストだけ）:**
```
/loop 240m 本ファイル内の「Verify Loop 指示」に従い、.loop-build-handoff.mdを読み込みBuild成果物をレビュー・テスト・デバッグ修正してください。修正はmainへcommit。
```

---

## 起動方法 B: シェルスクリプト（ログ保全・CI/CD 向け）

```bash
# triple-loop-15h.sh をプロジェクトルートに配置
chmod +x triple-loop-15h.sh
bash triple-loop-15h.sh
```

**方法 A との違い:**

| 比較項目 | 方法 A: /loop | 方法 B: シェルスクリプト |
|---------|--------------|------------------------|
| 手軽さ | 簡単（2ステップ） | やや手間 |
| ログ保存 | Claude セッション内 | `.loop-logs/` に自動保存 |
| エラー復旧 | Claude が判断 | シェルで制御可能 |
| CI/CD 連携 | 難しい | 可能 |
| 推奨用途 | 通常の日次運用 | ログ保全・CI 連携が必要な場合 |

---

## 起動方法 C: タスク別プロンプト（単発作業）

15時間フル運用不要のとき、`tasks/` フォルダのプロンプトを単発実行：

```bash
claude --dangerously-skip-permissions
# → tasks/01_新規プロジェクト初期化.md などのプロンプトを貼り付け
```

---

## 15時間中のタイムライン

```
時間   | 何が起きるか                          | あなたがやること
──────┼──────────────────────────────────────┼────────────────
 0:00 | Cycle 1 Monitor: 現状把握             | 何もしない
 0:40 | Cycle 1 Build: タスク自動実装         | 何もしない
       | → main に自動 commit（5回/タスク）   |
 3:00 | Cycle 1 Verify: レビュー＆テスト      | 何もしない
       | → バグがあれば修正＆再 commit        |
 7:00 | 30分休憩（サイクル間クールダウン）    | 何もしない
 7:30 | Cycle 2 Monitor: 差分把握             | 何もしない
 8:10 | Cycle 2 Build: タスク自動実装         | 何もしない
10:30 | Cycle 2 Verify: レビュー＆テスト      | 何もしない
14:30 | 最終処理                              |
       | → git push / PR 作成 / PR マージ    |
       | → docs/ に作業日報出力              |
15:00 | 自動停止                              | 日報を確認する
```

---

## 終了後の確認ポイント

```
1. docs/[日付]_自律開発作業報告.md  → 2サイクル分の詳細レポート
2. git log --oneline                → 15時間分の全 commit 履歴
3. TASKS.md                         → 完了タスクに ✅ がついている
4. AGENTS.md                        → 設計判断・学習事項の記録
5. GitHub の PR                     → マージ済みの Pull Request
```

---

## トラブルシューティング

| 症状 | 対処 |
|-----|------|
| 途中で止めたい | `Ctrl+C` で即座に停止。commit 済みの変更は保持される |
| TASKS.md を途中で編集したい | Monitor Loop の時間帯（30分）またはサイクル間 CD（30分）が安全 |
| エラーで止まった | CI 修復 AI が最大15回自動修復。それでも無理なら TASKS.md に記録して次タスクへ |
| リミットに到達した | 自動的に最終処理に移行。未完了タスクは TASKS.md に記録 |

---

## 関連ドキュメント

- [ループコマンドリファレンス](loop-command-usage.md)
- [自律開発ワークフロー](autonomous-development-workflow.md)
- [Triple Loop アーキテクチャ](../architecture/triple-loop-architecture.md)
- [タスクプロンプト一覧](../tasks/README.md)
