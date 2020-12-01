### sqoop的安装、配置与使用

***

#### sqoop的安装与配置

> 解压缩sqoop

``` bash
tar -zxvf sqoop-1.4.7.bin__hadoop-2.6.0.tar.gz

```

> 配置环境变量

``` bash
vi /etc/profile

# 在profile中添加以下内容
export SQOOP_HOME=/sqoop/sqoop-1.4.7
export CATALINA_HOME=$SQOOP_HOME/server
export PATH=$SQOOP_HOME/bin:$PATH 

```

> 导jar包

``` bash
# 将sqoop家目录下的sqoop-1.4.7.jar复制到hadoop中
cp $SQOOP_HOME/sqoop-1.4.7.jar $HADOOP_HOME/share/hadoop/mapreduce/

# 将JDBC放到sqoop的lib目录下
cp mysql-connector-java-5.1.48.jar $SQOOP_HOME/lib

```



#### sqoop的使用

> 将mysql中的数据导入hdfs，在hdfs的/mytargetdir下可以看见我们导入的数据
>
> 这个案例用过，确认没什么问题

``` bash
#执行sqoop命令，m是指map任务的个数，当m大于1时，需要指定split-by
sqoop import --connect jdbc:mysql://192.168.0.124:3306/test --username ziyunIot --password Pass1234 --table uat_test --warehouse-dir /mytargetdir -m 1

```

> 将mysql中的数据导入hive，并采用BZip2压缩格式
>
> 没有使用过

``` bash
sqoop import --connect jdbc:mysql://192.168.0.124:3306/test --username ziyunIot --password 123456 --table uat_test --hive-import --hive-database test --create-hive-table --compress --compression-codec org.apache.hadoop.io.compress.BZip2Codec -m 1

```

> 一次性导入一个数据库的所有表，这里演示的是oracle，把connect换成mysql因该就可以适用于mysql

``` bash
sqoop import-all-tables \
  --connect jdbc:mysql://localhost:3306/retry_db \
  --username cloudera \
  --password secret \
  --warehouse-dir /mydata
```





参考网页 https://ask.hellobi.com/blog/marsj/4114

sqoop的常用案例和进阶 https://www.cnblogs.com/piperck/p/9984236.html