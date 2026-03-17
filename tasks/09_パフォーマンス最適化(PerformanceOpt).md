# 09 パフォーマンス最適化（Performance Optimization）

## 概要

プロファイリングでボトルネックを特定し、DB クエリ・キャッシュ・非同期処理・
バンドルサイズを最適化。ビフォー/アフターのベンチマークで効果を検証します。

---

## Claude Code 起動コマンド

```bash
claude --dangerously-skip-permissions
```

---

## プロンプト指示

```
以下の機能・エンドポイントのパフォーマンスを最適化してください。

【現状の課題】
- 対象: [FILE_PATH / API エンドポイント / ページ名]
- 現在のパフォーマンス:
  - レスポンスタイム: [xx ms（p50 / p95 / p99）]
  - スループット: [xx req/s]
  - バンドルサイズ: [xx KB]（フロントエンドの場合）
  - DB クエリ数: [xx 件/リクエスト]
- 問題の症状: [遅い / メモリ使用量が多い / CPU 負荷が高い]

【実行してほしいこと（Architect / DevAPI Agent が担当）】
1. プロファイリングツールを実行してボトルネックを特定する
   - バックエンド: Node.js Clinic.js / py-spy / Go pprof
   - フロントエンド: Lighthouse / WebPageTest / Chrome DevTools
   - DB: EXPLAIN ANALYZE
2. 特定した問題を優先度順に整理する（ROI が高い順）
3. 最適化計画を提示して承認を得る
4. 以下の観点で最適化を実施する:
   - N+1 クエリの解消（include / join でバッチ取得）
   - キャッシュ（Redis / メモリキャッシュ / HTTP Cache-Control）の導入
   - DB インデックスの追加・最適化
   - 非同期処理・並列処理の活用（Promise.all / バッチ処理）
   - バンドルサイズ削減（コード分割・Tree shaking・lazy loading）
   - 画像・アセットの最適化（WebP / AVIF / CDN）
5. ビフォー / アフターのベンチマーク結果を比較・記録する
6. git commit（perf: [対象] のパフォーマンスを最適化）する

【目標値】
- レスポンスタイム p95: [目標 ms]
- バンドルサイズ: [目標 KB]
- DB クエリ数: [目標 件/リクエスト]
```

---

## 使用場面

| シナリオ | 説明 |
|---------|------|
| ページロードが遅い | LCP / FID / CLS の改善 |
| API レスポンスタイムの改善 | SLA 違反の解消 |
| DB クエリの最適化 | スロークエリの解消 |
| コスト削減 | CPU / メモリ使用量の削減でインフラ費用を下げる |

---

## プロファイリングコマンド

```bash
# Node.js: Clinic.js でプロファイリング
npx clinic doctor -- node src/server.js
npx clinic flame -- node src/server.js  # フレームグラフ

# PostgreSQL: スロークエリの特定
SELECT query, mean_exec_time, calls
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 20;

# フロントエンド: Lighthouse CLI
npx lighthouse https://example.com --output=html --view

# バンドル分析（webpack）
npx webpack-bundle-analyzer dist/stats.json

# k6 で負荷テスト
k6 run --vus=100 --duration=30s load-test.js
```

---

## パフォーマンス最適化チェックリスト

| カテゴリ | チェック項目 |
|---------|------------|
| DB | N+1 解消・インデックス・接続プール |
| API | レスポンスキャッシュ・圧縮（gzip/brotli）・ページネーション |
| Frontend | コード分割・画像最適化・Critical CSS |
| インフラ | CDN・Edge Caching・Auto Scaling |

---

## ポイント

- 「計測 → 分析 → 最適化 → 再計測」のサイクルを自動化する
- 推測で最適化せず、必ずプロファイリング結果に基づく
- ビフォー/アフターの数値を記録することで効果を明確にする
- Triple Loop の Monitor Loop がパフォーマンス劣化を継続監視する
