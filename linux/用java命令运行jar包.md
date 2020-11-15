### 用java命令运行jar包

> 利用java命令来运行jar包有两种方式。
> 一是java -jar test.jar。这种方法只接收一个参数。主类的全类名的这个参数由系统从MANIFEST.MF文件中的MainClass这个属性获得。当我们的jar包里只有一个主类时，MainClass的值就是这个主类的全类名。如果有多个jar包，则不会有MainClass这个属性
>
> 当我们的jar包里有多个主类，我们需要指定主类的全类名才能运行我们想运行的那个主类。这时我们只能用java -classpath test.jar Test
