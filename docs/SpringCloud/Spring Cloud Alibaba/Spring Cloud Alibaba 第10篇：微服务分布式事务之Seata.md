---
layout: default
title: Spring Cloud Alibaba 第10篇：微服务分布式事务之Seata
parent: SpringCloudAlibaba
nav_order: 1.10
---

# 1. 概述

在构建微服务的过程中，不管是使用什么框架、组件来构建，都绕不开一个问题，跨服务的业务操作如何保持数据一致性。

# 2. 什么是分布式事务？

首先，设想一个传统的单体应用，无论多少内部调用，最后终归是在同一个数据库上进行操作来完成一向业务操作，如图：

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第10篇：微服务分布式事务之Seata_image_0.png)

随着业务量的发展，业务需求和架构发生了巨大的变化，整体架构由原来的单体应用逐渐拆分成为了微服务，原来的3个服务被从一个单体架构上拆开了，成为了3个独立的服务，分别使用独立的数据源，也不在之前共享同一个数据源了，具体的业务将由三个服务的调用来完成，如图：

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第10篇：微服务分布式事务之Seata_image_1.png)

此时，每一个服务的内部数据一致性仍然有本地事务来保证。但是面对整个业务流程上的事务应该如何保证呢？这就是在微服务架构下面临的挑战，如何保证在微服务中的数据一致性。

# 3. 常见的分布式事务解决方案

## 3.1 两阶段提交方案/XA方案

所谓的 XA 方案，即两阶段提交，有一个事务管理器的概念，负责协调多个数据库（资源管理器）的事务，事务管理器先问问各个数据库你准备好了吗？如果每个数据库都回复 ok，那么就正式提交事务，在各个数据库上执行操作；如果任何其中一个数据库回答不 ok，那么就回滚事务。

分布式系统的一个难点是如何保证架构下多个节点在进行事务性操作的时候保持一致性。为实现这个目的，二阶段提交算法的成立基于以下假设：

- 该分布式系统中，存在一个节点作为协调者(Coordinator)，其他节点作为参与者(Cohorts)。且节点之间可以进行网络通信。

- 所有节点都采用预写式日志，且日志被写入后即被保持在可靠的存储设备上，即使节点损坏不会导致日志数据的消失。

- 所有节点不会永久性损坏，即使损坏后仍然可以恢复。

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第10篇：微服务分布式事务之Seata_image_2.png)

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第10篇：微服务分布式事务之Seata_image_3.png)

## 3.2 TCC 方案

TCC的全称是：Try、Confirm、Cancel。

- Try 阶段：这个阶段说的是对各个服务的资源做检测以及对资源进行锁定或者预留。

- Confirm 阶段：这个阶段说的是在各个服务中执行实际的操作。

- Cancel 阶段：如果任何一个服务的业务方法执行出错，那么这里就需要进行补偿，就是执行已经执行成功的业务逻辑的回滚操作。（把那些执行成功的回滚）

这种方案说实话几乎很少人使用，但是也有使用的场景。因为这个事务回滚实际上是严重依赖于你自己写代码来回滚和补偿了，会造成补偿代码巨大。

TCC的理论有点抽象，下面我们借助一个账务拆分这个实际业务场景对TCC事务的流程做一个描述，希望对理解TCC有所帮助。

业务流程：分别位于三个不同分库的帐户A、B、C，A和B一起向C转帐共80元：

Try：尝试执行业务。

完成所有业务检查(一致性)：检查A、B、C的帐户状态是否正常，帐户A的余额是否不少于30元，帐户B的余额是否不少于50元。

预留必须业务资源(准隔离性)：帐户A的冻结金额增加30元，帐户B的冻结金额增加50元，这样就保证不会出现其他并发进程扣减了这两个帐户的余额而导致在后续的真正转帐操作过程中，帐户A和B的可用余额不够的情况。

Confirm：确认执行业务。

