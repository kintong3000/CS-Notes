## spring中使用rabbitmq

前面我们已经完成了RabbitMQ的安装和简单使用，并且通过Java连接到服务器。现在我们来尝试在SpringBoot中整合消息队列客户端，首先是依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

接着我们需要配置RabbitMQ的地址等信息：

```yaml
spring:
  rabbitmq:
    addresses: 192.168.0.4
    username: admin
    password: admin
    virtual-host: /test
```

这样我们就完成了最基本信息配置，现在我们来看一下，如何像之前一样去声明一个消息队列，我们只需要一个配置类就行了：

```java
@Configuration
public class RabbitConfiguration {
    @Bean("directExchange")  //定义交换机Bean，可以很多个
    public Exchange exchange(){
        return ExchangeBuilder.directExchange("amq.direct").build();
    }

    @Bean("yydsQueue")     //定义消息队列
    public Queue queue(){
        return QueueBuilder
          				.nonDurable("yyds")   //非持久化类型
          				.build();
    }

    @Bean("binding")
    public Binding binding(@Qualifier("directExchange") Exchange exchange,
                           @Qualifier("yydsQueue") Queue queue){
      	//将我们刚刚定义的交换机和队列进行绑定
        return BindingBuilder
                .bind(queue)   //绑定队列
                .to(exchange)  //到交换机
                .with("my-yyds")   //使用自定义的routingKey
                .noargs();
    }
}
```

接着我们来创建一个生产者，这里我们直接编写在测试用例中：

```java
@SpringBootTest
class SpringCloudMqApplicationTests {

  	//RabbitTemplate为我们封装了大量的RabbitMQ操作，已经由Starter提供，因此直接注入使用即可
    @Resource
    RabbitTemplate template;												//AmqpTemplate template也可以

    @Test
    void publisher() {
      	//使用convertAndSend方法一步到位，参数基本和之前是一样的
      	//最后一个消息本体可以是Object类型，真是大大的方便
        template.convertAndSend("amq.direct", "my-yyds", "Hello World!");//Exchange交换机，routingKey，消息
    }

}
```

现在我们再来看看如何创建一个消费者，因为消费者实际上就是一直等待消息然后进行处理的角色，这里我们只需要创建一个监听器就行了，它会一直等待消息到来然后再进行处理：

```java
@Component  //注册为Bean
@RabbitListener(queues = "yyds")
public class TestListener {

    @RabbitHandler  //用于定义一个方法，该方法会在接收到特定队列的消息时被调用
    public void test(Message message){
        System.out.println(new String(message.getBody()));
    }
}
```

