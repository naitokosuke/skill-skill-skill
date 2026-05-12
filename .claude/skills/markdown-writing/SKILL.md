---
name: markdown-writing
description: Rules for creating or editing Markdown (`.md`) files
---

# Markdown writing guide

## Principles

- Aim for simple, readable documents
- Save files as UTF-8
- Watch out for mojibake when the file contains Japanese
- Keep prose as concise as possible
- Avoid redundant phrasing

## Forbidden

- Don't use numbered lists (`1.` `2.` `3.`); use `-`
- Don't number headings (`### 1. xxx`, `### 2. xxx`, `## 1. xxx`, etc.)
  - This is the classic hack of routing a numbered list through headings — it's a numbered list in disguise
  - Adding or removing items forces renumbering and maintenance falls apart
  - If you only want to enumerate, use a bullet list (`-`); if you need separate sections, give each heading a content-based title
- Don't use paragraph labels (short name line + blank line + value, repeated)
  - Example: `Source\n\n<content>\n\nDescription\n\n<content>\n\nSuggestion\n\n<content>`
  - This routes the banned `**Label**:` pattern through newlines — structurally it's the same key/value
  - Promote labels to headings (`###` etc.), use bullets if you're enumerating, otherwise write natural prose
- Don't use the non-Markdown structuring pattern `**Label**:`
- Never use the colon-separated "key: value" style in bullets
  - `- Name: Taro Yamada` is forbidden
  - Write information as a natural sentence, or structure it with headings and paragraphs
  - Just like `**Label**:`, this isn't real Markdown syntax — never use it
- Don't use tables
- Don't use bold (`**bold**`)
- Don't use horizontal rules (`---`)
- Don't use the Japanese full stop `。`
  - Use a newline at the end of a sentence

Bad (the `**Label**:` pattern):

```markdown
**Name**: Taro Yamada
**Role**: Engineer
```

Bad (colon-separated bullets — also strictly forbidden):

```markdown
- Name: Taro Yamada
- Role: Engineer
```

Good (write as a natural sentence):

```markdown
Taro Yamada is an engineer
```

Good (structure with headings and paragraphs):

```markdown
## Taro Yamada

Engineer
```

## Japanese-language rules

- Insert a half-width space between Japanese characters and ASCII (alphanumeric)
  - Good: `Claude は AI です`
  - Bad: `Claudeは AIです`
- Use `、` for the comma
- Use either full-width `（）` or half-width `()`, but stay consistent

## Document structure

- Use headings starting from `#` and don't skip levels
- Use only one top-level `#` per document
- One blank line between paragraphs
- Blank lines around lists too
- Don't run sentences together with full stops — break with newlines

```markdown
# Title

Overview

## Section

Body

- Item A
- Item B

Next paragraph
```

## Code blocks

- Inline code uses backticks `` ` ``
- Multi-line code uses fenced blocks with a language tag

````markdown
Use `variable_name`

```python
def hello():
    print("Hello")
```
````

## Links and images

```markdown
[link text](URL)
![alt text](image-path)
```