真正执行业务：如果Try阶段帐户A、B、C状态正常，且帐户A、B余额够用，则执行帐户A给账户C转账30元、帐户B给账户C转账50元的转帐操作。

不做任何业务检查：这时已经不需要做业务检查，Try阶段已经完成了业务检查。

只使用Try阶段预留的业务资源：只需要使用Try阶段帐户A和帐户B冻结的金额即可。

Cancel：取消执行业务。

释放Try阶段预留的业务资源：如果Try阶段部分成功，比如帐户A的余额够用，且冻结相应金额成功，帐户B的余额不够而冻结失败，则需要对帐户A做Cancel操作，将帐户A被冻结的金额解冻掉。

# 4. Spring Cloud Alibaba Seata

Seata 的方案其实一个 XA 两阶段提交的改进版，具体区别如下：

架构的层面：

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第10篇：微服务分布式事务之Seata_image_4.png)

XA 方案的 RM 实际上是在数据库层，RM 本质上就是数据库自身（通过提供支持 XA 的驱动程序来供应用使用）。

而 Seata 的 RM 是以二方包的形式作为中间件层部署在应用程序这一侧的，不依赖与数据库本身对协议的支持，当然也不需要数据库支持 XA 协议。这点对于微服务化的架构来说是非常重要的：应用层不需要为本地事务和分布式事务两类不同场景来适配两套不同的数据库驱动。

这个设计，剥离了分布式事务方案对数据库在 协议支持 上的要求。

两阶段提交：

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第10篇：微服务分布式事务之Seata_image_5.png)

无论 Phase2 的决议是 commit 还是 rollback，事务性资源的锁都要保持到 Phase2 完成才释放。

设想一个正常运行的业务，大概率是 90% 以上的事务最终应该是成功提交的，我们是否可以在 Phase1 就将本地事务提交呢？这样 90% 以上的情况下，可以省去 Phase2 持锁的时间，整体提高效率。

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第10篇：微服务分布式事务之Seata_image_6.png)

- 分支事务中数据的 本地锁 由本地事务管理，在分支事务 Phase1 结束时释放。

- 同时，随着本地事务结束，连接 也得以释放。

- 分支事务中数据的 全局锁 在事务协调器侧管理，在决议 Phase2 全局提交时，全局锁马上可以释放。只有在决议全局回滚的情况下，全局锁 才被持有至分支的 Phase2 结束。

这个设计，极大地减少了分支事务对资源（数据和连接）的锁定时间，给整体并发和吞吐的提升提供了基础。

# 5. Seata实战案例

## 5.1 目标介绍

在本节，我们将通过一个实战案例来具体介绍Seata的使用方式，我们将模拟一个简单的用户购买商品下单场景，创建3个子工程，分别是 order-server （下单服务）、storage-server（库存服务）和 pay-server （支付服务），具体流程图如图：

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第10篇：微服务分布式事务之Seata_image_7.png)

## 5.2 环境准备

在本次实战中，我们使用Nacos做为服务中心和配置中心，Nacos部署请参考本书的第十一章，这里不再赘述。

