### 使用Flume监听端口并收集信息

***

#### 大概流程

> 2. 启动一个Agent监控某个端口
> 3. 如果有数据被发送到这个端口，Agent中的Source就将数据捕获，放进Channel
> 4. Sink从Channel中拿数据出来，放进目的地



#### 实现步骤

##### 安装netcat工具

``` bash
#netcat俗称瑞士军刀，可以通过使用tcp或udp的网络去读写数据
sudo yum install -y nc

#演示nc的用法
nc -lk 052343 #开启服务端，在hadoop01上执行
nc hadoop01 052343 #开启客户端，连接上服务端。在其他主机上执行
#这时客户端和服务端就可以互相通信了
```

***

##### 判断052343这个端口是否被占用

```bash
sudo netstat -tunlp | grep 052343
```

***

##### 创建Agent的配置文件——flume-netcat-logger.conf

> 当你启动Agent的时候，会读取这个配置文件里的配置
>
> 这个配置文件的文件名可以自定义

``` bash
#在Flume的家目录创建job文件夹，用来放Agent的配置文件
mkdir job

#文件名可以自己取，尽量取一个能体现此次任务特点的名字
touch netcat-flume-logger.conf
```

> ``` bash
> #编辑配置文件，加入以下内容
> 
> #为Agent的各个组件命名
> #a1指的是当前Agent的名称，当一台主机上同时运行多个Agent时，这就是区分不同Agent的标识
> #r1是当前Agent的一个source，可以有多个
> a1.sources = r1
> a1.sinks = k1
> a1.channels = c1
> 
> #为名为r1的source写一些配置信息，如果还有一个source的话，也要配置那一个
> a1.sources.r1.type = netcat
> a1.sources.r1.bind = localhost
> a1.sources.r1.port = 052343
> 
> #为名为k1的sink写配置
> a1.sinks.k1.type = logger
> 
> #指定名为c1的channel的一些属性
> a1.channels.c1.type = memory
> a1.channels.c1.capacity = 1000 #设置c1的容量为1000个事件
> a1.channels.c1.transactionCapacity = 100 #设置一次传输的数据量是100个事件
> 
> #把source和sink绑定在特定的channel上
> #绑定的时候，不能在句尾接注释，不然可能会出错！！！虽然不知道为什么
> #我猜测是解析配置文件的程序有问题，以后在linux中写配置文件，注释和代码不要放在一行
> a1.sources.r1.channels = c1
> a1.sinks.k1.channel = c1 
> #此处为channel和不是channels，说明一个sink只能绑定一个channel
> 
> 
> ```
>

***

##### 启动agent

``` bash
#切换到flume的安装目录，执行以下命令
#第一个参数是conf目录的位置，第二个参数是我们自己写的配置文件的位置。
#第三个参数是这个agent的名字，与我们自己写的配置文件里的相对应
#第四个是logger-sink特殊的参数，确定取出数据后的存放位置，这里选的是控制台。INFO指的是日志的一个级别，除INFO外，还有debug,error等
bin/flume-ng agent --conf conf --conf-file job/netcat-flume-logger.conf --name a1 -Dflume.root.logger=INFO,console

#agent启动之后，根据配置文件中sources的配置得知。我们在localhost的052343端口启动了netcat的服务端，如果这个端口出现数据，r1会将其捕获并加入a1中。
```

```bash
#第二种启动方式，这种更简便
bin/flume-ng agent -c conf -f job/netcat-flume-logger.conf -n a1 -Dflume.root.logger=INFO,console
```

