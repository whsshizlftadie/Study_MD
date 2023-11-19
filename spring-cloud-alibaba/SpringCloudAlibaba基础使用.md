# SpringCloud Alibaba基础使用



## 基础服务的拆分搭建（简单例子）

service dao controller和普通springboot一致，接下来展示maven构造pom.xml的过程

### 项目结构展示图

![image-20231107224505889](D:\note-md\spring-cloud-alibaba\image-20231107224505889.png)

### 搭建父亲模块

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.whs.springcloud_alibaba.study</groupId>
    <artifactId>spting-cloud-alibaba</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>shop-common</module>
        <module>shop-user</module>
        <module>shop-product</module>
        <module>shop-order</module>
    </modules>


    <!--版本依赖锁定-->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <java.version>1.8</java.version>
        <spring-cloud-alibaba.version>2.2.0.RELEASE</spring-cloud-alibaba.version>
        <spring-cloud.version>Hoxton.RELEASE</spring-cloud.version>
        <spring-boot.version>2.2.0.RELEASE</spring-boot.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring-cloud-alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>8.0.28</version>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>


    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>


</project>
```



### 搭建用户模块

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spting-cloud-alibaba</artifactId>
        <groupId>org.whs.springcloud_alibaba.study</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>shop-user</artifactId>


    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.whs.springcloud_alibaba.study</groupId>
            <artifactId>shop-common</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
</project>
```

### 搭建产品模块

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spting-cloud-alibaba</artifactId>
        <groupId>org.whs.springcloud_alibaba.study</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>shop-product</artifactId>

    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.whs.springcloud_alibaba.study</groupId>
            <artifactId>shop-common</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>


</project>
```



### 搭建订单模块

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spting-cloud-alibaba</artifactId>
        <groupId>org.whs.springcloud_alibaba.study</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>shop-order</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.whs.springcloud_alibaba.study</groupId>
            <artifactId>shop-common</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
    </dependencies>


</project>
```

### 三个子模块配置文件一致

使用nacos注册中心

```yml
spring:
  application:
    name: product-server
  datasource:
    username: root
    password: 123456
    url: jdbc:mysql://lohcalhost:3306/shop?useSSL=false&useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai&autoReconnect=true&rewriteBatchedStatements=true&allowMultiQueries=true
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:
    show-sql: true
    hibernate:
      ddl-auto: update
      use-new-id-generator-mappings: true
  jackson:
    serialization:
      indent_output: false
  cloud:
    nacos:
      discovery:
        server-addr: http://localhost:8848
```

主类配上注解@EnableDiscoveryClient才能被nacos注册发现

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@EnableDiscoveryClient
@SpringBootApplication
public class OrderApp {

    public static void main(String[] args) {
        SpringApplication.run(OrderApp.class, args);
    }

}

```



## 服务间调用

### 方法1：RestTemplate

#### @Bean 放入Spring容器

```java
@EnableDiscoveryClient
@SpringBootApplication
public class OrderApp {

    public static void main(String[] args) {
        SpringApplication.run(OrderApp.class, args);
    }
	@Bean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}

```

#### 调用服务

```java
	 @Autowired
    private OrderDao orderDao;

    @Autowired
    private RestTemplate restTemplate;

	@Transactional
    @Override
    public Order order(Integer pid) {

		//get方法获取
        Product product = restTemplate.getForObject("http://localhost:8072/product/" + pid, Product.class);
        
        Order order = new Order();
        order.setNumber(1);
        order.setpName(product.getpName());
        order.setPid(pid);
        order.setpPrice(product.getpPrice());
        order.setUid(1);
        order.setUsername("test");
        orderDao.save(order);
        
        return order;
    }