接下来我们需要部署Seata的Server端，下载地址为：[https://github.com/seata/seata/releases](https://github.com/seata/seata/releases) ，建议选择最新版本下载，目前笔者看到的最新版本为 v0.8.0 ，下载 seata-server-0.8.0.tar.gz 解压后，打开 conf 文件夹，我们需对其中的一些配置做出修改。

### 5.2.1 registry.conf 文件修改，如下

```
registry {
    type = "nacos"
    nacos {
    serverAddr = "192.168.0.128"
    namespace = "public"
    cluster = "default"
    }
}
config {
    type = "nacos"
    nacos {
    serverAddr = "192.168.0.128"
    namespace = "public"
    cluster = "default"
    }
}
```

这里我们选择使用Nacos作为服务中心和配置中心，这里做出对应的配置，同时可以看到Seata的注册服务支持：file 、nacos 、eureka、redis、zk、consul、etcd3、sofa等方式，配置支持：file、nacos 、apollo、zk、consul、etcd3等方式。

### 5.2.2 file.conf 文件修改

这里我们需要其中配置的数据库相关配置，具体如下：

```
## database store
db {
    ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp) etc.
    datasource = "dbcp"
    ## mysql/oracle/h2/oceanbase etc.
    db-type = "mysql"
    driver-class-name = "com.mysql.jdbc.Driver"
    url = "jdbc:mysql://192.168.0.128:3306/seata"
    user = "root"
    password = "123456"
    min-conn = 1
    max-conn = 3
    global.table = "global_table"
    branch.table = "branch_table"
    lock-table = "lock_table"
    query-limit = 100
}
```

这里数据库默认是使用mysql，需要配置对应的数据库连接、用户名和密码等。

### 5.2.3 nacos-config.txt 文件修改，具体如下

```
service.vgroup_mapping.spring-cloud-pay-server=default
service.vgroup_mapping.spring-cloud-order-server=default
service.vgroup_mapping.spring-cloud-storage-server=default
```

这里的语法为：service.vgroup_mapping.${your-service-gruop}=default ，中间的${your-service-gruop}为自己定义的服务组名称，这里需要我们在程序的配置文件中配置，笔者这里直接使用程序的spring.application.name。

### 5.2.4 数据库初始化

需要在刚才配置的数据库中执行数据初始脚本 db_store.sql ，这个是全局事务控制的表，需要提前初始化。

这里我们只是做演示，理论上上面三个业务服务应该分属不同的数据库，这里我们只是在同一台数据库下面创建三个 Schema ，分别为 db_account 、 db_order 和 db_storage ，具体如图：

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第10篇：微服务分布式事务之Seata_image_8.png)

### 5.2.5 服务启动

因为我们是使用的Nacos作为配置中心，所以这里需要先执行脚本来初始化Nacos的相关配置，命令如下：

```
cd conf
sh nacos-config.sh 192.168.0.128
```

执行成功后可以打开Nacos的控制台，在配置列表中，可以看到初始化了很多 Group 为 SEATA_GROUP 的配置，如图：

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第10篇：微服务分布式事务之Seata_image_9.png)

初始化成功后，可以使用下面的命令启动Seata的Server端：

```
cd bin
sh seata-server.sh -p 8091 -m file
```

启动后在 Nacos 的服务列表下面可以看到一个名为 serverAddr 的服务

到这里，我们的环境准备工作就做完了，接下来开始代码实战。

## 5.3 代码实战

由于本示例代码偏多，这里仅介绍核心代码和一些需要注意的代码，其余代码各位读者可以访问本书配套的代码仓库获取。子工程common用来放置一些公共类，主要包含视图 VO 类和响应类 OperationResponse.java。

### 5.3.1 父工程 seata-nacos-jpa 依赖 pom.xml 文件

