---
title: "Microsoft EdgeのIEモードについて調べてみた"
date: 2021-06-27T17:36:04+09:00
tags: ["Edge"]
series: []
---
Microsoft Edge (Chromium) の Internet Explorerモード（IEモード）について調べたことをまとめます。（[Internet Explorer モードとは \| Microsoft Docs](https://docs.microsoft.com/ja-jp/deployedge/edge-ie-mode)）

<!--more-->

まず、

* IEモードはエンタープライズ向けの機能なので、Windows 10 Homeでは基本的に使えない。（レジストリいじれば使える？）
* IEモードを有効にするには、[まずグループポリシーを追加する必要がある](https://docs.microsoft.com/ja-jp/deployedge/edge-ie-mode-policies#enable-internet-explorer-integration-using-group-policy)。

です。

IEモードに関して検索すると、よく「『その他のツール』から『Internet Explorerモードで再度読み込む』を選択」という記述を見ます。GPOを設定するとこのメニューが出てくるんじゃないかと思います。

この『Internet Explorerモードで再度読み込む』をすると、そのタブはIEコンポーネントで動きます。この場合、どこをどう画面遷移しようとIEモードのままです。

で、巷で言われている「IEを前提としたレガシーシステムが存在する場合はIEモードを使え」というのは、利用者自身が操作するこの『Internet Explorerモードで再度読み込む』機能のことではないです。

Edgeはアクセスするサイトによって、「Edgeモード？（Chomium）」と「IEモード」を自動的に切り替えることができます。企業ではこの自動切り替えの機能を使って、社員に意識させずにIEモードを導入していくことになります。

この自動切り替えのやりかたは大きく2つあります。ひとつは「エンタープライズモードサイトリストを作成してルールとして使う」、もうひとつは「ローカルイントラネットゾーンのサイトすべてをIEモードで開く」です。MSさんでは前者を推奨しています。そりゃそうですね、後者を選択してしまうとIEモードで開いてUIがぶっ壊れるシステムもでてくるでしょう（IEサポートしていないWebデザイン系のフレームワークも珍しくなくなったし）。
（[サイトを Microsoft Edge から IE モードにリダイレクトする](https://docs.microsoft.com/ja-jp/deployedge/edge-ie-mode-policies#redirect-sites-from-microsoft-edge-to-ie-mode)）

前者の「エンタープライズモードサイトリスト」とは、エンタープライズモード…つまり「互換表示」で表示するサイトを指定するためのXMLファイルです。フォーマットが同じというだけで、エンタープライズモードを指定するファイルと別でもいいみたいです。すでにエンタープライズモードサイトリストを導入している場合は、流用できるメリットがありそうです。

```xml
<site-list version="1">
  <site url="example.com">
    <open-in>IE11</open-in>
  </site>
</site-list>
```

このスキーマ定義は [エンタープライズ モード スキーマ v\.2 ガイダンス](https://docs.microsoft.com/ja-jp/internet-explorer/ie11-deploy-guide/enterprise-mode-schema-version-2-guidance) に解説があります。また、GUIで編集する [Enterprise Site List Manager](https://docs.microsoft.com/ja-jp/deployedge/edge-ie-mode-site-list-manager) というツールもあります。注意点として、ガイダンスにもある通り以下がハマりどころです。

> * プロトコルは使用しないでください。 たとえば、http://、https://、またはカスタム プロトコルです。 これらによって解析が中断されます。
> * ワイルドカードは使用しないでください。
> * クエリ文字列は使用しないでください。アンパサンドによって解析が中断されます。

エンタープライズモードサイトリストに含まれるサイトと、含まれないサイトを行き来すると、IEモードとEdgeモードがパチパチ切り替わります。リストに含まれるサイトからリストに含まれないサイトに移動した際に自動的にEdgeモードに戻らないように設定することもできますが、これは推奨されていません。（[Internet Explorer モードでページ内ナビゲーションを保持する](https://docs.microsoft.com/ja-jp/deployedge/edge-learnmore-inpage-nav)）

IEモードとEdgeモードは内部的には別のブラウザなのですが、サイトリストに設定をすることによってCookieの共有をすることもできます。[Microsoft Edge から Internet Explorer への Cookie の共有 \| Microsoft Docs](https://docs.microsoft.com/ja-jp/deployedge/edge-ie-mode-add-guidance-cookieshare) にある通り、Edge→IEモードへの向きの共有のみであり、逆向きに共有はできません。また、共有できるCookieはオンセッションのCookieのみです。

しかし、Edgeから渡されるCookieは「インターネットゾーン」として判断される（か、「保護モード」の対象になる）ようで、Cookieを渡したいIEモード側のサイトが「信頼済みサイトゾーン」である場合（たぶん「ローカルイントラネットゾーン」も）、Cookieを共有することができません。Edgeにはゾーンの概念がないのでしょうがないです。

SSOの認証サーバから発行される認証Cookieを例にとると、以下のような感じになります。SSOの製品によって多少違ったりすると思います。

1. IEモード対象の WebアプリケーションA （信頼済みサイト）にアクセスする
2. 認証されていないので、認証サーバの認証画面が表示や認証画面へのリダイレクトがされる
3. 認証画面はIEモード対象外なのでEdgeモードで認証画面が表示される
4. ログインする（EdgeにCookieが保存される）
5. 最初の WebアプリケーションA を表示しようとするが、認証Cookieが共有されていないので、認証画面が表示や認証画面へのリダイレクトしようとする
6. Edgeモードで認証画面が表示しようとするが、認証済みなので5にもどる

WebアプリケーションA にフォーカスすると、認証画面もIEモードの対象にしてやれば解決します。しかし、同じSSOを使用し、Edgeモードで動作する別の WebアプリケーションB がある場合、認証画面をIEモードに入れてしまうとIEモード→EdgeモードのCookieの共有はされないので、今度は WebアプリケーションB で上の事象が発生することになります。

この問題を解決するために、直前のブラウザモードに従う「ニュートラル」という設定を使用します。認証画面を「ニュートラル」に登録しておけば、IEモードでもEdgeモードそれぞれで認証画面を表示することができます。ニュートラルは `<open-in>None</open-in>` のように指定します。

```xml
<site-list version="1">
  <site url="example.com">
    <open-in>IE11</open-in>
  </site>
  <site url="login.example.com">
    <open-in>None</open-in>
  </site>
</site-list>
```

ここまで書いといてあれですが、ほぼ [認証がうまくいかないケース - IE モードのよくあるご質問 \| Japan Developer Support Internet Team Blog](https://jpdsi.github.io/blog/internet-explorer-microsoft-edge/ie-mode-faq/#%E8%AA%8D%E8%A8%BC%E3%81%8C%E3%81%86%E3%81%BE%E3%81%8F%E3%81%84%E3%81%8B%E3%81%AA%E3%81%84%E3%82%B1%E3%83%BC%E3%82%B9) の丸写しです。当たり前ですがMSさんの記事のほうがより詳しく書かれています。

このIEモード、よくできてますが、利用者の意識しないところでパチパチ切り替わるので、そのへんがトラブルにつながる気はします。
MSさんの戦略もわかるけど、デスクトップアプリとして分離していたほうが利用者のサポートはしやすかっただろうと思います。

とはいえ、IEモードは本当に延命措置と考えて、IEでしか動かないレガシーアプリを抱えている場合はすぐにでもリプレースに走り出すべきだと私は思います。MSさんはIEモードを2029年までサポートすると言っていますが、それまでフル機能がサポートされるとは限りません。もしかして、Windows 11 のEdgeはIEモードなかったりして？
また、非推奨の設定を使うことも極力避けたほうがいいと思います。私は「作成者が意図したように使う、アクロバティックな使い方をしない」が信条です。非推奨の設定を使うと後々つらい思いをすることになる気がします。