---
layout: default
title: Mybatis源码(4)-Excuter框架
parent: Mybatis
nav_order: 4
---

Mybatis会将所有数据库操作转换成iBatis编程模型，通过门面类SqlSession来操作数据库，但是我们深入SqlSession源码我们会发现，SqlSession啥都没干，它将数据库操作都委托给你了Excuter，如图：

![](../../assets/images/Mybatis/attachments/Mybatis源码(4)-Excuter框架_image_0.png)

### BaseExecutor

在BaseExecutor定义了Executor的基本实现，如查询一级缓存，事务处理等不变的部分，操作数据库等变化部分由子类实现，使用了模板设计模式，下面我们来看下查询方法的源码：

```java
@Override
public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
  BoundSql boundSql = ms.getBoundSql(parameter);
  CacheKey key = createCacheKey(ms, parameter, rowBounds, boundSql);
  return query(ms, parameter, rowBounds, resultHandler, key, boundSql);
}
/**
 * 所有的查询操作最后都是由该方法来处理的
 */
@SuppressWarnings("unchecked")
@Override
public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
  ErrorContext.instance().resource(ms.getResource()).activity("executing a query").object(ms.getId());
  if (closed) {
    throw new ExecutorException("Executor was closed.");
  }
  if (queryStack == 0 && ms.isFlushCacheRequired()) {
    // 清空本地缓存
    clearLocalCache();
  }
  List<E> list;
  try {
    // 查询层次加一
    queryStack++;
    // 查询一级缓存
    list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
    if (list != null) {
      // 处理存储过程的OUT参数
      handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
    } else {
      // 缓存未命中，查询数据库
      list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
    }
  } finally {
    // 查询层次减一
    queryStack--;
  }
  if (queryStack == 0) {
    for (DeferredLoad deferredLoad : deferredLoads) {
      deferredLoad.load();
    }
    // issue #601
    deferredLoads.clear();
    if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {
      // issue #482
      clearLocalCache();
    }
  }
  return list;
}
private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
  List<E> list;
  // 添加缓存占位符
  localCache.putObject(key, EXECUTION_PLACEHOLDER);
  try {
    list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);
  } finally {
    // 删除缓存占位符
    localCache.removeObject(key);
  }
  // 将查询结果添加到本地缓存
  localCache.putObject(key, list);
  if (ms.getStatementType() == StatementType.CALLABLE) {
    // 如果是存储过程则，缓存参数
    localOutputParameterCache.putObject(key, parameter);
  }
  return list;
}
```

![](../../assets/images/Mybatis/attachments/Mybatis源码(4)-Excuter框架_image_1.png)

### SimpleExecutor

普通的执行器，Mybatis的默认使用该执行器，每次新建Statement。我们还是来看下查询方法的源码：

```java
@Override
public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
  Statement stmt = null;
  try {
    // 获取Mybatis配置类
    Configuration configuration = ms.getConfiguration();
    // 根据配置类获取StatementHandler
    StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
    // 创建Statement
    stmt = prepareStatement(handler, ms.getStatementLog());
    return handler.query(stmt, resultHandler);
  } finally {
    closeStatement(stmt);
  }
}
private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
  Statement stmt;
  // 获取Connection连接
  Connection connection = getConnection(statementLog);
  // 根据Connection获取Statement
  stmt = handler.prepare(connection, transaction.getTimeout());
  // 设置参数
  handler.parameterize(stmt);
  return stmt;
}
```

### ReuseExecutor

可以重用的执行器，复用的是Statement，内部以sql语句为key使用一个Map将Statement对象缓存起来，只要连接不断开，那么Statement就可以重用。

因为每一个新的SqlSession都有一个新的Executor对象，所以我们缓存在ReuseExecutor上的Statement的作用域是同一个SqlSession，所以其实这个缓存用处其实并不大。我们直接看下获取Statement源码，其他部分和SimpleExecutor查询方法一样。

```java
private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
  Statement stmt;
  BoundSql boundSql = handler.getBoundSql();
  String sql = boundSql.getSql();
  if (hasStatementFor(sql)) {
    // 获取复用的Statement
    stmt = getStatement(sql);
    applyTransactionTimeout(stmt);
  } else {
    // 新建Statement，并缓存
    Connection connection = getConnection(statementLog);
    stmt = handler.prepare(connection, transaction.getTimeout());
    putStatement(sql, stmt);
  }
  handler.parameterize(stmt);
  return stmt;
}
private boolean hasStatementFor(String sql) {
  try {
    // 根据sql判断是否缓存了Statement，并判断Connection是否关闭
    return statementMap.keySet().contains(sql) && !statementMap.get(sql).getConnection().isClosed();
  } catch (SQLException e) {
    return false;
  }
}
private Statement getStatement(String s) {
  return statementMap.get(s);
}
private void putStatement(String sql, Statement stmt) {
  statementMap.put(sql, stmt);
}
```

![](../../assets/images/Mybatis/attachments/Mybatis源码(4)-Excuter框架_image_2.png)

### BatchExecutor

批处理执行器，通过封装jdbc的 statement.addBatch(String sql) 以及 statement.executeBatch(); 来实现的批处理。该执行器的事务只能是手动提交模式。

我们平时执行批量的处理是一般还可以使用sql拼接的方式。

### CachingExecutor

如果开启了二级缓存那么Mybatis会使用CachingExecutor执行器，CachingExecutor使用了装饰器模式。

```java
@Override
public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql)
  throws SQLException {
  // 获取二级缓存
  Cache cache = ms.getCache();
  if (cache != null) {
    // 刷新缓存
    flushCacheIfRequired(ms);
    if (ms.isUseCache() && resultHandler == null) {
      ensureNoOutParams(ms, boundSql);
      // 查缓存
      @SuppressWarnings("unchecked")
      List<E> list = (List<E>) tcm.getObject(cache, key);
      if (list == null) {
        // 调用被装饰则的方法
        list = delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
        // 将数据放入缓存
        tcm.putObject(cache, key, list); // issue #578 and #116
      }
      return list;
    }
  }
  // 没找到缓存，直接调用被装饰则的方法
  return delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
}
```

### 总结

- BaseExecutor：使用了模板方法模式，定义了Executor的基本实现，它是一个抽象类，不能直接对外提供服务。

- SimpleExecutor：普通的执行器，Mybatis的默认使用该执行器，每次新建Statement。

- ReuseExecutor：可以重用Statement的执行器，但是这个Statement缓存只在一次SqlSession中有效，我们平时生少有在一次SqlSession中进行多次一样的查询操作，所以性能提升并不大。

- BatchExecutor：批处理执行器

- CachingExecutor：二级缓存执行器，使用装饰器模式，整个查询流程变成 了 L2 -> L1 -> DB。建议直接使用第三方缓存框架，如：为监控而生的多级缓存框架 layering-cache。