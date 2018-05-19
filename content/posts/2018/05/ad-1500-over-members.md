---
title: "Active Directoryのグループから1,500件を超えるメンバーを取得する方法（はもう考えなくていい）"
date: 2018-05-19T10:40:48+09:00
tags: ["Active Directory","LDAP"]
series: []
---

Active Directoryにおいて属性値が1,500件を超えている場合、すべての値を一度に取得することができません（でした）。この場合の属性値の取得方法について解説します。

この記事では`ldapsearch`コマンドで検証しています。また、そこまで値が増える属性は`member`くらいしか無いと思いますので、`member`属性に絞って書いています。

<!--more-->

## TL;DR

最近のWindows Serverではこのような考慮が不要で、どんなに属性値が多くても一度に取得できるようです。やったね！

## 1,500件以上だと返却される属性名が`member`ではなくなる

まず、普通に`ldapsearch`コマンドでグループのメンバを取得してみます。

```text
$ ldapsearch -h dc.example.com -D user000000@example.com -w p@ssw0rd -x -b "dc=example,dc=com" "(cn=group1)" member
dn: cn=group1,ou=groups,dc=example,dc=com
member: cn=user000000,ou=users,dc=example,dc=com
member: cn=user000001,ou=users,dc=example,dc=com
member: cn=user000002,ou=users,dc=example,dc=com
```

3件のグループメンバを取得することができました。

しかし、該当のグループのメンバが1,500件を超えていると、以下のように返却されます。

```text
$ ldapsearch -h dc.example.com -D user000000@example.com -w p@ssw0rd -x -b "dc=example,dc=com" "(cn=group1)" member
dn: cn=group1,ou=groups,dc=example,dc=com
member;range=0-1499: cn=user000000,ou=users,dc=example,dc=com
member;range=0-1499: cn=user000001,ou=users,dc=example,dc=com
member;range=0-1499: cn=user000002,ou=users,dc=example,dc=com
.................
member;range=0-1499: cn=user001499,ou=users,dc=example,dc=com
```

返却される属性名が`member`から`member;range=0-1499`に変わっています。Active Directoryでは属性値を一度に**1,500件**までしか取得することができないため、範囲を絞って取得されたことを属性名を使って示しているのです。

属性名自体が変わっているので、Javaなどのプログラムで属性値を取得する場合は注意が必要です。

```java
SearchResult sr = (SearchResult)results.next();
Attributes attrs = si.getAttributes();
if (attrs != null) {

  // メンバ情報を取得できない
  Attribute attr1 = attrs.get("member");

  // メンバ情報を取得できる
  Attribute attr2 = attrs.get("member;range=0-1499");

}
```

さらに、1,501件目以降のメンバを取得を取得してみます。

リクエストする属性名を`member;range=1500-*`または`member;range=1500-2999`とすると、1,501件目以降の値を取得できます。TOにあたる部分を`*`を指定することは、最大件数、つまりこの場合だと`2999`を指定するのと同じ意味になります。

```text
$ ldapsearch -h dc.example.com -D user000000@example.com -w p@ssw0rd -x -b "dc=example,dc=com" "(cn=group1)" "member;range=1500-*"
dn: cn=group1,ou=groups,dc=example,dc=com
member;range=1500-2999: cn=user001500,ou=users,dc=example,dc=com
member;range=1500-2999: cn=user001501,ou=users,dc=example,dc=com
member;range=1500-2999: cn=user001502,ou=users,dc=example,dc=com
.................
member;range=1500-2999: cn=user002999,ou=users,dc=example,dc=com
```

`member;range=1500-*`として投げたのに、返却された属性名が`member;range=1500-2999`でした。属性名の範囲のTOにあたる部分が具体的な数字で返却される場合は、取得した範囲以降にまだ値が残っていることを表しています。

次に`3001`件目以降を取得します。リクエストする属性名を`member;range=3000-4499`として検索してみます。

```text
$ ldapsearch -h dc.example.com -D user000000@example.com -w p@ssw0rd -x -b "dc=example,dc=com" "(cn=group1)" "member;range=3000-4499"
dn: cn=group1,ou=groups,dc=example,dc=com
member;range=3000-*: cn=user003000,ou=users,dc=example,dc=com
member;range=3000-*: cn=user003001,ou=users,dc=example,dc=com
member;range=3000-*: cn=user003002,ou=users,dc=example,dc=com
.................
member;range=3000-*: cn=user003999,ou=users,dc=example,dc=com
```

`range=3000-*`という属性名で1000件分が返却されました。指定した範囲内に収まる件数の場合には、属性名のTOにあたる部分が`*`となっていると、それ以上属性値がないことを示しています。つまりこのグループには4000件のメンバが登録されていたことになります。

じゃあキッチリ3001件目から4000件目までを指定して取得したらどうなるでしょう？

```text
$ ldapsearch -h dc.example.com -D user000000@example.com -w p@ssw0rd -x -b "dc=example,dc=com" "(cn=group1)" "member;range=3000-3999"
dn: cn=group1,ou=groups,dc=example,dc=com
member;range=3000-*: cn=user003000,ou=users,dc=example,dc=com
member;range=3000-*: cn=user003001,ou=users,dc=example,dc=com
member;range=3000-*: cn=user003002,ou=users,dc=example,dc=com
.................
member;range=3000-*: cn=user003999,ou=users,dc=example,dc=com
```

やっぱり`member;range=1500-*:`として返却されます。それ以上値が残っていないためです。

## Windows 2000 Serverの場合は1000件まで

上記は**Windows Server 2003**の場合の話です。（もうそんなサーバは残っていないと思いますが）**Windows 2000 Server**の場合は一度に取得できる属性値の件数が`1,500`件ではなく`1,000`件となります。取得方法は同じです。

## Windows Server 2008以降

Windows Server 2008以降は、上記のような上限がなくなったように見受けられます。少なくとも`6,000`件を超える属性値を一度に取得できました。残念ながら文献を見つけることはできませんでした。ご存知の方がいらっしゃいましたら教えていただけますと幸いです。
