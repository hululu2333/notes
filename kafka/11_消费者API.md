### 消费者API

***

#### 自定义一个同步提交offset的消费者

``` java
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;

import java.util.Arrays;
import java.util.Properties;

/**
 * 自定义一个消费者
 */
public class MyConsumer {
    public static void main(String[] args) {
        Properties props = new Properties();

        //指定Kafka集群
        props.put("bootstrap.servers", "hadoop102:9092");

        //消费者组，只要 group.id 相同，就属于同一个消费者组
        props.put("group.id", "test");
        
        //关闭自动提交 offset
        props.put("enable.auto.commit", "false");

        //指定key和value的反序列化器
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

        //生成consumer对象
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);

        //指定消费者订阅的主题的集合
        consumer.subscribe(Arrays.asList("first"));

        while (true) {

            //消费者拉取数据             
            ConsumerRecords<String, String> records = consumer.poll(100);

            for (ConsumerRecord<String, String> record : records) {

                System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value());

            }

            //提交offset，当前线程会阻塞直到 offset 提交成功             
            consumer.commitSync();
        }
    }
}
```



#### 定义一个消费者，offset的储存位置由自己确定

``` java
import org.apache.kafka.clients.consumer.*;
import org.apache.kafka.common.TopicPartition;

import java.util.*;

/**
 * 定义一个消费者，offset存在自定义的地方
 * 重点在于消费者订阅主题的时候传入一个ConsumerRebalanceListener对象
 */
public class MyConsumer1 {

    private static Map<TopicPartition, Long> currentOffset = new HashMap<>();

    public static void main(String[] args) {
        //创建配置信息
        Properties props = new Properties();

        //Kafka 集群
        props.put("bootstrap.servers", "hadoop01:9092");

        //消费者组，只要 group.id 相同，就属于同一个消费者组
        props.put("group.id", "test");

        //关闭自动提交 offset
        props.put("enable.auto.commit", "false");

        //Key 和 Value 的反序列化类
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

        //创建一个消费者
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);

        //消费者订阅主题
        consumer.subscribe(Arrays.asList("first"), new ConsumerRebalanceListener() {
            //该方法会在 Rebalance 之前调用
            @Override
            public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
                commitOffset(currentOffset);
            }

            //该方法会在 Rebalance 之后调用
            @Override
            public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
                currentOffset.clear();
                for (TopicPartition partition : partitions) {
                    consumer.seek(partition, getOffset(partition));// 定位到最近提交的 offset 位置继续消费
                }
            }
        });

        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(100);//消费者拉取数据             
            for (ConsumerRecord<String, String> record : records) {
                System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value());
                currentOffset.put(new TopicPartition(record.topic(), record.partition()), record.offset());
            }
            commitOffset(currentOffset);//异步提交         
        }
    }

    //获取某分区的最新 offset
    private static long getOffset(TopicPartition partition) {
        return 0;
    }

    //提交该消费者所有分区的 offset
    private static void commitOffset(Map<TopicPartition, Long> currentOffset) {

    }
}
```

