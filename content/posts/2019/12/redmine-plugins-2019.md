---
title: "私の作ったRedmineプラグイン2019"
date: 2019-12-15T00:00:00+09:00
tags: ["Redmine"]
series: []
---

[Redmine Advent Calendar 2019](https://adventar.org/calendars/4221) 15日目の記事です。今年作ったプラグインをまとめて紹介します。[去年と同じ]({{< relref "/posts/2018/12/my-redmine-plugins.md" >}})流れになってしまいました。本当はアドカレのためにもう一個作りたかったけど…勘弁して…。

<!--more-->

## [Callout Macro](https://github.com/taikii/redmine_callout_macro)

よくある注意書きを書くためのマクロです。TextileはCSSを書けるので目立つ文章を書くことができますが、Markdownの場合は太字にするか斜体にするくらいしかありません。そういう時にちょっと便利。Wikiフォーマットでも書けるよ。

```
{{info(INFO)
This is information block.
}}
{{warning(WARNING)
This is warning block.
}}
{{danger(DANGER)
This is danger block.
}}
```

![](https://github.com/taikii/redmine_callout_macro/raw/master/doc/images/sample.png)

[Redmine Callout Macroプラグイン - taikii blog]({{< relref "/posts/2019/05/redmine-callout-macro.md" >}})

## [Related Wiki](https://github.com/taikii/redmine_related_wiki)

[Redmine Tagsプラグイン](https://github.com/ixti/redmine_tags)がインストールされている必要があります。チケットに登録されているタグと同じ名前のWikiページへのリンクをサイドバーに表示します。Scrapboxはタグ＝Wikiページになっていてすごくキモチイイので、同じような使い勝手にならないかなと思って作りました。個人的にはめちゃめちゃ使ってますが他のメンバが使ってるかは謎。

![](https://github.com/taikii/redmine_related_wiki/raw/master/doc/images/screenshot.gif)

[Redmine: タグと同名のWikiページへのリンクを張るプラグインを作った話 - taikii blog]({{< relref "/posts/2019/05/redmine-related-wiki.md" >}})

## [TSV Macro](https://github.com/taikii/redmine_tsv_macro)

タブ区切りのテキストをテーブルで表示するためのマクロです。TextileもMarkdownもテーブルは鬼門です。Wikiページだと頑張ってテーブル書式で書くのですが、チケットでそこまで労力を使いたくなくて作りました。DBのデータを貼り付けるのによく使ってます。他のメンバが使ってるかは（以下略

```
{{tsv
Value1  Value2  Value3
}}
```

```
{{tsv_h
Head1 Head2 Head3
Value1  Value2  Value3
}}
```

[Redmine: タブ区切りのテキストをテーブルで表示するMacroを作った - taikii blog]({{< relref "/posts/2019/05/redmine-tsv-macro.md" >}})

...

それにしても皆さんちゃんとプラグインのテストしててすごいですね。Advent Calendarでもテストに関する記事ありましたし、私もテストを書いていこうと思いましたまる
