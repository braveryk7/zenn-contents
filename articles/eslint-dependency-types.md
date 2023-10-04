---
title: "【ESLint】依存関係によってインストールされた型定義ファイルがリストに無いと怒られた時の対処法"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["typescript", "react", "eslint", "wordpress"]
published: true
---

# 依存関係としてインストール済みの型定義ファイルがリストに無いと怒られた時の対処法

## 前提要約
* 型定義ファイルがnpmパッケージの依存関係としてインストールされた
* つまり自分で明示的にインストールしていないが、`node_modules/`配下に存在している状態
* ESLintに`'@types/react' should be listed in the project's dependencies. Run 'npm i -S @types/react' to add it`と怒られた

## A.ESLintの設定ファイル内で`import/no-extraneous-dependencies`の設定を記述する。

以下のように、ESLintの設定ファイルに`import/no-extraneous-dependencies`の設定を記述する。

`import/no-extraneous-dependencies`は`eslint-plugin-import`に内包されているので無い場合はインストールして使用する。

```shell
$ npm i -D eslint-plugin-import
$ yarn add -D eslint-plugin-import
```

`import/no-extraneous-dependencies`の詳細は[公式のGitHubリポジトリ](https://github.com/import-js/eslint-plugin-import/blob/main/docs/rules/no-extraneous-dependencies.md)を参照。


## json

```json:.eslintrc.json
"rules": {
  "import/no-extraneous-dependencies": [
    "error",
    {
      "packageDir": [
        "./",
        "./node_modules/@wordpress/scripts/"
      ]
    }
  ]
}
```

## yaml

```yaml:.eslintrc.yml
rules:
  import/no-extraneous-dependencies:
    - error
    - packageDir:
        - ./
        - ./node_modules/@wordpress/scripts/
```

## 注意点

* `packageDir`セクションに依存関係が記述されている`package.json`が存在するディレクトリのpathを追加する
* `package.json`へのpathではなく、`package.json`が存在するディレクトリへのpathである点に注意
* 上記の場合、ルートディレクトリと`node_modules/@wordpress/scripts`ディレクトリの`package.json`が読み込まれる

# 背景
最近のWordPress開発にはJavaScript/TypeScriptを使用することがほとんどで、多くの場合`@wordpress/scripts`パッケージが使用されます。

この`@wordpress/scripts`パッケージには多くの依存関係が設定されており、中には`webpack`や`ESLint`等のパッケージが含まれています。

:::details node_modules/@wordpress/scripts/package.jsonの依存関係 (必要ない箇所は削除しています)
```json:package.json
{
	"name": "@wordpress/scripts",
	"version": "26.13.0",
	"dependencies": {
		"@babel/core": "^7.16.0",
		"@pmmmwh/react-refresh-webpack-plugin": "^0.5.2",
		"@svgr/webpack": "^8.0.1",
		"@wordpress/babel-preset-default": "^7.26.0",
		"@wordpress/browserslist-config": "^5.25.0",
		"@wordpress/dependency-extraction-webpack-plugin": "^4.25.0",
		"@wordpress/e2e-test-utils-playwright": "^0.10.0",
		"@wordpress/eslint-plugin": "^16.0.0",
		"@wordpress/jest-preset-default": "^11.13.0",
		"@wordpress/npm-package-json-lint-config": "^4.27.0",
		"@wordpress/postcss-plugins-preset": "^4.26.0",
		"@wordpress/prettier-config": "^2.25.0",
		"@wordpress/stylelint-config": "^21.25.0",
		"adm-zip": "^0.5.9",
		"babel-jest": "^29.6.2",
		"babel-loader": "^8.2.3",
		"browserslist": "^4.21.9",
		"chalk": "^4.0.0",
		"check-node-version": "^4.1.0",
		"clean-webpack-plugin": "^3.0.0",
		"copy-webpack-plugin": "^10.2.0",
		"cross-spawn": "^5.1.0",
		"css-loader": "^6.2.0",
		"cssnano": "^6.0.1",
		"cwd": "^0.10.0",
		"dir-glob": "^3.0.1",
		"eslint": "^8.3.0",
		"expect-puppeteer": "^4.4.0",
		"fast-glob": "^3.2.7",
		"filenamify": "^4.2.0",
		"jest": "^29.6.2",
		"jest-dev-server": "^6.0.2",
		"jest-environment-jsdom": "^29.6.2",
		"jest-environment-node": "^29.6.2",
		"markdownlint-cli": "^0.31.1",
		"merge-deep": "^3.0.3",
		"mini-css-extract-plugin": "^2.5.1",
		"minimist": "^1.2.0",
		"npm-package-json-lint": "^6.4.0",
		"npm-packlist": "^3.0.0",
		"playwright-core": "1.32.0",
		"postcss": "^8.4.5",
		"postcss-loader": "^6.2.1",
		"prettier": "npm:wp-prettier@3.0.3-beta-3",
		"puppeteer-core": "^13.2.0",
		"react-refresh": "^0.10.0",
		"read-pkg-up": "^7.0.1",
		"resolve-bin": "^0.4.0",
		"sass": "^1.35.2",
		"sass-loader": "^12.1.0",
		"source-map-loader": "^3.0.0",
		"stylelint": "^14.2.0",
		"terser-webpack-plugin": "^5.3.9",
		"url-loader": "^4.1.1",
		"webpack": "^5.47.1",
		"webpack-bundle-analyzer": "^4.4.2",
		"webpack-cli": "^4.9.1",
		"webpack-dev-server": "^4.4.0"
	},
	"peerDependencies": {
		"@playwright/test": "^1.32.0",
		"react": "^18.0.0",
		"react-dom": "^18.0.0"
	},
}

```
:::

他にも`@wordpress/scripts`の依存関係にある`@wordpress/babel-preset`には`@wordpress/element`が指定されています。

::: details @wordpress/babel-presetのpackage.json (必要ない箇所は削除しています)
```json:package.json
{
	"name": "@wordpress/babel-preset-default",
	"version": "7.26.0",
	"dependencies": {
		"@babel/core": "^7.16.0",
		"@babel/plugin-transform-react-jsx": "^7.16.0",
		"@babel/plugin-transform-runtime": "^7.16.0",
		"@babel/preset-env": "^7.16.0",
		"@babel/preset-typescript": "^7.16.0",
		"@babel/runtime": "^7.16.0",
		"@wordpress/babel-plugin-import-jsx-pragma": "^4.25.0",
		"@wordpress/browserslist-config": "^5.25.0",
		"@wordpress/element": "^5.19.0",
		"@wordpress/warning": "^2.42.0",
		"browserslist": "^4.21.9",
		"core-js": "^3.31.0"
	},
}

```
:::

更に`@wordpress/element`の依存関係を見てみると、`react`や`@types/react`が指定されています。

::: details @wordpress/elementのpackage.json (必要ない箇所は削除しています)
```json:package.json
{
	"name": "@wordpress/element",
	"dependencies": {
		"@babel/runtime": "^7.16.0",
		"@types/react": "^18.0.21",
		"@types/react-dom": "^18.0.6",
		"@wordpress/escape-html": "^2.42.0",
		"change-case": "^4.1.2",
		"is-plain-object": "^5.0.0",
		"react": "^18.2.0",
		"react-dom": "^18.2.0"
	},
}

```
:::

他にも依存関係が回り回って`TypeScript`やら`Prettier`やら`Jest`やらなんやらかんやらインストールされます。

つまり`@wordpress/scripts`をインストールするだけで、React開発がリンターやフォーマッターもある状態で開発できるという寸法です。

もちろんTypeScriptや関連パッケージもインストールされるので、`node_modules/@wordpress/scripts/config`内にある`webpack.config.js`を継承してTypeScript用に再構築すればTypeScriptで開発することも可能です。

# ESLintが型定義ファイルがインストールパッケージの依存関係の場合、型定義が依存関係リストに存在しないと指摘する
ここからが本題です。

先述したように、`@wordpress/scripts`パッケージの依存関係として様々なパッケージをインストールしたためプロジェクトルート上の`package.json`の`dependencies`欄にはほとんど記載がありません。

参考までに以下は私の開発中のプラグインの`package.json`の一部です。

```json:package.json
{
  "devDependencies": {
    "@wordpress/env": "^8.8.0",
    "@wordpress/scripts": "^26.13.0",
    "esbuild-loader": "^4.0.2",
    "eslint-import-resolver-typescript": "^3.6.1",
    "fork-ts-checker-webpack-plugin": "^8.0.0",
    "thread-loader": "^4.0.2"
  },
}

```

`@wordpress/scripts`パッケージで提供されていないTypeScriptで開発するのに必要なパッケージと、WordPressの仮想環境を提供する`@wordpress/env`ぐらいしかありません。

この状態で以下の記述を行ったところ、ESLintのエラーが出力されました。

```tsx:.tsx
import { Dispatch, SetStateAction } from 'react';
```

```:ESlintのエラー
'@types/react' should be listed in the project's dependencies. Run 'npm i -S @types/react' to add it
```

「`@types/react`が依存関係にリストされていないので、`npm i -S @types/react`を実行して追加してください」という旨のエラーです。

リストされていない、つまり型定義ファイルはたしかに`node_modules`内に存在しているが`package.json`に記述が無いということ。

```shell
# @types/reactパッケージが存在するかどうかチェック
$ npm list @types/react
example-project@ /Users/xxx/example-project
└─┬ @wordpress/data@9.12.0
  └─┬ @wordpress/element@5.19.0
    ├─┬ @types/react-dom@18.2.7
    │ └── @types/react@18.2.24 deduped
    └── @types/react@18.2.24
```

依存関係というのは`package.json`の`dependencies`または`devDependencies`セクションを指します。

今回のようにパッケージとして公開する訳ではないプロジェクトの場合は通常型定義ファイルを`devDependencies`に記述しますが、今回の場合`dependencies`セクションは存在せず`devDependencies`セクションには`@types/react`が記述されていません。

そのためESLintが「型定義がリストに無いよ」と怒っているわけです。

## import/no-extraneous-dependencies
そこで冒頭で記述した、`import/no-extraneous-dependencies`の設定です。

`import/no-extraneous-dependencies`は`eslint-plugin-import`が提供する機能の1つで、プロジェクトの`package.json`に依存関係として宣言されていない外部モジュールのインポートを禁止するものです。

ただし、冒頭に記述したように読み込んでほしい`package.json`のpathを明示的に示すことによって今回のようなエラーを回避することが可能です。

インストールされていない場合はnpm/yarn経由でインストールしてください。

```bash
# npmの場合
$ npm i -D eslint-plugin-import

# yarnの場合
$ yarn add -D eslint-plugin-import
```

私のようにWordPressの開発で、`@wordpress/scripts`パッケージをインストールしている場合は既にインストール済みのはずです。

`eslint-plugin-import`は`@wordpress/scripts`の依存関係に指定されている`@wordpress/eslint-plugin`の依存関係としてインストールされています。

::: details @wordpress/eslint-pluginのpackage.json (必要ない箇所は削除しています)
```json:package.json
{
	"name": "@wordpress/eslint-plugin",
	"version": "16.0.0",
	"dependencies": {
		"@babel/eslint-parser": "^7.16.0",
		"@typescript-eslint/eslint-plugin": "^6.4.1",
		"@typescript-eslint/parser": "^6.4.1",
		"@wordpress/babel-preset-default": "^7.26.0",
		"@wordpress/prettier-config": "^2.25.0",
		"cosmiconfig": "^7.0.0",
		"eslint-config-prettier": "^8.3.0",
		"eslint-plugin-import": "^2.25.2",
		"eslint-plugin-jest": "^27.2.3",
		"eslint-plugin-jsdoc": "^46.4.6",
		"eslint-plugin-jsx-a11y": "^6.5.1",
		"eslint-plugin-playwright": "^0.15.3",
		"eslint-plugin-prettier": "^5.0.0",
		"eslint-plugin-react": "^7.27.0",
		"eslint-plugin-react-hooks": "^4.3.0",
		"globals": "^13.12.0",
		"requireindex": "^1.2.0"
	},
}

```
:::

`node_modules`ディレクトリ内を直接探すか、`npm list`コマンドを使ってインストール済みかチェックしましょう。

```shell
$ npm list eslint-plugin-import
example-project@ /Users/xxx/example-project
├─┬ @wordpress/scripts@26.13.0
│ └─┬ @wordpress/eslint-plugin@16.0.0
│   └── eslint-plugin-import@2.28.1 deduped
└─┬ eslint-import-resolver-typescript@3.6.1
  └── eslint-plugin-import@2.28.1
```

## import/no-extraneous-dependenciesの設定

以下は`import/no-extraneous-dependencies`のオプションの一覧です。

| オプション名 | デフォルト値 | 設定できる値 | 内容 |
| :---: | :---: | :---: | ---- |
| エラーレベル | `error` | `error`, `warn`, `off` | 警告レベルを設定できる。 |
| `devDependencies` | `true` | `true`, `false`, 配列 | devDependenciesセクションのパッケージ認める（`true`）か認めない（`false`）か。または特定のファイルパターンの配列。 |
| `optionalDependencies` | `true` | `true`, `false` | optionalDependenciesセクションのパッケージ認める（`true`）か認めない（`false`）か。 |
| `peerDependencies` | `true` | `true`, `false` | peerDependenciesセクションのパッケージ認める（`true`）か認めない（`false`）か。 |
| `bundledDependencies` | `true` | `true`, `false` | bundledDependenciesセクションのパッケージ認める（`true`）か認めない（`false`）か。 |
| `packageDir` | なし | `string`, `array` | `package.json`ファイルが存在するディレクトリへのpath。複数の場合配列でも可。`package.json`へのpathではなくあくまでディレクトリへのpath。 |
| `includeInternal` | `false` | `true`, `false` | `package.json`に含まれない内部モジュールのチェックを行う（`true`）か行わない（`false`）か。 |
| `includeTypes` | `false` | `true`, `false` | `import type { ... } from '...'`構文時、型のインポートもチェック対象とする（`true`）かしない（`false`）か。 |

今回使用するのは`packageDir`オプションです。

通常であればルートディレクトリの`package.json`を探せば良いので`./`だけを指定すればOKですが、今回のようにルートディレクトリの`package.json`と`@wordpress/scripts`配下の`package.json`を指定したい場合配列で記述します。

私はyaml派なので以下のように記述しています（json派の方は冒頭のjsonを御覧ください）。

```yaml:.eslintrc.yml
rules:
  import/no-extraneous-dependencies:
    - error
    - packageDir:
        - ./
        - ./node_modules/@wordpress/scripts/
```

このように配列として複数のpathを設定することが可能です。

今回のように外部パッケージの`package.json`に依存している、またはMonorepoな環境の場合は適切にESLintの設定を行うことで脳死でエラー除外やインラインコメント連打によるignoreを行わなくても済みます。

どうでもいいですが今回`packageDir`に言及した日本語記事が全く無くて地味ハマりしました、ちゃんと英語記事読めってことですね・・・。