---
layout: default
title: IaaS、PaaS、SaaS、BaaS和FaaS, 这些区别你真的了解吗？
parent: Experience
---

IaaS、PaaS、SaaS、BaaS、FaaS，这些名词后面都带着aas三个字母，aas 是 As-a-Service，即为服务的意思。我们看下面这个架构图：

![](../../assets/images/Experience/attachments/IaaS、PaaS、SaaS、BaaS和FaaS,%20这些区别你真的了解吗？_image_0.png)

IaaS、PaaS、SaaS

云平台一般都会提供以上架构图中的三种云服务

- IaaS：Infrastructure as a Service（基础设施即服务）

从上面的架构图可以看出，IaaS处于最底层，服务商提供底层/物理层基础设施资源（服务器，数据中心，环境控制，电源，服务器机房），客户自己部署和执行操作系统或应用程序等各种软件。

- PaaS：Platform as a Service（平台即服务）

PaaS处于中间层，服务商提供基础设施底层服务，提供操作系统（Windows，Linux）、数据库服务器、Web服务器、域控制器和其他中间件，以及服务模型中的备份服务等中件层服务。例如IIS，.NET，Apache，MySQL …，客户自己控制上层的应用程序部署与应用托管的环境。

- SaaS：Software as a Service（软件即服务）

SaaS处于最上层，服务商提供基于软件的解决方案，满足客户最终需求；如OA、CRM、MIS、ERP、HRM、CM、Office 365、iCloud、G Suite等应用，客户不需考虑任何形式的专业技术知识，获得完整的软件包，使他们的日常工作和生活变得更轻松。

那它们之间又有什么区别呢？

网上流传着一个用开披萨店来解释云服务的例子：

![](../../assets/images/Experience/attachments/IaaS、PaaS、SaaS、BaaS和FaaS,%20这些区别你真的了解吗？_image_1.png)

请设想你是一个餐饮业者，打算做披萨生意。你可以从头到尾，自己生产披萨，但是这样比较麻烦，需要准备的东西多，因此你决定外包一部分工作，采用他人的服务。你有三个方案。

1. 方案一：IaaS

他人提供厨房、炉子、煤气，你使用这些基础设施，来烤你的披萨。

1. 方案二：PaaS

除了方案一的基础设施，他人还提供披萨饼皮。你只要把自己的配料洒在饼皮上，让他帮你烤出来就行了。也就是说，你要做的就是设计披萨的味道（海鲜披萨或者鸡肉披萨），他人提供平台服务，让你把自己的设计实现。

1. 方案三：SaaS

他人直接做好了披萨，不用你的介入，到手的就是一个成品。你要做的就是把它卖出去，最多再包装一下，印上你自己的 Logo。

三种方案总结如下图：

![](../../assets/images/Experience/attachments/IaaS、PaaS、SaaS、BaaS和FaaS,%20这些区别你真的了解吗？_image_2.png)

披萨即服务

从左到右，自己承担的工作量（上图蓝色部分）越来越少，IaaS > PaaS > SaaS。对应软件开发，则是下面这张图：

![](../../assets/images/Experience/attachments/IaaS、PaaS、SaaS、BaaS和FaaS,%20这些区别你真的了解吗？_image_3.png)

披萨云架构图

- 整体而言：

IaaS 是云服务的最底层，主要提供一些基础资源。

PaaS 提供软件部署平台（runtime），抽象掉了硬件和操作系统细节，可以无缝地扩展（scaling）。开发者只需要关注自己的业务逻辑，不需要关注底层。

SaaS 是软件的开发、管理、部署都交给第三方，不需要关心技术问题，可以拿来即用。

那么BaaS和FaaS又是什么呢？

- BaaS：Backend as a Service（后端即服务）

服务商为客户(开发者)提供整合云后端的服务，如提供文件存储、数据存储、推送服务、身份验证服务等功能，以帮助开发者快速开发应用。

- FaaS：Function as a service（函数即服务）

无服务器计算，当前使用最广泛的是AWS的Lambada。

服务商提供一个平台，允许客户开发、运行和管理应用程序功能，而无需构建和维护通常与开发和启动应用程序相关的基础架构的复杂性。 按照此模型构建应用程序是实现“无服务器”体系结构的一种方式，通常在构建微服务应用程序时使用。

其实还有很多的aaS，比如DaaS(Data as a service，数据即服务)、NaaS(Network as a service，网络即服务) 等等。