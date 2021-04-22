## 流式分析

***

### Event Time and Watermarks

#### Event Time

> flink支持三种时间语义，事件时间、摄取时间和处理时间

> 在大多数场景中，我们一般使用事件时间。例如在计算过去的特定一天里第一个小时股票的最高价格时，我们应该使用事件时间。这样的话，无论什么时间去计算都不会影响输出结果。

> 如果想要使用事件时间，需要额外给 Flink 提供一个时间戳提取器和 Watermark 生成器，Flink 将使用它们来跟踪事件时间的进度。

***

#### Watermarks

> **应用场景：**如果用event time来当事件的时间，那么流中数据很可能是乱序的。为了解决乱序，我们可以引入watermakrs。我们可以在流中插入带有时间戳的watermark

> **watermarks的作用：**带有时间戳为*t* 的 watermark到达一个算子，说明事件时间小于t的事件都很可能已经到了这个算子。watermark一般配合窗口来使用，时间戳为t的watermark到达了窗口，那么所有结束时间小于t的窗口就可以开始执行了

> **watermark生成器：**watermark生成器将带有时间戳的watermark对象插入流中。

> **watermark生成策略：**我们可以在数据源上就生成watermark，也可以在流中间再生成。在数据源上生成会更好。当我们在流中间生成时，我们一般会记录下流中事件的最大事件时间t。然后根据我们设置的最大无序边界s，发出时间戳为t-s的watermark。是定时发出watermark还是定点发出（某个特点事件到来，发出watermark），可以自己设置。

> **总结：**我们可以把`assignTimestampsAndWatermarks()`看成是一个算子，如果不手动设置的话，env的并行度是多少，就有几个`assignTimestampsAndWatermarks`。也就有几个watermark生成器，就这导致不同的dataflow中的的watermark可能不同。不过没关系，`assignTimestampsAndWatermarks()`算子把watermark向下广播时，只会把最小的watermark发出去。这就保证下游算子的所有子算子收到的watermark是一致且有序的
>
> 假如多条dataflow中，有一条一直没有数据，就会导致这条dataflow中的`assignTimestampsAndWatermarks()`算子的watermark生成器产生的watermark的时间戳始终是初始值。就会导致下游算子只能收到时间戳为初始值的watermark，导致窗口无法被启用

``` java
//示例
.assignTimestampsAndWatermarks(new AssignerWithPeriodicWatermarks(){
    //在流中加上watermark，确保数据是有序得
    // 定义两分钟的容忍间隔时间，即允许数据的最大乱序时间
    private long maxOutofOrderness = 60 * 1000 * 2;
    // 观察到的最大时间戳
    private long currentMaxTs = Long.MIN_VALUE;

    @SneakyThrows
    @Override
    public long extractTimestamp(Object o, long previousElementTimestamp) {
        String time = JSONObject.parseObject((String) o).getString("data_col_time");
        long currentTs = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").parse(time).getTime();

        currentMaxTs = Math.max(currentTs,currentMaxTs);

        return currentTs;
    }

    @Nullable
    @Override
    public Watermark getCurrentWatermark() {
        return new Watermark(currentMaxTs - maxOutofOrderness);
    }
})
```



### Windows

#### Windows简介

> 我们在操作无界数据流时，经常需要把无界数据流分解成有界数据流聚合分析

> 用 Flink 计算窗口分析取决于两个主要的抽象操作：*Window Assigners*，将事件分配给窗口（根据需要创建新的窗口对象），以及 *Window Functions*，处理窗口内的数据。

> Flink 的窗口 API 还具有 *Triggers* 和 *Evictors* 的概念，*Triggers* 确定何时调用窗口函数，而 *Evictors* 则可以删除在窗口中收集的元素。
>
> 我们可以在keyby()后调用window()，也可以不keyby()调用windowAll()。但后者使程序无法并行执行

> **总结：**`keyby()`后形成一个keyedStream。这时调用`window()`，达到下图这个效果。假如key为user1和user2的事件进入dataflow1，user3的事件进入dataflow2。这时我们在dataflow1上调用`window()`，实际上是创建了两个window，key相同的所有事件共用一个window，当window被触发执行时，也是key相同的所有事件公用一个驱动函数对象。

<img src="F:\学习笔记\flink\imgs\窗口.png" alt="image-20210421143341390" style="zoom:50%;" />

***

#### windows（窗口）应用函数

> 创建好窗口后，等watermark到了。我们就可以将结束时间小于watermark的窗口执行然后销毁了。但是如何执行这个窗口呢？所以我们调用window()后还必须指定函数来对窗口中的数据进行操作。如max()，或者是`ProcessWindowFunction` 等。

> **ProcessWindowFunction：**调用`ProcessWindowFunction` 会把窗口中的内容缓存下来，供接下来全量计算，相当于是批处理。flink会缓存这些事件，直到触发窗口为止，这个开销是比较大的
>
> **ReduceFunction或ReduceFunction ：** 或者像流处理，每一次有事件被分配到窗口时，都会调用 `ReduceFunction` 或者 `AggregateFunction` 来增量计算；
>
> **结合两者：**通过 `ReduceFunction` 或者 `AggregateFunction` 预聚合的增量计算结果在触发窗口时， 提供给 `ProcessWindowFunction` 做全量计算。
>
> 

***

#### 窗口的创建规则

> **事件会触发窗口的创建：**换句话说，如果在特定的窗口内没有事件，就不会有窗口，就不会有输出结果。

> **时间窗口会和时间对齐：**仅仅因为我们使用的是一个小时的处理时间窗口并在 12:05 开始运行您的应用程序，并不意味着第一个窗口将在 1:05 关闭。第一个窗口将长 55 分钟，并在 1:00 关闭。



### 晚到的事件

默认场景下，超过最大无序边界的事件会被删除，但是 Flink 给了我们两个选择去控制这些事件。

您可以使用一种称为[旁路输出](https://ci.apache.org/projects/flink/flink-docs-release-1.12/zh/learn-flink/event_driven.html#side-outputs) 的机制来安排将要删除的事件收集到侧输出流中，这里是一个示例:

```
OutputTag<Event> lateTag = new OutputTag<Event>("late"){};

SingleOutputStreamOperator<Event> result = stream.
    .keyBy(...)
    .window(...)
    .sideOutputLateData(lateTag)
    .process(...);

DataStream<Event> lateStream = result.getSideOutput(lateTag);
```

我们还可以指定 *允许的延迟(allowed lateness)* 的间隔，在这个间隔时间内，延迟的事件将会继续分配给窗口（同时状态会被保留），默认状态下，每个延迟事件都会导致窗口函数被再次调用（有时也称之为 *late firing* ）。

默认情况下，允许的延迟为 0。换句话说，watermark 之后的元素将被丢弃（或发送到侧输出流）。

举例说明:

```
stream.
    .keyBy(...)
    .window(...)
    .allowedLateness(Time.seconds(10))
    .process(...);
```

当允许的延迟大于零时，只有那些超过最大无序边界以至于会被丢弃的事件才会被发送到侧输出流（如果已配置）。

