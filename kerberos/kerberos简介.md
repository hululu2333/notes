## kerberos简介

> kerberos主要用来做网络通讯中的身份认证，帮我们高效安全的识别访问者



### Kerberos的一些关键组件

> **Key Distribution Center(KDC)：**密钥分发中心，也就是Kerberos Server，在KDC这个服务中，他包含了AS和TGS这两个服务和一个数据库
>
> **authorization server(AS)：**授权服务，用户向AS发送一个用密码加密后的请求（这个请求中包含了我是谁，我要访问什么），AS用提供的密码对请求进行解密后得到请求内容，然后返回给用户一个TGT（ticket granting tickets）（TGT被密钥加密了）
>
> **Ticket Granting Server（TGS）：**票授予服务，TGS验证TGT后（使用密钥解密）返回一个Ticket给用户
>
> **Principal：**主体，用于在kerberos加密系统中标记一个唯一的身份。主体可以是用户（如zhangsan）或服务（如namenode或hive）。



### kerberos的认证流程

> 1. client向授权认证中心（AS）发送一个访问某服务的请求（这个请求被client用密码加密），然后授权认证中心返回给client一个临时票据
>
> 2. client拿着临时票据，去票据中心(TGS)换得一个真实的票据（ticket）
> 3. client拿着ticket去访问服务（假设是访问hdfs）。name node收到这个票据后先去和kerberos验证票据的合法性。验证通过后，就允许client继续访问
> 4. 下一次client还想访问hdfs时，还得带上这张ticket。namenode会去验证这张ticket是否过期了，没过期就可以直接用，过期了就得去重新申请一张



### Kerberos认证流程2

> （１）客户端执行kinit命令，输入Principal及Password，向AS证明身份，并请求获取TGT。
> （２）AS检查Database中是否存有客户端输入的Principal，如有则向客户端返回TGT。
> （３）客户端获取TGT后，向TGS请求ServerTicket。
> （４）TGS收到请求，检查Database中是否存有客户端所请求服务的Principal，如有则向客户端返回ServerTicket。
> （５）客户端收到ServerTicket，则向目标服务发起请求。
> （６）目标服务收到请求，响应客户端。
>