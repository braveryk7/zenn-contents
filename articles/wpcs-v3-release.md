---
title: "遂にwpcs新バージョンがリリースされ、PHP8系でも動作可能に"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["wordpress", "php", "phpcodesniffer", "wpcs"]
published: true
---

遂に待ちに待ったwpcs v3.0.0がリリースされました！！！！

# wpcs is なに
[wpcs](https://github.com/WordPress/WordPress-Coding-Standards)はWordPressのコーディング規約に準じたコード品質に準拠するよう支援するPHP_CodeSniffer用のルールセットです。
composerで配布されているのでcomposerが導入されていればすぐに導入できる便利なやつです。

この度遂に、待ちに待った[ｖ3.0.0](https://github.com/WordPress/WordPress-Coding-Standards/releases/tag/3.0.0)がリリースされました。

[前回のアップデート（v2.3.0）](https://github.com/WordPress/WordPress-Coding-Standards/releases/tag/2.3.0)が2020年5月なので、おおよそ3年半振りのリリースとなります。

## v2系の問題点
v2系は2020年5月にリリースされたため、2020年11月にリリースされたPHP8.0以降には未対応でした。
そのためPHP8系では動作しない既知の問題がありました。
> https://github.com/WordPress/WordPress-Coding-Standards/issues/2070

正確にはPHP8.0系では動作していたんですが、PHP8.1系以降では全く動作していなかった状況です。

# wpcs v3.0.0の概要
v3.0.0では主に以下のような機能追加・改善・変更がありました。

* PHP_CodeSniffer（phpcs）の最小要件がv3.7.2以上になりました
* インストールはcomposerのみがサポートされます（git clone/pear/pharはインストール可能ですが未サポート）
* filter / libxml / XMLReaderパッケージが明示的に必要になりました
* 多くのsniff（ルール）の名前が変更・もしくは削除されました
* PHPCSUtils / PHPCSExtra / Composer PHPCS pluginが依存パッケージとして設定されています
* PHP7.2以降のいくつかの未対応だったsniffに対応しました

## PHP_CodeSnifferの最小要件
前バージョンまでPHP_CodeSniffer v3.3.1が必須でしたが、v3.0.0からPHP_CodeSniffer v3.7.2（2023年9月4日時点の最新バージョン）が必須になりました。

## インストール方法
今まではgit cloneやpear、pharを使ったインストール方法もサポートされていましたが、v3.0.0からcomposerを用いたインストールのみサポートされます。

別途、以下のコマンドを実行する必要があります。

```bash
$ composer config allow-plugins.dealerdirect/phpcodesniffer-composer-installer true
```

これはcomposerで`dealerdirect/phpcodesniffer-composer-installer`というプラグインを許可する設定です。
`dealerdirect/phpcodesniffer-composer-installer`はPHP_CodeSnifferのヘルパーで、インストール時に色々やってくれる子です。

実行すると`composer.json`に以下が追記されます。

```json:composer.json
"config": {
    "allow-plugins": {
        "dealerdirect/phpcodesniffer-composer-installer": true
    }
}
```

`composer require`や`composer update`の前に実行、もしくは`composer.json`に追記しておきましょう。

## パッケージ
filter / libxml / XMLReaderパッケージが明示的に必要になりました。
なおMbstringとiconvも推奨されています。
なんらかのエラーや不具合があった場合、この辺のパッケージの有無や稼働状況をチェックすると解決の糸口になるかもしれません。
（基本的にPHPコアに含まれている拡張モジュールなのでDockerで通常PHPイメージを指定したりhomebrew等でphpパッケージを使用している場合はデフォルトで使用できるはずです）

## sniff（ルール）の名前変更・削除
多くのsniff名称が変更または削除されています。

一例として、PHPの配列短縮表記に関する`Generic.Arrays.DisallowShortArraySyntax`が`Universal.Arrays.DisallowShortArraySyntax`に変更となりました。

詳しい一覧は[Upgrade Guide to WordPressCS 3.0.0 for ruleset maintainers](https://github.com/WordPress/WordPress-Coding-Standards/wiki/Upgrade-Guide-to-WordPressCS-3.0.0-for-ruleset-maintainers)に記載されているので必ず確認しておきましょう。

今まで効いていたカスタムルールがおかしくなった場合だいたいこれが原因だと思われます。

## 依存パッケージ

以前のバージョンまで、wpcsの依存関係はPHP5.4以上とPHP_CodeSniffer v3.3.1以上だけでした。

```json:composer.lock
{
    "name": "wp-coding-standards/wpcs",
    "version": "2.3.0",
    // 中略
    "require": {
        "php": ">=5.4",
        "squizlabs/php_codesniffer": "^3.3.1"
    },
}
```

v3.0.0からは以下のように変更されています。

```json:composer.lock
{
    "name": "wp-coding-standards/wpcs",
    "version": "3.0.0",
    // 中略
    "require": {
        "ext-filter": "*",
        "ext-libxml": "*",
        "ext-tokenizer": "*",
        "ext-xmlreader": "*",
        "php": ">=5.4",
        "phpcsstandards/phpcsextra": "^1.1.0",
        "phpcsstandards/phpcsutils": "^1.0.8",
        "squizlabs/php_codesniffer": "^3.7.2"
    },
}
```

PHP5.4以上が必須な点は変わりませんが、PHPの拡張モジュール（ext-から始まるパッケージ）やPHPCSExtra、PHPCSUtilsが追加されています。

そしてPHP_CodeSnifferはｖ3.7.2以上が指定されていますね。

基本的に新規プロジェクトなら問題ありませんが、既存プロジェクトでcomposer updateを使用してパッケージの更新をする時は`--with-dependencies`オプションを明示的に使用しないとPHPCSUtilsやExtra等がインストールされない点に注意してください。

```bash
# 新規プロジェクト
$ composer require wp-coding-standards/wpcs:3.0.0

# 既存プロジェクト
$ composer update wp-coding-standards/wpcs:3.0.0 --with-dependencies
```

ちなみに私は依存関係の更新がされていると思わず見切り発車で`--with-dependencies`をつけ忘れたところ「PHPCSUtilsが見つからない！」と怒られました。
その場合は`composer install`コマンドを叩いて最新化させればOKです。

## 未対応sniffの追加
ついにPHP8系以降の新記法に対応しました。
PHP7.4のアロー関数やPHP8.0のmatch式等です。

```php:.php
# あえて()の中に2つスペースを挿入したmatch式
$expression = match (  $int ) {
	1 => 'one',
	2 => 'two',
	default => 'other',
};
```

例えばこのようにmatch式の()の中身のスペースがおかしい（このケースでは$intの前に2つスペースがある）場合、前バージョンまでは指摘してくれませんでした。

v3.0.0からは`Expected exactly one space after opening parenthesis; "  " found.`というエラーを指摘してくれるようになりました。

できれば変数の`=`を揃えるようにと指摘するように、`=>`を揃えるようにしてほしいと思ったり。

個人的には開発しているWordPressプラグインの最小要件にPHP8.0以上を指定して7系ユーザーが利用できないようにしておりモダンなPHP記法が増えているのでとても嬉しい改善です。

# WordPress開発もモダン化しよう
WordPressのエコシステム全体で見るとgutenbergリポジトリでは積極的にTypeScriptが導入されていたり、様々な箇所でレガシー対応が少しずつですが進んでいます。

もちろんWordPress自体がユーザーが様々なバージョンを使用しているため簡単に破壊的変更が行えないのですが、我々デベロッパーがモダンな開発環境を取り入れることはエコシステム全体にとって良いことなのでぜひ積極的に取り入れたいですね。

中でもPHPのPHP_CodeSniffer、JavaScript/TypeScriptのESLint等のツールはダイレクトにコード品質に影響してコーディング規約に沿った読みやすいコード品質を保つのに非常に便利です。

もし今PHP_CodeSniffer / wpcsを使っていない場合本当に便利なのでぜひ導入してみてください！