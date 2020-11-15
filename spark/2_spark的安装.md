### Spark的安装

***

#### Local模式

> Local模式是spark的一种运行模式，指在一台计算机上运行
>
> 我们可以设为local，即所有计算运行在一个线程中
>
> 我们也可以设为local[K]，如local[3]指的是用三个线程来运行计算
>
> local[*]，你的处理器有多少核，就用多少线程来计算



#### 下载spark2.1.1

> 进入download界面，拉到最下面

<img src="F:\学习笔记\spark\images\spark下载1.png" alt="image-20200926110241724" style="zoom: 33%;" />



> 我的hadoop是2.7版本的，所以选择hadoop为2.7的spark2.1.1

<img src="F:\学习笔记\spark\images\选择spark版本.png" alt="image-20200926110607577" style="zoom:33%;" />



#### 安装spark

> 1.先将压缩包通winscp传到linux中

> 2.解压缩安装包，并修改文件夹名称

``` bash
tar -zxvf spark-2.1.1-bin-hadoop2.7.tgz -C ../module
mv spark-2.1.1-bin-hadoop2.7 spark
```

> 3.修改目录的所有者为hadoop，-R表示该目录下的所有子文件和子目录都被修改

``` bash
sudo chown -R hadoop:hadoop spark
```

> 4.测试spark，切换到spark安装目录。每行结尾的\表示命令还没打完，换行继续打

``` bash
bin/spark-submit --class org.apache.spark.examples.SparkPi \
--executor-memory 1G \
--total-executor-cores 2 \
./examples/jars/spark-examples_2.11-2.1.1.jar \
100

```

> 5.启动spark的交互式界面，即spark shell，也就是spark的命令行窗口

``` bash
bin/spark-shell
```



![image-20200926135835314](F:\学习笔记\spark\images\spark-shell.png)

> 这三行分别是spark-shell的图形用户化界面的地址、context对象和session对象
>
> context对象和session对象都是可以直接拿来用的