---
name: staged-type-rename-via-deprecated-alias
description: Use when widening and renaming a TypeScript type but a bulk regex rename is impossible — staged migration via a `@deprecated` alias. Apply when the old name collides with enum members or other identifiers (so regex rename breaks), or when introducing a wider type under a new name and keeping the old name as an alias of the narrower type.
---

# Staged Type Rename via Deprecated Alias

A staged migration strategy for renaming a type while widening its meaning, in cases where bulk replacement is impossible due to collisions with enum members and the like.

## When to use

- You want to replace an existing type name with a wider union type
- But the old name collides with enum members (`Foo.Bar`) or other identifiers, breaking regex-based bulk rename
- You want `pnpm check` (type check) to keep passing throughout the migration

## Key principles

### The old name should alias the narrow type, not the broader type

If the old name aliases the broader type, existing call sites end up receiving the broader type and exhaustive switches break. Introduce the pre-widening narrow type under the new name, and make the old name an alias of that new name. Expose the broader type under a separate name.

### Aliases must be plain type-equivalence

`type A = B` is fine. Putting transformations like `type A = Omit<B, ...>` into the alias changes the meaning, so it isn't allowed.

### For functions, `const foo = bar` is enough

You can assign the function expression directly. As long as the signature matches, the type carries over.

### Use `@deprecated` JSDoc to surface warnings

The TypeScript compiler treats `@deprecated` specially and propagates deprecation info to call sites regardless of `--strict` and similar settings. ESLint's `deprecation/deprecation` rule and IDEs pick this up and show warnings.

## Migration steps

### Step 1: introduce the narrow new name and the deprecated alias

Define the broader type under a temporary internal name (e.g. `_AllX`) and derive the narrow new name from it via `Exclude`. Make the old name a `@deprecated` alias of the narrow new name.

```ts
// entities.ts
export type _AllQuestion = // temporary internal name (will be renamed back to Question later)
  | ExerciseQuestion
  | CorrectionPracticeQuestion
  | GradedScorePracticeQuestion
  | PronunciationScoredPracticeQuestion;

export type ScorePersistedQuestion = Exclude<_AllQuestion, PronunciationScoredPracticeQuestion>;

/** @deprecated use ScorePersistedQuestion */
export type Question = ScorePersistedQuestion;
```

Confirm `pnpm check` passes in this state → commit.

### Step 2: replace usages with the new name

Replace occurrences of `Question` in the app layer with `ScorePersistedQuestion`, gradually. Because of collisions with enum members and similar, do this with the IDE's symbol rename or context-aware manual replacement.

### Step 3: remove the alias and rename the internal name back to the old name

After all replacements are done:

- Delete the `export type Question = ScorePersistedQuestion` alias
- Rename `_AllQuestion` back to `Question` (canonicalizing it as the broader type)

This properly establishes "`Question` including the widened variants", while the old code now uses `ScorePersistedQuestion`.
