## 事件驱动应用

### Process Functions

#### 简介

> 我们调用`map()`算子时需要传入`MapFunction`。那我们要调用`process()`算子时需要传入`ProcessFunction`
>
> `map()`算子的`MapFunction`可以对事件进行处理（对事件进行映射），`ProcessFunction`同样可以对事件进行处理，而且可以使用`Timer`和`State`。因此`ProcessFunction`是Flink 创建`Event-driven Applications`的基础。
>
> `ProcessFunction`和 `RichFlatMapFunction` 十分相似， 但是增加了 `Timer`。

***

#### KeyedProcessFunction

> ProcessFunction的类型有好几种，包括 `KeyedProcessFunction`、`CoProcessFunctions`、`BroadcastProcessFunctions` 等。`KeyedProcessFunction`应该是最常用的一种
>
> `KeyedProcessFunction` 是一种 `RichFunction`。类似于`RichFlatMapFunction`，它可以访问使用 Managed Keyed State 所需的 `open` 和 `getRuntimeContext` 方法。

***

#### 示例

``` java
//利用processFunction实现类似窗口的功能
.process(new KeyedProcessFunction<String, Tuple2<String, Long>, Object>() {
    //key相同的所有事件共用一个KeyedProcessFunction对象（对keyedSream进行操作的算子应该都是如此）

    //key相同的所有事件共用一个map对象来存储状态
    private MapState<Long, String> state;

    long hour = 60 * 1000; //一分种这么多毫秒

    //KeyedProcessFunction对象被创建的时候调用一次，可以拿到这个key所对应的状态
    @Override
    public void open(Configuration parameters) throws Exception {
        MapStateDescriptor<Long, String> desc = new MapStateDescriptor<Long, String>("state", Long.class, String.class);
        state = getRuntimeContext().getMapState(desc);
    }

    //processElement 和 onTimer 都提供了一个上下文对象，该对象可用于与 TimerService 交互
    //计时器触发时调用onTimer方法
    @Override
    public void onTimer(long timestamp, OnTimerContext ctx, Collector<Object> out) throws Exception {
        String result = state.get(timestamp);
        out.collect(result);
        state.remove(timestamp);

    }

    //一个事件进来，调用一次
    @Override
    public void processElement(Tuple2<String, Long> stringLongTuple2, Context context, Collector<Object> collector) throws Exception {
        long eventTime = stringLongTuple2.f1;
        TimerService timerService = context.timerService(); //拿到时间服务，通过这个可以获得当前的watermark

        long currentWatermark = timerService.currentWatermark();

        if (eventTime <= currentWatermark) {
            //如果当前的事件事件小于等于当前的watermark
            //说明该事件所属的窗口已经被触发且销毁了
        } else {
            // 这个事件所属的窗口的结束时间戳
            long endOfWindow = (eventTime - (eventTime % hour) + hour - 1);

            System.out.println("注册时间戳为" + endOfWindow + "的timer");

            // 在service中注册一个时间戳为endOfWindow的timer，这个timer可以调用onTimer方法
            // 当时间戳大于endOfWindow的watermark到了，时间戳为endOfWindow的timer就调用ontimer方法，并把时间戳传给ontimer方法
            timerService.registerEventTimeTimer(endOfWindow);

            if (state.contains(endOfWindow)) {
                String result = state.get(endOfWindow);
                result = stringLongTuple2.f0 + result + "-";
                state.remove(endOfWindow);
                state.put(endOfWindow, result);
            } else {
                state.put(endOfWindow, stringLongTuple2.f0);
            }

        }
    }
})
```

***

#### 总结

> 用`ProcessFunction`能实现窗口的功能，但是能用window()来实现就用window()。当window()实现起来比较困难时再考虑用`ProcessFunction`
>
> `ProcessFunctions` 的另一个常见用例是清理过时 State，我们可以用timer来达到这点



### 旁路输出（Side Outputs）

#### 简介

> 有几种情况，我们需要从dataflow中获取一个侧dataflow，将在主dataflow中会被抛弃的事件输出到侧输出流。比如迟到的事件

> 除迟到事件外，还可以把异常情况、格式错误的事件和operator 告警输出到侧输出流

> 除此之外，旁路输出也是实现流的 n 路分割的好方法。

***

#### 示例

> 上一个示例中我们把迟到的事件抛弃了，现在我们可以尝试把他放进侧输出流

``` java
//创建一个OutputTag对象，我们可以利用context将事件输出到这个Tag里，然后通过lateEvents这个对象，可以从主dataflow中获取侧dataflow
private static final OutputTag<Tuple2<String,Long>> lateEvents = new OutputTag<Tuple2<String,Long>>("lateEvents"){};
```

``` java
//在processElement方法中
if (eventTime <= currentWatermark) {
    //如果当前的事件事件小于等于当前的watermark
    //说明该事件所属的窗口已经被触发且销毁了
    //这是我们不抛弃，而是输出到侧输出流
    context.output(lateEvents,stringLongTuple2);
}
```

``` java
//拿到侧输出流
SingleOutputStreamOperator hourlyTips = ds
    .process(new KeyedProcessFunction<String, Tuple2<String, Long>, Object>());

hourlyTips.getSideOutput(lateEvents).print();
```

