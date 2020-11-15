### Flume的使用

***

> Flume的介绍、安装、配置等细节信息，在单独的Flume笔记中有详细的介绍



#### Flume中组件的选择

##### source

> 我们这里选用Taildir source，原因是可以监控动态变化的文件，而且有断点续传功能

***

##### channel

> 我们使用Kfka channel，因为我们要把数据从磁盘文件中传输到Kafka中。使用Kafka channel就可以省去sink，提高传输效率



#### 创建agent的配置信息

> file-flume-kafka.conf

```bash
#为Agent的各个组件命名
a1.sources = r1
a1.channels = c1 c2


#为名为r1的source写一些配置信息，如果还有一个source的话，也要配置那一个
a1.sources.r1.type = TAILDIR
a1.sources.r1.positionFile = /opt/module/flume/test/log_position.json
a1.sources.r1.filegroups = f1
a1.sources.r1.filegroups.f1 = /tmp/logs/app.+
a1.sources.r1.fileHeader = true

#为r1配置拦截器
a1.sources.r1.interceptors = i1 i2
a1.sources.r1.interceptors.i1.type = com.hu.flume.interceptor.LogETLInterceptor$Builder
a1.sources.r1.interceptors.i2.type=
com.hu.flume.interceptor.LogTypeInterceptor$Builder

#为r1配置channel选择器
a1.sources.r1.selector.type = multiplexing
#一个Event包括一个header和一个body，header中的数据是键值对形式。
#根据header中key为topic的值来选择channel，topic对应的value为topic_start就发往c1，如果是topic_event就发往c2
a1.sources.r1.selector.header = topic
a1.sources.r1.selector.mapping.topic_start = c1
a1.sources.r1.selector.mapping.topic_event = c2


#配置c1的属性
a1.channels.c1.type = org.apache.flume.channel.kafka.KafkaChannel
a1.channels.c1.kafka.bootstrap.servers = hadoop01:9092,hadoop02:9092,hadoop03:9092
a1.channels.c1.kafka.topic = topic_start
a1.channels.c1.parseAsFlumeEvent = false
a1.channels.c1.kafka.consumer.group.id = flume-consumer

#配置c2的属性
a1.channels.c2.type = org.apache.flume.channel.kafka.KafkaChannel
a1.channels.c2.kafka.bootstrap.servers =hadoop01:9092,hadoop02:9092,hadoop03:9092
a1.channels.c2.kafka.topic = topic_event
a1.channels.c2.parseAsFlumeEvent = false
a1.channels.c2.kafka.consumer.group.id = flume-consumer

#把source和channel绑定
a1.sources.r1.channels = c1 c2
```



#### 将配置文件分发

> xsync file-flume-kafka.conf



#### 在hadoop01和hadoop02中启动这个agent

##### 编写flume的一键启动脚本

> nohub表示在退出账户或关闭终端之后继续运行此进程

``` bash
#!/bin/bash

flume_home="/opt/module/flume"
case $1 in
"start"){
	for i in hadoop01 hadoop02
	do
		echo ------- $i flume启动---------
		ssh i "nohub $flume_home/bin/flume-ng agent -c $flume/conf -f $flume/job/files-flume-kafka.conf -n a1"
	done
};; 
"stop"){
	for i in hadoop01 hadoop02
	do
		echo ------- $i flume停止---------
		ssh $i "ps -ef | grep files-flume-kafka.conf | grep -v grep | awk '{print \$2}' | xargs kill -9"
	done
};;
esac
```

