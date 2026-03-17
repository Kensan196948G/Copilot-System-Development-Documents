# 11 フロントエンド開発（Frontend Development）

## 概要

UI コンポーネント・画面を実装。レスポンシブ・アクセシビリティ・パフォーマンス
（Lighthouse 90+）を基準に、テストコード付きで納品します。

---

## Claude Code 起動コマンド

```bash
claude --dangerously-skip-permissions
```

---

## プロンプト指示

```
以下の UI コンポーネント / 画面を実装してください。

【実装対象】
- フレームワーク: [React / Next.js / Vue 3 / Svelte / Nuxt など]
- UI ライブラリ: [Tailwind CSS / shadcn/ui / MUI / Ant Design / Radix UI など]
- 状態管理: [Zustand / Redux Toolkit / Pinia / Jotai / TanStack Query など]
- 画面 / コンポーネント名: [NAME]
- デザイン参考: [Figma URL / スクリーンショット / テキスト説明]

【実装要件（DevUI / QA Agent が担当）】
- レスポンシブデザイン（モバイル: 375px / タブレット: 768px / デスクトップ: 1280px）
- アクセシビリティ（WCAG 2.1 AA 準拠）
  - 適切な ARIA 属性
  - キーボードナビゲーション
  - カラーコントラスト比 4.5:1 以上
- UI 状態の完全な実装:
  - Loading（スケルトン UI を使用）
  - Error（ユーザーフレンドリーなエラーメッセージ）
  - Empty（空状態のイラスト付き案内）
  - Success
- アニメーション / トランジション（Framer Motion / CSS Transition）

【実行してほしいこと】
1. コンポーネント設計を提示する
   - Props インターフェース（TypeScript）
   - 状態設計（useState / useReducer / store）
   - コンポーネント分割方針
2. コンポーネントを実装する
3. Storybook ストーリーを作成する（Storybook が導入済みの場合）
4. テストを作成する
   - スナップショットテスト（UI の見た目の回帰防止）
   - インタラクションテスト（@testing-library/user-event）
5. アクセシビリティチェック（axe-core）を実行して指摘を修正する
6. Lighthouse でスコアを確認する（Performance / Accessibility / Best Practices）
7. git commit（feat: [コンポーネント名] を実装）する

【完了基準】
- Lighthouse スコア 90 以上（Performance / Accessibility / Best Practices）
- テストがすべてパス
- 既存コンポーネントとデザインが統一されている
```

---

## 使用場面

| シナリオ | 説明 |
|---------|------|
| 新規画面の実装 | ページ単位での UI 実装 |
| デザインシステムの構築 | 再利用可能なコンポーネントライブラリ |
| UI のリニューアル | 既存画面の現代化・UX 改善 |
| インタラクティブダッシュボード | チャート・テーブルを含む複雑な画面 |

---

## デザイン参考の渡し方

```
# Figma の場合
「Figma URL: https://figma.com/... の "UserProfile" フレームを実装してください」

# スクリーンショットの場合
「[画像] この画面をそのまま実装してください」

# テキスト説明の場合
「ユーザープロフィール画面:
  - ヘッダー: アバター（左）+ 名前・メールアドレス（右）
  - タブ: プロフィール / 設定 / 通知
  - プロフィールタブ: 自己紹介文（編集可）+ SNS リンク
  - モバイルではタブがボトムナビになる」
```

---

## アクセシビリティ確認コマンド

```bash
# axe-core で自動チェック
npx axe http://localhost:3000 --tags wcag2a,wcag2aa

# Lighthouse CI
npx lhci autorun --config=.lighthouserc.json

# キーボードナビゲーション手動確認
Tab / Shift+Tab でフォーカス移動 → 全インタラクション要素に到達できるか
Enter / Space でボタン・リンクが実行できるか
```

---

## ポイント

- デザイン参考を具体的に与えるほど意図に近い実装が生成される
- `shadcn/ui` + `Tailwind CSS` の組み合わせが現在最も生成精度が高い
- アクセシビリティは後から直すのが大変なので最初から組み込む
- Triple Loop の Verify Loop で UI の回帰テストが自動実行される
