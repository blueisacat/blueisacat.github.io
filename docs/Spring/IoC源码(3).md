---
layout: default
title: IoC源码(3)
parent: Spring
---

### 注册bean后置处理器

registerBeanPostProcessors方法负责初始化实现了BeanPostProcessor接口的Bean，并将其注册到BeanFactory中。

后置处理器BeanPostProcessor定义了两个方法postProcessBeforeInitialization和postProcessAfterInitialization。Spring会添加所有的BeanPostProcessor到BeanFactory的beanPostProcessors列表中。

BeanPostProcessor可以理解成辅助类，在所有其它类型的Bean(应用Bean)初始化过程中，Spring会分别在它们初始化前后调用BeanPostProcessor实例的postProcessBeforeInitialization和postProcessAfterInitialization。

前面讲的BeanFactoryPostProcessor，和BeanPostProcessor是一样的。在BeanFactory创建之后，Spring会调用所有BeanFactoryPostProcessor实例的postProcessBeanFactory方法，这样为用户提供了在BeanFactory创建之后，对Bean进行扩展的机会。

现在来看registerBeanPostProcessors源码：

```java
protected void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory) {
    // 调用PostProcessorRegistrationDelegate的registerBeanPostProcessors方法
    PostProcessorRegistrationDelegate.registerBeanPostProcessors(beanFactory, this);
}
```

看PostProcessorRegistrationDelegate的registerBeanPostProcessors的源代码：

```java
public static void registerBeanPostProcessors(
        ConfigurableListableBeanFactory beanFactory, AbstractApplicationContext applicationContext) {
    // 获取所有的postProcessor names
    String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);
    List<String> nonOrderedPostProcessorNames = new ArrayList<>();
    for (String ppName : postProcessorNames) {
        // 放到nonOrderedPostProcessorNames列表中，这里省略了其它代码，都是处理优先级的。BesnPostProcessor也可以设置优先级的，优先级高的会先被调用。                      
        nonOrderedPostProcessorNames.add(ppName);
    }
    // Now, register all regular BeanPostProcessors.
    List<BeanPostProcessor> nonOrderedPostProcessors = new ArrayList<>();
    for (String ppName : nonOrderedPostProcessorNames) {
        // 实例化BeanPostProcessor实例
        BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
        nonOrderedPostProcessors.add(pp);
    }
    // 注册BestProcessor
    registerBeanPostProcessors(beanFactory, nonOrderedPostProcessors);
}
```

看registerBeanPostProcessors源码：

```java
private static void registerBeanPostProcessors(
        ConfigurableListableBeanFactory beanFactory, List<BeanPostProcessor> postProcessors) {
    // 把所有的BeanPostProcessor添加到beanFactory中
    for (BeanPostProcessor postProcessor : postProcessors) {
        beanFactory.addBeanPostProcessor(postProcessor);
    }
}
```

beanFactory是一个DefaultListableBeanFactory对象，我们来看其代码，很简单，几乎就是一行代码：

```java
@Override
public void addBeanPostProcessor(BeanPostProcessor beanPostProcessor) {     
    // Remove from old position, if any
    this.beanPostProcessors.remove(beanPostProcessor);      
    // 将beanPostProcessor添加到beanPostProcessors列表中
    this.beanPostProcessors.add(beanPostProcessor);
}
```

在创建所有的应用bean调用initializeBean方法时，会轮询beanPostProcessors中的对象，并在Bean实例化前后分别调用postProcessBeforeInitialization和postProcessAfterInitialization方法。

好的，注册BeanPostProcessor讲完了，注册过程比较简单，主要是讲了一下它在Spring Bean初始化过程中的作用。我们接下来看最最重要的一个方法finishBeanFactoryInitialization。

### 初始化Bean对象

refresh方法中我们最后要讲的一个方法是finishBeanFactoryInitialization，它负责根据BeanDefinition创建Bean对象，并执行一些与Bean生命周期相关的回调函数。我们直接看源码：

```java
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
    // 省略其它处理代码，直接调用beanFactory的preInstantiateSingletons.
    beanFactory.preInstantiateSingletons();
}
```

