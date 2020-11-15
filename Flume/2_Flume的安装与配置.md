### Flume的安装与配置

***

#### 解压Flume

``` bash
tar -zxvf apache-flume-1.7.0-bin.tar.gz -C /opt/module
```



#### 修改Flume配置文件

##### 修改flume-env.sh文件——指定JAVA_HOME

```bash
#Agent是一个JVM进程，所以需要知道jdk的安装路径
export JAVA_HOME=/opt/module/jdk
```



