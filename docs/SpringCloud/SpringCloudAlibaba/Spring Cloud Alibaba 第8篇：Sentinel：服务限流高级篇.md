---
layout: default
title: Spring Cloud Alibaba 第8篇：Sentinel：服务限流高级篇
parent: SpringCloudAlibaba系列教程
grand_parent: SpringCloud
nav_order: 1.8
---

# 1. Sentinel整合Feign和RestTemplate

Sentinel目前已经同时支持Feign和RestTemplate，需要我们引入对应的依赖，在使用Feign的时候需要在配置文件中打开Sentinel对Feign的支持：feign.sentinel.enabled=true，同时需要加入openfeign starter依赖使sentinel starter中的自动化配置类生效。在使用RestTemplate的时候需要在构造RestTemplate的Bean的时候加上@SentinelRestTemplate注解，开启Sentinel对RestTemplate的支持。

## 1.1 创建父工程sentinel-springcloud-high

父工程pom.xml如下：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

公共组件中引入Sentinel做流量控制，引入Nacos做服务中心。

## 1.2 创建子工程provider_server

配置文件application.yml如下：

```
server:
  port: 8000
spring:
  application:
    name: spring-cloud-provider-server
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.44.129:8848
    sentinel:
      transport:
        dashboard: localhost:8080
        port: 8720
management:
  endpoints:
    web:
      cors:
        allowed-methods: '*'
```

接口测试类HelloController.java如下：

```
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello(HttpServletRequest request) {
        return "Hello, port is: " + request.getLocalPort();
    }
}
```

## 1.3 创建子工程consumer_server

子工程依赖pom.xml如下：

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

配置文件application.yml如下：

```
server:
  port: 9000
spring:
  application:
    name: spring-cloud-consumer-server
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.44.129:8848
    sentinel:
      transport:
        dashboard: localhost:8080
        port: 8719
management:
  endpoints:
    web:
      cors:
        allowed-methods: '*'
feign:
  sentinel:
    enabled: true
```

这里使用feign.sentinel.enabled=true开启Sentinel对Feign的支持。

接口测试类HelloController.java

```
@RestController
public class HelloController {
    @Autowired
    HelloRemote helloRemote;
    @Autowired
    RestTemplate restTemplate;
    @GetMapping("/helloByFeign")
    public String helloByFeign() {
        return helloRemote.hello();
    }
    @GetMapping("/helloByRestTemplate")
    public String helloByRestTemplate() {
        return restTemplate.getForObject("
    }
}
```

Sentinel已经对做了整合，我们使用Feign的地方无需额外的注解。同时，@FeignClient注解中的所有属性，Sentinel都做了兼容。

启动主类Ch122ConsumerServerApplication.java如下：

