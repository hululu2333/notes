### hbase的详细架构



![image-20201110184736566](F:\学习笔记\hbase\image\hbase的详细架构.png)

> hbase是依赖hdfs和zookeeper的，所以启动hbase集群之前得先启动zookeeper集群和hdfs集群
>
> hbase中的数据会储存在hdfs中，hbase会通过hdfs客户端将HLog和HFile写进hdfs
>
> zookeeper会替HMaster分担一部分功能，meta表是存在zookeeper中的。所以对数据进行增删改查都是通过zookeeper，而不用经过HMaster。**也就是说DML只需通过zookeeper就能完成，DDL必须由Master来完成，因为Master是高可用的，不确定处于active的Master在哪台主机上，所以还得通过zookeeper来找Master的位置**
>
> HLog是预写入日志，当客户端将读写请求发送到RegionServer中时，RegionServer先将改动写入HLog中，然后再写入Memory Store中。这样的话，当这个RegionServer挂掉，内存中的数据丢失，这时可以通过HLog中的记录来恢复数据。
>
> 一个HRegionServer中会包含多个region，每个region中会有多个store，store个数由列族数决定。
>
> HFile是一种文件格式
>
> 合适的时候，RegionServer会将Memory Store中的数据fullsh到StoreFile中。**StoreFile虽然画在HBase中，但StoreFile是存在hdfs中的**
>
> HLog会不时的和hdfs中储存的那份进行同步