始终记住beanFactory是DefaultListableBeanFactory对象，所以看它的preInstantiateSingletons源码。

```java
// 这里要注意，只初始化所有singletons的Bean,关于prototype类型的bean，是每次调用getBean都会创建的，不会在容器启动的时候初始化。
public void preInstantiateSingletons() throws BeansException {      
    List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);     
    for (String beanName : beanNames) {
        // 这里省略了FactoryBean对象处理逻辑，如果是isEglarBean，也会调用getBean方法。所以简单理解为，不管是哪种sigleton bean，都会调用getBean方法
        getBean(beanName);
    }
    // 省略SmartInitializingSingleton的处理，有兴趣的可以去了解一下这个类的作用...     
}
public Object getBean(String name) throws BeansException {
    // getBean其实调用了doGetBean方法。
    return doGetBean(name, null, null, false);
}
```

来看doGetBean的代码，为了减少文章内容，只贴了核心代码：

```java
final String beanName = transformedBeanName(name);
    Object bean;
    // 取单利Bean，这里简单介绍一下DefaultListableBeanFactory的另一种集成关系，它继承了DefaultSingletonBeanRegistry类。这个类允许单利管理，简单讲就是其中定义了一个HashMap<String,Object>，维护了bean名字和Bean对象的关系，并且保证每一个Bean名字只创建一个Bean对象。
    Object sharedInstance = getSingleton(beanName);
    if (sharedInstance != null && args == null) {
        // 如果缓存(上面说的HashMap)中存在bean，则取这个bean
        bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
    }
    else {
        // BeanFactory是可以继承的，如果当前BeanFactory中找不到Bean定义，就从父BeanFactory中去找。
        BeanFactory parentBeanFactory = getParentBeanFactory();
        if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
            // Not found -> check parent.
            String nameToLookup = originalBeanName(name);
            // 省略了参数处理的过程。直接从父BeanFactory去找。
            return (T) parentBeanFactory.getBean(nameToLookup);
        }           
        try {
            final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
            // 这里先处理Bean依赖，如果当前Bean依赖其它Bean，则先注册并实例化依赖的Bean.
            String[] dependsOn = mbd.getDependsOn();
            if (dependsOn != null) {
                for (String dep : dependsOn) {
                    // 先注册依赖Bean
                    registerDependentBean(dep, beanName);
                    // 再实例化依赖Bean
                    getBean(dep);
                }
            }
            // Create bean instance.
            if (mbd.isSingleton()) {
                sharedInstance = getSingleton(beanName, () -> {
                    // 创建单利Bean
                    return createBean(beanName, mbd, args);
                });
                bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
            }
            else if (mbd.isPrototype()) {
                // Prototyp类型的Bean，是每次都会创建的。
                Object prototypeInstance = null;
                try {
                    beforePrototypeCreation(beanName);
                    prototypeInstance = createBean(beanName, mbd, args);
                }
                finally {
                    afterPrototypeCreation(beanName);
                }
                bean = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
            }
            else {
                // 处理其它类型的Bean,如一些扩展的类型Session，Request等等，代码省略了..
            }
        }
        catch (BeansException ex) {
            cleanupAfterBeanCreationFailure(beanName);
            throw ex;
        }
    }
    // 类型转换的代码也省略了...
    return (T) bean;
```

来看一下createBean做了哪些事情。

```java
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args){  
    // 其它代码都省略了，就是调用了doCreateBean方法
    Object beanInstance = doCreateBean(beanName, mbdToUse, args);
    return beanInstance;
}
```

来看doCreateBean的代码：

