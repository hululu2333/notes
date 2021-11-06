## maven生命周期解释

### 一、Maven的生命周期

Maven的生命周期就是对所有的构建过程进行抽象和统一。包含了项目的清理、初始化、编译、测试、打包、集成测试、验证、部署和站点生成等几乎所有的构建步骤。

Maven的生命周期是抽象的，即生命周期不做任何实际的工作，实际任务由插件完成，类似于设计模式中的模板方法。

### 二、三套生命周期

Maven有三套相互独立的生命周期，分别是clean、default和site。每个生命周期包含一些阶段（phase），阶段是有顺序的，后面的阶段依赖于前面的阶段。

**1、clean生命周期**：清理项目，包含三个phase。

1）pre-clean：执行清理前需要完成的工作

2）clean：清理上一次构建生成的文件

3）post-clean：执行清理后需要完成的工作

**2、default生命周期**：构建项目，重要的phase如下。

1）validate：验证工程是否正确，所有需要的资源是否可用。
2）compile：编译项目的源代码。  
3）test：使用合适的单元测试框架来测试已编译的源代码。这些测试不需要已打包和布署。
4）Package：把已编译的代码打包成可发布的格式，比如jar。
5）integration-test：如有需要，将包处理和发布到一个能够进行集成测试的环境。
6）verify：运行所有检查，验证包是否有效且达到质量标准。
7）install：把包安装到maven本地仓库，可以被其他工程作为依赖来使用。
8）Deploy：在集成或者发布环境下执行，将最终版本的包拷贝到远程的repository，使得其他的开发者或者工程可以共享。

**3、site生命周期**：建立和发布项目站点，phase如下

1）pre-site：生成项目站点之前需要完成的工作

2）site：生成项目站点文档

3）post-site：生成项目站点之后需要完成的工作

4）site-deploy：将项目站点发布到服务器

### 三、命令行和生命周期

各个生命周期相互独立，一个生命周期的阶段前后依赖。

举例如下：

1、mvn clean

调用clean生命周期的clean阶段，实际执行pre-clean和clean阶段

2、mvn test

调用default生命周期的test阶段，实际执行test以及之前所有阶段

3、mvn clean install

调用clean生命周期的clean阶段和default的install阶段，实际执行pre-clean和clean，install以及之前所有阶段

### 四、m2eclipse和生命周期

1、m2eclipse中预置的mvn命令

右键maven项目或pom.xml文件>Run As 可以看到预置的mvn命令

 2、自定义mvn命令

单击 上图中的maven Build...，自定义命令 mvn clean install：

定义完成后，点击maven Build，可以看到定义好的命令：