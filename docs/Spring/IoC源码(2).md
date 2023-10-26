### 加载BeanDefinition

refresh其实实在AbstractApplicationContext类中实现的。

AbstractApplicationContext实现了BeanDefinitionRegistry以及BeanFactory的大部分功能，所以其子类如AnnotationConfigApplicationContext或ClassPathXmlApplicationContext只需要实现部分相关功能即可。

refresh方法代码:

```java
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // 设置一些属性值，如开始日期等，不是重点
        prepareRefresh();
        // Tell the subclass to refresh the internal bean factory.
        // 获取beanFactory，对于AnnotationConfigApplicationContext而言，其实在其父类GenericApplicationContext的构造函数中就已经创建好了，beanFactory是DefaultListableBeanFactory类的一个实例。
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
        // 配置一些必要的对象，如ClassLoader，也不是重点。
        prepareBeanFactory(beanFactory);
        try {
            // BeanFactory后置处理器回调函数，让子类有机会在创建beanFactory之后，有机会对beanFactory做额外处理。模板方法模式的使用。
            postProcessBeanFactory(beanFactory);
            // Invoke factory processors registered as beans in the context.
            // 调用BeanFactoryPostProcessor的postProcessBeanFactory方法。
            // Spring提供了BeanFactoryPostProcessor接口，用于对BeanFactory进行扩展。如果一个Bean实现了BeanFactoryPostProcessor接口的话，在这里会调用其接口方法。
            // 这个方法和上一个方法的区别在于，上一个方法是调用ApplicationContext子类的postProcessBeanFactory方法，而这个方法是调用所有BeanFactoryPostProcessor实例的postProcessBeanFactory方法。这个方法做了很多事情，是我们要重点分析的对象。
            invokeBeanFactoryPostProcessors(beanFactory);
            // Register bean processors that intercept bean creation.
            // 只是注册BeanPostProcessor，还没调用后置处理器相关的方法。如果Bean实现了BeanPostProcessor接口的话，在这里注册其后置处理器。
            // BeanPostProcessor也是Spring提供的另一个有用接口，所有的BeanPostProcessor会被添加到Spring容器列表中。在应用Bean被实例化前后，Spring会一次调用BeanPostProcessor列表中的接口方法。后边会有具体分析。
            // 所有实现了BeanFactoryPostProcessor和BeanPostProcessor接口的Bean，理论上是提供给Spring使用的，我们叫他功能性Bean。而其它Bean，如UserService，叫应用型Bean。
            registerBeanPostProcessors(beanFactory);
            // Initialize message source for this context.
            // 国际化的东西，不是重点。
            initMessageSource();
            // Initialize event multicaster for this context.
            // 我们现在不讨论事件机制，主要关注Bean的加载和创建过程，所以跳过。
            initApplicationEventMulticaster();
            // Initialize other special beans in specific context subclasses.
            // 好吧，又来一个回调方法，模板方法模式哟。
            onRefresh();
            // Check for listener beans and register them.
            // 不讨论事件机制，跳过。
            registerListeners();
            // Instantiate all remaining (non-lazy-init) singletons.
            // 这又是一个重点方法。这个方法会实例化所有非lazy-init的singleton bean，并执行与Spring Bean生命周期相关的方法。如init-method，*aware接口方法等。
            finishBeanFactoryInitialization(beanFactory);
            // Last step: publish corresponding event.
            // 与事件有关，跳过
            finishRefresh();
        }
    }
}
```

refresh方法中执行了许多方法，但其实我们如果想了解bean加载及创建，只要关注3个即可，如下:

```java
invokeBeanFactoryPostProcessors(beanFactory);
registerBeanPostProcessors(beanFactory);
finishBeanFactoryInitialization(beanFactory);
```

先看invokeBeanFactoryPostProcessors

```java
protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
    // 委托给PostProcessorRegistrationDelegate执行，调用以下方法
    PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());
// 省略其它
    }
}
```

再看PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors方法