```
  <dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- Spring Cloud Nacos Service Discovery -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    <!-- Spring Cloud Nacos Config -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    </dependency>
    <!-- Spring Cloud Seata -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>${spring-cloud-alibaba.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

说明：本示例是使用 JPA 作为数据库访问 ORM 层， Mysql 作为数据库，需引入 JPA 和 Mysql 相关依赖， spring-cloud-alibaba-dependencies 的版本是 2.1.0.RELEASE ， 其中有关Seata的组件版本为 v0.7.1 ，虽然和服务端版本不符，经简单测试，未发现问题。

### 5.3.2 数据源配置

Seata 是通过代理数据源实现事务分支，所以需要配置 io.seata.rm.datasource.DataSourceProxy 的 Bean，且是 @Primary默认的数据源，否则事务不会回滚，无法实现分布式事务，数据源配置类DataSourceProxyConfig.java如下：

```
@Configuration
public class DataSourceProxyConfig {
    @Bean
    @ConfigurationProperties(prefix = "spring.datasource")
    public DruidDataSource druidDataSource() {
        return new DruidDataSource();
    }
    @Primary
    @Bean
    public DataSourceProxy dataSource(DruidDataSource druidDataSource) {
        return new DataSourceProxy(druidDataSource);
    }
}
```

### 5.3.3 开启全局事务

我们在order-server服务中开始整个业务流程，需要在这里的方法上增加全局事务的注解@GlobalTransactional，具体代码如下：

```
@Service
@Slf4j
public class OrderServiceImpl implements OrderService {
    @Autowired
    private RestTemplate restTemplate;
    @Autowired
    private OrderDao orderDao;
    private final String STORAGE_SERVICE_HOST = "
    private final String PAY_SERVICE_HOST = "
    @Override
    @GlobalTransactional
    public OperationResponse placeOrder(PlaceOrderRequestVO placeOrderRequestVO) {
        Integer amount = 1;
        Integer price = placeOrderRequestVO.getPrice();
        Order order = Order.builder()
                .userId(placeOrderRequestVO.getUserId())
                .productId(placeOrderRequestVO.getProductId())
                .status(OrderStatus.INIT)
                .payAmount(price)
                .build();
        order = orderDao.save(order);
        log.info("保存订单{}", order.getId() != null ? "成功" : "失败");
        log.info("当前 XID: {}", RootContext.getXID());
        // 扣减库存
        log.info("开始扣减库存");
        ReduceStockRequestVO reduceStockRequestVO = ReduceStockRequestVO.builder()
                .productId(placeOrderRequestVO.getProductId())
                .amount(amount)
                .build();
        String storageReduceUrl = String.format("%s/reduceStock", STORAGE_SERVICE_HOST);
        OperationResponse storageOperationResponse = restTemplate.postForObject(storageReduceUrl, reduceStockRequestVO, OperationResponse.class);
        log.info("扣减库存结果:{}", storageOperationResponse);
        // 扣减余额
        log.info("开始扣减余额");
        ReduceBalanceRequestVO reduceBalanceRequestVO = ReduceBalanceRequestVO.builder()
                .userId(placeOrderRequestVO.getUserId())
                .price(price)
                .build();
        String reduceBalanceUrl = String.format("%s/reduceBalance", PAY_SERVICE_HOST);
        OperationResponse balanceOperationResponse = restTemplate.postForObject(reduceBalanceUrl, reduceBalanceRequestVO, OperationResponse.class);
        log.info("扣减余额结果:{}", balanceOperationResponse);
        Integer updateOrderRecord = orderDao.updateOrder(order.getId(), OrderStatus.SUCCESS);
        log.info("更新订单:{} {}", order.getId(), updateOrderRecord > 0 ? "成功" : "失败");
        return OperationResponse.builder()
                .success(balanceOperationResponse.isSuccess() && storageOperationResponse.isSuccess())
                .build();
    }
}
```

其次，我们需要在另外两个服务的方法中增加注解@Transactional，表示开启事务。

这里的远端服务调用是通过 RestTemplate ，需要在工程启动时将 RestTemplate 注入 Spring 容器中管理。

### 5.3.4 配置文件

工程中需在 resources 目录下增加有关Seata的配置文件 registry.conf ，如下：

```
registry {
  type = "nacos"
  nacos {
    serverAddr = "192.168.0.128"
    namespace = "public"
    cluster = "default"
  }
}
config {
  type = "nacos"
  nacos {
    serverAddr = "192.168.0.128"
    namespace = "public"
    cluster = "default"
  }
}
```

在 bootstrap.yml 中的配置如下：

```
spring:
  application:
    name: spring-cloud-order-server
  cloud:
    nacos:
      # nacos config
      config:
        server-addr: 192.168.0.128
        namespace: public
        group: SEATA_GROUP
      # nacos discovery
      discovery:
        server-addr: 192.168.0.128
        namespace: public
        enabled: true
    alibaba:
      seata:
        tx-service-group: ${spring.application.name}