```

#### 自定义负载均衡

```java
	 @Autowired
    private OrderDao orderDao;

    @Autowired
    private RestTemplate restTemplate;

	@Autowired
    private DiscoveryClient discoveryClient;

	@Transactional
    @Override
    public Order order(Integer pid) {
        //拿到名为product-server服务的服务信息列表（该服务必须注册到了nacos上）
        List<ServiceInstance> instances = discoveryClient.getInstances("product-server");

        //对于列表中服务数量随机生成index，做随机访问（也可以做成轮询）
        int i = new Random().nextInt(instances.size());
		//根据随机数，从列表中拿出服务相关信息对象（包含host，端口等信息）
        ServiceInstance serviceInstance = instances.get(i);

		//get方法获取
        Product product = 		restTemplate
            .getForObject("http://"+
                          serviceInstance.getHost()+//获取服务host，比如localhost
                          ":"+serviceInstance.getPort()//获取端口 如8080
                          +"/product/" + pid, Product.class);
        
        Order order = new Order();
        order.setNumber(1);
        order.setpName(product.getpName());
        order.setPid(pid);
        order.setpPrice(product.getpPrice());
        order.setUid(1);
        order.setUsername("test");
        orderDao.save(order);
        
        return order;
    }
```

#### Ribbon负载均衡

给RestTemplate加上注解 

```java
	@Bean
    @LoadBalanced //开启支持负载均衡
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
```

使用不再需要DiscoveryClient获取服务信息列表，直接通过输入服务名即可发起访问（默认采用轮询）

```java
 	@Autowired
    private OrderDao orderDao;

    @Autowired
    private RestTemplate restTemplate;



    @Transactional
    @Override
    public Order order(Integer pid) {

        //直接在url中写服务注册的名称
        Product product = restTemplate.getForObject("http://product-server/product/" + pid, Product.class);
        Order order = new Order();
        order.setNumber(1);
        order.setpName(product.getpName());
        order.setPid(pid);
        order.setpPrice(product.getpPrice());
        order.setUid(1);
        order.setUsername("test");
        orderDao.save(order);
        restTemplate.getForObject("http://product-server/product/del/" + pid, Boolean.class);
        return order;
    }
```

配置文件中指定负债均衡策略

```yaml
product-server: #这里写的是nacos注册的名称，spring.application.name可以指定
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule //采用随机访问
```

Ribbon所支持的负载均衡策略如下：

轮询（默认）

```yaml
springcloud-nacos-provider: # nacos中的服务id
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RoundRobinRule #设置负载均衡
```

随机策略

```yaml
springcloud-nacos-provider: # nacos中的服务id
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule #设置负载均衡
```

权重策略

权重策略：WeightedResponseTimeRule，根据每个服务提供者的响应时间分配一个权重，响应时间越长，权重越小，被选中的可能性也就越低。它的实现原理是，刚开始使用轮询策略并开启一个计时器，每一段时间收集一次所有服务提供者的平均响应时间，然后再给每个服务提供者附上一个权重，权重越高被选中的概率也越大。

```yaml
springcloud-nacos-provider: # nacos中的服务id
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.WeightedResponseTimeRule
```

最小连接数策略

BestAvailableRule，也叫最小并发数策略，它是遍历服务提供者列表，选取连接数最小的⼀个服务实例。如果有相同的最小连接数，那么会调用轮询策略进行选取。

```yaml
springcloud-nacos-provider: # nacos中的服务id
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.BestAvailableRule #设置负载均衡
```

重试策略

RetryRule，按照轮询策略来获取服务，如果获取的服务实例为 null 或已经失效，则在指定的时间之内不断地进行重试来获取服务，如果超过指定时间依然没获取到服务实例则返回 null

```yaml
ribbon:
  ConnectTimeout: 2000 # 请求连接的超时时间
  ReadTimeout: 5000 # 请求处理的超时时间
