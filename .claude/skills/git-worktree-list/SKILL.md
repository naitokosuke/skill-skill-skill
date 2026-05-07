---
name: git-worktree-list
description: Specialist skill for listing and inspecting the status of Git worktrees. Invoke whenever the user is about to run `git worktree list`, wants to see what worktrees currently exist, or wants to check the git status of each worktree at once. Don't run plain `git worktree list` directly — load this skill first.
---

# git-worktree-list: list worktrees with gwq

When listing or inspecting worktrees, use `gwq list` / `gwq status` instead of `git worktree list`.

## Listing

- `gwq list`: worktrees of the current repo
- `gwq list -g`: global view across all repos
- `gwq list -v`: verbose
- `gwq list --json`: JSON output

## Status

- `gwq status`: worktrees of the current repo with their git status
- `gwq status --global`: cross-repo status
- `gwq status --watch`: live monitoring
- `gwq status --json`: structured output

## Caveat about where you run it

Running the non-global form outside a git repository (directly under the ghq root, or any unrelated path) fails with "not a git repository". For cross-repo or directory-agnostic invocations, always pass the global flag:

- `gwq list ...` → `-g`
- `gwq status ...` → `--global`

## JSON schemas

`gwq list --json` and `gwq status --json` are different formats:

- `gwq list --json`: worktree metadata (path, branch, commit_hash, etc.)
- `gwq status --json`: also includes change state (modified / untracked, etc.)

Pick whichever fits the jq filter you have in mind.

## Fallback

If `gwq` is not installed, tell the user.

## Related skills

- Add: git-worktree-add
- Remove: git-worktree-rm
- Move / exec inside: git-worktree-cd
