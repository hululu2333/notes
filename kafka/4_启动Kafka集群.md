### 启动Kafka集群

***

#### 启动单个Kafka节点

``` bash
#切换到Kafka安装目录下，执行以下操作。这样的话，会进入一个阻塞状态
bin/kafka-server-start.sh config/server.properties

#我们可以加一个选项，让服务在后台运行
bin/kafka-server-start.sh -daemon config/server.properties
```



#### 一键启动所有Kafka节点

> 编写脚本 kk.sh

``` bash
#!/bin/bash

case $1 in
"start"){
	for i in hadoop01 hadoop02 hadoop03
	do
	echo ------ $i ------
	ssh $i "/opt/module/kafka/bin/kafka-server-start.sh -daemon /opt/module/kafka/config/server.properties"
	done
};;

"stop"){
	for i in hadoop01 hadoop02 hadoop03
	do
	echo ------ $i ------
	ssh $i "/opt/module/kafka/bin/kafka-server-stop.sh"
	done
};;
esac
```

> 一键启动
>
> kk.sh start