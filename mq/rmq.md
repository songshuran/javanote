### RMQ整合Spring使用

* 依赖

  ```xml
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
  </dependency>

  spring:
      rabbitmq:
        host:
        port:
        username:
        password:
        virtual-host:
  ```

* 生产者

  ```java
  @Bean
  public ConnectionFactory connectionFactory() {
    CachingConnectionFactory connectionFactory = new CachingConnectionFactory("localhost",5672);
    //在构造方法传入了
    //        connectionFactory.setHost();
    //        connectionFactory.setPort();
    connectionFactory.setUsername("admin");
    connectionFactory.setPassword("admin");
    connectionFactory.setVirtualHost("testhost");
    //是否开启消息确认机制
    //connectionFactory.setPublisherConfirms(true);
    return connectionFactory;
  }

  @Bean
  public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
    //注意  这个ConnectionFactory 是使用javaconfig方式配置连接的时候才需要传入的  如果是yml配置的连接的话是不需要的
    RabbitTemplate template = new RabbitTemplate(connectionFactory)；
    return template;
  }

  @Bean
  public DirectExchange  defaultExchange() {
    return new DirectExchange("directExchange");
  }

  @Bean
  public Queue queue() {
    //名字  是否持久化
    return new Queue("testQueue", true);
  }

  @Bean
  public Binding binding() {
    //绑定一个队列  to: 绑定到哪个交换机上面 with：绑定的路由建    （routingKey）
    return                 BindingBuilder.bind(queue()).to(defaultExchange()).with("direct.key");
  }

  @Component
  public class TestSend {

    @Autowired
    RabbitTemplate rabbitTemplate;
    public void send() {
        //至于为什么调用这个API 后面会解释
        //参数介绍： 交换机名字，路由建， 消息内容
        rabbitTemplate.convertAndSend("directExchange", "direct.key", "hello");
    }
  }
  ```

* 消费者
  ```java
  @Component
  public class TestListener  {
    @RabbitListener(queues = "testQueue")
    public void get(String message) throws Exception{
        System.out.println(message);
    }
  }
  ```

* 确保消息发送到RMQ  
    使用失败回调的方式，也必须开启发送方确认

  * RabbitmqTemplate
    ```java
    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) { 
      RabbitTemplate template = new RabbitTemplate(connectionFactory); 
      //开启mandatory模式(开启失败回调)
      template.setMandatory(true);
      //指定失败回调接口的实现类 
      template.setReturnCallback(new MyReturnCallback()); 
      return template;
    }
    ```
  * 回调接口
    ```java
    public class MyReturnCallback implements RabbitTemplate.ReturnCallback {
       @Override
      public void returnedMessage(Message message, int replyCode, String replyText,String exchange, String routingKey){
      }
    }
    ```

* 开启发送方确认模式

  ```java
     connectionFactory.setPublisherConfirms(true);
  ```

  * yml
    ```yaml
    spring:
       rabbitmq:
          publisher-confirms: true
    ```
  * 实现接口
    ```java
    public class MyConfirmCallback implements RabbitTemplate.ConfirmCallback{
      @Override
      public void confirm(CorrelationData correlationData, boolean ack, String cause) { 
      } 
    }
    ```
  * RabbitmqTemplate修改
    `template.setConfirmCallback(new MyConfirmCallback());`

* 要注意的是 confirm模式的发送成功 的意思是发送到RabbitMq\(Broker\)成功 而不是发送到队列成功,所以要和失败回调结合使用,这样才能确认消息投递成功了，

  * confirm机制是确认我们的消息是否投递到了 RabbitMq\(Broker\)上面，而        
  * mandatory是在我们的消息进入队列失败时候不会被遗弃\(让我们自己进行处理\)

### 消费者确认

* Container配置

  ```java
  @Bean
  public SimpleRabbitListenerContainerFactory
  simpleRabbitListenerContainerFactory(ConnectionFactory connectionFactory){
    SimpleRabbitListenerContainerFactory simpleRabbitListenerContainerFactory =
        new SimpleRabbitListenerContainerFactory();
    //这个connectionFactory就是我们自己配置的连接工厂直接注入进来             

    simpleRabbitListenerContainerFactory
        .setConnectionFactory(connectionFactory); 
    //这边设置消息确认方式由自动确认变为手动确认     
    simpleRabbitListenerContainerFactory
        .setAcknowledgeMode(AcknowledgeMode.MANUAL); 
    return simpleRabbitListenerContainerFactory;    
  }
  ```

