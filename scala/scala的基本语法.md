### scala的基本语法

***

#### 定义变量和类型推导

> scala中没有基本数据类型，只有值对象和引用对象
>
> java中的基本数据类型在scala中就是值对象，如Int，Float。继承与AnyVal
>
> java中的引用类型在scala中就是引用对象，继承与AnyRef
>
> scala中定义变量必须初始化

``` scala
object TestVariable {
  def main(args: Array[String]): Unit = {
    var name: String = "bighu" //这是最基本的定义形式

    var age = 1 //scala有类型推导的功能，不指定数据类型，它会根据变量的值指定一个类型

    val sex: String = "male" //val为不可变变量，变量定义后就不能改变它的值

    val yellowMan=new Human //yellowMan为val型，yellowMan中的属性可以改变，但不能指向另一个对象
    //在scala中调用方法时，如果方法没有形参，则可以省路括号

    println(100.toString) //100在java中是一个int型的常量，他在scala中是一个Int对象，也能调用方法

    //Null是一种数据类型，是所有AntRef的子类，这个数据类型只有一个取值，那就是null
    //而Nothing是所有类的子类

  }


  class Human {
    var name = "hunam"
    var age = 1
  }

}
```



#### 使用值类型的注意事项

``` scala
/**
 * 测试scala中的值对象的性质
 */
object TestAnyVal {
  def main(args: Array[String]): Unit = {
    var a = 123456789456L //当你输入的值为整数时，系统自动类型推导为Int型，但此时明显超出Int所能表示的最大数。
    //必须在数后面加L,表示这个数为long类型

    //var b: Float=1.2 这是错误的，小数会被自动推导为Double型，高精度不能自动转化为低精度

    var char1: Char = 'a' //和c,java一样，单引号表示子字符
    var char2: Char = 98 //98虽然是Int型，但一个字面量没有经过计算就赋值给char时，编译器只会进行范围的判定。

    //scala值类型的强制转换，由于scala中万物皆对象，所以每个值都自带了强制转换的方法
    var double1 = 1.2
    var int1: Int=1.2.toInt //将double强转为Int
  }
}
```

