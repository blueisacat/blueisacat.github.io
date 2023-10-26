---
layout: default
title: Mybatis源码(2)-初始化
parent: Mybatis
nav_order: 1.2
---

Mybatis 初始化是由SqlSessionFactoryBuilder来完成的，主要的工作解析XML文件，并将解析的类容封装到Configuration类中，最后将Configuration类封装到SqlSessionFactory中并返回，自此初始化完成。

完成对XML文件解析的是XMLConfigBuilder、XMLMapperBuilder、XMLStatementBuilder三个类来完成：

- XMLConfigBuilder：负责全局配置文件（mybatis-config.xml）中除了mappers节点的解析。

- XMLMapperBuilder：负责解析xxxMapper.xml映射文件中cache-ref、cache、parameterMap、resultMap、sql节点；根据 namespace 将Mapper接口的动态代理工厂注册到 MapperRegistry 中。

- XMLStatementBuilder：负责解析xxxMapper.xml映射文件中SQL语句节点，如：select、insert、update、delete。

- XMLScriptBuilder：负责解析SQL脚本，然后封装成SqlSource。

### Mybatis 初始化流程

#### SqlSessionFactoryBuilder

SqlSessionFactoryBuilder会将解析任务托给XMLConfigBuilder类，源码如下：

```java
public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
  try {
    // 委托给XMLConfigBuilder去解析配置文件
    XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
    // 开始解析
    return build(parser.parse());
  } 
  ...
}
public SqlSessionFactory build(Configuration config) {
  return new DefaultSqlSessionFactory(config);
}
```

#### XMLConfigBuilder

负责全局配置文件（mybatis-config.xml）中除了mappers节点的解析。解析<mappers>节点委托给XMLMapperBuilder解析器。

```java
private void parseConfiguration(XNode root) {
  try {
    //issue #117 read properties first
    // 解析<properties>节点
    propertiesElement(root.evalNode("properties"));
    // 解析<settings>节点
    Properties settings = settingsAsProperties(root.evalNode("settings"));
    loadCustomVfs(settings);
    loadCustomLogImpl(settings);
    // 解析<typeAliases>节点
    typeAliasesElement(root.evalNode("typeAliases"));
    pluginElement(root.evalNode("plugins"));
    objectFactoryElement(root.evalNode("objectFactory"));
    objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
    reflectorFactoryElement(root.evalNode("reflectorFactory"));
    settingsElement(settings);
    // read it after objectFactory and objectWrapperFactory issue #631
    environmentsElement(root.evalNode("environments"));
    databaseIdProviderElement(root.evalNode("databaseIdProvider"));
    typeHandlerElement(root.evalNode("typeHandlers"));
    // 解析<mappers>节点
    mapperElement(root.evalNode("mappers"));
  } catch (Exception e) {
    throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
  }
}
private void mapperElement(XNode parent) throws Exception {
  if (parent != null) {
    for (XNode child : parent.getChildren()) {
      if ("package".equals(child.getName())) {
        String mapperPackage = child.getStringAttribute("name");
        configuration.addMappers(mapperPackage);
      } else {
        String resource = child.getStringAttribute("resource");
        String url = child.getStringAttribute("url");
        String mapperClass = child.getStringAttribute("class");
        if (resource != null && url == null && mapperClass == null) {
          ErrorContext.instance().resource(resource);
          InputStream inputStream = Resources.getResourceAsStream(resource);
          // 委托XMLMapperBuilder来解析映射文件
          XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());
          mapperParser.parse();
        } else if (resource == null && url != null && mapperClass == null) {
          ErrorContext.instance().resource(url);
          InputStream inputStream = Resources.getUrlAsStream(url);
          // 委托XMLMapperBuilder来解析映射文件
          XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, url, configuration.getSqlFragments());
          mapperParser.parse();
        } else if (resource == null && url == null && mapperClass != null) {
          Class<?> mapperInterface = Resources.classForName(mapperClass);
          configuration.addMapper(mapperInterface);
        } else {
          throw new BuilderException("A mapper element may only specify a url, resource or class, but not more than one.");
        }
      }
    }
  }
}
```

