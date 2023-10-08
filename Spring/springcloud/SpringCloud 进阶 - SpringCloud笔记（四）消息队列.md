![image-20220415163559986](https://s2.loli.net/2023/03/08/9a2q4ZBuWxJs861.jpg)

# 消息队列

经过前面的学习，我们已经了解了我们之前的技术在分布式环境下的应用，接着我们来看最后一章的内容。

那么，什么是消息队列呢？

我们之前如果需要进行远程调用，那么一般可以通过发送HTTP请求来完成，而现在，我们可以使用第二种方式，就是消息队列，它能够将发送方发送的信息放入队列中，当新的消息入队时，会通知接收方进行处理，一般消息发送方称为生产者，接收方称为消费者。

![image-20220415165805716](https://s2.loli.net/2023/03/08/yknBVt2jGgFSTO8.jpg)

这样我们所有的请求，都可以直接丢到消息队列中，再由消费者取出，不再是直接连接消费者的形式了，而是加了一个中间商，这也是一种很好的解耦方案，并且在高并发的情况下，由于消费者能力有限，消息队列也能起到一个削峰填谷的作用，堆积一部分的请求，再由消费者来慢慢处理，而不会像直接调用那样请求蜂拥而至。

那么，消息队列具体实现有哪些呢：

* RabbitMQ  -  性能很强，吞吐量很高，支持多种协议，集群化，消息的可靠执行特性等优势，很适合企业的开发。
* Kafka - 提供了超高的吞吐量，ms级别的延迟，极高的可用性以及可靠性，而且分布式可以任意扩展。
* RocketMQ  -  阿里巴巴推出的消息队列，经历过双十一的考验，单机吞吐量高，消息的高可靠性，扩展性强，支持事务等，但是功能不够完整，语言支持性较差。

我们这里，主要讲解的是RabbitMQ消息队列。

## RabbitMQ 消息队列

**官方网站：**https://www.rabbitmq.com

> RabbitMQ拥有数万计的用户，是最受欢迎的开源消息队列之一，从[T-Mobile](https://www.youtube.com/watch?v=1qcTu2QUtrU)到[Runtastic](https://medium.com/@runtastic/messagebus-handling-dead-letters-in-rabbitmq-using-a-dead-letter-exchange-f070699b952b)，RabbitMQ在全球范围内用于小型初创企业和大型企业。
>
> RabbitMQ轻量级，易于在本地和云端部署，它支持多种消息协议。RabbitMQ可以部署在分布式和联合配置中，以满足大规模、高可用性要求。
>
> RabbitMQ在许多操作系统和云环境中运行，并为[大多数流行语言](https://www.rabbitmq.com/devtools.html)提供了[广泛的开发者工具](https://www.rabbitmq.com/devtools.html)。

我们首先还是来看看如何进行安装。

### 安装消息队列

**下载地址：**https://www.rabbitmq.com/download.html

由于除了消息队列本身之外还需要Erlang环境（RabbitMQ就是这个语言开发的）所以我们就在我们的Ubuntu服务器上进行安装。

首先是Erlang，比较大，1GB左右：

```sh
sudo apt install erlang
```

接着安装RabbitMQ：

```sh
sudo apt install rabbitmq-server
```

安装完成后，可以输入：

```sh
sudo rabbitmqctl status
```

来查看当前的RabbitMQ运行状态，包括运行环境、内存占用、日志文件等信息：

```
Runtime

OS PID: 13718
OS: Linux
Uptime (seconds): 65
Is under maintenance?: false
RabbitMQ version: 3.8.9
Node name: rabbit@ubuntu-server-2
Erlang configuration: Erlang/OTP 23 [erts-11.1.8] [source] [64-bit] [smp:2:2] [ds:2:2:10] [async-threads:64]
Erlang processes: 280 used, 1048576 limit
Scheduler run queue: 1
Cluster heartbeat timeout (net_ticktime): 60
```

这样我们的RabbitMQ服务器就安装完成了，要省事还得是Ubuntu啊。

可以看到默认有两个端口名被使用：

```
Listeners

Interface: [::], port: 25672, protocol: clustering, purpose: inter-node and CLI tool communication
Interface: [::], port: 5672, protocol: amqp, purpose: AMQP 0-9-1 and AMQP 1.0
```

我们一会主要使用的就是amqp协议的那个端口`5672`来进行连接，25672是集群化端口，之后我们也会用到。

接着我们还可以将RabbitMQ的管理面板开启，这样话就可以在浏览器上进行实时访问和监控了：

```sh
sudo rabbitmq-plugins enable rabbitmq_management
```

再次查看状态，可以看到多了一个管理面板，使用的是HTTP协议：

```
Listeners

Interface: [::], port: 25672, protocol: clustering, purpose: inter-node and CLI tool communication
Interface: [::], port: 5672, protocol: amqp, purpose: AMQP 0-9-1 and AMQP 1.0
Interface: [::], port: 15672, protocol: http, purpose: HTTP API
```

我们打开浏览器直接访问一下：

![image-20220415203431587](https://s2.loli.net/2023/03/08/HxtXlqi7BUYWdC2.jpg)

可以看到需要我们进行登录才可以进入，我们这里还需要创建一个用户才可以，这里就都用admin：

```sh
sudo rabbitmqctl add_user 用户名 密码
```

将管理员权限给予我们刚刚创建好的用户：

```sh
sudo rabbitmqctl set_user_tags admin administrator
```

创建完成之后，我们登录一下页面：

![image-20220415203728664](https://s2.loli.net/2023/03/08/eEJMsxhc5Onpld8.jpg)

进入了之后会显示当前的消息队列情况，包括版本号、Erlang版本等，这里需要介绍一下RabbitMQ的设计架构，这样我们就知道各个模块管理的是什么内容了：

![image-20220416103043845](https://s2.loli.net/2023/03/08/j5kIgD9ZRQiGtd6.jpg)

* **生产者（Publisher）和消费者（Consumer）：**不用多说了吧。
* **Channel：**我们的客户端连接都会使用一个Channel，再通过Channel去访问到RabbitMQ服务器，注意通信协议不是http，而是amqp协议。
* **Exchange：**类似于交换机一样的存在，会根据我们的请求，转发给相应的消息队列，每个队列都可以绑定到Exchange上，这样Exchange就可以将数据转发给队列了，可以存在很多个，不同的Exchange类型可以用于实现不同消息的模式。
* **Queue：**消息队列本体，生产者所有的消息都存放在消息队列中，等待消费者取出。
* **Virtual Host：**有点类似于环境隔离，不同环境都可以单独配置一个Virtual Host，每个Virtual Host可以包含很多个Exchange和Queue，每个Virtual Host相互之间不影响。

### 使用消息队列

我们就从最简的的模型开始讲起：

![image-20220417103647609](https://s2.loli.net/2023/03/08/GWkUJx1g8ZnTV57.jpg)

（一个生产者 -> 消息队列 -> 一个消费者）

生产者只需要将数据丢进消息队列，而消费者只需要将数据从消息队列中取出，这样就实现了生产者和消费者的消息交互。我们现在来演示一下，首先进入到我们的管理页面，这里我们创建一个新的实验环境，只需要新建一个Virtual Host即可：

![image-20220419143014974](https://s2.loli.net/2023/03/08/PzehXHuDyFANIKV.jpg)

添加新的虚拟主机之后，我们可以看到，当前admin用户的主机访问权限中新增了我们刚刚添加的环境：

![image-20220419143115507](https://s2.loli.net/2023/03/08/9cGyunKrTvjfDRp.jpg)

现在我们来看看交换机：

![image-20220419143338487](https://s2.loli.net/2023/03/08/GDnFoizW86pC5l9.jpg)

交换机列表中自动为我们新增了刚刚创建好的虚拟主机相关的预设交换机，一共7个，这里我们首先介绍一下前面两个`direct`类型的交换机，一个是`（AMQP default）`还有一个是`amq.direct`，它们都是直连模式的交换机，我们来看看第一个：

![image-20220419143612318](https://s2.loli.net/2023/03/08/lIpfxGjLPrOatE5.jpg)

第一个交换机是所有虚拟主机都会自带的一个默认交换机，并且此交换机不可删除，此交换机默认绑定到所有的消息队列，如果是通过默认交换机发送消息，那么会根据消息的`routingKey`（之后我们发消息都会指定）决定发送给哪个同名的消息队列，同时也不能显示地将消息队列绑定或解绑到此交换机。

我们可以看到，详细信息中，当前交换机特性是持久化的，也就是说就算机器重启，那么此交换机也会保留，如果不是持久化，那么一旦重启就会消失。实际上我们在列表中看到`D`的字样，就表示此交换机是持久化的，包含一会我们要讲解的消息队列列表也是这样，所有自动生成的交换机都是持久化的。

我们接着来看第二个交换机，这个交换机是一个普通的直连交换机：

![image-20220419144200533](https://s2.loli.net/2023/03/08/DnpENxIPgOUTSbM.jpg)

这个交换机和我们刚刚介绍的默认交换机类型一致，并且也是持久化的，但是我们可以看到它是具有绑定关系的，如果没有指定的消息队列绑定到此交换机上，那么这个交换机无法正常将信息存放到指定的消息队列中，也是根据`routingKey`寻找消息队列（但是可以自定义）

我们可以在下面直接操作，让某个队列绑定，这里我们先不进行操作。

介绍完了两个最基本的交换机之后（其他类型的交换机我们会在后面进行介绍），我们接着来看消息队列：

![image-20220419144508881](https://s2.loli.net/2023/03/08/q7WcUvZp8NhMb9f.jpg)

可以看到消息队列列表中没有任何的消息队列，我们可以来尝试添加一个新的消息队列：

![image-20220419144553817](https://s2.loli.net/2023/03/08/D8hv6Kbo3eSNzVp.jpg)

第一行，我们选择我们刚刚创建好的虚拟主机，在这个虚拟主机下创建此消息队列，接着我们将其类型定义为`Classic`类型，也就是经典类型（其他类型我们会在后面逐步介绍）名称随便起一个，然后持久化我们选择`Transient`暂时的（当然也可以持久化，看你自己）自动删除我们选择`No`（需要至少有一个消费者连接到这个队列，之后，一旦所有与这个队列连接的消费者都断开时，就会自动删除此队列）最下面的参数我们暂时不进行任何设置（之后会用到）

现在，我们就创建好了一个经典的消息队列：

![image-20220419145109450](https://s2.loli.net/2023/03/08/yGSt4HbT7iX3Nze.jpg)

点击此队列的名称，我们可以查看详细信息：

![image-20220419145238458](https://s2.loli.net/2023/03/08/NGCFKhcUf9lOADX.jpg)

详细相信中包括队列的当前负载状态、属性、消息队列占用的内存，消息数量等，一会我们发送消息时可以进一步进行观察。

现在我们需要将此消息队列绑定到上面的第二个直连交换机，这样我们就可以通过此交换机向此消息队列发送消息了：

![image-20220419145520844](https://s2.loli.net/2023/03/08/NGCFKhcUf9lOADX.jpg)

这里填写之前第二个交换机的名称还有我们自定义的`routingKey`（最好还是和消息队列名称一致，这里是为了一会演示两个交换机区别用）我们直接点击绑定即可：

![image-20220419145635179](https://s2.loli.net/2023/03/08/u95NJG2YskOBpXl.jpg)

绑定之后我们可以看到当前队列已经绑定对应的交换机了，现在我们可以前往交换机对此消息队列发送一个消息：

![image-20220419145725499](https://s2.loli.net/2023/03/08/MBIDVqzf8oce2L4.jpg)

回到交换机之后，可以卡到这边也是同步了当前的绑定信息，在下方，我们直接向此消息队列发送信息：

![image-20220419145808450](https://s2.loli.net/2023/03/08/htEoZ49zu6mipCM.jpg)

点击发送之后，我们回到刚刚的交换机详细页面，可以看到已经有一条新的消息在队列中了：

![image-20220419145903723](https://s2.loli.net/2023/03/08/nO9eUjMRbCmBqPT.jpg)

我们可以直接在消息队列这边获取消息队列中的消息，找到下方的Get message选项：

![image-20220419145936160](https://s2.loli.net/2023/03/08/emrY3SZ98CJRAOh.jpg)

可以看到有三个选择，首先第一个Ack Mode，这个是应答模式选择，一共有4个选项：

![image-20220419150053926](https://s2.loli.net/2023/03/08/nrWPuoGRTp7F36e.jpg)

* Nack message requeue true：拒绝消息，也就是说不会将消息从消息队列取出，并且重新排队，一次可以拒绝多个消息。
* Ack message requeue false：确认应答，确认后消息会从消息队列中移除，一次可以确认多个消息。
* Reject message requeue true/false：也是拒绝此消息，但是可以指定是否重新排队。

这里我们使用默认的就可以了，这样只会查看消息是啥，但是不会取出，消息依然存在于消息队列中，第二个参数是编码格式，使用默认的就可以了，最后就是要生效的操作数量，选择1就行：

![image-20220419150712314](https://s2.loli.net/2023/03/08/c6auDXoHFqZT9M2.jpg)

可以看到我们刚刚的消息已经成功读取到。

现在我们再去第一个默认交换机中尝试发送消息试试看：

![image-20220419150913859](https://s2.loli.net/2023/03/08/t5V3yQ8kbOKRpxf.jpg)

如果我们使用之前自定义的`routingKey`，会显示没有路由，这是因为默认的交换机只会找对应名称的消息队列，我们现在向`yyds`发送一下试试看：

![image-20220419151016735](https://s2.loli.net/2023/03/08/LCVPvykIjMox9m1.jpg)

可以看到消息成功发布了，我们来接收一下看看：

![image-20220419151058659](https://s2.loli.net/2023/03/08/9jsdfADB5HRC7wP.jpg)

可以看到成功发送到此消息队列中了。

当然除了在交换机发送消息给消息队列之外，我们也可以直接在消息队列这里发：

![image-20220419151155264](https://s2.loli.net/2023/03/08/cYPwJnbiZlmvqD3.jpg)

效果是一样的，注意这里我们可以选择是否将消息持久化，如果是持久化消息，那么就算服务器重启，此消息也会保存在消息队列中。

最后如果我们不需要再使用此消息队列了，我们可以手动对其进行删除或是清空：

![image-20220419151548923](https://s2.loli.net/2023/03/08/kJE5xwgZOTGWjLq.jpg)

点击Delete Queue删除我们刚刚创建好的`yyds`队列，到这里，我们对应消息队列的一些简单使用，就讲解完毕了。

### 使用Java操作消息队列

现在我们来看看如何通过Java连接到RabbitMQ服务器并使用消息队列进行消息发送（这里一起讲解，包括Java基础版本和SpringBoot版本），首先我们使用最基本的Java客户端连接方式：

```xml
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>5.14.2</version>
</dependency>
```

依赖导入之后，我们来实现一下生产者和消费者，首先是生产者，生产者负责将信息发送到消息队列：

```java
public static void main(String[] args) {
    //使用ConnectionFactory来创建连接
    ConnectionFactory factory = new ConnectionFactory();

    //设定连接信息，基操
    factory.setHost("192.168.0.12");
    factory.setPort(5672);  //注意这里写5672，是amqp协议端口
    factory.setUsername("admin");
    factory.setPassword("admin");
    factory.setVirtualHost("/test");
  
 		//创建连接
    try(Connection connection = factory.newConnection()){
        
    }catch (Exception e){
        e.printStackTrace();
    }
}
```

这里我们可以直接在程序中定义并创建消息队列（实际上是和我们在管理页面创建一样的效果）客户端需要通过连接创建一个新的通道（Channel），同一个连接下可以有很多个通道，这样就不用创建很多个连接也能支持分开发送了。

```java
try(Connection connection = factory.newConnection();
    Channel channel = connection.createChannel()){   //通过Connection创建新的Channel
  	//声明队列，如果此队列不存在，会自动创建
    channel.queueDeclare("yyds", false, false, false, null);
  	//将队列绑定到交换机
    channel.queueBind("yyds", "amq.direct", "my-yyds");
  	//发布新的消息，注意消息需要转换为byte[]
    channel.basicPublish("amq.direct", "my-yyds", null, "Hello World!".getBytes());
}catch (Exception e){
    e.printStackTrace();
}
```

其中`queueDeclare`方法的参数如下：

* queue：队列的名称（默认创建后routingKey和队列名称一致）
* durable：是否持久化。
* exclusive：是否排他，如果一个队列被声明为排他队列，该队列仅对首次声明它的连接可见，并在连接断开时自动删除。排他队列是基于Connection可见，同一个Connection的不同Channel是可以同时访问同一个连接创建的排他队列，并且，如果一个Connection已经声明了一个排他队列，其他的Connection是不允许建立同名的排他队列的，即使该队列是持久化的，一旦Connection关闭或者客户端退出，该排他队列都会自动被删除。
* autoDelete：是否自动删除。
* arguments：设置队列的其他一些参数，这里我们暂时不需要什么其他参数。

其中`queueBind`方法参数如下：

* queue：需要绑定的队列名称。
* exchange：需要绑定的交换机名称。
* routingKey：不用多说了吧。

其中`basicPublish`方法的参数如下：

* exchange: 对应的Exchange名称，我们这里就使用第二个直连交换机。
* routingKey：这里我们填写绑定时指定的routingKey，其实和之前在管理页面操作一样。
* props：其他的配置。
* body：消息本体。

执行完成后，可以在管理页面中看到我们刚刚创建好的消息队列了：

![image-20220419153630431](https://s2.loli.net/2023/03/08/baiDgVyoPQ65TMX.jpg)

并且此消息队列已经成功与`amq.direct`交换机进行绑定：

![image-20220419154618613](https://s2.loli.net/2023/03/08/5lENjHswniC4Zg8.jpg)

那么现在我们的消息队列中已经存在数据了，怎么将其读取出来呢？我们来看看如何创建一个消费者：

```java
public static void main(String[] args) throws IOException, TimeoutException {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("10.37.129.4");
    factory.setPort(5672);
    factory.setUsername("admin");
    factory.setPassword("admin");
    factory.setVirtualHost("/test");

    //这里不使用try-with-resource，因为消费者是一直等待新的消息到来，然后按照
    //我们设定的逻辑进行处理，所以这里不能在定义完成之后就关闭连接
    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();

    //创建一个基本的消费者
    channel.basicConsume("yyds", false, (s, delivery) -> {
        System.out.println(new String(delivery.getBody()));
        //basicAck是确认应答，第一个参数是当前的消息标签，后面的参数是
        //是否批量处理消息队列中所有的消息，如果为false表示只处理当前消息
        channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        //basicNack是拒绝应答，最后一个参数表示是否将当前消息放回队列，如果
        //为false，那么消息就会被丢弃
        //channel.basicNack(delivery.getEnvelope().getDeliveryTag(), false, false);
        //跟上面一样，最后一个参数为false，只不过这里省了
        //channel.basicReject(delivery.getEnvelope().getDeliveryTag(), false);
    }, s -> {});
}
```

其中`basicConsume`方法参数如下：

* queue  -  消息队列名称，直接指定。
* autoAck - 自动应答，消费者从消息队列取出数据后，需要跟服务器进行确认应答，当服务器收到确认后，会自动将消息删除，如果开启自动应答，那么消息发出后会直接删除。
* deliver  -  消息接收后的函数回调，我们可以在回调中对消息进行处理，处理完成后，需要给服务器确认应答。
* cancel  -  当消费者取消订阅时进行的函数回调，这里暂时用不到。

现在我们启动一下消费者，可以看到立即读取到我们刚刚插入到队列中的数据：

![image-20220419155938158](https://s2.loli.net/2023/03/08/rR7eThxXbufjsEo.jpg)

我们现在继续在消息队列中插入新的数据，这里直接在网页上进行操作就行了，同样的我们也可以在消费者端接受并进行处理。

现在我们把刚刚创建好的消息队列删除。

官方文档：https://docs.spring.io/spring-amqp/docs/current/reference/html/

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
    RabbitTemplate template;

    @Test
    void publisher() {
      	//使用convertAndSend方法一步到位，参数基本和之前是一样的
      	//最后一个消息本体可以是Object类型，真是大大的方便
        template.convertAndSend("amq.direct", "my-yyds", "Hello World!");
    }

}
```

现在我们来运行一下这个测试用例：

![image-20220419221426545](https://s2.loli.net/2023/03/08/UxVemu9B2cGifWv.jpg)

可以看到后台自动声明了我们刚刚定义好的消息队列和交换机以及对应的绑定关系，并且我们的数据也是成功插入到消息队列中：

![image-20220419221532673](https://s2.loli.net/2023/03/08/RjY4JUn7v9pmryx.jpg)

现在我们再来看看如何创建一个消费者，因为消费者实际上就是一直等待消息然后进行处理的角色，这里我们只需要创建一个监听器就行了，它会一直等待消息到来然后再进行处理：

```java
@Component  //注册为Bean
public class TestListener {

    @RabbitListener(queues = "yyds")   //定义此方法为队列yyds的监听器，一旦监听到新的消息，就会接受并处理
    public void test(Message message){
        System.out.println(new String(message.getBody()));
    }
}
```

接着我们启动服务器：

![image-20220419230223151](https://s2.loli.net/2023/03/08/ZjRs8u2cHbBEOaW.jpg)

可以看到控制台成功输出了我们之前放入队列的消息，并且管理页面中也显示此消费者已经连接了：

![image-20220419230315376](https://s2.loli.net/2023/03/08/RwUFdgqXlDKk7AI.jpg)

接着我们再通过管理页面添加新的消息看看，也是可以正常进行接受的。

当然，如果我们需要确保消息能够被消费者接受并处理，然后得到消费者的反馈，也是可以的：

```java
@Test
void publisher() {
  	//会等待消费者消费然后返回响应结果
    Object res = template.convertSendAndReceive("amq.direct", "my-yyds", "Hello World!");
    System.out.println("收到消费者响应："+res);
}
```

消费者这边只需要返回一个对应的结果即可：

```java
@RabbitListener(queues = "yyds")
public String receiver(String data){
    System.out.println("一号消息队列监听器 "+data);
    return "收到!";
}
```

测试没有问题：

![image-20220421142425891](https://s2.loli.net/2023/03/08/OkV6zN9PJRlwnQF.jpg)

那么如果我们需要直接接收一个JSON格式的消息，并且希望直接获取到实体类呢？

```java
@Data
public class User {
    int id;
    String name;
}
```

```java
@Configuration
public class RabbitConfiguration {
  	...

    @Bean("jacksonConverter")   //直接创建一个用于JSON转换的Bean
    public Jackson2JsonMessageConverter converter(){
        return new Jackson2JsonMessageConverter();
    }
}
```

接着我们只需要指定转换器就可以了：

```java
@Component
public class TestListener {

  	//指定messageConverter为我们刚刚创建的Bean名称
    @RabbitListener(queues = "yyds", messageConverter = "jacksonConverter")
    public void receiver(User user){  //直接接收User类型
        System.out.println(user);
    }
}
```

现在我们直接在管理页面发送：

```json
{"id":1,"name":"LB"}
```

!![image-20220416225912100](https://s2.loli.net/2023/03/08/3dXbs5naViUMrDO.jpg)](https://tva1.sinaimg.cn/large/e6c9d24ely1h1byhcakabj221m0lwac0.jpg)

可以看到成功完成了转换，并输出了用户信息：

![image-20220416225829807](https://s2.loli.net/2023/03/08/aM8SCL12hkKynUu.jpg)

同样的，我们也可以直接发送User，因为我们刚刚已经配置了Jackson2JsonMessageConverter为Bean，所以直接使用就可以了：

```java
@Test
void publisher() {
    template.convertAndSend("amq.direct", "yyds", new User());
}
```

可以看到后台的数据类型为：

![image-20220419232715025](https://s2.loli.net/2023/03/08/xVSpC7KHE1RyOk6.jpg)

![image-20220416231709750](https://s2.loli.net/2023/03/08/Q9tBuprGwfleNLZ.jpg)

这样，我们就通过SpringBoot实现了RabbitMQ的简单使用。

### 死信队列

消息队列中的数据，如果迟迟没有消费者来处理，那么就会一直占用消息队列的空间。比如我们模拟一下抢车票的场景，用户下单高铁票之后，会进行抢座，然后再进行付款，但是如果用户下单之后并没有及时的付款，这张票不可能一直让这个用户占用着，因为你不买别人还要买呢，所以会在一段时间后超时，让这张票可以继续被其他人购买。

这时，我们就可以使用死信队列，将那些用户超时未付款的或是用户主动取消的订单，进行进一步的处理，以下类型的消息都会被判定为死信：

- 消息被拒绝(basic.reject / basic.nack)，并且requeue = false
- 消息TTL过期
- 队列达到最大长度

![image-20220419112336088](https://s2.loli.net/2023/03/08/itUWySuA9kvcEgs.jpg)

那么如何构建这样的一种使用模式呢？实际上本质就是一个死信交换机+绑定的死信队列，当正常队列中的消息被判定为死信时，会被发送到对应的死信交换机，然后再通过交换机发送到死信队列中，死信队列也有对应的消费者去处理消息。

这里我们直接在配置类中创建一个新的死信交换机和死信队列，并进行绑定：

```java
@Configuration
public class RabbitConfiguration {

    @Bean("directDlExchange")
    public Exchange dlExchange(){
        //创建一个新的死信交换机
        return ExchangeBuilder.directExchange("dlx.direct").build();
    }

    @Bean("yydsDlQueue")   //创建一个新的死信队列
    public Queue dlQueue(){
        return QueueBuilder
                .nonDurable("dl-yyds")
                .build();
    }

    @Bean("dlBinding")   //死信交换机和死信队列进绑定
    public Binding dlBinding(@Qualifier("directDlExchange") Exchange exchange,
                           @Qualifier("yydsDlQueue") Queue queue){
        return BindingBuilder
                .bind(queue)
                .to(exchange)
                .with("dl-yyds")
                .noargs();
    }

		...

    @Bean("yydsQueue")
    public Queue queue(){
        return QueueBuilder
                .nonDurable("yyds")
                .deadLetterExchange("dlx.direct")   //指定死信交换机
                .deadLetterRoutingKey("dl-yyds")   //指定死信RoutingKey
                .build();
    }
  
  	...
}
```

接着我们将监听器修改为死信队列监听：

```java
@Component
public class TestListener {
    @RabbitListener(queues = "dl-yyds", messageConverter = "jacksonConverter")
    public void receiver(User user){
        System.out.println(user);
    }
}
```

配置完成后，我们来尝试启动一下吧，注意启动之前记得把之前的队列给删了，这里要重新定义。

![image-20220420103846981](https://s2.loli.net/2023/03/08/AdrS9yxnRojfWgL.jpg)

队列列表中已经出现了我们刚刚定义好的死信队列，并且yyds队列也支持死信队列发送功能了，现在我们尝试向此队列发送一个消息，但是我们将其拒绝：

![image-20220420105359931](https://s2.loli.net/2023/03/08/mLokEWYcQ4PXnar.jpg)

可以看到拒绝后，如果不让消息重新排队，那么就会变成死信，直接被丢进死信队列中，可以看到在拒绝后：

![image-20220420105455291](https://s2.loli.net/2023/03/08/rgiWVJMbpKzQX46.jpg)

现在我们来看看第二种情况，RabbitMQ支持将超过一定时间没被消费的消息自动删除，这需要消息队列设定TTL值，如果消息的存活时间超过了Time To Live值，就会被自动删除，自动删除后的消息如果有死信队列，那么就会进入到死信队列中。

现在我们将yyds消息队列设定TTL值（毫秒为单位）：

```java
@Bean("yydsQueue")
public Queue queue(){
    return QueueBuilder
            .nonDurable("yyds")
            .deadLetterExchange("dlx.direct")
            .deadLetterRoutingKey("dl-yyds")
            .ttl(5000)   //如果5秒没处理，就自动删除
            .build();
}
```

现在我们重启测试一下，注意修改了之后记得删除之前的yyds队列：

![image-20220420110317997](https://s2.loli.net/2023/03/08/u8xboyv3aTJ9ZE6.jpg)

可以看到现在yyds队列已经具有TTL特性了，我们现在来插入一个新的消息：

![image-20220420110504022](https://s2.loli.net/2023/03/08/2qensPxuf3zLoQ1.jpg)

可以看到消息5秒钟之后就不见了，而是被丢进了死信队列中。

最后我们来看一下当消息队列长度达到最大的情况，现在我们将消息队列的长度进行限制：

```java
@Bean("yydsQueue")
public Queue queue(){
    return QueueBuilder
            .nonDurable("yyds")
            .deadLetterExchange("dlx.direct")
            .deadLetterRoutingKey("dl-yyds")
            .maxLength(3)   //将最大长度设定为3
            .build();
}
```

现在我们重启一下，然后尝试连续插入4个消息：

![image-20220420135316458](https://s2.loli.net/2023/03/08/56TsMf24QlhZCYL.jpg)

可以看到yyds消息队列新增了Limit特性，也就是限定长度：

```java
@Test
void publisher() {
    for (int i = 0; i < 4; i++) 
        template.convertAndSend("amq.direct", "my-yyds", new User());
}
```

![image-20220420135419673](https://s2.loli.net/2023/03/08/d3lEHLPR4VNF92T.jpg)

可以看到因为长度限制为3，所以有一个消息直接被丢进了死信队列中，为了能够更直观地观察消息队列的机制，我们为User类新增一个时间字段：

```java
@Data
public class User {
    int id;
    String name;
    String date = new Date().toString();
}
```

接着每隔一秒钟插入一个：

```java
@Test
void publisher() throws InterruptedException {
    for (int i = 0; i < 4; i++) {
        Thread.sleep(1000);
        template.convertAndSend("amq.direct", "my-yyds", new User());
    }
}
```

再次进行上述实验，可以发现如果到达队列长度限制，那么每次插入都会把位于队首的消息丢进死信队列，来腾出空间给新来的消息。

### 工作队列模式

**注意：**XX模式只是一种设计思路，并不是指的具体的某种实现，可以理解为实现XX模式需要怎么去写。

前面我们了解了最简的一个消费者一个生产者的模式，接着我们来了解一下一个生产者多个消费者的情况：

![image-20220420151258324](https://s2.loli.net/2023/03/08/8AR4H5LbOCrXZmu.jpg)

实际上这种模式就非常适合多个工人等待新的任务到来的场景，我们的任务有很多个，一个一个丢进消息队列，而此时工人有很多个，那么我们就可以将这些任务分配个各个工人，让他们各自负责一些任务，并且做的快的工人还可以做完成一些（能者多劳）。

非常简单，我们只需要创建两个监听器即可：

```java
@Component
public class TestListener {
    @RabbitListener(queues = "yyds")
    public void receiver(String data){   //这里直接接收String类型的数据
        System.out.println("一号消息队列监听器 "+data);
    }

    @RabbitListener(queues = "yyds")
    public void receiver2(String data){
        System.out.println("二号消息队列监听器 "+data);
    }
}
```

可以看到我们发送消息时，会自动进行轮询分发：

![image-20220420154602883](https://s2.loli.net/2023/03/08/YgibmNxD9qtHajQ.jpg)

那么如果我们一开始就在消息队列中放入一部分消息在开启消费者呢？

![image-20220420154654901](https://s2.loli.net/2023/03/08/Rv6YkDSTPl83Hmo.jpg)

可以看到，如果是一开始就存在消息，会被一个消费者一次性全部消耗，这是因为我们没有对消费者的Prefetch count（预获取数量，一次性获取消息的最大数量）进行限制，也就是说我们现在希望的是消费者一次只能拿一个消息，而不是将所有的消息全部都获取。

![image-20220420160253144](https://s2.loli.net/2023/03/08/UNEniupt5cRHz93.jpg)

因此我们需要对这个数量进行一些配置，这里我们需要在配置类中定义一个自定义的ListenerContainerFactory，可以在这里设定消费者Channel的PrefetchCount的大小：

```java
@Resource
private CachingConnectionFactory connectionFactory;

@Bean(name = "listenerContainer")
public SimpleRabbitListenerContainerFactory listenerContainer(){
    SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
    factory.setConnectionFactory(connectionFactory);
    factory.setPrefetchCount(1);   //将PrefetchCount设定为1表示一次只能取一个
    return factory;
}
```

接着我们在监听器这边指定即可：

```java
@Component
public class TestListener {
    @RabbitListener(queues = "yyds",  containerFactory = "listenerContainer")
    public void receiver(String data){
        System.out.println("一号消息队列监听器 "+data);
    }

    @RabbitListener(queues = "yyds", containerFactory = "listenerContainer")
    public void receiver2(String data){
        System.out.println("二号消息队列监听器 "+data);
    }
}
```

现在我们再次启动服务器，可以看到PrefetchCount被限定为1了：

![image-20220420164702864](https://s2.loli.net/2023/03/08/KgWzOUu8ry2V9Ej.jpg)

再次重复上述的实现，可以看到消息不会被一号消费者给全部抢走了：

![image-20220420164827502](https://s2.loli.net/2023/03/08/vmAfh68GpuQXdUk.jpg)

当然除了去定义两个相同的监听器之外，我们也可以直接在注解中定义，比如我们现在需要10个同样的消费者：

```java
@Component
public class TestListener {
    @RabbitListener(queues = "yyds",  containerFactory = "listenerContainer", concurrency = "10")
    public void receiver(String data){
        System.out.println("一号消息队列监听器 "+data);
    }
}
```

可以看到在管理页面中出现了10个消费者：

![image-20220420170349298](https://s2.loli.net/2023/03/08/REl1qIaMXLTK6js.jpg)

至此，有关工作队列模式就讲到这里。

### 发布订阅模式

前面我们已经了解了RabbitMQ客户端的一些基本操作，包括普通的消息模式，接着我们来了解一下其他的模式，首先是发布订阅模式，它支持多种方式：

![image-20220420172252440](https://s2.loli.net/2023/03/08/fetLjQszH7cTZmO.jpg)

比如我们在阿里云买了云服务器，但是最近快到期了，那么就会给你的手机、邮箱发送消息，告诉你需要去续费了，但是手机短信和邮件发送并不一定是同一个业务提供的，但是现在我们又希望能够都去执行，所以就可以用到发布订阅模式，简而言之就是，发布一次，消费多个。

实现这种模式其实也非常简单，但是如果使用我们之前的直连交换机，肯定是不行的，我们这里需要用到另一种类型的交换机，叫做`fanout`（扇出）类型，这时一种广播类型，消息会被广播到所有与此交换机绑定的消息队列中。

这里我们使用默认的交换机：

![image-20220420225300171](https://s2.loli.net/2023/03/08/Er7RBCjm3nNJZHT.jpg)

这个交换机是一个`fanout`类型的交换机，我们就是要它就行了：

```java
@Configuration
public class RabbitConfiguration {

    @Bean("fanoutExchange")
    public Exchange exchange(){
      	//注意这里是fanoutExchange
        return ExchangeBuilder.fanoutExchange("amq.fanout").build();
    }

    @Bean("yydsQueue1")
    public Queue queue(){
        return QueueBuilder.nonDurable("yyds1").build();
    }

    @Bean("binding")
    public Binding binding(@Qualifier("fanoutExchange") Exchange exchange,
                           @Qualifier("yydsQueue1") Queue queue){
        return BindingBuilder
                .bind(queue)
                .to(exchange)
                .with("yyds1")
                .noargs();
    }

    @Bean("yydsQueue2")
    public Queue queue2(){
        return QueueBuilder.nonDurable("yyds2").build();
    }

    @Bean("binding2")
    public Binding binding2(@Qualifier("fanoutExchange") Exchange exchange,
                           @Qualifier("yydsQueue2") Queue queue){
        return BindingBuilder
                .bind(queue)
                .to(exchange)
                .with("yyds2")
                .noargs();
    }
}
```

这里我们将两个队列都绑定到此交换机上，我们先启动看看效果：

![image-20220420230954785](https://s2.loli.net/2023/03/08/pFXEmbv7LCMKxwq.jpg)

绑定没有什么问题，接着我们搞两个监听器，监听一下这两个队列：

```java
@Component
public class TestListener {
    @RabbitListener(queues = "yyds1")
    public void receiver(String data){
        System.out.println("一号消息队列监听器 "+data);
    }

    @RabbitListener(queues = "yyds2")
    public void receiver2(String data){
        System.out.println("二号消息队列监听器 "+data);
    }
}
```

现在我们通过交换机发送消息，看看是不是两个监听器都会接收到消息：

![image-20220420231113658](https://s2.loli.net/2023/03/08/k7V1xXyGTPKO6eb.jpg)

可以看到确实是两个消息队列都能够接受到此消息：

![image-20220420231145578](https://s2.loli.net/2023/03/08/vhwydqXr9Ue61t4.jpg)

这样我们就实现了发布订阅模式。

### 路由模式

路由模式实际上我们一开始就已经实现了，我们可以在绑定时指定想要的`routingKey`只有生产者发送时指定了对应的`routingKey`才能到达对应的队列。

![image-20220420232826848](https://s2.loli.net/2023/03/08/52vs9bualApXGMR.jpg)

当然除了我们之前的一次绑定之外，同一个消息队列可以多次绑定到交换机，并且使用不同的`routingKey`，这样只要满足其中一个都可以被发送到此消息队列中：

```java
@Configuration
public class RabbitConfiguration {

    @Bean("directExchange")
    public Exchange exchange(){
        return ExchangeBuilder.directExchange("amq.direct").build();
    }

    @Bean("yydsQueue")
    public Queue queue(){
        return QueueBuilder.nonDurable("yyds").build();
    }

    @Bean("binding")   //使用yyds1绑定
    public Binding binding(@Qualifier("directExchange") Exchange exchange,
                           @Qualifier("yydsQueue") Queue queue){
        return BindingBuilder
                .bind(queue)
                .to(exchange)
                .with("yyds1")
                .noargs();
    }

    @Bean("binding2")   //使用yyds2绑定
    public Binding binding2(@Qualifier("directExchange") Exchange exchange,
                           @Qualifier("yydsQueue") Queue queue){
        return BindingBuilder
                .bind(queue)
                .to(exchange)
                .with("yyds2")
                .noargs();
    }
}
```

启动后我们可以看到管理面板中出现了两个绑定关系：

![image-20220420233606749](https://s2.loli.net/2023/03/08/n9NxMEsoCeWSaVL.jpg)

这里可以测试一下，随便使用哪个`routingKey`都可以。

### 主题模式

实际上这种模式就是一种模糊匹配的模式，我们可以将`routingKey`以模糊匹配的方式去进行转发。

![image-20220420233721239](https://s2.loli.net/2023/03/08/z45gI7UaKmCipEL.jpg)

我们可以使用`*`或`#`来表示：

- \* - 表示任意的一个单词
- \# - 表示0个或多个单词

这里我们来测试一下：

```java
@Configuration
public class RabbitConfiguration {

    @Bean("topicExchange")  //这里使用预置的Topic类型交换机
    public Exchange exchange(){
        return ExchangeBuilder.topicExchange("amq.topic").build();
    }

    @Bean("yydsQueue")
    public Queue queue(){
        return QueueBuilder.nonDurable("yyds").build();
    }

    @Bean("binding")
    public Binding binding2(@Qualifier("topicExchange") Exchange exchange,
                           @Qualifier("yydsQueue") Queue queue){
        return BindingBuilder
                .bind(queue)
                .to(exchange)
                .with("*.test.*")
                .noargs();
    }
}
```

启动项目，可以看到只要是满足通配符条件的都可以成功转发到对应的消息队列：

![image-20220421103753962](https://s2.loli.net/2023/03/08/aS37QitoUdf4FZ9.jpg)

接着我们可以再试试看`#`通配符。

除了我们这里使用的默认主题交换机之外，还有一个叫做`amq.rabbitmq.trace`的交换机：

![image-20220421104035463](https://s2.loli.net/2023/03/08/CWfRIwoYLjQrbpH.jpg)

可以看到它也是`topic`类型的，那么这个交换机是做什么的呢？实际上这是用于帮助我们记录和追踪生产者和消费者使用消息队列的交换机，它是一个内部的交换机，那么如果使用呢？首先创建一个消息队列用于接收记录：

![image-20220421104619325](https://s2.loli.net/2023/03/08/s7B38pjkd4EGFLI.jpg)

接着我们需要在控制台将虚拟主机`/test`的追踪功能开启：

```sh
sudo rabbitmqctl trace_on -p /test
```

开启后，我们将此队列绑定到上面的交换机上：

![image-20220421104843224](https://s2.loli.net/2023/03/08/VsD2dYIpHhbt6R9.jpg)

![image-20220421105141144](https://s2.loli.net/2023/03/08/EM4WKHqtyz3vLSk.jpg)

由于发送到此交换机上的`routingKey`为routing key为 publish.交换机名称 和 deliver.队列名称，分别对应生产者投递到交换机的消息，和消费者从队列上获取的消息，因此这里使用`#`通配符进行绑定。

现在我们来测试一下，比如还是往yyds队列发送消息：

![image-20220421105242770](https://s2.loli.net/2023/03/08/vHKPqJFahV8y7l3.jpg)

可以看到在发送消息，并且消费者已经处理之后，`trace`队列中新增了两条消息，那么我们来看看都是些什么消息：

![image-20220421105528532](https://s2.loli.net/2023/03/08/vHKPqJFahV8y7l3.jpg)

通过追踪，我们可以很明确地得知消息发送的交换机、routingKey、用户等信息，包括信息本身，同样的，消费者在取出数据时也有记录：

![image-20220421105638715](https://s2.loli.net/2023/03/08/NApBodythmfjzMV.jpg)

我们可以明确消费者的地址、端口、具体操作的队列以及取出的消息信息等。

到这里，我们就已经了解了3种类型的交换机。

### 第四种交换机类型

通过前面的学习，我们已经介绍了三种交换机类型，现在我们来介绍一下第四种交换机类型`header`，它是根据头部信息来决定的，在我们发送的消息中是可以携带一些头部信息的（类似于HTTP），我们可以根据这些头部信息来决定路由到哪一个消息队列中。

```java
@Configuration
public class RabbitConfiguration {

    @Bean("headerExchange")  //注意这里返回的是HeadersExchange
    public HeadersExchange exchange(){
        return ExchangeBuilder
                .headersExchange("amq.headers")  //RabbitMQ为我们预置了两个，这里用第一个就行
                .build();
    }

    @Bean("yydsQueue")
    public Queue queue(){
        return QueueBuilder.nonDurable("yyds").build();
    }

    @Bean("binding")
    public Binding binding2(@Qualifier("headerExchange") HeadersExchange exchange,  //这里和上面一样的类型
                           @Qualifier("yydsQueue") Queue queue){
        return BindingBuilder
                .bind(queue)
                .to(exchange)   //使用HeadersExchange的to方法，可以进行进一步配置
          			//.whereAny("a", "b").exist();   这个是只要存在任意一个指定的头部Key就行
                //.whereAll("a", "b").exist();   这个是必须存在所有指定的的头部Key
                .where("test").matches("hello");   //比如我们现在需要消息的头部信息中包含test，并且值为hello才能转发给我们的消息队列
      					//.whereAny(Collections.singletonMap("test", "hello")).match();  传入Map也行，批量指定键值对
    }
}
```

现在我们来启动一下试试看：

![image-20220421110926077](https://s2.loli.net/2023/03/08/NApBodythmfjzMV.jpg)

结果发现，消息可以成功发送到消息队列，这就是使用头部信息进行路由。

这样，我们就介绍完了所有四种类型的交换机。

### 集群搭建

前面我们对于RabbitMQ的相关内容已经基本讲解完毕了，最后我们来尝试搭建一个集群，让RabbitMQ之间进行数据复制（镜像模式）稍微有点麻烦，跟着视频走吧。

可能会用到的一些命令：

```sh
sudo rabbitmqctl stop_app
sudo rabbitmqctl join_cluster rabbit@ubuntu-server
sudo rabbitmqctl start_app
```

实现复制即可。

***

## SpringCloud 消息组件

前面我们已经学习了如何使用RabbitMQ消息队列，接着我们来简单介绍一下SpringCloud为我们提供的一些消息组件。

### SpringCloud Stream

**官方文档：**https://docs.spring.io/spring-cloud-stream/docs/3.2.2/reference/html/

前面我们介绍了RabbitMQ，了解了消息队列相关的一些操作，但是可能我们会遇到不同的系统在用不同的消息队列，比如系统A用的Kafka、系统B用的RabbitMQ，但是我们现在又没有学习过Kafka，那么怎么办呢？有没有一种方式像JDBC一样，我们只需要关心SQL和业务本身，而不用关心数据库的具体实现呢？

SpringCloud Stream能够做到，它能够屏蔽底层实现，我们使用统一的消息队列操作方式就能操作多种不同类型的消息队列。

![image-20220421225215709](https://s2.loli.net/2023/03/08/VWvry9TSDBinatH.jpg)

它屏蔽了RabbitMQ底层操作，让我们使用统一的Input和Output形式，以Binder为中间件，这样就算我们切换了不同的消息队列，也无需修改代码，而具体某种消息队列的底层实现是交给Stream在做的。

这里我们创建一个新的项目来测试一下：

![image-20220421215534386](https://s2.loli.net/2023/03/08/pJefuIUXzNHhsxP.jpg)

依赖如下：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-dependencies</artifactId>
    <version>2021.0.1</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

```xml
<dependencies>
    <!--  RabbitMQ的Stream实现  -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

首先我们来编写一下生产者，首先是配置文件：

```yaml
server:
  port: 8001
spring:
  cloud:
    stream:
      binders:   #此处配置要绑定的rabbitmq的服务信息
        local-server: #绑定名称，随便起一个就行
          type: rabbit #消息组件类型，这里使用的是RabbitMQ，就填写rabbit
          environment:  #服务器相关信息，按照下面的方式填写就行，爆红别管
            spring:
              rabbitmq:
                host: 192.168.0.6
                port: 5672
                username: admin
                password: admin
                virtual-host: /test
       bindings:
        test-out-0:
          destination: test.exchange
```

接着我们来编写一个Controller，一会访问一次这个接口，就向消息队列发送一个数据：

```java
@RestController
public class PublishController {

    @Resource
    StreamBridge bridge;  //通过bridge来发送消息

    @RequestMapping("/publish")
    public String publish(){
        //第一个参数其实就是RabbitMQ的交换机名称（数据会发送给这个交换机，到达哪个消息队列，不由我们决定）
      	//这个交换机的命名稍微有一些规则:
      	//输入:    <名称> + -in- + <index>
      	//输出:    <名称> + -out- + <index>
      	//这里我们使用输出的方式，来将数据发送到消息队列，注意这里的名称会和之后的消费者Bean名称进行对应
        bridge.send("test-out-0", "HelloWorld!");
        return "消息发送成功！"+new Date();
    }
}
```

现在我们来将生产者启动一下，访问一下接口：

![image-20220421220955906](https://s2.loli.net/2023/03/08/pvc8udVL9EwMW56.jpg)

可以看到消息成功发送，我们来看看RabbitMQ这边的情况：

![image-20220421221027145](https://s2.loli.net/2023/03/08/1fBHoQe6gc7XizO.jpg)

新增了一个`test-in-0`交换机，并且此交换机是topic类型的：

![image-20220421221107547](https://s2.loli.net/2023/03/08/mN4EfOehP8Ta2JC.jpg)

但是目前没有任何队列绑定到此交换机上，因此我们刚刚发送的消息实际上是没有给到任何队列的。

接着我们来编写一下消费者，消费者的编写方式比较特别，只需要定义一个Consumer就可以了，其他配置保持一致：

```java
@Component
public class ConsumerComponent {

    @Bean("test")   //注意这里需要填写我们前面交换机名称中"名称"，这样生产者发送的数据才会正确到达
    public Consumer<String> consumer(){
        return System.out::println;
    }
}
```

配置中需要修改一下目标交换机：

```yaml
server:
  port: 8002
spring:
  cloud:
    stream:
    	...
      bindings:
      	#因为消费者是输入，默认名称为 方法名-in-index，这里我们将其指定为我们刚刚定义的交换机
        test-in-0:
          destination: test.exchange
```

接着我们直接启动就可以了，可以看到启动之后，自动为我们创建了一个新的队列：

![image-20220421221733723](https://s2.loli.net/2023/03/08/kUelcRgb7MrGdB6.jpg)

而这个队列实际上就是我们消费者等待数据到达的队列：

![image-20220421221807577](https://s2.loli.net/2023/03/08/lzDjiI9SLH1rVY3.jpg)

可以看到当前队列直接绑定到了我们刚刚创建的交换机上，并且`routingKey`是直接写的`#`，也就是说一会消息会直接过来。

现在我们再来访问一些消息发送接口：

![image-20220421221938730](https://s2.loli.net/2023/03/08/cSPRdoY43gzVNXk.jpg)

![image-20220421221952663](https://s2.loli.net/2023/03/08/8TEv1KQGSNA9luY.jpg)

可以看到消费者成功地进行消费了：

![image-20220421222011924](https://s2.loli.net/2023/03/08/lICtpeK2oAGZynD.jpg)

这样，我们就通过使用SpringCloud Stream来屏蔽掉底层RabbitMQ来直接进行消息的操作了。

### SpringCloud Bus

**官方文档：**https://cloud.spring.io/spring-cloud-bus/reference/html/

实际上它就相当于是一个消息总线，可用于向各个服务广播某些状态的更改（比如云端配置更改，可以结合Config组件实现动态更新配置，当然我们前面学习的Nacos其实已经包含这个功能了）或其他管理指令。

这里我们也是简单使用一下吧，Bus需要基于一个具体的消息队列实现，比如RabbitMQ或是Kafka，这里我们依然使用RabbitMQ。

我们将最开始的微服务拆分项目继续使用，比如现在我们希望借阅服务的某个接口调用时，能够给用户服务和图书服务发送一个通知，首先是依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

接着我们只需要在配置文件中将RabbitMQ的相关信息配置：

```yaml
spring:
  rabbitmq:
    addresses: 192.168.0.6
    username: admin
    password: admin
    virtual-host: /test
management:
  endpoints:
    web:
      exposure:
        include: "*"    #暴露端点，一会用于提醒刷新
```

然后启动我们的三个服务器，可以看到在管理面板中：

![image-20220421232118952](https://s2.loli.net/2023/03/08/UfTVhAiOnMqoPX7.jpg)

新增了springCloudBug这样一个交换机，并且：

![image-20220421232146646](https://s2.loli.net/2023/03/08/2VdCOuPLAb9Qhfx.jpg)

自动生成了各自的消息队列，这样就可以监听并接收到消息了。

现在我们访问一个端口：

![image-20220421233200950](https://s2.loli.net/2023/03/08/H3szAX82xhpWw6j.jpg)

此端口是用于通知别人进行刷新，可以看到调用之后，消息队列中成功出现了一次消费：

![image-20220421233302328](https://s2.loli.net/2023/03/08/LoviBfecC1DbMOg.jpg)

现在我们结合之前使用的Config配置中心，来看看是不是可以做到通知之后所有的配置动态刷新了。
