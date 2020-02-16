---
title: "Spring Bootのかっこいいページネーションを考える"
date: 2020-02-16T23:08:00+09:00
tags: ["Spring Boot", "Thymeleaf"]
series: []
---

Spring Bootのページネーションを考えてみます。

コントローラ側の処理は省略しますが、簡単に言うとメソッドの仮引数 `Pageable` を定義するだけです。デフォルト値はこの仮引数にアノテーション `@PageableDefault` で指定します。

<!--more-->

## 簡単に

ネットを見ると、大体 `${#numbers.sequence(0, page.totalPages - 1)}` で作るように書いてあります。こんな感じ。ちなみにスタイルはBootstrapを使ってます。（ここで書かれている `page` 変数は、JPAで取得できる `org.springframework.data.domain.Page` インタフェースです。）

```html
<nav aria-label="Page navigation">
  <ul class="pagination">
    <th:block th:each="i : ${ #numbers.sequence(0, page.totalPages - 1) }">
      <li th:if="${i ge 0}" class="page-item" th:classappend="${i eq page.number} ? 'active'">
        <a th:href="@{''(page=${i})}" th:text="${i + 1}" class="page-link"></a>
      </li>
    </th:block>
  </ul>
</nav>
```

しかしこのままだと、ページが増えるとダサいことになります。

![](https://i.gyazo.com/fbaf56953cfd8e70c53b131026118b6e.png)

## かっこよくする

よくある感じで、現在のページに近いページだけ表示したいわけですが、ネットの先輩方を尋ねるとPageインターフェースのラッパークラスを作ればいいと仰っている方もおり、「まじかよ…」という気持ちです。

身近なページネーションがあるシステムがRedmineだったんで確認してみると、「Prev」「1ページ目」「最後のページ番号」「Next」が固定で表示され、その間に現在のページと±2ページが表示されるようです。このUIはわかりやすくて好きです。

![](https://i.gyazo.com/263a8199ce6b2908c3b2f4c923770a31.png)

同じことをThymeleafだけで実装してみたいと思います。

。。。

はい。

{{< gist taikii bc485eba97fd31f1728a010117e3a6b2 >}}

少し解説します。

* `page.number` に現在のページ番号（0スタート）が入るので、 `#numbers.sequence(page.number - 2, page.number + 2)` で現在のページ±2ページ分をループする。
* そのままだと、マイナスになる可能性があるので（また、最大ページを超える可能性があるので）、`th:if="${i ge 0 and i lt page.totalPages}"` を条件に出力する。
* 4ページ目以降（`page.number gt 2`）は固定で1ページ目のリンクを表示。
* 5ページ目以降（`page.number gt 3`）は固定で1ページ目の後ろに「…」を表示。
* 同じことを最後のページにも。

Viewに条件分岐を持ち込むべきじゃないとかあると思いますが、私はこの程度はViewの範疇かなと思います。これは見せ方の問題でもありますし。

## その他

### 1ページあたりの件数を制御する

1ページあたりの件数は `size` パラメータを渡せば制御できます。リクエストの度にデフォルト値になってしまうため、ページ遷移のパラメータに追加してあげる必要があります。

```html
<a th:href="@{''(page=${i}, size=${page.size})}" th:text="${i + 1}" class="page-link"></a>
```

### 検索条件を維持する

検索画面でのページネーションの場合は、検索条件もページ遷移のパラメータとして渡してやる必要があります。下記の例では `q` が検索条件です。

```html
<a th:href="@{''(page=${i}, size=${page.size}, q=${q})}" th:text="${i + 1}" class="page-link"></a>
```

### ソートを維持する

ソータブルな場合も、同じようにソートを `sort` パラメータに追加する必要がありますが、指定方法に少し癖があります。

> Properties that should be sorted by in the format `property,property(,ASC|DESC)`. Default sort direction is ascending. Use multiple `sort` parameters if you want to switch directions — for example, `?sort=firstname&sort=lastname,asc`.

[Spring Data JDBC - Reference Documentation](https://docs.spring.io/spring-data/jdbc/docs/1.1.4.RELEASE/reference/html/#core.web.basic.paging-and-sorting)

`size` と同様に `${page.sort}` をそのまま渡したいところですが、この `org.springframework.data.domain.Sort` （と `Sort.Order`）は `toString` をオーバーライドしており `フィールド名: ダイレクション` という形式で出力されてしまいます。これは `Pageable` が期待するフォーマットではありません。このため `Pageable` が期待するフォーマットにシコシコ変換してやる必要があります（私が知る限り）。

* [Sort#toString](https://github.com/spring-projects/spring-data-commons/blob/master/src/main/java/org/springframework/data/domain/Sort.java#L263)
* [Sort.Order#toString](https://github.com/spring-projects/spring-data-commons/blob/master/src/main/java/org/springframework/data/domain/Sort.java#L641)
