### 安装并使用Eagle

***

> Eagle的版本为1.3.7



#### 修改kafka的启动脚本

``` bash
vi kafka-server-start.sh
```

``` sh
#修改以下if语句中的内容
if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then
fi

#改为以下内容
if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then 
export KAFKA_HEAP_OPTS="-server -Xms2G -Xmx2G -XX:PermSize=128m -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:ParallelGCThreads=8 XX:ConcGCThreads=5 -XX:InitiatingHeapOccupancyPercent=70"     
export JMX_PORT="9999"     
#export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G" 
fi
```



#### 将修改后的启动脚本同步到所有节点

``` bash
xsync kafka-server-start.sh
```



#### 解压eagle压缩包

``` bash
tar -zxvf kafka-eagle-bin-1.3.7.tar.gz 
#解压后出现kafka-eagle-web-1.3.7-bin.tar.gz

#再对kafka-eagle-web-1.3.7-bin.tar.gz进行解压
tar -zxvf kafka-eagle-web-1.3.7-bin.tar.gz -C /opt/module/
```



#### 配置环境变量

> vi /etc/profile.d/env.sh	//所有节点都要配置

``` sh
export KE_HOME=/opt/module/eagle
export PATH=$PATH:$KE_HOME/bin
```



#### 修改eagle的配置文件

> vi /opt/module/conf/system-config.properties
>
> **集群中还没有安装mysql，Eagle暂时就不用了。以下的配置文件还没有配置完全**

``` sh
######################################
# multi zookeeper&kafka cluster list
######################################
kafka.eagle.zk.cluster.alias=cluster1
cluster1.zk.list=hadoop01:2181,hadoop02:2181,hadoop03:2181

######################################
# zk client thread limit
######################################
kafka.zk.limit.size=25

######################################
# kafka eagle webui port
######################################
kafka.eagle.webui.port=8048

######################################
# kafka offset storage
######################################
cluster1.kafka.eagle.offset.storage=kafka

######################################
# enable kafka metrics
######################################
kafka.eagle.metrics.charts=true
kafka.eagle.sql.fix.error=false
# kafka sql topic records max
######################################
kafka.eagle.sql.topic.records.max=5000

######################################
# alarm email configure
######################################
kafka.eagle.mail.enable=false
kafka.eagle.mail.sa=alert_sa@163.com
kafka.eagle.mail.username=alert_sa@163.com
kafka.eagle.mail.password=mqslimczkdqabbbh
kafka.eagle.mail.server.host=smtp.163.com
kafka.eagle.mail.server.port=25

######################################
# alarm im configure
######################################

#kafka.eagle.im.wechat.enable=true
#kafka.eagle.im.wechat.toparty=
#kafka.eagle.im.wechat.totag=
#kafka.eagle.im.wechat.agentid=

######################################
# delete kafka topic token
######################################
kafka.eagle.topic.token=keadmin

######################################
# kafka sasl authenticate
######################################
cluster1.kafka.eagle.sasl.enable=false
cluster1.kafka.eagle.sasl.protocol=SASL_PLAINTEXT
cluster1.kafka.eagle.sasl.mechanism=PLAIN
cluster1.kafka.eagle.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="admin" password="kafka-eagle";

cluster2.kafka.eagle.sasl.enable=false
cluster2.kafka.eagle.sasl.protocol=SASL_PLAINTEXT
cluster2.kafka.eagle.sasl.mechanism=PLAIN
cluster2.kafka.eagle.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="admin" password="kafka-eagle";

######################################
# kafka jdbc driver address
######################################
kafka.eagle.driver=com.mysql.jdbc.Driver
kafka.eagle.url=jdbc:mysql:/hadoop01:3306/kafka-eagle/db/ke.db
kafka.eagle.username=root
kafka.eagle.password=www.kafka-eagle.org

```



