---
title: Jackson によるシリアライズで循環参照を止める
category: jackson
tags:
- jackson
- jpa
- json
- kotlin
---

Jackson を使用した JSON の出力で、オブジェクトが循環参照を起こしているような場合、
これを止める方法についてのメモ。

以下のように、`JsonIdentityInfo` アノテーションをクラスにつけるだけで良いです。
これを使用することで、循環が検出された場合に、循環しているオブジェクトの内容を、
ID 文字列に置き換えてくれるようになります。

JPA エンティティなどをそのまま JSON 出力したいような場合には、
`OneToMany`、`ManyToOne` を使用したオブジェクトの参照で、
このようなニーズが度々発生します。

```kotlin
@Entity
@JsonIdentityInfo(property = "_id", generator = ObjectIdGenerators.UUIDGenerator::class)
data class Person(
        @Id
        var id: String = "",
        var name: String = "",
        @OneToMany(targetEntity = Schedule::class, mappedBy = "person")
        var schedules: MutableList<Schedule> = mutableListOf()
)

@Entity
@JsonIdentityInfo(property = "_id", generator = ObjectIdGenerators.UUIDGenerator::class)
data class Schedule(
        @Id
        var id: String = "",
        @ManyToOne(targetEntity = Person::class)
        var helper: Person = Person(),
        var title: String = "",
        var start: LocalDateTime = LocalDateTime.MIN,
        var end: LocalDateTime = LocalDateTime.MIN
)
```

例では、`UUIDGenerator` を使用しているので、生成される ID は UUID になります。
生成器には他にも種類があって、以下の中から選択できます。

- `UUIDGenerator`
- `IntSequenceGenerator`
- `StringIdGenerator`
- `PropertyGenerator`

また、循環参照を止めるためのアノテーションは `JsonIdentityInfo` 以外にもあり、
以下を選択することができます。

- `JsonIdentityInfo`
- `JsonBackReference`
- `JsonManagedReference`

参考: https://qiita.com/sengoku/items/56bc6319759fee6d15e3
