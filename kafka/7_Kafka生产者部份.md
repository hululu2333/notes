### kafka生产者部份

***

#### 生产者分区策略

> 生产者生产数据时可以指定分区。
>
> 如果没有指定分区，但指定了key值，就按照key的哈希值模上分区数，来分区
>
> 如果都没指定，那就按照轮询的方式来分区

#### 确认Kafka成功收到数据——ISR

> 为了确保Kafka一定能收到生产者发来的消息，当相应的partition收到消息后，要给生产者回一个ack。如果生产者收到了ack，证明发送成功，再继续发送下一条消息。
>
> 这个ack什么时候发送呢？如果leader partition收到了消息就发送ack，这时其他follower还没将消息备份过来，如果这时leader发生故障，新接任的leader可能没有这条消息，造成数据丢失
>
> 为此Kafka引入ISR机制，ISR是一个同步备份集。如topic-0有三个备份，分别在host1,host2,host3中，这时我们将host1和host2加入ISR，当ISR中的所有broker都备份好了的新的数据时，发送ack给生产者。如果ISR中有broker迟迟没有备份新数据，就将其从ISR中踢出。如果leader故障，新的leader就从ISR中选出。这样就避免了数据丢失，也不需要所有broker都备份好了在发送ack



#### 确认Kafka成功收到数据——ack

> 为确保Kafka收到数据，Kafka要给生产者回复ack后生产者才会发送下一条数据
>
> 有些数据不怎么重要，这时可以降低数据的安全性来提高工作效率
>
> 通过acks参数，我们可以规定数据安全的几个级别
>
> acks=0	生产者不等待ack就直接发送下一条数据
>
> acks=1	leader接收到数据后直接给生产者发送ack
>
> acks=-1	ISR中所有broker都将数据备份好后，leader给生产者发送ack



#### 故障处理

![image-20200701092349931](F:\学习笔记\kafka\imgs\故障处理.png)

> 在生产数据的过程中，Kafka的任意节点都有可能坏掉。为了保证消费者正确的读取数据，并且所有副本保存的数据一致，这里引入了HW和LEO

***

##### 保证消费者正确读取数据

> 当消费者读到leader的第16条数据时，leader挂掉，第二个follower接任。这时消费者向新任leader索要第17条数据，新任leader一共才12条。这样会报错
>
> 每个副本最高的offset被叫做LEO，所有副本中最低的LEO被叫做HW。消费者读取数据时，最多读到HW处，这样可以确保将来无论谁接任leader，消费者都不会读取数据时出错

***

##### 保证各副本保存的数据一致

> 上图可知，在同一时刻各副本保存的数据都不完全一样。这时如果leader挂掉，第二个follower接任，由于ISR中没有全部同步完成，ack就没有发送给生产者。生产者还会将13-19的数据再发一遍，这时第三个follower已经有13-15了，最后会导致新leader数据正常，第三个follower多出三条数据
>
> 每当有follower接任leader时，他会把所有follower的HW之后的数据都清除。以此来保证各副本间的数据一致性
>
> 新任leader的HW之后的数据不会清除，follower会同步leaderHW后的数据。然后leader再接收生产者的数据。这可能会导致数据重复



#### 幂等性——Exactly Once

> 同一份数据无论生产者发送几次，最终我们只保留一份
>
> 以前是在消费者端做去重，现在引入了幂等性，可以在Kafka中完成去重

> 为实现幂等性，我们将生产者端的enable.idompotence设置为true。
>
> 然后Kafka集群会给每个生产者分配一个pid
>
> 生产者发送不同消息时会附带不同的Sequence Number，Kafka会缓存以往发送过的消息的<pid,Sequence Number>。当新发送的消息的pid和Sequence Number与以前重复了，就判定消息重复，不保存这一条
>
> 当生产者重启后，生产者会拿到一个新的pid



#### 事务性

> 引入事务性，可以解决幂等性不能跨分区跨会话的问题。
>
> 我们可以给生产者指定一个TransactionID，当生产者生产数据时，broker也会给生产者分配一个pid，并且把TransactionID和pid绑定起来。当生产者重启后，它会根据自己的TransactionID找回原来的pid，这样可以保证生产者重启后还能实现幂等性