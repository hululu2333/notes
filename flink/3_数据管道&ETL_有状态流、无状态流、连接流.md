## 数据管道 & ETL

***

> Apache Flink 的一种常见应用场景是 ETL（抽取、转换、加载）管道任务。从一个或多个数据源获取数据，进行一些转换操作和信息补充，将结果存储起来。
>
> Flink 的 Table 和 SQL API完全可以满足很多 ETL 使用场景。但无论最终是否直接使用 DataStream API，对这里介绍的基本知识有扎实的理解都是有价值的。
>
> 接下来会使用Flink 的 DataStream API 实现这类应用。



### 无状态的Transformations

> `map()` 和 `flatmap()`这两种算子可以用来实现无状态的Transformation（一般来说一个Transformation 只包含一个算子，但有时候会包含多个，如keyby().max()）

```java
//以下两个算子都是无状态的Transformation
//如果我们想让他们带有状态，可以使用他们所对应的Rich Function
ds.map(new MapFunction<Person, Person>() {
    @Override
    public Person map(Person value) throws Exception {
        if(value.age==20){
            value.age=21;
        }
        return value;
    }
}).flatMap(new FlatMapFunction<Person, Person>() {
    @Override
    public void flatMap(Person value, Collector<Person> out) throws Exception {
        //flatMap可以实现一对多、多对一或多对多，每个事件都会经过flatMap方法，
        //可以选择不将这个事件放入out，也可以将这个事件拆分为几块放进out中
        //每调用一次out.collect()，就会把一个事件放入流中
        if(value.age!=18){
            out.collect(value);
            //out.collect(value.name);
        }
    }
})
```



### Keyed Streams

> 我们调用`keyBy()`能获得Keyed Stream，在keyed Stream上我们能进行一些聚合操作，例如max。这种Transformation是有状态的，不过状态的处理对我们是透明的

> keyBy算子会通过 shuffle 来为数据流进行重新分区。总体来说这个开销是很大的，它涉及网络通信、序列化和反序列化。

> KeySelector 不仅可以从事件中抽取键。你也可以按想要的方式计算得到键值，只要最终结果是确定的，并且实现了 `hashCode()` 和 `equals()`。`hashCode()`用于判断分区，`equals()`用于判断键是不是相同

> 键必须按确定的方式产生，因为它们会在需要的时候被重新计算，而不是一直被带在流记录中。
>
> 状态快照在`flink应用` 执行时会获取并存储dataflows中整体的状态，也会保存当前状态下数据源数据消费的偏移量。当进程出现故障，这时数据源的消费位置恢复为上一个check point中记录的位置，dataflows中各个Transformations的状态也恢复为上一个check point中记录的。所以`keyBy()`的键不能用随机数，这样会导致故障恢复后的运行结果与之前不一致

``` java
ds.keyBy(new KeySelector<Person, Object>() {
    @Override
    public Object getKey(Person value) throws Exception {
        //filier、map这几个算子返回的都是单输出流
        //调用keyby后返回的是keyedStream
        //调用keyby算子，keyby算子和父算子不再是一一对应，而是多对多
        //key值相同的事件会被分配至同一个keyby子算子，这个案例中key是name
        //因为是流处理，所以name相同的事件会先后到达同一个keyby子算子
        //因为key相同的事件会被分到同一个keyby子算子，所以我们可以对key相同的所有事件进行聚合操作
        //但因为是流处理，所以聚合的过程和批处理不太一样，聚合的中间结果应该是存在状态中，但这是比较初级的状态
        //从当前文章开始，这啥第一个涉及到有状态流的例子。尽管状态的处理是透明的，Flink 必须跟踪每个不同的键的最大时长。
        //只要应用中有状态，你就应该考虑状态的大小。如果键值的数量是无限的，那 Flink 的状态需要的空间也同样是无限的。
        //在流处理场景中，考虑有限窗口的聚合往往比整个流聚合更有意义。
        return value.name;
    }
}).max("age").print();
```



### 有状态的Transformations

#### Flink 为什么要参与状态管理？

> 在 Flink 不参与管理状态的情况下，你的应用也可以使用状态，但 Flink 为其管理状态提供了一些引人注目的特性：

