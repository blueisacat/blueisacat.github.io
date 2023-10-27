---
layout: default
title: Spring Cloud Gateway+nacos灰度发布
parent: Experience
---

# 基础环境搭建

很久没有搭建 SpringCloud 项目了, 首先搭建一个基础示例, 供代码调试。

1. 首先, 快速搭建 nacos-server, 这里基于 官方文档 压缩包安装。安装完成后访问 

1. 项目创建。我们计划创建 2 个服务, 一个服务提供者 provider， 一个网关 gateway。项目采用聚合项目的形式。

## 根目录 pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="
         xmlns:xsi="
         xsi:schemaLocation="
    <modelVersion>4.0.0</modelVersion>
    <groupId>cn.idea360</groupId>
    <artifactId>spring-cloud-demo</artifactId>
    <version>0.0.1</version>
    <packaging>pom</packaging>
    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <spring-boot.version>2.3.7.RELEASE</spring-boot.version>
        <spring-cloud-alibaba.version>2.2.2.RELEASE</spring-cloud-alibaba.version>
        <spring-cloud.version>Hoxton.SR9</spring-cloud.version>
    </properties>
    <modules>
        <module>provider</module>
        <module>gateway</module>
    </modules>
    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
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
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
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
</project>
```

## 服务提供者 provider

这里我们计划引入 nacos, 所以先创建一个 nacos 配置文件 dataId 为 provider.properties, 这里用默认的命名空间 public, 默认分组 DEFAULT_GROUP

```
version=2
```

provider 的 pom 文件如下:

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="
         xsi:schemaLocation="
    <modelVersion>4.0.0</modelVersion>
    <artifactId>provider</artifactId>
    <parent>
        <groupId>cn.idea360</groupId>
        <artifactId>spring-cloud-demo</artifactId>
        <version>0.0.1</version>
    </parent>
    <name>provider</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>2.3.7.RELEASE</spring-boot.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>2.3.7.RELEASE</version>
                <configuration>
                    <mainClass>cn.idea360.provider.ProviderApplication</mainClass>
                </configuration>
                <executions>
                    <execution>
                        <id>repackage</id>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

application.properties 配置:

```
# 应用名称
spring.application.name=provider
# 应用服务 WEB 访问端口
server.port=9001
spring.cloud.nacos.config.server-addr=127.0.0.1:8848
spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848
```

在启动类, 我们增加了服务发现注解 @EnableDiscoveryClient

```
package cn.idea360.provider;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
@EnableDiscoveryClient
@SpringBootApplication
public class ProviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProviderApplication.class, args);
    }
}
```

controller 逻辑很简单, 只是简单返回版本号

```
package cn.idea360.provider.controller;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.core.env.Environment;
import org.springframework.web.bind.annotation.*;
/**
 * @author cuishiying
 * @date 2021-01-22
 */
@RefreshScope
@RestController
@RequestMapping("/test")
public class TestController {
    @Autowired
    private Environment env;
    @Value("${version:0}")
    private String version;
    /**
     * 
     * @return
     */
    @GetMapping("/port")
    public Object port() {
        return String.format("port=%s, version=%s", env.getProperty("local.server.port"), version);
    }
}
```

## 网关 gateway

gateway 服务的 pom 配置如下:

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="
         xsi:schemaLocation="
    <modelVersion>4.0.0</modelVersion>
    <artifactId>gateway</artifactId>
    <parent>
        <groupId>cn.idea360</groupId>
        <artifactId>spring-cloud-demo</artifactId>
        <version>0.0.1</version>
    </parent>
    <name>gateway</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>2.3.7.RELEASE</spring-boot.version>
        <spring-cloud-alibaba.version>2.2.2.RELEASE</spring-cloud-alibaba.version>
        <spring-cloud.version>Hoxton.SR9</spring-cloud.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
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
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>2.3.7.RELEASE</version>
                <configuration>
                    <mainClass>cn.idea360.gateway.GatewayApplication</mainClass>
                </configuration>
                <executions>
                    <execution>
                        <id>repackage</id>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

网关同样做服务发现

```
package cn.idea360.gateway;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
@EnableDiscoveryClient
@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```

网关核心配置, 这里我们先采用静态路由的方式测试下项目是否有问题

application.yml

```
# 应用服务 WEB 访问端口
server:
  port: 9000
