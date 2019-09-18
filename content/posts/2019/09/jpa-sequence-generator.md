---
title: "JPAでシーケンスを使ってIDを自動採番する場合の注意点"
date: 2019-09-18T23:40:30+09:00
tags: ["Java", "JPA", "Oracle"]
series: []
---

[Oracle シーケンスのデフォルトはNOORDERである罠]({{< relref "./oracle-sequence-noorder.md" >}}) の続きです。

<!--more-->

JPAでEntityのIDを自動採番するとき、DBMSがOracleの場合はシーケンスを使うことになります。シーケンスを使う場合は、`@GeneratedValue`の`strategy`プロパティに`GenerationType.SEQUENCE`を指定します。たぶん`GenerationType.AUTO`でもシーケンスになります。（`GenerationType.AUTO` を指定するとはDBMSによって適切な採番方法が選択されます）

```java
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE)
@Column(name="user_id")
private String userId;
```

`SequenceGenerator`は、デフォルトではシステムでひとつのシーケンス（名前は`hibernate_sequence`）を共有で使用します。該当テーブル専用のシーケンスを使う場合は`@SequenceGenerator`の`sequenceName`プロパティで指定することができます。以下の例では`user_id_seq`がシーケンスオブジェクト名です。

```java
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "user_id_generator")
@SequenceGenerator(name = "user_id_generator", sequenceName = "user_id_seq")
@Column(name="user_id")
private String userId;
```

`SequenceGenerator`は採番都度シーケンスにアクセスして採番するのではなく、あらかじめ一定の採番枠を確保し、その採番枠内の増分であればDBアクセスを発生させないようにしています。

この採番枠は`@SequenceGenerator`の`allocationSize`というプロパティで指定でき、デフォルトは`50`となっています。

また、`user_id_seq.nextval`した時に`allocationSize`分増加している必要があるので、`allocationSize`とシーケンスの`increment by`は一致している必要があります。（事実、Spring Bootにおいて`spring.jpa.hibernate.ddl-auto=update`としてシーケンスオブジェクトを自動作成すると`increment by 50`のシーケンスが作成されます。）

しかし、これには一つ問題があり、同じテーブルのIDを採番する`SequenceGenerator`インスタンスが複数存在すると（つまり複数のアプリケーションサーバが存在するLB構成だと）、インスタンスAは`1～50`、インスタンスBは`51～100`の枠内でそれぞれ採番することになるので、テーブルのIDは`1, 51, 2, 52...`のように番号の戻りが発生してしまいます。これは [Oracle シーケンスのデフォルトはNOORDERである罠]({{< relref "./oracle-sequence-noorder.md" >}}) と同じ問題です。

このため、IDに対して順序を期待する場合は`allocationSize = 1`を指定して都度SEQUENCEから採番させる必要があります。

```java
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "user_id_generator")
@SequenceGenerator(name = "user_id_generator", sequenceName = "user_id_seq", allocationSize = 1)
@Column(name="user_id")
private String userId;
```

そんな感じです。
