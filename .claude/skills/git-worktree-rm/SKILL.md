---
name: git-worktree-rm
description: Specialist skill for deleting Git worktrees. Invoke whenever the user is about to run `git worktree remove`, wants to clean up unused worktrees, or wants to delete a worktree along with its branch. Don't run plain `git worktree remove` directly — load this skill first.
---

# git-worktree-rm: remove worktrees with `gwq remove`

When deleting a worktree, use `gwq remove` instead of `git worktree remove`.

## Basic commands

- Use `gwq remove <pattern>` to remove a worktree
- Use `gwq remove -b <branch>` to remove the worktree and its branch together
- Use `gwq remove --dry-run <pattern>` to preview
- Use `gwq remove -f <pattern>` to force-remove even with uncommitted changes (for dirty worktrees)
- Use `gwq remove --force-delete-branch <branch>` to delete the branch even if unmerged (used with `-b`)

`-f` is a worktree-side override (ignores dirty changes); `--force-delete-branch` is a branch-side override (ignores the unmerged-branch refusal). Different purposes.

## Using dry-run

Preview with the same flag combination you intend to use for the real removal. Whether or not `-b` is present changes the "will the branch also be deleted" preview, so keep them aligned:

```sh
gwq remove --dry-run -b feature/old     # real run will also use -b
gwq remove -b feature/old
```

## Fallback

If `gwq` is not installed, tell the user.

## Related skills

- Adding worktrees — git-worktree-add
- Listing worktrees — git-worktree-list
