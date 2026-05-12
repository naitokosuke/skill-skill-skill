---
name: git-worktree-list
description: Specialist skill for listing and inspecting the status of Git worktrees. Invoke whenever the user is about to run `git worktree list`, wants to see what worktrees currently exist, or wants to check the git status of each worktree at once. Don't run plain `git worktree list` directly — load this skill first.
---

# git-worktree-list: list worktrees with gwq

When listing or inspecting worktrees, use `gwq list` / `gwq status` instead of `git worktree list`.

## Listing

- Use `gwq list` for worktrees of the current repo
- Use `gwq list -g` for a global view across all repos
- Use `gwq list -v` for verbose output
- Use `gwq list --json` for JSON output

## Status

- Use `gwq status` for worktrees of the current repo with their git status
- Use `gwq status --global` for cross-repo status
- Use `gwq status --watch` for live monitoring
- Use `gwq status --json` for structured output

## Caveat about where you run it

Running the non-global form outside a git repository (directly under the ghq root, or any unrelated path) fails with "not a git repository". For cross-repo or directory-agnostic invocations, always pass the global flag:

- `gwq list ...` → `-g`
- `gwq status ...` → `--global`

## JSON schemas

`gwq list --json` and `gwq status --json` are different formats:

- `gwq list --json` returns worktree metadata such as path, branch, and commit_hash
- `gwq status --json` also includes change state such as modified / untracked

Pick whichever fits the jq filter you have in mind.

## Fallback

If `gwq` is not installed, tell the user.

## Related skills

- Adding worktrees — git-worktree-add
- Removing worktrees — git-worktree-rm
- Moving / executing inside — git-worktree-cd
