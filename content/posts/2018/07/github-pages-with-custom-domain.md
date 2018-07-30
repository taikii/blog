---
title: "GitHub Pagesを独自ドメインで運用する"
date: 2018-07-30T18:00:19+09:00
tags: ["GitHub"]
series: []
---

このブログはGitHub Pagesにホスティングされています。先日ブログのURLを [https://taikii.net](https://taikii.net/) という独自ドメインに変更したのですが、このカスタムドメインの設定がお昼休み中にできちゃうくらい簡単な作業でしたので、ちょっと紹介したいと思います。

<!--more-->

※設定中にスクショを撮り忘れたので、初めて設定する際の画面と異なります。

## ドメインを取得する

何はともあれ独自のドメインが必要です。ちょうど[ムームードメイン](https://muumuu-domain.com/)で .net が安かったのでそれにしました。

個人的にムームードメインのいいところ。

- [今年の2月からALIASレコードを使用できるようになりました](https://muumuu-domain.com/?mode=info&id=3775)。ホスト名に対してCNAMEをつける場合はwwwなどのサブドメインである必要がありますが（つまり`https://taikii.net`ではなく`https://www.taikii.net`にする必要がある）、ALIASレコードであればサブドメインのないドメインに対してホスト名を指定することができます。
- 無料でWHOISの代理公開してもらえる（ドメインによってはできないものもありそう）
- コンビニ払いができる（ただし、[事務手数料が162円かかります](https://muumuu-domain.com/?mode=guide&state=pay_howto)）

基本的には流れのままに設定すればよいのですが、WHOIS設定は必ず「弊社情報代理公開」を選択します。じゃないと自分の個人情報がWHOISに載ってしまいます。また、ネームサーバは「ムームーＤＮＳ」を選択します。

{{< figure src="https://gyazo.com/18572fb205282d620ab5109d8d49bcbf/thumb/1000" >}}

その後、[カスタム設定のセットアップ方法 | ムームードメイン](https://muumuu-domain.com/?mode=guide&state=custom_setup)を参考にしてカスタム設定を行います。今回は、[https://taikii.net](https://taikii.net/) にアクセスされると [https://taikii.github.io](https://taikii.github.io/) に飛ばしたいため、以下のようにALIASレコードを設定します。

{{< figure src="https://gyazo.com/86bf24488c6dd41d8e228641626999a0/thumb/1000" >}}

これでムームードメイン側の設定は完了です。

## GitHub側の設定

GitHub PagesプロジェクトのSettingページにアクセスして、Custom domain項目に取得したドメインを入力します。ついでに`Enforce HTTPS`にチェックを入れてHTTPSを強制します。設定の反映まで少し時間がかかりますので、コーヒーでも淹れつつ待ちます。カスタムドメインのSSL証明書は[Let's Encryptの証明書が自動的に発行・適用されます](https://blog.github.com/2018-05-01-github-pages-custom-domains-https/)。[Chrome68で非暗号化サイトには警告が表示されるようになった](https://japan.cnet.com/article/35122977/)ので簡単にサイトをHTTPS化できるのはありがたい！

{{< figure src="https://gyazo.com/634355b2dde94ab6859e7ff8f772aaba/thumb/1000" >}}

ブラウザで取得したドメインを開いて確認します。`Enforce HTTPS`を設定したので、`http://`にすると`https://`に飛ばされるはずです。

これですべての設定が完了です。超簡単。

