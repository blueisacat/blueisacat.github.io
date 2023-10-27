---
layout: default
title: Spring Cloud Alibaba 第3篇：Nacos服务注册与发现
parent: SpringCloudAlibaba系列教程
grand_parent: SpringCloud
nav_order: 1.3
---

上一篇《Spring Cloud Alibaba | Nacos服务中心初探》我们聊了什么是Nacos以及Nacos如何搭建，这一篇我们接着聊Nacos如何简单使用。

首先，Nacos是一个服务注册和服务发现的注册中心，在Spring Cloud中，可以替代Eureka的功能，我们先聊一下Nacos如何和Spring Cloud集成做一个注册中心。

整体流程为：

1. 先启动注册中心Nacos

1. 启动服务的提供者将提供服务，并将服务注册到注册中心Nacos上

1. 启动服务的消费者，在Nacos中找到服务并完成消费

# 1. 服务提供者

新建一个producer的项目，项目依赖如下：

## 1.1 pom.xml项目依赖

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="
    xsi:schemaLocation="
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.6.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.springcloud</groupId>
    <artifactId>producer</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>producer</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <version>0.9.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
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

增加Nacos的服务发现的依赖：spring-cloud-starter-alibaba-nacos-discovery，根据版本SpringCloud和SpringBoot的版本，这里我们使用0.9.0.RELEASE版本。其他版本使用请看上一篇《Spring Cloud Alibaba | Nacos服务中心初探》。

## 1.2 配置文件application.yml

```
server:
  port: 9000
spring:
  application:
    name: spring-cloud-nacos-producer
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
```

## 1.3 启动类ProducerApplication.java

```
package com.springcloud.producer;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
@SpringBootApplication
@EnableDiscoveryClient
public class ProducerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProducerApplication.class, args);
    }
}
```

@EnableDiscoveryClient 注册服务至Nacos。

## 1.4 controller

```
package com.springcloud.producer.controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
/**
 * Created with IntelliJ IDEA.
 *
 * @Date: 2019/7/14
 * @Time: 10:04
 * @email: inwsy@hotmail.com
 * Description:
 */
@RestController
public class HelloController {
    @RequestMapping("/hello")
    public String hello(@RequestParam String name) {
        return "hello "+name+"，producer is ready";
    }
}
```

## 1.5 测试

启动服务producer，在浏览器访问链接：[http://localhost:9000/hello?name=nacos](http://localhost:9000/hello?name=nacos)， 可以看到页面显示hello nacos，producer is ready。

打开Nacos显示页面，可以看到服务spring-cloud-nacos-producer正常上线。

![](../../../assets/images/SpringCloud/SpringCloudAlibaba/attachments/Spring%20Cloud%20Alibaba%20第3篇：Nacos服务注册与发现_image_0.png)

到这里，我们的服务提供者已经正常搭建完毕。

# 2. 服务消费者

## 2.1 pom.xml项目依赖

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="
    xsi:schemaLocation="
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.6.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.spring</groupId>
    <artifactId>nacos-cosumers</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>nacos-cosumers</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR2</spring-cloud.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <version>0.9.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
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

这里增加了spring-cloud-starter-openfeign依赖包

## 2.2 配置文件application.yml

```
server:
  port: 8080
spring:
  application:
    name: spring-cloud-nacos-consumers
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
```

## 2.3 启动类NacosCosumersApplication.java

```
package com.spring.nacoscosumers;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class NacosCosumersApplication {
    public static void main(String[] args) {
        SpringApplication.run(NacosCosumersApplication.class, args);
    }
}
```

@EnableFeignClients这个注解是声明Feign远程调用

## 2.4 Feign远程调用

创建一个remote接口

```
package com.spring.nacoscosumers.remote;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
@FeignClient(name= "spring-cloud-nacos-producer")
public interface HelloRemote {
    @RequestMapping(value = "/hello")
    String hello(@RequestParam(value = "name") String name);
}
```

## 2.5 web层调用远程接口 Controller

```
package com.spring.nacoscosumers.controller;
import com.spring.nacoscosumers.remote.HelloRemote;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
/**
 * Created with IntelliJ IDEA.
 *
 * @Date: 2019/7/14
 * @Time: 10:24
 * @email: inwsy@hotmail.com
 * Description:
 */
@RestController
public class HelloController {
    @Autowired
    HelloRemote helloRemote;
    @RequestMapping("/hello/{name}")
    public String index(@PathVariable("name") String name) {
        return helloRemote.hello(name);
    }
}
```

## 2.6 测试

启动服务消费者nacos-consumers，打开浏览器访问链接：[http://localhost:8080/hello/nacos](http://localhost:8080/hello/nacos)， 这时页面正常返回hello nacos，producer is ready，证明我们的已经通过Nacos作为注册中心已经正常提供了服务注册与发现。

# 3. 集成Gateway

上面介绍了Nacos可以和Feign集成使用，更多的情况下，我们需要和API网关来集成。

这里我们还是使用之前的服务提供者，新建一个服务网关。

这里我们使用了Gateway做演示，想使用Zuul的朋友可以作为参考，在原有Zuul+Eureka的基础上只需要更换配置和依赖包就可以，无需其他过多的修改。

## 3.1 pom.xml项目依赖

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="
    xsi:schemaLocation="
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.6.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.spring</groupId>
    <artifactId>nacos-gateway</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>nacos-gateway</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR2</spring-cloud.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <version>0.9.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
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

## 3.2 配置文件application.xml

```
server:
  port: 8088
spring:
  application:
    name: spring-cloud-nacos-gateway
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
    gateway:
      discovery:
        locator:
          enabled: true
      routes:
        - id: hello_route
          #格式为：lb://应用注册服务名
          uri: lb://spring-cloud-nacos-producer
          predicates:
            - Method=GET
```

## 3.3 启动类NacosGatewayApplication.java

```
package com.spring.nacosgateway;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication
public class NacosGatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(NacosGatewayApplication.class, args);
    }
}
```

## 3.4 测试

我们启动Gateway，打开浏览器访问链接：[http://localhost:8088/spring-cloud-nacos-producer/hello?name=nacos](http://localhost:8088/spring-cloud-nacos-producer/hello?name=nacos)， 浏览器正常返回：hello nacos，producer is ready， 证明我们通过服务网关来访问服务是正常的。

最后，我们打开看一下Nacos的服务列表：

![](../../../assets/images/SpringCloud/SpringCloudAlibaba/attachments/Spring%20Cloud%20Alibaba%20第3篇：Nacos服务注册与发现_image_1.png)

这里我把消费者服务停止掉了，可以看到Nacos可以实时的显示出来。

Nacos就介绍到这里，有不清楚的可以给我留言~