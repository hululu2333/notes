### 使用Flume监控本地多个文件夹中的文件

***

> files-flume-hdfs

```bash
#将source的配置改为以下示例
#source的类型是taildir source，具有断点续传的功能
a1.sources.r1.type = TAILDIR

#指定文件组的个数，并取一个名称
a1.sources.r1.filegroups = f1 f2

#指定positionFile的位置
a1.sources.r1.positionFile = /opt/module/flume/position.json
#这里面存的是source读取到了文件的哪个位置，如果agent进程中断了，根据这个文件，就可以从上次结束的地方继续读取

#指定名为f1的文件组具体包含哪些文件
a1.sources.r1.filegroups.f1 = /opt/module/flume/files/app*.log

#指定名为f2的文件组包含哪些文件
a1.sources.r1.filegroups.f2 = /opt/module/flume/logs/app*.log
```

