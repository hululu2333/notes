### hbase的基础架构

***

![image-20201107164616173](F:\学习笔记\hbase\image\hbase的不完整架构.png)

> hbase集群中分为Master和RegionServer
>
> RegionServer负责数据的增删查，也负责合并或切分region
>
> Master负责表级操作，也负责将regions分配到每个RegionServer中，并监控RegionServer的状态
>
> **hbase是天生高可用的，你可以直接启动多个Master。**所以hbase依赖于zookeeper。最先启动的那个Master处于active状态，他会把元数据存在zookeeper中。当处于active的Master挂掉，其余几个处于standby的Master就争夺active的位置，新上位的Master就从zookeeper中读取数据，接替上一个Master的工作

