---
layout: default
title: K8S入门-监控与日志
parent: Kubernetes
---

### 一、监控

#### 监控类型

1. 资源监控

    比较常见的像CPU、内存、网络这种资源类的一个指标，通常这些指标会以数值、百分比的单位进行统计，是最常见的一个监控方式。这种监控方式在常规的监控里面，类似项目zabbix telegraph，这些系统都是可以做到的。

1. 性能监控

    性能监控指的是APM监控，也就是说常见的一些应用性能类的监控指标的检查。通常是通过一些Hook的机制在虚拟机层、字节码执行层通过隐式调用，或者是在应用层显示注入，或者更深层次的一个监控指标，一般是用来应用的调优和诊断的。比较常见的类似像jvm或者php的Zend Engine，通过一些常见的Hook机制，拿到类似像jvm里面的GC的次数，各种内存代的分布以及网络连接数的一些指标，通过这种方式来进行应用的性能诊断和调优。

1. 安全监控

    安全监控主要是对安全进行的一些列的监控策略，类似像越权管理、安全漏洞扫描等等。

1. 事件监控

    事件监控是K8S中比较另类的一种监控方式。K8S中的一个设计理念，就是基于状态机的一个状态转换。从正常的状态转换成另一个正常的状态的时候，会发生一个normal的事件，而从一个正常状态转换成一个异常状态的时候，会发生一个warning的事件。通常情况下，warning的事件是我们比较关心的，而事件监控就是可以把normal的事件或者是warning事件离线到一个数据中心，然后通过数据中心的分析以及报警，把相应得一些异常通过像钉钉或者短信、邮件的方式进行暴露，弥补常规监控的一些缺陷和弊端。

#### K8S的监控演进

在早期，也就是1.10以前的K8S版本。大家都会使用类似像Heapster这样的组件来去进行监控的采集，Heapster的设计原理其实也比较简单。

![](../../assets/images/Kubernetes/attachments/[K8S入门]监控与日志_image_0.png)

首先，我们在每一个K8S上面有一个包裹好的cadvisor，这个cadvisor是负责数据采集的组件。当cadvisor把数据采集完成，K8S会把cadvisor采集到的数据进行包裹，暴露成相应的API。在早期的时候，实际上是有三种不同的API：

- 第一种是summary接口

- 第二种是kubelet接口

- 第三种是prometheus接口

这三种接口，其实对应的数据源都是cadvisor，只是数据格式有缩不同。而在Heapster里面，其实支持了summary接口和kubelet两种数据采集接口，Heapster会定期去每一个节点拉取数据，在自己的内存里面进行聚合，然后再暴露相应得service，供上层的消费者进行使用。再K8S中比较常见的消费者，类似像dashboard，或者是HPA-Controller，它通过调用service获取相应得监控数据，来实现相应得弹性伸缩，以及监控数据得一个展示。

这个是以前的一个数据消费链路，这条消费链路看上去很清晰，也没有太多的一个问题，那为什么K8S会将Heapster放弃掉而转换到metrics-service呢?其实这个主要的一个动力来源是由于Heapster在做监控数据接口的标准化。为什么要做监控数据接口标准化呢?

- 第一点在于客户的需求是千变万化的，比如说今天用Heapster进行了基础数据的一个资源采集，那明天的时候，我想在应用里面暴露在线人数的一个数据接口，放到自己的接口系统里进行数据得一个展现，以及类似像HPA的一个数据消费。那这个场景在Heapster下能不能做呢?答案是不可以的，所以这就是Heapster自身拓展性的弊端

- 第二点是Heapster里面为了保证数据得离线能力，提供了很多的sink，而这个sink包含了类似像influxdb、sls、钉钉等一系列sink。这个sink主要做的是把数据采集下来，并且把这个数据离线走，然后很多客户会用influxdb做这个数据离线，在influxdb上去接入类似像grafana监控数据得一个可视化的软件，来实践监控数据得可视化。

但是后来社区发现，这些sink很多时候都是没有人来维护的。这也导致整个Heapster的项目有很多的bug，这个bug一致存留在社区里面，是没有人修复的，这个也是会给社区的项目的活跃度包括项目的稳定性带来了很多的挑战。

基于这两点原因，K8S把Heapster进行了break掉，然后做了一个精简版的监控采集组件，叫做metric-server。

![](../../assets/images/Kubernetes/attachments/[K8S入门]监控与日志_image_1.png)

上图是Heapster内部的一个架构。大家可以发现它分为几个部分，第一个部分是core部分，然后上层是一个通过标准的http或者https暴露的这个API。然后中间是source的部分，source部分相当于是采集数据暴露的不同的接口，然后processor的部分是进行数据转换以及数据聚合的部分。最后sink部分，sink部分是复杂数据离弦的，这个是早期的Heapster的一个应用的架构。那到后期的时候呢，K8S做了这个监控接口得一个标准化，主键就把Heapster进行了剪裁，转化成了metrics-server。

