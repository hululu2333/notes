### JDBC

***

> JDBC的全称为Java Data Base Connection。是sun公司提供的一套编程接口，由java的类和接口组成。
>
> 如果有哪个数据库公司想让java能连上自己的数据库，他就得实现这套接口。实现这套接口后产生的jar包，我们就把他称为数据库驱动



#### 使用JDBC的几个步骤

> 注册驱动

``` java
//这个语句的返回值是一个Class实例，但是这个实例我们没有接收，那为什么还要创建呢
//我们把这个类加载到JVM中时，会进行一些操作，其中一个就是运行它的静态初始化代码块
Class.forName("com.mysql.jdbc.Driver");
```



> 建立连接

``` java
/**
 * 传入的参数为url,user,password
 * Connection是接口，DriverManager.getConnection返回的是JDBC4Connection型的对象
 * conn中包含了socket对象，是JVM所在主机和数据库所在主机之间的通道。创建连接是个比较耗时的过程
 */
Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/bd2005",
                    "root","1008");
```



> 创建Statement对象并执行语句

``` java
//创建statement对象
Statement stmt = conn.createStatement();
String  sql = "select * from s_dept";
boolean flag = stmt.execute(sql); //如果返回值是结果集就返回true，不然就返回false
ResultSet rs = stmt.executeQuery(sql); //运行查询语句，返回ResultSet
int effect = stmt.executeUpdate("sql"); //运行更新语句，返回被影响的行数

/**
 * Statement用来执行sql语句
 * 但Statement用的不多
 * 一是要把某个变量中的数据加入到sql语句中时要进行字符串拼接，比较麻烦
 * 二是可能会造成sql注入
 *
 * 所以我们一般使用PreparedStatement
 * PrepareStatement会进行预处理，处理效率更高。
 * 而且也能防止sql注入
 */
String sql2 = "delete from s_dept where id = ?"; //待传入参数用占位符代替
PreparedStatement ps = conn.prepareStatement(sql2);

ps.setInt(1,2); //补充占位符只能用某个具体的类型，就不会出现传入"1 or 1=1"这种破坏性语句了

ps.execute(); //执行语句
```



> 对ResultSet进行处理

``` java
/**
 * 对ResultSet进行操作
 * rs.next()相当于把游标指向下一个位置，并返回一个boolean值，判断当前位置是不是非空
 * 游标一开始是指向第一个数据的前面一个位置
 * rs.getInt(1)表示获得这行数据的第一列，且为int型的数据
 */
while(rs.next()){
	rs.getInt(1);
	rs.getString(2);
}
```



> 关闭资源

``` java
rs.close();
stmt.close();
conn.close();
```





#### 批处理

> 当我们要运行大量sql语句时，如果还是用一般的方法，效率就会很低
>
> 我们可以通过stmt.addBatch(sql)；将要运行的sql语句都添加进去，然后运行stmt.executeBatch()将这些sql语句一起编译，然后送到mysql中去运行
>
> 对于量特别大的批处理，使用Statement。因为PreparedStatement的预编译空间有限，可能会出问题

``` java
//进行批处理
for(int i = 0;i<100;i++){
	stmt.addBatch("insert into s_emp(id,name) values("+i+",bighu)");
}
stmt.executeBatch();//将这一百条sql语句编译，并送去mysql执行
```

