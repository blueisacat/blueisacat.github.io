---
layout: default
title: Mybatis源码(6)-Mybatis-Spring框架实现原理
parent: Mybatis
nav_order: 1.6
---

我在使用mybatis-spring过程中一直有一个疑问，在Mybatis 源码（一）总揽中我提到过，SqlSession和Mapper对象的声明周期是方法级别的，也就是每个请求的SqlSession和Mapper对象是不一样的，是一个非单例的Bean。但是与Spring集成后，为什么我们可以直接注入Mapper对象，如果通过直接注入的话Mapper对象却成了单例的了？

我们带着疑问来看下Mybatis-Spring是如何实现的。

### 初始化 SqlSessionFactory

我们是通过SqlSessionFactoryBean来完成Mybatis与Spring集成的，类图如下：

![](../../assets/images/Mybatis/attachments/Mybatis源码(6)-Mybatis-Spring框架实现原理_image_0.png)

通过类图我们发现SqlSessionFactoryBean实现了FactoryBean接口，那么在Spring实例化Bean的时候会调用FactoryBean的getObject()方法。所以Mybatis与Spring的集成的入口就是org.mybatis.spring.SqlSessionFactoryBean#getObject()方法，源码如下：

```java
public SqlSessionFactory getObject() throws Exception {
    if (this.sqlSessionFactory == null) {
        afterPropertiesSet();
    }
    return this.sqlSessionFactory;
}
```

通过跟源码发现，在afterPropertiesSet();方法中完成了sqlSessionFactory的初始化。

和Mybatis 源码（二）Mybatis 初始化中介绍的一样，还是通过XMLConfigBuilder、XMLMapperBuilder和XMLStatementBuilder三个建造者来完成了对Mybatis XML文件的解析。

### 装载映射器到Spring容器

Mybatis和Spring集成后，有三种方式将Mapper的实例装载到Spring容器，如下：

- 使用 

- 使用 @MapperScan 注解

- Spring XML 配置文件中注册一个 MapperScannerConfigurer

### MapperScannerConfigurer

![](../../assets/images/Mybatis/attachments/Mybatis源码(6)-Mybatis-Spring框架实现原理_image_1.png)

通过类图我们发现，MapperScannerConfigurer实现了BeanDefinitionRegistryPostProcessor，那么会执行BeanDefinitionRegistryPostProcessor的postProcessBeanDefinitionRegistry方法来完成Bean的装载。

```java
public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
	if (this.processPropertyPlaceHolders) {
	  processPropertyPlaceHolders();
	}
	ClassPathMapperScanner scanner = new ClassPathMapperScanner(registry);
	scanner.setAddToConfig(this.addToConfig);
	scanner.setAnnotationClass(this.annotationClass);
	scanner.setMarkerInterface(this.markerInterface);
	scanner.setSqlSessionFactory(this.sqlSessionFactory);
	scanner.setSqlSessionTemplate(this.sqlSessionTemplate);
	scanner.setSqlSessionFactoryBeanName(this.sqlSessionFactoryBeanName);
	scanner.setSqlSessionTemplateBeanName(this.sqlSessionTemplateBeanName);
	scanner.setResourceLoader(this.applicationContext);
	scanner.setBeanNameGenerator(this.nameGenerator);
	scanner.registerFilters();
	scanner.scan(StringUtils.tokenizeToStringArray(this.basePackage, ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS));
}
```

我们可以看到通过包扫描，将会扫描出所有的Mapper类，然后注册Bean定义到Spring容器。

但是Mapper是一个接口类，是不能直接进行实例化的，所以在ClassPathMapperScanner中，它将所有Mapper对象的BeanDefinition给改了，将所有Mapper的接口对象指向MapperFactoryBean工厂Bean，所以在Spring中Mybatis所有的Mapper接口对应的类是MapperFactoryBean，源码如下：

```java
@Override
public Set<BeanDefinitionHolder> doScan(String... basePackages) {
	Set<BeanDefinitionHolder> beanDefinitions = super.doScan(basePackages);
	for (BeanDefinitionHolder holder : beanDefinitions) {
		GenericBeanDefinition definition = (GenericBeanDefinition) holder.getBeanDefinition();
		definition.getPropertyValues().add("mapperInterface", definition.getBeanClassName());
		definition.setBeanClass(MapperFactoryBean.class);
		definition.getPropertyValues().add("addToConfig", this.addToConfig);
		...
        // 设置按类型注入属性，这里主要是注入sqlSessionFactory和sqlSessionTemplate
		definition.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_BY_TYPE);
	}
	return beanDefinitions;
}
```

从源码我们看出ClassPathMapperScanner主要如下的BeanDefinition：

1. 将Class指向MapperFactoryBean

1. 修改需要注入的属性值，如：addToConfig、sqlSessionFactory、sqlSessionFactory

1. 修改注入方式AbstractBeanDefinition.AUTOWIRE_BY_TYPE

通过上述修改使得Mapper接口可以实例化成对象并放到Spring容器中。

### MapperFactoryBean

![](../../assets/images/Mybatis/attachments/Mybatis源码(6)-Mybatis-Spring框架实现原理_image_2.png)

从类图我们可以看出它是一个FactoryBean，所以实例化的时候回去调用其getObject()方法完成Bean的装载，源码如下：

```java
@Override
public T getObject() throws Exception {
	return getSqlSession().getMapper(this.mapperInterface);
}
```

