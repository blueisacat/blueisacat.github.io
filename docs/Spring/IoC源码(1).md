> IoC(Inversion of Control)控制反转


### 核心概念

**BeanDefinition**

这个是一个接口，它是用来存储bean定义的一些信息的，比如ClassName，Scope等等。它的实现有RootBeanDefinition，AnnotatedGenericBeanDefinition等。

**BeanDefinitionHolder**

这是BeanDefinition的包装类，用来存储Beandifinition，name以及aliases等。

**BeanFactory & AbstractBeanFactory**

BeanFactory就是bean工厂，所有的bean都是由BeanFactory统一创建和管理，spring提供了一个AbstractBeanFactory，它继承了BeanFactory接口，并且实现了大部分BeanFactory的功能。

AbstractBeanFactory有一个非常重要的方法叫getBean()

```java
public <T> T getBean(String name, Class<T> requiredType) throws BeansException {
    return doGetBean(name, requiredType, null, false);
}
```

它是用来根据名字和类型获取bean对象的。它获取对象的原理是，如果BeanFactory中存在bean，则从缓存中取bean，否则创建并返回该bean，并且将该bean添加到缓存中(Singleton类型的bean)。所以确切的讲，bean实在什么时候创建的，要看什么时候调用getBean方法获取这个bean，根据bean定义的不同，getBean方法会在不同时候被调用。spring中bean大致可以分为两类，应用类bean以及功能性bean。spring提供了一些功能性接口如BeanFactoryPostProcessor和BeanPostProcessor，实现了这些接口的bean可以成为功能性bean。如果bean实现了BeanFactoryPostProcessor，则在refresh方法中调用invokeBeanFactoryPostProcessors方法时创建bean，如果实现了BeanPostProcessor方法，则在registerBeanPostProcessor方法中创建bean。其它的bean应该属于应用类bean，在默认的finishBeanFactoryInitialization方法中创建bean。

**AnnotationConfigApplicationContext & BeanFactory & BeanDefinitionRegistry & DefaultListableBeanFactory**

AnnotationConfigApplicationContext这个类是基于注解的容器类，它实现了BeanFactory和BeanDefinitionRegistry两个接口，拥有bean对象的管理和BeanDefinition注册功能。同时这个类拥有一个DefaultListableBeanFactory的对象。

DefaultListableBeanFactory也实现了BeanFactory和BeanDefinitionRegistry接口，拥有bean对象管理和BeanDefinition注册功能。AnnotationConfigAppliationContext是委托。

DefaultListableBeanFactory来实现对象管理和BeanDefinition注册的。这里使用的是代理模式。

spring容器启动过程主要做了两件事情:

- 加载BeanDefinition

- 创建Bean

### 加载BeanDefinition

启动一个基于注解的spring容器，并从容器中获取helloService对象。

```java
public static void main(String[] args) {
    AbstractApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);
    context.registerShutdownHook();
    HelloService helloService = context.getBean(HelloService.class);
    helloService.sayHello();
}
```

之所以要使用AbstractAppliationContext类，是因为我们的HelloService实现了一些BeanPostProcessor这样的接口，需要调用AnnotationConfigApplicationContext对象的registerShutdwonHook方法。

进入AnnotationConfigApplicationContext的构造函数:

```java
public AnnotationConfigApplicationContext(Class<?>... annotatedClasses) {
    this();
    register(annotatedClasses);
    refresh();
}
```

两个主方法，register和refresh。

**register**

进入register方法，它调用了reader.register方法。

```java
public void register(Class<?>... annotatedClasses) {
    Assert.notEmpty(annotatedClasses, "At least one annotated class must be specified");
    this.reader.register(annotatedClasses);
}
```

这个reader是AnnotatedBeanDefinitionReader类的一个实例，在AnnotationConfigApplicationContext的构造函数中创建的。构造函数还创建了一个scanner，用于包扫描。

```java
public AnnotationConfigApplicationContext() {
    this.reader = new AnnotatedBeanDefinitionReader(this);
    this.scanner = new ClassPathBeanDefinitionScanner(this);
}
```

进入reader的register方法:

```java
public void register(Class<?>... annotatedClasses) {
    for (Class<?> annotatedClass : annotatedClasses) {
        registerBean(annotatedClass);
    }
}
public void registerBean(Class<?> annotatedClass) {
    doRegisterBean(annotatedClass, null, null, null);
}
<T> void doRegisterBean(Class<T> annotatedClass, @Nullable Supplier<T> instanceSupplier, @Nullable String name,
        @Nullable Class<? extends Annotation>[] qualifiers, BeanDefinitionCustomizer... definitionCustomizers) {
    // 创建BeanDefinition
    AnnotatedGenericBeanDefinition abd = new AnnotatedGenericBeanDefinition(annotatedClass);
    // 生成bean name
    String beanName = (name != null ? name : this.beanNameGenerator.generateBeanName(abd, this.registry));
    AnnotationConfigUtils.processCommonDefinitionAnnotations(abd);
    // ...省去了其它代码
    // 创建BeanDefinitionHolder，它是BeanDefiniton的一个包装类，包含BeanDefiniton对象和它的名字
    BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(abd, beanName);
    // 最后调用registerBeanDefinition
    BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, this.registry);
}
```

进入BeanDefinitionReaderUtils.registerBeanDefinition

```java
public static void registerBeanDefinition(
        BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
        throws BeanDefinitionStoreException {
    // Register bean definition under primary name.
    String beanName = definitionHolder.getBeanName();
    // 调用registry的registerBeanDefinition方法，registry是我们传进来的AnnotationConfigApplicationContext，因为它实现了BeanDefinitionRegistry，所以我们进入AnnotationConfigApplicationContext的registerBeanDefinition方法。
    registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());
    // Register aliases for bean name, if any.
    String[] aliases = definitionHolder.getAliases();
    if (aliases != null) {
        for (String alias : aliases) {
            registry.registerAlias(beanName, alias);
        }
    }
}
```

进入AnnotationConfigApplicationContext的registerBeanDefinition方法。在她的父类GenericApplicationContext中。

```java
public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
        throws BeanDefinitionStoreException {
    // 调用beanFactory的registerBeanDefinition。我们之前讲过，AnnotationConfigApplicationContext是委托DefaultListableBeanFactory来注册BeanDefinition和管理Bean的，所以调用了beanFactory的registerBeanDefinition
    this.beanFactory.registerBeanDefinition(beanName, beanDefinition);
}
```

在GenericApplicationContext的构造函数中，创建了这个beanFactory。

```java
既然beanFactory是DefaultListableBeanFactory的实例，所以进入DefaultListableBeanFactory的registerBeanDefinition方法。
public GenericApplicationContext() {
    // 前面也讲过，DefaultListableBeanFactory也实现了BeanDefinitionRegistry和BeanFactory接口的。
    this.beanFactory = new DefaultListableBeanFactory();
}
```

既然beanFactory是DefaultListableBeanFactory的实例，所以进入DefaultListableBeanFactory的registerBeanDefinition方法。

```java
public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
        throws BeanDefinitionStoreException {
    //... 省去重复注册代码逻辑以及其它一些逻辑判断，
    //最终执行以下代码，可以看出，最终是将beanDefinition放到beanDefinitionMap中。所有的beanDefinition都会放到这个map中，以beanName为主键。如果一个beanDefinition有多个别名，那个它会被注册多次。
    this.beanDefinitionMap.put(beanName, beanDefinition);
}
```

到此为止，register方法就分析完了。但是这里只是注册完了配置文件的beanDefinition。至于我们自己注入的类，或者通过扫描注入的类，在refresh方法中。