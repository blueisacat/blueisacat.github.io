### 整体架构

![](../../assets/images/Mybatis/attachments/Mybatis源码(1)-总览_image_0.png)

这只是MySql的一个逻辑划分架构。

- 接口层：通SqlSession类提供对数据库访问能力，隐藏了后续复杂的处理逻辑。

- 核心处理层：主要负责执行SQL，并返回结果。

- 基础支撑层：对一些基础功能进行封装，为核心处理层提供服务。

### 代码结构

![](../../assets/images/Mybatis/attachments/Mybatis源码(1)-总览_image_1.png)

Mybatis的代码结构非常工整，堪称完美的java编程规范教科书，当我们深入源码我们会发现，Mybatis的注释量相当少，那是因为基本上我们可以通过名称就能明白其中的含义。

### Mybatis中的设计模式

如果想学习设计模式在代码中的应用，阅读Mybatis源码也是一个不错的选择，如：

- SqlSession使用门面模式

- 日志模块使用了适配器模式

- 数据源模块使用工厂模式

- 数据连接池使用策略模式

- 缓存模块使用了装饰器模式

- Executor模块使用了模板方法模式

- Builder模块使用了建造者模式

- Mapper接口使用了代理模式

- 插件模块使用责任链模式

### Mybatis执行流程

![](../../assets/images/Mybatis/attachments/Mybatis源码(1)-总览_image_2.png)

1. new SqlSessionFactoryBuilder().build(inputStream);读取mybatis配置文件构建SqlSessionFactory 。

1. sqlSessionFactory.openSession();获取sqlSession资源

1. sqlSession.getMapper(PersonMapper.class);获取对应mapper

1. mapper.selectByPrimaryKey(1L);执行查询语句并返回结果

1. 关闭资源

上图是Mybatis的执行流程，由此我们可以看出Mybatis的核心类有4个，分别是SqlSessionFactoryBuilder、SqlSessionFactory、SqlSession、SQL Mapper。

- SqlSessionFactoryBuilde：读取配置信息(XML文件)，创建SqlSessionFactory，建造者模式，方法级别生命周期；

- SqlSessionFactory：创建Sqlsession，工厂单例模式，存在于程序的整个应用程序生命周期；

- SqlSession：代表一次数据库连接，可以直接发送SQL执行，也可以通过调用Mapper访问数据库；线程不安全，要保证线程独享，方法级生命周期；

- SQL Mapper：由一个Java接口和XML文件组成，包含了要执行的SQL语句和结果集映射规则。方法级别生命周期；

### Mybatis核心流程三大阶段

从上面的执行流程可以看出，Mybatis核心流程主要分为以下三个阶段：

- 初始化阶段：读取XML配置文件和注解中的配置信息，创建配置对象，并完成各个模块的初始化的工作；

- 代理阶段：封装iBatis的编程模型，使用mapper接口开发的初始化工作；

- 数据读写阶段：通过SqlSession完成SQL的解析，参数的映射、SQL的执行、结果的解析过程；