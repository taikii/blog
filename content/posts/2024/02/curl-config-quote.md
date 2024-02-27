---
title: "curlのconfigではダブルクォートを使う"
date: 2024-02-27T21:41:36+09:00
tags: ["curl"]
---

curl でコンフィグファイルを指定するオプション `-K / --config` で指定するファイルではダブルクォートで記述する必要があります。シングルクォートだと機能しません。manにも書いてありました。いやーハマったハマった。

また `-K` に `-` を渡すことで標準入力から取ることができます。

https://curl.se/docs/manpage.html#-K

```sh
$ curl -K- https://example.com/ <<< "-H \"Authorization: Bearer ${_token}\""
$ curl -K- https://example.com/ <<< "Header = \"Authorization: Bearer ${_token}\""
```

参考

* [そんな curl で大丈夫か？認証情報を非対話で安全に入力する方法 #Linux - Qiita](https://qiita.com/ktooi/items/958bab82b828b389969a)
* [http - how do I set a custom header in a curl config file - Stack Overflow](https://stackoverflow.com/questions/27127195/how-do-i-set-a-custom-header-in-a-curl-config-file)
