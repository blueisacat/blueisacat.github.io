---
layout: default
title: Mybatis源码(3)-代理模块
parent: Mybatis
nav_order: 1.3
---

### Mybatis代理模块

Mybatis是在iBatis上演变而来ORM框架，所以Mybatis最终会将代码转换成iBatis编程模型，而 Mybatis 代理阶段主要是将面向接口编程模型，通过动态代理转换成ibatis编程模型。

#### 代理模块核心类

- MapperRegistry：Mapper接口动态代理工厂（MapperProxyFactory）的注册中心

- MapperProxyFactory：Mapper接口对应的动态代理工厂类。Mapper接口和MapperProxyFactory工厂类是一一对应关系

- MapperProxy：Mapper接口的增强器，它实现了InvocationHandler接口，通过该增强的invoke方法实现了对数据库的访问

- MapperMethod：对insert, update, delete, select, flush节点方法的包装类，它通过sqlSession来完成了对数据库的操作

![](../../assets/images/Mybatis/attachments/Mybatis源码(3)-代理模块_image_0.png)

#### 代理初始化

##### 加载Mapper接口到内存

当配置文件解析完成的最后一步是调用           org.apache.ibatis.builder.xml.XMLMapperBuilder#bindMapperForNamespace方法。该方法的主要作用是：根据 namespace 属性将Mapper接口的动态代理工厂（MapperProxyFactory）注册到 MapperRegistry 中。源码如下：

```java
private void bindMapperForNamespace() { 
    // 获取namespace属性（对应Mapper接口的全类名） 
    String namespace = builderAssistant.getCurrentNamespace(); 
    if (namespace != null) { 
        Class<?> boundType = null; 
        try { 
            boundType = Resources.classForName(namespace); 
        } catch (ClassNotFoundException e) {
            //ignore, bound type is not required 
        } if (boundType != null) {
            // 防止重复加载 
            if (!configuration.hasMapper(boundType)) {
                // Spring may not know the real resource name so we set a flag 
                // to prevent loading again this resource from the mapper interface 
                // look at MapperAnnotationBuilder#loadXmlResource      
                configuration.addLoadedResource("namespace:" + namespace); 
                // 将Mapper接口的动态代理工厂注册到 MapperRegistry 中 
                configuration.addMapper(boundType); 
            }
        }
    }
}
```

1. 读取namespace属性，获取Mapper接口的全类名

1. 根据全类名将Mapper接口加载到内存

1. 判断是否重复加载Mapper接口

1. 调用Mybatis 配置类（configuration）的addMapper方法，完成后续步骤

##### 注册代理工厂类

org.apache.ibatis.session.Configuration#addMapper该方法直接回去调用org.apache.ibatis.binding.MapperRegistry#addMapper方法完成注册。

```java
public <T> void addMapper(Class<T> type) {
    // 必须是接口
    if (type.isInterface()) {
        if (hasMapper(type)) {
            // 防止重复注册
            throw new BindingException("Type " + type + " is already known to the MapperRegistry.");
        }
        boolean loadCompleted = false;
        try {
            // 根据接口类，创建MapperProxyFactory代理工厂类
            knownMappers.put(type, new MapperProxyFactory<>(type));
            // It's important that the type is added before the parser is run
            // otherwise the binding may automatically be attempted by the
            // mapper parser. If the type is already known, it won't try.
            MapperAnnotationBuilder parser = new MapperAnnotationBuilder(config, type);
            parser.parse();
            loadCompleted = true;
        } finally {
            // 如果加载出现异常需要移除对应Mapper
            if (!loadCompleted) {
                knownMappers.remove(type);
            }
        }
    }
}
```

1. 判断加载类型是否是接口

1. 重复注册校验，如果校验不通抛出BindingException异常

1. 根据接口类，创建MapperProxyFactory代理工厂类

1. 如果加载出现异常需要移除对应Mapper

#### 获取代理对象

##### getMapper获取代理对象

sqlSession.getMapper(PersonMapper.class)最终调用的是org.apache.ibatis.binding.MapperRegistry#getMapper方法，最后返回的是PersonMapper 接口的代理对象，源码如下：