springcloud-nacos-provider: # nacos 中的服务 id
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule #设置负载均衡
```

可用性敏感策略

可用敏感性策略：AvailabilityFilteringRule，先过滤掉非健康的服务实例，然后再选择连接数较小的服务实例。

```yaml
springcloud-nacos-provider: # nacos中的服务id
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.AvailabilityFilteringRule
```

区域敏感策略

区域敏感策略：ZoneAvoidanceRule，根据服务所在区域（zone）的性能和服务的可用性来选择服务实例，在没有区域的环境下，该策略和轮询策略类似

```yaml
springcloud-nacos-provider: # nacos中的服务id
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.ZoneAvoidanceRule
```



### 方法2：Feign

#### RestTemplate的坏处

1-直接在代码中拼接字符串url，代码可读性差

2-编程风格不统一

#### maven依赖

```xml
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```

#### 主类加上启动Feign注解

```java
@EnableDiscoveryClient
@SpringBootApplication
@EnableFeignClients //开启Feign
public class OrderApp {

    public static void main(String[] args) {
        SpringApplication.run(OrderApp.class, args);
    }

}
```

#### 创建feign服务接口

```java
import com.whs.springcloud_alibaba.study.domain.Product;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;

@FeignClient("product-server") //声明调用的服务的名字
public interface ProductFeign {

    //@FeignClient+@RequestMapping 就是一个完整的请求路径http://product-server/product/{pid}
    @RequestMapping(value = "/product/{pid}")
    Product product(@PathVariable("pid") Integer pid);//在这里要在PathVariable中先写入参数名称不然会报错

    @RequestMapping("/product/del/{pid}")
    Boolean delNum(@PathVariable("pid") Integer pid);
    
}
```

#### 使用Feign调用

```java
@Service
public class OrderServiceImpl implements OrderService {


    @Autowired
    private OrderDao orderDao;

    @Autowired
    private RestTemplate restTemplate;


    @Autowired 
    private ProductFeign productFeign;

    @Transactional
    @Override
    public Order order(Integer pid) {
        Product product = productFeign.product(pid);//直接使用接口即可，很方便
        Order order = new Order();
        order.setNumber(1);
        order.setpName(product.getpName());
        order.setPid(pid);
        order.setpPrice(product.getpPrice());
        order.setUid(1);
        order.setUsername("test");
        orderDao.save(order);
        productFeign.delNum(pid);
        return order;
    }
}
```

#### 关于Feign超时问题(两种方案)

##### 方案一

```yaml
# feign调用超时时间配置
feign:
  client:
    config:
      default:
        connectTimeout: 600000
        readTimeout: 600000
  hystrix:
    enabled: false    # 不要开启hystrix，会导致超时配置不生效
```

##### 方案二：

```yaml
# 配置 feign 默认请求时间仅几秒钟，配置请求时间长一些(毫秒)
ribbon:
  ReadTimeout: 60000
  ConnectTimeout: 60000
```

#### Feign的负载均衡

feign的负载均衡集成了Ribbon

配置更改负载均衡策略如下：

（1）修改配置文件（和之前演示的RestTemplate的更改负债均衡策略一样）

```yaml
provider001Nacosconfig:#服务提供者name
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```

（2）修改JavaConfig类，在 JavaConfig 类中添加负载 Bean 方法。全局所有feign对应服务都可以生效。

```java
@Configuration
public class FeignConfiguration {
    /**
     * 配置随机的负载均衡策略
     * 特点：对所有的服务都生效
     */
    @Bean
    public IRule loadBalancedRule() {
        return new RandomRule();
    }
}
```

（3）负载均衡器SpringCloudLoadBalancer
由于 Netflix 对于 Ribbon 的维护已经暂停，所以 Spring Cloud 对于负载均衡建议使用由其自己定义的 Spring Cloud LoadBalancer。对于Spring Cloud LoadBalancer 的使用非常简单。
1、关闭Ribbon的负载均衡器

```yaml
spring:
  application:
    name: consumer-name
  cloud:
    loadbalancer:
      # 关闭Ribbon的负载均衡器
      ribbon:
        enabled: false
```

2、pom中添加LoadBalancer依赖

```xml
 <!--spring cloud loadbalancer 依赖--> 
 <dependency> 
      <groupId>org.springframework.cloud</groupId> 
      <artifactId>spring-cloud-starter-loadbalancer</artifactId>
  </dependency>