```java
public static void invokeBeanFactoryPostProcessors(
        ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {
    if (beanFactory instanceof BeanDefinitionRegistry) {
        List<BeanDefinitionRegistryPostProcessor> currentRegistryProcessors = new ArrayList<>();
        // 先处理BeanDefinitionRegistryPostProcessor，并调用其postProcessBeanDefinitionRegistry方法。
        // BeanDefinitionRegistryPostProcessor是BeanFactoryPostProcessor的一个子接口，它允许注册更多的BeanDefinition。我们之前讲过，前面的register方法中，只注册了配置类(添加了@Configuration的类)，其它类的BeabnDifination还没有被注册进来
        String[] postProcessorNames =
                beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
        for (String ppName : postProcessorNames) {
            // 这里会得到一个ConfigurationClassPostProcessor类，这个类的作用很明显，就是要继续注册配置类中要求注册的BeanDefination，如在配置类上添加了@ComponentScan注解，那么会进行包扫描，指定包下的所有添加了@Component,@Service等注解的类全部注册进来。
            // 那么这个类是什么时候注册的呢，我们稍后会讲。
            if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
                currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
                processedBeans.add(ppName);
            }
        }
        sortPostProcessors(currentRegistryProcessors, beanFactory);
        registryProcessors.addAll(currentRegistryProcessors);
        // 在这里开始调用BeanDefinitionRegistryPostProcessor的后置处理器postProcessBeanDefinitionRegistry方法。
        invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
        currentRegistryProcessors.clear();
        // 省略大量重复代码，主要处理各种优先级的BeanFactoryPostProcessor对象。可以为BeanFactoryPostProcessor设置优先级，优先级高的对象会先被调用。
        // Now, invoke the postProcessBeanFactory callback of all processors handled so far.
        invokeBeanFactoryPostProcessors(registryProcessors, beanFactory);
        invokeBeanFactoryPostProcessors(regularPostProcessors, beanFactory);
    }
    else {
        // Invoke factory processors registered with the context instance.
        invokeBeanFactoryPostProcessors(beanFactoryPostProcessors, beanFactory);
    }
    // Do not initialize FactoryBeans here: We need to leave all regular beans
    // uninitialized to let the bean factory post-processors apply to them!
    String[] postProcessorNames =
            beanFactory.getBeanNamesForType(BeanFactoryPostProcessor.class, true, false);
    // 对所有的postProcessor进行分类，高优先级的和正常优先级的分别放在不同的列表中，高优先级的先调用其postProcessBeanFactory方法。      
    List<String> nonOrderedPostProcessorNames = new ArrayList<>();
    for (String ppName : postProcessorNames) {
        if (processedBeans.contains(ppName)) {
            // skip - already processed in first phase above
        }
        else if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
            priorityOrderedPostProcessors.add(beanFactory.getBean(ppName, BeanFactoryPostProcessor.class));
        }
        else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
            orderedPostProcessorNames.add(ppName);
        }
        else {
            nonOrderedPostProcessorNames.add(ppName);
        }
    }
    // 此处省略重复调用代码，就是对高优先级的BeanFactoryPostProcessor先执行invokeBeanFactoryPostProcessors方法，然后是低优先级，再之后是无优先级的
    List<BeanFactoryPostProcessor> nonOrderedPostProcessors = new ArrayList<>();
    for (String postProcessorName : nonOrderedPostProcessorNames) {
        // 这里这行代码，它调用了beanFactory.getBean方法。我们知道只要调用了getBean方法，如果Bean没有被加载，Spring就会创建该Bean，并对其进行相关属性复制。所以对于实现了BeanFactoryPostProcessor的Bean，在这里就已经进行实例化了。因为它要拿到当前对象，才能执行其接口方法。
        nonOrderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
    }
    // 开始进行回调。
    invokeBeanFactoryPostProcessors(nonOrderedPostProcessors, beanFactory);     
    beanFactory.clearMetadataCache();
}
```

BeanFactoryPostProcessor接口，它叫BeanFactory后置处理器。如果你的类实现了这个接口，那么在BeanFactory创建完成之后，spring会调用这个类的postProcessBeanFactory方法。这个类为用户提供了拓展BeanFactory的机会。

先看ConfigurationClassPostProcessor的注册过程。回到最初AnnotationConfigApplicationContext的构造函数中。

```java
public AnnotationConfigApplicationContext() {
    // 创建了一个AnnotatedBeanDefinitionReader对象，ConfigurationClassPostProcessor这个类就是在这里注入的。
    this.reader = new AnnotatedBeanDefinitionReader(this);
    this.scanner = new ClassPathBeanDefinitionScanner(this);
}
```

继续看AnnotatedBeanDefinitionReader的构造函数。

```java
public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry) {
    this(registry, getOrCreateEnvironment(registry));
}
public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment environment) {
    this.registry = registry;
    this.conditionEvaluator = new ConditionEvaluator(registry, environment, null);
    // 注册注解配置类的处理器。
    AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
}
```

