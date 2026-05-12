---
name: git-worktree-cd
description: Git worktree ディレクトリへの移動・パス取得・worktree 内でのコマンド実行の専門スキル。特定の worktree に cd したいとき、worktree のパスを取得して別コマンドに渡したいとき、worktree 内でテストやビルドを走らせたいとき、手動で ghq root を辿ろうとしているときは必ずこのスキルを invoke すること。手動のパス推測をせず、このスキルを先に読み込む
---

# git-worktree-cd: gwq で worktree へ移動・内部実行する

worktree へ移動する、パスを取得する、あるいは worktree 内でコマンドを実行する際は `gwq cd` / `gwq get` / `gwq exec` を使用する

## 移動・パス取得

- `gwq cd <name>` で worktree へ移動（新シェル起動）
- `cd "$(gwq get <name>)"` で現在のシェルで移動（パスに空白が含まれても安全なようダブルクォートで囲む）
- `gwq get -g <repo>:<branch>` でグローバルスコープのパス解決

## worktree 内でコマンド実行

- `gwq exec <name> -- <command>` で移動せずに実行
- `gwq exec -s <name> -- <command>` で実行後にその worktree へ留まる
- 例として `gwq exec feature -- npm test` で `feature` worktree 内のテストを走らせる

## フォールバック

`gwq` が未インストールの場合はその旨をユーザーに伝える

## 関連スキル

- 追加は git-worktree-add
- 一覧は git-worktree-list
- 削除は git-worktree-rm