![](../../assets/images/Kubernetes/attachments/[K8S入门]监控与日志_image_2.png)

metrics-server大致的一个接口如上图：有一个core层、中间的source层，以及简单的API层，额外增加了API Registration这层。这层的作用就是它可以把相应得数据接口注册到K8S的API servershang，以后客户不再需要通过这个API层去访问metrics-server，而是可以通过这个API注册层，通过API server访问API注册层，再到metrics-server。这样的话，真正的数据消费方可能感知到的并不是一个metrics-server，而是说感知到的是实现了这样一个API的具体的实现，而这个实现是metrics-server。这个就是metrics-server改动最大的一个地方。

#### K8S的监控接口标准

在K8S里面针对于监控，有三种不同的接口标准。它将监控的数据消费能力进行了标准化和解耦，实现了一个于社区的融合，社区里面主要分为三类。

- 第一类Resource Metrics

    对应的接口是metrics.k8s.io，主要的实现就是metrics-server，它提供的是资源的监控，比较常见的是节点级别、pod级别、namespace级别、class级别。这类的监控指标都可以通过metrics.k8s.io这个接口获取到。

- 第二类Custom Metrics

    对应的API是custom.metrics.k8s.io，主要的实现是Prometheus。它提供的是资源监控和自定义监控，资源监控和上面的资源监控其实是有覆盖关系的，而这个自定义监控指的是：比如应用上面想暴露一个类似像在线人数，或者说调用后面的这个数据库的MySQL的慢查询。这些其实都是可以在应用层做自己的定义的，然后并通过标准的Prometheus的client，暴露出相应的metrics，然后再被Prometheus进行采集。

    而这类的接口一旦采集上来也是可以通过类似像custom.metrics.k8s.io这样一个接口的标准来进行数据消费的，也就是说现在如果以这种方式接入的Promethus，那你就可以通过custom.metrics.k8s.io这个接口来进行HPA，进行数据消费。

- 第三类External Metrics

    External Metrics 其实是比较特殊的一类，因为我们知道 K8S 现在已经成为了云原生接口的一个实现标准。很多时候在云上打交道的是云服务，比如说在一个应用里面用到了前面的是消息队列，后面的是 RBS 数据库。那有时在进行数据消费的时候，同时需要去消费一些云产品的监控指标，类似像消息队列中消息的数目，或者是接入层 SLB 的 connection 数目，SLB 上层的 200 个请求数目等等，这些监控指标。

    那怎么去消费呢？也是在 K8S 里面实现了一个标准，就是 external.metrics.k8s.io。主要的实现厂商就是各个云厂商的 provider，通过这个 provider 可以通过云资源的监控指标。在阿里云上面也实现了阿里巴巴 cloud metrics adapter 用来提供这个标准的 external.metrics.k8s.io 的一个实现。

#### Prometheus-开源社区的监控标准

接下来我们来看一个比较常见的开源社区里面的监控方案，就是 Prometheus。Prometheus 为什么说是开源社区的监控标准呢？

- 一是因为首先 Prometheus 是 CNCF 云原生社区的一个毕业项目。然后第二个是现在有越来越多的开源项目都以 Prometheus 作为监控标准，类似说我们比较常见的 Spark、Tensorflow、Flink 这些项目，其实它都有标准的 Prometheus 的采集接口。

- 第二个是对于类似像比较常见的一些数据库、中间件这类的项目，它都有相应的 Prometheus 采集客户端。类似像 ETCD、zookeeper、MySQL 或者说 PostgreSQL，这些其实都有相应的这个 Prometheus 的接口，如果没有的，社区里面也会有相应的 exporter 进行接口的一个实现。

那我们先来看一下 Prometheus 整个的大致一个结构。

![](../../assets/images/Kubernetes/attachments/[K8S入门]监控与日志_image_3.png)

上图是 Prometheus 采集的数据链路，它主要可以分为三种不同的数据采集链路。

- 第一种，是这个 push 的方式，就是通过 pushgateway 进行数据采集，然后数据线到 pushgateway，然后 Prometheus 再通过 pull 的方式去 pushgateway 去拉数据。这种采集方式主要应对的场景就是你的这个任务可能是比较短暂的，比如说我们知道 Prometheus，最常见的采集方式是拉模式，那带来一个问题就是，一旦你的数据声明周期短于数据的采集周期，比如我采集周期是 30s，而我这个任务可能运行 15s 就完了。这种场景之下，可能会造成有些数据漏采。对于这种场景最简单的一个做法就是先通过 pushgateway，先把你的 metrics push下来，然后再通过 pull 的方式从 pushgateway 去拉数据，通过这种方式可以做到，短时间的不丢作业任务。

- 第二种是标准的 pull 模式，它是直接通过拉模式去对应的数据的任务上面去拉取数据。

