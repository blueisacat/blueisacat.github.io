---
layout: default
title: Mybatis源码(5)-数据读写
parent: Mybatis
nav_order: 1.5
---

### 数据读写的本质

不管是哪种ORM框架，数据读写其本质都是对JDBC的封装，其目的主要都是简化JDBC的开发流程，进而让开发人员更关注业务。下面是JDBC的核心流程：

1. 注册 JDBC 驱动（Class.forName("XXX");）

1. 打开连接（DriverManager.getConnection("url","name","password")）

1. 根据连接，创建 Statement（conn.prepareStatement(sql)）

1. 设置参数（stmt.setString(1, "wyf");）

1. 执行查询（stmt.executeQuery();）

1. 处理结果，结果集映射（resultSet.next()）

1. 关闭资源（finally）

Mybatis也是在对JDBC进行封装，它将注册驱动和打开连接交给了数据库连接池来负责，Mybatis支持第三方数据库连接池，也可以使用自带的数据库连接池；创建 Statement和执行查询交给了StatementHandler来负责；设置参数交给ParameterHandler来负责；最后两步交给了ResultSetHandler来负责。

### Executor内部运作过程

下面是一个简单的数据读写过程，我们在宏观上先来了解一下每一个组件在整个数据读写上的作用：

![](../../assets/images/Mybatis/attachments/Mybatis源码(5)-数据读写_image_0.png)

Mybatis的数据读写主要是通Excuter来协调StatementHandler、ParameterHandler和ResultSetHandler三个组件来实现的：

- StatementHandler：它的作用是使用数据库的Statement或PrepareStatement执行操作，启承上启下作用；

- ParameterHandler：对预编译的SQL语句进行参数设置，SQL语句中的的占位符“？”都对应BoundSql.parameterMappings集合中的一个元素，在该对象中记录了对应的参数名称以及该参数的相关属性

- ResultSetHandler：对数据库返回的结果集（ResultSet）进行封装，返回用户指定的实体类型；

### StatementHandler

#### RoutingStatementHandler

通过StatementType来创建StatementHandler，使用静态代理模式来完成方法的调用，主要起到路由作用。它是Excutor组件真正实例化的组件。

```java
public class RoutingStatementHandler implements StatementHandler {
  /**
   * 静态代理模式
   */
  private final StatementHandler delegate;
  public RoutingStatementHandler(Executor executor, MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
    // 根据{@link org.apache.ibatis.mapping.StatementType} 来创建不同的实现类 （策略模式）
    switch (ms.getStatementType()) {
      case STATEMENT:
        delegate = new SimpleStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
        break;
      case PREPARED:
        delegate = new PreparedStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
        break;
      case CALLABLE:
        delegate = new CallableStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
        break;
      default:
        throw new ExecutorException("Unknown statement type: " + ms.getStatementType());
    }
  }
  @Override
  public Statement prepare(Connection connection, Integer transactionTimeout) throws SQLException {
    return delegate.prepare(connection, transactionTimeout);
  }
...
}
```

#### BaseStatementHandler

所有子类的抽象父类，定义了初始化statement的操作顺序，由子类实现具体的实例化不同的statement（模板模式）。

```java
@Override
public Statement prepare(Connection connection, Integer transactionTimeout) throws SQLException {
  ErrorContext.instance().sql(boundSql.getSql());
  Statement statement = null;
  try {
    // 实例化Statement（由子类实现）【模板方法+策略模式】
    statement = instantiateStatement(connection);
    // 设置超时时间
    setStatementTimeout(statement, transactionTimeout);
    // 设置获取数据记录条数
    setFetchSize(statement);
    return statement;
  } catch (SQLException e) {
    closeStatement(statement);
    throw e;
  } catch (Exception e) {
    closeStatement(statement);
    throw new ExecutorException("Error preparing statement.  Cause: " + e, e);
  }
}
protected abstract Statement instantiateStatement(Connection connection) throws SQLException;
```

#### SimpleStatementHandler

使用JDBCStatement执行模式，不需要做参数处理，源码如下：

```java
@Override
protected Statement instantiateStatement(Connection connection) throws SQLException {
  // 实例化Statement
  if (mappedStatement.getResultSetType() == ResultSetType.DEFAULT) {
    return connection.createStatement();
  } else {
    return connection.createStatement(mappedStatement.getResultSetType().getValue(), ResultSet.CONCUR_READ_ONLY);
  }
}
@Override
public void parameterize(Statement statement) {
  // N/A
  // 使用Statement是直接执行sql 所以没有参数
}
```

#### PreparedStatementHandler

使用JDBCPreparedStatement预编译执行模式。

