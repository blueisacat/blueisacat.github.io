---
layout: default
title: bean定义动态生成
parent: Spring
---

> bean定义在spring中的组件名叫:BeanDefinition当我们创建了一个BeanDefinition后可以通过BeanDefinitionRegistry接口(ApplicationContext会实现此接口)将新的BeanDefinition注册到spring上下文中。


```java
void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
```

### BeanFactoryPostProcessor(bean工厂的后置处理器)

这个接口的作用是在spring上下文的注册bean定义的逻辑都跑完后,但是所有的bean都还没真正实例化之前调用。

```java
void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;
```

这个方法的主要用途就是通过注入进来的BeanFactory,在真正初始化bean之前,再对spring上下文做一些动态修改。增加或者修改某些bean定义的值,甚至再动态创建一些BeanDefinition都可以。

示例:

```java
@Override
  public void postProcessBeanFactory(final ConfigurableListableBeanFactory beanFactory) throws BeansException {
    GenericBeanDefinition beanDefinition = getGenericBeanDefinition();
    ((DefaultListableBeanFactory) beanFactory) .registerBeanDefinition("dynamicUser", beanDefinition);
  }
  private GenericBeanDefinition getGenericBeanDefinition() {
    GenericBeanDefinition beanDefinition = new GenericBeanDefinition();
    beanDefinition.setBeanClass(User.class);
    beanDefinition.getPropertyValues().add("name", "张三");
    return beanDefinition;
  }
```

### BeanDefinitionRegistryPostProcessor(bean注册器的后置处理器)

如上我们可以定义一个有关User这个bean的定义，并且预先设置此bean的name属性值是"张三",最后注册到beanFactory中。

有鉴于此,spring特意提供了一个更具体的子接口BeanDefinitionRegistryPostProcessor这个子接口的方法postProcessBeanDefinitionRegistry会被spring先于postProcessBeanFactory这个方法执行。不过呢，其实这2个方法都是注入的spring的上下文，只不过声明类型不同罢了。所以可以做的使一摸一样。

```java
 private GenericBeanDefinition getGenericBeanDefinition() {
    GenericBeanDefinition beanDefinition = new GenericBeanDefinition();
    beanDefinition.setBeanClass(User.class);
    beanDefinition.getPropertyValues().add("name", "张三");
    return beanDefinition;
  }
  @Override
  public void postProcessBeanDefinitionRegistry(final BeanDefinitionRegistry registry) throws BeansException {
    GenericBeanDefinition beanDefinition = getGenericBeanDefinition();
    registry.registerBeanDefinition("dynamicUser2",beanDefinition);
  }
```

最后调用此spring上下文得到这2个bean:

```java
public static void main(String[] args)
    {
        ConfigurableApplicationContext ctx = SpringApplication.run(DemoApplication.class, args);
        User dynamicUser = ctx.getBean("dynamicUser", User.class);
        System.out.println(dynamicUser);
        User dynamicUser2 = ctx.getBean("dynamicUser2", User.class);
        System.out.println(dynamicUser2);
    }
```

### BeanDifinitionBuilder

建造者模式去创建一个BeanDefinition。从2.5之后GenericBeanDefinition代替了原本的RootBeanDefinition和ChildBeanDefinition。我们通常使用GenericBeanDefinition就足够了。

我们通过Builder模式去设置BeanDefinition一些常用方法:

```java
//设置Bean的构造函数传入的参数值
public BeanDefinitionBuilder addConstructorArgValue(Object value)
//设置构造函数引用其他的bean
public BeanDefinitionBuilder addConstructorArgReference(String beanName)
//设置这个bean的 init方法和destory方法
public BeanDefinitionBuilder setInitMethodName(String methodName) 
public BeanDefinitionBuilder setDestroyMethodName(String methodName) 
//设置单例/多例
public BeanDefinitionBuilder setScope(String scope) 
//设置是否是个抽象的BeanDefinition，如果为true，表明这个BeanDefinition只是用来给子BeanDefinition去继承的，Spring不会去尝试初始化这个Bean。
public BeanDefinitionBuilder setAbstract(boolean flag) 
//是否懒加载，默认是false
public BeanDefinitionBuilder setLazyInit(boolean lazy) 
//自动注入依赖的模式，默认不注入
public BeanDefinitionBuilder setAutowireMode(int autowireMode) 
//检测依赖。
public BeanDefinitionBuilder setDependencyCheck(int dependencyCheck) 
```

### ImportBeanDefinitionRegistrar

这个接口的实现类的作用是当这个接口的实现类被@Import接口引入某个被标记了

@Configuration的注册类时，可以得到这个类的所有注解，然后做一些动态注册bean的事儿。

```java
    @Override
    public void registerBeanDefinitions(AnnotationMetadata metadata,
            BeanDefinitionRegistry registry) {
        registerDefaultConfiguration(metadata, registry);
        registerFeignClients(metadata, registry);
    }
```

这里面的AnnotationMetadata到底会包含什么呢?其实就是被Import标记的Application这个注册文件的所有注解。

我们可以通过拿到注解上的一些配置信息去动态生成BeanDifinition。