```java
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
        throws BeanCreationException {
    // 创建BeanWrapper.
    BeanWrapper instanceWrapper = null;     
    if (instanceWrapper == null) {
        // 调用createBeanInstance创建Bean实例，这个方法我们不再仔细往下阅读了，大致的思路是根据RootBeanDefinition找到其类型classType，然后再获取其构造函数，然后根据构造函数使用反射动态创建出一个bean实例，再对这个实例进行一下包装，返回BeanWrapper对象。当需要对bean增强的时候，创建bean也可能使用CGLib动态创建，我们这里只讲最简单的情况。
        instanceWrapper = createBeanInstance(beanName, mbd, args);
    }
    final Object bean = instanceWrapper.getWrappedInstance();       
    Object exposedObject = bean;
    try {
        // 根据BeanDefinition对bean属性进行赋值。
        populateBean(beanName, mbd, instanceWrapper);
        // 初始化Bean,在这里执行Bean生命周期的回调函数，如设置beanName，调用postBestProcessor等。
        exposedObject = initializeBean(beanName, exposedObject, mbd);
    }
    catch (Throwable ex) {          
    }
    return exposedObject;
}
```

上面的两个重要方法分别是createBeanInstance和initializeBean。其中createBeanInstance所做的事情大致是根据RootBeanDefinition找到其类型classType，然后再获取其构造函数，然后根据构造函数使用反射动态创建出一个bean实例，再对这个实例进行一下包装，返回BeanWrapper对象。因为这个类的业务很简单，大部分代码都是在处理一些与反射有关的东西，所以我们不再进行分析了。我们把重点放在initializeBean这个方法上。

来看initializeBean的代码：

```java
protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
    // 执行*Aware接口的方法，Spring提供了大量的*Aware接口，用来给Bean设置值，如可以向Bean中注入ApplicationContext，ClassLoader等。    
    invokeAwareMethods(beanName, bean);
    Object wrappedBean = bean;
    if (mbd == null || !mbd.isSynthetic()) {
        // 如果Bean实现了BeanPostProcessor接口，则执行postProcessBeforeInitialization方法。
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
    }
    try {
        // 执行bean初始化相关方法。如果bean实现了InitializingBean接口，则会调用其afterPropertiesSet方法。如果bean指定了用户自定义的init-method方法，自定义方法也会被执行。
        invokeInitMethods(beanName, wrappedBean, mbd);
    }
    catch (Throwable ex) {
        throw new BeanCreationException(
                (mbd != null ? mbd.getResourceDescription() : null),
                beanName, "Invocation of init method failed", ex);
    }
    if (mbd == null || !mbd.isSynthetic()) {
        // 如果Bean实现了BeanPostProcessor接口，则执行postProcessAfterInitialization方法。
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    }
    return wrappedBean;
}
```

上面代码中相关接口方法的执行都很直接，就是判断Bean对象是否实现了某个接口，如果实现了该接口，就调用接口定义的方法。initializeBean是比较重要的方法，它涉及到了Bean的生命周期，是面试的常考点。它也确实比较重要，我们可以在Bean初始化期间做很多事情。

到目前为止，Bean初始化也讲完了，从finishBeanFactoryInitialization->getBean->createBean->initializeBean。完成了bean的初始化并执行Bean生命周期中相关的回调方法。

### 总结

Spring定义了BeanPostProcessor接口，所有实现了该接口的bean，都将被注册到spring的后置处理器列表中。后置处理器相当于是一些辅助类，所有的应用bean在初始化之后，都会调用所有的后置处理器。这为用户提供了在bean初始化前后，修改或扩展bean的机会。

然后我们讲了最重要的bean加载。ApplicationContext会在启动容器时，加载所有的非lazi-init的singleton bean。ApplicationContext实现了BeanDefinitionRegistry接口，并在之前就已经注册了所有的BeanDefination，将其存储在一个HashMap中。在加载bean时，从HashMap中读取所有的BeanDefinition，并调用其getBean方法。getBean本意是获取bean，但是如果没有，则会创建并进行初始化。

获取bean时调用了doGetBean方法，首先会检查当前bean是否有定义，如果没有定义，则会去父BeanFactory中去查找。并且它也会检查当前bean是否依赖其它bean，如果依赖其它bean，则会先创建所依赖的bean。

创建bean是根据BeanDefinition中的classType，获取其构造函数，然后利用反射创建bean实例，然后调用populateBean方法进行属性设置，最后调用initializeBean回调一些与bean生命周期有关的接口方法。

至此，整个Spring的容器就启动了。