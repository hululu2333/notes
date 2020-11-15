![agent运行失败](C:\Users\hu\Desktop\agent运行失败.png)

> 最近在启动agent的时候总是出现这样的情况，一开始觉得SLF4J后面的只是日志信息。
>
> 所以总觉得是自己配置出错了，试验了多次，还是这样。
>
> 最后将SLF4J后的信息放在网上搜索，发现是jar包重复了
>
> 现在我们删除/opt/module/flume/lib/slf4j-log4j12-1.6.1.jar
>



> 还一个问题，在使用file_roll sink时，报Directory may not be null的错
>
> 后来仔细查看，原来是a1.sinks.k1.sink.directory写成了a1.sinks.k1.directory 



> 这次做的是一个channel副本的案例
>
> 即a1中的taildir source采集数据，接着发到两个channel中。然后两个avro sink将channel中的数据分别发送到a2和a3中
>
> a2将数据存到hdfs，a3将数据存到本地文件系统