```java
@Override
protected Statement instantiateStatement(Connection connection) throws SQLException {
  // 实例化PreparedStatement
  String sql = boundSql.getSql();
  if (mappedStatement.getKeyGenerator() instanceof Jdbc3KeyGenerator) {
    String[] keyColumnNames = mappedStatement.getKeyColumns();
    if (keyColumnNames == null) {
      return connection.prepareStatement(sql, PreparedStatement.RETURN_GENERATED_KEYS);
    } else {
      return connection.prepareStatement(sql, keyColumnNames);
    }
  } else if (mappedStatement.getResultSetType() == ResultSetType.DEFAULT) {
    return connection.prepareStatement(sql);
  } else {
    return connection.prepareStatement(sql, mappedStatement.getResultSetType().getValue(), ResultSet.CONCUR_READ_ONLY);
  }
}
@Override
public void parameterize(Statement statement) throws SQLException {
  // 参数处理
  parameterHandler.setParameters((PreparedStatement) statement);
}
```

#### CallableStatementHandler

使用JDBCCallableStatement执行模式，用来调用存储过程。现在很少用。

#### ParameterHandler

主要作用是给PreparedStatement设置参数，源码如下：

```java
@Override
public void setParameters(PreparedStatement ps) {
  ErrorContext.instance().activity("setting parameters").object(mappedStatement.getParameterMap().getId());
  // 获取参数映射关系
  List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
  if (parameterMappings != null) {
    // 循环获取处理参数
    for (int i = 0; i < parameterMappings.size(); i++) {
      // 获取对应索引位的参数
      ParameterMapping parameterMapping = parameterMappings.get(i);
      if (parameterMapping.getMode() != ParameterMode.OUT) {
        Object value;
        // 获取参数名称
        String propertyName = parameterMapping.getProperty();
        // 判断是否是附加参数
        if (boundSql.hasAdditionalParameter(propertyName)) { // issue #448 ask first for additional params
          value = boundSql.getAdditionalParameter(propertyName);
        }
        // 判断是否是没有参数
        else if (parameterObject == null) {
          value = null;
        }
        // 判断参数是否有相应的 TypeHandler
        else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
          value = parameterObject;
        } else {
          // 以上都不是，通过反射获取value值
          MetaObject metaObject = configuration.newMetaObject(parameterObject);
          value = metaObject.getValue(propertyName);
        }
        // 获取参数的类型处理器
        TypeHandler typeHandler = parameterMapping.getTypeHandler();
        JdbcType jdbcType = parameterMapping.getJdbcType();
        if (value == null && jdbcType == null) {
          jdbcType = configuration.getJdbcTypeForNull();
        }
        try {
          // 根据TypeHandler设置参数
          typeHandler.setParameter(ps, i + 1, value, jdbcType);
        } catch (TypeException | SQLException e) {
          throw new TypeException("Could not set parameters for mapping: " + parameterMapping + ". Cause: " + e, e);
        }
      }
    }
  }
}
```

1. 获取参数映射关系

1. 获取参数名称

1. 根据参数名称获取参数值

1. 获取参数的类型处理器

1. 根据TypeHandler设置参数值

通过上面的流程可以发现，真正设置参数是由TypeHandler来实现的。

#### TypeHandler

Mybatis基本上提供了我们所需要用到的所有TypeHandler，当然我们也可以自己实现。TypeHandler的主要作用是：

1. 设置PreparedStatement参数值

1. 获取查询结果值

```java
public interface TypeHandler<T> {
  /**
   * 给{@link PreparedStatement}设置参数值
   *
   * @param ps        {@link PreparedStatement}
   * @param i         参数的索引位
   * @param parameter 参数值
   * @param jdbcType  参数类型
   * @throws SQLException
   */
  void setParameter(PreparedStatement ps, int i, T parameter, JdbcType jdbcType) throws SQLException;
  /**
   * 根据列名获取结果值
   *
   * @param columnName Colunm name, when configuration <code>useColumnLabel</code> is <code>false</code>
   */
  T getResult(ResultSet rs, String columnName) throws SQLException;
  /**
   * 根据索引位获取结果值
   */
  T getResult(ResultSet rs, int columnIndex) throws SQLException;
  /**
   * 获取存储过程结果值
   */
  T getResult(CallableStatement cs, int columnIndex) throws SQLException;
}
```

#### ResultSetHandler

ResultSetHandler主要作用是：对数据库返回的结果集（ResultSet）进行封装，通过通过ResultMap配置和反射完成自动映射，返回用户指定的实体类型；核心思路如下：

1. 根据RowBounds做分页处理

1. 根据ResultMap配置的返回值类型和constructor配置信息实例化目标类

1. 根据ResultMap配置的映射关系，获取到TypeHandler，进而从ResultSet中获取值

1. 根据ResultMap配置的映射关系，获取到目标类的属性名称，然后通过反射给目标类赋值

### 数据读取流程图

![](../../assets/images/Mybatis/attachments/Mybatis源码(5)-数据读写_image_1.png)

![](../../assets/images/Mybatis/attachments/Mybatis源码(5)-数据读写_image_2.png)