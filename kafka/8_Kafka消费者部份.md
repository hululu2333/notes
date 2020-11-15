### Kafka消费者部份

***

#### 消费者分区策略

> 这是对于消费者组而言的，单个消费者没有消费分区这一说法
>
> 当消费者个数发生变化，会触发重新分配机制

##### RoundRobin

> 循环分区法，将消费者组订阅的所有Topic中的所有Partition按哈希值排序。消费者组中的第一个消费者消费第一个Partition，以此类推
>
> 用这种方法的前提是消费者组中的消费者订阅的主题相同

***

##### Range

> 默认的分区方法，用于处理消费者组中的消费者订阅的主题不同的情况
>
> Range方法是根据Topic来分配的，假如有一个消费者组，消费者a订阅了TopicA和TopicB。消费者b订阅了TopicB。TopicB被同一消费者组的两个消费者订阅了，所以TopicB中的Partition均匀分给a和b，TopicA只被同一消费组中的一个消费者订阅，所有Partition都发给a



#### 消费者offset的储存

> 0.9之前，消费者把offset储存在zookeep中，0.9之后存在Kafka集群中
>
> offset是根据消费者组，Topic，Partition来存的

##### 存在zookeeper中

> 消费者组节点是/consumers的子节点

***

##### 存在Kafka中

> offset信息存在Kafka中的一个默认Topic中。这个Topic叫_consumer_offsets
>
> 如果我们要用消费者去消费这个Topic，我们需要在consumer.properties中添加一行exclude.internal.topics=false



#### 消费者offset的重置

##### ConsumerConfig.AUTO_OFFSET_RESET_CONFIG

> 决定消费者重置offset的策略
>
> 这个配置在两种情况下会发挥作用，一是消费者组第一次被创建。二是offset对应的消息已经被删除了，比如消费者组消费到了offset为1000的消息，这时消费者组关闭了，过了若干天，offset为2000以前的消息都被删除了，这时消费者组又被启动，想消费offset为1001的数据，发现数据不存在
>
> 这个属性有两个值，一是earliest，同步到当前所有Partition的最小offset。二是lastest，同步到最大offset