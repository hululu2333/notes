### hbase的命令行操作

***

#### 一些基本操作

``` bash
# 进入hbase客户端
bin/hbase shell

# 查看帮助命令(以下命令都是进入hbase客户端后)，hbase中的所有命令都会显示
help

# 查看数据库中有哪些表
list
```



#### DDL

``` bash
# 创建一个student表，他有info和study两个列族
create 'student','info','study'

# 查看student的表结构，version为1表示同一格数据只保留一个版本的
describe 'student'

{NAME => 'info', VERSIONS => '1', EVICT_BLOCKS_ON_CLOSE => 'false', KEEP_DELETED_CELLS =>
 'FALSE', CACHE_DATA_ON_WRITE => 'false', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER'
, MIN_VERSIONS => '0', REPLICATION_SCOPE => '0', BLOOMFILTER => 'ROW', CACHE_INDEX_ON_WRI
TE => 'false', IN_MEMORY => 'false', CACHE_BLOOMS_ON_WRITE => 'false', PREFETCH_BLOCKS_ON
_OPEN => 'false', COMPRESSION => 'NONE', BLOCKCACHE => 'true', BLOCKSIZE => '65536'} 


# 修改表的结构
alter 'student',{NAME=>'info1',VERSIONS=>3}

# 使表下线，并删除表。表如果在enable状态下，就不能被删除
disbale 'student'
drop 'student'

# 对namesapce进行操作，和对表操作是一样的，只是把命令后面加上_namespace
list_namespace #查看有哪些namespace
create_namespace 'bigdata'
create 'bigdata:student' #在bigdata这个命名空间下创建一个student表
```



#### DML

``` bash
# 向表中插入数据，这四个参数分别是表名，行键，列族:列名，值
put 'student','1001','info:name','zhangsan'
put 'student','1001','info:name','zhangsansan'
#插入一个行键、列族、列名都相等的值，等于是修改。前提是新插入的值的时间戳大于前面一个

# 对student表进行扫描，除此之外我们可以加上一些限制条件
scan 'student'
scan 'student',{STARTROW=>'1001',STOPROW=>'1003'}
scan 'student',{RAW=> true,VERSIONS=> 10} #查看十个版本内的数据

# 获得student表中行键为1001的数据，get命令最大范围是指定到行键
get 'student','1001'
get 'student','1001',{COLUMN=>'info:name',VERSIONS=>3}
# 获得行键为1001，且列族列名为info:name的三个版本的记录，前提是建表的时候确定VERSIONS在3以上

# 删除一个数据，前提是当前的时间戳大于之前数据的最大时间戳。
# 按理说插入一个delete型数据，之前版本的都会被覆盖，但现在只会覆盖版本最大的那个。假如我们有三个版本的记录，如果要删除这三个，我们要delete三次。疑点是我们插入的delete型记录的时间戳不是当前的时间戳，而是被覆盖的那个记录的时间戳
delete 'student','1001','info:name'

# 根据行键删除某一行记录
deleteall 'student','1001'

# 清空一个表
truncate 'student'
```

