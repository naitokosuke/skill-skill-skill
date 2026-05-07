---
name: git-worktree-list
description: Git worktree の一覧・状態確認をする場面の専門スキル。ユーザーが `git worktree list` を実行しようとしているとき、現在どんな worktree があるか把握したいとき、各 worktree の git status をまとめて確認したいときは必ずこのスキルを invoke すること。素の `git worktree list` を直接実行せず、このスキルを先に読み込む
---

# git-worktree-list: gwq list で worktree を一覧する

worktree を一覧・状態確認する際は `git worktree list` ではなく `gwq list` / `gwq status` を使用する

## 一覧

- `gwq list`: 現在のリポジトリの worktree 一覧
- `gwq list -g`: 全リポジトリにまたがってグローバル表示
- `gwq list -v`: 詳細表示
- `gwq list --json`: JSON 出力

## ステータス

- `gwq status`: 現リポジトリの worktree の git status を含めて表示
- `gwq status --global`: 全リポジトリを横断して状態確認
- `gwq status --watch`: ライブ監視
- `gwq status --json`: 構造化出力

## 実行位置の注意

カレントディレクトリが git リポジトリ外（ghq root 直下や無関係なパス）のときに非 global で実行すると「not a git repository」で失敗する。横断表示やカレント不問で叩きたいときは必ず global フラグを付ける

- `gwq list` 系 → `-g`
- `gwq status` 系 → `--global`

## JSON スキーマ

`gwq list --json` と `gwq status --json` は別フォーマット

- `gwq list --json`: worktree のメタデータ中心（path, branch, commit_hash 等）
- `gwq status --json`: 変更状態（modified / untracked など）まで含む

jq でのフィルタ用途によって使い分ける

## フォールバック

`gwq` が未インストールの場合はその旨をユーザーに伝える

## 関連スキル

- 追加: git-worktree-add
- 削除: git-worktree-rm
- 移動・内部実行: git-worktree-cd
