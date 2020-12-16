### MR与Spark

***

> Spark在很多方面比mapreduce要强，所以大多数情况下我们会用Spark代替MR。
>
> 但在运行一些超大数据量的任务并且资源比较有限时，MR会更适合。因为这种情况使用Spark不太稳定。
>
> 这是由MR和Spark底层的运行机制决定的
>
> MR的一个job会分为若干task，无论是mapTask还是reduceTask，都会启动一个独立的进程。
>
> Spark则会在需要的节点上启动一个executor，executor是一个进程。Spark的job也会分为若干task，但Spark的task会在executor中以线程的形式运行
>
> 以上是个人总结，原文如下



#### 概括MR和Spark

> 从整体上看，无论是Spark还是MapReduce都是多进程模型。如，MapReduce是由很多MapTask、ReduceTask等进程级别的实例组成的；Spark是由多个worker、executor等进程级别实例组成。
>
> 但是当细分到具体的处理任务，MapReduce仍然是多进程级别，这一点在文章[《详解MapReduce》](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s%3F__biz%3DMzI0Mjc0MDU2NQ%3D%3D%26mid%3D2247483720%26idx%3D1%26sn%3D84362d3588a219d223210cf046f8a19d%26scene%3D21%23wechat_redirect)已有说明。而Spark处理任务的单位task是运行在executor中的线程，是多线程级别的。



#### 进程过多的劣势

> MR在运行一个任务时会创建很多进程，对于多进程，我们可以很容易控制它们能够使用的资源，并且一个进程的失败一般不会影响其他进程的正常运行，但是进程的启动和销毁会占用很多时间，同时该进程申请的资源在进程销毁时也会释放，这就造成了对资源的频繁申请和释放也是很影响性能的，这也是MapReduce广为诟病的原因之一。



#### MR运行机制的特点

> 1.每个MapTask、ReduceTask都各自运行在一个独立的JVM进程中，因此便于细粒度控制每个task占用的资源（资源可控性好）
>
> 2.每个MapTask/ReduceTask都要经历申请资源 -> 运行task -> 释放资源的过程。强调一点：每个MapTask/ReduceTask运行完毕所占用的资源必须释放，并且这些释放的资源不能够为该任务中其他task所使用
>
> 3.可以通过JVM重用在一定程度上缓解MapReduce让每个task动态申请资源且运行完后马上释放资源带来的性能开销
>
> 但是JVM重用并不是多个task可以并行运行在一个JVM进程中，而是对于同一个job，一个JVM上最多可以顺序执行的task数目，这个需要配置参数mapred.job.reuse.jvm.num.tasks，默认1。



#### Spark运行机制的特点

> 对于多线程模型的Spark正好与MapReduce相反，这也决定了Spark比较适合运行低延迟的任务。在Spark中处于同一节点上的task以多线程的方式运行在一个executor进程中，构建了一个可重用的资源池，有如下特点：
>
> 1.每个executor单独运行在一个JVM进程中，每个task则是运行在executor中的一个线程。很显然线程线程级别的task启动速度更快
>
> 2.同一节点上所有task运行在一个executor中，有利于共享内存。比如通过Spark的广播变量，将某个文件广播到executor端，那么在这个executor中的task不用每个都拷贝一份处理，而只需处理这个executor持有的共有文件即可
>
> 3.executor所占资源不会在一些task运行结束后立即释放掉，可连续被多批任务使用，这避免了每个任务重复申请资源带来的开销
>
> **Spark的缺陷：**但是多线程模型有一个缺陷：同一节点的一个executor中多个task很容易出现资源征用。毕竟资源分配最细粒度是按照executor级别的，无法对运行在executor中的task做细粒度控制。**这也导致在运行一些超大数据量的任务并且资源比较有限时，运行不太稳定。相比较而言，MapReduce更有利于这种大任务的平稳运行。**