进入AnnotationConfigUtils.registerAnnotationConfigProcessors方法。

```java
public static void registerAnnotationConfigProcessors(BeanDefinitionRegistry registry) {
    // 调用了registerAnnotationConfigProcessors方法
    registerAnnotationConfigProcessors(registry, null);
}
public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(
        BeanDefinitionRegistry registry, @Nullable Object source) {
    DefaultListableBeanFactory beanFactory = unwrapDefaultListableBeanFactory(registry);    
    Set<BeanDefinitionHolder> beanDefs = new LinkedHashSet<>(8);
    if (!registry.containsBeanDefinition(CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        // 看见没有，在这里注册了ConfigurationClassPostProcessor类。
        RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
        def.setSource(source);
        // 主要是调用了registerPostProcessor来进行注册的。
        beanDefs.add(registerPostProcessor(registry, def, CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME));
    }
    if (!registry.containsBeanDefinition(AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);
        def.setSource(source);
        // 同时也注册了另一个重要类AutowiredAnnotationBeanPostProcessor
        beanDefs.add(registerPostProcessor(registry, def, AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME));
    }
    // 其它代码就不看了。省略...
    return beanDefs;
}
```

看下registerPostProcessor，就是调用registerBeanDefinition注册一个BeanDifination。

```java
private static BeanDefinitionHolder registerPostProcessor(
        BeanDefinitionRegistry registry, RootBeanDefinition definition, String beanName) {
    definition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
    registry.registerBeanDefinition(beanName, definition);
    return new BeanDefinitionHolder(definition, beanName);
}
```

ConfigurationClassPostProcessor这个类是spring帮我们注册进来，其作用是用于注册用户自定义的类。

```java
// 接着PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors的代码分析，会调用以下方法
private static void invokeBeanDefinitionRegistryPostProcessors(
        Collection<? extends BeanDefinitionRegistryPostProcessor> postProcessors, BeanDefinitionRegistry registry) {
    for (BeanDefinitionRegistryPostProcessor postProcessor : postProcessors) {
        // 这里的postProcessor就是ConfigurationClassPostProcessor
        postProcessor.postProcessBeanDefinitionRegistry(registry);
    }
}
```

进入ConfigurationClassPostProcessor类的postProcessBeanDefinitionRegistry方法。

```java
public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
    // 开始注册Config类中定义的BeanDefinition
    processConfigBeanDefinitions(registry);
}
```

```java
public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
    List<BeanDefinitionHolder> configCandidates = new ArrayList<>();
    String[] candidateNames = registry.getBeanDefinitionNames();
    // 将配置类放在configCandidates列表中。代码省略...
    // 创建配置类的分析器，对配置类进行分析。配置类就是我们添加了@Configuration注解的类，可以有多个的。我写的demo中只有一个，叫SpringConfig。
    ConfigurationClassParser parser = new ConfigurationClassParser(
            this.metadataReaderFactory, this.problemReporter, this.environment,
            this.resourceLoader, this.componentScanBeanNameGenerator, registry);
    Set<BeanDefinitionHolder> candidates = new LinkedHashSet<>(configCandidates);
    do {
        // 开始分析配置类
        parser.parse(candidates);
        parser.validate();
        // Read the model and create bean definitions based on its content
        if (this.reader == null) {
            this.reader = new ConfigurationClassBeanDefinitionReader(
                    registry, this.sourceExtractor, this.resourceLoader, this.environment,
                    this.importBeanNameGenerator, parser.getImportRegistry());
        }
        // 加载BeanDefinition
        this.reader.loadBeanDefinitions(configClasses);
    }
    while (!candidates.isEmpty());
}
```

分析parser.parse(candidates)方法

```java
public void parse(Set<BeanDefinitionHolder> configCandidates) {
    for (BeanDefinitionHolder holder : configCandidates) {
        // 调用链往下看
        parse(bd.getBeanClassName(), holder.getBeanName());
    }
    this.deferredImportSelectorHandler.process();
}
protected final void parse(Class<?> clazz, String beanName) throws IOException {
    // 继续往下看
    processConfigurationClass(new ConfigurationClass(clazz, beanName));
}
protected void processConfigurationClass(ConfigurationClass configClass) throws IOException {   
    // Recursively process the configuration class and its superclass hierarchy.
    SourceClass sourceClass = asSourceClass(configClass);
    do {
        // 最终调到了这个方法，处理配置类
        sourceClass = doProcessConfigurationClass(configClass, sourceClass);
    }
    while (sourceClass != null);
    this.configurationClasses.put(configClass, configClass);
}
```

