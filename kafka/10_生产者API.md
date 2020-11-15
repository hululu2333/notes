### 10_生产者API

***

#### 生产者发送消息的流程

![image-20200703200534507](F:\学习笔记\kafka\imgs\Producer发送消息流程.png)

> 生产者的main线程生产数据，经过拦截器，序列化器，分区器然后进入缓冲区。
>
> 生产者向Kafka中生产数据是异步的，main线程一直生产数据到缓冲区中，sender线程将数据从缓冲区发送到Topic中，然后等待ack。等到了ack就发送下一条，等不到就重新发送上一条



#### 自定义一个生产者

``` java
import org.apache.kafka.clients.producer.*;

import java.util.Properties;
import java.util.concurrent.ExecutionException;

/**
 * 自定义一个带回调函数的生产者
 */
public class CustomProducter {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        Properties props = new Properties();

        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "hadoop01:9092");//指定kafka集群，这里用于测试，只指定一个
        //指定key和value所用的序列化类
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");

        //这五个配置会有默认值，不配置也可以
        props.put("acks", "all");//ack类型为all，即-1
        props.put("retries", 30);//重试次数
        props.put("batch.size", 16384);//批次大小
        props.put("linger.ms", 50);//等待时间
        props.put("buffer.memory", 33554432);//RecordAccumulator 缓 冲区大小


        //producter对象中包含了生产者的一些关键信息，我们可以把他抽象的看成生产者
        Producer<String, String> producer = new KafkaProducer<>(props);

        for (int i = 0; i < 100; i++) {
            System.out.println("执行第" + i + "次循环");
            //Callback是一个接口，但我们需要一个Callback对象，所以要用匿名类的方法实现它
            //因为这个匿名类中只需要实现一个方法，我们可以用lambda表达式去代替匿名类
            //生产者分区可以在send方法中指定。或者是给value指定key值，按key的哈希值来分区
            producer.send(new ProducerRecord<String, String>("first",  Integer.toString(i), "hu" + Integer.toString(i)), new Callback() {

                //回调函数，该方法会在 Producer 收到 ack 时调用，为异步调用
                @Override
                public void onCompletion(RecordMetadata metadata, Exception exception) {
                    if (exception == null) {
                        System.out.println("success->" + metadata.offset() + "--" + metadata.toString());
                    } else {
                        exception.printStackTrace();
                    }
                }
            });
        }
        //回收资源，并将缓冲区中没发送的数据发送出去
        producer.close();
    }
}
```



#### 自定义分区器

``` java 
import org.apache.kafka.clients.producer.Partitioner;
import org.apache.kafka.common.Cluster;

import java.util.Map;

/**
*调用自定义分区器：props.put(ProducerConfig.PARTITIONER_CLASS_CONFIG,"com.hu.kafka.MyPartitioner")
**/
public class MyPartitioner implements Partitioner {
    @Override
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        //分区的逻辑代码写在这个方法里就可以
        //获得当前集群还活着的分区个数
		Integer i = cluster.partitionCountForTopic(topic);
        return 0;
    }

    @Override
    public void close() {

    }

    @Override
    public void configure(Map<String, ?> configs) {

    }
}
```



#### 自定义拦截器

``` java
import org.apache.kafka.clients.producer.ProducerInterceptor;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;

import java.util.Map;

/**
 * 用于在value前加上时间戳的拦截器
 */
public class TimeInterceptor implements ProducerInterceptor<String, String> {
    @Override
    public ProducerRecord<String, String> onSend(ProducerRecord<String, String> record) {
        //因为record中的属性不能修改，所以返回时要新建一个record对象
        String value = record.value();

        return new ProducerRecord<>(record.topic(), record.partition(),
                record.key(), System.currentTimeMillis()+","+record.value());
    }

    @Override
    public void onAcknowledgement(RecordMetadata metadata, Exception exception) {

    }

    @Override
    public void close() {

    }

    @Override
    public void configure(Map<String, ?> configs) {

    }
}
```



#### 带拦截器的生产者

``` java
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;

import java.util.ArrayList;
import java.util.Properties;

/**
 * 使用拦截器的生产者
 */
public class InterceptorProducer {
    public static void main(String[] args) {
        Properties props = new Properties();

        //指定kafka地址
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "hadoop01:9092");

        //指定序列化器
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");

        //这几个配置会有默认值，不配置也可以
        props.put("retries", 30);//重试次数

        //添加拦截器
        ArrayList<String> interceptors = new ArrayList<>();
        interceptors.add("com.hu.kafka.interceptor.TimeInterceptor");
        interceptors.add("com.hu.kafka.interceptor.CountInterceptor");
        props.put(ProducerConfig.INTERCEPTOR_CLASSES_CONFIG, interceptors);

        //创建生产者对象
        Producer<String, String> producer = new KafkaProducer<>(props);

        //开始生产
        for (int i = 0; i < 100; i++) {
            producer.send(new ProducerRecord<String, String>("first",String.valueOf(i),"bighu_"+i));
        }

        //关闭资源，除了将缓存区的数据发送出去外，还会调用拦截器和分区器中的close方法
        producer.close();
    }
}
```

