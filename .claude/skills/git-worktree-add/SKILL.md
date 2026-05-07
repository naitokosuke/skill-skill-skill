---
name: git-worktree-add
description: Specialist skill for creating new Git worktrees. Invoke whenever the user is about to run `git worktree add`, wants to cut a new worktree, needs a working directory on a different branch for parallel work, or is preparing a worktree to run multiple Claude Code instances. Don't run plain `git worktree add` directly — load this skill first.
---

# git-worktree-add: create worktrees with `gwq add`

When creating a new worktree, use `gwq add` instead of `git worktree add`.

## Basic commands

- `gwq add -b <new-branch>`: create a worktree on a new branch (start point is the current HEAD)
- `gwq add <existing-branch>`: create on an existing branch
- `gwq add -i`: pick a branch interactively
- `gwq add -s ...`: stay in the new worktree after creation
- `gwq add -f ...`: force creation

`gwq add` only takes two positional arguments, `[branch] [path]` — **there is no argument for a start point**. Writing `gwq add -b <new> <ref>` makes the second argument get interpreted as a path (a common gotcha). When in doubt, read `gwq add --help` first.

## Creating a worktree on a new branch from a specific ref

Because `gwq add -b` uses the current HEAD as start point, use a two-step flow when you want to base off `main` or `origin/<branch>`:

```sh
git fetch origin                                              # update remote
git branch --no-track <new-branch> <start-point>              # e.g. origin/feature-x
gwq add <new-branch>
```

`--no-track` is essentially mandatory. Branching off a remote-tracking ref (`origin/xxx`) sets the upstream automatically by default, so a stray `git push` later can mix work into the start-point branch (e.g. `feature-x`). New branches are normally treated as independent work branches, so add `--no-track`. Drop it only in the rare case you actually want to track explicitly.

## Destination

Worktrees are created under `<basedir>/<expanded template>` per gwq config. Check current values with `gwq config list`.

**Always verify it lines up with the parent tree first.** Look up the parent tree's path with `git worktree list`, then check whether gwq's default destination follows the same hierarchy. A common misalignment:

* GitLab nested groups (`gitlab.com/owner/sub1/sub2/repo`) — gwq v0.0.19's `{{.Owner}}` only picks up the top-level group, dropping the subgroups entirely (e.g. for parent `~/src/gitlab.com/group/sub1/sub2/repo` the worktree ends up at `~/src/gitlab.com/group/repo=...`)

Fix: place a **local config (`.gwq.toml`) in that repo to override the global config**. Templating variables can't express subgroups, so the simplest, surest fix is to pin `basedir` to the parent of the parent tree:

```sh
gwq config set --local worktree.basedir "$(dirname "$(git rev-parse --show-toplevel)")"
gwq config set --local naming.template "{{.Repository}}={{.Branch}}"
printf '\n.gwq.toml\n' >> .git/info/exclude  # don't commit it (env-specific)
```

After this, `gwq add <branch>` (no path) creates as a sibling of the parent tree. Avoid the explicit-path workaround — fixing config is less maintenance.

If the branch name contains `/` (e.g. `feature/oauth-login`), it becomes a subdirectory hierarchy as-is.

## Creating from an existing branch

If the branch only exists on the remote, fetch first:

```sh
git fetch origin
gwq add <existing-branch>
```

If the ref is already local, no fetch is needed.

## Fallback

If `gwq` is not installed, tell the user; if they ask, fall back to plain `git worktree add`.

## Related skills

- List: git-worktree-list
- Remove: git-worktree-rm
- Move / exec inside: git-worktree-cd
