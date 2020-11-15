### Zookeeper的作用

***

#### 选举Controller

> zookeeper中有一个叫controller的节点，最先在zookeeper中注册的kafka节点会在controller节点中注册。然后这个kafka节点就是整个kafka集群的controller
>
> controller负责管理broker的上下线，也负责Topic的分区、分区副本的分配和leader的选举