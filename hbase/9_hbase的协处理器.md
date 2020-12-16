### hbase的协处理器

***

#### 协处理器的作用

> 协处理器允许用户在region服务器上运行自己的代码，允许用户执行region级别的操作，并且可以使用与RDBMS中触发器(trigger)类似的功能。在客户端，用户不用关心操作具体在哪里执行，HBase的分布式框架会帮助用户把这些工作变得透明。
>
> 例如：在你向hbase的表中put一个记录时，Observer协处理器可以在你put操作执行前或执行后做一些操作，这个操作的具体逻辑由我们自己制定。我们需要定义一个类，并实现Observer相关接口。



#### 使用Observer协处理器

> 我使用的是hbase2.1.8版本，在2.1.9的hbase reference guide中得知，要使用协处理器，我需要定义一个类并实现RegionObserver接口。但是我在com.apache.hadoop.hbase.coprocess包中根本找不到RegionObserver接口。
>
> 我猜测是2.1.8和2.1.9发生了一些改变，但是reference guide只有2.1.9，于是我打算去下载2.1.9版本的jar包，看看有没有这个接口。结果是没有
>
> 最后发现在UserAPI文档中没有这个接口，但是DeveloperAPI中有这个接口。也就是说我下载的habse-client这个jar包是“User”版的，我需要搞一个“develop”版的jar包
>
> 结果：我们导入的jar包不分“User“版和"Developer"版。hbase-client-2.1.8.jar中有org.apache.hadoop.hbase.coprocessor这个包，但是没有我们想要的接口。但是hbase-server-2.1.8.jar中也有这个包，并且有我们要的接口。所以说hbase-client和hbase-server是两个独立的maven项目。我们导入了hbase-client，有了org.apache.hadoop.hbase.coprocessor这个包，但是没有对应接口，所以用atl+tab键也找不到RegionObserver这个接口在哪
>
> 我们用alt+tab键想导入这个接口所在的依赖，但是显示没有结果，那是因为idea只会去本地的maven仓库找有没有含RegionObserver的jar包。我们一直都没下载过hbase-server包，所以自然找不到
>
> 其实通过findjar.com是可以通过类名找到jar包的，搜索的结果有几百条，但我以为只有七八条，所以错过了这个提示
>
> 总的来说，要实现hbase的协处理，必须得导入hbase-server这个jar包



Observer协处理器的具体使用：https://cloud.tencent.com/developer/article/1436989