```

### loadbalancer负载平衡

前面已经提到Ribbon已经停止维护，所以学习loadbanlancer很有必要！

#### ReactiveLoadBalancer

Spring Cloud LoadBalancer中使用ReactiveLoadBalancer接口提供负载均衡的实现，目前只有两个实现：

RoundRobinLoadBalancer：轮询（默认）
RandomLoadBalancer：随机

##### RoundRobinLoadBalancer

关键代码

```java
ServiceInstance instance = instances.get(pos % instances.size());
//很简单求余，轮询
```

##### RandomLoadBalancer

关键代码

```java
		int index = ThreadLocalRandom.current().nextInt(instances.size());

		ServiceInstance instance = instances.get(index);
		//实现也很简单，随机数然后从服务表中获取相关服务访问
```

##### 修改默认的LoadBalancer

```java
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.loadbalancer.core.RandomLoadBalancer;
import org.springframework.cloud.loadbalancer.core.ReactorLoadBalancer;
import org.springframework.cloud.loadbalancer.core.ServiceInstanceListSupplier;
import org.springframework.cloud.loadbalancer.support.LoadBalancerClientFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.core.env.Environment;

@Configuration
public class MyLoadBalancerConfiguration {

    @Bean
    ReactorLoadBalancer<ServiceInstance> randomLoadBalancer(Environment environment,
                                                            LoadBalancerClientFactory loadBalancerClientFactory) {
        String name = environment.getProperty(LoadBalancerClientFactory.PROPERTY_NAME);
        return new RandomLoadBalancer(loadBalancerClientFactory
                .getLazyProvider(name, ServiceInstanceListSupplier.class),name);
    }

}


```

或者

```java
@Configuration
public class MyLoadBalancerConfig {

    @Bean
    @Primary
    public LoadBalancer myLoadBalancer() {
        return new MyCustomLoadBalancer();
    }
}

public class MyCustomLoadBalancer implements LoadBalancer {
    // 实现负载均衡的逻辑...
}

```



## Sentinel组件的使用

### 基本安装以及使用

#### 1-下载和配置及启动

下载地址:[github sentinel jar 包版本下载](https://github.com/alibaba/Sentinel/releases)

下载好版本之后，启动如下：

```bash
java -Dserver.port=8099 -Dcsp.sentinel.dashboard.server=localhost:8099 -Dproject.name=sentinel-center -jar sentinel-dashboard-1.7.1.jar
```

或者配置到IDEA中：

![image-20231119114336933](D:\note-md\spring-cloud-alibaba\image-20231119114336933.png)

接下来对于上游需要限流的服务进行配置

pom.xml

```xml
         <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
```

.yml配置文件

```yaml
spring:
	cloud:
        sentinel:
          transport:
            dashboard: localhost:8099
            port: 9400
    #      eager: true #取消懒加载
```



#### 2-启动相关服务以及Sentinel

按照道理这样就可以启动服务，调用一下接口就可以进行监控了，但是很奇怪监控不到我的资源。。。。。

![image-20231119115500154](D:\note-md\spring-cloud-alibaba\image-20231119115500154.png)

最后在Controller中加上@SentinelResource才能被监控，目前我只有这样解决。。。等后续有时间研究一下为什么。

```java
 	@SentinelResource //不加上这个注解无法被监控
    @RequestMapping("/order/test")
    public String test(){
        return "测试并发";
    }
