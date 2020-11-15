### Yarn

***

#### 什么是Yarn

> Yarn是一个资源管理，任务调度的框架。主要包含ResourceManager，NodeManager和ApplicationMaster三个模块
>
> **ResourceManager：**负责集群中所有资源的监控、分配和管理
>
> **ApplicationMaster：**一个ApplicationMaster负责一个Application的运行
>
> **NodeManager：**每个NodeManager负责其对应节点的维护



#### Yarn的运行流程

> 1.用户向Yarn提交执行Application的请求。例如我们在linux的命令行窗口中输入
>
> hadoop jar online-bd2002-hadoop.jar WordCount，来运行一个mapReduce任务
>
> 然后相关信息会被发送到ResourceManager的一个端口，这时Yarn就收到我们的请求了

> 2.ResourceManager先在一个节点上创建一个Container，Container是一批资源的集合，如内存磁盘cup。然后ResourceManager和该节点的NodeManager通信，让它在这个Container中启动一个ApplicationMaster

> 3.ApplicationMaster被建立起来后，第一件事就是去ResourceManager中注册。这样的话，用户就能通过ResourceManager来查看各个Application的运行状态。

> 4.通常来说，一个Application会被分为很多小任务，这些小任务会被分在不同的节点上。这时，ApplicationMaster会通过RPC协议向ResourceManager申请执行这些小任务所需的资源

> 5.资源申请到了后，ApplicationMaster会和资源所在节点的NodeManager通信，让他们在这些资源上启动小任务

> 6.这些小任务会不断地通过RPC协议向ApplicationMaster汇报自己的状态和进度。这样的话，当有的小任务挂了，ApplicationMaster可以通知对应的NodeManager重新启动任务。我们也可以通过ResourceManager来查看整个大任务的进度了

> 7.当所有的小任务都运行完了后，ApplicationMaster从ResourceManager中注销，然后关闭自己