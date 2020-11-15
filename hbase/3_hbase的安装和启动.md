### hbase的安装和启动

***

> hbase是依赖于zookeeper和hdfs的，所以这两个服务要先启动



#### 解压缩hbase

``` bash
# 解压缩hbase
tar -zxvf /opt/software/hbase-2.3.2-bin.tar.gz -C ../module

# 改文件夹名
mv hbase-2.3.2/ hbase
```



#### 修改hbase的配置文件

``` bash
# regionservers类似于hadoop中的slaves，添加到regionservers中的ip地址，在一键启动时，会启动regionserver服务
vi regionservers
# 将文件内容替换为以下部分
```

``` xml
hadoop01
hadoop02
hadoop03
```



``` bash
# 修改hbase-env.sh
vi hbase-env.sh
# hbase-env.sh中本身有内容，我们修改其中的一些参数
```

``` bash
# 将这行的注释去掉，并指定正确的jdk路径
export JAVA_HOME=/opt/module/jdk

# 将这行的注释去掉，并修改为false，表示我们不用habse自带的zookeeper
export HBASE_MANAGES_ZK=false
```



``` bash
# 修改hbase-site.xml
vi hbase-site.xml
# 将文件内容替换为以下部分
```

``` xml
<configuration>
    <!-- 设置HBase储存的根目录 -->
    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://hadoop01:9000/hBase</value>
    </property>
    
    <!-- 将HBase设置为分布式模式 -->
    <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
    </property>
    
    <!-- 将Master的端口设置为16000，其实默认端口就是16000。16010是master的ui端口 -->
    <property>
        <name>hbase.master.port</name>
        <value>16000</value>
    </property>
    
    <!-- 配置zookeeper集群 -->
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>hadoop01,hadoop02,hadoop03</value>
    </property>
    
    <!-- 配置储存在zookeeper中的数据的位置 -->
    <property>
        <name>hbase.zookeeper.property.dataDir</name>
        <value>/opt/module/zookeeper/zkData</value>
    </property>
</configuration>
```



``` bash
# hbase是依赖hdfs的，hbase的conf目录下需要有一个core-site.xml。建议创建一个软链接
ln -s /opt/module/hadoop/etc/hadoop/core-site.xml /opt/module/hbase/conf/core-site.xml
```



``` bash
# 分发hbase文件夹
~/bin/xsync /opt/module/hbase
```



#### 启动hbase集群

``` bash
# 启动hbase的主节点
hbase-deamon.sh start master

# 启动hbase的从节点，reginmaster所在的节点和master所在的节点，时间差不能超过30秒。不然会出错
hbase-daemon.sh start reginserver

#我们也可以通过start-hbase.sh或stop-hbase.sh进行一键启动或关闭集群。使用stop-hbase.sh时确保master节点已经被启动，不然这个命令会执行失败
```

