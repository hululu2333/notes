### 使用logrotate对日志进行切割

***

> logrotate是linux的一个日志切割工具，大部分发行版的linux都会自带这个工具



#### logrotate简介

> logrotate用来切割和管理日志文件，防止日志文件太大，导致磁盘空间不足
>
> 我们想使用logrotate时，要先写一个配置文件，然后运行logrotate时指定相应的配置文件
>
> logrotate一般和crontab一起配合使用



#### centos7.6自带的logrotate

> logrotate自带的一个配置文件是`/etc/logrotate.conf`，在`/etc/cron.daily`下会有一个脚本调用logrotate任务并指定这个配置文件。所以系统默认每天都会执行一次logrotate
>
> `/etc/logrotate.conf`中有一行代码是`include /etc/logrotate.d`，所以放在`/etc/logrotate.d`中的配置文件，也会被每天调用一次



#### logrotate原理

> logrotate 是怎么做到滚动日志时不影响程序正常的日志输出呢
>
> logrotate 提供了两种解决方案。 create和copytruncate，具体实现看参考链接



#### logrotate的配置文件

> 执行文件：`/usr/sbin/logrotate`  logrotate本质上是一个可执行文件，每次要切割日志文件时，只需要调用这个可执行文件并指定配置文件即可
>
> 主配置文件: `/etc/logrotate.conf` 
>
> 自定义配置文件: `/etc/logrotate.d/*.conf`	主配置文件中会引入`/etc/logrotate.d`，所以放在这的自定义配置文件每天也会被执行一次

> 主配置文件示例

``` conf
# see "man logrotate" for details
# rotate log files weekly
weekly

# keep 4 weeks worth of backlogs
rotate 4

# create new (empty) log files after rotating old ones
create

# use date as a suffix of the rotated file
dateext

# uncomment this if you want your log files compressed
#compress

# RPM packages drop log rotation information into this directory
include /etc/logrotate.d

# no packages own wtmp and btmp -- we'll rotate them here
/var/log/wtmp {
    monthly
    create 0664 root utmp
    minsize 1M
    rotate 1
}

/var/log/btmp {
    missingok
    monthly
    create 0600 root utmp
    rotate 1
}

# system-specific logs may be also be configured here.
```



#### 运行logrotate

> ```text
> logrotate [OPTION...] <configfile>
> -d, --debug ：debug 模式，测试配置文件是否有错误。使用debug模式就是模拟运行一次程序，但不修改文件
> -f, --force ：强制转储文件。
> -m, --mail=command ：压缩日志后，发送日志到指定邮箱。
> -s, --state=statefile ：使用指定的状态文件。
> -v, --verbose ：显示转储过程。
> ```

> 运行logrotate可以通过crontab或者直接手动运行
>
> 如：`/usr/sbin/logrotate /etc/logrotate.d/rsyslog` ，手动执行一个logrotate任务



#### 如何写一个logrotate配置文件

> 一个案例，更多的案例见链接

```text
/var/log/log_file {
    size=50M
    rotate 5
    dateext
    create 644 root root
    postrotate
        /usr/bin/killall -HUP rsyslogd
    endscript
}
```

在上面的配置文件中，我们只想要轮询一个日志文件，size=50M 指定日志文件大小可以增长到 50MB,dateext 指 示让旧日志文件以创建日期命名。

> 常见配置参数

``` text 
- daily ：指定转储周期为每天
- weekly ：指定转储周期为每周
- monthly ：指定转储周期为每月
- rotate count ：指定日志文件删除之前转储的次数，0 指没有备份，5 指保留 5 个备份
- tabooext [+] list：让 logrotate 不转储指定扩展名的文件，缺省的扩展名是：.rpm-orig, .rpmsave, v, 和～
- missingok：在日志轮循期间，任何错误将被忽略，例如 “文件无法找到” 之类的错误。
- size size：当日志文件到达指定的大小时才转储，bytes (缺省) 及 KB (sizek) 或 MB (sizem)
- compress： 通过 gzip 压缩转储以后的日志
- nocompress： 不压缩
- copytruncate：用于还在打开中的日志文件，把当前日志备份并截断
- nocopytruncate： 备份日志文件但是不截断
- create mode owner group ： 转储文件，使用指定的文件模式创建新的日志文件
- nocreate： 不建立新的日志文件
- delaycompress： 和 compress 一起使用时，转储的日志文件到下一次转储时才压缩
- nodelaycompress： 覆盖 delaycompress 选项，转储同时压缩。
- errors address ： 专储时的错误信息发送到指定的 Email 地址
- ifempty ：即使是空文件也转储，这个是 logrotate 的缺省选项。
- notifempty ：如果是空文件的话，不转储
- mail address ： 把转储的日志文件发送到指定的 E-mail 地址
- nomail ： 转储时不发送日志文件
- olddir directory：储后的日志文件放入指定的目录，必须和当前日志文件在同一个文件系统
- noolddir： 转储后的日志文件和当前日志文件放在同一个目录下
- prerotate/endscript： 在转储以前需要执行的命令可以放入这个对，这两个关键字必须单独成行
```



#### 我的案例

``` conf
/data/software/druid/apache-druid-0.15.1-incubating/var/sv/historical.log {
    size=100M
    rotate 4
    dateext
}
```





原理和使用方法：https://zhuanlan.zhihu.com/p/90507023