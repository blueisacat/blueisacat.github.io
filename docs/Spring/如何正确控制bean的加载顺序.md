---
layout: default
title: 如何正确控制bean的加载顺序
parent: Spring
---

### 一、为什么控制

springboot遵从约定大于配置的原则，极大程度的解决了配置繁琐的问题。在此基础上，又提供了spi机制，用spring.factories可以完成一个小组件的自动装配功能。

在一般业务场景，可能你不大关心一个bean是如何被注册进spring容器的。只需要把需要注册进容器的bean声明为@Component即可，spring会自动扫描到这个Bean完成初始化并加载到spring上下文容器。

而当你在项目启动时需要提前做一个业务的初始化工作时，或者你正在开发某个中间件需要完成自动装配时。你会声明自己的Configuration类，但是可能你面对的是好几个有互相依赖的Bean。如果不加以控制，这时候可能会报找不到依赖的错误。

但是你明明已经把相关的Bean都注册进spring上下文了呀。这时候你需要通过一些手段来控制springboot中的bean加载顺序。

### 二、误区

在正式说如何控制加载顺序之前，先说2个误区。

- 在标注了@Configuration的类中，写在前面的@Bean一定会被先注册

这个不存在的，spring在以前xml的时代，也不存在写在前面一定会被先加载的逻辑。因为xml不是渐进的加载，而是全部parse好，再进行依赖分析和注册。到了springboot中，只是省去了xml被parse成spring内部对象的这一过程，但是加载方式并没有大的改变。

- 利用@Order这个标注能进行加载顺序的控制

严格的说，不是所有的Bean都可以通过@Order这个标注进行顺序的控制。你把@Order这个标注加在普通的方法上或者类上一点鸟用都没有。

那@Order能控制哪些bean的加载顺序呢，我们先看看官方的解释：

```
{@code @Order} defines the sort order for an annotated component. Since Spring 4.0, annotation-based ordering is supported for many kinds of components in Spring, even for collection injection where the order values of the target components are taken into account (either from their target class or from their {@code @Bean} method). While such order values may influence priorities at injection points, please be aware that they do not influence singleton startup order which is an orthogonal concern determined by dependency relationships and {@code @DependsOn} declarations (influencing a runtime-determined dependency graph).
```

最开始@Order注解用于切面的优先级指定；在 4.0 之后对它的功能进行了增强，支持集合的注入时，指定集合中 bean 的顺序，并且特别指出了，它们不会影响单例启动顺序。

目前用的比较多的有以下3点：

- 控制AOP的类的加载顺序，也就是被@Aspect标注的类

- 控制ApplicationListener实现类的加载顺序

- 控制CommandLineRunner实现类的加载顺序

### 三、如何控制

#### 3.1@DependsOn

@DependsOn注解可以用来控制bean的创建顺序，该注解用于声明当前bean依赖于另外一个bean。所依赖的bean会被容器确保在当前bean实例化之前被实例化。

```
@Configuration
public class BeanOrderConfiguration {
    @Bean
    @DependsOn("beanB")
    public BeanA beanA(){
        System.out.println("bean A init");
        return new BeanA();
    }
 
    @Bean
    public BeanB beanB(){
        System.out.println("bean B init");
        return new BeanB();
    }
 
    @Bean
    @DependsOn({"beanD","beanE"})
    public BeanC beanC(){
        System.out.println("bean C init");
        return new BeanC();
    }
 
    @Bean
    @DependsOn("beanE")
    public BeanD beanD(){
        System.out.println("bean D init");
        return new BeanD();
    }
 
    @Bean
    public BeanE beanE(){
        System.out.println("bean E init");
        return new BeanE();
    }
}
```

以上代码bean的加载顺序为：

> bean B initbean A initbean E initbean D initbean C init


@DependsOn的使用：


- 直接或者间接标注在带有@Component注解的类上面;

- 直接或者间接标注在带有@Bean注解的方法上面;

- 使用@DependsOn注解到类层面仅仅在使用 component-scanning 方式时才有效，如果带有@DependsOn注解的类通过XML方式使用，该注解会被忽略，<bean depends-on="..."/>这种方式会生效。

#### 3.2 参数注入

在@Bean标注的方法上，如果你传入了参数，springboot会自动会为这个参数在spring上下文里寻找这个类型的引用。并先初始化这个类的实例。

利用此特性，我们也可以控制bean的加载顺序。

```
@Bean
public BeanA beanA(BeanB demoB){
  System.out.println("bean A init");
  return new BeanA();
}
 
 
@Bean
public BeanB beanB(){
  System.out.println("bean B init");
  return new BeanB();
}
```

以上结果，beanB先于beanA被初始化加载。

需要注意的是，springboot会按类型去寻找。如果这个类型有多个实例被注册到spring上下文，那你就需要加上@Qualifier("Bean的名称")来指定

#### 3.3 利用bean的生命周期中的扩展点

在spring体系中，从容器到Bean实例化&初始化都是有生命周期的，并且提供了很多的扩展点，允许你在这些步骤时进行逻辑的扩展。

这些可扩展点的加载顺序由spring自己控制，大多数是无法进行干预的。我们可以利用这一点，扩展spring的扩展点。在相应的扩展点加入自己的业务初始化代码。从来达到顺序的控制。

具体关于spring容器中大部分的可扩展点的分析，之前已经写了一篇文章详细介绍了：《Springboot启动扩展点超详细总结，再也不怕面试官问了》。

#### 3.4 @AutoConfigureOrder

```
@Configuration
@AutoConfigureOrder(2)
public class BeanOrderConfiguration1 {
    @Bean
    public BeanA beanA(){
        System.out.println("bean A init");
        return new BeanA();
    }
}
 
 
@Configuration
@AutoConfigureOrder(1)
public class BeanOrderConfiguration2 {
    @Bean
    public BeanB beanB(){
        System.out.println("bean B init");
        return new BeanB();
    }
}
```

无论你2个数字填多少，都不会改变其加载顺序结果。

那这个@AutoConfigureOrder到底是如何使用的呢。

经过测试发现，@AutoConfigureOrder只能改变外部依赖的@Configuration的顺序。如何理解是外部依赖呢。

能被你工程内部scan到的包，都是内部的Configuration，而spring引入外部的Configuration，都是通过spring特有的spi文件：spring.factories

换句话说，@AutoConfigureOrder能改变spring.factories中的@Configuration的顺序。

```
@Configuration
@AutoConfigureOrder(10)
public class BeanOrderConfiguration1 {
    @Bean
    public BeanA beanA(){
        System.out.println("bean A init");
        return new BeanA();
    }
}
 
 
@Configuration
@AutoConfigureOrder(1)
public class BeanOrderConfiguration2 {
    @Bean
    public BeanB beanB(){
        System.out.println("bean B init");
        return new BeanB();
    }
}
```

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.example.demo.BeanOrderConfiguration1,\
  com.example.demo.BeanOrderConfiguration2
```

### 四、总结

其实在工作中，我相信很多人碰到过复杂的依赖关系的bean加载，把这种不确定性交给spring去做，还不如我们自己去控制，这样在阅读代码的时候 ，也能轻易看出bean之间的依赖先后顺序。