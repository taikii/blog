---
title: "Markdownでリンクを定義して使いまわす"
date: 2018-02-15T11:16:56+09:00
tags: ["Markdown"]
series: []
---
先日、たまたま[JeninsのREADME.md](https://github.com/jenkinsci/jenkins/blob/master/README.md)のRAWデータを見ていたら、ロゴの部分が`[![][ButlerImage]][website]`と書かれていました。

<!--more-->

初めてこの書き方を見たのでちょっとびっくりしたのですが、これはURLにラベルをつけて使い回しする方法のようでした。CommonMarkにも[Link reference definitions](http://spec.commonmark.org/0.27/#link-reference-definition)として記述があります。

```markdown
[taikii-blog]: https://taikii.github.io

[taikii-blog] や taikii-blog と書くとリンクになる。
```

[taikii-blog] や taikii-blog と書くとリンクになる。


少し調べたところ、このラベル定義は実装によって癖があるようです。

* GitHub、Qiita: ラベル定義はドキュメント上のどこでもOK
* GitLab、Redmine: ラベル定義は参照箇所より上に書く必要がある


覚えておくと便利・・・いや、別に覚えなくていい気がします。使い所が謎。