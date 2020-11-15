### Hdfs

***

#### 什么是HDFS

> HDFS是分布式文件系统，可以在逻辑上将多台计算机的磁盘变成一个整体
>
> HDFS也有块的概念，只不过一般文件系统的块只有几KB，而hadoop2中HDFS块的大小是128M。但是HDFS中小于一个一个块的文件并不会占据整个块的空间
>
> HDFS把块设为这么大主要是为了减少寻址时间
>
> HDFS中，只允许一个客户端对文件进行追加写操作



#### HDFS的组成

> HDFS由NameNode、DataNode和secondNameNode组成

> **NameNode：**NameNode负责储存分布式文件系统中所有目录和文件的元数据，元数据指的是一些基本信息，比如文件被分为几块，分别储存在哪些节点的哪些位置
>
> 这些数据被NameNode保存在内存中。同时这些数据也会以fsimage和editlog的形式在磁盘中保存一份。fsimage是文件系统映像，namenode启动的时候将fsimage加载到内存中，在NameNode启动后，对分布式文件系统的操作首先会被记录到editlog文件中，当客户端操作成功，相应的元数据会更新到内存中
>
> fsimage和editlog这两个文件综合起来，就是一个完整的分布式文件系统的元数据
>
> 当我们重启NameNode时，NameNode会将fsimage和editlogs合并，形成一个新的fsimage，然后把fsimage加载到内存中。
>
> 但是namenode一般不重启，这就导致editslog会越来越大。而且NameNode重启时，将editslog与fsimage合并会消耗大量时间
>
> 
>
> NameNode还负责所有DataNode的监控
>
> DataNode会定时向NameNode发送心跳和它储存的块的信息，于是我们可以在NameNode的web界面看到所有datanode的信息。



> **DataNode：**DataNode负责数据块的实际存储和读写工作，hadoop2.0版本，一个块的大小默认是128M
>
> DataNode每隔一段时间会向NameNode发送心跳，如果长时间没有和NameNode通信，NameNode判断这个节点死亡，然后将这个节点上的数据备份到集群中其他机器上。



> **SecondNameNode：**SecondNameNode一般被称为检查节点，它负责将NameNode的fsimage和editslog合并
>
> 第一次工作时，它将NameNode中的fsimage和editslog读取过来，将这两个文件合并为一个新的fsimage，然后用这个fsimage替换NameNode中的fsimage。以后工作时，就直接读取NameNode中的editslog，因为本地保存了一份和NameNode中一样的fsimage
>
> SecondNameNode解决了editslog越来越大的问题，也解决了namenode重启，将editslog与fsimage合并时花费太长时间的问题。如果将来NameNode节点坏掉，SecondNameNode中保存的fsimage也可以保证大部分的元数据信息不被丢失



#### 客户端上传文件的流程

> 1.向NameNode请求上传文件
>
> 2.NameNode检查客户端要上传的文件是否存在，父目录是否存在，然后响应客户端
>
> 3.如果客户端得到的响应是允许上传，客户端会向NameNode发出请求，询问第一个block上传到哪里
>
> 4.NameNode查看DataNode的情况，并挑三个DataNode的地址返回给客户端（备份数为3的情况）
>
> 5.客户端与最近的DataNode1建立传输通道，然后告知DataNode1它还要将数据传给DataNode2和DataNode3。然后DataNode1会去与DataNode2建立传输通道，DataNode2会与DataNode3建立传输通道
>
> 6.当这些通道都构建完成后，DataNode1响应客户端。这时客户端以64kB的package为单位，将数据发送到DataNode1中。DataNode1一开始用内存接收这些数据，然后一边将这数据发送给DataNode2，一边将这些数据持久化到磁盘中
>
> 7.当一个block传输完成后，客户端再次请求NameNode上传第二个block

``` bash
//判断分布式文件系统中是否已经有/home/a.txt
//判断/home这个目录是否存在
hdfs dfs -put a.txt /home/a.txt
```

