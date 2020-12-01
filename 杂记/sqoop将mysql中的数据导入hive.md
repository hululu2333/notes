### sqoop将mysql中的数据导入hive

***

> mysql中存在一张表uat_test，现在我们要用sqoop将uat_test表中的数据导入到hive的一张表中
>
> 导入的时候使用ORC格式，并采用LZO压缩
>
> hive的表要分区，并且是动态和静态结合的方式



#### 新建hive表

> 35个字段
>
> 指定分区、储存格式和压缩方式都是在建表的时候指定
>
> 

``` sql
create table static_dyna ( 
client_id string, 
dev_template_unique_id string, 
finalStatusColor string, 
SpindleNum string, 
odbst_motion string, 
statusColor string, 
transformed_date string,
CurProgramNo string, 
odbst_run string, 
create_date_number string, 
CycleTime string, 
FinishPartsNum string, 
odbst_edit string, 
iot_unique_message_id string, 
odbst_emergency string, 
AlarmNum string, 
id string, 
ServoNum string, 
create_date string, 
data_col_time string, 
FeedOverride string, 
odbst_aut string, 
finalStatusName string, 
device_id string, 
odbst_hdck string, 
that_moment_program_version_id string, 
Feedrate string, 
receive_date string, 
PathNo string, 
TotalMachiningTime string, 
pbox_id string, 
odbst_tmmode string, 
TotalPowerOnTime string, 
Servos string, 
Spindles string ) 
partitioned by (dt string, hr string) 
row format delimited fields terminated by ',' 
stored as orc tblproperties ("orc.compress"="LZO");
```



> 用于在beeline中建表
>
> create table uat_test2( client_id string, dev_template_unique_id string, finalStatusColor string, SpindleNum string, odbst_motion string, statusColor string, transformed_date string, CurProgramNo string, odbst_run string, create_date_number string, CycleTime string, FinishPartsNum string, odbst_edit string, AlarmNum string, id string, ServoNum string, create_date string, data_col_time string, FeedOverride string, odbst_aut string, finalStatusName string, device_id string, odbst_hdck string, that_moment_program_version_id string, Feedrate string, receive_date string, PathNo string, TotalMachiningTime string, pbox_id string, odbst_tmmode string, TotalPowerOnTime string, Servos string, Spindles string ) row format delimited fields terminated by ',';



#### 将数据从mysql导入hdfs

> 默认是以逗号分割，导入细节见sqoop用法



#### hive的正确打开方式

> 我们使用的是ziyunbd01的hive，使用hive时要采用beeline登录，不然会出错。用户名是hive，密码是Pass1234。（也可以直接这样beeline -u jdbc:hive2://192.168.0.121:10000 -n hive）
>
> 用beeline前要启动hive2服务：hive --service hiveserver2
>
> 使用hive命令登录不行，应该是权限的问题，因为hive在hdfs上储存数据的文件夹的所有者是hive用户，而我们的是root用户。而我们在使用beeline登录时，采用的用户名是hive就能正常运行。



#### 将数据从hdfs导入hive—只测试静态分区

> 因为sqoop将数据从mysql导入hdfs时，各字段之间是以逗号分割，所以建hive表时行分隔符要设为逗号，而不是默认的\t
>
> load data inpath '/mytargetdir/uat_test' into table uat_test2;

> 如果要导入分区的话，可以参考以下语句
>
> load data inpath '/mytargetdir/uat_test' into table uat_test2 partition(genger='M');
>
> insert overwrite table partition_test partition(stat_date='2015-01-18',province='jiangsu')  select member_id,name from partition_test_input 
>
> **前提是你在建表的时候就要分好区，并且一此只能插入一个分区**
>
> 更多hql案例https://www.cnblogs.com/liaozhilong/p/9681450.html



#### 将数据从hdfs导入hive—动静结合

> 我的理解是，建表的时候指定好分区。插入数据的时候结合静态分区和动态分区的插入语句。实现的话参考https://www.cnblogs.com/sunpengblog/p/10396442.html
>
> 首先我们要打开动态分区
>
> set hive.exec.dynamic.partition=true;
> set hive.exec.dynamic.partition.mode=nonstrict;

``` sql
# 执行以下hql语句
insert overwrite table static_dyna 
partition(dt='2020-11-25', hr)
select *, split(split(data_col_time, ' ')[1],':')[0] 
from uat_temp 
where split(data_col_time, ' ')[0] = '2020-11-25'
```