#### XMLMapperBuilder

负责解析xxxMapper.xml映射文件中cache-ref、cache、parameterMap、resultMap、sql节点；根据 namespace 将Mapper接口的动态代理工厂（MapperProxyFactory）注册到 MapperRegistry 中。

```java
public void parse() {
  // 防止重复解析
  if (!configuration.isResourceLoaded(resource)) {
    // 解析xxxMapper.xml 映射文件中的 mapper节点
    configurationElement(parser.evalNode("/mapper"));
    // 标记为已经加载过
    configuration.addLoadedResource(resource);
    // 根据 namespace 将Mapper接口的动态代理工厂注册到 MapperRegistry 中
    bindMapperForNamespace();
  }
  // 兜底
  parsePendingResultMaps();
  parsePendingCacheRefs();
  parsePendingStatements();
}
private void configurationElement(XNode context) {
  try {
    String namespace = context.getStringAttribute("namespace");
    if (namespace == null || namespace.equals("")) {
      throw new BuilderException("Mapper's namespace cannot be empty");
    }
    builderAssistant.setCurrentNamespace(namespace);
    // 解析<cache-ref>节点
    cacheRefElement(context.evalNode("cache-ref"));
    // 解析<cache>节点
    cacheElement(context.evalNode("cache"));
    // 解析<parameterMap>节点
    parameterMapElement(context.evalNodes("/mapper/parameterMap"));
    // 解析<resultMap>节点
    resultMapElements(context.evalNodes("/mapper/resultMap"));
    sqlElement(context.evalNodes("/mapper/sql"));
    // 解析<select|insert|update|delete>节点
    buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
  } catch (Exception e) {
    throw new BuilderException("Error parsing Mapper XML. The XML location is '" + resource + "'. Cause: " + e, e);
  }
}
private void buildStatementFromContext(List<XNode> list) {
  if (configuration.getDatabaseId() != null) {
    buildStatementFromContext(list, configuration.getDatabaseId());
  }
  buildStatementFromContext(list, null);
}
private void buildStatementFromContext(List<XNode> list, String requiredDatabaseId) {
  for (XNode context : list) {
    // 解析<select|insert|update|delete>节点委托给 XMLStatementBuilder
    final XMLStatementBuilder statementParser = new XMLStatementBuilder(configuration, builderAssistant, context, requiredDatabaseId);
    try {
      statementParser.parseStatementNode();
    } catch (IncompleteElementException e) {
      configuration.addIncompleteStatement(statementParser);
    }
  }
}
```

#### XMLStatementBuilder

负责解析xxxMapper.xml映射文件中SQL语句节点，如：select、insert、update、delete。解析<select|insert|update|delete>节点委托给XMLStatementBuilder解析器。