这里值得说一下的是，getSqlSession()获取到的是SqlSessionTemplate对象，在Mapper是单例的情况下，如何保证每次访问数据库的Sqlsession是不一样的，就是在SqlSessionTemplate中实现的。

MapperFactoryBean它还实现了InitializingBean接口，利用InitializingBean的特性，它会将Mapper接口放到Mybatis中的Configuration对象中，源码如下：

```java
@Override
public final void afterPropertiesSet() throws IllegalArgumentException, BeanInitializationException {
    // Let abstract subclasses check their configuration.
    checkDaoConfig();
    ...
}
protected void checkDaoConfig() {
    super.checkDaoConfig();
    notNull(this.mapperInterface, "Property 'mapperInterface' is required");
    Configuration configuration = getSqlSession().getConfiguration();
    if (this.addToConfig && !configuration.hasMapper(this.mapperInterface)) {
        try {
            // 将Mapper放到Mybatis的Configuration对象中
            configuration.addMapper(this.mapperInterface);
        } catch (Exception e) {
            logger.error("Error while adding the mapper '" + this.mapperInterface + "' to configuration.", e);
            throw new IllegalArgumentException(e);
        } finally {
            ErrorContext.instance().reset();
        }
    }
}
```

1. 通过上面的源码我们可以发现装载到Spring容器中的Mapper对象其实是，对应Mapper接口的代理对象MapperProxy，并且在容器中它是单例的。

1. Mapper是单例其实是没问题的，因为Mapper本身是没有共享变量的，它是一个线程安全的类，只需要保证我们每次请求数据库所用到的Sqlsession不是单例的就行。为了实现这一点，MapperProxy的SqlSession不是直接使用DefaultSqlSession，而是使用了SqlSessionTemplate。

### SqlSessionTemplate

SqlSessionTemplate使用了动态代理模式+静态代理模式，对SqlSession进行增强，每次请求数据库使用新的SqlSession放到了增强器SqlSessionInterceptor里面来实现。

```java
public class SqlSessionTemplate implements SqlSession, DisposableBean {
  private final SqlSessionFactory sqlSessionFactory;
  private final ExecutorType executorType;
  private final SqlSession sqlSessionProxy;
  private final PersistenceExceptionTranslator exceptionTranslator;
  public SqlSessionTemplate(SqlSessionFactory sqlSessionFactory, ExecutorType executorType,
                            PersistenceExceptionTranslator exceptionTranslator) {
    notNull(sqlSessionFactory, "Property 'sqlSessionFactory' is required");
    notNull(executorType, "Property 'executorType' is required");
    this.sqlSessionFactory = sqlSessionFactory;
    this.executorType = executorType;
    this.exceptionTranslator = exceptionTranslator;
    // 使用动态代理模式，对SqlSession进行增强
    this.sqlSessionProxy = (SqlSession) newProxyInstance(SqlSessionFactory.class.getClassLoader(),
      new Class[]{SqlSession.class}, new SqlSessionInterceptor());
  }
  /**
   * {@inheritDoc}
   */
  @Override
  public <T> T selectOne(String statement) {
    return this.sqlSessionProxy.selectOne(statement);
  }
}
```

### SqlSessionInterceptor

这个才是Mybatis-Spring实现原理的核心之一，在每次请求数据库的过程中它会新创建一个SqlSession，源码如下：

```java
private class SqlSessionInterceptor implements InvocationHandler {
  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    // 每次获取新的SqlSession
    SqlSession sqlSession = getSqlSession(SqlSessionTemplate.this.sqlSessionFactory,
      SqlSessionTemplate.this.executorType, SqlSessionTemplate.this.exceptionTranslator);
    try {
      Object result = method.invoke(sqlSession, args);
      if (!isSqlSessionTransactional(sqlSession, SqlSessionTemplate.this.sqlSessionFactory)) {
        // force commit even on non-dirty sessions because some databases require
        // a commit/rollback before calling close()
        // 事务提交
        sqlSession.commit(true);
      }
      return result;
    } catch (Throwable t) {
      // 异常处理
      Throwable unwrapped = unwrapThrowable(t);
      if (SqlSessionTemplate.this.exceptionTranslator != null && unwrapped instanceof PersistenceException) {
        // release the connection to avoid a deadlock if the translator is no loaded. See issue #22
        // 资源释放
        closeSqlSession(sqlSession, SqlSessionTemplate.this.sqlSessionFactory);
        sqlSession = null;
        Throwable translated = SqlSessionTemplate.this.exceptionTranslator
          .translateExceptionIfPossible((PersistenceException) unwrapped);
        if (translated != null) {
          unwrapped = translated;
        }
      }
      throw unwrapped;
    } finally {
      if (sqlSession != null) {
        // 资源释放
        closeSqlSession(sqlSession, SqlSessionTemplate.this.sqlSessionFactory);
      }
    }
  }
}
```

在这个增强器中它实现了每次请求新创建Sqlsession，并且还做了统一的资源释放，事务处理等，这使得我们在和Spring集成后，不用关心资源释放等操作，将工作重心放到业务上。

### 总结

Mybatis-Spring的实现有两个核心点：

1. 通过MapperFactoryBean巧妙的将Mapper接口对应的代理对象MapperProxy装载到了Spring容器中。

1. 通过SqlSessionTemplate，使用静态代理+动态代理模式，巧妙的实现了每次访问数据库都是用新的Sqlsession对象。