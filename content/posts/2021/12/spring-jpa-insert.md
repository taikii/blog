---
title: "Spring Data JPAのsave時にSELECTしないようにする"
date: 2021-12-04T23:07:00+09:00
tags: ["Spring"]
---

Spring Data JPAでは `save` する際にレコード存在チェックをしに行くので、これを抑制して `INSERT` だけするようにしたい。

同じことを解説している良質な記事は他にたくさんありますので、そっち見てください…。

<!--more-->

公式で解説してあります。

> Persistable の実装: エンティティが Persistable を実装している場合、Spring Data JPA は新しい検出をエンティティの isNew(…) メソッドに委譲します。

https://spring.pleiades.io/spring-data/jpa/docs/current/reference/html/#jpa.entity-persistence.saving-entites.strategies

ということなので、該当のエンティティで `Persistable` を `implements` して、 `isNew()` メソッドを実装して `true` を返すようにすれば、 `save` の際に `SELECT` せずに `INSERT` をするようになります。逆に `false` を返すと `SELECT` してから `INSERT` または `UPDATE` をするデフォルトの状態になります。強制的に `UPDATE` することはできないようです。

そんな面倒なことしなくてもJPQLでINSERTしちゃえばいいんじゃね？とちょっと思ったんですけど、JPQLにはINSERTステートメントがないですね。そりゃそうですね、中身JPAですもんね。

ただ、エンティティ毎にINSERTだけしたい！というのが決まればいいんですが、同じエンティティでも「INSERTだけしたい」時と「INSERT/UPDATEしたい」時がある場合、それぞれでクラスを作るのはアホくさい。そうなると切り替え用のフィールドをもたせて、一つのクラスでシーンによって切り替えられるようにしたい。しかし、ベタでフィールドを持たせると、Lombokの `@AllArgsConstructor()` の対象になってしまう。キショイ。

なので、以下のような抽象クラス作ってそれを継承させたほうがいいかなって思います。同様のエンティティで使い回せるし。

※エンティティクラスで `getId()` メソッドを実装する必要があります。



```java
import lombok.Setter;

public abstract class BaseEntity<T> implements Persistable<T> {
	
	@Setter
	private boolean newrec;

	@Override
	public boolean isNew() {
		return newrec;
	}
}
```

以下がそれを継承するエンティティクラス。

```java
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

@NoArgsConstructor
@AllArgsConstructor()
@Entity
@Table(name = "samples")
public class Sample extends BaseEntity<String> {

	@Id
	@Column(name = "sample_id")
	@Getter
	@Setter
	private String sampleId;

	@Column(name = "name")
	@Getter
	@Setter
	private String name;

	@Override
	public String getId() {
		return sampleId;
	}
}
```
