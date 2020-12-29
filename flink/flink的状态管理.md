### flink的状态管理

***

#### 什么是flink中的状态

> 我们知道，Flink的一个算子有多个子任务，每个子任务分布在不同实例上，我们可以把状态理解为某个算子子任务在其当前实例上的一个变量，变量记录了数据流的历史信息。
>
> 当新数据流入时，我们可以结合历史信息来进行计算。实际上，Flink的状态是由算子的子任务来创建和管理的。一个状态更新和获取的流程如下图所示，一个算子子任务接收输入流，获取对应的状态，根据新的计算结果更新状态。一个简单的例子是对一个时间窗口内输入流的某个整数字段求和，那么当算子子任务接收到新元素时，会获取已经存储在状态中的数值，然后将当前输入加到状态上，并将状态数据更新。

![img](F:\学习笔记\flink\imgs\flink状态.jpg)



#### flink状态的作用

> 有状态的计算是流处理框架要实现的重要功能，因为稍复杂的流处理场景都需要记录状态，然后在新流入数据的基础上不断更新状态。下面的几个场景都需要使用流处理的状态功能：
>
> - 数据流中的数据有重复，我们想对重复数据去重，需要记录哪些数据已经流入过应用，当新数据流入时，根据已流入过的数据来判断去重。
> - 检查输入流是否符合某个特定的模式，需要将之前流入的元素以状态的形式缓存下来。比如，判断一个温度传感器数据流中的温度是否在持续上升。
> - 对一个时间窗口内的数据进行聚合分析，分析一个小时内某项指标的75分位或99分位的数值。
> - 在线机器学习场景下，需要根据新流入数据不断更新机器学习的模型参数。



#### Flink的几种状态类型

> flink的状态可以分为两种，托管状态（Managed State）和原生状态（Raw State）。
>
> 从名称中也能读出两者的区别：Managed State是由Flink管理的，Flink帮忙存储、恢复和优化，Raw State是开发者自己管理的，需要自己序列化。

![img](F:\学习笔记\flink\imgs\flink的两种状态.jpg)

##### 两种状态的具体区别

- 从状态管理的方式上来说，Managed State由Flink Runtime托管，状态是自动存储、自动恢复的，Flink在存储管理和持久化上做了一些优化。当我们横向伸缩，或者说我们修改Flink应用的并行度时，状态也能自动重新分布到多个并行实例上。Raw State是用户自定义的状态。
- 从状态的数据结构上来说，Managed State支持了一系列常见的数据结构，如ValueState、ListState、MapState等。Raw State只支持字节，任何上层数据结构需要序列化为字节数组。使用时，需要用户自己序列化，以非常底层的字节数组形式存储，Flink并不知道存储的是什么样的数据结构。
- 从具体使用场景来说，绝大多数的算子都可以通过继承Rich函数类或其他提供好的接口类，在里面使用Managed State。Raw State是在已有算子和Managed State不够用时，用户自定义算子时使用。



##### Keyed State和Operator State

> 对Managed State继续细分，它又有两种类型：Keyed State和Operator State。这里先简单对比两种状态，后续还将展示具体的使用方法。
>
> Keyed State是`KeyedStream`上的状态。假如输入流按照id为Key进行了`keyBy`分组，形成一个`KeyedStream`，数据流中所有id为1的数据共享一个状态，可以访问和更新这个状态，以此类推，每个Key对应一个自己的状态。下图展示了Keyed State，因为一个算子子任务可以处理一到多个Key，算子子任务1处理了两种Key，两种Key分别对应自己的状态。



![img](F:\学习笔记\flink\imgs\keyedState.jpg)

> Operator State可以用在所有算子上，每个算子子任务或者说每个算子实例共享一个状态，流入这个算子子任务的数据可以访问和更新这个状态。下图展示了Operator State，算子子任务1上的所有数据可以共享第一个Operator State，以此类推，每个算子子任务上的数据共享自己的状态。

![img](F:\学习笔记\flink\imgs\operator.jpg)

> 无论是Keyed State还是Operator State，Flink的状态都是基于本地的，即每个算子子任务维护着这个算子子任务对应的状态存储，算子子任务之间的状态不能相互访问。
>
> 在之前各算子的介绍中曾提到，为了自定义Flink的算子，我们可以重写Rich Function接口类，比如`RichFlatMapFunction`。使用Keyed State时，我们也可以通过重写Rich Function接口类，在里面创建和访问状态。对于Operator State，我们还需进一步实现`CheckpointedFunction`接口。

![img](F:\学习笔记\flink\imgs\keyedState和operatorState的区别.jpg)



参考：https://zhuanlan.zhihu.com/p/104171679

横向扩展和keyedState、OperatorState的具体使用可以参考这个链接