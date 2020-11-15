### 为什么把环境变量配置在profile.d中？——登录式shell与非登录式shell

***

#### 登录式shell

> 当你通过用户名和密码登录时，这就是登录式shell。这时系统会自动应用/etc/profile文件。



#### 非登录时shell

> 当你通过ssh远程连接时，如ssh hadoop02。你用hadoop用户的身份登陆了hadoop02，但这时/etc/profile没有没应用。
>
> 这种情况下只有家目录的.bashrc会被应用，也就是说，定义在/etc/profile中的环境变量你现在用不到了。
>
> .bashrc中有一串代码，他会运行/etc/profile.d/下的所有以sh结尾的脚本。所以我们把环境变量定义在/etc/profile.d/env.sh中

