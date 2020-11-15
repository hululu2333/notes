### 启动hadoop集群

***

#### 启动hdfs集群

``` bash
#第一次启动hdfs集群时，要初始化namenode
/opt/module/hadoop/bin/hdfs namenode -format

#一键启动hdfs集群。本质是根据配置文件中的配置，远程登录到目标主机上，挨个启动hdfs服务
/opt/module/hadoop/sbin/start-dfs.sh
```



#### 启动yarn集群

``` bash
#一键启动yarn集群，和启动hdfs一样，具体在哪台主机启动哪个服务，是根据配置文件中的配置
/opt/module/hadoop.sbin/start-yarn.sh
```



#### 启动历史作业服务器

``` bash
#具体在哪台主机上启动历史服务器也是由配置文件决定
/opt/module/hadoop/sbin/mr-jobhistory-daemon.sh start historyserver
```

