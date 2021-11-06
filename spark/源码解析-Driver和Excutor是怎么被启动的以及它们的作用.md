## 通过源码来解析Driver和Excutor

具体的内容以现在的水平还有点难看懂，目前就只关注以下这几句话



流程就是用户以client的身份向master提交任务，master去worker上面创建执行任务的载体（driver和excutor）。

以正常提交spark任务的流程来说，client是spark-submit这个脚本，master是ResourceManager，worker是NodeManager

![img](F:\学习笔记\scala\imgs\spark集群的部署模式.jpg)



### client、driver、excutor的关系

Master和Worker是服务器的部署角色，程序从执行上，则分成了client、driver、excutor三种角色。按照模式的不同，client和driver可能是同一个。（此处的client是什么？不太理解）

以2.2.0版本的standalone模式来说，他们三个是独立的角色。client用于提交程序，初始化一些环境变量；（此处的client是什么？今后看了文末链接中的内容后应该就能理解）

**driver用于生成task并追踪管理task的运行；excutor负责最终task的执行。**





https://zhuanlan.zhihu.com/p/59461420