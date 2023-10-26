> Springboot: 2.1.6.RELEASE
> SpringCloud: Greenwich.SR1
> 如无特殊说明，本系列教程全采用以上版本


上一篇文章我们介绍了Gateway基于Nacos动态网关路由的解决方案《Spring Cloud Alibaba | Gateway基于Nacos动态网关路由》，同为Spring Cloud服务网关组件的Spring Cloud Zuul在生产环境中使用更为广泛，那么它有没有方便的动态路由解决方案呢？答案当然是肯定的，Zuul作为一个老牌的开源服务网关组件，动态路由对它来讲是一个十分必要的功能，毕竟我们不能随便重启服务网关，服务网关是一个微服务系统的大门，今天我们介绍的Zuul动态路由的解决方案来自于携程开源的配置中心Apollo。


# 1.Apollo概述

Apollo（阿波罗）是携程框架部门研发的开源配置管理中心，能够集中化管理应用不同环境、不同集群的配置，配置修改后能够实时推送到应用端，并且具备规范的权限、流程治理等特性。

Apollo支持4个维度管理Key-Value格式的配置：

1. application (应用)

1. environment (环境)

1. cluster (集群)

1. namespace (命名空间)

# 2.Apollo相比于Spring Cloud Config优势

前面的文章我们也介绍了Spring Cloud Config《跟我学SpringCloud | 第七篇：Spring Cloud Config 配置中心高可用和refresh》，但是它和我们今天要使用的相比，又有什么劣势呢？

Spring Cloud Config的精妙之处在于它的配置存储于Git，这就天然的把配置的修改、权限、版本等问题隔离在外。通过这个设计使得Spring Cloud Config整体很简单，不过也带来了一些不便之处。

| 功能点 | Apollo | Spring Cloud Config | 备注 | 
| -- | -- | -- | -- |
| 配置界面 | 一个界面管理不同环境、不同集群配置 | 无，需要通过git操作 |   | 
| 配置生效时间 | 实时 | 重启生效，或手动refresh生效 | Spring Cloud Config需要通过Git webhook，加上额外的消息队列才能支持实时生效 | 
| 版本管理 | 界面上直接提供发布历史和回滚按钮 | 无，需要通过git操作 |   | 
| 灰度发布 | 支持 | 不支持 |   | 
| 授权、审核、审计 | 界面上直接支持，而且支持修改、发布权限分离 | 需要通过git仓库设置，且不支持修改、发布权限分离 |   | 
| 实例配置监控 | 可以方便的看到当前哪些客户端在使用哪些配置 | 不支持 |   | 
| 配置获取性能 | 快，通过数据库访问，还有缓存支持 | 较慢，需要从git clone repository，然后从文件系统读取 |   | 
| 客户端支持 | 原生支持所有Java和.Net应用，提供API支持其它语言应用，同时也支持Spring annotation获取配置 | 支持Spring应用，提供annotation获取配置 | Apollo的适用范围更广一些 | 


# 3. 工程实战

这里需要准备一个Apollo配置中心，具体如何构建Apollo配置中心我这里不多做介绍，大家可以参考Apollo的官方文档：[https://github.com/ctripcorp/apollo/wiki](https://github.com/ctripcorp/apollo/wiki)

## 3.1 工程依赖pom.xml如下：

```
<dependency>
    <groupId>com.ctrip.framework.apollo</groupId>
    <artifactId>apollo-client</artifactId>
    <version>${apollo-client.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
```

## 3.2 配置文件

app.properties如下：

```
app.id=123456789
```

这里配置的app.id是在Apollo中创建项目时配置的。

application.yml如下：

```
apollo:
  bootstrap:
    enabled: true
    namespaces: zuul-config-apollo
  Meta: 
```

在Apollo上新建一个命名空间zuul-config-apollo。

其余的配置都配置在Apollo中，具体如图：

![](../../../assets/images/SpringCloud/Spring Cloud/attachments/SpringCloud系列教程%20第17篇：服务网关Zuul基于Apollo动态路由_image_0.png)

## 3.3 启动主类Chapter16Application.java如下：

```
@SpringBootApplication
@EnableEurekaClient
@EnableZuulProxy
@EnableApolloConfig
public class Chapter16Application {
    public static void main(String[] args) {
        SpringApplication.run(Chapter16Application.class, args);
    }
}
```

其中@EnableZuulProxy表示开启Zuul网关代理，@EnableApolloConfig表示开启Apollo配置。

## 3.4 路由刷新

```
@Component
public class ZuulProxyRefresher implements ApplicationContextAware {
    private ApplicationContext applicationContext;
    @Autowired
    private RouteLocator routeLocator;
    @ApolloConfigChangeListener(value = "zuul-config-apollo")
    public void onChange(ConfigChangeEvent changeEvent) {
        boolean zuulProxyChanged = false;
        for (String changedKey : changeEvent.changedKeys()) {
            if (changedKey.startsWith("zuul.")) {
                zuulProxyChanged = true;
                break;
            }
        }
        if (zuulProxyChanged) {
            refreshZuulProxy(changeEvent);
        }
    }
    private void refreshZuulProxy(ConfigChangeEvent changeEvent) {
        this.applicationContext.publishEvent(new EnvironmentChangeEvent(changeEvent.changedKeys()));
        this.applicationContext.publishEvent(new RoutesRefreshedEvent(routeLocator));
    }
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

@ApolloConfigChangeListener(value = "zuul-config-apollo")中value的默认参数是application，因为这里我们自定义了namespace，所以需要指定，我们使用@ApolloConfigChangeListener监听Apollo的配置下发，有配置更新时会调用refreshZuulProxy()刷新路由信息。

## 3.5 测试

我们启动Client-Apollo工程和Zuul-Apollo工程，打开浏览器访问：[http://localhost:9091/client/hello](http://localhost:9091/client/hello) ，页面可以正常显示，我们在Apollo中修改路由信息，具体如图：

![](../../../assets/images/SpringCloud/Spring Cloud/attachments/SpringCloud系列教程%20第17篇：服务网关Zuul基于Apollo动态路由_image_1.png)

修改完后点击发布，待发布成功后，我们刷新浏览器，之前的路由访问已经报错404，我们使用修改过后的路由[http://localhost:9091/client_new/hello](http://localhost:9091/client_new/hello) ，页面可以正常显示Hello, i am dev from apollo update.，测试成功，我们通过Apollo实现了Zuul的路由信息动态刷新。