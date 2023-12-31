---
layout: default
title: Sentinel原理（2）：重要概念
parent: Sentinel
nav_order: 8
---

# Sentinel原理：重要概念

Sentinel 中有很多比较重要的概念，我们要了解一个框架，首先要对框架中重要的概念实体进行分析，本文我将跟大家一起来分析一下 Sentinel 中非常重要的几个概念。

## Resource

Resource 是 Sentinel 中最重要的一个概念， Sentinel 通过资源来保护具体的业务代码或其他后方服务。 Sentinel 把复杂的逻辑给屏蔽掉了，用户只需要为受保护的代码或服务定义一个资源，然后定义规则就可以了，剩下的通通交给 Sentinel 来处理了。并且资源和规则是解耦的，规则甚至可以在运行时动态修改。定义完资源后，就可以通过在程序中埋点来保护你自己的服务了，埋点的方式有两种：

- try-catch 方式（通过 

- if-else 方式（通过 

以上这两种方式都是通过硬编码的形式定义资源然后进行资源埋点的，对业务代码的侵入太大，从0.1.1版本开始， Sentinel 加入了注解的支持，可以通过注解来定义资源，具体的注解为：

在 Sentinel 中具体表示资源的类是：

顾名思义，

![](../../assets/images/Sentinel/attachments/Sentinel原理（2）：重要概念_image_0.png)

## Slot

Slot 是另一个 Sentinel 中非常重要的概念， Sentinel 的工作流程就是围绕着一个个插槽所组成的插槽链来展开的。需要注意的是每个插槽都有自己的职责，他们各司其职完好的配合，通过一定的编排顺序，来达到最终的限流降级的目的。默认的各个插槽之间的顺序是固定的，因为有的插槽需要依赖其他的插槽计算出来的结果才能进行工作。

但是这并不意味着我们只能按照框架的定义来，Sentinel 通过 

那SlotChain是在哪创建的呢？是在 

![](../../assets/images/Sentinel/attachments/Sentinel原理（2）：重要概念_image_1.png)

## Context

Context 上下文是 Sentinel 中一个比较难懂的概念。源码中是这样描述context类的：

> This class holds metadata of current invocation


就是说在context中维护着当前调用链的元数据，那元数据有哪些呢，从context类的源码中可以看到有：


- entranceNode：当前调用链的入口节点

- curEntry：当前调用链的当前entry

- node：与当前entry所对应的curNode

- origin：当前调用链的调用源

![](../../assets/images/Sentinel/attachments/Sentinel原理（2）：重要概念_image_2.png)

每次调用 

那如果我想自己在调用  

另外context是保存在ThreadLocal中的，每次执行的时候会优先到ThreadLocal中获取。如果context为null时才会再次去创建一个context。

那什么时候context会被置为null并从ThreadLocal中清空呢？当Entry执行exit方法时，当当前entry的parent为null时，也就说明当前entry是最上层的节点了，此时要把保存在ThreadLocal中的context也清空掉。

**在NodeSelectorSlot类中有一个Map保存了DefaultNode，但是key是用的contextName，而不是resourceName，这是为什么呢？**

试想一下，如果用resourceName来做map的key，那对于同一个资源resourceA来说，在context1中获取到的defaultNodeA和在context2中获取到的defaultNodeA是同一个，那么怎么在这两个context中对defaultNodeA进行更改呢，修改了一个必定会对另一个产生影响。

![](../../assets/images/Sentinel/attachments/Sentinel原理（2）：重要概念_image_3.png)

而如果用contextName来作为key，那对于同一个资源resourceA来说，在context1中获取到的是defaultNodeA1，在context2中获取到是defaultNodeA2，那在不同的context中对同一个资源可以使用不同的DefaultNode进行分别统计和计算，最后再通过ClusterNode进行合并就可以了。

![](../../assets/images/Sentinel/attachments/Sentinel原理（2）：重要概念_image_4.png)

所以在NodeSelectorSlot这个类里面，map里面保存的是contextName和DefaultNode的映射关系，目的是为了可以在不同的context对相同的资源进行分开统计。

同一个context中对同一个resource进行多次entry()调用时，会形式一颗调用树，这个树是通过CtEntry之间的parent/child关系维护的。

> 具体的调用链的原理分析可以参考笔者的另一篇文章：限流降级神器-哨兵(Sentinel)的资源调用链原理分析


## Entry

Entry 是 Sentinel 中用来表示是否通过限流的一个凭证，就像一个token一样。每次执行 

entry中保存了本次执行 entry() 方法的一些基本信息，包括：

- createTime：当前Entry的创建时间，主要用来后期计算rt

- node：当前Entry所关联的node，该node主要是记录了当前context下该资源的统计信息

- origin：当前Entry的调用来源，通常是调用方的应用名称，在

- resourceWrapper：当前Entry所关联的资源

当在一个上下文中多次调用了 

**需要注意的是：parent和child是在 CtSph 类的一个私有内部类 CtEntry 中定义的，CtEntry 是 Entry 的一个子类。**

由于context中总是保存着调用链树中的当前入口，所以当当前entry执行exit退出时，需要将parent设置为当前入口。

![](../../assets/images/Sentinel/attachments/Sentinel原理（2）：重要概念_image_5.png)

## Node

Node 中保存了资源的实时统计数据，例如：passQps，blockQps，rt等实时数据。正是有了这些统计数据后， Sentinel 才能进行限流、降级等一系列的操作。

node是一个接口，他有一个实现类：StatisticNode，但是StatisticNode本身也有两个子类，一个是DefaultNode，另一个是ClusterNode，DefaultNode又有一个子类叫EntranceNode。

其中entranceNode是每个上下文的入口，该节点是直接挂在root下的，是全局唯一的，每一个context都会对应一个entranceNode。另外defaultNode是记录当前调用的实时数据的，每个defaultNode都关联着一个资源和clusterNode，有着相同资源的defaultNode，他们关联着同一个clusterNode。

![](../../assets/images/Sentinel/attachments/Sentinel原理（2）：重要概念_image_6.png)

## Metric

Metric 是 Sentinel 中用来进行实时数据统计的度量接口，node就是通过metric来进行数据统计的。而metric本身也并没有统计的能力，他也是通过Window来进行统计的。

> 具体的统计原理，可以参考笔者另一篇文章： Sentinel 基于滑动时间窗口的实时指标统计原理分析


Metric有一个实现类：ArrayMetric，在ArrayMetric中主要是通过一个叫WindowLeapArray的对象进行窗口统计的。



![](../../assets/images/Sentinel/attachments/Sentinel原理（2）：重要概念_image_7.png)