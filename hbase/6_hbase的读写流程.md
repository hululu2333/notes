### hbase的读写流程

***

> hbase的读操作要比写操作慢，这是个与众不同的地方



#### hbase的写流程

> 假设我们在客户端执行put 'student','1001','info:name','bighu'

> 1.客户端向zookeeper集群请求meta表所在的位置（meta表中存了用户表的所有region的位置信息，当一个表有多个region时，meta表中的行键里会表明该region的起始行键。这样我们就能找到与我们插入数据的行键匹配的region位置）
>
> <img src="C:\Users\hu\AppData\Roaming\Typora\typora-user-images\image-20201110194427309.png" alt="image-20201110194427309" style="zoom: 50%;" />
>
> 2.zookeeper集群返回一个地址
>
> 3.客户端通过这个地址拿到meta表，并将meta表中的信息缓存在本地。以后再需要meta表时就不需要去连接zookeeper了，这时我们可以通过meta表拿到我们想要的region的地址。（如果按照缓存的meta表拿不到我们想要的东西，那就是meta表更新了，这时会去zookeeper请求新的meta表）
>
> 4.客户端将put请求发送到region所在的那个RegionServer中
>
> 5.RegionServer首先将改动写入预写入日志，然后才写入内存中。对于客户端来说，这时写操作已经结束。因为数据已经写到内存中，可以读到了
>
> 6.RegionServer接下来会将数据从内存中fullsh到storefile中。storefile是储存在hdfs中的。wal也会和hdfs中的进行同步

![image-20201110195621388](F:\学习笔记\hbase\image\hbase的写流程.png)



#### flush流程

> flush指的是将储存在HRegion中的内存中的数据写到hdfs中。
>
> 每flush一次就会生成一次HFile（具体生成几个HFile要看HRegion的内存中的数据属于几个Store）
>
> **当内存中有多个相同的字段时，HRegion只会把时间戳最大的那个字段flush到hdfs中，时间戳较小的那几个就被删了。所以说在大合并和flush阶段都会删除数据**
>
> 在hbase中，无论是增删改，都是以增加记录的形式。所以在我们进行写操作的过程中，HRegion中的内存里会有越来越多的记录。当占用到了一定程度的内存时，HRegion会将这些记录从内存中上传到hdfs中
>
> 在flush过程中会暂停客户端的读写操作
>
> flush的具体策略可以在hbase-site.xml中通过配置来确定



#### hbase的读流程

> 假设我们在客户端执行get 'student','1001'（如果是执行scan怎么办？？）

> 1.前几步和写流程一样，从zookeeper中拿到meta表的地址，然后获取meta表。从meta表中找到你要访问的region的地址。
>
> 2.将get请求发送到对应的Hregion中，Hregion将hdfs中的student表中的行键为1001的记录全加载到Block Cache中。（Block Cache是用作缓冲的一块内存）
>
> 3.对比Block Cache和Memory Store中的数据，如果有行键、列族列名都相同的数据，就将时间戳大的那个返回给客户端

![image-20201112164200271](C:\Users\hu\AppData\Roaming\Typora\typora-user-images\image-20201112164200271.png) 