```
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class Ch122ConsumerServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(Ch122ConsumerServerApplication.class, args);
    }
    @Bean
    @LoadBalanced
    @SentinelRestTemplate
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

在使用RestTemplate的时候需要增加@SentinelRestTemplate来开启Sentinel对RestTemplate的支持。

## 1.4 测试

启动工程provider_server和consumer_server，provider_server修改启动配置，启动两个实例，打开浏览器访问：[http://localhost:9000/helloByFeign](http://localhost:9000/helloByFeign) 和 [http://localhost:9000/helloByRestTemplate](http://localhost:9000/helloByRestTemplate) ，刷新几次，可以看到页面交替显示Hello, port is: 8000和Hello, port is: 8001，说明目前负载均衡正常，现在查看Sentinel控制台，如图：

![](../../../assets/images/SpringCloud/SpringCloudAlibaba/attachments/Spring%20Cloud%20Alibaba%20第8篇：Sentinel：服务限流高级篇_image_0.png)

## 1.5 流量控制测试

这时选择左侧的簇点流控，点击流控，如图：

![](../../../assets/images/SpringCloud/SpringCloudAlibaba/attachments/Spring%20Cloud%20Alibaba%20第8篇：Sentinel：服务限流高级篇_image_1.png)

这里我们配置一个最简单的规则，配置QPS限制为1，点击新增，如图：

![](../../../assets/images/SpringCloud/SpringCloudAlibaba/attachments/Spring%20Cloud%20Alibaba%20第8篇：Sentinel：服务限流高级篇_image_2.png)

这里解释一下什么是QPS，简单来说QPS是一个每秒访问数，这里我们测试时需要重复快速刷新[http://localhost:9000/helloByFeign](http://localhost:9000/helloByFeign) 和 [http://localhost:9000/helloByRestTemplate](http://localhost:9000/helloByRestTemplate) ，在刷新的过程中，我们可以看到页面会显示错误信息，如：Blocked by Sentinel (flow limiting)，说明我们配置Sentinel已经限流成功，这时我们再看一下Sentinel的控制台，可以看到我们刚才访问的成功和限流的数量，如图：

![](../../../assets/images/SpringCloud/SpringCloudAlibaba/attachments/Spring%20Cloud%20Alibaba%20第8篇：Sentinel：服务限流高级篇_image_3.png)

# 2. 服务降级

在上一小结，我们介绍了Feign和RestTemplate整合Sentinel使用，并且在Sentinel控制台上做了QPS限流，并且限流成功，限流成功后，默认情况下，Sentinel对控制资源的限流处理是直接抛出异常。在没有合理的业务承接或者前端对接情况下可以这样，但是正常情况为了更好的用户业务，都会实现一些被限流之后的特殊处理，我们不希望展示一个生硬的报错。这一小节，我们介绍一下服务降级处理。

## 2.1 创建子工程consumer_fallback

Feign服务降级类HelloRemoteFallBack.java如下：

```
@Component
public class HelloRemoteFallBack implements HelloRemote {
    @Override
    public String hello() {
        return "Feign FallBack Msg";
    }
}
```

相对应的，这里需要在HelloRemote.java上做一部分配置，使得限流后，触发服务降级执行我们的服务降级类，代码如下：

```
@FeignClient(name = "spring-cloud-provider-server", fallback = HelloRemoteFallBack.class)
public interface HelloRemote {
    @GetMapping("/hello")
    String hello();
}
```

fallback = HelloRemoteFallBack.class指定服务降级的处理类为HelloRemoteFallBack.class。

RestTemplate服务降级工具类ExceptionUtil.java如下：

```
public class ExceptionUtil {
    private final static Logger logger = LoggerFactory.getLogger(ExceptionUtil.class);
    public static SentinelClientHttpResponse handleException(HttpRequest request, byte[] body, ClientHttpRequestExecution execution, BlockException ex) {
        logger.error(ex.getMessage(), ex);
        return new SentinelClientHttpResponse("RestTemplate FallBack Msg");
    }
}
```

这里同样需要修改RestTemplate注册成为Bean的地方，使得RestTemplate触发服务降级以后代码执行我们为它写的处理类，Ch122ConsumerFallbackApplication.java代码如下：

```
@Bean
@LoadBalanced
@SentinelRestTemplate(blockHandler = "handleException", blockHandlerClass = ExceptionUtil.class)
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

这里需要注意，@SentinelRestTemplate注解的属性支持限流(blockHandler, blockHandlerClass)和降级(fallback, fallbackClass)的处理。

其中blockHandler或fallback属性对应的方法必须是对应blockHandlerClass或fallbackClass属性中的静态方法。

@SentinelRestTemplate注解的限流(blockHandler, blockHandlerClass)和降级(fallback, fallbackClass)属性不强制填写。

当使用RestTemplate调用被Sentinel熔断后，会返回RestTemplate request block by sentinel信息，或者也可以编写对应的方法自行处理返回信息。这里提供了 SentinelClientHttpResponse用于构造返回信息。

## 2.2 测试

