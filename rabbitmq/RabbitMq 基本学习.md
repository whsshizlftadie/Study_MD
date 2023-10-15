# RabbitMq 基本学习

### 0. 准备工作以及配置

```xml
                <dependency>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-amqp</artifactId>
                </dependency>
                <dependency>
                    <groupId>org.springframework.amqp</groupId>
                    <artifactId>spring-rabbit-test</artifactId>
                    <scope>test</scope>
                </dependency>
```

```yaml
//配置文件 单节点
spring:
  rabbitmq:
    host: 192.168.101.130
    port: 5672
    username: guest
    password: guest
    virtual-host: /   #用于隔离不同应用程序之间的消息队列的逻辑分组，可以理解为命名空间。
```

```yaml
//拉取docker镜像 并启动DOCKER rabbitmq
docker pull rabbitmq:management
docker run -d --hostname localhost --name rabbitmq -p 15672:15672 -p 5672:5672 rabbitmq:management
```



### 1. 工作模式



###### 1-1 普通模式

```java
/**
1，普通模式只需要声明队列即可
**/
@Configuration
public class RabbitSimpleConfig {
    @Bean
    public Queue simpleQueue(){
        return new Queue("simpleQueue");
    }
}
```

```java
/**
简单模式生产者
**/
@SpringBootTest(classes = RabbitmqProducerApplication.class)
public class ProducerTest {
    @Autowired
    RabbitTemplate rabbitTemplate;	
    @Test
    public void simpleProduct(){
        for (int num = 0; num < 20; num++) {
            rabbitTemplate.convertAndSend("simpleQueue", "简单模式"+num);
        }
    }
}
```

```java
/**
简单模式消费者
**/
@Component
public class MessageListener {
    @RabbitListener(queues = "simpleQueue")
    public void simpleListener(String message){
        System.out.println("简单模式监听器："+message);
    }	
}
```





###### 1-2 工作队列模式

```java
 //工作队列模式，同样直需要声明队列
 @Bean
 public Queue workQueue(){
     return new Queue("workQueue");
 }
```

```java
//工作队列模式生产者,和普通模式没有什么区别
@Test
public void workProduct(){
    for (int num = 0; num < 20; num++) {
        rabbitTemplate.convertAndSend("workQueue", "工作模式"+num);
    }
}
```

```java
/**
**多个消费者平均消费，每个消息只被消费一次
**/
 @RabbitListener(queues = "workQueue")
 public void workListener1(String message) {
     System.out.println("工作模式监听器1：" + message);
 }
 
 @RabbitListener(queues = "workQueue")
 public void workListener2(String message) {
     System.out.println("工作模式监听器2：" + message);
 }
```





###### 1-3 发布订阅模式

```java
//配置文件
/**
与前两者简单的声明队列不同，需要声明交换机，并绑定队列到交换机上
**/

//配置FanoutExchange交换器
@Bean
public FanoutExchange fanoutExchange() {
    return new FanoutExchange("fanoutExchange");
}
//配置队列
@Bean
public Queue fanoutQueue1() {
    /**
    声明队列参数也需要设置，与前两者简单的不同
    "fanoutQueue1"：队列的名称。
    true：指定队列是否为持久化队列。如果设置为 true，则 RabbitMQ 服务器会在重启后自动重新创建该队列。
    false：指定队列是否为排他队列。如果设置为 true，则只有创建该队列的连接才能使用它。
    false：指定队列是否为自动删除队列。如果设置为 true，则当没有任何消费者连接到该队列时，RabbitMQ 服务器会自动删除该队列。
    null：用于指定队列的其他参数，例如队列的过期时间等。
    **/
    return new Queue("fanoutQueue1", true, false, false, null);
}
 
@Bean
public Queue fanoutQueue2() {
    return new Queue("fanoutQueue2", true, false, false, null);
}
//配置绑定
@Bean
public Binding fanoutBinding1(FanoutExchange fanoutExchange, Queue fanoutQueue1) {
        return BindingBuilder.bind(fanoutQueue1).to(fanoutExchange);//将队列绑定到交换机，下面同理
}
 
@Bean
public Binding fanoutBinding2(FanoutExchange fanoutExchange, Queue fanoutQueue2) {
    return BindingBuilder.bind(fanoutQueue2).to(fanoutExchange);
}
```



