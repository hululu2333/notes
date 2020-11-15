### 使用Flume监控本地单个文件

***

#### 大概流程

> 在生产环境中，更常见的可能是监控文件系统中的某个日志文件，而这日志文件中的内容会随着生产活动的进行而动态增加
>
> 1. 启动agent监控某个日志文件
> 2. 当日志文件内容有新增时，用source将其捕获，并放进channel
> 3. sink将数据从channel中取出，并放进hdfs



#### 实现步骤

##### 创建Agent的配置文件

> 在job目录下创建file-flume-hdfs.conf
> 在文件中增加以下内容

``` bash
#为Agent的各个组件命名
#a2指的是当前Agent的名称，当一台主机上同时运行多个Agent时，这就是区分不同Agent的标识
#r1是当前Agent的一个source，可以有多个
a2.sources = r1
a2.sinks = k1
a2.channels = c1

#为名为r1的source写一些配置信息，如果还有一个source的话，也要配置那一个
a2.sources.r1.type = exec
a2.sources.r1.command = tail -F /tmp/logs/app-2020-06-20.log
a2.sources.r1.shell = /bin/bash -c 
#用bash来解释command
#source的type不同，需要配置的内容也不同
#具体什么type对应什么配置，可以看官方文档

#为名为k1的sink写配置
a2.sinks.k1.type = hdfs
a2.sinks.k1.hdfs.path = hdfs://hadoop01:9000/flume/%y-%m-%d/%H
#%y%m%d是年月日信息
#以上两个属性是必须配置的，除此之外还有一些方便开发的属性，可以按照需求选择配或不配
a2.sinks.k1.hdfs.filePrefix = logs- 
#上传的文件的前缀

a2.sinks.k1.hdfs.round = true 
#是否按照时间滚动文件夹
a2.sinks.k1.hdfs.roundValue = 1 
#多少时间单位新建一个文件夹
a2.sinks.k1.hdfs.roundUnit = hour 
#定义一个时间单位是多久

a2.sinks.k1.hdfs.useLocalTimeStamp = true 
#是否使用本地时间戳
a2.sinks.k1.hdfs.batchSize = 1000 
#积攒多少Event才flush到HDFS一次
a2.sinks.k1.hdfs.fileType = DataStream 
#设置文件类型

a2.sinks.k1.hdfs.rollInterval = 60 
#多久生成一个新文件,60秒
a2.sinks.k1.hdfs.rollSize = 134217700 
#当文件为多大时创建一个新文件
a2.sinks.k1.hdfs.rollCount = 0 
#当文件中有几个事件时创建新文件 0代表不按这个标准来新建文件

#指定名为c1的channel的一些属性
a2.channels.c1.type = memory
a2.channels.c1.capacity = 1000 
#设置c1的容量为1000个事件
a2.channels.c1.transactionCapacity = 100 
#设置一次传输的数据量是100个事件

#把source和sink绑定在特定的channel上 
a2.sources.r1.channels = c1
a2.sinks.k1.channel = c1 
#此处为channel和不是channels，说明一个sink只能绑定一个channel
```



##### 执行agent

``` bash
#切换到flume安装目录，执行以下命令
bin/flume-ng agent -c conf/ -f job/file-flume-hdfs.conf -n a2
```
> 因为要把数据存到hdfs中，所以要把一些和hadoop相关的jar包放进flume的lib中
>
> 这些jar包都在$HADOOP_HOME/share下
>
> hadoop-common-2.7.2.jar
>
> commons-configuration-1.6.jar
>
> commons-io-2.4.jar
>
> hadoop-hdfs-2.7.2.jar
>
> htrace-core-3.1.0-incubating.jar


