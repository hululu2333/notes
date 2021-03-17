### crontab执行命令的shell环境

> 引用：https://blog.csdn.net/weixin_30483495/article/details/97965175
>
> 参考：crontab比较详细的介绍：https://blog.csdn.net/weixin_36343850/article/details/79217611

> **我们手动调用脚本能成功，但是放在定时任务就不行的几种情况：**
>
> **环境变量不同：**crontab使用的环境变量和我们不同，具体可以在/etc/contab中配置。解决这个问题，我们可以在shell脚本开头指定shell环境，也可以在脚本中使用绝对路径
>
> **解释器不同：**有的命令bash能识别但sh不能识别，如source	((a++))。虽然你脚本开头指定了#!/bin/bash。但定时任务中如果指定了用sh去执行，那最后就是用sh去执行。
>
> **总结：**要保证定时器得到的结果和手动执行一致，我们得在shell中配置好环境变量，然后用shell中第一行指定的解释器去执行定时任务



### 背景

　　1，定时任务命令 crontab -e

　　2，默认的环境变量

```
SHELL=/bin/sh
PATH=/usr/bin:/bin
PWD=/home/owl
LANG=zh_CN.UTF-8
SHLVL=1
HOME=/home/owl
LANGUAGE=zh_CN:zh
LOGNAME=owl
_=/usr/bin/env
```

### 解决方法

　　一、使用绝对路径；

　　二、手动设置环境变量

　　　　在shell文件开头　　　　

```
PATH=/...
export PATH
```

　　三、批量设置环境变量

　　　1，在shell文件中执行2

　　　2，使用source指令执行shell文件

### 遇到的问题

　　问题：在crontab定时执行的shell文件中无法执行source指令

　　分析：在命令行中执行该shell文件正常，推测环境变量问题。

　　解决：1，尝试在运行source指令前，修改PATH环境变量，无效。

　　　　原因是，source是bash指令，其执行不依赖环境变量，只取决于shell的执行器。

　　　　2，修改SHELL环境变量为/bin/bash，执行成功。

### 总结

　　shell的执行器有bash、sh等

　　在shell的开头通过 #!/bin/sh或 #!/bin/bash 注明该shell的执行器。

　　手动执行方式 sh+shell文件 或bash+shell文件