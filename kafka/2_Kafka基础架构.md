### Kafka基础架构

***

![image-20200624222951864](F:\学习笔记\kafka\imgs\Kafka架构图.png)

#### Kafka集群缓存消息

> Kafka是分布式的，所以有着Kafka集群
>
> Kafka的每个节点中，也就是每个队列中有多种Topic，每种Topic代表一种类型的消息。而每种Topic分为几个Partition，Partition0和Partition1等其余Partition共同构成了这种Partition的所有消息。这样做可以提高生产和消费的并发性
>
> 因为是分布式的，为了保证高可用性，每个Partition都有一些副本。如在节点1中的Partition0和在节点二中的Partition0的内容是完全一致的。这些相同的Partition中也分为Leader和Follower，生产者和消费者进行操作时也只操作Leader，Follower仅仅是作为备份



#### 消费者消费消息

> 消费者分为单个消费者和消费者组
>
> 消费者组订阅了Topic为A的消息，当消费者组去读取Topic A的Partition 0时，只能由消费者组中的一个消费者来消费。其他的消费者只能消费Topic A的其他Partition
>
> 消费者组提高了消费能力 
>
> 消费者在消费时也会记录当前的消费位置，当消费者重启后以便继续从上次断开的位置继续消费。而这位置信息就储存在zookeeper或Kafka节点中。0.9版本之前在zookeeper中，0.9之后在Kafka中
>
> 