顺次启动provider_server和consumer_fallback两个子工程。先在浏览器中交替访问[http://localhost:9090/helloByFeign](http://localhost:9090/helloByFeign) 和 [http://localhost:9090/helloByRestTemplate](http://localhost:9090/helloByRestTemplate) ，而后打开Sentinel控制台，在这两个接口上增加限流信息，注意，这里要将限流信息加在资源上，具体如图：

![](../../../assets/images/SpringCloud/SpringCloudAlibaba/attachments/Spring%20Cloud%20Alibaba%20第8篇：Sentinel：服务限流高级篇_image_4.png)

在浏览器中刷新两个链接，两个限流信息都可以正常浏览器中显示，测试成功，再次查看Sentinel控制台，也可以看到被拒接的流量统计，如图：

![](../../../assets/images/SpringCloud/SpringCloudAlibaba/attachments/Spring%20Cloud%20Alibaba%20第8篇：Sentinel：服务限流高级篇_image_5.png)

# 3. Sentinel整合服务网关限流

Sentinel目前支持Spring Cloud Gateway、Zuul 等主流的 API Gateway 进行限流。看一下官方的结构图，如图：

![](../../../assets/images/SpringCloud/SpringCloudAlibaba/attachments/Spring%20Cloud%20Alibaba%20第8篇：Sentinel：服务限流高级篇_image_6.png)

从这张官方的图中，可以看到，Sentinel对Zuul的限流主要是通过3个Filter来完成的，对Spring Cloud Gateway则是通过一个SentinleGatewayFilter和一个BlockRequestHandler来完成的。

Sentinel 1.6.0 引入了 Sentinel API Gateway Adapter Common 模块，此模块中包含网关限流的规则和自定义 API 的实体和管理逻辑：

- GatewayFlowRule：网关限流规则，针对 API Gateway 的场景定制的限流规则，可以针对不同 route 或自定义的 API 分组进行限流，支持针对请求中的参数、Header、来源 IP 等进行定制化的限流。

- ApiDefinition：用户自定义的 API 定义分组，可以看做是一些 URL 匹配的组合。比如我们可以定义一个 API 叫 my_api，请求 path 模式为 /foo/ 和 /baz/ 的都归到 my_api 这个 API 分组下面。限流的时候可以针对这个自定义的 API 分组维度进行限流。

## 3.1 Zuul 1.x

Sentinel 提供了 Zuul 1.x 的适配模块，可以为 Zuul Gateway 提供两种资源维度的限流：

- route 维度：即在 Spring 配置文件中配置的路由条目，资源名为对应的 route ID（对应 RequestContext 中的 proxy 字段）

- 自定义 API 维度：用户可以利用 Sentinel 提供的 API 来自定义一些 API 分组

### 3.1.1 创建子工程zuul_server

工程依赖pom.xml如下：

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-zuul-adapter</artifactId>
</dependency>
```

这里因为sentinel-zuul-adapter未包含在spring-cloud-starter-alibaba-sentinel，需要手动单独引入。

### 3.1.2 配置文件application.yml如下

```
server:
  port: 18080
spring:
  application:
    name: spring-cloud-zuul-server
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.44.129:8848
    sentinel:
      transport:
        dashboard: localhost:8080
        port: 8720
zuul:
  routes:
    consumer-route:
      path: /consumer/**
      serviceId: spring-cloud-consumer-fallback
```

### 3.1.3 定义降级处理类ZuulFallbackProvider.java如下

```
public class ZuulFallbackProvider implements ZuulBlockFallbackProvider {
    @Override
    public String getRoute() {
        return "*";
    }
    @Override
    public BlockResponse fallbackResponse(String route, Throwable cause) {
        RecordLog.info(String.format("[Sentinel DefaultBlockFallbackProvider] Run fallback route: %s", route));
        if (cause instanceof BlockException) {
            return new BlockResponse(429, "Sentinel block exception", route);
        } else {
            return new BlockResponse(500, "System Error", route);
        }
    }
}
```

### 3.1.4 同时，我们需要将3个Sentinel的Filter注入Spring，配置类如下

```
@Configuration
public class ZuulConfig {
    @Bean
    public ZuulFilter sentinelZuulPreFilter() {
        // We can also provider the filter order in the constructor.
        return new SentinelZuulPreFilter();
    }
    @Bean
    public ZuulFilter sentinelZuulPostFilter() {
        return new SentinelZuulPostFilter();
    }
    @Bean
    public ZuulFilter sentinelZuulErrorFilter() {
        return new SentinelZuulErrorFilter();
    }
    /**
     * 注册 ZuulFallbackProvider
     */
    @PostConstruct
    public void doInit() {
        ZuulBlockFallbackManager.registerProvider(new ZuulFallbackProvider());
    }
}
```

最终，启动前需要配置JVM启动参数，增加-Dcsp.sentinel.app.type=1，来告诉Sentinel控制台我们启动的服务是为 API Gateway 类型。

### 3.1.5 测试

顺次启动子工程provider_server、consumer_fallback、zuul_server，打开浏览器访问：[http://localhost:18080/consumer/helloByFeign](http://localhost:18080/consumer/helloByFeign) ，然后我们打开Sentinel控制台，查看zuul_server服务，如图：

![](../../../assets/images/SpringCloud/SpringCloudAlibaba/attachments/Spring%20Cloud%20Alibaba%20第8篇：Sentinel：服务限流高级篇_image_7.png)

我们定制限流策略，依旧是QPS为1，我们再次刷新[http://localhost:18080/consumer/helloByFeign](http://localhost:18080/consumer/helloByFeign) 页面，这时，页面上已经可以正产限流了，限流后显示的内容为：

```
{"code":429, "message":"Sentinel block exception", "route":"consumer-route"}
```

这里注意，定义限流的是资源，千万不要定义错地方，限流定义如图：

![](../../../assets/images/SpringCloud/SpringCloudAlibaba/attachments/Spring%20Cloud%20Alibaba%20第8篇：Sentinel：服务限流高级篇_image_8.png)

## 3.2 Spring Cloud Gateway

从 1.6.0 版本开始，Sentinel 提供了 Spring Cloud Gateway 的适配模块，可以提供两种资源维度的限流：

- route 维度：即在 Spring 配置文件中配置的路由条目，资源名为对应的 routeId

- 自定义 API 维度：用户可以利用 Sentinel 提供的 API 来自定义一些 API 分组

### 3.2.1 创建子工程gateway_server

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-spring-cloud-gateway-adapter</artifactId>
</dependency>
```

