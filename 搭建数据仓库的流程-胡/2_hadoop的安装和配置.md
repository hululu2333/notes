### hadoop的安装和配置——2.7.2

***

#### 集群规划

> hadoop01:	NameNode	DataNode	NodemManager	Zookeeper
>
> hadoop02:	DataNode	ResourceManager	NodeManager	Zookeeper
>
> hadoop03:	DataNode	SecondaryNameNode	NodeManager	Zookeeper



#### 软件准备

>hadoop 2.7.2
>
>jdk 8
>
>hadoop-lzo 0.4.20	用于对文件进行lzo压缩
>
>zookeeper 3.4.10



#### 安装jdk

##### 解压jdk压缩包

> 用hadoop用户对jdk安装包进行解压，这样的话jdk目录下的文件的所有者是hadoop
>
> tar -zxvf jdk*** -C /opt/module

***

##### 配置jdk环境变量

> sudo vi /etc/profile.d/env.sh	建议把环境变量设置在/etc/profile.d/env.sh中。将来远程登录的时候会比较方便（env.sh是一个新文件）
>
> export JAVA_HOME=/opt/module/jdk
>
> export PATH=$PATH:$JAVA_HOME/bin
>
> export CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar	CLASS_PATH的作用是帮助运行时的java程序找到他所需的class文件
>
> source /etc/profile.d/env.sh
>
> centos6.5会默认安装openjdk，这时我们还要把openjdk给卸载了



#### 安装hadoop

##### 解压hadoop安装包

> 同样是用hadoop用户进行解压缩
>
> tar -zxvf hadoop-2.7.2 -C ../module

***

##### 配置hadoop环境变量

> sudo vi /etc/profile.d/env.sh	同样是把环境变量写在这个文件中
>
> export HADOOP_HOME=/opt/module/hadoop
>
> export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

***

##### 修改hadoop的配置文件

> core-site.xml	添加以下内容

``` xml
<!-- 指定hdfs中namenode的地址 -->
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://hadoop01:9000</value>
</property>

<!-- 指定hadoop运行时，产生的文件的存放位置。 -->
<property>
    <name>hadoop.tmp.dir</name>
    <value>/opt/module/hadoop/data/tmp</value>
</property>
```



> hadoop-env.sh	将JAVA_HOME的值改为我们jdk的家目录

``` sh
export JAVA_HOME=/opt/module/jdk
```



> hdfs-site.xml	添加以下内容

``` xml
<!-- 设置一个数据保存几份，一般是3份。但这里资源紧缺，设为1份 -->
<property>
    <name>dfs.replication</name>
    <value>1</value>
</property>

<!-- 指定hadoop中第二名称节点的位置 -->
<property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>hadoop03:50090</value>
</property>
```



> yarn-env.sh	和hadoop-env.sh一样，配置jdk的安装路径

``` sh
export JAVA_HOME=/opt/module/jdk
```



> yarn-site.xml	添加以下内容

``` xml
<!-- 设置Reducer获取数据的方式为mapreduce_shuffle，当然也可以用其他类型的shuffle -->
<property>
	<name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>

<!-- 指定yarn中ResourceManager的地址 -->
<property>
	<name>yarn.resourcemanager.hostname</name>
    <value>hadoop02</value>
</property>

<!-- 设置日志是否能聚集起来，设置为能 -->
<property>
	<name>yarn.log-aggregation-enable</name>
    <value>true</value>
</property>

<!-- 设置日志的保存时间，设为7天 -->
<property>
	<name>yarn.log-aggregation.retain-seconds<name>
    <value>60480</value>
</property>
```



> mapred-env.sh	设置java安装目录

``` sh
export JAVA_HOME=/opt/module/jdk
```



> mapred-site.xml	将mapred-site.xml.template改名为mapred-site.xml	然后添加以下内容

``` xml
<!-- 指定mapreduce运行在哪个框架上，现指定为yarn框架 -->
<property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
</property>

<!-- 指定历史服务器服务端的地址 -->
<property>
    <name>mapreduce.jobhistory.address</name>
    <value>hadoop01:10020</value>
</property>
        
<!-- 指定历史服务器web端的地址 -->
<property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>hadoop01:19888</value>
</property>
```



> slaves	slaves文件中有一些主机名，当hdfs启动时，这些主机上的datanode都会被启动。        		       slaves文件中不能存在空行和空格

``` txt
hadoop01
hadoop02
hadoop03
```



#### 配置免密登录

##### 免密登录的原理

> 每个linux的每个用户只能拥有一个私钥，同时有一个用于储存公钥的authorized_keys
>
> 当hadoop01想要远程登录别的linux时，他会带着自己的私钥去登陆，如果目标linux发现自己的authorized_keys中有和hadoop01的私钥相匹配的值，就允许hadoop01免密登录



##### 在客户端生成公钥和私钥

``` bash
ssh-keygen -t rsa	#生成的公钥和私钥默认放在~/.ssh中
```



##### 将公钥导入目标linux的authorized_keys文件

``` bash
scp ~/.ssh/id_rsa.pub hadoop@hadoop02:/home/hadoop/.ssh/	#将公钥复制到目标linux中
ssh hadoop02	#远程登录目标linux
cat id_rsa.pub >> authorized_keys	#将公钥导入目标的authorized_keys文件中
chmod 600 authorized_keys	#将这个文件的权限改为只有拥有者能读写，不然不能免密登录
```



##### 开始免密登录

``` bash
ssh hadoop02	#免密登录设置成功
```



#### 将此虚拟机中的文件同步到其他虚拟机中

##### 编写同步脚本——放在/home/hadoop/bin目录下

作用是将指定的文件或者文件夹，复制给其他的虚拟机一份。如果目标虚拟机上有了，则跳过这个虚拟机

``` bash
#!/bin/bash

#获取输入参数个数，如果没有参数，直接退出
pcount=$#  
if ((pcount==0)); then 
echo no args;
exit;
fi
 
#获取文件名称 
p1=$1
fname=`basename $p1`
echo fname=$fname 

#获取上级目录到绝对路径
pdir=`cd -P $(dirname $p1); pwd`
echo pdir=$pdir

#获取当前用户名称
user=`whoami`

#循环
for host in hadoop01 hadoop02 hadoop03
do
    echo ------------- $host ---------
    rsync -av $pdir/$fname $user@$host:$pdir/
done
```

##### 使用同步脚本

```bash
xsync /opt/module/hadoop	#将hadoop文件夹同步到脚本中出现的几台虚拟机中

xsync /opt/modusle/jdk

sudo ./xsync /etc/profile.d/env.sh	#因为xsync放在/home/hadoop/bin目录下，所以xsync是hadoop的专有命令。但同步env.sh需要用root用户来操作，所以我们要把工作目录切换到/home/hadoop/bin
```



#### 删除其余linux中的openjdk

> 因该在拷贝虚拟机之前就把这些做好的，唉