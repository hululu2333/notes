### hbase简介

***

> hbase是一种分布式、可扩展、支持海量数据存储的NoSQL数据库（NoSQL表示非关系型）
>
> hbase中的数据是储存在hdfs中的，其实habse和hdfs都是服务，数据最终还是存在linux中的文件系统中



#### hbase的数据模型

##### hbase的逻辑结构

![image-20201104200458021](F:\学习笔记\hbase\image\hbase的逻辑结构.png)

> hbase在结构上更像一个多维表，分为列、列族、行键、region。
>
> 多个列构成一个列族，同一个列族的数据储存在一个文件夹中
>
> 一个region是一个表的切片，同一个region的数据储存在一个文件夹中
>
> 也就是说，每个region都会创建一个文件夹，然后每个列族会在这个文件夹中再创建一个文件夹
>
> 行键是按字典序来排的，先按第一个字符来排，小的在上，大的在下。第一个字符相同就按第二个来排，以此类推

***

##### hbase的物理结构

![image-20201104200458021](F:\学习笔记\hbase\image\hbase的物理结构.png)

> 理解的时候我们可以把hbase的表看作一个二维表，这个二维表被region和列族分为若干个份。每份是一个store
>
> 每份会被储存在不同的文件中，每个store中的每行会以上图的形式储存
>
> 第三行和第四行是相同的数据，但t4>t3，当行键、列族、列名都相同时，hbase会显示时间戳大的那行数据



##### NameSpace

> 类似于关系型数据库的database概念，每个命名空间下会有多个表。Hbase有两个自带的命名空间，一个叫hbase，一个叫default。hbase放的是hbase内置的表，default是用户默认使用的命名空间



##### Region

> region相当于关系型数据库中表的概念，我们在一个namespace中创建一个region，然后我们一直向这个region中写入数据，当这个region足够大了，他就会被RegionServer切分为两个region。这个新的region被分配到哪个RegionServer由Master决定
>
> hbase在定义表的时候只需要声明列族，不需要声明具体的列，所以我们在向hbase中写入数据时，可以动态的指定列名



##### Row

> hbase中每行数据都由一个RowKey和多个Column(列)组成，数据是按照RowKey的字典顺序储存的，**我们查询的时候只能根据RowKey进行检索**，所以RowKey的设计十分重要



##### Column

> hbase中每个列都由Column Family（列族）和Column Qualifier（列限定符）进行限定。例如info: name, info: age。建表时，只需指明列族，而列限定符无需预先定义。



##### TimeStamp

> 用于标识相同数据的不同版本，如果两条数据，他们的行键，列族和列名都相同，他们就是一条数据。如果我们查询这条数据，hbase会返回时间戳大的那条数据。当我们写入数据时，如果不指定时间戳，系统会以当前时间为时间戳。



##### Cell

> 由{RowKey，Column Family，Column，Time Stamp}唯一确定的单元格，当‘我们确定了行键，列族和列名，我们就确定了一条数据，但这时这条数据会有不同版本。当我们再加上时间戳，那就确定了唯一的一条数据。
>
> 在定义关系型数据库的表时，我们要给每个列定义类型，但hbase中不需要定义列，所以habse中的数据没有类型，就是字节码或者字节数组



