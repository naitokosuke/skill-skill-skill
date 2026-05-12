---
name: memo-colocated
description: "Create or update a personal memo (`.naito.md`) co-located next to a source file. Usage: /memo-colocated <target file path> <memo content or instruction>"
argument-hint: <target file path> <memo content or instruction>
---

# Personal memo (`.naito.md`) creation / update

Create or update a personal memo at `{filename}.naito.md` co-located next to the source file.

## Parsing the instruction

```
$ARGUMENTS
```

From the user's instruction, identify:

- The path of the target file
- What to write in the memo (content / intent)

## File naming convention

Append `.naito.md` to the target file's name.

- Target: `src/app/foo/Bar.vue`
- Memo: `src/app/foo/Bar.vue.naito.md`

## Frontmatter

Every memo file must have frontmatter.

### On creation

```yaml
---
updated: YYYY-MM-DD
---
```

Use today's date for `YYYY-MM-DD`.

### On update

Rewrite the existing `updated` to today's date.

## Content

Following the user's instruction, read the target file and summarize what's needed. Typical content includes:

- Component structure or template tree
- Key branching logic, organized
- Notes on planned changes
- Related issue context

## Style

- Write in plain declarative form (Japanese: だ / である調)
- Be concise — only the essentials

## Markdown rules

When creating or editing Markdown, you must invoke the `markdown-writing` skill via the Skill tool and follow its rules.
