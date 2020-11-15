### XML

***

> xml的全称是可扩展标记语言。xml是纯文本形式，在任何平台都能快速的被利用，所以扩展性非常好



#### XML的一些性质

> xml一般被用于信息的记录和传递，因为是纯文本形式，在任何平台用起来都很简单，所以xml经常充当与配置文件。而html是用来显示数据的，它q注重于数据的外观。
>
> xml十分灵活，没有固定的标签，所有的标签都可以自定义。而html中的标签是预定义的，不能自创



#### 编写xml文档

##### 格式良好的xml

> 必须有xml声明语句
>
> 有且只有一个根元素。元素指由开始标签，元素内容和结束标签组成的内容。也就是说一个xml只能有一对根标签
>
> 标签大小写敏感
>
> 属性值用双引号包括
>
> 标签要成对
>
> 元素要正确嵌套

``` xml
<!-- 声明信息，用于描述xml的版本和编码 -->
<?xml version="1.0" encoding="UTF-8"?> 

<books>
    <book id="b01">
        <name>java高级编程</name>
        <author>张三</author>
        <price>50.5</price>
    </book>
    <book id="b02">
        <name>java中级编程</name>
        <author>李四</author>
        <price>30.5</price>
    </book>
</books>
```

***

##### 有效的xml文档

> 有效的xml文档首先必须是格式良好的，然后还要用DTD或XSD进行约束



#### DTD

> DTD的全称为Document Type Definition，也就是文档类型定义
>
> DTD用来约束xml文档的格式，确保xml是个有效的xml

##### 内部DTD 

> 内部DTD指将DTD代码定义在xml文档内部，
>
> 格式为<!DOCTYPE 根元素 [元素声明]>

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- 写一个内部DTD的案例 -->

<!-- 定义内部DTD -->
<!DOCTYPE books[
	<!-- 表示books标签下可以有任意个book标签，*表示任意个，?表示0或1个，+表示一个或多个 -->
	<!ELEMENT books (book*)> 

	<!-- 表示book标签下必须分别有一个name，author和price -->
	<!ELEMENT book (name,author,price)

	<!-- 给book设置一个名为id的属性，属性值为字符数据，#REQUIRED表示这个值是必须的 -->
	<!ATTLIST book id CDATA #REQUIRED>
]>

<!-- 正式开始写xml -->
<books>
    <book id="1">
        <name></name>
        <author></author>
        <price></price>
    </book>
</books>
```

![image-20200920152143298](F:\学习笔记\java\images\xml元素.png)



#### 解析XML的几种方式

##### Dom4j

>要使用这种解析方式，首先得将dom4j这个jar包导入项目

``` xml
<dependency>
	<groupId>org.dom4j</groupId>
	<artifactId>dom4j</artifactId>
	<version>2.1.1</version>
</dependency>
```



> dom4j的使用方法如下

``` java
//获取解析器
SAXReader r=new SAXReader();

//使用解析器  开始解析xml文件
Document doc=r.read(new File("F:\\IDEAProject\\homework\\src\\main\\java\\com\\hu\\xml\\user.xml"));
//这个代码执行完毕，表示硬盘的xml加载到内存中

//3.获取doc的根节点
Element root=doc.getRootElement();
System.out.println(root.getName());//打印根节点的名称

//4.获取root下一级的标签
List<Element> elements = root.elements();
//遍历
for (Element e:elements){
	String name = e.attributeValue("name");//获得name属性的值
	String ele = e.getName();//获得标签的名字
    String text = e.getText();//获得文本域值
}
```


