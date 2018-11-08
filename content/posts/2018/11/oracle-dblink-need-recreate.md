---
title: "Oracle: データベース・リンクはtnsnames.ora変更後に再作成が必要"
date: 2018-11-04T23:42:20+09:00
tags: ["Oracle"]
series: []
draft: true
---

[SSIA](https://ejje.weblio.jp/content/SSIA)って感じですが…

<!--more-->

※今回検証した環境はOracle Database 11gです。（古い）

データベース・リンク（以下DBリンク）は他のデータベースのオブジェクトを操作できる機能です。以下のようにして作成します。（参考: [CREATE DATABASE LINK](https://docs.oracle.com/cd/E16338_01/server.112/b56299/statements_5005.htm)）

```sql
CREATE DATABASE LINK remotedb CONNECT TO scott IDENTIFIED BY tiger USING 'remotedbname';
```

* `remotedb`: DBリンクオブジェクトの名前
* `scott`: 接続するリモートDBの接続ユーザ名
* `tiger`: 接続ユーザのパスワード
* `remotedbname`: 通常は`tnsnames.ora`上の接続名になると思います（接続情報をべた書きすることも可能）

あらかじめOracle Databaseサーバ上の`tnsnames.ora`には以下のように接続するリモートDBの接続情報が記載されている必要があります。

```tnsnames.ora
remotedbname =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = dbserver-1.example.com)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVICE_NAME = XE)
    )
  )
```

以下のようにオブジェクト名の後ろに`@`に続いて作成したDBリンクオブジェクトの名前`remotedb`をくっつけることで、リモートDBのオブジェクトを検索することができます。

```sql
SELECT * FROM tab1@remotedb;
```

同じようにして`UPDATE`や`DELETE`もできます。

で、このリモートDBがリプレース等で`dbserver-1.example.com`から`dbserver-2.example.com`にホスト名が変わる場合、ホスト名は`tnsnames.ora`にしか書かれていないので`tnsnames.ora`を修正すればいいのかというと、それだけでは接続先は変更されません。どうもホスト名やサービス名などの`tnsnames.ora`に書かれている接続情報は、DBリンクを作成した時点でDBリンクオブジェクトが持ってしまうようで、いくら`tnsnames.ora`を書き換えても古いリモートDBに繋がってしまいます。（文献を見つけることができませんでした。また、試してないですが、DBインスタンスを再起動したりすれば反映されるかも？）

じゃあ、[ALTER DATABASE LINK](https://docs.oracle.com/cd/E16338_01/server.112/b56299/statements_1005.htm) かとなるわけですが、これは接続ユーザのパスワードが変わった場合のみ使用され、接続ユーザや接続情報は変更することはできません。

> 接続または認証ユーザーのパスワードが変更された場合に固定ユーザー・データベース・リンクを変更するには、ALTER DATABASE LINK文を使用します。
>この文を使用して、データベース・リンクに関連付けられた接続または認証ユーザーを変更することはできません。userを変更するには、データベース・リンクを再作成する必要があります。

このことから、リモートDBのホスト名が変わる等で`tnsnames.ora`を変更した場合は、DBリンクの再作成（`DROP` & `CREATE`）が必要となると考えられます。

('A`)ﾏﾝﾄﾞｸｾ