```

- spring.cloud.nacos.config.group ：这里的 Group 是 SEATA_GROUP ，也就是我们前面在使用 nacos-config.sh 生成 Nacos 的配置时生成的配置，它的 Group 是 SEATA_GROUP。

- spring.cloud.alibaba.seata.tx-service-group ：这里是我们之前在修改 Seata Server 端配置文件 nacos-config.txt 时里面配置的 service.vgroup_mapping.${your-service-gruop}=default 中间的 ${your-service-gruop} 。这两处配置请务必一致，否则在启动工程后会一直报错 no available server to connect 。

### 5.3.5 业务数据库初始化

数据库初始脚本位于：Alibaba/seata-nacos-jpa/sql ，请分别在三个不同的 Schema 中执行。

### 5.3.6 测试

测试工具我们选择使用 PostMan ，启动三个服务，顺序无关 order-server、pay-server 和 storage-server 。

使用 PostMan 发送测试请求，如图：

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第10篇：微服务分布式事务之Seata_image_10.png)

数据库初始化余额为 10 ，这里每次下单将会消耗 5 ，我们可以正常下单两次，第三次应该下单失败，并且回滚 db_order 中的数据。数据库中数据如图：

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第10篇：微服务分布式事务之Seata_image_11.png)

我们进行第三次下单操作，如图：

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第10篇：微服务分布式事务之Seata_image_12.png)

这里看到直接报错500，查看数据库 db_order 中的数据，如图：

![](../../../assets/images/SpringCloud/Spring Cloud Alibaba/attachments/Spring%20Cloud%20Alibaba%20第10篇：微服务分布式事务之Seata_image_13.png)

可以看到，这里的数据并未增加，我们看下子工程_rder-server的控制台打印：

日志已经过简化处理

```
Hibernate: insert into orders (pay_amount, product_id, status, user_id) values (?, ?, ?, ?)
c.s.b.c.service.impl.OrderServiceImpl    : 保存订单成功
c.s.b.c.service.impl.OrderServiceImpl    : 当前 XID: 192.168.0.102:8091:2021674307
c.s.b.c.service.impl.OrderServiceImpl    : 开始扣减库存
c.s.b.c.service.impl.OrderServiceImpl    : 扣减库存结果:OperationResponse(success=true, message=操作成功, data=null)
c.s.b.c.service.impl.OrderServiceImpl    : 开始扣减余额
i.s.core.rpc.netty.RmMessageListener     : onMessage:xid=192.168.0.102:8091:2021674307,branchId=2021674308,branchType=AT,resourceId=jdbc:mysql://192.168.0.128:3306/db_order,applicationData=null
io.seata.rm.AbstractRMHandler            : Branch Rollbacking: 192.168.0.102:8091:2021674307 2021674308 jdbc:mysql://192.168.0.128:3306/db_order
i.s.rm.datasource.undo.UndoLogManager    : xid 192.168.0.102:8091:2021674307 branch 2021674308, undo_log deleted with GlobalFinished
io.seata.rm.AbstractRMHandler            : Branch Rollbacked result: PhaseTwo_Rollbacked
i.seata.tm.api.DefaultGlobalTransaction  : [192.168.0.102:8091:2021674307] rollback status:Rollbacked
```

从日志中没有可以清楚的看到，在服务order-server是先执行了订单写入操作，并且调用扣减库存的接口，通过查看storage-server的日志也可以发现，一样是先执行了库存修改操作，直到扣减余额的时候发现余额不足，开始对 xid 为 192.168.0.102:8091:2021674307 执行回滚操作，并且这个操作是全局回滚。

# 6. 注意

目前在 Seata v0.8.0 的版本中，Server端尚未支持集群部署，不建议应用于生产环境，并且开源团队计划在 v1.0.0 版本的时候可以使用与生产环境，各位读者可以持续关注这个开源项目。