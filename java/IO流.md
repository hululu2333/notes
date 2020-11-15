### IO流

***

#### IO流中的一些概念

> 节点流：直接对接数据源或数据目的地的流；以ByteArray,File开头的流一般是节点流。如FileInputStream，ByteArrayInputStream
>
> 包装流或处理流：对节点流进行封装，用于简化操作或提高性能；如BufferedInputStream，ObjectInputStream



#### 字节流

> 字节流是以字节为单位读取数据。InputStream和OutputStream是两个接口，所有字节流都是根据这两个接口衍生而来

<img src="F:\学习笔记\java\images\字节流.png" alt="image-20200825193811317" style="zoom:50%;" />



#### 字符流

> 字符流是以字符为单位读取数据。字符流的底层也是由字节流实现。
>
> Reader和Writer是字符流的两个接口，所有字符流都由这两个接口衍生而来

<img src="F:\学习笔记\java\images\字符流.png" alt="image-20200825194353273" style="zoom:50%;" />



#### 数据源——File

> JVM不能直接操作硬盘上的数据，File对象是程序和硬盘上文件的一种联系。所以只有在运行时才知道File对象所对应的文件是否存在
>
> File对象不仅可以表示文件，也可以表示文件夹。在写代码的过程中，我们把File对象当作是一个文件或文件夹就好了

##### 构造器

``` java
//通过路径创建File对象
File f1 = new File("/opt/module/a.txt");

//通过父路径和文件名
File f2 = new File("/opt/module","a.txt");

//通过文件夹对象创建
File f3 = new File(new File("/opt/module"),"a.txt");
```

***

##### File的一些方法

``` java
f.isDirectory(); //判断这个file是否是路径
f.length(); //返回该文件的长度
f.list(); //返回一个String数组，为该路径下的文件夹名和文件名
f.listRoots(); //列出可用的文件系统根。linux下就是root,windows就是盘符
```



#### 使用IO流的标准步骤

> 如果输入流读取不到值，如is.read()，发现数据源已经没数据了，这时返回-1
>
> **输入流和输出流不能同时指向同一文件，不然文件会被清空，就读不到数据了**

``` java
//1.创建源
File f = new File("src/main/java");

//2.选择流
InputStream is = new FileInputStream(f);

//3.进行读取操作
int b = is.read();//从输入源读取一个字节，但是以int形式返回
System.out.println((byte)b);

//4.关闭流
is.close();
```



#### 特殊的流

> 以File和ByteArray开头的流是节点流，但ByteArrayInputStream和ByteArrayOutputStream有点特殊。这两个流所对接的节点是虚拟机中的一块内存。可以实现程序向程序内输入输出
>
> ByteArrayOutputStream用于程序向程序中输出

``` java
ByteArrayOutputStream bos = new ByteArrayOuputStream();
ObjectOutputStream oos = new ObjectOutputStream(bos);

oos.writeObject(new Student());//将一个对象写到了虚拟机中的字节数组中
```

