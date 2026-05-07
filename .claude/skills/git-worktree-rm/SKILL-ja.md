---
name: git-worktree-rm
description: Git worktree を削除する場面の専門スキル。ユーザーが `git worktree remove` を実行しようとしているとき、不要になった worktree を片付けたいとき、ブランチごと worktree を消したいときは必ずこのスキルを invoke すること。素の `git worktree remove` を直接実行せず、このスキルを先に読み込む
---

# git-worktree-rm: gwq remove で worktree を削除する

worktree を削除する際は `git worktree remove` ではなく `gwq remove` を使用する

## 基本コマンド

- `gwq remove <pattern>`: worktree を削除
- `gwq remove -b <branch>`: worktree とブランチを同時削除
- `gwq remove --dry-run <pattern>`: プレビュー
- `gwq remove -f <pattern>`: 未コミット変更があっても強制削除（dirty worktree 用）
- `gwq remove --force-delete-branch <branch>`: ブランチが未マージでも削除（`-b` 併用時）

`-f` は worktree 側の強制（dirty な変更を無視）、`--force-delete-branch` はブランチ側の強制（未マージ拒否を無視）で、目的が異なる

## dry-run の使い方

プレビューは実削除で打つ予定のフラグ構成をそのまま `--dry-run` 付きで打つ。`-b` を付けるかどうかで「ブランチも消えるか」の見え方が変わるので忠実に揃える

```sh
gwq remove --dry-run -b feature/old     # 実削除も -b 付きで行う想定
gwq remove -b feature/old
```

## フォールバック

`gwq` が未インストールの場合はその旨をユーザーに伝える

## 関連スキル

- 追加: git-worktree-add
- 一覧: git-worktree-list