# 应用名称
spring:
  application:
    name: gateway
  cloud:
    nacos:
      config:
        server-addr: 127.0.0.1:8848
      discovery:
        server-addr: 127.0.0.1:8848
    gateway:
      routes: # 
        - id: provider  # 路由 ID，保持唯一
          uri: lb://provider # uri指目标服务地址，lb代表从注册中心获取服务
          predicates:
            - Path=/provider/**  # 
          filters:
            - StripPrefix=1 # StripPrefix=1就代表截取路径的个数为1, 这样请求 
```

接下来就是激动人心的时刻了, 先测试下服务是否 ok.

```
➜  blog git:(master) ✗ curl 
port=9001, version=2%
➜  blog git:(master) ✗ curl 
port=9001, version=2%
```

通过结果可见, 网关的基本配置已经生效了。

## 基于 nacos 动态路由

动态路由的实现有 2 种方式, 一个就是像之前一样改写 RouteDefinitionRepository, 一个就是基于 nacos 的监听器给 RouteDefinitionRepository 动态更新值。实现逻辑大同小异。

### 基于 nacos 监听器实现动态路由

1. 配置 nacos 配置文件 gateway-router.json, 为了避免项目 application.yml 中配置文件影响, 先注释掉 gateway 相关的配置

```
[{
    "id": "provider",
    "predicates": [{
        "name": "Path",
        "args": {
            "_genkey_0": "/provider/**"
        }
    }],
    "filters": [{
        "name": "StripPrefix",
        "args": {
            "_genkey_0": "1"
        }
    }],
    "uri": "lb://provider",
    "order": 0
}]
```

1. 基于 nacos 监听器实现路由刷新

```
package cn.idea360.gateway.config;
import com.alibaba.nacos.api.NacosFactory;
import com.alibaba.nacos.api.config.ConfigService;
import com.alibaba.nacos.api.config.listener.Listener;
import com.alibaba.nacos.api.exception.NacosException;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.gateway.event.RefreshRoutesEvent;
import org.springframework.cloud.gateway.route.RouteDefinition;
import org.springframework.cloud.gateway.route.RouteDefinitionWriter;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.context.ApplicationEventPublisherAware;
import org.springframework.stereotype.Component;
import reactor.core.publisher.Mono;
import javax.annotation.PostConstruct;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.Executor;
/**
 * @author cuishiying
 * @date 2021-01-22
 */