```java
//说明生产者
@Test
public void FanoutProduct(){
    for (int num = 0; num < 10; num++) {
        //与之前的直接发送至队列不同，这里发送到交换机，第二个参数韦routekey在此模式下设为“”即可，第三个参数为传递的消息
        rabbitTemplate.convertAndSend("fanoutExchange","","发布订阅模式"+num);
    }
}
```



```java
//消费者 
/**
生产者的消息，所有队列都会接收并处理，与工作队列只消费一次不同，类似广播
**/
@RabbitListener(queues = "fanoutQueue1")
public void fanoutListener1(String message) {
    System.out.println("发布订阅监听器1：" + message);
}
 
@RabbitListener(queues = "fanoutQueue2")
public void fanoutListener2(String message) {
    System.out.println("发布订阅监听器2：" + message);
}
```





###### 1-4 路由模式

```java
//配置文件
//配置DirectExchange交换机
@Bean
public DirectExchange directExchange() {
    return new DirectExchange("directExchange");
}
 
//配置队列
@Bean
public Queue directQueue1() {
    return new Queue("directQueue1", true, false, false, null);
}
 
@Bean
public Queue directQueue2() {
    return new Queue("directQueue2", true, false, false, null);
}
//配置绑定
@Bean
public Binding directBinding1(Queue directQueue1, DirectExchange directExchange) {
    //与之前的发布订阅模式不同，需要将队列绑定到交换机上并指定routekey，下面同理
    return BindingBuilder.bind(directQueue1).to(directExchange).with("one");
}
 
@Bean
public Binding directBinding2(Queue directQueue2, DirectExchange directExchange) {
    return BindingBuilder.bind(directQueue2).to(directExchange).with("two");
}
```



```java
//生产者
/**
与之前的发布订阅不同的是，第二个routeKey需要指定
根据配置文件里队列绑定到交换机设置的routekey发送到指定队列，一个routeKey对应发送到一个对列
**/
@Test
public void directProduct1() {
    for (int num = 0; num < 5; num++) {
        rabbitTemplate.convertAndSend("directExchange","one", "发送到路由队列1消息"+num);
    }
}
@Test
public void directProduct2() {
    for (int num = 0; num < 5; num++) {
        rabbitTemplate.convertAndSend("directExchange","two", "发送到路由队列2消息"+num);
    }
}
```



```java
//消费者
/**
生产者指定routekey发送到指定队列消费
**/
@RabbitListener(queues = "directQueue1")
public void fanoutListener1(String message) {
    System.out.println("路由模式监听器1：" + message);
}
 
@RabbitListener(queues = "directQueue2")
public void fanoutListener2(String message) {
    System.out.println("路由模式监听器2：" + message);
}
```





###### 1-5 主题模式

```Java
//配置文件
//配置队列
@Bean
public Queue topicQueue1() {
   return new Queue("topicQueue1");
}
 
@Bean
public Queue topicQueue2() {
   return new Queue("topicQueue2");
}
//配置TopicExchange交换器
@Bean
public TopicExchange topicExchange() {
   return new TopicExchange("topicExchange");
}
//配置绑定
/**
    *（星号）：用于匹配一个单词，可以代表任意一个单词。
    #（井号）：用于匹配零个或多个单词。
    举例：
    topic.*：匹配以 "topic." 开头的路由键，例如 "topic.news"、"topic.weather"。
    *.rabbit：匹配以 ".rabbit" 结尾的路由键，例如 "animal.rabbit"、"pet.rabbit"。
    topic.#：匹配以 "topic." 开头的任意长度的路由键，例如	"topic.news"、"topic.news.politics"、"topic.weather.forecast"。
**/
@Bean
public Binding topicBinding1(Queue topicQueue1, TopicExchange topicExchange) {
    //这里不再是一个队列绑定到指定一个routekey上，而是通过通配符表示一系列routekey均可
   return BindingBuilder.bind(topicQueue1).to(topicExchange).with("topic.*");
}
 
@Bean
public Binding topicBinding2(Queue topicQueue2, TopicExchange topicExchange) {
   return BindingBuilder.bind(topicQueue2).to(topicExchange).with("topic.#");
}
```



