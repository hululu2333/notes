## 修改解释器的属性

> 无论是执行shell脚本还是在shell窗口输入shell命令，最终都需要解释器去执行。我一般使用的是bash，我们可以通过set -o或set +o的方式开启或关闭bash解释器的一些特征，从而能够控制解释器的一些行为



### set的几种用法

> 假设以下都是利用bash解释器进行解释

> **直接用set：**我们直接输入set时，解释器会返回当前shell环境中的所有变量和函数。当我们用crontab执行脚本时，往往会得到与我们手动执行脚本不同的结果，这时我们可以用set来判断两者的shell环境是否相同
>
> **set -o：**将解释器中的某个属性置为开启的状态，如set -o history 可以打开解释器的历史记录功能。最近用bash解释的shell命令都会被保存下来，然后可以利用history命令进行查看
>
> **set +o：**将解释器中的某个属性置为关闭的状态，如set +o xtrace 可以关闭shell命令的自身打印（如果将xtrace设置为开启状态，则每执行一行shell命令前，都会将这行shell命令打印一遍，比较适合生成shell脚本的日志）
>
> **缩写：**set -o xtrace可以缩写为set -x。因为解释器的属性中只有一个以x开头的属性。同样，关闭这个属性可以写为set +x。    解释器的属性中有两个以v开头的属性，但我们使用set -v 时，开启的是verbose这个属性而不是另一个，具体的选择机制还不明确



### 解释器的属性列表

> 目前只知道两个属性的作用

> allexport      	off
> braceexpand    	on
> emacs          	on
> errexit        	off
> errtrace       	off
> functrace      	off
> hashall        	on
> histexpand     	on
> history        	记录以往的shell命令
> ignoreeof      	off
> interactive-comments	on
> keyword        	off
> monitor        	on
> noclobber      	off
> noexec         	off
> noglob         	off
> nolog          	off
> notify         	off
> nounset        	off
> onecmd         	off
> physical       	off
> pipefail       	off
> posix          	off
> privileged     	off
> verbose        	off
> vi             	off
> xtrace         	执行一行shell命令前，将这行shell命令打印出来