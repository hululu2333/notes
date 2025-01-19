## 安装并配置kerberos

> 以下是在CDH6.3.2中对Kerberos进行安装和配置
>
> 参考：https://blog.csdn.net/ytp552200ytp/article/details/109643832
>
> https://www.bilibili.com/video/BV1pV411k7ut?p=4（视频，比较全）

> 目前进展到初始化KDC数据库（初始化KDC数据库的命令找不到）



03 PART
Kerberos部署

1.安装Kerberos相关服务

       选择集群中的一台主机（hadoop102.example.com）作为Kerberos服务端，安装KDC，所有主机都需要部署Kerberos客户端。

服务端主机执行以下安装命令

yum install -y krb5-server krb5-workstation krb5-libs
客户端主机执行以下安装命令

yum install -y krb5-workstation krb5-libs
2.修改配置文件

（1）服务端主机（hadoop102.example.com）

修改/var/kerberos/krb5kdc/kdc.conf文件，内容如下

[kdcdefaults]
 kdc_ports = 88
 kdc_tcp_ports = 88

[realms]
 EXAMPLE.COM = {
  #master_key_type = aes256-cts
  acl_file = /var/kerberos/krb5kdc/kadm5.acl
  dict_file = /usr/share/dict/words
  max_life = 1d
  max_renewable_life = 7d
  admin_keytab = /var/kerberos/krb5kdc/kadm5.keytab
  supported_enctypes = aes128-cts:normal des3-hmac-sha1:normal arcfour-hmac:normal camellia256-cts:normal camellia128-cts:normal des-hmac-sha1:normal des-cbc-md5:normal des-cbc-crc:normal
 }
（2）客户端主机（所有主机）

修改/etc/krb5.conf文件，内容如下

# Configuration snippets may be placed in this directory as well
includedir /etc/krb5.conf.d/

[logging]
 default = FILE:/var/log/krb5libs.log
 kdc = FILE:/var/log/krb5kdc.log
 admin_server = FILE:/var/log/kadmind.log

[libdefaults]
 dns_lookup_realm = false
 ticket_lifetime = 24h
 renew_lifetime = 7d
 forwardable = true
 rdns = false
 pkinit_anchors = FILE:/etc/pki/tls/certs/ca-bundle.crt
 default_realm = EXAMPLE.COM
 #default_ccache_name = KEYRING:persistent:%{uid}
 udp_preference_limit = 1
[realms]
 EXAMPLE.COM = {
  kdc = hadoop102.example.com
  admin_server = hadoop102.example.com
 }

[domain_realm]
 .example.com = EXAMPLE.COM
# example.com = EXAMPLE.COM
3.初始化KDC数据库

在服务端主机（hadoop102.example.com）执行以下命令

kdb5_utilcreate -s
4.修改管理员权限配置文件

在服务端主机（hadoop102.example.com）修改/var/kerberos/krb5kdc/kadm5.acl文件，内容如下

*/admin@EXAMPLE.COM     *
5.启动Kerberos相关服务

启动KDC

systemctl start krb5kdc
启动Kadmin，该服务为KDC数据库访问入口

systemctl start kadmin
04PART
CDH集群启用Kerberos

１.为CM创建Kerberos管理员主体

kadmin.local -q "addprinc cloudera-scm/admin"
２.启用Kerberos


3.环境确认（勾选全部）


4.填写KDC配置


5.继续

6.填写主体名和密码

7.等待导入KDC

8.重启集群

9.完毕


05 PART
Kerberos安全环境使用

在集群启用Kerberos之后，用户访问各服务都需要先通过Kerberos认证。下面通过访问HFDS，Hive演示具体操作方式。

1.为用户向Kerberos注册账号（Principal）

在Kerberos服务端主机（hadoop102.example.com）执行以下命令，并输入密码，完成注册

kadmin.local -q "addprinc hdfs/hdfs@EXAMPLE.COM"
2.用户认证，执行以下命令，并输入密码，完成认证

kinit hdfs/hdfs@EXAMPLE.COM
查看当前认证状态

$ klist

Ticket cache: FILE:/tmp/krb5cc_0
Default principal: hdfs/hdfs@EXAMPLE.COM

Valid starting       Expires              Service principal
11/05/2020 14:29:23  11/06/2020 14:29:23  krbtgt/EXAMPLE.COM@EXAMPLE.COM
  renew until 11/12/2020 14:29:23
注：如需在非交互环境认证，例如在代码中认证，则可通过以下命令生成密钥文件，在代码中指定密钥文件路径即可。需要注意的是，生成密钥文件之后，密码随之失效。

kadmin.local -q "xst -k /path/to/your/keytab/admin.keytab hdfs/hdfs@EXAMPLE.COM"
3.访问HDFS

认证前

$ hadoop fs -ls /

20/11/05 14:28:28 WARN ipc.Client: Exception encountered while connecting to the server : org.apache.hadoop.security.AccessControlException: Client cannot authenticate via:[TOKEN, KERBEROS]
ls: Failed on local exception: java.io.IOException: org.apache.hadoop.security.AccessCont
认证后

$ hadoop fs -ls /
Found 2 items

drwxrwxrwt   - hdfs    supergroup          0 2020-11-02 15:52 /tmp
drwxr-xr-x   - hdfs    supergroup          0 2020-11-03 09:23 /user
4.访问Hive

（1）hive客户端

认证前

$ hive

Exception in thread "main" java.lang.RuntimeException: java.io.IOException: Failed on local exception: java.io.IOException: org.apache.hadoop.security.AccessControlException: Client cannot authenticate via:[TOKEN, KERBEROS]; Host Details : local host is: "hadoop102.example.com/172.26.131.1"; destination host is: "hadoop102.example.com":8020; 
  at org.apache.hadoop.hive.ql.session.SessionState.start(SessionState.java:604)
  at org.apache.hadoop.hive.ql.session.SessionState.beginStart(SessionState.java:545)
认证后

$ hive

WARNING: Hive CLI is deprecated and migration to Beeline is recommended.
hive>
（2）beeline客户端

注：开启Kerberos之后，jdbc的url需增加hiveserver2的principal信息，如下

认证前

$ beeline -u " jdbc:hive2://hadoop102.example.com:10000/;principal=hive/hadoop102.example.com@EXAMPLE.COM"

Connecting to jdbc:hive2://hadoop102.example.com:10000/;principal=hive/hadoop102.example.com@EXAMPLE.COM
20/11/05 14:42:57 [main]: ERROR transport.TSaslTransport: SASL negotiation failure
javax.security.sasl.SaslException: GSS initiate failed
  at com.sun.security.sasl.gsskerb.GssKrb5Client.evaluateChallenge(GssKrb5Client.java:2
认证后

$ beeline -u " jdbc:hive2://hadoop102.example.com:10000/;principal=hive/hadoop102.example.com@EXAMPLE.COM"

Connecting to jdbc:hive2://hadoop102.example.com:10000/;principal=hive/hadoop102.example.com@EXAMPLE.COM
Connected to: Apache Hive (version 2.1.1-cdh6.3.2)
Driver: Hive JDBC (version 2.1.1-cdh6.3.2)
Transaction isolation: TRANSACTION_REPEATABLE_READ
Beeline version 2.1.1-cdh6.3.2 by Apache Hive
0: jdbc:hive2://hadoop102.example.com:10000/>
————————————————
版权声明：本文为CSDN博主「贝拉美」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/ytp552200ytp/article/details/109643832