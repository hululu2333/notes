``` java
2021/10/21 03:08:49 [WARN ] - Lost task 194.0 in stage 32.1 (TID 125696, 10-90-48-60-jhdxyjd.mob.local, executor 239): TaskCommitDenied (Driver denied task commit) for job: 32, partition: 264, attemptNumber: 0
```

> 开启speculation参数后，运行较慢的task会在其他executor上同时再启动一个相同的task,如果其中一个task执行完毕，相同的另一个task就会被禁止提交。因此产生了这个WARN。
>
> 这个WARN是因为task提交commit被driver拒绝引发，这个错误不会被统计在stage的failure中，这样做的目的是防止你看到一些具有欺骗性的提示。



``` java
org.apache.spark.shuffle.MetadataFetchFailedException: Missing an output location for shuffle 10
```

> shuffle read时向堆外内存中写？然后发现没地方写了？所以报错？



``` java
2021/10/21 03:15:50 [ERROR] - RECEIVED SIGNAL TERM
```



``` java
java.lang.OutOfMemoryError: Java heap space		
java.lang.OutOfMemoryError: GC overhead limit execeeded
```

> 堆内内存溢出和堆外溢出