* 注解设置

  ```java
  @Component
  public class TestListener  {    
    //containerFactory:指定我们刚刚配置的容器
  @RabbitListener(queues = "testQueue",containerFactory = "simpleRabbitListenerContainerFactory")
    public void getMessage(Message message, Channel channel) throws Exception{
        System.out.println(new String(message.getBody(),"UTF-8"));
        System.out.println(message.getBody());
        //这里我们调用了一个下单方法  如果下单成功了 那么这条消息就可以确认被消费了
        boolean f =placeAnOrder();
        if (f){
            //传入这条消息的标识， 这个标识由rabbitmq来维护 我们只需要从message中拿出来就可以
            //第二个boolean参数指定是不是批量处理的 什么是批量处理我们待会儿会讲到
            channel.basicAck(message.getMessageProperties().getDeliveryTag(),false);
        }else {
            //当然 如果这个订单处理失败了  我们也需要告诉rabbitmq 告诉他这条消息处理失败了 可以退回 也可以遗弃 要注意的是 无论这条消息成功与否  一定要通知 就算失败了 如果不通知的话 rabbitmq端会显示这条消息一直处于未确认状态，那么这条消息就会一直堆积在rabbitmq端 除非与rabbitmq断开连接 那么他就会把这条消息重新发给别人  所以 一定要记得通知！
            //前两个参数 和上面的意义一样， 最后一个参数 就是这条消息是返回到原队列 还是这条消息作废 就是不退回了。
            channel.basicNack(message.getMessageProperties().getDeliveryTag(),false,true);
            //其实 这个API也可以去告诉rabbitmq这条消息失败了  与basicNack不同之处 就是 他不能批量处理消息结果 只能处理单条消息   其实basicNack作为basicReject的扩展开发出来的 
            //channel.basicReject(message.getMessageProperties().getDeliveryTag(),true);
        }
    }
  }
  ```

### 消费预取

* RMQ消费是采用轮训平均分配的，不管该消费者的消费能力是否快慢，都是平均消费，RMQ是一下子平均分配给所有消费者的。解决方案：消息预取，在使用消息预取前 要注意一定要设置为手动确认消息
* 设置
  ```java
  @Bean
  public SimpleRabbitListenerContainerFactory
  simpleRabbitListenerContainerFactory(ConnectionFactory connectionFactory){
    SimpleRabbitListenerContainerFactory simpleRabbitListenerContainerFactory =
            new SimpleRabbitListenerContainerFactory();
  simpleRabbitListenerContainerFactory.setConnectionFactory(connectionFactory); //手动确认消息 simpleRabbitListenerContainerFactory.setAcknowledgeMode(AcknowledgeMode.MANUAL); //设置消息预取的数量
  simpleRabbitListenerContainerFactory.setPrefetchCount(1);
    return simpleRabbitListenerContainerFactory;
  }
  ```
* 如果设置为1 能极大的利用客户端的性能\(我消费完了就可以赶紧消 费下一条 不会导致忙的很忙 闲的很闲\) 但是， 我们每消费一条消息 就要通知一次rabbitmq 然后再取出新的消 息， 这样对于rabbitmq的性能来讲 是非常不合理的 所以这个参数要根据业务情况设置。这个数值的大小与性能成正比 但是有上限2500

### 死信队列

* 死信交换机有什么用呢? 在创建队列的时候 可以给这个队列附带一个交换机， 那么这个队列作废的消息就会被重新发到附带的交换机，然后让这个交换机重新路由这条消息
  ```java
  @Bean
  public Queue queue() {
    Map<String,Object> map = new HashMap<>();
    //设置消息的过期时间 单位毫秒
    map.put("x-message-ttl",10000);
    //设置附带的死信交换机
    map.put("x-dead-letter-exchange","exchange.dlx");
    //指定重定向的路由建 消息作废之后可以决定需不需要更改他的路由建 如果需要 就在这里指定 map.put("x-dead-letter-routing-key","dead.order");
    return new Queue("testQueue", true,false,false,map);
  }
  ```
* 消息变成死信有以下情况
  * 消息被拒绝\(basic.reject / basic.nack\)，并且requeue = false
  * 消息TTL过期
  * 队列达到最大长度

### RMQ集群

* 集群模式

  * 主备模式
  * 镜像模式：100%数据不丢失
  * 异地多活：

* 集群中的节点分为两种

  * 磁盘节点：会把集群的所有信息\(比如交换机,队列等信息\)持久化到磁盘当中
  * 内存节点：内存节点只会将这些信息保存到内存当中，重启一遍就没了
  * rabbitmq官方强调集群环境至少需要有一个磁盘节点， 而且为了高可用的话， 必须至少要有2个 磁盘节点， 因为如果只有一个磁盘节点 而刚好这唯一的磁盘节点宕机了的话， 集群虽然还是可以运作， 但是不能 对集群进行任何的修改操作\(比如 队列添加，交换机添加，增加/移除 新的节点等\)