```

测试调用一下，成功监控到了资源：

![image-20231119115934090](D:\note-md\spring-cloud-alibaba\image-20231119115934090.png)

#### 3-流控规则

当点开+流控可以发现，有很多可以配置的规则：

![image-20231119120136030](D:\note-md\spring-cloud-alibaba\image-20231119120136030.png)

基本的QPS：每秒可以接收的请求

线程数：最大可以接收的线程数：可以用jmeter可以模拟线程并发展示效果。



流控模式：

-直接：只针对当前资源进行流控。

-关联：针对关联的资源，当关联资源达到了流控设置的QPS或者线程数，就会对当前资源进行限流(适合做资源的让步，或者说确定优先级)。

-链路：这个模式很好理解，尤其是在微服务间相互调用情况下尤为常见（a->b->d）

假设我以a为入口资源，d为终点资源，对这条链路进行限制的话，则资源a,b,d均会被限制访问。

##### 链路规则的坑

sentinel需要达到1.7.0及以上，SpringCloudAlibaba 在2.1.1以上可以如下配置。

并且设置：

让[Sentinel](https://so.csdn.net/so/search?q=Sentinel&spm=1001.2101.3001.7020) 源码中 CommonFilter 中的 WEB_CONTEXT_UNIFY 参数为 false，将其配置为 false 即可根据不同的URL 进行链路限流，如果不配置将不会生效。

```yaml
spring:
  cloud:
    sentinel:
      transport:
        dashboard: localhost:8099
        port: 9400
	  # 配置为false
      web-context-unify: false
```



springcloud Alibaba在2.1.1以下或者上述不生效可以尝试如下：

```yaml
    sentinel:
#      web-context-unify: false #关闭context整合
      transport:
        dashboard: localhost:8099
        port: 9400
      filter:
        enabled: false #关闭，并且自定义过滤器

```

引入依赖：

```xml
       <!--需要这个依赖，不然后续配置SentinelContextFilter没有CommonFilter-->
		<dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-web-servlet</artifactId>
        </dependency>
```

配置自定义filter for sentinel context

```java
import com.alibaba.csp.sentinel.adapter.servlet.CommonFilter;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FilterContextConfig {
	/**
     * @NOTE 在spring-cloud-alibaba v2.1.1.RELEASE及前，sentinel1.7.0及后，关闭URL PATH聚合需要通过该方式，spring-cloud-alibaba v2.1.1.RELEASE后，可以通过配置关闭：spring.cloud.sentinel.web-context-unify=false
     * 手动注入Sentinel的过滤器，关闭Sentinel注入CommonFilter实例，修改配置文件中的 spring.cloud.sentinel.filter.enabled=false
     * 入口资源聚合问题：https://github.com/alibaba/Sentinel/issues/1024 或 https://github.com/alibaba/Sentinel/issues/1213
     * 入口资源聚合问题解决：https://github.com/alibaba/Sentinel/pull/1111
     */
    @Bean
    public FilterRegistrationBean sentinelFilterRegistration() {
        FilterRegistrationBean registration = new FilterRegistrationBean();
        registration.setFilter(new CommonFilter());
        registration.addUrlPatterns("/*");
        // 入口资源关闭聚合
        registration.addInitParameter(CommonFilter.WEB_CONTEXT_UNIFY, "false");
        registration.setName("sentinelFilter");
        registration.setOrder(1);
        return registration;
    }
}

```



接下来针对两个链路来模拟一下：

```java
//service中
 	@SentinelResource("lianlu")
    @Override
    public String test() {
        return "链路";
    }
//controller中调用
	@SentinelResource
    @RequestMapping("/order/message")
    public String message(){
        return "测试高并发"+orderService.test();
    }

    @SentinelResource
    @RequestMapping("/order/test")
    public String test(){
        return "测试并发"+orderService.test();
    }
```



成功之后还有一个点。。如果是链路模式下的话限制链路的资源不要写@SentinelResource中设置的名称，不然不生效，后续研究为啥。。关联和直接的话没问题。

#### 4-流控效果

前面默认的都是快速失败：

以下三种;

1-快速失败：达到最大阈值直接抛出异常

2-warm up：一开始阈值设置的最大阈值的三分之一，然后慢慢增长直到最大阈值，适用于将突然增大的流量徒步转换为徒步增长的场景

3-排队等待: 字面意思，设置超时时间，排队时间超过时间限制则失败抛异常。

## 未完。。。。。。
