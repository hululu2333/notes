### 使用spark-shell完成WordCount

***

#### 创建输入源

> 在spark目录创建一个input文件夹，在文件夹中创建两个文本文件



#### 编写代码

> 编写代码的过程我们在spark的交互界面完成，使用scala语言
>
> 我们会用到一个名为sc的对象，这是spark的上下文对象，启动交互界面时，这对象就被创建好了

``` scala
sc.textFile("../input").flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_).collect

//textFile("../input"): 读取input文件夹中的数据，结果为一行一行的字符串
//flatMap(_.split(" ")): 压平操作，以空格为分隔符，将一行数据映射成一个个单词
//map((_,1)): 对每一个元素操作，将单词映射为元组。单词为key，1为value
//redeuceByKey(_+_): 把key值相同的元组合为一个元组，key值不变，value为这些元组value的和
//collect: 将数据收集到Driver端展示
```

![image-20200928201533140](C:\Users\hu\AppData\Roaming\Typora\typora-user-images\image-20200928201533140.png)