---
title: "thymeleaf-extras-springsecurityでURLへのアクセス権の有無によってリンクとラベルを切り替える"
date: 2020-08-02T00:17:49+09:00
tags: ["Spring Boot", "Spring Security", "Thymeleaf"]
updated: 2020-08-30T16:18:56+09:00
---

[thymeleaf-extras-springsecurity](https://github.com/thymeleaf/thymeleaf-extras-springsecurity) を使うと Thymeleaf で Spring Security のオブジェクトにアクセスできるようになります。

thymeleaf-extras-springsecurity には、指定のURLにアクセス可能である場合に要素を出力する `sec:authorize-url="url"` という属性があり、結構便利に使えます。この属性はアクセス権がない場合の処理には使えないので「アクセス権の有無によってリンクとラベルを切り替えたい」みたいなニーズは、この属性だけでは実現できません（と思います）。それをスマートに書けないか考えてみました。

<!--more-->

以下の例では `/admin` にアクセス権がある場合に `<a>` タグが出力され、アクセス権がない場合は何も出力されません。

```html
<a th:href="@{/admin}" sec:authorize-url="/admin">Admin</a>
```

Spring Security の認可系の処理は `#authorization` オブジェクトでアクセスできます。`sec:authorize-url="/admin"` は、このオブジェクトを使って `th:if="${#authorization.url('/admin')}"` と記述するのと同義です。これを使って、以下の例ではアクセス権がない場合はラベルで表示しています。

```html
<span sec:authorize-url="/admin"><a th:href="@{/admin}">Admin</a></span>
<span th:unless="${#authorization.url('/admin')}">Admin</span>
```

書き方が統一されてなくてキモいですね。あと `th:unless` は個人的にあまり使いたくないし、条件文を2つ書いているのも「もにょり」ます。

個人的には `th:switch` を使うのが可読性もあり、素直な感じがしていいかなぁと思います。

```html
<th:block th:switch="${#authorization.url('/admin')}">
  <span th:case="true"><a th:href="@{/admin}">Admin</a></span>
  <span th:case="false">Admin</span>
</th:block>
```

もっとスマートな書き方があれば教えていただけると嬉しいです。

## th:remove を使うべき（2020/08/30追記）

`th:remove` に `tag` を設定することで子の要素を残したまま自身のタグを削除することができます。これを使うのがスマートですね。（`th:remove="tag"` しても `th:text="xxx"` で出力されるボディは残ります。）

```html
<a th:href="@{/admin}" th:text="|Admin|"
    th:remove="${not #authorization.url('/admin')}? tag"></a>
```

[8.4 テンプレートフラグメントの削除 - Tutorial: Using Thymeleaf (ja)](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf_ja.html#%E3%83%86%E3%83%B3%E3%83%97%E3%83%AC%E3%83%BC%E3%83%88%E3%83%95%E3%83%A9%E3%82%B0%E3%83%A1%E3%83%B3%E3%83%88%E3%81%AE%E5%89%8A%E9%99%A4)
