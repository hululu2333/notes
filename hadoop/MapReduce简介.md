### MapReduce简介

***

mapreduce是一个并行的计算框架，或者说是一个编程模型

即然是一个编程模型，那么mapreduce中大部分的辅助功能都已经完成，我们需要写的只是核心部份

适用于处理数据密集性：1.数据量大，数据相互没有关系，计算简单

数据会从map到reduce，这是进程间的传输，所以传输的对象需要序列化。进程间的通信需要用到RPC协议

一个输入分片进入一个map中，在TextInputFormat中，一行数据就是一个输入分片



#### mapreduce的执行过程

> **map阶段：**待处理的数据以数据分片的形式进入map任务，经过map处理后输出键值对
>
> **shuffle阶段：**shuffle分为mapshuffle和reduceshuffle，mapshuffle对从map任务出来的键值对进行分区操作。然后将这些键值对按照分区储存在磁盘中。相同分区的储存在一起。reduceshuffle对键值对进行分组和排序
>
> **reduce阶段**：reduce对输入的数据进行处理，最终将这些数据输出到目的地。可以是hdfs，也可以是数据库。如果输出到hdfs，一个reduce任务会生成一个part文件。



