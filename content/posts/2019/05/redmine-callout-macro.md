---
title: "Redmine Callout Macroプラグイン"
date: 2019-05-01T09:07:06+09:00
tags: ["Redmine"]
---

Redmine Callout Macroプラグイン作りました。[GitBookではHintsとかCallout](https://docs.gitbook.com/content-editing/rich-content#hints-and-callouts)、[VuePressではCustom Containers](https://vuepress.vuejs.org/guide/markdown.html#custom-containers)とか呼ばれてるやつです。

<!--more-->

以下のように使います。他と同じですね。

```text
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

![](https://github.com/taikii/redmine_callout_macro/blob/master/doc/images/sample.png?raw=true)

第一引数は省略を省略できます。ブロック内にはWikiフォーマットで記述できます。

```text
{{info
This is information block.

| key | value |
| --- | ----- |
| a   | Hoge  |
| b   | Fuga  |

- Foo
- Bar
}}
```

![](https://i.gyazo.com/ee58c63b10a08c661e16518d876a976a.png)

子供が寝てる間に突貫で作ったので、色やマージンはかなりテキトーです。

お気づきの点があったら指摘いただけるとありがたいです。
