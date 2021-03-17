### **相关概念**

> 引用：https://blog.csdn.net/qq_35440040/article/details/82462269

1.Metadata概念：

元数据包含用Hive创建的database、table等的元信息。元数据存储在关系型数据库中。如Derby、MySQL等。

2.Metastore作用：

**客户端连接metastore服务，metastore再去连接MySQL数据库来存取元数据。有了metastore服务，就可以有多个客户端同时连接，而且这些客户端不需要知道MySQL数据库的用户名和密码，只需要连接metastore 服务即可。** 

3.Metastore 有3中开启方式:
  --1-->默认开启方式:
    没有配置metaStore的时候,每当开启bin/hive;或者开启hiveServer2的时候,都会在内部启动一个metastore
      嵌入式服务;资源比较浪费,如果开启多个窗口,就会存在多个metastore server。（）
      
  --2-->local mataStore(本地)
    当metaStore和装载元数据的数据库(MySQL)存在同一机器上时配置是此模式,
    开启metastore服务就只需要开启一次就好,避免资源浪费!
    
  --3-->Remote Metastore(远程)
    当metaStore和装载元数据的数据库(MySQL)不存在同一机器上时配置是此模式,
    开启metastore服务就只需要开启一次就好,避免资源浪费!

### Metastore三种配置方式

由于元数据不断地修改、更新，所以Hive元数据不适合存储在HDFS中，一般存在RDBMS中。

1、内嵌模式（Embedded）

- **hive服务和metastore服务运行在同一个进程中**，derby服务也运行在该进程中.内嵌模式使用的是内嵌的Derby数据库来存储元数据，也不需要额外起Metastore服务。
- 这个是默认的，配置简单，但是一次只能一个客户端连接（这句话说实在有点坑，其实就是你启动一个hive服务会内嵌一个metastore服务，然后在启动一个又会内嵌一个metastore服务，并不是说你的客户端只能启动一个hive，是能启动多个，但是每个都有metastore，浪费资源），适用于用来实验，不适用于生产环境。

2、本地模式（Local）:本地安装mysql 替代derby存储元数据

- 不再使用内嵌的Derby作为元数据的存储介质，而是使用其他数据库比如MySQL来存储元数据。**hive服务和metastore服务运行在同一个进程中**，mysql是单独的进程，**可以同一台机器，也可以在远程机器上**。（我之前有种方式是：只在接口机配置hive，并配置mysql数据库，用户和密码等；但是集群不配置hive，不起hive任何服务，就属于这种情况）
- 这种方式是一个多用户的模式，运行多个用户client连接到一个数据库中。这种方式一般作为公司内部同时使用Hive。每一个用户必须要有对MySQL的访问权利，即每一个客户端使用者需要知道MySQL的用户名和密码才行。
- 服务端配置如下：　

```html
<?xml version="1.0"?>



<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>



 



<configuration>



<property>



  <name>hive.metastore.warehouse.dir</name>



  <value>/user/hive_remote/warehouse</value>



</property>



 



<property>



  <name>hive.metastore.local</name>



  <value>true</value>



</property>



 



<property>



  <name>javax.jdo.option.ConnectionURL</name>



  <value>jdbc:mysql://localhost/hive_remote?createDatabaseIfNotExist=true</value>



</property>



 或者 jdbc:mysql://ip:3306/hive?characterEncoding=UTF-8（推荐这个。不会web，不然两个参数都给加上。。。。）



 



<property>



  <name>javax.jdo.option.ConnectionDriverName</name>



  <value>com.mysql.jdbc.Driver</value>



</property>



 



<property>



  <name>javax.jdo.option.ConnectionUserName</name>



  <value>hive</value>



</property>



 



<property>



  <name>javax.jdo.option.ConnectionPassword</name>



  <value>password</value>



</property>



</configuration>
```

3、远程模式（Remote）: 远程安装mysql 替代derby存储元数据

