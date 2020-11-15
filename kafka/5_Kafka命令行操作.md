### Kafka命令行操作

***

#### kafka-topics.sh

``` bash
#查看当前服务器中所有topic
bin/kafka-topics.sh --zookeeper hadoop01:2181 --list

#创建新topic,3个副本一个分区，topic名为first
bin/kafka-topics.sh --zookeeper hadoop01:2181 --create --replication-factor 3 --partitions 1 --topic first

#删除topic
bin/kafka-topics.sh --zookeeper hadoop01:2181 --delete --topic first


```



#### kafka-console-***.sh

> 在控制台生产或消费数据的脚本，一般用于测试

``` bash
#指定数据放在哪个kafka节点的哪个topic中
bin/kafka-console-producer.sh --topic first --broker-list hadoop01:9092

#订阅并消费某一主题的的消息。kafka0.9以后，消费者消费的位置信息放在kafka集群中
bin/kafka-console-consumer.sh --topic first --bootstrap-server hadoop01:9092 --from-beginning

```

