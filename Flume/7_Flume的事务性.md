### Flume的事务性

***

![image-20200621134342427](F:\学习笔记\Flume\imgs\Flume事务.png)



#### Put事务

> source从数据源拿到一行数据，并将其封装从Event。这时source并不是直接将Event发给channel。
>
> 这时启动了Put事务，Put事务分为两个动作。
>
> 第一个是doPut，将Event放进临时缓冲区putList中
>
> 第二个动作是doCommit，查看channel中是否能够放的下缓冲区的Event，如果可以，就将数据发送到channel中，如果不可以，数据就回滚到临时缓冲区



#### Take事务

> Take事务也分为两个动作
>
> 第一个是doTake，将Event从channel中取出，放进临时缓冲区takeList，然后发送到目的地
>
> 第二个是doCommit，如果数据全部发送成功，就清除临时缓存区。如果失败，就将临时缓冲区中的数据还给channel