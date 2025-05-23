### Kafka介绍

***

> Kafka是一个分布式的基于发布/订阅模式的消息队列
>
> Kafka依赖于zookeeper，启动Kafka前必须先启动zookeeper
>
> 发布/订阅模式分为两种，一种是消费者主动去拉取消息，一种是队列主动把消息推送给订阅了的消费者。类似于微信公众号。Kafka属于第一种



#### 什么是消息队列

> 消息队列一般用于异步处理，用来缓存数据
>
> 比如qq，当两个人同时在线时，a向b发送一个信息，信息从a直接就到了b的客户端，这就是同步处理
>
> 当b不在线时，a向b发送个信息，b接收不到。这时消息就进入消息队列中，当b上线了，消息队列再把信息发送给b



#### Kafka的作用

> 灵活性：Kafka是分布式的消息队列，所以可以根据需求动态的增减Kafka节点
>
> 峰值处理：当大量数据来袭，处理数据的能力跟不上时，Kafka可以缓存大量数据
>
> 异步处理：用户的数据发送出来后，可以先放在消息队列中不马上处理
>
> 解耦：生产数据模块和消费数据模块方便进行修改，只要符合Kafka的接口就行



#### 消息队列的两种模式

##### 点对点模式（一对一）

> 当一个消费者从队列中取出一个消息后，消息队列中的这个消息就被删除。
>
> 虽然一个消息队列可以有多个消费者来消费，但一个消息只能被一个消费者拿到

***

##### 发布/订阅模式（一对多）

> 当消费者从topic中读取一个消息时，这个消息不会被删除。这个消息会被所有订阅了该消息的消费者消费。
>
> 但消息队列终究是拿来缓存数据的，一个消息并不会永远存在这里，过了一定的时间，它就会被删除