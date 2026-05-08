---
name: prevent-ugly-fmt
description: Use when a source-code formatter (prettier, biome, rustfmt, black, gofmt, etc.) produces output that feels awkward or hard to read. Restructure the source so the formatter's natural output is clean, instead of silencing the formatter.
---

# prevent-ugly-fmt

## Principle

When a formatter's output looks awkward, the cause is **a mismatch between the source structure and the formatter's rules**. Fix the source side with a minimal rewrite so the formatter naturally produces clean output. Don't suppress the formatter (ignore comments) or hide the problem in another medium (CSS pseudo-elements, images, etc.).

## Procedure

1. **Name the awkward shape as a pattern.** e.g. "the closing `>` sits alone on the next line", "the `?` and `:` of a ternary don't line up with the conditions they belong to", "the method chain breaks at points that don't match the logical phases".
2. **Locate the smallest unit producing it** in the source — usually a single expression, a single element, or a few constructs adjacent on one line.
3. **Rewrite that unit, preserving meaning,** with one of:
   - Separate adjacent things (insert a line break, place on its own line)
   - Extract an intermediate variable / intermediate element
   - Decompose a compound expression (conditional → if/else, nested ternary → early return)
   - Move whitespace control to the parent (e.g. `display: flex` on the parent so the inter-child whitespace is dropped, while semantic equivalence is preserved structurally)
4. **Re-run the formatter** (whatever the project's wrapper is) and inspect. If the awkward shape recurs, try a different rewrite.

## Verification

Run the formatter after every change. A construct that came out clean once can break again when neighbouring code changes.

## Tempting alternatives that just shift the cost

| Alternative | Cost it shifts to |
|---|---|
| `// prettier-ignore`, `// fmt: skip`, etc. | Project conventions usually forbid them; another formatter may not honour them; reviewers are surprised |
| Move the visible string into a CSS `content` / image / annotation | The text drops out of copy, search, and screen-reader reach |
| Layouts that depend on the data shape (length, count, etc.) | Breaks the moment the data changes |
| "Visually equivalent" alternative structures (e.g. two adjacent inline elements where one was) | The same formatter rule re-breaks them; the underlying structural conflict is unchanged |

## Worked example: Prettier-style HTML formatter

Input `<dt><b>X</b>/total</dt>` gets reformatted to `</b\n>/total` because Prettier treats inline-element + adjacent text as whitespace-sensitive and refuses to insert whitespace. **Resolve the conflict by placing the adjacent text on its own line:**

```html
<dt>
  <span class="count">X</span>
  /total
</dt>
```

Now the closing `>` sits at the end of a line and the trailing text is a child text node — accessibility, copy, and search all keep working.

(Note: HTML whitespace collapsing means the new layout introduces one space character between the element and the trailing text. If you need *no* visual gap, also drop `display: flex` on the parent — children separated by source whitespace alone are visually adjacent under flex.)

## Generalisation

The specific "broken shape" differs by formatter, but the procedure does not: **observe the shape → rewrite the source structure → re-run the formatter**.
