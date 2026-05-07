---
name: git-repo-list
description: Specialist skill for searching, listing, and resolving paths of locally-cloned Git repositories. Invoke whenever you need a repository path, want to filter repos under ghq, or are about to comb the ghq root with `find` / `ls`. Don't reach for raw `find` / `ls` — load this skill first.
---

# git-repo-list: find local repos with `ghq list`

When listing or searching locally-cloned Git repositories, use `ghq list`.

## Basic commands

- `ghq list`: relative-path listing
- `ghq list -p`: absolute-path listing
- `ghq list -e <name>`: exact-match search
- `ghq list <pattern>`: substring-match search
- `ghq root`: show the primary root
- `ghq root --all`: show all roots

`-e` accepts both `<repo>` alone and `<owner>/<repo>` form (use the latter to disambiguate same-named repos).

## Filtering by owner

A trailing-slash pattern avoids accidental substring matches:

```sh
ghq list -p x-motemen/
```

Plain `x-motemen` could also match owners like `x-motemen-lab`.

## When same-named repos exist

Two-step: enumerate candidates, then pin down with `owner/repo` form:

```sh
ghq list ghq                         # list candidates
ghq list -p -e x-motemen/ghq         # absolute path of the specific one
```

## Idiom for piping a path elsewhere

```sh
cd "$(ghq list -p -e <owner>/<repo>)"
```

## Counts / totals

`ghq list` outputs one line per repo, so line-counting works:

```sh
ghq list | wc -l
```

## Fallback

If `ghq` isn't installed, tell the user. If they ask for an alternative, you can scan conventional clone paths like `~/ghq` or `~/src` with `find` / `fd`, but accuracy will drop.
