### RDD简介

***

> RDD的全称是弹性分布式数据集（Resilient Distributed DataSet），是Spark中最基本的数据（计算）抽象。在代码中是一个抽象类，表示一个不可变、可分区、里面的元素可并行计算的集合
>
> 分布式表示它的数据源可以是分布式的，来自不同的计算机
>
> 说RDD是最基本的数据抽象不如说是计算抽象，因为在调用collect方法前，RDD中没有数据，只有计算逻辑 
>
> 不可变表示RDD子类对象中的数据是不能修改的，你想改变的话顶多是创建一个新的RDD子类对象。类似于val
>
> 分区是为了后期的并行计算，一个分区的数据交给一个executor来执行



#### 什么是RDD

> 弹性分布式数据集这个名词太过抽象，可以从我们熟知的java IO流来入手

##### 基于装饰者模式的IO流

> 当我们需要从文件系统中读取一行时，我们需要这样做
>
> BufferedReader br = new BufferedReader(new InputStreamReader(new (FileInputStream("D://1.txt"))));
>
> br.readLine();
>
> 真正从文件中读取数据的还是FileInputStream这个节点流，其他的包装流只是为这个节点流加一些功能
>
> 只有在br对象运行readLine()时，节点流才会去读取数据。在读取数据之前，所有的包装引起的只是结构的变化

![image-20201011142115288](F:\学习笔记\spark\images\javaIO图.png)

***

#### 基于装饰者模式的RDD

> 我们先看一串执行wordCount的spark代码

``` scala
sc.textFile("../input").flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_).collect
```

> sc对象调用textFile方法返回一个RDD子类的对象，我们把这个对象叫做lines
>
> lines对象调用flatMap方法时，有大概这样一行代码new MapPartitionsRDD[U, T](this, (context, pid, iter)）也就是说，把lines当作输入值，输出一个新的RDD子类对象。对原来的RDD子类对象做一些增强
>
> 和IO一样，在RDD子类对象没有调用collect方法时，第一个RDD子类对象lines并不会去文件中读取一行行的值。
>
> IO流在读数据之前利用装饰者模式进行了结构的封装。但RDD在读取数据之前，感觉封装与没封装没什么区别，其实RDD是对逻辑进行了封装，flatMap(_.split(""))中的\_.split("")就是逻辑
>
> 第一个RDD子类对象lines读取文件，获得一行行的数据。第二个RDD子类对象对lines进行包装，将一行行数据变成一个个单词

![image-20201011143214976](F:\学习笔记\spark\images\基于装饰着模式的RDD.png)

