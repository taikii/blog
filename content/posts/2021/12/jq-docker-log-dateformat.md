---
title: "jqでDockerログをJSTに変換する"
date: 2021-12-04T23:36:00+09:00
tags: ["Docker"]
---

JSON形式のDockerログのタイムスタンプをJSTに変換したい。

<!--more-->

JSON形式で保存されたログは1行が以下のようになっています。タイムスタンプはGMTで、 `ISO 8601` フォーマットなんだけどナノ秒になっています。

```json
{
  "log": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "stream": "stdout",
  "time": "2021-12-01T15:14:22.856018519Z"
}
 ```

`jq -r '[.time, .log] | @tsv'` のようにすることで、タブ区切りで表示できますが、GMTなので見にくい。

JST（ローカルタイム）に変換するには、一度 `fromdateiso8601` 関数でUnixエポックに変換して `strflocaltime` でフォーマットしてやる必要があります。

しかし、 `fromdateiso8601` は秒までの `ISO 8601` フォーマットつまり `%Y-%m-%dT%H:%M:%SZ` を期待しているので、ナノ秒が邪魔。

なので、 `sub("\\..*Z"; "Z")` でナノ秒以下を削ってやる。

```
$ cat hogehoge.log | jq -r '[(.time | sub("\\..*Z"; "Z") | fromdateiso8601 | strflocaltime("%Y-%m-%dT%H:%M:%S") ), .log] | @tsv'
```

https://stedolan.github.io/jq/manual/#Dates

生のログを見ることってそうそうない気もしますが、私はJSONログを日次で退避しているので、ちょこちょこ参照する機会があります。たぶんjournaldに出すようにしたほうがいいんでしょうけど…。
