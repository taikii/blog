---
title: "Thymeleafの sec:authorize 内で変数にアクセスする"
date: 2020-02-17T00:06:38+09:00
tags: ["Spring Boot", "Spring Security", "Thymeleaf"]
series: []
---

ThymeleafでSpring Securityの機能を使用できる thymeleaf-extras-springsecurity を使っていて、`sec:authorize` 内で変数の参照に手間取ったのでメモ。

<!--more-->

READMEにめっちゃ書いてありました。`#vars` オブジェクト（[コンテキスト変数オブジェクト](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf_ja.html#%E5%BC%8F%E5%9F%BA%E6%9C%AC%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88)）を経由することで、変数にアクセスできます。

> As with normal Spring EL expressions, Thymeleaf allows you to access a series of objects from them including the context variables map (the `#vars` object). In fact, you are allowed to surround your access expression with `${...}` if it makes you feel more comfortable:
> 
> ```html
> <div sec:authorize="${hasRole(#vars.expectedRole)}">
>   This will only be displayed if authenticated user has a role computed by the controller.
> </div>
> ```

[thymeleaf-extras-springsecurity](https://github.com/thymeleaf/thymeleaf-extras-springsecurity/blob/3.0-master/README.markdown#using-the-attributes)

なるほど。
