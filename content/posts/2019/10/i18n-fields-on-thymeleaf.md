---
title: "Thymeleafで言語別フィールドをカッコよく表示する"
date: 2019-10-17T23:15:00+09:00
tags: ["TIL", "Thymeleaf"]
series: []
---

日本語名カラムと英語名カラムを持つテーブルってよくあるじゃないですか。…よくありますよね？まぁよくあるとして、それを画面表示する際にユーザのロケールによって切り替えたい。カッコよく。そんな試行錯誤のメモです。

<!--more-->

## TL;DR

```html
<p th:text="${item[#messages.msg('i18nfields.name')]}"></p>
```

## やっていき

Spring Boot 2.1.x です。

こういうクラスがあったとします。名前フィールドが`nameEn`と`nameJa`に別れてます。これをうまいことユーザのロケールによって切り替えて画面に表示させたいわけです。

ちなみに、私は「そもそも」と言われるとしんでしまいます。

```java
@Entity
public class Item {
  @Id
  @Getter
  @Setter
  private Integer id;

  @Getter
  @Setter
  private String nameEn;

  @Getter
  @Setter
  private String nameJa;
}
```

普通に考えると、条件分岐でしょうか。

```html
<p th:text="${#locale.language == 'ja'} ? ${item.nameJa} : ${item.nameEn}"></p>
```

でも、私はif文を書くとしんでしまう体質だし、三項演算子書いたら三項演算子警察にころされるじゃないですか。

ていうか、こんな事やってて別のロケールフィールドが増えたらどうすんじゃいって感じだし、固定のフィールドと言えどもResourceBundleであるところのメッセージプロパティと同じように扱いたい。

で、ちょっと思ったんですけど、Thymeleafってオブジェクトのフィールドを`${item.nameEn}`のようにドットを使った記法で取得する他に`${item['nameEn']}`のように角括弧を使って取得することもできるんですよね。角括弧の引数は文字列なので、これを変数で渡してやれば取得するフィールドを動的に切り替えることができるはずです。

余談ですが、対象のオブジェクトがMapの場合も`${map['key1']}`でも`${map.key1}`でも取れますね。Getterでのアクセスなのか`Map#get()`なのかView側では意識することなく扱えるってことですねー。Rubyっぽいっていうか。よく知りませんが、最近のテンプレートエンジンはみんなこんな感じなんですかね？

https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf_ja.html#%E5%A4%89%E6%95%B0

結局やりたいのは固定フィールドもメッセージプロパティと同じように取り回ししたいということなので、じゃあ角括弧の引数はメッセージプロパティから取ればいいじゃんと思いました。

**messages_en.properties**

```ini
i18nfields.name=nameEn
```

**messages_ja.properties**

```ini
i18nfields.name=nameJa
```

通常Thymeleafでメッセージを表示する場合は`#{i18nfields.name}`のような構文で記述しますが、`#messages`オブジェクトを使って`${#messages.msg('i18nfields.name')}`のように記述することも可能です。角括弧の引数に直接メッセージを渡す場合は`#messages`オブジェクトを使う必要があります。

```html
<p th:text="${item[#messages.msg('i18nfields.name')]}"></p>
```

`#{i18nfields.name}`構文で書きたければ、一旦変数に入れてから渡します。

```html
<p th:with='x=#{i18nfields.name}' th:text="${item[x]}"></p>
```

また余談ですが、ドットを使った記法だと、`${item?.nameJa}`のようにオブジェクトの後ろに`?`を付けることで、対象オブジェクトがNullでない場合のみGetterを呼び出す…というかメソッドチェーンを実行することが可能ですが（Null安全？）、角括弧を使った記法だとこれはできません。オブジェクトがNullになる可能性がある場合は愚直にNullチェックする必要があります。

...

これがスマートかって言われるとそうではない気がしますし、みんなどうやってるのか知りませんが、多少は格好がついたのではないかと思います。カッコなだけに。
