### 安装并使用git

***

> 目前常用的版本控制工具有git，svn等
>
> git是分布式版本控制工具，svn是集中式版本控制工具



#### 下载git

> 在官网下git可能会十分的慢，我们可以去淘宝的镜像网站	http://npm.taobao.org/mirrors/git-for-windows
>
> 进去后下载最新版git	2.29.0
>
> 安装时一直点下一步即可



#### 配置git

![image-20201020221623827](C:\Users\hu\AppData\Roaming\Typora\typora-user-images\image-20201020221623827.png)

``` bash
#在用户配置中加一个user.name的属性，它的值为bighu
git config --global user.name "bighu"

#配置自己的邮箱
git config --global user.email "1030499842@qq.com"
```



#### git的工作区域

![image-20201020223130203](C:\Users\hu\AppData\Roaming\Typora\typora-user-images\image-20201020223130203.png)



#### git的工作流程

![image-20201020223459146](C:\Users\hu\AppData\Roaming\Typora\typora-user-images\image-20201020223459146.png)



#### git的常用命令

``` bash
#在当前目录新建一个本地的git仓库
git init
#.git文件夹就是本地仓库

#从支持git的平台克隆一个项目和它的整个代码历史（版本信息）
git clone https://github.com/zhang0peter/bilibili-video-information-spider.git

#将所有文件提交到暂存区
git add .

#将暂存区文件提交到本地仓库，并指定提交信息
git commit -m "new file hello.txt"
```

![image-20201024121603709](C:\Users\hu\AppData\Roaming\Typora\typora-user-images\image-20201024121603709.png)



#### git文件的几种状态

![image-20201020231521923](C:\Users\hu\AppData\Roaming\Typora\typora-user-images\image-20201020231521923.png)



#### 查看文件状态

``` bash
#查看指定文件的状态
git status [filename]

#查看所有文件的状态
git status
```



#### 忽略文件

![image-20201021220725597](C:\Users\hu\AppData\Roaming\Typora\typora-user-images\image-20201021220725597.png)

> 有时你的工作区中有的文件不需要被纳入版本控制
>
> 这时我们可以在**主目录**中建立.gitignore文件，并写入以下规则

> 一定要在主目录下建立.gitignore文件，我之前在.idea文件夹下创建，结果修改这个.gitignore文件只能操作.idea文件夹下的文件



#### 如过在idea中应用git

> 1. 用idea新建一个项目
> 2. 用git的clone命令将github上的项目拉取到本地
> 3. 将git拉取过来的文件夹中的文件全部剪切到idea新建的项目目录中



> 在这之前，我们因该在本地电脑和远程仓库间配置免密登录



> 到此为止，idea的项目目录就是我们的工作目录，.git目录就是本地仓库。我们可以通过git命令来进行一些操作了