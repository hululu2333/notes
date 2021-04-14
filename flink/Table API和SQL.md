### Table API 和 SQL

***

> 



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



#### 