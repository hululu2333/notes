### MapReduce案例

***

#### 全局排序

> **什么是全局排序：**默认排序是局部排序，每个reduce任务会生成一个part文件，在单个part文件中，数据是有序的。但所有part文件综合起来看，就是无序的。全局排序就是要在所有part文件中，数据都是有序的             
>
> **实现全局排序：**创建一个采样器，在数据进入map任务之前进行采样。在这些样品中选出几个分界点，当这些样品排好序时，这几个分界点可以将他们分为相等的几份。分界点的个数由reduce个数决定，有三个reduce就有2个分界点
>
> 



