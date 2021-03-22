# kafka


## 编写代码

### maven 依赖

``` xml
<!-- https://mvnrepository.com/artifact/org.springframework.kafka/spring-kafka -->
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <version>2.3.5.RELEASE</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients -->
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>2.3.1</version>
</dependency>
```

### 配置

``` yaml
server:
  port: 9090
spring:
  kafka:
    consumer:
      group-id: testGroup01
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      bootstrap-servers: 192.168.200.17:9092
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
      bootstrap-servers: 192.168.200.17:9092
```

### 生产者

``` java
@Service
@Slf4j
public class Producer {
    private static final String TOPIC = "hikafka";
    private final KafkaTemplate<String, String> kafkaTemplate;

    @Autowired
    public Producer(KafkaTemplate<String, String> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    public void sendMessage(String message) {
        log.info(String.format("#### -> Producing message -> %s", message));
        this.kafkaTemplate.send(TOPIC, message);
    }
}
```

### 消费者

``` java
@Slf4j
@Service
public class Consumer {
    @KafkaListener(topics = "hikafka", groupId = "testGroup01")
    public void consume(String message) {
        log.info(String.format("#### -> Consumed message -> %s", message));
    }
}
```

### controller

``` java
@RestController
@RequestMapping(value = "kafka")
public class KafkaTestController {
    private final Producer producer;

    @Autowired
    public KafkaTestController(Producer producer) {
        this.producer = producer;
    }

    @GetMapping(value = "publish")
    public void publish(@RequestParam(value = "message") String message) {
        for (int i = 0; i < 1000; i++) {
            producer.sendMessage(message);
        }
    }
}
```

### 运行效果

运行应用后，访问 `http://localhost:9090/kafka/publish?message=hiKafka` 在 topic 发布消息，运行效果如下：

![](https://cdn.jsdelivr.net/gh/bolitao/PicRepository/img/20210322161044.png)
