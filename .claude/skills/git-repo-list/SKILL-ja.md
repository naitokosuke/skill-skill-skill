---
name: git-repo-list
description: ローカルにクローン済みの Git リポジトリを検索・一覧・パス解決する場面の専門スキル。リポジトリのパスを知りたいとき、ghq 配下のリポジトリを絞り込みたいとき、`find` や `ls` で ghq root を漁ろうとしているときは必ずこのスキルを invoke すること。素の `find` / `ls` を直接使わず、このスキルを先に読み込む
---

# git-repo-list: ghq list でローカルリポジトリを探す

ローカルにクローン済みの Git リポジトリを一覧・検索する際は `ghq list` を使用する

## 基本コマンド

- `ghq list`: 相対パスで一覧
- `ghq list -p`: 絶対パスで一覧
- `ghq list -e <name>`: 完全一致検索
- `ghq list <pattern>`: 部分一致検索
- `ghq root`: プライマリ root を表示
- `ghq root --all`: すべての root

`-e` には `<repo>` 単体だけでなく `<owner>/<repo>` 形式を渡せる（同名衝突時の曖昧性回避に使う）

## owner 単位で絞り込む

末尾スラッシュ付きのパターンで指定すると部分一致による誤マッチを避けられる

```sh
ghq list -p x-motemen/
```

`x-motemen` だけだと `x-motemen-lab` のような owner にも当たる可能性がある

## 同名リポジトリがあるとき

一段目で候補を洗い出し、二段目で `owner/repo` 形式の `-e` に絞る

```sh
ghq list ghq                         # まず候補を列挙
ghq list -p -e x-motemen/ghq         # 対象を特定して絶対パスを得る
```

## パスを他コマンドに渡すイディオム

```sh
cd "$(ghq list -p -e <owner>/<repo>)"
```

## 総数・集計

`ghq list` は 1 リポジトリ 1 行なので行数カウントできる

```sh
ghq list | wc -l
```

## フォールバック

`ghq` が未インストールの場合はその旨をユーザーに伝える。代替手段を求められた場合は `find` / `fd` で `~/ghq` や `~/src` などクローン配置の慣習パスを走査することになるが、精度は落ちる
