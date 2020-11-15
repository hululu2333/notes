### 编写Zookeeper一键启动，关闭的脚本

``` bash
#!/bin/bash

for i in hadoop01 hadoop02 hadoop03
do
	ssh $i "/opt/module/zookeeper/bin/zkServer.sh $1"
	#$1指的是使用这个脚本时的第一个参数，如 zk stop，那$1指的就是stop 
done
```

