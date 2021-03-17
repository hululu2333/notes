### shell脚本中的$# $0 $@ $* $$ $! $?的意义

转载自：http://www.cnblogs.com/davygeek/p/5670212.html

| 变量 | 含义                                                         |
| ---- | ------------------------------------------------------------ |
| $0   | 当前脚本的文件名                                             |
| $n   | 传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是$1，第二个参数是$2 |
| $#   | 传递给脚本或函数的参数个数                                   |
| $*   | 传递给脚本或函数的所有参数                                   |
| $@   | 传递给脚本或函数的所有参数。被双引号(" ")包含时，与 $* 稍有不同 |
| $?   | 上个命令的退出状态，或函数的返回值                           |
| $$   | 当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID   |

 

 

$* 和 $@ 的区别

$* 和 $@ 都表示传递给函数或脚本的所有参数，不被双引号(" ")包含时，都以"$1" "$2" … "$n" 的形式输出所有参数。但是当它们被双引号(" ")包含时，"$*" 会将所有的参数作为一个整体，以"$1 $2 … $n"的形式输出所有参数；"$@" 会将各个参数分开，以"$1" "$2" … "$n" 的形式输出所有参数。

例：

创建ex4-extend.sh



```bash
#!/bin/sh
echo "\$*=" $*
echo "\"\$*\"=" "$*"
echo "\$@=" $@
echo "\"\$@\"=" "$@"
echo "print each param from \$*"
for var in $*
do
echo "$var"
done
echo "print each param from \$@"
for var in $@
do
echo "$var"
done
echo "print each param from \"\$*\""
for var in "$*"
do 
echo "$var"
done
echo "print each param from \"\$@\""
for var in "$@"
do
echo "$var"
done
```



执行./ex4-extend.sh "a" "b" "c" "d"，结果如下

```bash
$*= a b c d
"$*"= a b c d
$@= a b c d
"$@"= a b c d
print each param from $*
a
b
c
d
print each param from $@
a
b
c
d
print each param from "$*"
a b c d
print each param from "$@"
a
b
c
d
```



> 退出状态

$? 可以获取上一个命令的退出状态。所谓退出状态，就是上一个命令执行后的返回结果。
退出状态是一个数字，一般情况下，大部分命令执行成功会返回 0，失败返回 1。
不过，也有一些命令返回其他值，表示不同类型的错误。