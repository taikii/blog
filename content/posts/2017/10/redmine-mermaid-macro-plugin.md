---
title: "Redmine Mermaid Macro Plugin"
date: 2017-10-15T22:56:54+09:00
tags: ["Redmine"]
---
Redmine上で`mermaid.js`を実行するプラグインを作りました。
<!--more-->

[mermaid](https://mermaidjs.github.io/) はテキストベースで作図するライブラリです。簡単に作図できますし、生成される図もスタイリッシュなので気に入って使っています。

RedmineではPlantUMLで作図できるRedmineプラグインは元々あってそれも使っていたのですが、UMLとは言えないちょっとした図を書こうとした時に仰々しい感じになってしまうのと、見た目がいかにも「URLです！！」という感じなので、mermaidを使いたいなぁと思っていました。

そんなとき、 [Redmine and mermaid.js integration](https://serol.ro/posts/2016/redmine_mermaid_js/) という記事を見つけて、プラグイン化できそうじゃね？と思い、作りました。

（このプラグインを作るにあたり、一部コードやメッセージはSerolさんに許可をいただいてそのまま使用させていただきました。）

## 使用するmermaid.jsについて
デフォルトではCDNのものを使用していますが、 _Redmine &gt; Plugin_ ページから変更ができます。

Redmineに添付ファルとしてアップロードしても使えます。プレビューページｎDownloadリンクのURLをコピーしてプラグイン設定ページに貼り付けてください。以下のようなURLのはずです。（プレビューページのURLではないので注意）

```
http://localhost:3000/attachments/download/1/mermaid.min.js
```

[taikii/redmine_mermaid_macro: Mermaid.js for Redmine](https://github.com/taikii/redmine_mermaid_macro)
