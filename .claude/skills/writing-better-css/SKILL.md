---
name: writing-better-css
description: Use when writing or refactoring CSS in a Baseline 2024+ browser-target project — choosing native CSS features (`:has()`, container queries, `@layer`, `color-mix()` / `oklch()`, native nesting, view transitions, anchor positioning, scroll-driven animations, `@property`, `text-wrap: balance`, popover, logical properties, subgrid) over legacy substitutes (JS state for open/close, media-query-only responsive, SCSS nesting, hex/rgb-only colors, IntersectionObserver reveal animations, JS-positioned tooltips).
---

# Writing Better CSS

## Overview

ブラウザに既に実装されている CSS 機能を、JS / プリプロセッサ / 親クラス・トグル等の代替手段より優先するためのリファレンス
多くは Baseline 2023〜2024 で widely available

Core principle として、「JS で状態管理」「親要素にクラス追加」「プリプロセッサでネスト」を書こうとしたら、まず CSS だけで書けないか確認する

## When to Use

- 新規 CSS を書くとき、まず本スキルの Quick Reference に該当パターンがないか確認する
- レガシーパターンを見つけたとき（下記）置き換えを検討する
  - JS の状態フラグ + className トグルでの open/close 表現
  - `window.matchMedia` / リサイズ監視でのレイアウト分岐（コンポーネント幅基準）
  - SCSS / Less の `&` ネスト（CSS native nesting で代替可）
  - `#rrggbb` / `rgba()` のみで色を表現（`oklch()` / `color-mix()` で意図が明確になる場合）
  - Intersection Observer でのフェードイン（scroll-driven animations で代替可）
  - JS で position 計算しているツールチップ・ポップオーバー（anchor positioning + popover API）

## When NOT to Use

- IE / 古いブラウザサポートが必須のプロジェクト
- 機能の Baseline が新しすぎてターゲットに届かない場合（個別に caniuse 確認）
- プロジェクトに既存の design system / utility framework があり、一貫性が優先される場合

## Quick Reference

レガシーから推奨への対応一覧

- 子要素の状態で親をスタイル
  - レガシー: JS で親に class を付与
  - 推奨: `:has()`
- コンポーネント幅で分岐
  - レガシー: `@media` (viewport)
  - 推奨: `@container`
- グローバル CSS の優先度戦争
  - レガシー: `!important` / BEM
  - 推奨: `@layer`
- 色の透明化・ミックス
  - レガシー: `rgba()` / 手計算
  - 推奨: `color-mix(in oklch, ...)`
- 一貫した知覚色相
  - レガシー: HSL
  - 推奨: `oklch()` / `oklab()`
- ページ遷移アニメ
  - レガシー: JS ライブラリ
  - 推奨: View Transitions API
- ツールチップ / メニュー位置
  - レガシー: JS で getBoundingClientRect
  - 推奨: CSS Anchor Positioning
- 行儀よく重なるグリッド子
  - レガシー: 各セルで再定義
  - 推奨: `subgrid`
- 複数セレクタの DRY
  - レガシー: カンマ羅列
  - 推奨: `:is()` / `:where()`
- LTR/RTL 両対応
  - レガシー: `margin-left` 個別管理
  - 推奨: logical props (`margin-inline-start`)
- プリプロセッサのネスト
  - レガシー: SCSS
  - 推奨: native CSS nesting
- スクロール連動アニメ
  - レガシー: IntersectionObserver
  - 推奨: `animation-timeline: view()`
- カスタムプロパティの型安全
  - レガシー: 文字列のみ
  - 推奨: `@property`
- 見出しの折り返し最適化
  - レガシー: `<br>` 手挿入
  - 推奨: `text-wrap: balance`
- 段落の最終行最適化
  - レガシー: 手調整
  - 推奨: `text-wrap: pretty`
- input サイズを内容に合わせる
  - レガシー: JS
  - 推奨: `field-sizing: content`
- `height: auto` への transition
  - レガシー: JS で実高さ計算
  - 推奨: `interpolate-size: allow-keywords`
- モーダル・ドロップダウン
  - レガシー: JS state 管理
  - 推奨: `popover` 属性 + `::backdrop`
- 親に依存しないスコープ
  - レガシー: id ネスト
  - 推奨: `@scope`

## Core Patterns

### 1. `:has()` — 親セレクタ

```css
/* 子に input:invalid がある form をハイライト */
form:has(:invalid) { border-color: var(--danger); }

/* チェックされた input を含む label */
label:has(:checked) { background: var(--accent); }

/* figure に figcaption がある場合だけ余白追加 */
figure:has(figcaption) { padding-block-end: 1rem; }
```

