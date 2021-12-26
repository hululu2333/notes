### sql的常用函数

***

``` sql
#将date类型转换为指定格式的字符串类型，只在oracle中测试过
to_char(day,'yyyy-mm-dd ')

#将字符串转换为指定格式的Date类型，只在oracle中测试过
to_date('2020-1-11' , 'yyyy-mm-dd hh24:mi:ss')

#截取字符串的第一个到第七个字符
SUBSTR('asfaadafaasdasd',1,7)

```





参考：https://www.cnblogs.com/winter-bamboo/p/10779466.html