# 1. 背景介绍

在Spring Cloud微服务体系下，常用的服务网关有Netflix公司开源的Zuul，还有Spring Cloud团队自己开源的Spring Cloud Gateway，其中NetFlix公司开源的Zuul版本已经迭代至2.x，但是Spring Cloud并未集成，目前Spring Cloud集成的Spring Cloud Zuul还是Zuul1.x，这一版的Zuul是基于Servlet构建的，采用的方案是阻塞式的多线程方案，即一个线程处理一次连接请求，这种方式在内部延迟严重、设备故障较多情况下会引起存活的连接增多和线程增加的情况发生。Spring Cloud自己开源的Spring Cloud Gateway则是基于Spring Webflux来构建的，Spring Webflux有一个全新的非堵塞的函数式 Reactive Web 框架，可以用来构建异步的、非堵塞的、事件驱动的服务，在伸缩性方面表现非常好。使用非阻塞API， Websockets得到支持，并且由于它与Spring紧密集成，将会得到更好的开发体验。

本文将基于Gateway服务网关来介绍如何使用Nacos的配置功能来实现服务网关动态路由。

# 2. 实现方案

在开始之前我们先介绍一下具体实现方式：

1. 路由信息不再配置在配置文件中，将路由信息配置在Nacos的配置中。

1. 在服务网关Spring Cloud Gateway中开启监听，监听Nacos配置文件的修改。

1. Nacos配置文件一旦发生改变，则Spring Cloud Gateway重新刷新自己的路由信息。

# 3. 环境准备

首先，需要准备一个Nacos服务，我这里的版本是使用的Nacos v1.1.3，如果不会配置Nacos服务的同学，请参考之前的文章《Nacos服务中心初探》

# 4. 工程实战

创建工程gateway-nacos-config，工程依赖pom.xml如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="
         xsi:schemaLocation="
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.7.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.springcloud.alibaba</groupId>
    <artifactId>gateway-nacos-config</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>gateway-nacos-config</name>
    <description>gateway-nacos-config</description>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR2</spring-cloud.version>
        <spring-cloud-alibaba.version>2.1.0.RELEASE</spring-cloud-alibaba.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webflux</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
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
        <dependency>
            <groupId>io.projectreactor</groupId>
            <artifactId>reactor-test</artifactId>
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
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring-cloud-alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
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

- 在使用Spring Cloud Alibaba组件的时候，在<dependencyManagement>中需配置spring-cloud-alibaba-dependencies，它管理了Spring Cloud Alibaba组件的版本依赖。

配置文件application.yml如下：

