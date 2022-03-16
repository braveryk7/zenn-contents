---
title: "GitHub CLIにgh searchコマンドが実装されたので試す"
emoji: "👀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["githubcli", "github"]
published: true
---
## GitHub CLI v2.6.0がリリース
v2.5.2で`list`のエイリアスで`ls`が追加されたGitHub CLIですが、この度[v2.6.0がリリース](https://github.com/cli/cli/releases/tag/v2.6.0)されたようです。
特にOSS活動をされている方にとって嬉しいアップデートなので目玉機能である`gh search`コマンドをみていきます！

## gh searchコマンド
gh searchコマンドはGitHub内を検索するコマンドですが、v2.6.0時点ではサブコマンドとして`repos`が指定できます。

```shell
$ gh search repos <OPTIONS>
```

### オプション
オプションは色々ありますが、実用的なものをいくつかピックアップします。

#### --good-first-issues
`--good-first-issues`はgood-first-issuesラベル件数を検索条件に指定できます。

```shell
$ gh search repos --good-first-issues=">=100"
```

例えばこのオプションはgood-first-issuesラベルが付いたissueが100件以上あるリポジトリの一覧を取得します。

#### --language
`--language`オプションは使用言語を指定できます。

```shell
$ gh search repos --language=typescript
```

#### --limit, -L
`--limit`オプションでコマンドを叩いた時の表示件数を指定できます。
なおデフォルトでは30件が上限です。

```shell
$gh search repos --language=rust --limit 100
```

エイリアスで`-L`も使用できます。

#### --sort
`--sort`オプションを使うと指定した条件でソートできます。

```shell
$ gh search repos --language=php --sort forks
```

以下の条件を指定できます。

* forks
* help-wanted-issues
* stars
* updated

#### --stars
`--stars`オプションでstar件数を検索条件に指定できます。

```shell
$ gh search repos --language=go --stars=">=50000"
```

5万件以上starがついているリポジトリを指定しています、`--language`と組み合わせると今その言語で熱いフレームワークやプロジェクトを調べる指標になりそうですね！

#### --web
`--web`オプションで検索条件をGitHubのWeb上で検索できます。

```shell
$ gh search repos --language=javascript --stars=">=150000" --web
```

![](/images/20220316/2022-03-16-17.26.45.png)

GitHubでなにか検索したいけど一々開いて検索画面に行く手間が省けていいですね！

#### --help
`gh search repos`コマンドの他のオプションは`--help`コマンドで確認ができます。

```shell
$ gh search repos --help
```

## v2.6.0 Release note
詳しい変更は以下のリリースノートに掲載されています。

[GitHub CLI 2.6.0](https://github.com/cli/cli/releases/tag/v2.6.0)