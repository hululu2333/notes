## Integer详解

> Interger是一个引用类型，但是java为了性能对其进行了一些优化导致和其它的引用类型有些不同。所有包装类的类型因该都有类似的特征。



### 核心值

>在Integer的源码中，核心值是这样定义的`private final int value;`
>
>Integer对象中的值被fanal修饰，是不能修改的，所以也不提供set方法。当我们试图用`a=1;`来给Integer对象a来换一个值时，实际上是调用了`Integer.valueOf()`方法获得了一个新对象。



### cache缓存

> 为了减小内存消耗，Integer类中有一个静态内部类`IntegerCache`

![image-20210611114252776](E:\notes\java\images\IntegerCache.png)

> 静态类中有个静态属性cache，在静态初始化代码块中会给这个数组赋值。数组中会包含值为-128~127的Integer对象。
>
> 当我们不使用new的方式来创建Integer对象时，会通过Integer.valueof()方法先判定cache中有没有相同值的Integer对象。如当我们执行`Integer a=1;Integer b=1;`时，发现cache中存在值为1的Integer对象，就把这个对象的引用返回给a和b，而不会在堆中创建新的对象。当然，此时a和b指向的是同一个对象。
>
> 当我们采用new的方式创建一个Integer对象，或我们赋的值不在-128~127之间，就会在堆中创建新的对象。



### 通过反射修改cache中的值

> 当我们执行`Intger a=23;`时，这时会把cache中下标为151的值赋给a
>
> 当我们通过反射修改a的值，会把cache中的值给修改掉

![image-20210611120018482](C:\Users\hujiaxiang\AppData\Roaming\Typora\typora-user-images\image-20210611120018482.png)