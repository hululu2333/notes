### spark算子的一些案例

***

#### groupBy

> dataset的数据结构在抽象上可以看作是一张二维表，对dataset进行groupBy操作的结果和对mysql表进行groupBy的结果一样

``` java
finance_analysis.groupBy("ssoId").agg(first(col("userName")).alias("userName"),first(col("userType")).alias("userType"));
```

> finance_analysis是一个dataset，以ssoId这个字段分组，返回一个RelationalGroupedDataset类型。
>
> 这时调用agg()，对分组后的结果进行聚合计算，就能返回一个dataset。这行代码等同于sql中的select ssoId, first(userName) userName, first(userType) userType from *** group by ssoId
>
> **使用first() col()这些方法需要import static org.apache.spark.sql.functions.*;**



#### filter

``` java
//过滤掉page为null和herf为本地的记录
Dataset data = dataOrig.filter((FilterFunction<Row>) row -> !(row.get(0)==null))
                .filter((FilterFunction<Row>) row -> !String.valueOf(row.get(13)).contains("8080"));
```



#### map和flatMap

> map算子是输入一个，输出一个。flatMap算子是输入一个，输出0或多个。
>
> 直接用lambda比较难理解，建议先实现匿名内部类，理解好逻辑后再转换为lambda表达式

