---
name: memo-aggregated
description: "リポジトリ内 ___naito___/ ディレクトリ配下の作業ログ・知識帳・スクリーンショット等の個人アーカイブを作成・参照する。「issue #1234 のメモ作って」「作業ログ書いて」「___naito___ にあるあれ参照して」などで起動。使用法: /memo-aggregated <内容や指示>"
argument-hint: <内容や指示>
---

# `___naito___/` 個人アーカイブの読み書き

リポジトリのルートに置かれた `___naito___/` ディレクトリは、ユーザー naito が repo に紐づく個人メモ・スクショ・スクリプト等を集約する場所。global gitignore (`~/.config/git/ignore`) で無視されるため、コミット対象にはならない。本 skill はその中身の **どこに何を置く / どこから読むか** の振り分けに特化する。

## 指示の解釈

```
$ARGUMENTS
```

ユーザーの指示から以下を特定する:

- 何を書く / 参照するのか(issue 番号、テーマ、種別)
- 新規作成か既存への追記か

## ___naito___ ルートの解決

1. `git rev-parse --show-toplevel` で repo root を取得
2. **重要**: 現在のディレクトリが git worktree の場合、worktree 内の `___naito___` は親 checkout への symlink になっている。そのまま `<repo-root>/___naito___/` をパスとして使えば symlink 経由で実体に書き込まれる。`readlink` で実体パスに正規化する必要はない(ただしユーザーへ報告する際はどちらのパスでもよい)
3. `___naito___/` が存在しなければ「このリポジトリにはまだ `___naito___/` がないが作成するか?」とユーザーに確認する。勝手にトップレベルへ作らない

## 振り分けルール

ユーザーの指示内容に応じて以下のサブディレクトリへ配置する。**既存ディレクトリを優先**: 同一テーマで既存ファイルがあればそこへ追記/隣接させる。新規ディレクトリを切るのは指示が明確に新規テーマを指している場合のみ。

| サブディレクトリ | 用途 | 命名規則 |
|---|---|---|
| `issue/<番号>/` | GitLab/GitHub issue 単位の作業ログ・スクショ・録画 | issue 番号は指示から抽出(`#1234` `issue 1234` など)。番号ディレクトリ内のファイル名は内容を表す kebab-case。例: `issue/1234/<topic>.md` |
| `wip/` | 横断・継続的なメモ。issue に紐付かないもの | `wip/<topic>.md` または `wip/<topic>/...` |
| `wip/about-repo/` | リポジトリ全体に関わる知識(architecture / tech-stack / design-system / culture / engineering / features 等) | 既存ファイル(`architecture.md` など)を優先して追記 |
| `code_tour/<topic>/` | VS Code Code Tour 用の素材 | 既存の兄弟 topic があれば命名を合わせる |
| `sentry/` | Sentry の事象・対応メモ | `sentry/categories/<category>.md` または `sentry/<topic>.md`。issue 一覧は `sentry/gitlab-issues.md` |
| `okr/<year>/` | OKR | `okr/2026/...` のように年で区切る |
| `ideal/<topic>/` | リアーキ提案・理想形のメモ | 既存があればそれに従う |
| `script/` | 個人運用シェルスクリプト等 | `script/<name>.sh` (実行権限注意) |

判断に迷う場合の優先順:

1. 指示に issue 番号があれば必ず `issue/<番号>/`
2. 「全体方針」「アーキテクチャ」「文化」など repo 全体の話なら `wip/about-repo/`
3. それ以外で継続メモ的なら `wip/` 直下
4. 上記いずれにも当てはまらない、かつ既存サブディレクトリに合致するテーマがあればそこへ
5. それでも決まらなければユーザーに「`issue/<番号>/` か `wip/` か」など 2 択で確認

## 参照(読み)時の探し方

ユーザーが「___naito___ にあるあれ」「issue 1234 のメモどうだった」のように既存メモを読みたい場合:

1. issue 番号があれば `issue/<番号>/` を `ls` してファイル一覧を取得
2. テーマ語があれば `rg -l <キーワード> ___naito___/` で全文検索
3. ヒットしたら Read で内容確認 → ユーザーへ要約

## 文体・フォーマットのルール

- スコープ外: 本 skill は **配置先の振り分け** に責任を持つ。メモの構造・文体は対象に合わせて適宜決定する
- **REQUIRED SUB-SKILL**: Markdown ファイルを作成・編集する際は必ず Skill ツールで `markdown-writing` を invoke し、そのルールに従うこと
- 日付を入れたい場合は ISO 形式 `YYYY-MM-DD`(現在日付は環境情報の `currentDate` から取得)

## 既存ファイルへの追記 vs 新規作成

- 同一 issue / 同一 topic に既存ファイルがあれば **追記を優先**。同じテーマで複数ファイルが乱立しないようにする
- 既存ファイルが大きく性質の違う話題を扱っているなら新規ファイルを切る
- 追記時、構造を壊さないようにセクション末尾 or 適切な見出し配下に置く

## 実行例

ユーザー: 「issue #5678 のメモ作って。○○の調査結果を残したい」
→ `<repo-root>/___naito___/issue/5678/` を作成、`<内容を表す kebab-case>.md` を作成

ユーザー: 「アーキテクチャの今の理解を残しておいて」
→ `<repo-root>/___naito___/wip/about-repo/architecture.md` を読み、既存内容を踏まえて追記

ユーザー: 「issue 1234 のあのメモ参照して」
→ `<repo-root>/___naito___/issue/1234/` を `ls`、md を Read

## 注意

- 本 skill は repo の `.git` 配下や tracked ファイルには触らない。`___naito___/` は gitignore 済みなので git 操作は不要
- `___naito___/` 配下を作成しても commit する必要はない
- worktree から書く場合も symlink 経由で親 checkout の実体に書かれることを念頭に置く(同一実体を共有するので worktree 間で内容は共通)
