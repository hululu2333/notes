### shell编程中的表达式(-d, -f, -L 等)

***



#### 文件表达式

> -e filename 如果 filename存在，则为真
> -d filename 如果 filename为目录，则为真 
> -f filename 如果 filename为常规文件，则为真
> -L filename 如果 filename为符号链接，则为真
> -r filename 如果 filename可读，则为真 
> -w filename 如果 filename可写，则为真 
> -x filename 如果 filename可执行，则为真
> -s filename 如果文件长度不为0，则为真
> -h filename 如果文件是软链接，则为真
> filename1 -nt filename2 如果 filename1比 filename2新，则为真。
> filename1 -ot filename2 如果 filename1比 filename2旧，则为真。



#### 字符串变量表达式

> If  [ $a = $b ]         如果string1等于string2，则为真
>                 字符串允许使用赋值号做等号
> if  [ $string1 !=  $string2 ]  如果string1不等于string2，则为真    
> if  [ -n $string  ]       如果string 非空(非0），返回0(true)  
> if  [ -z $string  ]       如果string 为空，则为真
> if  [ $sting ]          如果string 非空，返回0 (和-n类似) 

>  逻辑非 !          条件表达式的相反
> if [ ! 表达式 ]
> if [ ! -d $num ]        如果不存在目录$num

>  逻辑与 –a          条件表达式的并列
> if [ 表达式1  –a  表达式2 ]

> 逻辑或 -o          条件表达式的或
> if [ 表达式1  –o 表达式2 ]