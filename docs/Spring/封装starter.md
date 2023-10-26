# 为何封装？

SpringBoot的starter开箱即用，只需要引入依赖，就可以帮你自动装配bean，这样可以让开发者不需要过多的关注框架的配置。

# 如何封装？

## 依赖

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-configuration-processor</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-autoconfigure</artifactId>
    </dependency>
</dependencies>
```

官方的starter命名规范为：spring-boot-starter-{name}

非官方的starter明明规范为：{name}-spring-boot-starter

## 配置加载类

```
package cn.blueisacat.testspringbootstarter.properties;
import org.springframework.boot.context.properties.ConfigurationProperties;
@ConfigurationProperties(prefix = "test")
public class TestProperties {
    private String test1 = "a";
    private String test2 = "b";
    public String getTest1() {
        return test1;
    }
    public void setTest1(String test1) {
        this.test1 = test1;
    }
    public String getTest2() {
        return test2;
    }
    public void setTest2(String test2) {
        this.test2 = test2;
    }
}
```

## 业务实例类

```
package cn.blueisacat.testspringbootstarter.service;
import cn.blueisacat.testspringbootstarter.properties.TestProperties;
public class TestService {
    private TestProperties properties;
    public TestService(TestProperties properties) {
        this.properties = properties;
    }
    public void test() {
        System.out.println(properties.getTest1());
        System.out.println(properties.getTest2());
    }
}
```

## 自动配置类

```
package cn.blueisacat.testspringbootstarter.config;
import cn.blueisacat.testspringbootstarter.properties.TestProperties;
import cn.blueisacat.testspringbootstarter.service.TestService;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import javax.annotation.Resource;
@Configuration
@ConditionalOnClass(TestService.class)
@EnableConfigurationProperties(TestProperties.class)
public class TestAutoConfig {
    @Resource
    private TestProperties testProperties;
    @Bean
    @ConditionalOnMissingBean(TestService.class)
    TestService getTestService() {
        return new TestService(testProperties);
    }
}
```

## spring.factories

1. 在

1. 在

spring.factories内容如下：

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=cn.blueisacat.testspringbootstarter.config.TestAutoConfig
```

## 打包发布

```
mvn:install
```

## 测试

- 引入依赖

```
<dependency>
	<groupId>cn.blueisacat</groupId>
    <artifactId>test-spring-boot-starter</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>	
```

- 添加配置

```
test:
    test1: 1
    test2: 2
```

# 注解说明

## @ConfigurationProperties注解

在 

1. 其在使用上需要注意以下几点：

- 前缀定义了哪些外部属性将绑定到类的字段上

- 根据SpringBoot宽松的绑定规则，类的属性名称必须与外部属性的名称匹配

- 可以用一个值初始化一个字段来定义一个默认值

- 类本身可以是包私有的

- 类的字段必须有公共setter方法

1. SpringBoot宽松绑定规则，以下变体都将绑定到hostName属性上

```
hostName=localhost
hostname=localhost
host_name=localhost
host-name=localhost
HOST_NAME=localhost
```

1. 在SpringBoot中想要 

- @Component

注解添加在 

- @EnableConfigurationProperties

注解添加在 

1. 属性匹配问题

- 无法转换的属性

比如说在 

- 未知的属性

默认情况下，SpringBoot会忽略那些布恩那个绑定的字段，此时如果希望启动失败，可以设置

## @ConditionalOn相关注解

```
@ConditionalOnBean：当容器中有指定的Bean的条件下  
@ConditionalOnClass：当类路径下有指定的类的条件下  
@ConditionalOnExpression：基于SpEL表达式作为判断条件  
@ConditionalOnJava：基于JVM版本作为判断条件  
@ConditionalOnJndi：在JNDI存在的条件下查找指定的位置  
@ConditionalOnMissingBean：当容器中没有指定Bean的情况下  
@ConditionalOnMissingClass：当类路径下没有指定的类的条件下  
@ConditionalOnNotWebApplication：当前项目不是Web项目的条件下  
@ConditionalOnProperty：指定的属性是否有指定的值  
@ConditionalOnResource：类路径下是否有指定的资源  
@ConditionalOnSingleCandidate：当指定的Bean在容器中只有一个，或者在有多个Bean的情况下，用来指定首选的Bean @ConditionalOnWebApplication：当前项目是Web项目的条件下  
```