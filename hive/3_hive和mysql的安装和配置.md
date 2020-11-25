### hive和mysql的安装

***

#### 安装hive

``` bash
tar -zxvf apache-hive-2.3.6-bin.tar.gz -C /opt/module/
```



#### 配置hive

> 配置hive-env.sh

``` bash
# 去掉文件名后的.template
mv hive-env.sh.template hive-env.sh

# 在hive-env.sh中添加以下内容
export HADOOP_HOME=/opt/module/hadoop
export HIVE_CONF_DIR=/opt/module/hive/conf
```



#### 启动hive

``` bash
/opt/module/hive
```

> 不知道为什么总是报这样的错

> SemanticException org.apache.hadoop.hive.ql.metadata.HiveException: java.lang.RuntimeException: Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient

> 一开始以为我是hive2.x，默认是连接mysql数据库，所以我就去装好了mysql再来用hive。结果还是这样。



#### 卸载自带的mysql

> 卸载自带的mysql时一定要卸干净，使用whereis和find找出所有自带的mysql的残留

``` bash
# 查看主机中中有没有自带的mysql，有就将其卸载
rpm -qa | grep mysql

# 删除自带的mysql，光这样卸载还不够
sudo rpm -e --nodeps mysql-libs-5.1.71-1.el6.x86_64

# 这两个路径搜索出来的文件和目录，除了自己创建的的，其他的都删除
whereis mysql
find / -name mysql
```



#### 下载mysql包

> 去官网下载mysql包，下载redhat版的。因为centos和redhat的内核一致
>
> mysql8用起来有点问题，然后我又去下了5.6版本的

<img src="C:\Users\hu\AppData\Roaming\Typora\typora-user-images\image-20201119122042883.png" alt="image-20201119122042883" style="zoom: 80%;" />



#### 安装客户端和服务端

``` bash
sudo rpm -ivh MySQL-server-5.6.50-1.el6.x86_64.rpm 
sudo rpm -ivh MySQL-client-5.6.50-1.el6.x86_64.rpm
```



#### 对mysql进行一些操作

``` bash
# 查看mysql的密码
cat /root/.mysql_secret

# 启动mysql
service mysql start

# 查看mysql运行状态
service mysql status

# 登录mysql，需要安装好了client后才能登录
mysql -uroot -pDFldTj3ceR1wxxTH

# 修改mysql的密码，从命令行进入mysql后执行
SET PASSWORD=PASSWORD('123456');

# 配置无主机登录，不然远程登录可能会有问题，首先我们要进入mysql这个数据库
# 通过select user,host,password from user;可以看这个表结构
update user set host='%' where host='localhost';
delete from user where host='hadoop01';
delete from user where host='127.0.0.1';
delete from user where host='::1';

# 刷新mysql，使对user表的改动立即生效
flush privileges;
```



#### JDBC

> 将java连接mysql的驱动放到hive的lib目录下

``` 
mv mysql-connector-java-5.1.39-bin.jar /opt/module/hive/lib/
```



#### 配置hive-site.xml

>touch hive-site.xml
>
>vi hive-site.xml
>
> 将hive-site.xml中的内容替换为以下部分，hive-default.xml.temlate就是hive给我们提供的hive-site.xml的一个案例

``` xml
<?xml version="1.0">
<?xml-stylesheet type="text/xs1" href="configuration.xsl"?>
<configuration>
    <!-- 设置hive将连接到mysql的哪个数据库 -->
    <property>
        <name>javax.jdo.option.ConnetcionURL</name>
        <value>jdbc:mysql://hadoop01:3306/metastore?createDatabaseIfNotExist=true</value>
    </property>
    
    <!-- 设置驱动名 -->
	<property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
    </property>
    
    <!-- 设置登录的用户名 -->
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
    </property>
    
    <!-- 设置登录密码 -->
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>123456</value>
    </property>
</configuration>

```

