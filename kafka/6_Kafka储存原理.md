### Kafka储存原理

***

#### 文件储存

> 一个topic分为多个partition，一个partition又分为多个segment。一个segment分为一个.log和一个.index文件。
>
> .log中存的全是生产者生产的消息，.index中存的是索引信息
>
> 当数据量十分庞大，导致.log十分臃肿，会降低索引的速度。所以会分为多个segment。对应的.log和.index文件以.log中的首offset命名
>
> offset的值是消息的唯一标识，第一条message的offset就是1。
>
> 

![image-20200629151110403](F:\学习笔记\kafka\imgs\Kafka储存.png)


