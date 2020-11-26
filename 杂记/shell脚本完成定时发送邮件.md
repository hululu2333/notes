### shell脚本定时发送邮件

***

#### 环境

> centos7.7
>
> mailx



#### 前期准备

> 关闭centos7自带的邮件工具，并安装mailx

``` bash
chkconfig postfix off   
service postfix stop

yum install mailx -y
```



#### 配置qq邮箱

> 将两个SMTP服务都开启，会得到两个授权码
>
> POP3/SMTP服务授权码：mskkmybypfgabdai
>
> IMAP/SMTP服务授权码：cuenhqvbjcglbfgb

![image-20201124152458823](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201124152458823.png)



#### 配置mail.rc

> vi /etc/mail.rc
>
> 添加以下内容

``` properties
set from=1030499842@qq.com    #你的qq邮箱
set smtp=smtps://smtp.qq.com:465   #邮箱所在服务器和端口地址
set smtp-auth-user=1030499842@qq.com    #你的qq邮箱
set smtp-auth-password=mskkmybypfgabdai #这部分要划下重点，这里是配置授权码，而不是邮箱的独立密码，否则发送邮件时报错（smtp-server: 535 Error: Ȩ¼http://service.mail.qq.com/cgi-bin/help?subtype=1&&id=28&&no=1001256）
set smtp-auth=login    #默认login即可
set ssl-verify=ignore    #ssl认证方式
set nss-config-dir=/root/.certs    #证书所在目录，这个可以自定义目录所在位置
```



#### 请求数字证书

``` bash
mkdir -p /root/.certs/

echo -n | openssl s_client -connect smtp.qq.com:465 | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > ~/.certs/qq.crt

certutil -A -n "GeoTrust SSL CA" -t "C,," -d ~/.certs -i ~/.certs/qq.crt

certutil -A -n "GeoTrust Global CA" -t "C,," -d ~/.certs -i ~/.certs/qq.crt

cd  /root/.certs/

certutil -A -n "GeoTrust SSL CA - G3" -t "Pu,Pu,Pu" -d ~/.certs/./ -i qq.crt

certutil -L -d /root/.certs
```



#### 发送邮件的几种格式

> 配置完以上几步，就能通过mail命令发送邮件了

``` bash
cat file.txt | mail -s "邮件主题" xxx@163.com

mail -s "邮件主题" xxx@163.com < file.txt

echo "邮件正文" | mail -s "邮件主题" xxx@163.com
```



#### 编写邮件告警的脚本

``` bash
#!/bin/bash

# 用来装待检测文件夹的数组
folders=()
index=0
flag=-3

# 如果用crontab执行的话，命令必须绝对路径，我猜测用crontab执行时没有读取环境变量
# 将flume文件夹下的文件夹名加入待检测数组
for each in $(/hadoop/hadoop-3.1.3/bin/hdfs dfs -ls /flume)
do
	((flag++))
	if (($flag%8==0&&$flag>0))
    	then 
    	folders[index]=$each
	((index++))
    	fi
done

# 将data文件夹下的文件夹名加入待检测数组
flag=-3
for each in $(/hadoop/hadoop-3.1.3/bin/hdfs dfs -ls /data)
do
	((flag++))
        if (($flag%8==0&&$flag>0))
        then
        folders[index]=$each
        ((index++))
        fi
done

#info=$(hdfs dfs -du -s -h /flume/2020-09-27)

# 用crontab执行时，必须写文件的绝对路径，我猜测是因为crontab执行的时候和我们手动执行的时候路径不同。我们手动执行是在命令的所在位置执行
# 遍历待检测的文件夹路径,将大于100m的文件夹的路径重定向到文件中
for each in ${folders[@]}
do
	info=$(/hadoop/hadoop-3.1.3/bin/hdfs dfs -du -s -h $each)
	temp=($info)

	echo ${temp[0]} >> /root/bin/space_warning.log
	echo ${temp[1]} >> /root/bin/space_warning.log
	# 将大于100m的路径和大小输出到一个文件中
	if [ ${temp[1]} == M ]
	then
		if [ $(echo "${temp[0]} >= 100"|bc) = 1 ]
		then
			result=${temp[4]}"->"${temp[0]}${temp[1]}
			echo $result >> /root/bin/space_warning.log
			echo $result >> /root/bin/result.txt
		fi
	fi

	# 将以大小G结尾的文件夹的路径直接重定向到文件中
	if [ ${temp[1]} == G ]
	then
		result=${temp[4]}"->"${temp[0]}${temp[1]}
		echo $result >> /root/bin/space_warning.log
                echo $result >> /root/bin/result.txt
	fi

done

# 将文件内容发到指定邮箱，并将这个文件清空
mail -s "邮件告警" 2393957890@qq.com < /root/bin/result.txt
echo "邮件已发送" >> /root/bin/space_warning.log

echo "" > /root/bin/result.txt
```



#### 配置定时任务

> crontab的常用命令和格式

``` bash
# 编辑某个用户的定时任务
crontab -e

# 列出某个用户cron服务的详细内容
crontab -l

# 删除某个用户的cron服务
crontab -r

# 在每天的14时13分执行，用bash解释器执行space_warning.sh脚本
13 14 * * * /bin/bash /root/bin/space_warning.sh
# 第四位是表示每年的第几个月，*号表示每年的每个月都发送。第五位是表示每周的周几
# 第三位是表示每个月的几号，第二位是每天的几点，第一位是每时的第几分钟
```







> 查看hdfs中文件夹大小的命令：hdfs dfs -du -s -h /flume/2020-09-27
>
> 查看的结果：429.4 K  858.8 K  /flume/2020-09-27
>
> 超过100m发送邮件到hai.liu@ziyun-cloud.com（可以先发送到2393957890@qq.com）
>