```java
//生产者
/*
 * 通配符模式测试
 * */
@Test
public void topicProduct() {
    //对应topic.*
    rabbitTemplate.convertAndSend("topicExchange","topic.one", "routkey为topic.one的消息");
    //对应topic.#
    rabbitTemplate.convertAndSend("topicExchange","topic.one.two", "routkey为topic.one.two的消息");
}
```



```java
//消费者
@RabbitListener(queues = "topicQueue1")
public void fanoutListener1(String message) {
    System.out.println("通配符监听器1：" + message);
}
 
@RabbitListener(queues = "topicQueue2")
public void fanoutListener2(String message) {
    System.out.println("通配符监听器2：" + message);
}
```





### 2. 集群配置

#### 2.1，说明

```apl
单一模式：即单机情况不做集群，就单独运行一个 rabbitmq 而已。

普通模式：默认模式，以两个节点（rabbit01、rabbit02）为例，对于 Queue 来说，消息实体只存在于其中一个节点 rabbit01（或者 rabbit02），rabbit01 和 rabbit02 两个节点仅有相同的元数据，即队列的结构。当消息进入 rabbit01 的 Queue 后，consumer 从 rabbit02 消费时，RabbitMQ 会临时在 rabbit01、rabbit02 间进行消息传输，把 A 中的消息实体取出并经过 B 发送给 consumer。

镜像模式: 把需要的队列做成镜像队列，存在与多个节点属于 RabbitMQ 的 HA 方案。该模式解决了普通模式中的问题，其实质和普通模式不同之处在于，消息实体会主动在镜像节点间同步，而不是在客户端取数据时临时拉取。该模式带来的副作用也很明显，除了降低系统性能外，如果镜像队列数量过多，加之大量的消息进入，集群内部的网络带宽将会被这种同步通讯大大消耗掉。所以在对可靠性要求较高的场合中适用。
```

在rabbitmq集群部署时，集群中的节点标示默认都是：rabbit@[hostname]

计划部署3节点的mq集群，，三个节点在不同机器上，为了方便主机名称分别为mq1、mq2、mq3，节点也进行相应的映射；
15672映射为8081 、8082 、8083，5672映射为8071、8072、8073；如下：

| 主机名 | 节点标识   | 控制台端口amqp通信端口 | amqp通信端口 |     Ip地址      |
| ------ | ---------- | ---------------------- | ------------ | :-------------: |
| mq1    | rabbit@mq1 | 8081                   | 8071         | 111.229.160.173 |
| mq2    | rabbit@mq2 | 8082                   | 8072         | 111.229.160.174 |
| mq3    | rabbit@mq3 | 8083                   | 8073         | 111.229.160.175 |



#### 2.2、标准集群模式



#### 2.2.1.创建目录

```bash
mkdir -p /root/docker/rabbitmq-cluster/mq1[2|3]
```



#### 2.2.2 设置hosts

**注意：如果是同一台机器上，搭建不同的docker实例，则不进行设置。**

分别在三台不同的机器上编辑`hosts`文件



```bash
# mq1
vim  /root/docker/rabbitmq-cluster/mq1/hosts
# mq2
vim  /root/docker/rabbitmq-cluster/mq2/hosts
# mq3
vim  /root/docker/rabbitmq-cluster/mq3/hosts

```



#### 2.2.3.设置cookie：