JS で「子が変わったら親に class」を書きたくなったら、ほぼ `:has()` で済む

### 2. Container Queries

```css
.card { container-type: inline-size; container-name: card; }

@container card (inline-size > 30rem) {
  .card__layout { display: grid; grid-template-columns: auto 1fr; }
}
```

ビューポートではなくコンポーネント自身の幅で分岐する
再利用可能なコンポーネントに必須

### 3. Cascade Layers (`@layer`)

```css
@layer reset, base, components, utilities;

@layer base { h1 { font-size: 2rem; } }
@layer components { .btn { padding: .5rem 1rem; } }
@layer utilities { .mt-0 { margin-top: 0; } }
```

レイヤー順で優先度が決まり、詳細度の戦争を回避できる
`!important` を使う前に `@layer` を検討

### 4. `color-mix()` / `oklch()`

```css
:root {
  --brand: oklch(70% 0.18 150);
  --brand-soft: color-mix(in oklch, var(--brand) 20%, transparent);
  --brand-hover: color-mix(in oklch, var(--brand), white 12%);
}
```

- `oklch(L C H)`: 知覚的に均一な色空間
  - 同じ L で複数色を作ると明度感が揃う
- `color-mix()`: 透明色・hover の暗色化・テーマ補間が一行で書ける

### 5. Native Nesting

```css
.card {
  padding: 1rem;
  & > h2 { margin: 0; }
  &:hover { background: color-mix(in oklch, currentColor 5%, transparent); }
  @media (width > 40rem) { padding: 2rem; }
}
```

descendant は `&` 不要
親自身に擬似クラス・複合セレクタを付ける場合（`&:hover` / `&.active`）は `&` 必要

### 6. View Transitions API

```css
::view-transition-old(root),
::view-transition-new(root) {
  animation-duration: 0.4s;
}

.hero { view-transition-name: hero; }
```

```js
document.startViewTransition(() => updateDOM());
```

SPA / MPA 両対応の遷移アニメ
フレームワーク側で view transition を有効化するオプションを持つものもある（Nuxt の `experimental.viewTransition` など）

### 7. Anchor Positioning

```css
.tooltip-trigger { anchor-name: --trigger; }
.tooltip {
  position: absolute;
  position-anchor: --trigger;
  top: anchor(bottom);
  left: anchor(center);
  translate: -50% 8px;
  position-try-fallbacks: flip-block, flip-inline;
}
```

JS で位置計算する Floating UI 系の処理が CSS だけで書ける（Chrome 125+, Baseline 未到達なので要確認）

### 8. Subgrid

```css
.list { display: grid; grid-template-columns: auto 1fr auto; }
.list > li {
  display: grid;
  grid-template-columns: subgrid;
  grid-column: 1 / -1;
}
```

子グリッドが親のトラックを継承
カードの行揃えに有効

### 9. Scroll-driven Animations

```css
@keyframes reveal {
  from { opacity: 0; translate: 0 2rem; }
  to   { opacity: 1; translate: 0 0; }
}
.section {
  animation: reveal linear both;
  animation-timeline: view();
  animation-range: entry 0% entry 60%;
}
```

`view()` は要素自身がビューポートに入る進捗
`scroll()` はルートスクロール進捗

### 10. `@property`

```css
@property --angle {
  syntax: "<angle>";
  inherits: false;
  initial-value: 0deg;
}
.spin { --angle: 0deg; transition: --angle 1s; }
.spin:hover { --angle: 360deg; }
```

カスタムプロパティに型を付与
これがないと `--angle` の transition は補間されない

### 11. `text-wrap: balance` / `pretty`

```css
h1, h2 { text-wrap: balance; }   /* 見出しを均等な行幅に */
p      { text-wrap: pretty; }    /* 最終行が孤立しないよう調整 */
```

### 12. `field-sizing: content`

```css
textarea, input { field-sizing: content; min-inline-size: 10ch; }
```

JS なしで内容に合わせて伸縮

### 13. `interpolate-size`

```css
:root { interpolate-size: allow-keywords; }
details::details-content {
  block-size: 0;
  transition: block-size 0.3s, content-visibility 0.3s allow-discrete;
}
details[open]::details-content { block-size: auto; }
```

`auto` への transition が可能になる

### 14. Popover API

```html
<button popovertarget="menu">Menu</button>
<div id="menu" popover>...</div>
```

```css
[popover] { &::backdrop { background: color-mix(in oklch, black 40%, transparent); } }
[popover]:popover-open { animation: fade-in 0.2s; }
```