```java
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    // 根据类型获取对应的代理工厂
    final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
    if (mapperProxyFactory == null) {
        throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
    }
    try {
        // 根据工厂类新建一个代理对象，并返回
        return mapperProxyFactory.newInstance(sqlSession);
    } catch (Exception e) {
        throw new BindingException("Error getting mapper instance. Cause: " + e, e);
    }
}
```

1. 根据类型获取对应的代理工厂

1. 根据工厂类新建一个代理对象，并返回

##### newInstance 创建代理对象

每一个Mapper接口对应一个MapperProxyFactory工厂类。 MapperProxyFactory通过JDK动态代理创建代理对象，Mapper接口的代理对象是方法级别，所以每次访问数据库都需要新创建代理对象。源码如下：

```java
protected T newInstance(MapperProxy<T> mapperProxy) {
    // 使用JDK动态代理生成代理实例
    return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
}
public T newInstance(SqlSession sqlSession) {
    // Mapper的增强器
    final MapperProxy<T> mapperProxy = new MapperProxy<>(sqlSession, mapperInterface, methodCache);
    return newInstance(mapperProxy);
}
```

1. 先获取Mapper对应增强器(MapperProxy)

1. 根据增强器使用JDK动态代理产生代理对象

##### 代理类的反编译结果

```java
import com.sun.proxy..Proxy8;
import com.xiaolyuh.domain.model.Person;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;
public final class $Proxy8 extends Proxy implements Proxy8 {
    private static Method m3;
   ...
    public $Proxy8(InvocationHandler var1) throws  {
        super(var1);
    }
    ...
    public final Person selectByPrimaryKey(Long var1) throws  {
        try {
            return (Person)super.h.invoke(this, m3, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }
    static {
        try {
            m3 = Class.forName("com.sun.proxy.$Proxy8").getMethod("selectByPrimaryKey", Class.forName("java.lang.Long")); 
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}
```

从代理类的反编译结果来看，都是直接调用增强器的invoke方法，进而实现对数据库的访问。

#### 执行代理

通过上诉反编译代理对象，我们可以发现所有对数据库的访问都是在增强器org.apache.ibatis.binding.MapperProxy#invoke中实现的。

##### 执行增强器 MapperProxy

```java
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
  try {
    // 如果是Object本身的方法不增强
    if (Object.class.equals(method.getDeclaringClass())) {
      return method.invoke(this, args);
    }
    // 判断是否是默认方法
    else if (method.isDefault()) {
      if (privateLookupInMethod == null) {
        return invokeDefaultMethodJava8(proxy, method, args);
      } else {
        return invokeDefaultMethodJava9(proxy, method, args);
      }
    }
  } catch (Throwable t) {
    throw ExceptionUtil.unwrapThrowable(t);
  }
  // 从缓存中获取MapperMethod对象
  final MapperMethod mapperMethod = cachedMapperMethod(method);
  // 执行MapperMethod
  return mapperMethod.execute(sqlSession, args);
}
```

1. 如果是Object本身的方法不增强

1. 判断是否是默认方法

1. 从缓存中获取MapperMethod对象

1. 执行MapperMethod

##### 模型转换 MapperMethod

MapperMethod封装了Mapper接口中对应方法的信息(MethodSignature)，以及对应的sql语句的信息(SqlCommand)；它是mapper接口与映射配置文件中sql语句的桥梁； MapperMethod对象不记录任何状态信息，所以它可以在多个代理对象之间共享；

- SqlCommand ： 从configuration中获取方法的命名空间.方法名以及SQL语句的类型；

- MethodSignature：封装mapper接口方法的相关信息（入参，返回类型）；

- ParamNameResolver： 解析mapper接口方法中的入参；

