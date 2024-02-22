---
title: "Blueskyクライアントシェルを作った"
date: 2024-02-23T06:15:17+09:00
tags: ["Bluesky", "Bash"]
series: []
---

[Bluesky](https://bsky.app/) のリストをバルク処理したかったので、APIをコールするクライアントをBashで作りました。

<!--more-->

https://github.com/taikii/bsky-sh

最初にコマンドを実行したときにハンドルとAppパスワードを要求されます。Appパスワードは https://bsky.app/settings/app-passwords から生成してね。 `refleshJwt` は `~/.bskysession` に保存され、以降ログイン操作は必要としません。 `~/.bskysession` は秘匿にしてね。

ユーザを切り替えたいときは `./bsky.sh login` します。

機能は色々足らないですけど、タブ区切りで出てくるから `cut` コマンドで必要なフィールドだけにして流し込むとかできます。

本当はmattnさんの https://github.com/mattn/bsky にリスト系のコマンドを追加するPRだしたかったんですけど、あまりgo-langに明るくないのでできる範囲でつくっちゃった。

BlueskyはAPIもその [ドキュメント](https://docs.bsky.app/) も充実してるので、ありがたいですね。楽しい。しかも、今日 [連語を組めるようになった](https://bsky.social/about/blog/02-22-2024-open-social-web) そうで、さらには [500万人も達成](https://bsky.app/profile/jay.bsky.team/post/3klzreayifc2c) したそうなので、これからどうなっていくのか、ますます楽しみです。

```text
./bsky.sh login

./bsky.sh refresh-session

./bsky.sh profile [HANDLE]
  did handle displayName description

./bsky.sh search-user QUERY
  did handle displayName description

./bsky.sh follows [HANDLE]
  did handle displayName description

./bsky.sh followers [HANDLE]
  did handle displayName description

./bsky.sh lists [HANDLE]
    uri collection name

./bsky.sh list LIST_URI
    rkey did handle displayName description

./bsky.sh addmember LIST_URI USER_DID
USER_DIDs | ./bsky.sh addmember LIST_URI

./bsky.sh delmember LIST_URI USER_DID
USER_DIDs | ./bsky.sh delmember LIST_URI

./bsky.sh delmember-rkey LIST_MEMBER_RKEY 
LIST_MEMBER_RKEYs | ./bsky.sh delmember_rkey

./bsky.sh feed FEED_URI
  uri createdAt handle text

./bsky.sh list-feed LIST_URI
  uri createdAt handle text

./bsky.sh user-feed HANDLE
  uri createdAt handle text

./bsky.sh post TEXT
TEXT | ./bsky.sh post
```