- 第三种是 Prometheus on Prometheus，就是可以通过另一个 Prometheus 来去同步数据到这个 Prometheus。

这是三种 Prometheus 中的采集方式。那从数据源上面，除了标准的静态配置，Prometheus 也支持 service discovery。也就是说可以通过一些服务发现的机制，动态地去发现一些采集对象。在 K8S 里面比较常见的是可以有 Kubernetes 的这种动态发现机制，只需要配置一些 annotation，它就可以自动地来配置采集任务来进行数据采集，是非常方便的。

Prometheus提供了一个外置组件叫 Alertmanager，它可以将相应的报警信息通过邮件或者短信的方式进行数据的一个告警。在数据消费上面，可以通过上层的 API clients，可以通过 web UI，可以通过 Grafana 进行数据的展现和数据的消费。

总结起来 Prometheus 有如下五个特点：

- 第一种特点就是简介强大的接入标准，开发者只需要实现 Prometheus Client 这样一个接口标准，就可以直接实现数据的一个采集

- 第二种就是多种的数据采集、离线的方式。可以通过 push 的方式、 pull 的方式、Prometheus on Prometheus的方式来进行数据的采集和离线

- 第三种就是和 K8s 的兼容

- 第四种就是丰富的插件机制与生态

- 第五种是 Prometheus Operator 的一个助力，Prometheus Operator 可能是目前我们见到的所有 Operator 里面做的最复杂的，但是它里面也是把 Prometheus 这种动态能力做到淋漓尽致的一个 Operator，如果在 K8s 里面使用 Prometheus，比较推荐大家使用 Prometheus Operator 的方式来去进行部署和运维

### 二、日志

#### 日志的场景

接下来给大家来介绍一下在 K8s 里面日志的一个部分。首先我们来看一下日志的场景，日志在 K8s 里面主要分为四个大的场景：

1. 主机内核的日志

- 第一个是主机内核的日志，主机内核日志可以协助开发者进行一些常见的问题与诊断，比如说网栈的异常，类似像我们的 iptables mark，它可以看到有 controller table 这样的一些 message

- 第二个是驱动异常，比较常见的是一些网络方案里面有的时候可能会出现驱动异常，或者说是类似 GPU 的一些场景，驱动异常可能是比较常见的一些错误

- 第三个就是文件系统异常，在早期 docker 还不是很成熟的场景之下，overlayfs 或者是 AUFS，实际上是会经常出现问题的。在这些出现问题后，开发者是没有太好的办法来去进行监控和诊断的。这一部分，其实是可以主机内核日志里面来查看到一些异常

- 再往下是影响节点的一些异常，比如说内核里面的一些 kernel panic，或者是一些 OOM，这些也会在主机日志里面有相应的一些反映

1. Runtime 的日志

    第二个是 runtime 的日志，比较常见的是 Docker 的一些日志，我们可以通过 docker 的日志来排查类似像删除一些 Pod Hang 这一系列的问题。

1. 核心组件的日志

    第三个是核心组件的日志，在 K8s 里面核心组件包含了类似像一些外置的中间件，类似像 etcd，或者像一些内置的组件，类似像 API server、kube-scheduler、controller-manger、kubelet 等等这一系列的组件。而这些组件的日志可以帮我们来看到整个 K8s 集群里面管控面的一个资源的使用量，然后以及目前运行的一个状态是否有一些异常。

    还有的就是类似像一些核心的中间件，如 Ingress 这种网络中间件，它可以帮我们来看到整个的一个接入层的一个流量，通过 Ingress 的日志，可以做到一个很好的接入层的一个应用分析。

1. 部署应用的日志

    最后是部署应用的日志，可以通过应用的日志来查看业务层的一个状态。比如说可以看业务层有没有 500 的请求？有没有一些 panic？有没有一些异常的错误的访问？那这些其实都可以通过应用日志来进行查看的。

#### 日志的采集

首先我们来看一下日志采集，从采集位置是哪个划分，需要支持如下三种：

![](../../assets/images/Kubernetes/attachments/[K8S入门]监控与日志_image_4.png)

- 首先是宿主机文件，这种场景比较常见的是说我的这个容器里面，通过类似像 volume，把日志文件写到了宿主机之上。通过宿主机的日志轮转的策略进行日志的轮转，然后再通过我的宿主机上的这个 agent 进行采集

- 第二种是容器内有日志文件，那这种常见方式怎么处理呢，比较常见的一个方式是说我通过一个 Sidecar 的 streaming 的 container，转写到 stdout，通过 stdout 写到相应的 log-file，然后再通过本地的一个日志轮转，然后以及外部的一个 agent 采集

- 第三种我们直接写到 stdout，这种比较常见的一个策略，第一种就是直接我拿这个 agent 去采集到远端，第二种我直接通过类似像一些 sls 的标准 API 采集到远端