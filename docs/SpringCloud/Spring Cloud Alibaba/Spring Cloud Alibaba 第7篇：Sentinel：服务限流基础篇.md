# 1. Sentinel控制台概述

在介绍入门实战之前，先来介绍一下Sentinel。Sentinel控制台提供一个轻量级的开源控制台，它提供机器发现以及健康情况管理、监控（单机和集群），规则管理和推送的功能。

Sentinel控制台主要功能：

- 查看机器列表以及健康情况：收集 Sentinel 客户端发送的心跳包，用于判断机器是否在线。

- 监控 (单机和集群聚合)：通过 Sentinel 客户端暴露的监控 API，定期拉取并且聚合应用监控信息，最终可以实现秒级的实时监控。

- 规则管理和推送：统一管理推送规则。

- 鉴权：生产环境中鉴权非常重要，每个用户都需要相应的权限可以对控制台的信息进行修改。

# 2. Sentinel控制台部署

Sentinel控制台部署有两种方案：

## 2.1 下载

直接使用官方编译好的Release版本部署启动，下载地址：[https://github.com/alibaba/Sentinel/releases](https://github.com/alibaba/Sentinel/releases) ，目前最新版本是v1.6.3，下载sentinel-dashboard-1.6.3.jar ，如图：

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第7篇：Sentinel：服务限流基础篇_image_0.png)

## 2.2 启动

