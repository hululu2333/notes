### Table API 和 SQL

***

> Apache Flink 有两种关系型 API 来做流批统一处理：Table API 和 SQL。Table API 是用于 Scala 和 Java 语言的查询API，它可以用一种非常直观的方式来组合使用选取、过滤、join 等关系型算子。Flink SQL 是基于 [Apache Calcite](https://calcite.apache.org/) 来实现的标准 SQL。这两种 API 中的查询对于批（DataSet）和流（DataStream）的输入有相同的语义，也会产生同样的计算结果。



#### 简介

> **怎么区分Table API 和 SQL：**第一个语句和第三个语句通过调用以sql关键字为名的方法来实现sql查询功能，是Table API
>
> 第四个语句通过调用sqlQuery方法，直接传入一个SQL语句，这是SQL

``` java
// 在 Table API 里不经注册直接“内联”调用函数
env.from("MyTable").select(call(SubstringFunction.class, $("myField"), 5, 12));

// 注册函数
env.createTemporarySystemFunction("SubstringFunction", SubstringFunction.class);

// 在 Table API 里调用注册好的函数
env.from("MyTable").select(call("SubstringFunction", $("myField"), 5, 12));

// 在 SQL 里调用注册好的函数
env.sqlQuery("SELECT SubstringFunction(myField, 5, 12) FROM MyTable");
```



#### 计划器 planner

> Planner 的作用主要是把关系型的操作翻译成可执行的、经过优化的 Flink 任务。
>
> 目前有两种计划器，old planner和 blink planner。两种 Planner 所使用的优化规则以及运行时类都不一样。 它们在支持的功能上也有些差异。
>
> 在创建TableEnvironment时我们需要指定所使用的planner的类型，新版本推荐使用blink planner



#### Catalogs

> Catalog 提供了元数据信息，例如数据库、表、分区、视图以及数据库或其他外部系统中存储的函数和信息。
>
> 数据处理最关键的方面之一是管理元数据。 元数据可以是临时的，例如临时表、或者通过 TableEnvironment 注册的 UDF。 元数据也可以是持久化的，例如 Hive Metastore 中的元数据。Catalog 提供了一个统一的API，用于管理元数据，并使其可以从 Table API 和 SQL 查询语句中来访问。

> 我们在flink session中注册了函数、表什么的。元数据都会存在默认的catalog中，之后我们用的时候就会去catalog中找