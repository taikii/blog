---
title: "ブラウザのカラースキームによってWhiteplainの明暗を切り替わるようにしました"
date: 2019-10-29T23:27:40+09:00
tags: ["Hugo", "CSS"]
series: []
---

Chrome 78で試験的にWebページにダークモードを強制する機能が実装されたというニュースから、`prefers-color-scheme`というブラウザのカラースキームによってスタイルを切り替えるメディアクエリがあることを知りました。Hugoのテーマ[Whiteplain](https://github.com/taikii/whiteplain)の[暗色版](https://github.com/taikii/whiteplain-dark)を作ったときから、自動で切り替わるといいのに…とは思っていたので、このメディアクエリを使って明色と暗色が切り替わるようにしました。

<!--more-->

[Chrome 78公開。未対応ページを強制ダークモード化、保存したパスワードの安全度確認機能など搭載 - Engadget 日本版](https://japanese.engadget.com/2019/10/23/chrome-78/)

[prefers-color-scheme - CSS: カスケーディングスタイルシート | MDN](https://developer.mozilla.org/ja/docs/Web/CSS/@media/prefers-color-scheme)

## 明色または暗色のままにする

ユーザの環境によらず明色または暗色のままにする場合は、[Whiteplain Light](https://github.com/taikii/whiteplain-light) / [Whiteplain Dark](https://github.com/taikii/whiteplain-dark)を`themes`ディレクトリに入れて、`config.toml`で`whiteplain`の左側に併記します。

ちなみに`whiteplain-light`は今回新しく作りました。

```toml
#theme = ["whiteplain-light", "whiteplain"]
theme = ["whiteplain-dark", "whiteplain"]
```

ただし、上記のChrome 78のForce Darkが有効な場合は、Whiteplain Lightを指定しても自動で暗色にされます。。。

## Chrome 78のForce Darkについて

Chrome 78のForce Dark Mode for Web Contentsは、`prefers-color-scheme`メディアクエリのブラケット内のカラー指定は変更しませんが、それ以外は暗色化します。

Whiteplainの場合、構造系のCSSと色系のCSSがごちゃまぜになっていたので、Whiteplain Darkを作ったときは色系のCSSだけパッチを当てる感じで実現しました。今回はその暗色CSSパッチを`prefers-color-scheme: dark`メディアクエリで自動適用するようにしただけなので、明色のCSSは`prefers-color-scheme`のブラケットで囲われていません。このため、Force Darkの対象となってしまいます。

## CSSファイルのインポート

構造系のCSSと色系のCSSを分離して、`prefers-color-scheme: light`だろうが`prefers-color-scheme: dark`だろうが明色のCSSを読み込むようにしてやればいいと思ったのですが、一つのCSSファイルは二度読み込まれることがない（？）ため、下記の例だとダークモードの時に色系Style全く効かなくなってしまいます。

さらに、下記の例では、`prefers-color-scheme`に対応してないブラウザ（例えばIE）でも`color-light.css`が適用されません。IEを考慮すると明色か暗色を`prefers-color-scheme`ブラケットの外に置く必要があります。

```css
@import url(./structure.css);

@media (prefers-color-scheme: light) {
  @import url(./color-light.css);
}

@media (prefers-color-scheme: dark) {
  /* color-light.cssは上でインポート済みなので、インポートされない？＝ダークモードで色系Styleが適用されない */
  @import url(./color-light.css);
}
```

あっちを立てればこっちが立たずって感じです。。。

まぁ、現時点でForce Darkは試験機能なので、あまり考えなくていいんですけど。

## ToDo

* テーマのスクリーンショットを明暗の両方を表現したものに変更する。
* whiteplain-lightテーマは、Chromeがダークモードを強制しても明色で表示できるようにする。
* 明色の色系のスタイルを抜き出して別のCSSファイルにする。