```
server:
  port: 8080
spring:
  application:
    name: spring-cloud-gateway-server
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.44.129:8848
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

- spring.cloud.nacos.discovery.server-addr：配置为Nacos服务地址，格式为ip:port

接下来进入核心部分，配置Spring Cloud Gateway动态路由，这里需要实现一个Spring提供的事件推送接口ApplicationEventPublisherAware，代码如下：

```
@Component
public class DynamicRoutingConfig implements ApplicationEventPublisherAware {
    private final Logger logger = LoggerFactory.getLogger(DynamicRoutingConfig.class);
    private static final String DATA_ID = "zuul-refresh-dev.json";
    private static final String Group = "DEFAULT_GROUP";
    @Autowired
    private RouteDefinitionWriter routeDefinitionWriter;
    private ApplicationEventPublisher applicationEventPublisher;
    @Bean
    public void refreshRouting() throws NacosException {
        Properties properties = new Properties();
        properties.put(PropertyKeyConst.SERVER_ADDR, "192.168.44.129:8848");
        properties.put(PropertyKeyConst.NAMESPACE, "8282c713-da90-486a-8438-2a5a212ef44f");
        ConfigService configService = NacosFactory.createConfigService(properties);
        configService.addListener(DATA_ID, Group, new Listener() {
            @Override
            public Executor getExecutor() {
                return null;
            }
            @Override
            public void receiveConfigInfo(String configInfo) {
                logger.info(configInfo);
                boolean refreshGatewayRoute = JSONObject.parseObject(configInfo).getBoolean("refreshGatewayRoute");
                if (refreshGatewayRoute) {
                    List<RouteEntity> list = JSON.parseArray(JSONObject.parseObject(configInfo).getString("routeList")).toJavaList(RouteEntity.class);
                    for (RouteEntity route : list) {
                        update(assembleRouteDefinition(route));
                    }
                } else {
                    logger.info("路由未发生变更");
                }
            }
        });
    }
    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        this.applicationEventPublisher = applicationEventPublisher;
    }
    /**
     * 路由更新
     * @param routeDefinition
     * @return
     */
    public void update(RouteDefinition routeDefinition){
        try {
            this.routeDefinitionWriter.delete(Mono.just(routeDefinition.getId()));
            logger.info("路由更新成功");
        }catch (Exception e){
            logger.error(e.getMessage(), e);
        }
        try {
            routeDefinitionWriter.save(Mono.just(routeDefinition)).subscribe();
            this.applicationEventPublisher.publishEvent(new RefreshRoutesEvent(this));
            logger.info("路由更新成功");
        }catch (Exception e){
            logger.error(e.getMessage(), e);
        }
    }
    public RouteDefinition assembleRouteDefinition(RouteEntity routeEntity) {
        RouteDefinition definition = new RouteDefinition();
        // ID
        definition.setId(routeEntity.getId());
        // Predicates
        List<PredicateDefinition> pdList = new ArrayList<>();
        for (PredicateEntity predicateEntity: routeEntity.getPredicates()) {
            PredicateDefinition predicateDefinition = new PredicateDefinition();
            predicateDefinition.setArgs(predicateEntity.getArgs());
            predicateDefinition.setName(predicateEntity.getName());
            pdList.add(predicateDefinition);
        }
        definition.setPredicates(pdList);
        // Filters
        List<FilterDefinition> fdList = new ArrayList<>();
        for (FilterEntity filterEntity: routeEntity.getFilters()) {
            FilterDefinition filterDefinition = new FilterDefinition();
            filterDefinition.setArgs(filterEntity.getArgs());
            filterDefinition.setName(filterEntity.getName());
            fdList.add(filterDefinition);
        }
        definition.setFilters(fdList);
        // URI
        URI uri = UriComponentsBuilder.fromUriString(routeEntity.getUri()).build().toUri();
        definition.setUri(uri);
        return definition;
    }
}
```

这里主要介绍一下refreshRouting()这个方法，这个方法主要负责监听Nacos的配置变化，这里先使用参数构建一个ConfigService，再使用ConfigService开启一个监听，并且在监听的方法中刷新路由信息。

Nacos配置如图：

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第9篇：Gateway基于Nacos动态网关路由_image_0.png)

```
{
    "refreshGatewayRoute":false,
    "routeList":[
        {
            "id":"github_route",
            "predicates":[
                {
                    "name":"Path",
                    "args":{
                        "_genkey_0":"/meteor1993"
                    }
                }
            ],
            "filters":[
            ],
            "uri":"
            "order":0
        }
    ]
}
```

配置格式选择JSON，Data ID和Group与程序中的配置保持一致，注意，我这里的程序配置了namespace，如果使用默认namespace，可以不用配置。

这里配置了一个路由/meteor1993，直接访问这个路由会访问到作者的Github仓库。

剩余部分的代码这里就不一一展示了，已经上传至代码仓库，有需要的同学可以自行取用。

# 5. 测试

启动工程，这时是没有任何路由信息的，打开浏览器访问：[http://localhost:8080/meteor1993](http://localhost:8080/meteor1993) ，页面返回404报错信息，如图：

同时，也可以访问链接：[http://localhost:8080/actuator/gateway/routes](http://localhost:8080/actuator/gateway/routes) ，可以看到如下打印：

```
[]
```

打开在Nacos Server端的UI界面，选择监听查询，选择namespace为springclouddev的栏目，输入DATA_ID为zuul-refresh-dev.json和Group为DEFAULT_GROUP，点击查询，可以看到我们启动的工程gateway-nacos-config正在监听Nacos Server端，如图：

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第9篇：Gateway基于Nacos动态网关路由_image_1.png)

笔者这里的本地ip为：192.168.44.1。监听正常，这时，我们修改刚才创建的配置，将里面的refreshGatewayRoute修改为true，如下：

```
{"refreshGatewayRoute": true, "routeList":[{"id":"github_route","predicates":[{"name":"Path","args":{"_genkey_0":"/meteor1993"}}],"filters":[],"uri":"
```

点击发布，可以看到工程gateway-nacos-config的控制台打印日志如下：

```
2019-09-02 22:09:49.254  INFO 8056 --- [38-2a5a212ef44f] c.s.a.g.config.DynamicRoutingConfig      : {
    "refreshGatewayRoute":true,
    "routeList":[
        {
            "id":"github_route",
            "predicates":[
                {
                    "name":"Path",
                    "args":{
                        "_genkey_0":"/meteor1993"
                    }
                }
            ],
            "filters":[
            ],
            "uri":"
            "order":0
        }
    ]
}
2019-09-02 22:09:49.268  INFO 8056 --- [38-2a5a212ef44f] c.s.a.g.config.DynamicRoutingConfig      : 路由更新成功
```

这时，我们的工程gateway-nacos-config的路由已经更新成功，访问路径：[http://localhost:8080/actuator/gateway/routes](http://localhost:8080/actuator/gateway/routes) ，可以看到如下打印：

```
[{"route_id":"github_route","route_definition":{"id":"github_route","predicates":[{"name":"Path","args":{"_genkey_0":"/meteor1993"}}],"filters":[],"uri":"
```

我们再次在浏览器中访问链接：[http://localhost:8080/meteor1993](http://localhost:8080/meteor1993) ，可以看到页面正常路由到Github仓库，如图：

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第9篇：Gateway基于Nacos动态网关路由_image_2.png)

# 6. 总结

至此，Nacos动态网关路由就介绍完了，主要运用了服务网关端监听Nacos配置改变的功能，实现服务网关路由配置动态刷新，同理，我们也可以使用服务网关Zuul来实现基于Nacos的动态路由功能。

基于这个思路，我们可以使用配置中心来实现网关的动态路由，而不是使用服务网关本身自带的配置文件，这样每次路由信息变更，无需修改配置文件而后重启服务。

目前市面上使用比较多的配置中心有携程开源的Apollo，服务网关还有Spring Cloud Zuul，下一篇文章我们介绍如何使用Apollo来实现Spring Cloud Zuul的动态路由。