分析doProcessConfigurationClass方法。

```java
protected final SourceClass doProcessConfigurationClass(ConfigurationClass configClass, SourceClass sourceClass)
        throws IOException {
    // 这两个方法简单过，就是处理配置类里边的内部类和@PropertySource注解
    processMemberClasses(configClass, sourceClass);
    processPropertySource(propertySource);      
    // 处理 @ComponentScan 注解类，重点，先获取加了@ComponentScan注解的配置类
    Set<AnnotationAttributes> componentScans = AnnotationConfigUtils.attributesForRepeatable(
            sourceClass.getMetadata(), ComponentScans.class, ComponentScan.class);
    if (!componentScans.isEmpty() &&
            !this.conditionEvaluator.shouldSkip(sourceClass.getMetadata(), ConfigurationPhase.REGISTER_BEAN)) {
        for (AnnotationAttributes componentScan : componentScans) {
            // The config class is annotated with @ComponentScan -> perform the scan immediately
            // 开始扫描指定package下的所有Bean，那些加了@Component，@Service，@Repository等注解的类，都会加进来。
            Set<BeanDefinitionHolder> scannedBeanDefinitions =
                    this.componentScanParser.parse(componentScan, sourceClass.getMetadata().getClassName());
            // Check the set of scanned definitions for any further config classes and parse recursively if needed
            for (BeanDefinitionHolder holder : scannedBeanDefinitions) {
                // 如果扫描到了配置类，则递归处理，继续分析配置类。
                if (ConfigurationClassUtils.checkConfigurationClassCandidate(bdCand, this.metadataReaderFactory)) {
                    // 这个方法我们就不分析了，递归哈
                    parse(bdCand.getBeanClassName(), holder.getBeanName());
                }
            }
        }
    }
    // Process any @Import annotations
    processImports(configClass, sourceClass, getImports(sourceClass), true);        
    // Process individual @Bean methods
    // 将所有都添加了@Bean注解的方法添加到configClass中，这里没有注册，仅仅只是添加。后边有一个叫reader.loadBeanDefinitions的方法来处理它。
    Set<MethodMetadata> beanMethods = retrieveBeanMethodMetadata(sourceClass);
    for (MethodMetadata methodMetadata : beanMethods) {
        configClass.addBeanMethod(new BeanMethod(methodMetadata, configClass));
    }
    // Process default methods on interfaces
    processInterfaces(configClass, sourceClass);        
    // No superclass -> processing is complete
    return null;
}
```

分析componentScanParser.parse方法。

```java
public Set<BeanDefinitionHolder> parse(AnnotationAttributes componentScan, final String declaringClass) {
        // 创建scanner
        ClassPathBeanDefinitionScanner scanner = new ClassPathBeanDefinitionScanner(this.registry,
                componentScan.getBoolean("useDefaultFilters"), this.environment, this.resourceLoader);
        Set<String> basePackages = new LinkedHashSet<>();
        // 添加所有的package，代码省略...
        // 开始扫描
        return scanner.doScan(StringUtils.toStringArray(basePackages));
    }
```

分析scanner.doScan方法。

```java
protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
    Set<BeanDefinitionHolder> beanDefinitions = new LinkedHashSet<>();
    for (String basePackage : basePackages) {
        // 找出当前包下面的所有符合条件的类，这个方法我们就不分析了。
        Set<BeanDefinition> candidates = findCandidateComponents(basePackage);
        for (BeanDefinition candidate : candidates) {
            // 处理一些其他情况，也不说了...
            if (candidate instanceof AbstractBeanDefinition) {
                postProcessBeanDefinition((AbstractBeanDefinition) candidate, beanName);
            }
            if (candidate instanceof AnnotatedBeanDefinition) {
                AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition) candidate);
            }
            if (checkCandidate(beanName, candidate)) {
                BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
                definitionHolder =
                        AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
                beanDefinitions.add(definitionHolder);
                // 最后注册BeanDefinition，这样就将包下的所有BeanDefintion都注册完成了
                registerBeanDefinition(definitionHolder, this.registry);
            }
        }
    }
    return beanDefinitions;
}
```

