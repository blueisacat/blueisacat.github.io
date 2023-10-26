---
layout: default
title: Spring Cloud Alibaba 第1篇：介绍
parent: SpringCloudAlibaba
nav_order: 1.1
---

# 1. Spring Cloud Alibaba是什么？

Spring Cloud Alibaba 致力于提供微服务开发的一站式解决方案。此项目包含开发分布式应用微服务的必需组件，方便开发者通过 Spring Cloud 编程模型轻松使用这些组件来开发分布式应用服务。

依托 Spring Cloud Alibaba，您只需要添加一些注解和少量配置，就可以将 Spring Cloud 应用接入阿里微服务解决方案，通过阿里中间件来迅速搭建分布式应用系统。

# 2. 主要功能

- 服务限流降级： 默认支持 Servlet、Feign、RestTemplate、Dubbo 和 RocketMQ 限流降级功能的接入，可以在运行时通过控制台实时修改限流降级规则，还支持查看限流降级 Metrics 监控。

- 服务注册与发现： 适配 Spring Cloud 服务注册与发现标准，默认集成了 Ribbon 的支持。

- 分布式配置管理： 支持分布式系统中的外部化配置，配置更改时自动刷新。

- 消息驱动能力： 基于 Spring Cloud Stream 为微服务应用构建消息驱动能力。

- 分布式事务： 使用 @GlobalTransactional 注解， 高效并且对业务零侵入地解决分布式事务问题。

- 阿里云对象存储： 阿里云提供的海量、安全、低成本、高可靠的云存储服务。支持在任何应用、任何时间、任何地点存储和访问任意类型的数据。

- 分布式任务调度： 提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。同时提供分布式的任务执行模型，如网格任务。网格任务支持海量子任务均匀分配到所有 Worker（schedulerx-client）上执行。

- 阿里云短信服务： 覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达通道。

# 3. 组件

- Sentinel： 把流量作为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

- Nacos： 一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

- RocketMQ： 一款开源的分布式消息系统，基于高可用分布式集群技术，提供低延时的、高可靠的消息发布与订阅服务。

- Dubbo： Apache Dubbo™ 是一款高性能 Java RPC 框架。

- Seata： 阿里巴巴开源产品，一个易于使用的高性能微服务分布式事务解决方案。

- Alibaba Cloud ACM： 一款在分布式架构环境中对应用配置进行集中管理和推送的应用配置中心产品。

- Alibaba Cloud OSS: 阿里云对象存储服务（Object Storage Service，简称 OSS），是阿里云提供的海量、安全、低成本、高可靠的云存储服务。您可以在任何应用、任何时间、任何地点存储和访问任意类型的数据。

- Alibaba Cloud SchedulerX: 阿里中间件团队开发的一款分布式任务调度产品，提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。

- Alibaba Cloud SMS: 覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达通道。

目前已经开源的组件有：Sentinel， Nacos，RocketMQ，Dubbo

本系列文章目的就是介绍以上开源组件的一些基本使用。

下面简单介绍一下开源组件和目前的SpringCloud、SpringBoot依赖关系。

# 4. 版本说明

## 4.1 版本依赖关系

| Spring Cloud Version | Spring Cloud Alibaba Version | Spring Boot Version | 
| -- | -- | -- |
| Spring Cloud Greenwich | 0.9.0.RELEASE | 2.1.X.RELEASE | 
| Spring Cloud Finchley | 0.2.X.RELEASE | 2.0.X.RELEASE | 
| Spring Cloud Edgware | 0.1.X.RELEASE | 1.5.X.RELEASE | 


- 注意： Spring Cloud Edgware 最低支持 Edgware.SR5 版本

## 4.2 组件版本关系

| Spring Cloud Alibaba Version | Sentinel Version | Nacos Version | RocketMQ Version | Dubbo Version | Seata Version | 
| -- | -- | -- | -- | -- | -- |
| 0.9.0.RELEASE or 0.2.2.RELEASE or 0.1.2.RELEASE | 1.5.2 | 1.0.0 | 4.4.0 | 2.7.1 | 0.4.2 | 
| 0.2.1.RELEASE or 0.1.1.RELEASE | 1.4.0 | 0.6.2 | 4.3.1 | ❌ | ❌ | 
| 0.2.0.RELEASE or 0.1.0.RELEASE | 1.3.0-GA | 0.3.0 | ❌ | ❌ | ❌ | 


## 4.3 依赖管理

Spring Cloud Alibaba BOM 包含了它所使用的所有依赖的版本。

### 4.3.1 Spring Cloud Greenwich

如果需要使用 Spring Cloud Greenwich 版本，请在 dependencyManagement 中添加如下内容

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>0.9.0.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

### 4.3.2 Spring Cloud Finchley

如果需要使用 Spring Cloud Finchley 版本，请在 dependencyManagement 中添加如下内容

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>0.2.2.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

### 4.3.3 Spring Cloud Edgware

如果需要使用 Spring Cloud Edgware 版本，请在 dependencyManagement 中添加如下内容

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>0.1.2.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```