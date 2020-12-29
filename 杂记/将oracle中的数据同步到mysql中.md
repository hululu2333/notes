### 将oracle中的数据同步到mysql中

***

> oracle中有一张表，他的数据量十分巨大。我们现在要把这张表中的某几个字段的数据导入到mysql。
>
> 并在之后每天的八点半再做一次同步，将oracle中新产生的数据再导入mysql



#### 遇见的困难

> 这次的任务遇见的困难主要是因为对oracle和spark不熟悉导致的
>
> **oracle url的写法：**我采用的是 jdbc:oracle:thin:@//10.170.1.36:1521/QTREPORT 这种写法。QTREPORT指的是服务名，一个服务名会包含一些节点的信息。服务名的定义要在oracle的某个配置文件中来做
>
> **oracle中的date类型：**这里用两个oracle函数可以解决，to_date('2020-12-1' , ,'yyyy-mm-dd hh24:mi:ss')可以将字符串'2020-12-1'这个字符串转换为oracle中的date类型。	to_char(day,'yyyy-mm-dd ')可以将day这个date类型以我们指定的格式转换为字符串
>
> **替换dataFrame中的空值：**因为mysql表中定义了字段都不能为空值，所以要把从oracle中获取的空值都替换为别的值。在scala中去除DF中的空值和在java中是不一样的，虽然不会报错，但使用java的那种方式会没有效果。										val df4 = oraDF.na.fill(value="0",Array\[String\]("output"))										 //将output这列的空值替换为字符串0
>
> **将dataFrame中的数据写进mysql：**可以使用df.write.jdbc()这个方法，.write是获得dataFrameWriter对象



#### 实现数据同步的代码

``` scala
import java.util.Properties

import org.apache.spark.sql.{DataFrame, SaveMode, SparkSession}

object OracleToMysql {
  def main(args: Array[String]): Unit = {
    val spark = SparkSession.builder().appName("OracleToMysql").master("local").getOrCreate()

    //从oracle中拿到数据
    val query="(select to_char(day,'yyyy-mm-dd ') day, SUBSTR(to_char(day,'yyyy-mm-dd '),1,7) month, " +
      "type, workshop_code, part_spec, output, line_code from T_WORKER_OUTPUT " +
      "where day = to_date('"+args(0)+"' , 'yyyy-mm-dd hh24:mi:ss')) t"
    val oraDF = spark.read
      .format("jdbc")
      .option("url", "jdbc:oracle:thin:@//10.170.1.36:1521/QTREPORT")
      .option("dbtable", query)
      .option("user", "ckmes")
      .option("password", "qt123mes")
      .option("driver", "oracle.jdbc.driver.OracleDriver")
      .load()


    //将output列的空值替换为o
    //将type列的空值替换为空字符串
    val df4 = oraDF.na.fill(value="0",Array[String]("output"))
    val df5 = df4.na.fill(value="",Array[String]("type"))

    //设置目标mysql的一些配置
    val url="jdbc:mysql://"+args(1)+"/ziyun-iot?useUnicode=true&characterEncoding=utf-8"
    val conn = new Properties()
    conn.setProperty("user", "ziyunIot")
    conn.setProperty("password", "Pass1234")
    conn.setProperty("driver", "com.mysql.jdbc.Driver")

    //将处理好的数据导入mysql
    //10.170.6.47:3306
    df5.write.mode(SaveMode.Append).jdbc(url, "mes_output", conn)

  }
}
```



#### 运行数据同步代码的脚本

``` bash
#!/bin/bash

# 获取昨天的日期，格式为yyyy-mm-dd
do_date=`date -d "-1 day" +%F` 
echo "获取日期$do_date"

# 指定mysql的ip地址
ip='10.170.6.161:3306'
echo "mysql的地址为$ip"

# 执行spark程序，将日期为昨天数据同步到mysql中
echo "开始执行spark程序"
spark-submit --class com.hu.datatansfer.OracleToMysql --master yarn --executor-memory 20G --num-executors 50 /home/ems/spark_job/ems/report/data_transfer_spark-1.0-SNAPSHOT.jar $do_date $ip
```



#### 创建定时任务

``` bash
30 8 * * * sh /home/ems/spark_job/ems/report/sync_oracle_to_mysql.sh > /home/ems/spark_job/ems/report/sync_oracle_to_mysql.log
```

