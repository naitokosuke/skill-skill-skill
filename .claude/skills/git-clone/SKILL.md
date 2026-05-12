---
name: git-clone
description: Specialist skill for cloning Git repositories. Invoke whenever the user is about to run `git clone`, fetch a fresh repository locally, or pull a repo by GitHub URL or `owner/repo`. Do not run plain `git clone` directly — load this skill first.
---

# git-clone: use `ghq get`

When cloning a Git repository, use `ghq get` instead of `git clone`.

## Basic commands

- Use `ghq get <URL or owner/repo>` to clone
- Cloned repos land at `$(ghq root)/<host>/<owner>/<repo>`

When showing an absolute destination path, run `ghq root` to get the actual value (it's not always `~/ghq` — users often configure it to `~/src` etc.).

## Main options

- Pass `-u` to update if already cloned (equivalent to fetch; checkout is unchanged)
- Pass `-p` to fetch via SSH protocol
- Pass `--shallow` for a shallow clone
- Pass `--branch <name>` to specify branch or tag (passed through to `git clone --branch`, so tag names also work)

Default to HTTPS unless told otherwise. Use `-p` only on explicit request.

## Switching to a specific ref when the repo already exists

`ghq get -u --branch <ref>` performs the fetch but does not switch the checked-out branch. If a switch is needed, follow up with:

```sh
git -C "$(ghq list -p -e <owner>/<repo>)" checkout <ref>
```

## cd idiom

```sh
cd "$(ghq list -p -e <owner>/<repo>)"
```

Specifying as `<owner>/<repo>` avoids collisions with same-named repositories.

## Fallback

If `ghq` is not installed, tell the user. If they ask for an alternative, fall back to:

```sh
git clone [--depth=1] [--branch <ref>] <URL> <path>
```

## Related skills

- Locating an existing local repository path — git-repo-list
