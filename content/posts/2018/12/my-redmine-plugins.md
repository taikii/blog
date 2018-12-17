---
title: "私の作ったRedmineプラグイン"
date: 2018-12-19T00:00:00+09:00
tags: ["Redmine"]
series: []
---

[Redmine Advent Calendar 2018](https://adventar.org/calendars/3352) 19日目の記事です。

これまでいくつかRedmineのプラグインを（自分用に）作ってきました。どれもわずか数ステップの、大したものではないのですが、個人的には便利に使ってるので布教を兼ねてまとめて紹介したいと思います。

<!--more-->

### [Redmine Custom Auto-Complete](https://github.com/taikii/redmine_custom_auto_complete)

最初に作ったプラグインです。テキストのカスタムフィールドにオートコンプリートを追加します。最近使われた順に10件並び、チケット件数も表示されます。

![](https://github.com/taikii/redmine_custom_auto_complete/raw/master/doc/images/example.png)

Redmine にはバージョンやカテゴリというリスト項目がありますが、あれらはプロジェクト毎に作成できるし、Issue画面で直接作れるし、管理者以外に作成する権限を渡すこともできますよね。簡単に追加できる上、入力値のゆらぎを抑えることができるので、そんな選択項目がもういくつか欲しかったんです。しかし、Redmineで項目を追加するとなるとカスタムフィールドになるわけですが、テキスト形式だと自由すぎるし、リスト形式だと管理者しか編集できない…。

そこで、目をつけたのが関連チケットを登録する際のオートコンプリートでした。オートコンプリートであれば、ゆらぎを抑えられて負担も少ない。しかも、この関連チケットのオートコンプリートライブラリは、エンドポイントさえ作ってやれば使い回せるものだったので、これを拝借してテキスト形式のカスタムフィールドで動くようにしました。

ちなみに、このプラグインはRedmine導入の際に「Excelのほうがマシ」と言わせないためのカナメでした。みんな普通に使ってるけど、俺、頑張ったんだっぜ…？

### [Redmine Include Macro Extension](https://github.com/taikii/redmine_include_macro_extension)

Redmine組み込みのincludeマクロはWikiページ全体を挿入しますが、それをセクション単位で挿入できるように拡張したプラグインです。`{{include(Page, Section)}}`のように第2引数にセクションのタイトルを書くことで、そのセクションだけを挿入できます。また、オプションを指定することで「タイトルを含めない」「子のセクションを含めない」ようにすることもできます。

![](https://i.gyazo.com/860ea5b19c73d667a761c9f6eb20857d.png)

本家の [#3547](https://www.redmine.org/issues/3547) で要望されている機能をプラグインとして実装したものです。このチケットでは`{{include(Page#Section)}}`という記述方法を提案されています。これだとプラグインをアンインストールしたときにエラーになってしまうので（`Page#Section`というWikiページを探しに行って存在しないのでエラーになる）、それが嫌で第2引数をとるようにしました。また、私はRedmine本体へのパッチを書くつもりはないので、誰かパッチ化してくれてもよろしくてよ？

Redmineには元々セクション単位に編集する機能があるため、セクション単位にWikiデータを抜くメソッドなりがあることはわかっていました。あとは該当のタイトルか何番目のセクションかを判別すればいいだけだったので、結構すぐに実装できた記憶があります。

いや〜、Rubyのscanって本当にいいものですね〜。

### [GitHub Style Fenced Code Block Plugin](https://github.com/taikii/redmine_github_style_fenced_code_block)

WikiFormattingがMarkdownの際に、ツールバーでコードブロックを作ると「`~~~`」になるのですが、それを「```」に変更するプラグインです。

![](https://github.com/taikii/redmine_github_style_fenced_code_block/raw/master/docs/images/screenshot.gif)

[#22843](https://www.redmine.org/issues/22843) で [@g_maeda](https://twitter.com/g_maeda) さんがパッチを当ててくれたので、Redmine 4.0からは「```」がデフォになり、晴れて御役御免となりました。それにしても無事リリースされてよかったですね、4.0。

そもそも三連バッククォート（書けない…）のほうがメジャーやろっていうのもあるんですが、Markdownでは「`~~`」（二連チルダ）が打ち消し線に割り当てられていて、全く意味が違う割に似通っているので、見分け付きづらッ！というのが理由です。

JSのツールバーボタンPrototypeを上書きする形で実装しました。

### [Mermaid Macro](https://github.com/taikii/redmine_mermaid_macro)

Mermaid.jsを書けるようにするマクロプラグインです。私はMermaid大好きっ子です。

![](https://github.com/taikii/redmine_mermaid_macro/raw/master/doc/images/example.png)

経緯は[こちらを見てください](https://taikii.net/posts/2017/10/redmine-mermaid-macro-plugin/)。

前はWikiの添付した`mermaid.js`でもイケてた気がするけど、なんかできないな…

### [BR Macro](https://github.com/taikii/redmine_br_macro)

改行するためだけのマクロプラグインです。

![](https://gyazo.com/f61ba57772278a3cc10e5a44f2c63019.png)

Markdownの場合、通常は行末にスペース2つ置くことで段落ではない改行をすることができますが、テーブル内ではなぜかこれができなかったのでマクロを作りました。引数に改行数を渡すことで無限に改行することができる…！！

### さいごに

本当ならAdvent Calendar用に新しいプラグインを作りたかったのですが、ゴタゴタしててできませんでした。無念。

コードブロックのやつ以外は Redmine 4.0 で動確済です。

StarとかIssueとかStarとかPRとかStarとか大歓迎です。Starとか、Starとか。