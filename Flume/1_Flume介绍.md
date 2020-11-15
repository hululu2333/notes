### Flume介绍

***

#### Flume的作用

> Flume是一个日志的传输工具
>
> 最主要的作用是实时的读取服务器本地磁盘的数据，并将其写入到hdfs中



#### 为什么用Flume

> 用户在前端页面浏览时，会源源不断的产生用户行为日志。我们要在最短的时间里分析这些日志，以便及时给用户推荐相关产品。
>
> Flume可以做到实时的将服务器产生的数据采集并发送到hdfs或kafka
>
> Flume的数据源可以是服务器的文件夹，也可以是服务器的网络端口



#### Flume的组成

![image-20200619103858141](F:\学习笔记\Flume\imgs\Flume组成.png)

##### Agent

> Flume的功能是由Agent来实现的。Agent是一个JVM进程，以事件的形式将数据从源头送至目的地。Agent分为三个部份，分别是Source，Channel，Sink
>
> Agent相当于是Flume的一个具体实现

***

##### Source

> Source是负责接收数据的组件，它可以接收各种类型的日志数据。Source的数据来源可以是服务器的文件夹，可以是服务器的网络端口，也可以自定义

***

##### Sink

> Sink不断的询问Channel中还有多少事件，并将这些事件批量的送到目的地。这目的地可以是储存系统，也可以是索引系统，或者是另一个Agent

***

##### Channel

> Channel是位于Source和Sink之间的缓冲，当Source生产数据的速度大于Sink消费数据的速度时，可以保证Agent的正常运转
>
> Channel有Memory Channel，File Channel和Kafka Channel这些形式。Memory Channel是以内存做为缓冲，速度快但是不安全，系统宕机或断电后会丢失数据

***

##### Event

> Event是Flume进行数据传输的基本单元，传输前将日志数据封装成一个Event
>
> Event由Header和Body组成。Header保存了该Event的一些属性，是键值对的形式
>
> Body中保存的是这条数据的信息，以字节数组的形式