```java
public void parseStatementNode() {
  String id = context.getStringAttribute("id");
  String databaseId = context.getStringAttribute("databaseId");
  if (!databaseIdMatchesCurrent(id, databaseId, this.requiredDatabaseId)) {
    return;
  }
  // 根据节点的名称来判断SQL语句的类型（select|insert|update|delete）
  String nodeName = context.getNode().getNodeName();
  SqlCommandType sqlCommandType = SqlCommandType.valueOf(nodeName.toUpperCase(Locale.ENGLISH));
  boolean isSelect = sqlCommandType == SqlCommandType.SELECT;
  boolean flushCache = context.getBooleanAttribute("flushCache", !isSelect);
  boolean useCache = context.getBooleanAttribute("useCache", isSelect);
  boolean resultOrdered = context.getBooleanAttribute("resultOrdered", false);
  // Include Fragments before parsing
  // 解析<include>节点
  XMLIncludeTransformer includeParser = new XMLIncludeTransformer(configuration, builderAssistant);
  includeParser.applyIncludes(context.getNode());
  String parameterType = context.getStringAttribute("parameterType");
  Class<?> parameterTypeClass = resolveClass(parameterType);
  String lang = context.getStringAttribute("lang");
  LanguageDriver langDriver = getLanguageDriver(lang);
  // Parse selectKey after includes and remove them.
  // 解析<selectKey>节点，并在XML中删除<selectKey>节点
  processSelectKeyNodes(id, parameterTypeClass, langDriver);
  // Parse the SQL (pre: <selectKey> and <include> were parsed and removed)
  KeyGenerator keyGenerator;
  String keyStatementId = id + SelectKeyGenerator.SELECT_KEY_SUFFIX;
  keyStatementId = builderAssistant.applyCurrentNamespace(keyStatementId, true);
  if (configuration.hasKeyGenerator(keyStatementId)) {
    keyGenerator = configuration.getKeyGenerator(keyStatementId);
  } else {
    keyGenerator = context.getBooleanAttribute("useGeneratedKeys",
        configuration.isUseGeneratedKeys() && SqlCommandType.INSERT.equals(sqlCommandType))
        ? Jdbc3KeyGenerator.INSTANCE : NoKeyGenerator.INSTANCE;
  }
  // 解析SQL语句，然后封装成SqlSource
  SqlSource sqlSource = langDriver.createSqlSource(configuration, context, parameterTypeClass);
  StatementType statementType = StatementType.valueOf(context.getStringAttribute("statementType", StatementType.PREPARED.toString()));
  Integer fetchSize = context.getIntAttribute("fetchSize");
  Integer timeout = context.getIntAttribute("timeout");
  String parameterMap = context.getStringAttribute("parameterMap");
  String resultType = context.getStringAttribute("resultType");
  Class<?> resultTypeClass = resolveClass(resultType);
  String resultMap = context.getStringAttribute("resultMap");
  String resultSetType = context.getStringAttribute("resultSetType");
  ResultSetType resultSetTypeEnum = resolveResultSetType(resultSetType);
  if (resultSetTypeEnum == null) {
    resultSetTypeEnum = configuration.getDefaultResultSetType();
  }
  String keyProperty = context.getStringAttribute("keyProperty");
  String keyColumn = context.getStringAttribute("keyColumn");
  String resultSets = context.getStringAttribute("resultSets");
  // 使用建造者模式构建MappedStatement对象
  builderAssistant.addMappedStatement(id, sqlSource, statementType, sqlCommandType,
      fetchSize, timeout, parameterMap, parameterTypeClass, resultMap, resultTypeClass,
      resultSetTypeEnum, flushCache, useCache, resultOrdered,
      keyGenerator, keyProperty, keyColumn, databaseId, langDriver, resultSets);
}
```

#### Mybatis 初始化流程图

![](../../assets/images/Mybatis/attachments/Mybatis源码(2)-初始化_image_0.png)

### 核心数据结构类

#### Configuration

Configuration其实就是XML配置文件的Java形态，Configuration是单例的，生命周期是应用级的；

![](../../assets/images/Mybatis/attachments/Mybatis源码(2)-初始化_image_1.png)

#### ResultMap

对应xxxMapper.xml映射文件中的resultMap节点。<resultMap>节点中的子节点使用ResultMapping来封装，如：<id>、<result>等节点。

![](../../assets/images/Mybatis/attachments/Mybatis源码(2)-初始化_image_2.png)

#### MappedStatement

对应xxxMapper.xml映射文件中的<select>、<insert>、<update>和<delete>节点。

![](../../assets/images/Mybatis/attachments/Mybatis源码(2)-初始化_image_3.png)

#### SqlSource

对应xxxMapper.xml映射文件中的sql语句，经过解析SqlSource包含的语句最终仅仅包含？占位符，可以直接提交给数据库执行；

#### MapperRegistry

它是Mapper接口动态代理工厂类的注册中心。在MyBatis中，通过MapperProxy实现InvocationHandler接口，通过MapperProxyFactory生成动态代理的实例对象；