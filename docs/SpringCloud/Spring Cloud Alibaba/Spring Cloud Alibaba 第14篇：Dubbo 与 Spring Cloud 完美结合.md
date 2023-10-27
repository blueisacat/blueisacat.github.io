---
layout: default
title: Spring Cloud Alibaba 第14篇：Dubbo 与 Spring Cloud 完美结合
parent: SpringCloudAlibaba系列教程
grand_parent: SpringCloud
nav_order: 1.14
---

# 1. 概述

可能说起来Dubbo，很多人都不陌生，这毕竟是一款从2012年就开始开源的Java RPC框架，中间由于各种各样的原因停止更新4年半的时间，中间只发过一个小版本修了一个小bug，甚至大家都以为这个项目已经死掉了，竟然又在2017年9月份恢复了更新，不可谓不神奇。

网络上很多人都拿Dubbo和Spring Cloud做对比，可能在大家的心目中，这两个框架是可以画上等号的吧，后来在网络上有一个非常流行的表格，比较详细的对比了 Spring Cloud 和 Dubbo ，表格如下：

|   | Dubbo | SpringCloud | 
| -- | -- | -- |
| 服务注册中心 | Zookeeper | Spring Cloud Netfix Eureka | 
| 服务调用方式 | RPC | REST API | 
| 服务监控 | Dubbo-monitor | Spring Boot Admin | 
| 熔断器 | 不完善 | Spring Cloud Netflix Hystrix | 
| 服务网关 | 无 | Spring Cloud Netflix Zuul | 
| 分布式配置 | 无 | Spring Cloud Config | 
| 服务跟踪 | 无 | Spring Cloud Sleuth | 
| 数据流 | 无 | Spring Cloud Stream | 
| 批量任务 | 无 | Spring Cloud Task | 
| 信息总线 | 无 | Spring Cloud Bus | 


以上列举了一些核心部件，当然这里需要申明一点，Dubbo对于上表中总结为“无”的组件不代表不能实现，而只是Dubbo框架自身不提供，需要另外整合以实现对应的功能，这样看起来确实Dubbo更像是Spring Cloud的一个子集。

Dubbo 在国内拥有着巨大的用户群，大家希望在使用 Dubbo 的同时享受 Spring Cloud 的生态，出现各种各样的整合方案，但是因为服务中心的不同，各种整合方案并不是那么自然，直到 Spring Cloud Alibaba 这个项目出现，由官方提供了 Nacos 服务注册中心后，才将这个问题完美的解决。并且提供了 Dubbo 和 Spring Cloud 整合的方案，命名为： Dubbo Spring Cloud 。

## 1.1 Dubbo Spring Cloud 概述

Dubbo Spring Cloud 构建在原生的 Spring Cloud 之上，其服务治理方面的能力可认为是 Spring Cloud Plus， 不仅完全覆盖 Spring Cloud 原生特性，而且提供更为稳定和成熟的实现，特性比对如下表所示：

| 功能组件 | Spring Cloud | Dubbo Spring Cloud | 
| -- | -- | -- |
| 分布式配置（Distributed configuration） | Git、Zookeeper、Consul、JDBC | Spring Cloud 分布式配置 + Dubbo 配置中心 | 
| 服务注册与发现（Service registration and discovery） | Eureka、Zookeeper、Consul | Spring Cloud 原生注册中心 + Dubbo 原生注册中心 | 
| 负载均衡（Load balancing） | Ribbon（随机、轮询等算法） | Dubbo 内建实现（随机、轮询等算法 + 权重等特性） | 
| 服务熔断（Circuit Breakers） | Spring Cloud Hystrix | Spring Cloud Hystrix + Alibaba Sentinel 等 | 
| 服务调用（Service-to-service calls） | Open Feign、RestTemplate | Spring Cloud 服务调用 + Dubbo @Reference | 
| 链路跟踪（Tracing） | Spring Cloud Sleuth + Zipkin | Zipkin、opentracing 等 | 


以上对比表格摘自Dubbo Spring Cloud官方文档。

而且Dubbo Spring Cloud 基于 Dubbo Spring Boot 2.7.1 和 Spring Cloud 2.x 开发，无论开发人员是 Dubbo 用户还是 Spring Cloud 用户， 都能轻松地驾驭，并以接近“零”成本的代价使应用向上迁移。Dubbo Spring Cloud 致力于简化云原生开发成本，以达成提高研发效能以及提升应用性能等目的。

