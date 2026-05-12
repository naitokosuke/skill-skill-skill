---
name: git-clone
description: Git リポジトリをクローンする場面の専門スキル。ユーザーが `git clone` を実行しようとしているとき、新しいリポジトリをローカルに取得しようとしているとき、GitHub の URL や owner/repo からリポジトリを持ってきたいときは必ずこのスキルを invoke すること。素の `git clone` を直接実行せず、このスキルを先に読み込む
---

# git-clone: ghq get を使う

Git リポジトリをクローンする際は `git clone` ではなく `ghq get` を使用する

## 基本コマンド

- `ghq get <URL or owner/repo>` でクローン
- 配置先は `$(ghq root)/<host>/<owner>/<repo>`

配置先の絶対パスを提示するときは `ghq root` を実行して実際の値を確認する（`~/ghq` とは限らない、ユーザー設定で `~/src` などに変わる）

## 主なオプション

- `-u` で既にクローン済みなら更新（fetch 相当、checkout は変えない）
- `-p` で SSH プロトコルで取得
- `--shallow` で shallow clone
- `--branch <name>` でブランチまたはタグを指定（内部で `git clone --branch` を呼ぶためタグ名でも動く）

指定が無ければ HTTPS をデフォルトとする
`-p` は明示要求があったときのみ使う

## 既存リポジトリがある状態で特定の ref に切り替えたいとき

`ghq get -u --branch <ref>` は fetch まで行うが checkout 済みブランチは切り替わらない
切り替えが必要なら続けて以下を実行する

```sh
git -C "$(ghq list -p -e <owner>/<repo>)" checkout <ref>
```

## cd するイディオム

```sh
cd "$(ghq list -p -e <owner>/<repo>)"
```

`<owner>/<repo>` の形で指定すると同名リポジトリとの衝突を避けられる

## フォールバック

`ghq` が未インストールの場合はその旨をユーザーに伝えたうえで、指示があれば次の形で代替する

```sh
git clone [--depth=1] [--branch <ref>] <URL> <path>
```

## 関連スキル

- ローカルリポジトリのパスを探す場面は git-repo-list
