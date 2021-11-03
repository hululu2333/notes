将hbase的数据导入到hive有两种方式，一是Hive 整合映射 HBase，二是通过spark读取HFile，然后做操作后插入hive



hive整合hbase：https://www.jianshu.com/p/a36d52b7be31 （将hive中的数据和Hbase中的关联起来）

spark读取HFile：https://blog.csdn.net/weixin_30753873/article/details/95164054  （其实也是经过了Hbase服务，并不是直接读HDFS是的HFile）



hbase2.x提供了HFileInputFormat，1.x没有提供。所以用2.x的版本的话，应该是可以直接从HDFS是读取HFile文件的