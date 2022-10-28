# ActiveMQ

## 1、配置类（Configuration）

```java
@Configuration
public class BeanConfig {

    @Bean
    //创建队列
    public Queue sendQueue(){
        return new ActiveMQQueue("sendQueue");
    }
}
```

## 2、发送消息（provider）

```java
@RestController
@RequestMapping("/active")
public class SendActiveController {
	
    @Resource
    //将sendQueue注入
    private Queue sendQueue;

    @Resource
    //Template中提供了发送接收的方法
    private JmsMessagingTemplate jmsMessagingTemplate;

    @PostMapping("/message")
    public String send(@RequestBody String data) {
        SendMessage sendMessage = JSONArray.parseObject(data, SendMessage.class);
        //向sendQueue队列发送消息
        jmsMessagingTemplate.convertAndSend(sendQueue,sendMessage.getData());
        System.out.println("发送成功" + "\t" + sendMessage.getData());
        return "发送成功" + "\t" + sendMessage.getData();
    }
}
```

#### Queues

![image-20220303112518518](http://qiniu.ningmengchuxia.xyz/img/image-20220303112518518.png)

## 3、接收消息（consumer）

```java
@Component
public class ReceiveActiveReceiver {

    @Resource
    private JmsMessagingTemplate jmsMessagingTemplate;

    //使用JmsListener配置消费者监听的队列
    @JmsListener(destination = "sendQueue")
    public void handleMessage(String data){
        System.out.println("成功接收data" + "\t" + data);
    }
}
```

### Queues

![image-20220303112719748](http://qiniu.ningmengchuxia.xyz/img/image-20220303112719748.png)

# RabbitMQ

## 1、配置（Configuration）

```java
@Configuration
public class DirectConfig {

    //队列
    @Bean
    public Queue sendDirectQueue(){
        return new Queue("sendDirectQueue",true);
    }

    //交换机
    @Bean
    public DirectExchange sendDirectExchange(){
        return new DirectExchange("sendDirectExchange",true,false);
    }

    //队列和交换机绑定
    @Bean
    public Binding bindingDirect(){
        return BindingBuilder.bind(sendDirectQueue()).to(sendDirectExchange()).with("sendDirectRouting");
    }

    @Bean
    public DirectExchange lonelyDirectExchange(){
        return new DirectExchange("lonelyDirectExchange");
    }
}
```



## 2、发送消息（provider）

```java
@RestController
@RequestMapping("/rabbit")
public class SendMessageController {

    @Resource
    RabbitTemplate rabbitTemplate;

    @GetMapping("/message")
    public String sendDirectMessage(){
        String messageId = String.valueOf(UUID.randomUUID());
        String messageData = "PHP是世界上最好的语言.java";
        String createTime = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
        Map<String, Object> messageMap = new HashMap<>();
        messageMap.put("messageId", messageId);
        messageMap.put("messageData", messageData);
        messageMap.put("createTime", createTime);
        ////将消息携带绑定键值：sendDirectRouting 发送到交换机sendDirectExchange
        rabbitTemplate.convertAndSend("sendDirectExchange", "sendDirectRouting", messageMap);
        return "发送成功";
    }
}
```

### Queues

![image-20220303133439789](http://qiniu.ningmengchuxia.xyz/img/image-20220303133439789.png)

## 3、接收消息（consumer）

```java
@Component
public class DirectReceiver {

    @RabbitListener(queues = "sendDirectQueue")
    @RabbitHandler
    public void process(Map messageMap){
        System.out.println("接收到sendDirectQueue" + "\t" + messageMap.get("messageData"));
    }
}
```



![image-20220303135453276](http://qiniu.ningmengchuxia.xyz/img/image-20220303135453276.png)

![image-20220303135630052](http://qiniu.ningmengchuxia.xyz/img/image-20220303135630052.png)