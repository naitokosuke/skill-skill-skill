---
name: git-worktree-cd
description: Specialist skill for moving into a Git worktree directory, getting its path, or running commands inside it. Invoke whenever you want to cd into a specific worktree, fetch a worktree's path to pass to another command, run tests / builds inside a worktree, or are about to walk the ghq root by hand. Don't guess paths manually — load this skill first.
---

# git-worktree-cd: move into / execute inside a worktree with gwq

When moving into a worktree, fetching its path, or running a command inside it, use `gwq cd` / `gwq get` / `gwq exec`.

## Move / get path

- `gwq cd <name>`: move into the worktree (spawns a new shell)
- `cd "$(gwq get <name>)"`: move within the current shell (always double-quote so paths with spaces stay safe)
- `gwq get -g <repo>:<branch>`: resolve a path in the global scope

## Running a command inside a worktree

- `gwq exec <name> -- <command>`: run without moving
- `gwq exec -s <name> -- <command>`: run, then stay in that worktree
- Example: `gwq exec feature -- npm test`

## Fallback

If `gwq` is not installed, tell the user.

## Related skills

- Add: git-worktree-add
- List: git-worktree-list
- Remove: git-worktree-rm
