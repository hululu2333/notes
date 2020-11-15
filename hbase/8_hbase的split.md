### hbase的split

***

> 一开始，一张表中只有一个region
>
> 这个region中的数据储存在hdfs中的一个文件夹内
>
> 而这个文件夹内又会有几个子文件夹，子文件夹的个数由这个表的列族数确定。每个子文件夹储存着一个列族的数据。
>
> 可以说，每个store的数据都会储存在某个子文件夹中。每个store中都会有若干HFile，每flush一次都可能会新增一个HFile，理论上说这些HFile最终都会合并为一个大HFile。但这个HFile过大也会带来不便，这时就要进行切分



#### 切分细节

> split并不是简单的将HFile一分为二，而是将这个region一分为二，创建一个新的region。
>
> 切分时机我们可以在hbase-site.xml里配置，低版本是当某个store中的某个HFile的大小到了指定值，就切分region。高版本是当某个store中的所有HFile的总大小到了一个值就切分
>
> 但是这样的话容易产生数据倾斜（具体是怎么产生数据倾斜的不太清楚）
>
> 想要解决这个问题，我们要在建表的时候进预分区

![image-20201114143538194](C:\Users\hu\AppData\Roaming\Typora\typora-user-images\image-20201114143538194.png) 



 #### 预分区

> 在建表的时候就确定将来要创建几个region，并且在建表的时候把这些分区创建好。然后还得指定每个region存放的rowkey的范围