- **Hive服务和metastore在不同的进程内**，可能是不同的机器（集群实例来说metastore有3个![img](https://img-blog.csdn.net/20180906160507721?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NDQwMDQw/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)，启动在三台机器，但是都是指向的一台主机的mysql，当然mysql也能配置成主备集群的模式），该模式需要将hive.metastore.local设置为false（但是在0.10 ，0.11或者之后的HIVE版本 hive.metastore.local 属性不再使用。），将hive.metastore.uris设置为metastore服务器URL，
- 远程元存储需要单独起metastore服务，然后每个客户端都在配置文件里配置连接到该metastore服务。将metadata作为一个单独的服务进行启动。各种客户端通过beeline来连接，连接之前无需知道数据库的密码。
- 仅连接远程的mysql并不能称之为“远程模式”，是否远程指的是metastore和hive服务是否在同一进程内.
- 服务端配置如下：

```html
<?xml version="1.0"?>



<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>



 



<configuration>



 



<property>



  <name>hive.metastore.warehouse.dir</name>



  <value>/user/hive/warehouse</value>



</property>



 



<property>



  <name>javax.jdo.option.ConnectionURL</name>



  <value>jdbc:mysql://192.168.1.214:3306/hive_remote?createDatabaseIfNotExist=true</value>



</property>



 



<property>



  <name>javax.jdo.option.ConnectionDriverName</name>



  <value>com.mysql.jdbc.Driver</value>



</property>



 



<property>



  <name>javax.jdo.option.ConnectionUserName</name>



  <value>hive</value>



</property>



 



<property>



  <name>javax.jdo.option.ConnectionPassword</name>



  <value>password</value>



</property>



 



<property>



  <name>hive.metastore.local</name>



  <value>false</value>



</property>



但是在0.10 ，0.11或者之后的HIVE版本 hive.metastore.local 属性不再使用。



 



<property>



  <name>hive.metastore.uris</name>



  <value>thrift://192.168.1.188:9083</value>



</property>



 



</configuration>
```

附：远程模式下，hive客户端配置：

**（1）hive-site.xml**

<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<property>
 <name>hive.metastore.uris</name>
 <value>thrift://hive服务端ip:9083</value>
</property>
</configuration>

（2）配置日志存放位置：配置文件为hive-log4j.properties.template

修改如下：

![img](https://img-blog.csdn.net/20180906164346251?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NDQwMDQw/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

hive.log.dir=/opt/beh/log/hive

或者修改为：

hive.log.dir=/opt/beh/logs/hive/${user.name} 

这样对于同一台机器不同的用户可以把日志动态写入各自用户目录下，前提是/opt/beh/log/hive权限足够：rwx

**重点：**hive-log4j.properties.template要修改为hive-log4j.properties不然无法识别。

（3）服务端metastore 启动方式：

第一种：hive --service metastore -p 9083 &

第二种：在hive-site.xml中配置hive.metastore.uris，指定端口，然后直接 hive --service metastore

**（4）对比测试：**（客户端主机2*6C，128Gmem）

=======我之前是本地模式，集群不配置hive，不启动hive metastore，直接在客户端配置链接数据库，使用用户名和密码，然后启动hive执行命令查看如下：

简单的showdatabase 执行时间为：3.29s ，费时

 

![img](https://img-blog.csdn.net/20180906165651512?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NDQwMDQw/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

cpu损耗较大：178%

![img](https://img-blog.csdn.net/20180906165552816?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NDQwMDQw/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

 

=======后来配置了远程模式，在集群配置了hive，并后台运行了metastore，然后配置客户端连接metastore后，测试如下：

简单的showdatabase 执行时间为：0.8s，明显快了很多

![img](https://img-blog.csdn.net/2018090617005874?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NDQwMDQw/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

cpu损耗：74.8% 少了一倍

![img](https://img-blog.csdn.net/20180906170215146?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NDQwMDQw/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

总结：为什么集群要启用metastore，因为启用后，直接bin/hive启动进程，只会有hive交互式客户端一个服务，不会再有metastore同时存在在该进程中，资源占用率下降，查询速度更快。