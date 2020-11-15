### 使用idea开发WordCount

***

> 我们用scala编写spark应用时，要导入一些spark依赖。不同版本的spark对scala的版本是有一些限制的。
>
> 我们现在用的spark是2.1.1，scala建议使用2.11版本。
>
> 一开始我scala使用的是2.13版本，结果是找不到对应的spark依赖 



#### 配置idea的开发环境

1. 添加scala语言的支持

   <img src="C:\Users\hu\AppData\Roaming\Typora\typora-user-images\image-20201002104719669.png" alt="image-20201002104719669" style="zoom: 50%;" />



2. 添加spark的依赖

   ``` xml
   <properties>
   	<spark.version>2.1.1</spark.version>
       <scala.version>2.11</scala.version>
   </properties>
   
   <dependency>
   	<groupId>org.apache.spark</groupId>
   	<artifactId>spark-core_${scala.version}</artifactId>
   	<version>${spark.version}</version>
   </dependency>
   ```

   

3. 创建scala目录，并将其标记为源目录

   ![image-20201002122524820](C:\Users\hu\AppData\Roaming\Typora\typora-user-images\image-20201002122524820.png)



#### 开始编写WordCount逻辑

``` scala
object WordCount {
  def main(args: Array[String]): Unit = {

    //创建SparkConf对象，并设置运行模式为local[*]，appid为WordCount
    //调用.setMaster方法后，返回的是一个修改过的SparkCong对象，所以后面还能接着点
    val config : SparkConf = new SparkConf().setMaster("local[*]").setAppName("WordCount")

    //创建spark的上下文对象
    //在spark的交互式界面，这个sc对象是你启动spark-shell时自动创建的
    val sc = new SparkContext(config)


    //开始WordCount的编写
    //将文件一行行的读取出来
    val lines: RDD[String] = sc.textFile("in/test.txt")

    //将一行行的文件分解为一个个的单词
    val words: RDD[String] = lines.flatMap(_.split(" "))

    //将一个个单词改为key为单词，value为1的元组
    val wordToOne: RDD[(String,Int)] = words.map((_, 1))

    //对key值相同的元组进行分组聚合
    val wordToSum: RDD[(String,Int)] = wordToOne.reduceByKey(_ + _)

    //将统计结果采集后打印到控制台
    val result: Array[(String,Int)] = wordToSum.collect()

    result.foreach(println)

  }
}
```



#### 对Yarn进行一些配置

> 修改yarn-site.xml，添加以下内容

``` xml 
<!-- 是否启动一个线程检查每个任务的正在使用的物理内存量，如果任务超出分配值，则将其直接杀掉，默认为true -->
<property>
    <name>yarn.nodemanager.pmem-check-enable</name>
    <value>false</value>
</property>

<!-- 是否启动一个线程检查每个任务的正在使用的物理内存量，如果任务超出分配值，则将其直接杀掉，默认为true -->
<property>
    <name>yarn.nodemanager.vmem-check-enable</name>
    <value>false</value>
</property>
```



#### 修改spark的配置文件

> 修改spark-env.sh
>
> 在conf目录下有个叫spark-env.sh.template的文件，将其改为spark-env.sh。并在其中添加以下内容

``` bash
YARN_CONF_DIR=/opt/module/hadoop/etc/hadoop
```



#### 将修改的这两个文件同步到其他主机上

> xsync是自己编写的同步脚本

``` bash
~/bin/xsync /opt/module/spark
~/bin/xsync /opt/module/hadoop/etc/hadoop/yarn-site.xml
```



#### 启动Yarn模式的spark-shell

> 在以上的配置都完成，且Yarn集群已开启的情况下，我们就可以让spark基于Yarn来运行了

``` bash
//开启基于Yarn的spark交互界面
spark-shell --master yarn
```



#### 通过Yarn查看spark的运行日志

> 使用mapreduce计算框架时，Resource会创建一个ApplicationMaster来监控一个job，job的运行详情都会汇总到ResourceManager中
>
> 但使用spark计算框架时，Yarn只提供资源管理，无法查看spark的运行详情
>
> 我们可以通过修改spark的配置文件spark-defaults.conf来达到这个功能，在spark-defaults.conf中添加以下内容

``` conf
spark.yarn.historyServer.address=hadoop02:18080
spark.history.ui.port=18080
```

***

> 重启spark的历史服务

``` bash
/opt/module/spark/sbin/stop-history-server.sh
/opt/module/spark/sbin/start-history-server.sh
```



#### 基于Yarn的spark的运行流程

> 这个流程和基于Yarn的MapReduce的运行流程有点类似

> 1.spark客户端向ResourceManager提交作业请求（在spark-shell中运行计算pi的案例）
>
> 2.ResourceManager在某个节点上创建一个container,并让这个节点上NodeManager启动一个ApplicationMaster
>
> 3.ApplicationMaster向ResourceManager申请资源
>
> 4.ResourceManager返回资源列表（目前可用的节点）
>
> 5.Application选择一个节点，并在这个节点的Container中创建一个executer
>
> 6.反向注册，executer创建完成后通知ApplicationMaster
>
> 7.分解并调度任务，ApplicationMaster将任务分解，并发送给executer运行