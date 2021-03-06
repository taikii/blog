---
title: "Redmine: タグと同名のWikiページへのリンクを張るプラグインを作った話"
date: 2019-05-04T00:37:00+09:00
tags: ["Redmine"]
---

Redmineのチケット画面から登録されたタグと同名のWikiページへのリンクを張る [Related Wikiプラグイン](https://github.com/taikii/redmine_related_wiki) というものを作りました。

<!--more-->

## 何ができるの？

* チケット詳細ページのサイドバーに、タグと同名のWikiページへのリンクが表示されます。（存在しないWikiページであってもリンクが表示されます）
* チケット一覧ページのサイドバーに、一覧表示されているチケットのタグと同名のWikiページへのリンクが表示されます。（存在しないWikiページは表示されません）

それだけ。

![](https://github.com/taikii/redmine_related_wiki/blob/master/doc/images/screenshot.gif?raw=true)

## 開発の背景

これはずっと欲しいと思っていた機能でした。

Redmine本体にはタグ/ラベルに該当する機能はありませんが、かの有名な [Redmine Tagsプラグイン](https://github.com/ixti/redmine_tags) を導入することでタグ機能を使うことができるようになります。しかし長く運用しているとタグも増えてきますので、そうするとそのタグに関する説明が欲しくなってきます。このため、私はタグと同名のWikiページを作ってそこに説明を書いているのですが、説明を見るためにはわざわざタグの文字列をコピーしてWikiページを検索する必要があって非常にめんどくさい。頻繁にWikiページを閲覧するタグについては、チケットにタグを登録すると同時に説明欄にWikiページへのリンクも書いたりしているのですが、これもめんどくさい。

このような理由で、チケット画面のタグから自動的に同名のWikiページへのリンクを張る機能が欲しいとずっと思っていたので、この連休でチャカチャカ作っちゃいました。

### なんでサイドバー？

すでにタグ自体がリンクになっているので（同じタグが登録されているチケットの一覧へのリンク）、これはいじりたくないので、別途サイドバーに設けるようにしました。

### なんでチケット一覧は存在するページのみなの？

チケットの一覧画面ではタグの種類が多くなってしまう可能性があるので、同名のWikiページが存在するもののみに制限するようにしています。

逆に、チケット作ると同時に新しいタグを作ってその手でWikiページも作る、という一連流れができるように、詳細画面はWikiページが存在しなくてもリンクを表示するようにしています。

### タグ名に `:` を含む場合の注意点

また、プロジェクトを跨いだWikiリンクは `[[foo:Bar]]` （この場合はfooプロジェクトのBarというWikiページへのリンク）のように記述する関係上、タグ名に `:` を含むと取り回しがしづらかったので、タグ名の `:` より前の文字列は無視するようにしています。

これを逆手にとって、 `:` をつかってタグをカテゴライズすることができます。

### Wiki Listsと組み合わせてもっと便利に

さらに、[Wiki Listsプラグイン](https://github.com/tkusukawa/redmine_wiki_lists) でWikiページに該当タグが登録されたチケットの一覧を表示しておくことで、チケットとタグとWikiをシームレスに行き来することができるようになります。

**これは…まるで…Scrapboxのような使い心地…！！！**

ちなみに、Wiki Listsプラグインでタグを条件にフィルタするには以下のように書きます。

```text
ref_issues(-f:tags = タグ名)
``` 

## 技術的な話

結構めんどくさい感じになるかと思ってたんですが、案外サックリできてしまいました。あまり書くこともないのでコード見てください。。。

## 今後の展開

* サイドバーは存在するWikiページと存在しないWikiページのリンクのスタイルが一緒なので見分けつかない。わかるようにしたい。（デフォルトテーマの話）
* タグ以外にも、説明やコメント欄に書かれたWikiページリンクや、カスタムフィールドなんかも関連Wikiとして抽出したい。
* 存在しないWikiページを関連Wikiに表示するかどうかはオプションで指定できるといいかも。
