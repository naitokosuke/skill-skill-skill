---
name: memo-aggregated
description: "Create or reference personal worklogs / knowledge notes / screenshots aggregated under the repo's ___naito___/ directory. Triggered by requests like \"create a memo for issue #1234\", \"write a worklog\", \"check that note in ___naito___\". Usage: /memo-aggregated <content or instruction>"
argument-hint: <content or instruction>
---

# Read / write the `___naito___/` personal archive

The `___naito___/` directory at the repo root is where naito aggregates personal memos, screenshots, scripts, etc. tied to the repo. It is ignored via global gitignore (`~/.config/git/ignore`), so nothing here is committed. This skill is responsible only for **routing** — deciding where in `___naito___/` to write, and where to read from.

## Parsing the instruction

```
$ARGUMENTS
```

From the user's instruction, identify:

- What to write or look up (issue number, topic, kind)
- Whether to create a new file or append to an existing one

## Resolving the ___naito___ root

1. Get the repo root with `git rev-parse --show-toplevel`
2. **Important**: When the current directory is a git worktree, the worktree's `___naito___` is a symlink pointing to the parent checkout. Just use `<repo-root>/___naito___/` as the path — writes go through the symlink to the actual storage. There is no need to normalize via `readlink` (you may report either path to the user)
3. If `___naito___/` does not exist, ask the user "this repo doesn't have `___naito___/` yet; create it?". Do not silently create it at the top level

## Routing rules

Place files into the following subdirectories based on the instruction. **Prefer existing directories**: if an existing file matches the topic, append/place alongside it. Only create a new directory when the instruction clearly introduces a new topic.

| Subdirectory | Purpose | Naming |
|---|---|---|
| `issue/<number>/` | Per-issue worklog / screenshots / recordings (GitLab/GitHub) | Extract issue number from the instruction (`#1234`, `issue 1234`, etc). Filenames inside are kebab-case describing the content. e.g. `issue/1234/<topic>.md` |
| `wip/` | Cross-cutting / ongoing notes not tied to any issue | `wip/<topic>.md` or `wip/<topic>/...` |
| `wip/about-repo/` | Repo-wide knowledge (architecture / tech-stack / design-system / culture / engineering / features, etc.) | Prefer appending to existing files (e.g. `architecture.md`) |
| `code_tour/<topic>/` | Materials for VS Code Code Tour | Match topic names to existing siblings if any |
| `sentry/` | Sentry incidents / responses | `sentry/categories/<category>.md` or `sentry/<topic>.md`. The issue list lives at `sentry/gitlab-issues.md` |
| `okr/<year>/` | OKRs | Year-partitioned, e.g. `okr/2026/...` |
| `ideal/<topic>/` | Re-architecture proposals, ideal-state notes | Follow existing names if any |
| `script/` | Personal shell scripts | `script/<name>.sh` (mind the executable bit) |

When unsure, decide in this order:

1. If there is an issue number, always `issue/<number>/`
2. If the topic concerns repo-wide policy / architecture / culture, use `wip/about-repo/`
3. Otherwise, for ongoing notes, use `wip/` directly
4. If none of the above fit and an existing subdirectory matches the topic, use it
5. If still undecided, ask the user a 2-way choice (e.g. "issue/<number>/ or wip/?")

## Reading existing notes

When the user says things like "that note in ___naito___" or "what was the memo on issue 1234":

1. If there is an issue number, `ls` `issue/<number>/` to enumerate files
2. If there is a topic keyword, full-text search via `rg -l <keyword> ___naito___/`
3. On a hit, Read the file and summarize back to the user

## Style / formatting

- Out of scope: this skill is responsible for **routing only**. Memo structure and tone are decided per case
- **REQUIRED SUB-SKILL**: when creating or editing Markdown, you must invoke the `markdown-writing` skill via the Skill tool and follow its rules
- Use ISO format `YYYY-MM-DD` for dates (current date comes from the `currentDate` env info)

## Append vs new file

- If an existing file covers the same issue / topic, **prefer appending**. Avoid spawning multiple files for the same theme
- If the existing file is clearly about a different concern, create a new file
- When appending, place content at the end of a section or under an appropriate heading without breaking structure

## Examples

User: "create a memo for issue #5678; I want to record the investigation result"
→ Create `<repo-root>/___naito___/issue/5678/` and a `<kebab-case-describing-content>.md`

User: "save my current understanding of the architecture"
→ Read `<repo-root>/___naito___/wip/about-repo/architecture.md`, append on top of existing content

User: "show me that memo on issue 1234"
→ `ls` `<repo-root>/___naito___/issue/1234/`, Read the markdown

## Notes

- This skill never touches `.git/` or tracked files. `___naito___/` is gitignored, so no git operations are needed
- Files under `___naito___/` are not committed
- Writes from a worktree resolve through the symlink to the parent checkout's actual storage (worktrees share the same content)
