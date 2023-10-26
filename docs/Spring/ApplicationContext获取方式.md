---
layout: default
title: ApplicationContext获取方式
parent: Spring
---

### 直接注入

```java
@Resource
private ApplicationContext ctx;
```

### 实现ApplicationContextAware接口

```java
import org.springframework.beans.BeansException;import org.springframework.context.ApplicationContext;import org.springframework.context.ApplicationContextAware;import org.springframework.stereotype.Component;
@Componentpublic class ApplicationContextProvider
    implements ApplicationContextAware{
    /**
     * 上下文对象实例
     */
    private ApplicationContext applicationContext;
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
    /**
     * 获取applicationContext
     * @return
     */
    public ApplicationContext getApplicationContext() {
        return applicationContext;
    }
    /**
     * 通过name获取 Bean.
     * @param name
     * @return
     */
    public Object getBean(String name){
        return getApplicationContext().getBean(name);
    }
    /**
     * 通过class获取Bean.
     * @param clazz
     * @param <T>
     * @return
     */
    public <T> T getBean(Class<T> clazz){
        return getApplicationContext().getBean(clazz);
    }
    /**
     * 通过name,以及Clazz返回指定的Bean
     * @param name
     * @param clazz
     * @param <T>
     * @return
     */
    public <T> T getBean(String name,Class<T> clazz){
        return getApplicationContext().getBean(name, clazz);
    }}
```

### 在自定义AutoConfiguration中获取

```java
@Configuration@EnableFeignClients("com.yidian.data.interfaces.client")public class FeignAutoConfiguration {
    FeignAutoConfiguration(ApplicationContext context) {
        // 在初始化AutoConfiguration时会自动传入ApplicationContext
         doSomething(context);
    }}
```

### 启动时获取ApplicationContext

```java
import org.springframework.boot.SpringApplication;import org.springframework.boot.autoconfigure.SpringBootApplication;import org.springframework.context.ApplicationContext;import org.springframework.context.ConfigurableApplicationContext;
@SpringBootApplicationpublic class WebApplication {
    private static ApplicationContext applicationContext;
    public static void main(String[] args) {
        applicationContext = SpringApplication.run(WebApplication.class, args);
        SpringBeanUtil.setApplicationContext(applicationContext);
    }}
```

### 通过WebApplicationContextUtils获取

```java
WebApplicationContextUtils.getRequiredWebApplicationContext(ServletContext sc);
WebApplicationContextUtils.getWebApplicationContext(ServletContext sc);
```