> **本地性**: Flink 状态是存储在使用它的机器本地的，并且可以以内存访问速度来获取
>
> **持久性**: Flink 状态是容错的，例如，它可以自动按一定的时间间隔产生 checkpoint，并且在任务失败后进行恢复
>
> **纵向可扩展性**: Flink 状态可以存储在集成的 RocksDB 实例中，这种方式下可以通过增加本地磁盘来扩展空间
>
> **横向可扩展性**: Flink 状态可以随着集群的扩缩容重新分布
>
> **可查询性**: Flink 状态可以通过使用 [状态查询 API](https://ci.apache.org/projects/flink/flink-docs-release-1.12/zh/dev/stream/state/queryable_state.html) 从外部进行查询。



#### Rich Functions

> 我们之前接触过`FilterFunction`， `MapFunction`，和 `FlatMapFunction`。这些Function和状态都没什么关系，不能创建或使用状态

> 对其中的每一个接口，Flink 同样提供了一个所谓 “rich” 的变体，如 `RichFlatMapFunction`，其中增加了以下方法，包括：

- `open(Configuration c)`
- `close()`
- `getRuntimeContext()`

> `open()` 仅在算子初始化时调用一次。可以用来加载一些静态数据，或者建立外部服务的链接等。

> `getRuntimeContext()` 返回当前UDF的runtime context，最明显的，它是你创建和访问 Flink 状态的途径。

> 从此以后，这些Function就可以和状态扯上关系



### 一个使用 Keyed State 的例子

> 这里`FlatMap`算子采用的是`RichFlatMapFunction`，用的是`keyed state`。然后`FlatMap`算子在从父算子的`keyed Stream`中拉取数据时就能创建并使用状态了

``` java
@Test
public void keyedState() throws Exception {
    StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
    DataStream<Person> ds =  env.fromElements(new Person("bighu",18),
                                              new Person("big1",17),
                                              new Person("big2",19),
                                              new Person("big1",20),
                                              new Person("big2",17),
                                              new Person("big1",19),
                                              new Person("big2",20));

    ds.keyBy(new KeySelector<Person, Object>() {
        @Override
        public Object getKey(Person value) throws Exception {
            return value.name;
        }
    }).flatMap(new RichFlat()).print();

    env.execute();

}

//继承RichFlatMapFunction并实现其中未实现的方法
public static class RichFlat extends RichFlatMapFunction<Person,Person>{
    //使用keyed state中的value state
    //keyed Stream中key相同的所有事件共用一个value
    //由这个泛型得知，key相同的所有事件共用一个boolean类型的值当作状态

    //keyHasBeenSeen在这里是一个局部变量
    ValueState<Boolean> keyHasBeenSeen;

    @Override
    public void open(Configuration parameters) throws Exception {
        ValueStateDescriptor<Boolean> desc = new ValueStateDescriptor<Boolean>("keyHasBeenSeen", Types.BOOLEAN());
        //keyHasBeenSeen是状态的名字，Types.BOOLEAN()决定了状态对象是如何被序列化的
        
        keyHasBeenSeen = getRuntimeContext().getState(desc); //获得当前被flatMap算子处理的事件的状态
        //如果之前没出现过key为1的事件，这时flatmap算子在keyd Stream上发现了个key为1的事件，这时flatmap算子在父算子，也就是keyBy算子上新建一个key为1对应的状态，并将状态的值获取到flatmap算子中
    }

    @Override
    public void flatMap(Person value, Collector<Person> out) throws Exception {
        if (keyHasBeenSeen.value()==null){
            out.collect(value);
        }
        keyHasBeenSeen.update(true);
        //将储存在keyBy算子中的该事件对应的状态更新为true
    }
}

```



### 清理状态

> 当我们处理的流是无界流，而且一直有新的key出现，就会导致flink维护的状态越来越多。占的内存越来越大。
>
> 所以清除再也不会使用的状态是很必要的。这通过在状态对象上调用 `clear()` 来实现`keyHasBeenSeen.clear()`

> 对一个给定的键值，你也许想在它一段时间不使用后来做这件事。当学习 `ProcessFunction` 的相关章节时，你将看到在事件驱动的应用中怎么用定时器来做这个。

> 也可以选择使用 [状态的过期时间（TTL）](https://ci.apache.org/projects/flink/flink-docs-release-1.12/zh/dev/stream/state/state.html#state-time-to-live-ttl)，为状态描述符配置你想要旧状态自动被清除的时间。



## Connected Streams

> 相比于下面这种预先定义的转换：

![simple transformation](F:\学习笔记\flink\imgs\transformation.svg)

> 有时你想要更灵活地调整转换的某些功能，比如数据流的阈值、规则或者其他参数。Flink 支持这种需求的模式称为 *connected streams* ，一个单独的算子有两个输入流。

![connected streams](F:\学习笔记\flink\imgs\connected-streams.svg)

> connected stream 也可以被用来实现流的关联。

> 连接两个流。keyed stream 被连接时，他们必须按相同的方式分区
>
> ControlFunction继承于RichCoFlatMapFunction
>
> 连接流的作用和具体实现还不太清楚，
>
> 需要的时候看https://ci.apache.org/projects/flink/flink-docs-release-1.12/zh/learn-flink/etl.html

``` java
public static void main(String[] args) throws Exception {
    StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

    DataStream<String> control = env.fromElements("DROP", "IGNORE").keyBy(x -> x);
    DataStream<String> streamOfWords = env.fromElements("Apache", "DROP", "Flink", "IGNORE").keyBy(x -> x);
  
    control
        .connect(datastreamOfWords)
        .flatMap(new ControlFunction())
        .print();

    env.execute();
}
```

