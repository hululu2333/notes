### Zookeeper的安装与配置

***

#### 解压Zookeeper

``` bash 
tar -zxvf zookeeper-3.4.10.tar.gz -C ../module/
```



#### 配置Zookeeper

##### 设置myid

> 来到zookeeper的家目录，创建名为zkData的文件夹。
> 新建一个名为myid的文件，填入一个数字，不同节点的值不一样。hadoop01的值设为1

***

##### 修改zoo.cfg文件

```bash
#来到zookeeper的conf目录下，发现有一个文件叫做zoo_sample.cfg。将其改名为zoo.cfg
#开始编辑zoo.cfg

#zoo.cfg一开始有以下配置，我们将dataDir的值修改为/opt/module/zookeeper/zkData

#一个tick为2000毫秒，tick指的是通一次信花的时间
tickTime=2000

#Zookeeper集群刚启动时，最多花10个tick的时间来进行同步
initLimit=10

#从发送一个请求到收到回复，最多等5个tick的时间。超过了这个时间就认为请求失败
syncLimit=5

#修改数据的存放目录，默认为/tmp/zookeeper
dataDir=/opt/module/zookeeper/zkData

#给客户端留的端口
clientPort=2181
```

```bash
#除此之外，添加以下几行配置
#写上集群中所有的Zookeeper的地址信息
#server.1是Zookeeper的名称，数字最好和之前创建的myid中的数字对应。这个数字是各个Zookeeper之间辨识的依据
#hadoop01:2888:3888分别是ip地址，Zookeeper之间通讯的端口号和选举端口号。哪个Zookeeper是leader不是我们指定的，而是这几个Zookeeper之间自己选举的，他们就是通过选举端口来确定谁是leader

server.1=hadoop01:2888:3888
server.2=hadoop02:2888:3888
server.3=hadoop03:2888:3888
```

***

##### 修改Zookeeper日志的存放路径

```bash
#编辑zkEnv.sh
vi /opt/module/zookeeper/bin/zkEnv.sh

#修改ZOO_LOG_DIR的值
ZOO_LOG_DIR="/opt/module/zookeeper/logs"
```



#### 启动Zookeeper

``` bash
#启动Zookeeper。Zookeeper没有一键启动，其他主机上的要自己去启动
/opt/module/zookeeper/bin/zkServer.sh start

#查看Zookeeper的运行状态
/opt/module/zookeeper/bin/zkServer.sh status
```

