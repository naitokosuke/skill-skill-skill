---
name: git-worktree-cd
description: Git worktree ディレクトリへの移動・パス取得・worktree 内でのコマンド実行の専門スキル。特定の worktree に cd したいとき、worktree のパスを取得して別コマンドに渡したいとき、worktree 内でテストやビルドを走らせたいとき、手動で ghq root を辿ろうとしているときは必ずこのスキルを invoke すること。手動のパス推測をせず、このスキルを先に読み込む
---

# git-worktree-cd: gwq で worktree へ移動・内部実行する

worktree へ移動する、パスを取得する、あるいは worktree 内でコマンドを実行する際は `gwq cd` / `gwq get` / `gwq exec` を使用する

## 移動・パス取得

- `gwq cd <name>`: worktree へ移動（新シェル起動）
- `cd "$(gwq get <name>)"`: 現在のシェルで移動（パスに空白が含まれても安全なようダブルクォートで囲む）
- `gwq get -g <repo>:<branch>`: グローバルスコープでパス解決

## worktree 内でコマンド実行

- `gwq exec <name> -- <command>`: 移動せずに実行
- `gwq exec -s <name> -- <command>`: 実行後にその worktree へ留まる
- 例: `gwq exec feature -- npm test`

## フォールバック

`gwq` が未インストールの場合はその旨をユーザーに伝える

## 関連スキル

- 追加: git-worktree-add
- 一覧: git-worktree-list
- 削除: git-worktree-rm
