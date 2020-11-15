### Zookeeper的客户端操作

***

> 运行zkCli.sh进入zookeeper的shell窗口

``` bash
./zkCli.sh
```



#### zookeeper的常用命令

``` bash
#查看该节点下有哪些子节点，用ls2就是查看详细信息
ls /

#创建节点并存入数据，不存数据就不让你新建节点
create /bighu "i am bighu"

#获得节点中的值
get /bighu

#创建短暂节点，当客户端和服务器端断开连接，节点被删除
create -e /smallhu "i am smallhu"

#创建带有序号的节点，用来记录节点创建的前后顺序
create -s /bighu/xiaoli "i am xiaoli"

#修改节点的值
set /bighu "no bighu"

#监听节点的值,当这个节点的值被修改后，zookeeper服务器端会通知执行监听命令的那个客户端。只有第一次被修改的时候通知
get /bighu watch

#监听节点的子节点，如果有子节点被删除或有新增的子节点，则通知客户端
ls /bighu watch

#删除节点
delete /bighu

#递归删除节点
rmr /bighu
```



#### 打印节点值时，一大堆信息的含义

![image-20201019152613617](C:\Users\hu\AppData\Roaming\Typora\typora-user-images\image-20201019152613617.png)