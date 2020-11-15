### 对Hadoop集群进行性能测试

***

#### 测试向hdsf写入数据的速度

``` bash
#运行tests.jar包里的TestDFSIO类
hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.7.2-tests.jar TestDFSIO -write -nrFiles 10 -Size 128MB
```

![image-20200617121609787](.\imgs\向hdfs写入数据的速度.png)

> 测试结果：写入速度为8Mbit/s



#### 测试从hdfs读取数据的速度

``` bash
#运行tests.jar包里的TestDFSIO类
hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.7.2-tests.jar TestDFSIO -read -nrFiles 10 -size 128MB
```

![image-20200617122229607](F:\学习笔记\搭建数据仓库的流程-胡\imgs\从hdfs读取数据的速度.png)

> 测试结果：读取速度为43Mbit/s