---
name: git-worktree-add
description: Git worktree を新しく作成する場面の専門スキル。ユーザーが `git worktree add` を実行しようとしているとき、新しい worktree を切りたいとき、並行作業のため別ブランチの作業ディレクトリが欲しいとき、Claude Code を複数動かすための worktree を用意したいときは必ずこのスキルを invoke すること。素の `git worktree add` を直接実行せず、このスキルを先に読み込む
---

# git-worktree-add: gwq add で worktree を作成する

新しい worktree を作成する際は `git worktree add` ではなく `gwq add` を使用する

## 基本コマンド

- `gwq add -b <new-branch>` で新規ブランチの worktree を作成（start-point は現在の HEAD）
- `gwq add <existing-branch>` で既存ブランチに作成
- `gwq add -i` で対話的にブランチ選択
- `gwq add -s ...` で作成後にその worktree へ留まる
- `gwq add -f ...` で強制作成

`gwq add` は位置引数が `[branch] [path]` の 2 つだけで、start-point を指定する引数は存在しない
`gwq add -b <new> <ref>` と書くと第 2 引数が path として解釈される（ハマりどころ）
不安なら先に `gwq add --help` を読むこと

## 特定の ref を起点に新規ブランチで worktree を作るとき

`gwq add -b` は現在の HEAD を起点にするため、`main` や `origin/<branch>` など別の ref を起点にしたい場合は 2 段階にする

```sh
git fetch origin                                              # remote を最新化
git branch --no-track <new-branch> <start-point>              # 例: origin/feature-x
gwq add <new-branch>
```

`--no-track` は必須級
remote-tracking ref (`origin/xxx`) からブランチを切ると Git のデフォルトで upstream が自動設定され、後でうっかり `git push` すると起点ブランチ（例: `feature-x`）に混ぜ込む事故になる
新ブランチは独立した作業ブランチとして扱うのが通常なので `--no-track` を付ける
明示的に track したい特殊ケースだけ外す

## 配置先

worktree は gwq の設定に従い `<basedir>/<template 展開結果>` に作られる
`gwq config list` で現在値を確認する

親 tree と揃っているか必ず先に検証する
`git worktree list` で親 tree のパスを見て、gwq のデフォルト出力先が親と同じ階層規則になっていなければズレている
よくあるズレ：

* GitLab の nested group (`gitlab.com/owner/sub1/sub2/repo` のようなサブグループ持ち) で gwq v0.0.19 の `{{.Owner}}` が top-level group しか拾わず、subgroup が丸ごと落ちる（例: 親 `~/src/gitlab.com/group/sub1/sub2/repo` に対し worktree が `~/src/gitlab.com/group/repo=...` に作られる）

ズレた場合の修正は、該当リポジトリに local config (`.gwq.toml`) を置いてグローバル設定を上書きする
templating 変数では subgroup が表現できないので、basedir を親 tree の親ディレクトリに固定してしまうのが単純で確実

```sh
gwq config set --local worktree.basedir "$(dirname "$(git rev-parse --show-toplevel)")"
gwq config set --local naming.template "{{.Repository}}={{.Branch}}"
printf '\n.gwq.toml\n' >> .git/info/exclude  # コミットに混ぜない（ユーザー環境依存）
```

これで `gwq add <branch>` が path 省略でも親 tree の sibling に作る
path を明示する運用はメンテが面倒なので避ける（config を直す）

ブランチ名に `/` が含まれる場合（例: `feature/oauth-login`）は、そのままサブディレクトリ階層として扱われる

## 既存ブランチから作るとき

remote にしか存在しないブランチをローカルに持ってきて worktree 化する場合は、先に fetch しておく

```sh
git fetch origin
gwq add <existing-branch>
```

ローカルに ref が既にあれば `git fetch` は不要

## フォールバック

`gwq` が未インストールの場合はその旨をユーザーに伝えたうえで、指示があれば素の `git worktree add` を使う

## 関連スキル

- 一覧は git-worktree-list
- 削除は git-worktree-rm
- 移動・内部実行は git-worktree-cd