可以使用最新版本的源码自行构建Sentinel控制台，首先需要下载控制台工程，下载路径：[https://github.com/alibaba/Sentinel/tree/master/sentinel-dashboard](https://github.com/alibaba/Sentinel/tree/master/sentinel-dashboard) ，使用命令mvn clean package将代码打成jar包即可。

在CentOS中使用如下命令启动Sentinel控制台工程：

> 注意：启动 Sentinel 控制台需要 JDK 版本为 1.8 及以上版本。


```
nohup java -Dserver.port=8080 -Dcsp.sentinel.dashboard.server=localhost:8080 -Dproject.name=sentinel-dashboard -jar sentinel-dashboard-1.6.3.jar >sentinel-dashboard.out 2>&1 &
```

-Dserver.port=8080 用于指定 Sentinel 控制台端口为 8080。启动完成后，使用浏览器访问：[http://ip:8080/](http://ip:8080/) ，这时会进入登录页面，从 Sentinel 1.6.0 起，Sentinel 控制台引入基本的登录功能，默认用户名和密码都是sentinel，如图：

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第7篇：Sentinel：服务限流基础篇_image_1.png)

Sentinel Dashboard可以通过如下参数进行配置：

- -Dsentinel.dashboard.auth.username=sentinel 用于指定控制台的登录用户名为 sentinel；

- -Dsentinel.dashboard.auth.password=123456 用于指定控制台的登录密码为 123456；如果省略这两个参数，默认用户和密码均为 sentinel；

- -Dserver.servlet.session.timeout=7200 用于指定 Spring Boot 服务端 session 的过期时间，如 7200 表示 7200 秒；60m 表示 60 分钟，默认为 30 分钟；

同样也可以直接在Spring的application.properties文件中进行配置。

# 3. Spring Cloud

Sentinel目前已支持Spring Cloud，需要引入spring-boot-starter-web来触发sentinel-starter中相关的自动配置。

## 3.1 创建父工程sentinel-springcloud

工程依赖pom.xml如下：

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
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

spring-cloud-alibaba-dependencies引入Spring Cloud Alibaba版本控制。spring-cloud-starter-alibaba-sentinel引入Sentinel组件。

## 3.2 创建子工程web_mvc

工程依赖pom.xml如下：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

工程配置application.yml如下：

```
server:
  port: 8000
spring:
  application:
    name: web-mvc
  cloud:
    sentinel:
      transport:
        dashboard: 192.168.44.129:8080
        port: 8719
```

- spring.cloud.sentinel.transport.dashboard：配置Sentinel控制台的ip和端口

- spring.cloud.sentinel.transport.port：这个端口配置会在应用对应的机器上启动一个 Http Server，该 Server 会与 Sentinel 控制台做交互。比如 Sentinel 控制台添加了1个限流规则，会把规则数据 push 给这个 Http Server 接收，Http Server 再将规则注册到 Sentinel 中。

创建测试接口HelloController.java如下:

```
@RestController
public class HelloController {
    @GetMapping(value = "/hello")
    @SentinelResource("hello")
    public String hello() {
        return "Hello Web MVC";
    }
}
```

@SentinelResource注解用来标识资源是否被限流、降级。上述例子上该注解的属性 ‘hello’ 表示资源名。该注解还有一些其他更精细化的配置，比如忽略某些异常的配置、默认降级函数等等，具体可见如下说明：

- value：资源名称，必需项（不能为空）

- entryType：entry 类型，可选项（默认为 EntryType.OUT）

- blockHandler / blockHandlerClass: blockHandler 对应处理 BlockException 的函数名称，可选项。blockHandler 函数访问范围需要是 public，返回类型需要与原方法相匹配，参数类型需要和原方法相匹配并且最后加一个额外的参数，类型为 BlockException。blockHandler 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 blockHandlerClass 为对应的类的 Class 对象，注意对应的函数必需为 static 函数，否则无法解析。

- fallback：fallback 函数名称，可选项，用于在抛出异常的时候提供 fallback 处理逻辑。fallback 函数可以针对所有类型的异常（除了 exceptionsToIgnore 里面排除掉的异常类型）进行处理。fallback 函数签名和位置要求：


- 返回值类型必须与原函数返回值类型一致；

- 方法参数列表需要和原函数一致，或者可以额外多一个 Throwable 类型的参数用于接收对应的异常。

- fallback 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 fallbackClass 为对应的类的 Class 对象，注意对应的函数必需为 static 函数，否则无法解析。

- defaultFallback（since 1.6.0）：默认的 fallback 函数名称，可选项，通常用于通用的 fallback 逻辑（即可以用于很多服务或方法）。默认 fallback 函数可以针对所有类型的异常（除了 exceptionsToIgnore 里面排除掉的异常类型）进行处理。若同时配置了 fallback 和 defaultFallback，则只有 fallback 会生效。defaultFallback 函数签名要求：


- 返回值类型必须与原函数返回值类型一致；

- 方法参数列表需要为空，或者可以额外多一个 Throwable 类型的参数用于接收对应的异常。

- defaultFallback 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 fallbackClass 为对应的类的 Class 对象，注意对应的函数必需为 static 函数，否则无法解析。

- exceptionsToIgnore（since 1.6.0）：用于指定哪些异常被排除掉，不会计入异常统计中，也不会进入 fallback 逻辑中，而是会原样抛出。

> 注：1.6.0 之前的版本 fallback 函数只针对降级异常（DegradeException）进行处理，不能针对业务异常进行处理。


特别地，若 blockHandler 和 fallback 都进行了配置，则被限流降级而抛出 BlockException 时只会进入 blockHandler 处理逻辑。若未配置 blockHandler、fallback 和 defaultFallback，则被限流降级时会将 BlockException 直接抛出。


## 3.3 测试

启动子工程web_mvc，启动成功后打开浏览器访问：[http://localhost:8000/hello](http://localhost:8000/hello) ，可以看到页面正常显示Hello Web MVC，多次刷新后打开Sentinel Dashboard，在Sentinel控制台上已经可以看到我们的web-mvc应用了，如图：

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第7篇：Sentinel：服务限流基础篇_image_2.png)

- 注意：请确保客户端有访问量，Sentinel 会在客户端首次调用的时候进行初始化，开始向控制台发送心跳包。

# 4. Spring WebFlux

Sentinel目前已经支持WebFlux，需要配合spring-boot-starter-webflux依赖触发 sentinel-starter中WebFlux相关的自动化配置。具体接入方式如下：

## 4.1 创建子工程web_flux

工程依赖pom.xml如下：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

spring-boot-starter-webflux引入WebFlux的相关依赖，Sentinel的依赖已在父工程引入。

## 4.2 配置文件application.yml如下

```
server:
  port: 9000
spring:
  application:
    name: web-flux
  cloud:
    sentinel:
      transport:
        dashboard: 192.168.44.129:8080
        port: 8720
```

## 4.3 测试接口HelloController.java

```
@RestController
public class HelloController {
    @GetMapping("/hello")
    @SentinelResource("hello")
    public Mono<String> mono() {
        return Mono.just("Hello Web Flux")
                .transform(new SentinelReactorTransformer<>("resourceName"));
    }
}
```

## 4.4 测试

启动子工程web_flux，打开浏览器访问：[http://localhost:9000/hello](http://localhost:9000/hello) ，页面正常显示Hello Web Flux，多次刷新页面后打开Sentinel控制台，可以看到web_flux工程正常注册，测试成功，如图：

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第7篇：Sentinel：服务限流基础篇_image_3.png)