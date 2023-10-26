上一篇《Spring Cloud Alibaba | Nacos服务注册与发现》我们聊了Nacos服务注册与发现，这一篇我们接着聊Nacos配置管理。

Nacos具有配置管理的功能，在Spring Cloud中可以用作配置中心，代替Spring Cloud Config组件，下面我们聊一下Nacos如何和Spring Cloud集成配置中心。

创建一个项目：nacos-config

# 1. pom.xml 项目依赖

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
    <artifactId>nacos-config</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>nacos-config</name>
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
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
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

主要引入spring-cloud-starter-alibaba-nacos-config，为开启nacos配置中心

# 2. 在 bootstrap.properties 中配置 Nacos server 的地址和应用名

```
spring.cloud.nacos.config.server-addr=127.0.0.1:8848
spring.application.name=spring-cloud-nacos-config
```

说明：之所以需要配置 spring.application.name ，是因为它是构成 Nacos 配置管理 dataId字段的一部分。

在 Nacos Spring Cloud 中，dataId 的完整格式如下：

```
${prefix}-${spring.profile.active}.${file-extension}
```

- prefix 默认为 spring.application.name 的值，也可以通过配置项 spring.cloud.nacos.config.prefix来配置。

- spring.profile.active 即为当前环境对应的 profile，详情可以参考 Spring Boot文档。 注意：当 spring.profile.active 为空时，对应的连接符 – 也将不存在，dataId 的拼接格式变成 ${prefix}.${file-extension}

- file-exetension 为配置内容的数据格式，可以通过配置项 spring.cloud.nacos.config.file-extension 来配置。目前只支持 properties 和 yaml 类型。

# 3. 通过 Spring Cloud 原生注解 @RefreshScope 实现配置自动更新

```
package com.springcloud.nacosconfig.controller;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
/**
 * Created with IntelliJ IDEA.
 *
 * @Date: 2019/7/14
 * @Time: 18:13
 * @email: inwsy@hotmail.com
 * Description:
 */
@RestController
@RequestMapping("/config")
@RefreshScope
public class ConfigController {
    @Value("${useLocalCache:false}")
    private boolean useLocalCache;
    @RequestMapping("/get")
    public boolean get() {
        return useLocalCache;
    }
}
```

# 4. 测试

首先通过调用 Nacos Open API 向 Nacos Server 发布配置：dataId 为example.properties，内容为useLocalCache=true

```
curl -X POST "
```

运行 NacosConfigApplication，调用 curl [http://localhost:8080/config/get](http://localhost:8080/config/get)，返回内容是 true。

再次调用 Nacos Open API 向 Nacos server 发布配置：dataId 为example.properties，内容为useLocalCache=false

```
curl -X POST "
```

再次访问 [http://localhost:8080/config/get](http://localhost:8080/config/get)，此时返回内容为false，说明程序中的useLocalCache值已经被动态更新了。

至此，Nacos配置管理已经介绍完成。