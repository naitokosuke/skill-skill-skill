---
name: git-worktree-cd
description: Specialist skill for moving into a Git worktree directory, getting its path, or running commands inside it. Invoke whenever you want to cd into a specific worktree, fetch a worktree's path to pass to another command, run tests / builds inside a worktree, or are about to walk the ghq root by hand. Don't guess paths manually — load this skill first.
---

# git-worktree-cd: move into / execute inside a worktree with gwq

When moving into a worktree, fetching its path, or running a command inside it, use `gwq cd` / `gwq get` / `gwq exec`.

## Move / get path

- Use `gwq cd <name>` to move into the worktree (spawns a new shell)
- Use `cd "$(gwq get <name>)"` to move within the current shell (always double-quote so paths with spaces stay safe)
- Use `gwq get -g <repo>:<branch>` to resolve a path in the global scope

## Running a command inside a worktree

- Use `gwq exec <name> -- <command>` to run without moving
- Use `gwq exec -s <name> -- <command>` to run and then stay in that worktree
- For example, `gwq exec feature -- npm test` runs the test suite inside the `feature` worktree

## Fallback

If `gwq` is not installed, tell the user.

## Related skills

- Adding worktrees — git-worktree-add
- Listing worktrees — git-worktree-list
- Removing worktrees — git-worktree-rm