## 1.2 Dubbo Spring Cloud 主要特性

- 面向接口代理的高性能RPC调用：提供高性能的基于代理的远程调用能力，服务以接口为粒度，屏蔽了远程调用底层细节。

- 智能负载均衡：内置多种负载均衡策略，智能感知下游节点健康状况，显著减少调用延迟，提高系统吞吐量。

- 服务自动注册与发现：支持多种注册中心服务，服务实例上下线实时感知。

- 高度可扩展能力：遵循微内核+插件的设计原则，所有核心能力如Protocol、Transport、Serialization被设计为扩展点，平等对待内置实现和第三方实现。

- 运行期流量调度：内置条件、脚本等路由策略，通过配置不同的路由规则，轻松实现灰度发布，同机房优先等功能。

- 可视化的服务治理与运维：提供丰富服务治理、运维工具：随时查询服务元数据、服务健康状态及调用统计，实时下发路由策略、调整配置参数。

## 1.3 Spring Cloud 为什么需要RPC

在Spring Cloud构建的微服务系统中，大多数的开发者使用都是官方提供的Feign组件来进行内部服务通信，这种声明式的HTTP客户端使用起来非常的简洁、方便、优雅，但是有一点，在使用Feign消费服务的时候，相比较Dubbo这种RPC框架而言，性能堪忧。

虽说在微服务架构中，会讲按照业务划分的微服务独立部署，并且运行在各自的进程中。微服务之间的通信更加倾向于使用HTTP这种简答的通信机制，大多数情况都会使用REST API。这种通信方式非常的简洁高效，并且和开发平台、语言无关，但是通常情况下，HTTP并不会开启KeepAlive功能，即当前连接为短连接，短连接的缺点是每次请求都需要建立TCP连接，这使得其效率变的相当低下。

对外部提供REST API服务是一件非常好的事情，但是如果内部调用也是使用HTTP调用方式，就会显得显得性能低下，Spring Cloud默认使用的Feign组件进行内部服务调用就是使用的HTTP协议进行调用，这时，我们如果内部服务使用RPC调用，对外使用REST API，将会是一个非常不错的选择，恰巧，Dubbo Spring Cloud给了我们这种选择的实现方式。

# 2. 实战

本小结将会以一个简单的入门案例，介绍一下在使用Nacos作为服务中心，使用Dubbo来实现服务提供方和服务消费方的案例。

Nacos的安装、部署配置和使用已经在前面的章节介绍过了，这里不再赘述，如果还有不清楚的读者，请参考前面的Nacos系列文章：

- 《Spring Cloud Alibaba | Nacos服务中心初探》

- 《Spring Cloud Alibaba | Nacos服务注册与发现》

- 《Spring Cloud Alibaba | Nacos集群部署》

- 《Spring Cloud Alibaba | Nacos配置管理》

# 2.1 创建父工程 dubbo-spring-cloud-demo

父工程依赖pom.xml如下：

```
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>${spring-cloud-alibaba.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- Dubbo Spring Cloud Starter -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-dubbo</artifactId>
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

注意：

1. 必须包含spring-boot-starter-actuator包，不然启动会报错。

1. spring-cloud-starter-dubbo包需要注意groupId，根据具体使用的spring cloud alibaba版本依赖来确定。

- 如果使用孵化版本，使用的groupId为：org.springframework.cloud

- 如果使用毕业版本，使用的groupId为：com.alibaba.cloud

1. 以上引用未指定版本，需显示的声明<dependencyManagement>

## 2.2 创建子工程 dubbo_api

API模块，存放Dubbo服务接口和模型定义，非必要，这里创建仅为更好的代码重用以及接口、模型规格控制管理。

定义抽象接口HelloService.java：

```
public interface HelloService {
    String hello(String name);
}
```

## 2.3 创建子工程 dubbo_provider ，Dubbo服务提供方

工程依赖pom.xml如下：

```
<!-- API -->
<dependency>
    <groupId>com.springcloud.book</groupId>
    <artifactId>ch13_1_dubbo_api</artifactId>
    <version>${project.version}</version>
