### RDD、DataFrame和DataSet

***

> 参考：https://blog.csdn.net/weixin_43087634/article/details/84398036
>
> 以下是个人总结

#### RDD和DataFrame相比

> **储存格式：**RDD是分布式的Java对象的集合。DataFrame是分布式的Row对象的集合。如下图所示
>
> **API：**DataFrame除了提供了比RDD更丰富的算子以外，更重要的特点是提升执行效率、减少数据读取以及执行计划的优化，比如filter下推、裁剪等。
>
> **储存方式：**RDD最大的问题是性能限制，因为RDD是储存在JVM内存中的。会因为GC的限制和java序列化成本的升高导致性能下降。而DataFrame直接将数据放在堆外内存（RDD储存在JVM中指的是RDD对象存在JVM？还是说RDD中这个数据集中的数据也是在JVM？我猜测指的是RDD中的数据）
>
> **优化：**除此之外，DataFrame还做了一些查询优化，有效的提高了效率

![image-20201215155239945](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201215155239945.png)



#### DataSet

> DataSet可以看作是DataFrame API的扩展，是Spark最新的数据抽象
>
> 在DataSet中增加了编解码器，当需要访问非堆上的数据时可以避免反序列化整个对象，提高了效率。
>
> DataSet是强类型的，可以避免Dataframe因为在编译期缺少类型安全检查，导致运行时出错.



#### 总结

> 我们可以这样理解，DataSet是DataFrame的升级版，绝大部分情况下我们可以用DataSet来代替DataFrame。
>
> RDD粒度比他们两个要小，就好比hive自动生成的mapreduce程序，虽然方便，但我们不能对程序的一些细节进行定制。
>
> 所以有些情况我们需要使用RDD，如spark mlib