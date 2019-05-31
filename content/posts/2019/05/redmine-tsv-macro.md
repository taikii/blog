---
title: "Redmine: タブ区切りのテキストをテーブルで表示するMacroを作った"
date: 2019-05-31T01:21:52+09:00
tags: ["Redmine"]
---

タブ区切り（TSV）のテキストをテーブルで表示するプラグインを作りました。

<!--more-->

[Redmine TSV Macro Plugin](https://github.com/taikii/redmine_tsv_macro)（リンク貼り忘れてた）

この機能は[元々Crowiに存在](https://medium.com/crowi-book/crowi-v1-5-0-5a62e7c6be90)していて、かなり感動したのを記憶しています。その~~パクリ~~移植です。

```
{{tsv
Value1  Value2  Value3
}}
```

<table>
  <tbody>
    <tr><td>Value1</td><td>Value2</td><td>Value3</td></tr>
  </tbody>
</table>

`tsv_h`で1行目がヘッダとして扱われます。

```
{{tsv_h
Head1 Head2 Head3
Value1  Value2  Value3
}}
```

<table>
  <thead>
    <tr><th>Head1</th><th>Head2</th><th>Head3</th></tr>
  </thead>
  <tbody>
    <tr><td>Value1</td><td>Value2</td><td>Value3</td></tr>
  </tbody>
</table>

右寄せとかセル内改行とか、難しいことをやりたければWikiフォーマットで書けばいいと思います。
