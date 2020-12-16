### spark的入口

***

> spark 有三大引擎，spark core、sparkSQL、sparkStreaming
>
> spark core 的关键抽象是：SparkContext、RDD
>
> SparkSQL 的关键抽象是 ：SparkSession、DataFrame
>
> sparkStreaming 的关键抽象是 ：StreamingContext、DStream



 ####  SparkSession

> SparkSession 是 spark2.x 引入的概念，主要用在 sparkSQL 中，当然也可以用在其他场合，**他可以代替 SparkContext**
>
> SparkSession作用是为用户提供统一的切入点
>
> 在 spark1.x 中，SparkContext 是 spark 的主要切入点，由于 RDD 作为主要的 API，我们通过 SparkContext 来创建和操作 RDD，*但是在不同的应用中，需要使用不同的 context，*在 Streaming 中需要使用 StreamingContext，在 sql 中需要使用 sqlContext，在 hive 中需要使用 hiveContext，比较麻烦
>
> SparkSession 实际上封装了 SparkContext，另外也封装了 SparkConf、sqlContext，随着版本增加，可能更多。所以我们在写不同类型的spark应用时，只需要通过SparkSesion即可



#### DataFrame

> dataframe 是 spark2.x 中新增的数据格式，类似于加强版的RDD。SparkSession无论读取什么类型的数据，他输出的都是dataframe类型
>
> 而SparkContext无论读什么类型，输出的都是RDD类型



SparkSession对象一般称为spark，SparkContext对象一般称为sc