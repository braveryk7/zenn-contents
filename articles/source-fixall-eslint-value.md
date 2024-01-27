---
title: "VSCodeのeditor.codeActionsOnSaveの指定方法がboolean値から変更されていた"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vscode", "eslint", "stylelint"]
published: true
---

# 新しい指定方法

`boolean`値ではなく、以下のいずれかを文字列で指定する。

* `explicit`
* `always`
* `never`

```json
# 今まで
"editor.codeActionsOnSave": {
    "source.fixAll.eslint": true,
    "source.fixAll.stylelint": true,
},

# これから
"editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit",
    "source.fixAll.stylelint": "always",
},
```

# 詳細
この変更は2023年11月のアップデートで行われたようで、長年当然のように設定していた`editor.codeActionsOnSave`の各設定項目が`boolean`値から`enum`値に変更されたようです。

おせっかいな事に、恐らくVSCode側が勝手に`settings.json`の`editor.codeActionsOnSave`に変更を加えます。

~~おかげで全く無関係なcommitに何故か`settings.json`が入っていてマジで迷惑しました、やめろ~~

このアップデートはVSCode v1.85.0で実装されました。

https://code.visualstudio.com/updates/v1_85

詳細は[Editor -> Code Actions on Save and Auto](https://code.visualstudio.com/updates/v1_85#_code-actions-on-save-and-auto)から確認できます。

## 設定値

boolean値から以下の3種類の文字列値に変更されました。

|設定値|動作|
|:--:|:--:|
|`explicit`|保存されたときにアクションを実行、以前の`true`と同じ。|
|`always`|保存、ウインドウ変更、フォーカス変更時にアクションを実行。|
|`never`|保存されたときにアクションを実行しない、以前の`false`と同じ。|

つまり今まで`true`だった人は`explicit`、`false`だった人は`never`を指定すればOK。

`always`は新しく追加された項目で、説明によると今まで以上の頻度でアクションを実行してくれるようです。

早速期待して`always`を試してみたんですが、少なくとも`eslint`はウインドウ切り替えやフォーカスをファイル一覧やVSCode内包ターミナルに切り替えても自動整形してくれませんでした。

もしかしたら拡張機能側の対応も必要なのかも知れません（どなたかできた方、ご教示頂けると嬉しいです）。