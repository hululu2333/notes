### 搭建typecho博客

***

#### 安装宝塔面板

> 云服务器：华为云
>
> 操作系统：centos7.5
>
> yum install -y wget && wget -O install.sh http://download.bt.cn/install/install_6.0.sh && sh install.sh

> 安装完成之后的结果如下，在华为云的安全组中放行8888端口，然后在浏览器中使用外网面板地址登录
>
> http://114.116.250.197:8888/abb235cd

<img src="F:\学习笔记\搭建typecho博客\imgs\image-20201017230538662.png" alt="image-20201017230538662" style="zoom: 80%;" />



>选择LNMP安装套件

<img src="F:\学习笔记\搭建typecho博客\imgs\image-20201017231417054.png" alt="image-20201017231417054" style="zoom:80%;" />



#### 创建站点

> 点击网站界面的添加站点

<img src="F:\学习笔记\搭建typecho博客\imgs\image-20201018184456693.png" alt="image-20201018184456693"  />

> 相关用户名和密码
>
> b6yGKKD8yTtiiw5s
>
> 4NmG3n46ry3n6hSG

![image-20201018184639702](F:\学习笔记\搭建typecho博客\imgs\账号信息.png)



#### 下载并配置typecho

![image-20201018185437436](F:\学习笔记\搭建typecho博客\imgs\image-20201018185437436.png)



> 将安装包上传到宝塔面板中我们网站的根目录

![image-20201018190523087](C:\Users\hu\AppData\Roaming\Typora\typora-user-images\image-20201018190523087.png)

> 解压压缩包，并将解压缩后的文件夹中的文件全都复制到网站根目录



> 在浏览器中输入www.hujiaxiang.xyz/install.php，进入typecho的安装界面
>
> 前提是你要把你的域名解析到对应的ip地址，我的域名是通过阿里云买的，进入阿里云的域名控制台设置

![image-20201018193932726](F:\学习笔记\搭建typecho博客\imgs\image-20201018193932726.png)



> 初始化配置时，会显示无法连接至数据库。因为这时的默认数据库名错了，我们把他改为www_hujiaxiang_x

![image-20201018195435608](C:\Users\hu\AppData\Roaming\Typora\typora-user-images\image-20201018195435608.png)



#### 安装完成

![image-20201018195518790](C:\Users\hu\AppData\Roaming\Typora\typora-user-images\image-20201018195518790.png)



#### 如何使用typecho

http://docs.typecho.org/doku.php

或者点击进入后台按钮

![image-20201018203607959](C:\Users\hu\AppData\Roaming\Typora\typora-user-images\image-20201018203607959.png)

