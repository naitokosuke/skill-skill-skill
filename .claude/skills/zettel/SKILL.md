---
name: zettel
description: "Create or update project-independent technical notes in a personal knowledge base (`~/zettel/`). Records research findings and insights in a form your future self can revisit. Usage: /zettel <content or instruction>"
argument-hint: <content or instruction>
---

# zettel (technical knowledge base) creation / update

Create or update project-independent technical notes under `~/zettel/`.

## Parsing the instruction

```
$ARGUMENTS
```

From the user's instruction, identify:

- The topic to write about (technology, concept, migration, troubleshooting, etc.)
- Whether this updates an existing file or creates a new one
- Related research targets (branch, file, error, etc.)

Run `ls ~/zettel/` to decide between updating an existing file and creating a new one.

## File naming

- Location: `~/zettel/{slug}.md`
- `slug` is lowercase kebab-case
- Pick a short name that captures the topic accurately
  - Good: `vue-transition-group.md`, `migrate-ghq.md`, `oxlint-raw-transfer-bumpalo.md`
  - Bad: `memo.md`, `vue.md`, `2026-05-01.md`

## Top principle: project-independent

A zettel is a generic knowledge base your future self — or a different project — can reference. Don't write project-specific information.

### Bad examples

```markdown
We use transition-group in `src/app/todos/__features/TodoTrees.vue`
```

```markdown
Combined with this project's `AfExpandTransition` component, it...
```

```markdown
Introduced in MR #3777
```

### Good examples

Replace project-specific component names and file paths with generic examples:

```markdown
Use it for list scenarios where each child node animates one at a time as a tree expands or collapses
```

```vue
<transition-group v-if="folder.isOpened && folder.children?.length" ...>
  <ChildNode v-for="child in folder.children" :key="child.key" />
</transition-group>
```

Even if the investigation started from project code, rewrite the takeaway as a "generalized principle" or "example that applies to other projects too".

## Document structure

### Outline

```markdown
# {topic name}

{1-2 paragraph overview: what this is about, when it's useful}

## {main section}

{principles or mechanism}

## {usage / code example}

{minimal generic code example}

## {applications / caveats}

{edge cases, common pitfalls}

## References

- [{title}]({URL})
```

### Recommended content

- Definition of the concept and "why it works that way"
- Minimal, runnable, generic code examples
- Links to official documentation
- Breaking changes between versions (when researched in a migration context)
- Comparison tables / mappings (API rename, config-value mappings, etc.)

### Avoid

- Project-specific file paths, component names, variable names
- References to specific MRs / PRs / issues / commits
- "On this branch" / "in this round of work" — context-bound phrasing
- One-off debug logs or work history (those are work notes, not zettel)

## Frontmatter and tags

Every zettel file must start with YAML frontmatter that records `tags`:

```yaml
---
tags:
  - frontend
  - vue
  - migration
---
```

### Tag selection procedure

1. Read `~/zettel/_tags.md` to confirm the existing tag definitions
2. Always include exactly one category tag (`frontend` / `tooling` / `migration`, etc.)
3. Add as many relevant technology tags as needed
4. Add a new tag only when no existing tag can express the concept
5. When you do add a new tag, also append its definition and intended use to `~/zettel/_tags.md`

### Tag conventions

- Don't prefix with `#` — write a flat list of strings (Obsidian recognizes these as tags automatically)
- Don't use nested tags (`vue/transition` etc.) — flat usage prevents naming drift
- Don't use inline `#tag` syntax in the body — keep tags in frontmatter only
- `worklog` is a marker tag for one-off work logs (separates them from pure technical knowledge)

## Markdown rules

Follow the `markdown-writing` skill. Key points:

- Use `-` lists, never numbered lists
- Don't use Japanese full stop `。` — break with newlines
- Don't use bold (`**...**`); use headings for structure
- Tables are reserved for technical-spec correspondences (e.g. API rename)
- Half-width space between Japanese and ASCII characters

## References section

Always end with a `## References` section listing official docs and primary sources. Only list URLs that actually exist (don't fabricate URLs).

## Consistency with existing zettel

Before creating a new file, check `ls ~/zettel/` and align the style and structure with what's there. If a zettel on a similar topic already exists, consider appending to it instead of creating a new one.