RabbitMQ底层依赖于[Erlang](https://so.csdn.net/so/search?q=Erlang&spm=1001.2101.3001.7020)，而Erlang虚拟机就是一个面向分布式的语言，默认就支持集群模式。集群模式中的每个RabbitMQ 节点使用 cookie 来确定它们是否被允许相互通信。

```bash
# Cookie配置
vim /root/docker/rabbitmq-cluster/mq1/.erlang.cookie

# 配置内容-cookie值（任意值）
UDCUIBNPHPETOIURAHRF

# 修改cookie文件的权限
chmod 600 /root/docker/rabbitmq-cluster/mq1/.erlang.cookie

```



#### 2.2.4、设置配置文件

```bash
# mq1
vim /root/docker/rabbitmq-cluster/mq1/rabbitmq.conf
```

**内容都如下：**

```bash
# 配置内容
loopback_users.guest = false
listeners.tcp.default = 5672
cluster_formation.peer_discovery_backend = rabbit_peer_discovery_classic_config
cluster_formation.classic_config.nodes.1 = rabbit@mq1
cluster_formation.classic_config.nodes.2 = rabbit@mq2
cluster_formation.classic_config.nodes.3 = rabbit@mq3
```

注意rabbit@xxx为每个rabbitmq实例的hostname，和启动容器时的--hostname值一致



#### 2.2.5. 拷贝目录

如果有多台服务器，则将配置好的一个rabbitmq节点的文件，拷贝到其他服务器上；如果是一台服务器，则拷贝到不同的目录下即可。

```bash
# 将mq1目录拷贝为mq2、mq3
cp /root/docker/rabbitmq-cluster/mq1 /root/docker/rabbitmq-cluster/mq2 -r

cp /root/docker/rabbitmq-cluster/mq1 /root/docker/rabbitmq-cluster/mq3 -r

```



#### 2.2.6. 启动容器

**注意：如果如果是同一台机器上，搭建不同的docker实例，则直接创建同一个网络环境**

```bash
#创建网络
docker network create mq-net

```



```bash
docker run -d \
# 如果是同一台机器则需要指定net
--net mq-net \
# 如果是不同机器，则需要挂载hosts文件，如果是同一台机器则不配做hosts挂载
-v /root/docker/rabbitmq-cluster/xxx/hosts:/etc/hosts \
-v /root/docker/rabbitmq-cluster/xxx/rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf \
-v /root/docker/rabbitmq-cluster/xxx/.erlang.cookie:/var/lib/rabbitmq/.erlang.cookie \
-e RABBITMQ_DEFAULT_USER=xxx\
-e RABBITMQ_DEFAULT_PASS=xxx\
--name xxx \
--hostname xxx \
-p xxxx:5672 \
-p xxxx:15672 \
rabbitmq:3.8.27-management

```



**本次搭建，在同一台服务器上完成，所以需要指定net，不需要挂载hosts文件**

```bash
//启动mq1：
docker run -d \
--net mq-net \
-v /root/docker/rabbitmq-cluster/mq1/rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf \
-v /root/docker/rabbitmq-cluster/mq1/.erlang.cookie:/var/lib/rabbitmq/.erlang.cookie \
-e RABBITMQ_DEFAULT_USER=admin \
-e RABBITMQ_DEFAULT_PASS=admin \
--name mq1 \
--hostname mq1 \
-p 8071:5672 \
-p 8081:15672 \
rabbitmq:3.8.27-management

//启动mq2
docker run -d \
--net mq-net \
-v /root/docker/rabbitmq-cluster/mq2/rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf \
-v /root/docker/rabbitmq-cluster/mq2/.erlang.cookie:/var/lib/rabbitmq/.erlang.cookie \
-e RABBITMQ_DEFAULT_USER=admin \
-e RABBITMQ_DEFAULT_PASS=admin \
--name mq2 \
--hostname mq2 \
-p 8072:5672 \
-p 8082:15672 \
rabbitmq:3.8.27-management

//启动mq3
docker run -d \
--net mq-net \
-v /root/docker/rabbitmq-cluster/mq3/rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf \
-v /root/docker/rabbitmq-cluster/mq3/.erlang.cookie:/var/lib/rabbitmq/.erlang.cookie \
-e RABBITMQ_DEFAULT_USER=admin \
-e RABBITMQ_DEFAULT_PASS=admin \
--name mq3 \
--hostname mq3 \
-p 8073:5672 \
-p 8083:15672 \
rabbitmq:3.8.27-management


```



