### spark简介

***

#### spark的定义

> spark是一种基于内存的快速、通用、可扩展的大数据分析引擎
>
> 类似于MapRuduce，但优于MapReduce



#### spark的来由

> spark是基于hadoop1.x推出的。因为hadoop1.x没有yarn，而且hadoop与mapreduce的耦合太高。除此之外还有一些问题，总之就是hadoop1.x太难用了
>
> 于是spark被开发出来代替haoop1.x，但后来hadoop出了2.x。于是现在spakr和hadoop都在使用



#### spark的特点

> **快：**和Hadoop的MapReduce相比，spark的速度要快非常多，因为spark将计算的中间结果放在内存中
>
> **易用：**spark支持Java、Python和Scala的API，也支持交互式的Python和Scala的Shell
>
> **通用：**spark提供了统一的解决方案，spark可以用来批处理，交互式查询(Spark sql)，实时流处理(Spark Stream)，机器学习和图计算。这些不同类型的处理都可以在同一个应用中无缝使用。
>
> **兼容性：**spark可以非常方便的和其他开源产品进行融合，比如spark可以让hadoop的yarn成为它的资源管理器
>
> Spark 是在 [Scala](https://baike.baidu.com/item/Scala/2462287) 语言中实现的，它将 Scala 用作其应用程序框架。与 Hadoop 不同，Spark 和 [Scala](https://baike.baidu.com/item/Scala/2462287) 能够紧密集成，其中的 [Scala](https://baike.baidu.com/item/Scala/2462287) 可以像操作本地集合对象一样轻松地操作分布式数据集。
>
> 尽管创建 Spark 是为了支持分布式数据集上的迭代作业，但是实际上它是对 Hadoop 的补充，可以在 Hadoop 文件系统中并行运行。



#### spark的内置模块

![image-20200926094420895](F:\学习笔记\spark\images\image-20200926094420895.png)

> spark core实现了spark的基本功能，包含任务调度、内存管理、错误恢复、与储存系统交互的模块，还包含了对弹性分布式数据集的API定义
>
> 独立调度器是spark自己的资源管理器，yarn是hadoop的资源管理器