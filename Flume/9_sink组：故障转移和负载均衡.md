### sink组：故障转移和负载均衡

***

> flume中有sink组的概念，当一个sink绑定一个channel时，这个sink就是一个默认的sink组。
>
> 当多个sink绑定一个channel时，我们就要显示定义一个sink组了。sink组分为failover和load_balance



#### 故障转移

> 多个sink绑定一个channel，为sink设置不同的优先级。channel中的Evnet全由优先级最高的sink拿去，当优先级最高的sink无法将Evnent成功发送至目的地时，这时弃用优先级最高的，使用优先级第二的

``` bash
#指定组件
a1.sources = r1
a1.sinks = k1 k2
a1.channels = c1
            
# Describe/configure the source
#指定名为r1的source的一些配置，如果还有一个source的话，也要配置那一个
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 4444
      
# Describe the sink
# 配置sink
a1.sinks.k1.type = avro
a1.sinks.k1.hostname = hadoop01
a1.sinks.k1.port = 523

a1.sinks.k2.type = avro
a1.sinks.k2.hostname = hadoop01
a1.sinks.k2.port = 524

# 配置sink组
a1.sinkgroups = g1
a1.sinkgroups.g1.sinks = k1 k2
a1.sinkgroups.g1.processor.type = failover
a1.sinkgroups.g1.processor.priority.k1 = 5
a1.sinkgroups.g1.processor.priority.k2 = 10
a1.sinkgroups.g1.processor.maxpenalty = 10000

# Use a channel with buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
#把source和sink绑定在特定的channel上 
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
a1.sinks.k2.channel = c1




#agent的一个source，可以有多个
a2.sources = r1
a2.sinks = k1
a2.channels = c1

# Describe/configure the source
#指定名为r1的source的一些配置，如果还有一个source的话，也要配置那一个
a2.sources.r1.type = avro
a2.sources.r1.bind = hadoop01
a2.sources.r1.port = 523

# Describe the sink
#指定sink的类型
a2.sinks.k1.type = logger

# Use a channel with buffers events in memory
a2.channels.c1.type = memory
a2.channels.c1.capacity = 1000
a2.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
#把source和sink绑定在特定的channel上 
a2.sources.r1.channels = c1
a2.sinks.k1.channel = c1 

```

> a3和a2只有avro source的端口不同。
>
> a1中优先级高的k2对接的是a3，当a3存活时，channel中的数据全都发到a3中去了，当我把a3停止，k2无法将数据发送出去，这时启用k1



#### 负载均衡

> 负载均衡指将channel中的数据随机分给sink组中的sink

> 相比于上面的代码，负载均衡只需改变sink组的配置

``` bash
a1.sinkgroups = g1
a1.sinkgroups.g1.sinks = k1 k2
a1.sinkgroups.g1.processor.type = load_balance
a1.sinkgroups.g1.processor.backoff = true
a1.sinkgroups.g1.processor.selector = random
```