到此为止，将所有的包下的BeanDefinition都注册完成了。

继续分析ConfigurationClassPostProcessor.processConfigBeanDefinitions方法。

```java
public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
    List<BeanDefinitionHolder> configCandidates = new ArrayList<>();
    String[] candidateNames = registry.getBeanDefinitionNames();
    ConfigurationClassParser parser = new ConfigurationClassParser(
            this.metadataReaderFactory, this.problemReporter, this.environment,
            this.resourceLoader, this.componentScanBeanNameGenerator, registry);
    Set<BeanDefinitionHolder> candidates = new LinkedHashSet<>(configCandidates);
    do {
        // 刚才是在这里一下向下分析的。
        parser.parse(candidates);
        parser.validate();
        // Read the model and create bean definitions based on its content
        if (this.reader == null) {
            this.reader = new ConfigurationClassBeanDefinitionReader(
                    registry, this.sourceExtractor, this.resourceLoader, this.environment,
                    this.importBeanNameGenerator, parser.getImportRegistry());
        }
        // 加载BeanDefinition
        this.reader.loadBeanDefinitions(configClasses);
    }
    while (!candidates.isEmpty());
}
```

分析reader.loadBeanDefinitions方法。

```java
private void loadBeanDefinitionsForConfigurationClass(
        ConfigurationClass configClass, TrackedConditionEvaluator trackedConditionEvaluator) {
    // 处理Imported的配置类
    if (configClass.isImported()) {
        registerBeanDefinitionForImportedConfigurationClass(configClass);
    }
    for (BeanMethod beanMethod : configClass.getBeanMethods()) {
        // 注册所有的@Bean方法上的类，重点看一下这个方法
        loadBeanDefinitionsForBeanMethod(beanMethod);
    }
    // 处理其它注解，这里就不分析了。
    loadBeanDefinitionsFromImportedResources(configClass.getImportedResources());
             loadBeanDefinitionsFromRegistrars(configClass.getImportBeanDefinitionRegistrars());
}
```

分析loadBeanDefinitionsForBeanMethod方法。

```java
private void loadBeanDefinitionsForBeanMethod(BeanMethod beanMethod) {
    ConfigurationClass configClass = beanMethod.getConfigurationClass();        
    // 一些别名处理的代码都省略了...
    //  创建BeanDefinition
    ConfigurationClassBeanDefinition beanDef = new ConfigurationClassBeanDefinition(configClass, metadata);
    // 处理initMethod和destroyMethod
    String initMethodName = bean.getString("initMethod");
    if (StringUtils.hasText(initMethodName)) {
        beanDef.setInitMethodName(initMethodName);
    }
    String destroyMethodName = bean.getString("destroyMethod");
    beanDef.setDestroyMethodName(destroyMethodName);        
    // Replace the original bean definition with the target one, if necessary
    BeanDefinition beanDefToRegister = beanDef;
    // 最后注册这个BeanDefinition
    this.registry.registerBeanDefinition(beanName, beanDefToRegister);
}
```

至此为止，所有加了@Bean注解的方法定义的BeanDefinition也都被注册了。

### 总结

AnnotationConfigApplicationContext构造函数中会创建一个AnnotatedBeanDefinitionReader实例，用于读取基于注解的BeanDefinition。在这个Reader的创建过程中，会注册ConfigurationClassPostProcessor类，ConfigurationClassPostProcessor负责加载配置类中的BeanDefinition。

之后调用register和refresh方法。

register方法中注册配置类的BeanDefinition。

refresh方法中包含很多方法，其中也包含了一些拓展方法，如postProcessBeanFactory，但我们主要关注下边3个。

```java
invokeBeanFactoryPostProcessors(beanFactory);
registerBeanPostProcessors(beanFactory);
finishBeanFactoryInitialization(beanFactory);
```

invokeBeanFactoryPostProcessor负责调用BeanFactory的后置处理方法，所有实现了BeanFactoryPostProcessor接口的bean，在这个阶段会调用其postProcessBeanFactory。

spring提供了一个重要的类ConfigurationClassPostProcessor，在这个阶段被调用，目的是分析并扫描配置类要求注入的类，并为这些类注册BeanDefinition。

BeanFactoryPostProcessor是spring提供的用于拓展BeanFactory的接口。用户可以自定义一些类并实现这个接口，这样在IoC容器初始化的过程中，这些BeanFatoryPostProcessor对象的接口方法会被执行。