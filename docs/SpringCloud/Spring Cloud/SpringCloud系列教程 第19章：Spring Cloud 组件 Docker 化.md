---
layout: default
title: SpringCloud系列教程 第19章：Spring Cloud 组件 Docker 化
parent: SpringCloud系列教程
grand_parent: SpringCloud
nav_order: 1.19
---

> Springboot: 2.1.6.RELEASE
> SpringCloud: Greenwich.SR1
> 如无特殊说明，本系列教程全采用以上版本


前面的文章《跟我学SpringCloud | 第十八篇：微服务 Docker 化之基础环境》我们介绍了基础环境系统和 JRE 的容器化，这一节我们介绍 Spring Cloud 组件的容器化，主要包括 eureka-server 、 gateway-server 和 provider-server 。


# 1. Docker 化配置

这里我们根据 dockerfile-maven-plugin 这个 maven 插件来构建，可以用来构建 docker 镜像的 maven 插件有很多，其中使用比较多的包括 docker-maven-plugin ，这两个插件是同一个人在 Github 上开源的， docker-maven-plugin 被作者标记为不活跃的，并且建议大家使用 dockerfile-maven-plugin ，作者可能是觉得 docker-maven-plugin 在使用的方式上并不是那么的优雅，才又新写了 dockerfile-maven-plugin ，具体信息可以访问 docker-maven-plugin 在 Github 上的官方仓库（[https://github.com/spotify/docker-maven-plugin](https://github.com/spotify/docker-maven-plugin)）。

本篇文章我们将介绍如何在 IDE 工具 idea 中使用 Docker 工具，并为我们的工程构建镜像。

# 2. 实战

## 2.1 创建父工程

父工程 pom.xml 依赖文件：

```
<build>
    <plugins>
        <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>dockerfile-maven-plugin</artifactId>
            <version>${dockerfile.maven.version}</version>
            <executions>
                <execution>
                    <id>default</id>
                    <goals>
                        <goal>build</goal>
                        <goal>push</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <repository>${docker.image.prefix}/${project.artifactId}</repository>
                <tag>${project.version}</tag>
                <buildArgs>
                    <JAR_FILE>${project.build.finalName}.jar</JAR_FILE>
                </buildArgs>
            </configuration>
        </plugin>
    </plugins>
</build>
```

这里笔者仅截取了 dockerfile-maven-plugin 相关的代码，其余代码可参考 Github 仓库。

- 版本笔者这里选择的是 v1.4.12 ，也是目前官方仓库上最新的版本，此版本于 2019年7月30日 发布。

- <configuration> 标签中是一些有关于 Docker 镜像相关的配置，包括当前的 <repository> 和 <tag> 。但是经笔者测试，如果使用 idea 生成远端镜像（本地无Docker服务），这两个部分还是会为 null 的，生成完后需要使用命令 docker tag [imageId] [repository]:[tag] 来重新命名镜像。如果本地有 Docker 服务，则无此问题。

## 2.2 创建子工程 eureka-server

### 2.2.1 子工程依赖 pom.xml

```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
        <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>dockerfile-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

由于在父工程中已经详细配置相关内容，这里仅需要将 dockerfile-maven-plugin 引入即可。

### 2.2.2 子工程Dockerfile

```
FROM registry.cn-shanghai.aliyuncs.com/springcloud-book/java:8u221-jre
ARG JAR_FILE
ENV PROFILE default
ADD target/${JAR_FILE} /opt/app.jar
EXPOSE 8080
ENTRYPOINT java ${JAVA_OPTS} -Djava.security.egd=file:/dev/./urandom -Duser.timezone=Asia/Shanghai -Dfile.encoding=UTF-8 -Dspring.profiles.active=${PROFILE} -jar /opt/app.jar
```

- 这里使用了我们在前一篇文章《跟我学SpringCloud | 第十八篇：微服务 Docker 化之基础环境》建立的 registry.cn-shanghai.aliyuncs.com/springcloud-book/java:8u221-jre 镜像。

- 这里的 JAR_FILE 是我们在父工程的 pom.xml 中配置的参数，作用是指明了我们所需要打成镜像的 jar 的名称，我们在父工程的配置内容为 ${project.build.finalName}.jar 。

- 端口号这里统一的指定为 8080 ，最后映射出来的端口号可以根据每个不同的应用进行修改。

- 这里统一指定了时区为 Asia/Shanghai ，如果不做指定，可能默认时区不是中国的时区，这里一定要注意

- 暴露了 JAVA_OPTS 环境变量，允许不同的应用去指定不同的 jvm 参数。

- 暴露了 PROFILE 环境变量，允许不同的应用去指定不同的 profile。

### 2.2.3 子工程配置文件 application.yml

```
server:
  port: 8080
spring:
  application:
    name: spring-cloud-eureka-server
  cloud:
    inetutils:
      use-only-site-local-interfaces: true
eureka:
  instance:
    prefer-ip-address: true
    lease-expiration-duration-in-seconds: 90
    lease-renewal-interval-in-seconds: 30
  server:
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 60000
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://${EUREKA_SERVER_HOST}:${EUREKA_SERVER_PORT}/eureka/
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

- 在配置文件中，我们留出两个环境变量 EUREKA_SERVER_HOST 和 EUREKA_SERVER_PORT ，不同的应用可以通过启动命令来进行配置。

### 2.2.4 子工程构建 Docker 镜像

其余部分代码笔者这里就不列出了，有需要的朋友请参考 Github 仓库，接下来尝试打包镜像，这里我们直接使用 idea 的 package 命令，也可以在工程的跟目录下执行命令：

```
mvn clean package
```

执行以上命令需本地配置 maven 环境变量，否则会报错 mvn 命令找不到。笔者使用 idea 提供的 package 命令，具体操作如图：

![](../../../assets/images/SpringCloud/Spring Cloud/attachments/SpringCloud系列教程%20第19章：Spring%20Cloud%20组件%20Docker%20化_image_0.png)

直接双击，可以看到控制台开始执行打包命令，在打包完成后会将 target/${project.build.finalName}.jar 打包成 Docker 镜像，我们可以看到控制台输出的命令：

```
[INFO] Image will be built as registry.cn-shanghai.aliyuncs.com/springcloud-book/eureka_server:0.0.1-SNAPSHOT
[INFO] 
[INFO] Step 1/6 : FROM registry.cn-shanghai.aliyuncs.com/springcloud-book/java:8u221-jre
[INFO] 
[INFO] Pulling from springcloud-book/java
[INFO] Image 9d48c3bd43c5: Already exists
[INFO] Image 828e49a80267: Already exists
[INFO] Image 00b99dcc31ec: Already exists
[INFO] Image e6e45bfebaec: Already exists
[INFO] Digest: sha256:32d9a2557298d7b56997ded086baf043145ca178b71f6c8f47e76f837614bda0
[INFO] Status: Downloaded newer image for registry.cn-shanghai.aliyuncs.com/springcloud-book/java:8u221-jre
[INFO]  ---> 5f59ebcc4b59
[INFO] Step 2/6 : ARG JAR_FILE
[INFO] 
[INFO]  ---> Running in 8e3e51705681
[INFO] Removing intermediate container 8e3e51705681
[INFO]  ---> 0bcb834dc28b
[INFO] Step 3/6 : ENV PROFILE default
[INFO] 
[INFO]  ---> Running in 4a8e840e7a95
[INFO] Removing intermediate container 4a8e840e7a95
[INFO]  ---> 11f97aef63ee
[INFO] Step 4/6 : ADD target/${JAR_FILE} /opt/app.jar
[INFO] 
[INFO]  ---> 61c0ff0c5b6e
[INFO] Step 5/6 : EXPOSE 8080
[INFO] 
[INFO]  ---> Running in 264bf0d042b6
[INFO] Removing intermediate container 264bf0d042b6
[INFO]  ---> d951ab655bd5
[INFO] Step 6/6 : ENTRYPOINT java ${JAVA_OPTS} -Djava.security.egd=file:/dev/./urandom -Duser.timezone=Asia/Shanghai -Dfile.encoding=UTF-8 -Dspring.profiles.active=${PROFILE} -jar /opt/app.jar
[INFO] 
[INFO]  ---> Running in 4bd69be29edc
[INFO] Removing intermediate container 4bd69be29edc
[INFO]  ---> 704768c9dd75
[INFO] Successfully built 704768c9dd75
[INFO] Successfully tagged registry.cn-shanghai.aliyuncs.com/springcloud-book/eureka_server:0.0.1-SNAPSHOT
[INFO] 
[INFO] Detected build of image with id 704768c9dd75
[INFO] Building jar: D:\Development\SpringCloudLearning\chapter18\eureka_server\target\eureka_server-0.0.1-SNAPSHOT-docker-info.jar
[INFO] Successfully built registry.cn-shanghai.aliyuncs.com/springcloud-book/eureka_server:0.0.1-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  01:38 min
[INFO] Finished at: 2019-09-21T20:05:50+08:00
[INFO] ------------------------------------------------------------------------
```

从以上命令，我们可以看到镜像每一步的构建过程，同样，这个镜像笔者这里 push 到阿里云的镜像仓库，有需要的朋友可以直接 pull 后使用。

```
docker pull registry.cn-shanghai.aliyuncs.com/springcloud-book/eureka_server:0.0.1-SNAPSHOT
```

### 2.2.5 子工程测试

测试这里笔者启动两个镜像，构建两个 eureka_server 服务，对外映射分别使用不同的端口， 8761 和 8762 ，命令如下：

```
docker run -p 8761:8080 --rm \
-e JAVA_OPTS='-server -Xmx1g' \
-e PROFILE='default' \
-e EUREKA_SERVER_HOST=192.168.0.128 \
-e EUREKA_SERVER_PORT=8762 \
registry.cn-shanghai.aliyuncs.com/springcloud-book/eureka_server:0.0.1-SNAPSHOT
```

```
docker run -p 8762:8080 --rm \
-e JAVA_OPTS='-server -Xmx1g' \
-e PROFILE='default' \
-e EUREKA_SERVER_HOST=192.168.0.128 \
-e EUREKA_SERVER_PORT=8761 \
registry.cn-shanghai.aliyuncs.com/springcloud-book/eureka_server:0.0.1-SNAPSHOT
```

启动成功后打开浏览器分别访问路径：[http://localhost:8761/](http://localhost:8761/) 、 [http://localhost:8762/](http://localhost:8762/) ，结果如图：

![](../../../assets/images/SpringCloud/Spring Cloud/attachments/SpringCloud系列教程%20第19章：Spring%20Cloud%20组件%20Docker%20化_image_1.png)

![](../../../assets/images/SpringCloud/Spring Cloud/attachments/SpringCloud系列教程%20第19章：Spring%20Cloud%20组件%20Docker%20化_image_2.png)

# 3. 小结

还有其余两个工程 provider_server 和 gateway_server 笔者这里就不做演示了，相关 Dockerfile 和运行所使用的命令都已经上传至 Github 仓库，有需要的读者可以直接访问 Github 仓库获取。