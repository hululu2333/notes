### Http协议

***

#### http请求

> 请求行：请求方式	请求的资源	Http协议版本
>
> 请求头：若干个key:value
>
> \r\n
>
> 请求体：（get请求的请求体是空的，post将请求的参数放在请求体中）

> http请求就是这么个结构，可以封装成HtttpRequest



#### http响应

> 响应行：协议版本	状态码	码值
>
> 响应头：若干key:value
>
> \r\n
>
> 响应体：目标资源的字节

> 可以封装为HttpResponse



#### 编写类Tomcat服务器的大概流程

> 服务器接到http请求，将请求封装成HttpRequest。封装过程由建造者HttpCreate完成，建造者根据Http请求生成对应的HttpRequest对象
>