```java
public Object execute(SqlSession sqlSession, Object[] args) {
  Object result;
  // 根据SQL类型，调用不同方法。
  // 这里我们可以看出，操作数据库都是通过 sqlSession 来实现的
  switch (command.getType()) {
    case INSERT: {
      Object param = method.convertArgsToSqlCommandParam(args);
      result = rowCountResult(sqlSession.insert(command.getName(), param));
      break;
    }
    case UPDATE: {
      Object param = method.convertArgsToSqlCommandParam(args);
      result = rowCountResult(sqlSession.update(command.getName(), param));
      break;
    }
    case DELETE: {
      Object param = method.convertArgsToSqlCommandParam(args);
      result = rowCountResult(sqlSession.delete(command.getName(), param));
      break;
    }
    case SELECT:
      // 根据方法返回值类型来确认调用sqlSession的哪个方法
      // 无返回值或者有结果处理器
      if (method.returnsVoid() && method.hasResultHandler()) {
        executeWithResultHandler(sqlSession, args);
        result = null;
      }
      // 返回值是否为集合类型或数组
      else if (method.returnsMany()) {
        result = executeForMany(sqlSession, args);
      }
      // 返回值是否为Map
      else if (method.returnsMap()) {
        result = executeForMap(sqlSession, args);
      }
      // 返回值是否为游标类型
      else if (method.returnsCursor()) {
        result = executeForCursor(sqlSession, args);
      }
      // 查询单条记录
      else {
        // 参数解析
        Object param = method.convertArgsToSqlCommandParam(args);
        result = sqlSession.selectOne(command.getName(), param);
        if (method.returnsOptional()
          && (result == null || !method.getReturnType().equals(result.getClass()))) {
          result = Optional.ofNullable(result);
        }
      }
      break;
    case FLUSH:
      result = sqlSession.flushStatements();
      break;
    default:
      throw new BindingException("Unknown execution method for: " + command.getName());
  }
  if (result == null && method.getReturnType().isPrimitive() && !method.returnsVoid()) {
    throw new BindingException("Mapper method '" + command.getName()
      + " attempted to return null from a method with a primitive return type (" + method.getReturnType() + ").");
  }
  return result;
}
private <E> Object executeForMany(SqlSession sqlSession, Object[] args) {
  List<E> result;
  // 将方法参数转换成SqlCommand参数
  Object param = method.convertArgsToSqlCommandParam(args);
  if (method.hasRowBounds()) {
    // 获取分页参数
    RowBounds rowBounds = method.extractRowBounds(args);
    result = sqlSession.selectList(command.getName(), param, rowBounds);
  } else {
    result = sqlSession.selectList(command.getName(), param);
  }
  // issue #510 Collections & arrays support
  if (!method.getReturnType().isAssignableFrom(result.getClass())) {
    if (method.getReturnType().isArray()) {
      return convertToArray(result);
    } else {
      return convertToDeclaredCollection(sqlSession.getConfiguration(), result);
    }
  }
  return result;
}
```

在execute方法中完成了面向接口编程模型到iBatis编程模型的转换，转换过程如下：

1. 通过MapperMethod.SqlCommand. type+MapperMethod.MethodSignature.returnType来确定需要调用SqlSession中的那个方法

1. 通过MapperMethod.SqlCommand. name来找到需要执行方法的全类名

1. 通过MapperMethod.MethodSignature.paramNameResolver来转换需要传递的参数

##### SqlSession

在Mybatis中SqlSession相当于一个门面，所有对数据库的操作都需要通过SqlSession接口，SqlSession中定义了所有对数据库的操作方法，如数据库读写命令、获取映射器、管理事务等，也是Mybatis中为数不多的有注释的类。

![](../../assets/images/Mybatis/attachments/Mybatis源码(3)-代理模块_image_1.png)

#### 流程图

![](../../assets/images/Mybatis/attachments/Mybatis源码(3)-代理模块_image_2.png)

通过上面的源码解析，可以发现Mybatis面向接口编程是通过JDK动态代理模式来实现的。主要执行流程是：

1. 在映射文件初始化完成后，将对应的Mapper接口的代理工厂类MapperProxyFactory注册到MapperRegistry

1. 每次操作数据库时，sqlSession通过MapperProxyFactory获取Mapper接口的代理类

1. 代理类通过增强器MapperProxy调用XML映射文件中SQL节点的封装类MapperMethod

1. 通过MapperMethod将Mybatis 面向接口的编程模型转换成iBatis编程模型（SqlSession模型）

1. 通过SqlSession完成对数据库的操作