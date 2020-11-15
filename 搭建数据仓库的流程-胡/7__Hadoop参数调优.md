### Hadoop参数调优

***

#### hdfs参数调优

> hdfs-site.xml

> dfs.namenode.handler.count=20*log2(Cluster size)	如果集群规模为8，则设为60
>
> dfs.name.handler.count的值默认是10，但如果集群的规模比较大，需要增大这个数
>
> 这个值代表用于处理DataNode心跳的线程的个数，每个DataNode中都有大量的块，每个块每隔一段时间都要去Namenode中注册。如果用于处理DataNode心跳的线程太少，会导致NameNode中的元数据不能及时更新
>
> 客户端对元数据的操作也是用这些线程，如果集群拥有大量客户端，也需要扩大这个线程池



#### Yarn参数调优

> yarn-site.xml

> yarn.nodemanager.resource.memory-mb
>
> 这个属性表示在当前节点中，yarn可用的物理内存的总量。默认为8G。如果这个节点的yarn的任务非常繁重，就算节点的内存很大，yarn也只能占用8G。这会导致虽然内存还剩下很多，相关任务还是跑的很慢

> yarn.scheduler.maximum-allocation-mb
>
> 这个属性表示单个任务可申请的最大物理内存