原文链接：https://blog.csdn.net/magic_kid_2010/article/details/97135426



一、提交flink任务到yarn。

./flink run -m yarn-cluster -yn 1 -p 2 -yjm 1024 -ytm 1024 -ynm FlinkOnYarnSession-MemberLogInfoProducer -d -c com.igg.flink.tool.member.rabbitmq.producer.MqMemberProducer /home/test_gjm/igg-flink-tool/igg-flink-tool-1.0.0-SNAPSHOT.jar
说明：

-m yarn-cluster 在yarn上运行独立的flink job
-yn 申请的TaskManager数量，目前已废弃，参数不生效
-p 并发数，即槽数
-yjm 申请的JobManager的内存大小
-ytm 申请的每个TaskManager的内存大小
-ynm yarn application 显示的名称
-d 使用分离模式，后续取消任务，需要使用yarn application
-c 类名
二、通过Yarn UI 进入 Flink UI



说明：

TaskManager=1，Task Slots=2，分别与任务参数一一对应。Available Task Slots=0，因在flink-conf.yaml中，设置了

taskmanager.numberOfTaskSlots: 2
所以，可用槽数为0。

三、Task Manager



可以看到，TM运行在230机器以及机器的一些配置信息。登陆230机器，查看TM进程。



四、Job Manager。



可以看到，TM运行在229机器以及flink一些配置信息。登陆230机器，查看JM进程。





其次，还可以切换到logs和stdout，分别查看日志文件数据 和 控制台输出。

说明：

当日志输出很多的时候，打开这里的页面会很慢，甚至卡死。所以线上的时候，一定不要输出一些无关紧要的日志。
如果非要看卡死的日志，可以到yarn配置的临时日志目录去查看。

<property>
    <name>yarn.nodemanager.log-dirs</name>
    <value>/home/hadoop/yarn_dir/log</value>
    <description>
    	Comma-separated list of paths on the local filesystem where logs are written. Multiple paths help spread disk i/o.
    </description>
</property>


tail -f taskmanager.log

五、Submit new Job。



对于独立模式和yarn-session模式，可以通过此ui操作来提交任务。对于独立job的flink只能通过命令。

六、checkpoint 的查看。



通过观察 checkpoint 记录，我们可以观察任务的运行情况。如果 checkpoint status 失败、或者时间较长，等等，说明程序存在

调整的地方，如 checkpoint 的数据过大，则可使用增量的方式，或者代码需要进行调优处理。如 checkpoint 时间较长，可改用

memory/rockdb 等存储。

七、背压情况。



当DAG的某个过程的背压状态为 low 或者 high 时，则说明下游的处理速度不及上游的输出速度。也就是说 下游的处理是整个任务的瓶颈所在，需要进行优化处理。

八、Task Metrics 的使用



可以根据需要，对DAG的某个过程的输入和输出情况进行观察，便于发现和排查问题。