@Component
public class DynamicGatewayRouteConfig implements ApplicationEventPublisherAware {
    private String dataId = "gateway-router.json";
    private String group = "DEFAULT_GROUP";
    @Value("${spring.cloud.nacos.config.server-addr}")
    private String serverAddr;
    @Autowired
    private RouteDefinitionWriter routeDefinitionWriter;
    private ApplicationEventPublisher applicationEventPublisher;
    private static final List<String> ROUTE_LIST = new ArrayList<String>();
    private ObjectMapper objectMapper = new ObjectMapper();
    @PostConstruct
    public void dynamicRouteByNacosListener() {
        try {
            ConfigService configService = NacosFactory.createConfigService(serverAddr);
            configService.getConfig(dataId, group, 5000);
            configService.addListener(dataId, group, new Listener() {
                public void receiveConfigInfo(String configInfo) {
                    clearRoute();
                    try {
                        List<RouteDefinition> gatewayRouteDefinitions = objectMapper.readValue(configInfo, new TypeReference<List<RouteDefinition>>() {});
                        for (RouteDefinition routeDefinition : gatewayRouteDefinitions) {
                            addRoute(routeDefinition);
                        }
                        publish();
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
                public Executor getExecutor() {
                    return null;
                }
            });
        } catch (NacosException e) {
            e.printStackTrace();
        }
    }
    private void clearRoute() {
        for (String id : ROUTE_LIST) {
            this.routeDefinitionWriter.delete(Mono.just(id)).subscribe();
        }
        ROUTE_LIST.clear();
    }
    private void addRoute(RouteDefinition definition) {
        try {
            routeDefinitionWriter.save(Mono.just(definition)).subscribe();
            ROUTE_LIST.add(definition.getId());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    private void publish() {
        this.applicationEventPublisher.publishEvent(new RefreshRoutesEvent(this.routeDefinitionWriter));
    }
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        this.applicationEventPublisher = applicationEventPublisher;
    }
}
```

1. 重启网关, 请求 

```
[
    {
        "predicate": "Paths: [/provider/**], match trailing slash: true",
        "route_id": "provider",
        "filters": [
            "[[StripPrefix parts = 1], order = 1]"
        ],
        "uri": "lb://provider",
        "order": 0
    }
]
```

路由测试结果如下:

```
➜  blog git:(master) ✗ curl 
port=9001, version=2%
```

### 基于 RouteDefinitionRepository 实现动态路由

Spring Cloud Gateway 中加载路由信息分别由以下几个类负责

1. PropertiesRouteDefinitionLocator：从配置文件中读取路由信息(如 YML、Properties 等)

1. RouteDefinitionRepository：从存储器中读取路由信息(如内存、配置中心、Redis、MySQL 等) 3、DiscoveryClientRouteDefinitionLocator：从注册中心中读取路由信息(如 Nacos、Eurka、Zookeeper 等)

```
package cn.idea360.gateway.router;
import com.alibaba.cloud.nacos.NacosConfigManager;
import com.alibaba.nacos.api.config.listener.Listener;
import com.alibaba.nacos.api.exception.NacosException;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.JsonMappingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.google.common.collect.Lists;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.gateway.event.RefreshRoutesEvent;
import org.springframework.cloud.gateway.route.RouteDefinition;
import org.springframework.cloud.gateway.route.RouteDefinitionRepository;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.context.ApplicationEventPublisherAware;
import org.springframework.stereotype.Component;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;
import javax.annotation.PostConstruct;
import java.util.List;
import java.util.concurrent.Executor;
/**
 * @author cuishiying
 * @date 2021-01-22
 */
@Component
public class NacosRouteDefinitionRepository implements RouteDefinitionRepository, ApplicationEventPublisherAware {
    private static final Logger log = LoggerFactory.getLogger(NacosRouteDefinitionRepository.class);
    @Autowired
    private NacosConfigManager nacosConfigManager;
    // 更新路由信息需要的
    private ApplicationEventPublisher applicationEventPublisher;
    private String dataId = "gateway-router.json";
    private String group = "DEFAULT_GROUP";
    @Value("${spring.cloud.nacos.config.server-addr}")
    private String serverAddr;
    private ObjectMapper objectMapper = new ObjectMapper();
    @PostConstruct
    public void dynamicRouteByNacosListener() {
        try {
            nacosConfigManager.getConfigService().addListener(dataId, group, new Listener() {
                public void receiveConfigInfo(String configInfo) {
                    log.info("自动更新配置...\r\n{}", configInfo);
                    applicationEventPublisher.publishEvent(new RefreshRoutesEvent(this));
                }
                public Executor getExecutor() {
                    return null;
                }
            });
        } catch (NacosException e) {
            e.printStackTrace();
        }
    }
    @Override
    public Flux<RouteDefinition> getRouteDefinitions() {
        try {
            String configInfo = nacosConfigManager.getConfigService().getConfig(dataId, group, 5000);
            List<RouteDefinition> gatewayRouteDefinitions = objectMapper.readValue(configInfo, new TypeReference<List<RouteDefinition>>() {
            });
            return Flux.fromIterable(gatewayRouteDefinitions);
        } catch (NacosException e) {
            e.printStackTrace();
        } catch (JsonMappingException e) {
            e.printStackTrace();
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
        return Flux.fromIterable(Lists.newArrayList());
    }
    @Override
    public Mono<Void> save(Mono<RouteDefinition> route) {
        return null;
    }
    @Override
    public Mono<Void> delete(Mono<String> routeId) {
        return null;
    }
    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        this.applicationEventPublisher = applicationEventPublisher;
    }
}
```

测试结果同上。

# 基于 SpringCloud Gateway + nacos 灰度路由

首先需要明白灰度的场景, 因为有不同版本的服务需要共存, 所以新的节点升级的时候必然代码及配置会存在差别, 所以我们根据这种差别来判断服务版本是新版本还是线上稳定版本。这里我们用 prod 和 gray 来标识 2 个版本。

实现的整体思路：

1. 编写带版本号的灰度路由(负载均衡策略)

1. 编写自定义 filter

1. nacos 服务配置需要灰度发布的服务的元数据信息以及权重(在服务 jar 中配置)

注意, 应该先修改 nacos 配置实现动态路由, 然后再升级灰度节点. 本案例只是简单示例灰度原理。


具体的实现步骤：


1. 首先排除掉默认的 ribbon

    ```
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    ```

1. 引入官方新的负载均衡包

    ```
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-loadbalancer</artifactId>
    </dependency>
    ```

1. 自定义负载均衡策略

    ```
    package cn.idea360.gateway.gray;
    import org.apache.commons.lang3.StringUtils;
    import org.springframework.beans.factory.ObjectProvider;
    import org.springframework.cloud.client.ServiceInstance;
    import org.springframework.cloud.client.loadbalancer.reactive.DefaultResponse;
    import org.springframework.cloud.client.loadbalancer.reactive.EmptyResponse;
    import org.springframework.cloud.client.loadbalancer.reactive.Request;
    import org.springframework.cloud.client.loadbalancer.reactive.Response;
    import org.springframework.cloud.loadbalancer.core.NoopServiceInstanceListSupplier;
    import org.springframework.cloud.loadbalancer.core.ReactorServiceInstanceLoadBalancer;
    import org.springframework.cloud.loadbalancer.core.ServiceInstanceListSupplier;
    import org.springframework.http.HttpHeaders;
    import reactor.core.publisher.Flux;
    import reactor.core.publisher.Mono;
    import java.util.List;
    import java.util.Random;
    import java.util.concurrent.atomic.AtomicInteger;
    import java.util.stream.Collectors;
    /**
    * @author cuishiying
    * @date 2021-01-22
    */
    public class VersionGrayLoadBalancer implements ReactorServiceInstanceLoadBalancer {
        private ObjectProvider<ServiceInstanceListSupplier> serviceInstanceListSupplierProvider;
        private String serviceId;
        private final AtomicInteger position;
        public VersionGrayLoadBalancer(ObjectProvider<ServiceInstanceListSupplier> serviceInstanceListSupplierProvider, String serviceId) {
            this(serviceInstanceListSupplierProvider, serviceId, new Random().nextInt(1000));
        }
        public VersionGrayLoadBalancer(ObjectProvider<ServiceInstanceListSupplier> serviceInstanceListSupplierProvider, String serviceId, int seedPosition) {
            this.serviceId = serviceId;
            this.serviceInstanceListSupplierProvider = serviceInstanceListSupplierProvider;
            this.position = new AtomicInteger(seedPosition);
        }
        @Override
        public Mono<Response<ServiceInstance>> choose(Request request) {
            HttpHeaders headers = (HttpHeaders) request.getContext();
            ServiceInstanceListSupplier supplier = this.serviceInstanceListSupplierProvider.getIfAvailable(NoopServiceInstanceListSupplier::new);
            return ((Flux) supplier.get()).next().map(list -> processInstanceResponse((List<ServiceInstance>) list, headers));
        }
        private Response<ServiceInstance> processInstanceResponse(List<ServiceInstance> instances, HttpHeaders headers) {
            if (instances.isEmpty()) {
                return new EmptyResponse();
            } else {
                String reqVersion = headers.getFirst("version");
                if (StringUtils.isEmpty(reqVersion)) {
                    return processRibbonInstanceResponse(instances);
                }
                List<ServiceInstance> serviceInstances = instances.stream()
                        .filter(instance -> reqVersion.equals(instance.getMetadata().get("version")))
                        .collect(Collectors.toList());
                if (serviceInstances.size() > 0) {
                    return processRibbonInstanceResponse(serviceInstances);
                } else {
                    return processRibbonInstanceResponse(instances);
                }
            }
        }
        /**
        * 负载均衡器
        * 参考 org.springframework.cloud.loadbalancer.core.RoundRobinLoadBalancer#getInstanceResponse
        *
        * @author javadaily
        */
        private Response<ServiceInstance> processRibbonInstanceResponse(List<ServiceInstance> instances) {
            int pos = Math.abs(this.position.incrementAndGet());
            ServiceInstance instance = instances.get(pos % instances.size());
            return new DefaultResponse(instance);
        }
    }
    ```

1. 自定义过滤器加载负载均衡策略

    ```
    package cn.idea360.gateway.gray;
    import org.apache.commons.logging.Log;
    import org.apache.commons.logging.LogFactory;
    import org.springframework.cloud.client.ServiceInstance;
    import org.springframework.cloud.client.loadbalancer.LoadBalancerUriTools;
    import org.springframework.cloud.client.loadbalancer.reactive.DefaultRequest;
    import org.springframework.cloud.client.loadbalancer.reactive.Request;
    import org.springframework.cloud.client.loadbalancer.reactive.Response;
    import org.springframework.cloud.gateway.config.LoadBalancerProperties;
    import org.springframework.cloud.gateway.filter.GatewayFilterChain;
    import org.springframework.cloud.gateway.filter.GlobalFilter;
    import org.springframework.cloud.gateway.filter.ReactiveLoadBalancerClientFilter;
    import org.springframework.cloud.gateway.support.DelegatingServiceInstance;
    import org.springframework.cloud.gateway.support.NotFoundException;
    import org.springframework.cloud.gateway.support.ServerWebExchangeUtils;
    import org.springframework.cloud.loadbalancer.core.ServiceInstanceListSupplier;
    import org.springframework.cloud.loadbalancer.support.LoadBalancerClientFactory;
    import org.springframework.core.Ordered;
    import org.springframework.http.HttpHeaders;
    import org.springframework.web.server.ServerWebExchange;
    import reactor.core.publisher.Mono;
    import java.net.URI;
    /**
    * @author cuishiying
    * @date 2021-01-22
    */
    public class GrayReactiveLoadBalancerClientFilter implements GlobalFilter, Ordered {
        private static final Log log = LogFactory.getLog(ReactiveLoadBalancerClientFilter.class);
        private static final int LOAD_BALANCER_CLIENT_FILTER_ORDER = 10150;
        private final LoadBalancerClientFactory clientFactory;
        private LoadBalancerProperties properties;
        public GrayReactiveLoadBalancerClientFilter(LoadBalancerClientFactory clientFactory, LoadBalancerProperties properties) {
            this.clientFactory = clientFactory;
            this.properties = properties;
        }
        @Override
        public int getOrder() {
            return LOAD_BALANCER_CLIENT_FILTER_ORDER;
        }
        @Override
        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
            URI url = exchange.getAttribute(ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR);
            String schemePrefix = exchange.getAttribute(ServerWebExchangeUtils.GATEWAY_SCHEME_PREFIX_ATTR);
            if (url != null && ("grayLb".equals(url.getScheme()) || "grayLb".equals(schemePrefix))) {
                ServerWebExchangeUtils.addOriginalRequestUrl(exchange, url);
                if (log.isTraceEnabled()) {
                    log.trace(ReactiveLoadBalancerClientFilter.class.getSimpleName() + " url before: " + url);
                }
                return this.choose(exchange).doOnNext((response) -> {
                    if (!response.hasServer()) {
                        throw NotFoundException.create(this.properties.isUse404(), "Unable to find instance for " + url.getHost());
                    } else {
                        URI uri = exchange.getRequest().getURI();
                        String overrideScheme = null;
                        if (schemePrefix != null) {
                            overrideScheme = url.getScheme();
                        }
                        DelegatingServiceInstance serviceInstance = new DelegatingServiceInstance((ServiceInstance)response.getServer(), overrideScheme);
                        URI requestUrl = this.reconstructURI(serviceInstance, uri);
                        if (log.isTraceEnabled()) {
                            log.trace("LoadBalancerClientFilter url chosen: " + requestUrl);
                        }
                        exchange.getAttributes().put(ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR, requestUrl);
                    }
                }).then(chain.filter(exchange));
            } else {
                return chain.filter(exchange);
            }
        }
        protected URI reconstructURI(ServiceInstance serviceInstance, URI original) {
            return LoadBalancerUriTools.reconstructURI(serviceInstance, original);
        }
        private Mono<Response<ServiceInstance>> choose(ServerWebExchange exchange) {
            URI uri = (URI)exchange.getAttribute(ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR);
            VersionGrayLoadBalancer loadBalancer = new VersionGrayLoadBalancer(clientFactory.getLazyProvider(uri.getHost(), ServiceInstanceListSupplier.class), uri.getHost());
            if (loadBalancer == null) {
                throw new NotFoundException("No loadbalancer available for " + uri.getHost());
            } else {
                return loadBalancer.choose(this.createRequest(exchange));
            }
        }
        private Request createRequest(ServerWebExchange exchange) {
            HttpHeaders headers = exchange.getRequest().getHeaders();
            Request<HttpHeaders> request = new DefaultRequest<>(headers);
            return request;
        }
    }
    ```

1. 注入过滤器

    ```
    package cn.idea360.gateway.gray;
    import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
    import org.springframework.cloud.gateway.config.LoadBalancerProperties;
    import org.springframework.cloud.loadbalancer.support.LoadBalancerClientFactory;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    /**
    * @author cuishiying
    * @date 2021-01-22
    */
    @Configuration
    public class GrayGatewayReactiveLoadBalancerClientAutoConfiguration {
        @Bean
        @ConditionalOnMissingBean({GrayReactiveLoadBalancerClientFilter.class})
        public GrayReactiveLoadBalancerClientFilter grayReactiveLoadBalancerClientFilter(LoadBalancerClientFactory clientFactory, LoadBalancerProperties properties) {
            return new GrayReactiveLoadBalancerClientFilter(clientFactory, properties);
        }
    }
    ```

1. 发布灰度服务

    ```
    # 应用名称
    spring.application.name=provider
    # 应用服务 WEB 访问端口
    server.port=9002
    spring.cloud.nacos.config.server-addr=127.0.0.1:8848
    spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848
    spring.cloud.nacos.discovery.metadata.version = gray
    ```

1. 测试

    ```
    curl -X GET -H "version:gray" -d '{"name": "admin"}' 
    ```

发现会永远路由到 9002