### 3.2.2 配置文件application.yml如下

```
server:
  port: 28080
spring:
  application:
    name: spring-cloud-gateway-server
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.44.129:8848
    sentinel:
      transport:
        dashboard: localhost:8080
        port: 8720
    gateway:
      enabled: true
      discovery:
        locator:
          lower-case-service-id: true
      routes:
        - id: consumer_server
          uri: lb://spring-cloud-consumer-fallback
          predicates:
            - Method=GET
```

### 3.2.3 全局配置类GatewayConfig.java如下

同上一小节介绍的Zuul，这里我们同样需要将两个Sentinel有关Spring Cloud Gateway的Filter注入Spring：SentinelGatewayFilter 和SentinelGatewayBlockExceptionHandler，这里因为在Sentinel v1.6.0版本才加入Spring Cloud Gateway的支持，很多地方还不是很完善，异常处理SentinelGatewayBlockExceptionHandler目前只能返回一个异常信息，在我们的系统中无法和上下游很好的结合，这里笔者自己重新实现了SentinelGatewayBlockExceptionHandler，并命名为JsonSentinelGatewayBlockExceptionHandler，返回参数定义成为JSON，这里不再注入Sentinel提供的SentinelGatewayBlockExceptionHandler，而是改为笔者自己实现的JsonSentinelGatewayBlockExceptionHandler。

