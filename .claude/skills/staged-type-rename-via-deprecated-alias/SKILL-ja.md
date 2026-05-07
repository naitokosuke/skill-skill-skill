---
name: staged-type-rename-via-deprecated-alias
description: TypeScript で型を widen しつつ rename する際、一括置換が不可能なケースで @deprecated alias を使い段階的に移行する。型名が enum メンバー等と衝突して正規表現置換ができないとき、または広い型を別名で導入し旧名を narrow 型の alias として残したいときに使う。
---

# Staged Type Rename via Deprecated Alias

型を rename しつつ意味を広げる (widen) 場面で、一括置換が enum メンバー等との衝突で不可能なときに使う段階的移行戦略。

## いつ使うか

- 既存の型名を、より広い union 型に置き換えたい
- ただし旧名が enum メンバー (`Foo.Bar`) や他の識別子と衝突しており、正規表現ベースの一括 rename が壊れる
- 移行中も常に `pnpm check` (type check) が通る状態を保ちたい

## 重要な原則

### 旧名は narrow 型への alias にする (broader 型への alias にしない)

旧名を broader 型の alias にすると、既存コードが broader 型を受け取ってしまい exhaustive switch が壊れる。広げる前の narrow 型を新名として導入し、旧名はその新名への alias にする。broader 型は別名で公開する。

### alias は単純な型等価に限る

`type A = B` は OK。`type A = Omit<B, ...>` のような変形を alias に入れると意味が変わるので不可。

### 関数の alias は `const foo = bar` で十分

関数式そのものを代入できる。シグネチャが一致する限り型もそのまま引き継がれる。

### `@deprecated` JSDoc で警告を出す

TypeScript コンパイラは `@deprecated` を特別扱いし、`--strict` 等の設定に関わらず使用箇所に deprecation 情報を流す。ESLint の `deprecation/deprecation` ルールや IDE がこれを拾って警告表示する。

## 移行手順

### Step 1: narrow 新名と deprecated alias を導入

broader 型を一時的な内部名 (`_AllX` 等) で定義し、そこから narrow 新名を Exclude で導出。旧名は narrow 新名への `@deprecated` alias にする。

```ts
// entities.ts
export type _AllQuestion = // 一時的な内部名 (後で Question に戻す)
  | ExerciseQuestion
  | CorrectionPracticeQuestion
  | GradedScorePracticeQuestion
  | PronunciationScoredPracticeQuestion;

export type ScorePersistedQuestion = Exclude<_AllQuestion, PronunciationScoredPracticeQuestion>;

/** @deprecated use ScorePersistedQuestion */
export type Question = ScorePersistedQuestion;
```

この状態で `pnpm check` が通ることを確認 → commit。

### Step 2: 使用箇所を新名に置換

app 層の `Question` 使用箇所を `ScorePersistedQuestion` に順次置換する。enum メンバー等との衝突があるため、IDE の symbol rename か文脈を見ての手作業置換を行う。

### Step 3: alias を削除し内部名を旧名に戻す

全置換完了後:

- `export type Question = ScorePersistedQuestion` の alias を削除
- `_AllQuestion` を `Question` に rename し戻す (broader 型として正規化)

これで「広げた variant も含んだ `Question`」が正規に成立し、旧コードは `ScorePersistedQuestion` を使う状態になる。