</dependency>
```

此处引入公共API模块。

实现Dubbo接口，HelloServiceI.java如下：

```
@Service
public class HelloServiceI implements HelloService {
    @Override
    public String hello(String name) {
        return "Hello " + name;
    }
}
```

注意：这里的@Service注解并不是来自Spring的org.springframework.stereotype.Service，而是Dubbo的org.apache.dubbo.config.annotation.Service，千万不要引用错误。这里的@Service注解仅声明该Java服务（本地）实现为Dubbo服务。

配置文件 application.yml 需要将Java服务（本地）配置为 Dubbo 服务（远程）如下：

```
server:
port: 8000
dubbo:
    scan:
        base-packages: com.springcloud.book.ch13_1_dubbo_provider.service
protocol:
    name: dubbo
    port: -1
registry:
    address: spring-cloud://192.168.44.129
spring:
application:
    name: dubbo-spring-cloud-provider
cloud:
    nacos:
    discovery:
        server-addr: 192.168.44.129:8848
main:
    allow-bean-definition-overriding: true
```

注意：在暴露Dubbo服务方面，推荐使用外部化配置的方式，即指定Java服务实现类的扫描基准包。

> Dubbo Spring Cloud 继承了 Dubbo Spring Boot 的外部化配置特性，也可以通过标注 @DubboComponentScan 来实现基准包扫描。


- dubbo.scan.base-packages：指定 Dubbo 服务实现类的扫描基准包

- dubbo.protocol：Dubbo服务暴露的协议配置，其中子属性name为协议名称，port为协议端口（-1 表示自增端口，从 20880 开始）

- dubbo.registry：Dubbo 服务注册中心配置，其中子属性address 的值 "spring-cloud://192.168.44.129"，说明挂载到 Spring Cloud 注册中心

- spring.application.name：Spring 应用名称，用于 Spring Cloud 服务注册和发现。该值在 Dubbo Spring Cloud 加持下被视作dubbo.application.name，因此，无需再显示地配置dubbo.application.name。

- spring.main.allow-bean-definition-overriding：在 Spring Boot 2.1 以及更高的版本增加该设定，因为 Spring Boot 默认调整了 Bean 定义覆盖行为。

- spring.cloud.nacos.discovery：Nacos 服务发现与注册配置，其中子属性 server-addr 指定 Nacos 服务器主机和端口。

创建应用主类Ch131DubboProviderApplication.java：

```
@SpringBootApplication
@EnableDiscoveryClient
public class DubboProviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(DubboProviderApplication.class, args);
    }
}
```

## 2.4 创建子工程 dubbo_consumer ，服务调用方

工程依赖pom.xml如下：

```
<!-- API -->
<dependency>
    <groupId>com.springcloud.book</groupId>
    <artifactId>ch13_1_dubbo_api</artifactId>
    <version>${project.version}</version>
</dependency>
```

工程配置application.yml如下：

```
server:
port: 8080
dubbo:
protocol:
    name: dubbo
    port: -1
registry:
    address: spring-cloud://192.168.44.129
cloud:
    subscribed-services: dubbo-spring-cloud-provider
spring:
application:
    name: dubbo-spring-cloud-consumer
cloud:
    nacos:
    discovery:
        server-addr: 192.168.44.129:8848
main:
    allow-bean-definition-overriding: true
```

- dubbo.cloud.subscribed-services：表示要订阅服务的服务名，可以配置'*'，代表订阅所有服务，不推荐使用。若需订阅多应用，使用 "," 分割。

测试接口HelloController.java如下：

```
@RestController
public class HelloController {
    @Reference
    private HelloService helloService;
    @GetMapping("/hello")
    public String hello() {
        return helloService.hello("Dubbo!");
    }
}
```

注意：这里的@Reference注解是org.apache.dubbo.config.annotation.Reference。

启动主类Ch131DubboConsumerApplication.java如下：

```
@SpringBootApplication
@EnableDiscoveryClient
public class DubboConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(DubboConsumerApplication.class, args);
    }
}
```

## 2.5 测试

启动子工程 dubbo_provider 和子工程 dubbo_consumer ，启动完成后，我们可以访问Nacos控制台的服务列表上看到两个服务，如图：

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第14篇：Dubbo%20与%20Spring%20Cloud%20完美结合_image_0.png)

我们打开浏览器访问：[http://localhost:8080/hello](http://localhost:8080/hello) ，可以看到页面正常显示Hello Dubbo!，测试成功，如图：

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第14篇：Dubbo%20与%20Spring%20Cloud%20完美结合_image_1.png)