```
@Configuration
public class GatewayConfig {
    private final List<ViewResolver> viewResolvers;
    private final ServerCodecConfigurer serverCodecConfigurer;
    public GatewayConfig(ObjectProvider<List<ViewResolver>> viewResolversProvider, ServerCodecConfigurer serverCodecConfigurer) {
        this.viewResolvers = viewResolversProvider.getIfAvailable(Collections::emptyList);
        this.serverCodecConfigurer = serverCodecConfigurer;
    }
    @Bean
    @Order(Ordered.HIGHEST_PRECEDENCE)
    public JsonSentinelGatewayBlockExceptionHandler jsonSentinelGatewayBlockExceptionHandler() {
        // Register the block exception handler for Spring Cloud Gateway.
        return new JsonSentinelGatewayBlockExceptionHandler(viewResolvers, serverCodecConfigurer);
    }
    @Bean
    @Order(-1)
    public GlobalFilter sentinelGatewayFilter() {
        return new SentinelGatewayFilter();
    }
}
```

### 3.2.4 降级处理类JsonSentinelGatewayBlockExceptionHandler.java如下

```
public class JsonSentinelGatewayBlockExceptionHandler implements WebExceptionHandler {
    private List<ViewResolver> viewResolvers;
    private List<HttpMessageWriter<?>> messageWriters;
    public JsonSentinelGatewayBlockExceptionHandler(List<ViewResolver> viewResolvers, ServerCodecConfigurer serverCodecConfigurer) {
        this.viewResolvers = viewResolvers;
        this.messageWriters = serverCodecConfigurer.getWriters();
    }
    private Mono<Void> writeResponse(ServerResponse response, ServerWebExchange exchange) {
        ServerHttpResponse serverHttpResponse = exchange.getResponse();
        serverHttpResponse.getHeaders().add("Content-Type", "application/json;charset=UTF-8");
        byte[] datas = "{\"code\":403,\"msg\":\"Sentinel block exception\"}".getBytes(StandardCharsets.UTF_8);
        DataBuffer buffer = serverHttpResponse.bufferFactory().wrap(datas);
        return serverHttpResponse.writeWith(Mono.just(buffer));
    }
    @Override
    public Mono<Void> handle(ServerWebExchange exchange, Throwable ex) {
        if (exchange.getResponse().isCommitted()) {
            return Mono.error(ex);
        }
        // This exception handler only handles rejection by Sentinel.
        if (!BlockException.isBlockException(ex)) {
            return Mono.error(ex);
        }
        return handleBlockedRequest(exchange, ex)
                .flatMap(response -> writeResponse(response, exchange));
    }
    private Mono<ServerResponse> handleBlockedRequest(ServerWebExchange exchange, Throwable throwable) {
        return GatewayCallbackManager.getBlockHandler().handleRequest(exchange, throwable);
    }
    private final Supplier<ServerResponse.Context> contextSupplier = () -> new ServerResponse.Context() {
        @Override
        public List<HttpMessageWriter<?>> messageWriters() {
            return JsonSentinelGatewayBlockExceptionHandler.this.messageWriters;
        }
        @Override
        public List<ViewResolver> viewResolvers() {
            return JsonSentinelGatewayBlockExceptionHandler.this.viewResolvers;
        }
    };
}
```

笔者这里仅重写了writeResponse()方法，讲返回信息简单的更改成了json格式，各位读者有需要可以根据自己的需求进行修改。

### 3.2.5 测试

顺次启动provider_server、consumer_server和gateway_server，配置gateway_server jvm启动参数-Dcsp.sentinel.app.type=1，如图：

![](../../../assets/images/SpringCloud/SpringCloudAlibaba/attachments/Spring%20Cloud%20Alibaba%20第8篇：Sentinel：服务限流高级篇_image_9.png)

打开浏览器访问：[http://localhost:28080/helloByFeign](http://localhost:28080/helloByFeign) ，刷新几次，页面正常返回Hello, port is: 8000，打开Sentinel控制台，配置限流策略，QPS限制为1，再刷新浏览器页面，这时，我们可以看到浏览器返回限流信息：

```
{"code":403,"msg":"Sentinel block exception"}
```

测试成功。