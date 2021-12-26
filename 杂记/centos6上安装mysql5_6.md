#### 下载mysql包

> 去官网下载mysql包，下载redhat版的。因为centos和redhat的内核一致
>
> mysql8用起来有点问题，然后我又去下了5.6版本的

<img src="F:\学习笔记\rootImages\image-20201119122042883.png" alt="image-20201119122042883" style="zoom: 80%;" />



#### 安装客户端和服务端

``` bash
sudo rpm -ivh MySQL-server-5.6.50-1.el6.x86_64.rpm 
sudo rpm -ivh MySQL-client-5.6.50-1.el6.x86_64.rpm
```



#### 对mysql进行一些操作

``` bash
# 查看mysql的密码
cat /root/.mysql_secret

# 启动mysql
service mysql start

# 查看mysql运行状态
service mysql status

# 登录mysql，需要安装好了client后才能登录
mysql -uroot -pDFldTj3ceR1wxxTH

# 修改mysql的密码，从命令行进入mysql后执行
SET PASSWORD=PASSWORD('123456');

# 配置无主机登录，不然远程登录可能会有问题，首先我们要进入mysql这个数据库
# 通过select user,host,passwors from user;可以看这个表结构
updata user set host='%' where host='localhost';
delete from user where host='hadoop01';
delete from user where host='127.0.0.1';
delete from user where host='::1';

# 刷新mysql，使对user表的改动立即生效
flush privileges;
```



#### 遇见的问题

> 教学视频上演示的是mysql5.6，但我下载的是mysql8。所以导致安装后的结果不太一样，安装完成总是找不到mysql的密码，mysql也启动不了。
>
> 这肯定不是版本的原因，但是由于和视频上的版本不一致，我也不好确定错误在哪，所以我把mysql8卸载了，重写下载了个mysql5.6
>
> 我把mysql8卸载了后，安装mysql5.6。但还是一样的错误，经过长时间的百度，我发现视频和我本机有点区别，视频上使用rpm -e --nodeps命令就能将过去安装过的mysql卸载干净，但我的不行。我还要使用whereis mysql 和 find / -name mysql命令来将系统自带的mysql彻底删除
>
> 在将自带的mysql彻底删除后，我的mysql5.6就可以正常的运行了
>
> 所以说，在学习的过程中，尽可能的和所学视频中软件的版本保持一致