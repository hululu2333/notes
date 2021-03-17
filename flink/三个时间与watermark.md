### 三种时间与watermark

***

#### 三种时间

> **Event-Time：**当数据源生产一个Event时，会在这个Event内部嵌入一个时间戳。这个时间戳就叫做事件时间，是这个Event产生的时间，事件时间反映了事件真实的发生时间。所以，基于事件时间的计算操作，其结果是具有确定性的
>
> **Ingestion-Time：**摄取时间，事件进入Flink的时间，即将每一个事件在数据源算子的处理时间作为事件时间的时间戳，并自动生成水位线
>
> **Processing-Time：**  处理时间，可以理解为对这个Event进行处理的时候的时间
>
> **Flink中默认使用Processing-Time**。使用Processing-Time时，因为不需要等watermark到了再触发窗口，所以延迟更低。但是相对于Event-Time来说，生成的结果就不确定了



#### 设置不同的时间语义

``` java
//将flink使用的时间修改为EventTime
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
//TimeCharacteristic是一个枚举类，里面有三个元素，IngestionTime、EventTime、ProcessingTime

```



#### 什么是watermark

> **时间窗口：**相比流处理和批处理，Flink采用了一种折中的方法，Flink以固定的缓存块为单位进行网络数据传输，用户可以通过缓存块超时值指定缓存块的传输时机。时间窗口的大小就是超时值，当时间窗口大小为0也就是超时值为0的时候，flink进行的就是纯实时计算
>
> 以下的消息等同于上文的Event

> 当操作符通过基于Event Time的时间窗口来处理数据时，它必须在确定所有属于该时间窗口的消息全部流入此操作符后才能开始数据处理。但是由于消息可能是乱序的，所以操作符无法直接确认何时所有属于该时间窗口的消息全部流入此操作符。
>
> 我们可以把watermark也看作是一个消息，这个消息中包含了一个特殊的时间戳，**Flink的数据源在确认所有小于某个时间戳的消息都已输出到Flink流处理系统后，会生成一个包含该时间戳的WaterMark，插入到消息流中输出到Flink流处理系统中。**
>
> **其实Flink的数据源无法确认某个时间点之前的数据是否已经发送到flink了，**只能根据我们的猜测加个延时。比如EventTime为10:15的数据产生了，我们过一分钟，10:16发送一个时间戳为10:15的waterMark到flink
>
> Flink操作符按照时间窗口缓存所有流入的消息，当操作符处理到WaterMark时，它对所有小于该WaterMark时间戳的时间窗口数据进行处理并发送到下一个操作符节点，然后也将WaterMark发送到下一个操作符节点。
>
> 为了保证能够处理所有属于某个时间窗口的消息，操作符必须等到大于这个时间窗口的WaterMark之后才能开始对该时间窗口的消息进行处理，相对于基于Ingestion-Time的时间窗口，Flink需要占用更多内存，且会直接影响消息处理的延迟时间。对此，一个可能的优化措施是，对于聚合类的操作符，可以提前对部分消息进行聚合操作，当有属于该时间窗口的新消息流入时，基于之前的部分聚合结果继续计算，这样的话，只需缓存中间计算结果即可，无需缓存该时间窗口的所有消息。



#### WaterMark的传播

> 关于水位线的传播策略可以归纳为3点：
>
> - 首先，水位线是以广播的形式在算子之间进行传播
> - Long.MAX_VALUE表示事件时间的结束，即未来不会有数据到来了
> - 单个分区的输入取最大值，多个分区的输入取最小值

![image-20210107161323714](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210107161323714.png)

> 一个算子会分为很多子任务，每个子任务会独立维护一个watermark，当有新的watermark来到这个子任务时，会将新的watermark替换旧的watermark(新的watermark一定高于旧的)。
>
> 这些子任务合起来就是个完整的算子任务，完整的任务也会维护一个watermark，这个watermark取各个子任务中最小的那个watermark
>
> 但这种方式比较适合各个子任务的数据都来自同一个流，如果来自多个流的话，可能会带来较大的性能开销



参考：https://mp.weixin.qq.com/s?__biz=MzU2ODQ3NjYyMA==&mid=2247483891&idx=1&sn=0c51684f2022e63814e90b0dac8e3e89&scene=21#wechat_redirect