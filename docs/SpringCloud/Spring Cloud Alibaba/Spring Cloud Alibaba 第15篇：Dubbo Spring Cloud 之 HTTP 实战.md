---
layout: default
title: Spring Cloud Alibaba 第15篇：Dubbo Spring Cloud 之 HTTP 实战
parent: SpringCloudAlibaba
nav_order: 1.15
---

上一篇文章《Spring Cloud Alibaba | Dubbo 与 Spring Cloud 完美结合》我们介绍了Dubbo Spring Cloud的基本使用，使用的服务中心为Spring Cloud Alibaba提供的Nacos，Dubbo内部提供了基于Dubbo的RPC调用，同时，Dubbo Spring Cloud在整合了Spring Cloud之后，可以直接提供HTTP接口，同Spring Cloud无缝衔接，直接支持Feign、RestTemplate等方式的远程调用，在提供HTTP服务的同时可以提供Dubbo服务。Dubbo Spring Cloud支持HTTP远程调用级大的方便了我们的对接外部系统，无需对Dubbo再做二次封装。

# 1. 案例实战

接下来，我们通过一个简单的案例来介绍一下Dubbo Spring Cloud通过注解的方式是如何同时提供Dubbo服务和HTTP服务的。

## 1.1 创建父工程dubbo-spring-cloud-http

工程依赖pom.xml如下：

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- Spring Cloud Nacos Service Discovery -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

## 1.2 创建子工程dubbo_provider_web，服务提供方

工程依赖pom.xml如下：

```
<dependencies>
    <!-- API -->
    <dependency>
        <groupId>com.springcloud</groupId>
        <artifactId>dubbo_api</artifactId>
        <version>${project.version}</version>
    </dependency>
    <!-- Dubbo Spring Cloud Starter -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-dubbo</artifactId>
    </dependency>
</dependencies>
```

这里引入Dubbo Spring Cloud工具包和Dubbo API依赖包。

配置文件参考上一节配置，这里不再赘述。

接口实现类UserServiceI.java如下：

```
@Service(version = "1.0.0")
@RestController
@Slf4j
public class UserServiceI implements UserService {
    private Map<Long, UserModel> usersRepository = Maps.newHashMap();
    @Override
    @PostMapping("/save")
    public UserModel save(@RequestBody UserModel user) {
        return usersRepository.put(user.getId(), user);
    }
    @Override
    @DeleteMapping("/remove")
    public void remove(@RequestParam("id") Long userId) {
        usersRepository.remove(userId);
    }
    @Override
    @GetMapping("/findAll")
    public Collection<UserModel> findAll() {
        return usersRepository.values();
    }
}
```

- @Service注解有很多有关服务的配置属性，这里使用 version 定义当前接口版本，此处版本仅在 Dubbo 调用时生效， HTTP 调用无效，更多相关配置可以参考源码org.apache.dubbo.config.annotation.Service。

## 1.3 创建子工程 spring_cloud_consumer ， web 服务消费方

工程依赖pom.xml如下：

```
<dependencies>
    <!-- API -->
    <dependency>
        <groupId>com.springcloud</groupId>
        <artifactId>dubbo_api</artifactId>
        <version>${project.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
</dependencies>
```

配置文件application.yml如下：

```
server:
port: 8080
spring:
application:
    name: spring-cloud-consumer-server
cloud:
    nacos:
    discovery:
        server-addr: 192.168.44.129:8848
```

接口测试类UserController.java如下：

```
@RestController
public class UserController {
    @Autowired
    UserRemote userRemote;
    @Autowired
    RestTemplate restTemplate;
    @PostMapping("/saveByFeign")
    public UserModel saveByFeign(@RequestBody UserModel user) {
        return userRemote.save(user);
    }
    @DeleteMapping("/removeByFeign")
    public void removeByFeign(@RequestParam("id") Long userId) {
        userRemote.remove(userId);
    }
    @GetMapping("/findAllByFeign")
    public Collection<UserModel> findAllByFeign() {
        return userRemote.findAll();
    }
    @PostMapping("/saveByRestTemplate")
    public UserModel saveByRestTemplate(@RequestBody UserModel user) {
        return restTemplate.postForObject("
    }
    @DeleteMapping("/removeByRestTemplate")
    public void removeByRestTemplate(@RequestParam("id") Long userId) {
        restTemplate.delete("
    }
    @GetMapping("/findAllByRestTemplate")
    public Collection<UserModel> findAllByRestTemplate() {
        return restTemplate.getForObject("
    }
}
```

共计三个测试接口，这里提供两种测试方式，一种是通过Feign调用，另一种是通过RestTemplate调用。

SpringCloudConsumerApplication.java如下：

```
@SpringBootApplication
@EnableFeignClients
@EnableDiscoveryClient
public class SpringCloudConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringCloudConsumerApplication.class, args);
    }
    @LoadBalanced
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

使用@EnableFeignClients开启Feign功能，将RestTemplate以Bean的形式注入Spring中。

## 1.4 创建子工程dubbo_consumer作为Dubbo服务的消费方

接口测试类UserController.java如下：

```
@RestController
public class UserController {
    @Reference(version = "1.0.0")
    UserService userService;
    @PostMapping("/save")
    public UserModel save(@RequestBody UserModel user) {
        return userService.save(user);
    }
    @DeleteMapping("/remove")
    public void remove(@RequestParam("id") Long userId) {
        userService.remove(userId);
    }
    @GetMapping("/findAll")
    public Collection<UserModel> findAll() {
        return userService.findAll();
    }
}
```

这里@Reference注解需指明调用服务提供者接口的版本号，如果未指明版本号，将无法调用我们前面的服务提供者的接口。

# 2. 测试

我们使用测试工具PostMan进行测试，顺次启动三个子工程provider_web、spring_cloud_consumer和dubbo_consumer，首先测试组件Feign访问，使用PostMan向：[http://localhost:8080/saveByFeign](http://localhost:8080/saveByFeign) 发送 POST 请求，如图：

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第15篇：Dubbo%20Spring%20Cloud%20之%20HTTP%20实战_image_0.png)

测试链接：[http://localhost:8080/findAllByFeign](http://localhost:8080/findAllByFeign) ，如图：

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第15篇：Dubbo%20Spring%20Cloud%20之%20HTTP%20实战_image_1.png)

测试 RestTemplate 访问，如图：

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第15篇：Dubbo%20Spring%20Cloud%20之%20HTTP%20实战_image_2.png)

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第15篇：Dubbo%20Spring%20Cloud%20之%20HTTP%20实战_image_3.png)

至此，spring_cloud_consumer测试成功，下面继续测试dubbo_consumer，如图：

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第15篇：Dubbo%20Spring%20Cloud%20之%20HTTP%20实战_image_4.png)

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第15篇：Dubbo%20Spring%20Cloud%20之%20HTTP%20实战_image_5.png)