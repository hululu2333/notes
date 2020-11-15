### 对DataNode进行磁盘扩展

***

#### 设置DataNode的多目录存储

yarn-site.xml

```xml
<!-- 指定hadoop运行时，产生的文件的存放位置 -->
<property>
    <name>hadoop.tmp.dir</name>
    <value>/opt/module/hadoop/data/tmp</value>
</property>
```

> hadoop.tmp.dir是hadoop文件系统依赖的基础配置，很多路径都依赖它。如果hdfs-site-xml中不配置namenode 和datanode的存放位置，默认就放在这个路径下
>
> 当挂载在/opt/module/hadoop/data/tmp路径上的磁盘分区的容量用完了之后，hadoop就不能正常储存文件了，DataNode自然也不能存放更多的数据了，这时我们可以配置Datanode的多目录，让DataNode的文件除了可以放在默认目录下，还可以放在其他分区所挂载的目录下



在hdfs-site.xml中添加以下属性

```xml
<!-- 指定hadoop运行时，产生的文件的存放位置 -->
<!-- 其中hd2是硬盘sdb2所挂载的目录 -->
<property>
    <name>dfs.datanode.data.dir</name>
    <value>file:///${hadoop.tmp.dir}/dfs/data1,file:///hd2/dfs/data2</value>
</property>
```



#### 开启磁盘的数据均衡

``` bash
$HADOOP_HOME/sbin/start-balancer.sh -threshold 10
#如果一个节点有多个磁盘来储存数据，则会尽量让这些磁盘的利用率相同。参数10的意思是让这些磁盘的利用率相差不超过百分之十
```



#### 停止磁盘的数据均衡

``` bash 
$HADOOP_HOME/sbin/stop-balancer.sh
#开启磁盘的数据均衡会消耗资源，当数据均衡的差不多了，就可以把这个关掉
```