軽量メニュー・ツールチップを JS state なしで実装
light-dismiss 自動

### 15. `@scope`

```css
@scope (.article) to (.comments) {
  :scope { color: var(--text); }
  a { color: var(--link); }
}
```

スコープの上限と下限を指定できる

### 16. Logical Properties

```css
.box {
  margin-inline: auto;          /* margin-left + margin-right */
  padding-block: 1rem;          /* padding-top + padding-bottom */
  border-inline-start: 2px;     /* LTR で left, RTL で right */
}
```

国際化と縦書きに自然対応
新規コードはこちらを default に

### 17. `:is()` / `:where()`

```css
:is(h1, h2, h3) + p { margin-top: 0; }
:where(ul, ol) li { line-height: 1.5; }   /* :where は詳細度 0 */
```

`:where()` はリセット・base レイヤで上書きしやすくするのに最適

## Browser Support の確認

採用前に必ず caniuse / Baseline を確認

```
https://caniuse.com/<feature>
https://web.dev/baseline
```

Baseline ステータス

- Widely available: 30 ヶ月以上前から全主要ブラウザで使える（採用 OK）
- Newly available: 全主要ブラウザで使えるが新しい（フォールバック検討）
- Limited: 一部ブラウザのみ（progressive enhancement で限定使用）

`@supports` でフォールバック

```css
@supports (anchor-name: --x) {
  .tooltip { /* anchor positioning 版 */ }
}
```

## Lightning CSS の有無を確認

書く前に、プロジェクトが [Lightning CSS](https://lightningcss.dev/) を使っているかを確認する
Lightning CSS は browserslist targets に基づいて native nesting / `color-mix()` / `oklch()` / logical props 等を自動 downlevel する
使用していればブラウザターゲットが古めでも採用できる機能の幅が広がる

確認方法（順に当てはめる）

```bash
# 1. 依存に lightningcss があるか
rg -n '"lightningcss"' package.json

# 2. Vite: css.transformer === "lightningcss" / css.lightningcss を見る
rg -n "lightningcss|transformer" vite.config.* nuxt.config.*

# 3. PostCSS 経由 (postcss-preset-env / browserslist) も同種の役割
rg -n "postcss-preset-env|browserslist" package.json .browserslistrc 2>/dev/null
```

判断

- Lightning CSS あり + browserslist targets が広い
  - ソースは新しい構文で書いてよい
  - 出力は targets に合わせて downlevel される
- Lightning CSS なし
  - `@supports` フォールバックや caniuse 確認をより厳密に行う
- Vite 6+ で `css.transformer: "lightningcss"` 明示
  - minify と downlevel の両方が有効

注意として、Lightning CSS は `:has()` や container queries のようなランタイム機能はポリフィルできない（構文の lowering のみ）
これらは引き続き Baseline で判定する

## Common Mistakes

- `@container` を使うのに `container-type` を設定し忘れる
  - 親に `container-type: inline-size` 必須
- native nesting で親自身への擬似クラスから `&` を省略（`.btn { :hover {} }`）
  - descendant の hover になってしまう
  - 親自身は `&:hover`
- `oklch()` のクロマを HSL 感覚で大きくする
  - C は 0〜0.4 程度が実用域
  - 0.18 前後が彩度標準
- `@property` なしで CSS variable を transition
  - 補間できない
  - `@property` で `syntax` 宣言必要
- `view-transition-name` を複数要素に同名で付与
  - 同一ページ内で一意でなければ動作しない
- `:has()` を多用してパフォーマンス劣化
  - スタイル計算が重くなる
  - 複雑なネストは避ける
- logical props と物理 props の混在
  - プロジェクト全体で一方に統一する
- Container Queries に `container-name` を付け忘れて意図しない親にマッチ
  - named container を使う

## Red Flags（置き換え候補）

書いている / レビューしているコードに以下があれば、本スキルの Quick Reference を再確認

- JS の boolean state + open/close className トグル
- `window.matchMedia` でのレイアウト分岐
- `getBoundingClientRect` でのポップアップ位置計算
- `IntersectionObserver` でのフェードイン
- SCSS / Sass の `&` ネスト（プリプロセッサ依存を外す好機）
- `rgba(0, 0, 0, 0.5)` の散在（`color-mix` でデザイントークン化）
- `!important` の連発（`@layer` で解決）
- `<br>` での改行調整（`text-wrap: balance`）

## Reference Links

- web.dev Baseline: https://web.dev/baseline
- MDN CSS: https://developer.mozilla.org/en-US/docs/Web/CSS
- caniuse: https://caniuse.com
- State of CSS (年次): https://stateofcss.com
