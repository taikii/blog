---
title: "RedmineのWiki REST APIでハマった話"
date: 2019-03-20T00:09:36+09:00
tags: ["Redmine"]
series: []
---

RedmineのWiki APIでちょっとハマったのでメモ。

<!--more-->

まず、Wiki APIの仕様についてはこちら。

http://www.redmine.org/projects/redmine/wiki/Rest_WikiPages

`http://localhost:3000/projects/sandbox/wiki/Child.json` とかするとJSON形式のデータをとれます。こんなカンジ。

```json
{
  "wiki_page": {
    "title": "Child",
    "parent": {
      "title": "Parent"
    },
    "text": "# Child",
    "version": 1,
    "author": {
      "id": 1,
      "name": "Redmine Admin"
    },
    "comments": "",
    "created_on": "2019-03-19T15:22:29Z",
    "updated_on": "2019-03-19T15:22:29Z"
  }
}
```

で、REST APIで親ページを `Parent` から `AnotherParent` に変えようと、以下のようなことをしてたんですが一向に変わりませんでした。

```text
$ curl -s http://localhost:3000/projects/sandbox/wiki/Child.json \
  -H "X-Redmine-API-Key: $REDMINE_KEY" \
  -H "Content-Type: application/json" \
  -X PUT \
  -d '{"wiki_page":{"parent":{"title":"AnotherParent"},"text":"# Child"}}'
```

調べてたら [Patch #14829: Patch for setting parent page via REST API - Redmine](https://www.redmine.org/issues/14829) というIssueを発見。親ページは `parent_title` というパラメータで取得してるっぽい。

そこで `"parent":{"title":"AnotherParent"}` を `"parent_title":"AnotherParent"` にしたところうまく更新できました。よかったよかった。

```text
$ curl -s http://localhost:3000/projects/sandbox/wiki/Child.json \
  -H "X-Redmine-API-Key: $REDMINE_KEY" \
  -H "Content-Type: application/json" \
  -X PUT \
  -d '{"wiki_page":{"parent_title":"AnotherParent","text":"# Child"}}'
```

公式マニュアルがもう少し整備されているといいなーなんて思ったり。

あと、ツイートした内容ちょっと違ってました。すみません。
