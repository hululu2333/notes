### linux条件判断：eq、ne、gt、lt、ge、le



条件判断：

​    如果用户不存在

​       添加用户，给密码并显示添加成功；

​    否则

​       显示如果已经没在，没有添加； 





整数比较:

​    ***\*-eq(equal)\****   : 测试两个整数是否相等；比如 $A -eq $B

​    ***\*-ne\*\*(\*\*inequality\*******\*)\**** : 测试两个整数是否不等；不等，为真；相等，为假；

​    ***\*-gt(greter than)\**** : 测试一个数是否大于另一个数；大于，为真；否则，为假；

​    ***\*-lt\*******\*(less than)\**** : 测试一个数是否小于另一个数；小于，为真；否则，为假；

​    ***\*-ge\*\*(greter equal)\*\**\***: 大于或等于

​     ***\*-le\*******\*(less equal)\**** ：小于或等于   

命令的间逻辑关系：

​    逻辑与： &&

​       第一个条件为假时，第二条件不用再判断，最终结果已经有；

​       第一个条件为真时，第二条件必须得判断；

​    逻辑或： ||  



*4*．命令实例： 

如果用户user6不存在，就添加用户user6

! id user6 && useradd user6

id user6 || useradd user6

如果/etc/inittab文件的行数大于100，就显示好大的文件；

[ `wc -l /etc/inittab | cut -d' ' -f1` -gt 100 ] && echo"Large file." 

如果用户存在，就显示用户已存在；否则，就添加此用户；

id user1 && echo "user1 exists." || useradd user1

 

如果用户不存在，就添加；否则，显示其已经存在；

! id user1 && useradd user1 || echo "user1 exists."

 

如果用户不存在，添加并且给密码；否则，显示其已经存在；

! id user1 && useradd user1 && echo "user1" |passwd --stdin user1  || echo "user1exists."