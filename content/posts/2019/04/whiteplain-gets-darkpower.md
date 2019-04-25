---
title: "Whiteplain gets DarkPower"
date: 2019-04-26T00:15:02+09:00
tags: ["Hugo"]
---

・・・・・・暗・・・こ・・・・・・・ロ・・・も・・チ・・・・・・。

<!--more-->

流行ってますね、ダークモード。Windows10にもダークモードが実装されたり、昨日リリースされたばかりのChrome74もダークモードに対応しました。私もダークモードを愛用してますが、いかにブラウザがダークモード対応しても、肝心のWebサイトは白基調のサイトが多いのであまり恩恵を受けられていない感じがします。かく言う私もWhiteplainとかいう白基調のテーマを作ってるわけで。。。これじゃいかん、ということで、[Whiteplain Dark](https://github.com/taikii/whiteplain-dark)というなんだかよくわからん名前の黒基調のテーマを作りました。

Hugoはテーマを複数指定して継承させることができるので、これを利用してWhiteplainの背景色と文字色だけ変えることでダークモードを作り出しています。

Readmeにも書いていますが、 `comfig.toml` に `whiteplain-dark` と `whiteplain` の2つのテーマを指定します。`whiteplain-dark` を先に指定する必要があります。

```toml
theme = ["whiteplain-dark", "whiteplain"]
```

また、Whiteplainは `static/css/custom.css` というファイルを作ることで、簡単に独自のスタイルで上書きできるようにしているのですが、このファイルを作っている方は `custom.css` の1行目に以下の行を追加する必要があります。

```css
@import url(./whiteplain-dark.css);
```

これでダークパワーを感じることができます。

このテーマ継承機能は、ショートコードだけのテーマを作ってプラグイン的に使うことが目的のような気もするんですが、これを使えばもっといろいろできそうですねー。
