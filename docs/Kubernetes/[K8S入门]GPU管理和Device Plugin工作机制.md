---
layout: default
title: [K8S入门]GPU管理和Device Plugin工作机制
parent: Kubernetes
---

### 一、需求来源

经过近几年的发展，AI有了许许多多的落地场景，包括只能客服、人脸识别、机器翻译、以图搜图等功能。其实机器学习或者说是人工智能，并不是什么新鲜的概念。而这次热潮的背后，云计算的普及以及算力的巨大提升，才是真正将人工只能从象牙塔带到工业界的一个重要推手。

![](../../assets/images/Kubernetes/attachments/[K8S入门]GPU管理和Device%20Plugin工作机制_image_0.png)

与之相对应的，从2016年开始，K8S社区就不断收到来自不同渠道的大量诉求：希望能在K8S集群上运行TensorFlow等机器学习框架。这些诉求中，除了之前文章所介绍的，像Job这些离线任务的管理之外，还有一个巨大的挑战：深度徐熙所依赖的异构设备以及英伟达的GPU支持。

我们不禁好奇起来：K8S管理GPU能带来什么好处呢?

本质上是成本和效率的考虑。由于相对CPU来说，GPU的成本偏高。在云上单CPU通常是一小时几毛钱，而GPU的花费则是从单GPU每小时10元~30元不等，这就要想方设法的提高GPU的使用率。

为什么要用K8S管理以GPU为代表的异构资源?

具体来说是三个方面：

- 加速部署：通过容器构想避免重复部署机器学习复杂环境

- 提升集群资源使用率：统一调度和分配集群资源

- 保障资源独享：利用容器隔离异构设备，避免互相影响

首先是加速部署，避免把时间浪费在环境准备的环节中。通过容器镜像技术，将整个部署过程进行固化和复用，如果关注机器学习领域，可以发现许许多多的框架都提供了容器镜像。我们可以借此提升GPU的使用效率。

荣国分时复用，来提升GPU的使用效率。当GPU的卡数达到一定数量后，就需要用到K8S的统一调度能力，使得资源使用方能够作导用即申请、完即释放，从而盘活整个GPU的资源池。

而此时还需要通过Docker自带的设备隔离能力，避免不同应用的进程运行同一个设备上，造成互相影响。在高效低成本的同时，也保障了系统的稳定性。

### 二、GPU的容器化

#### 1. 容器环境下使用GPU应用

在容器环境下使用GPU应用，实际上不复杂。主要分为两步：

- 构建支持GPU的容器镜像

- 利用Docker将该镜像运行起来，并且把GPU设备和依赖库映射到容器中。

#### 2. 如何准备GPU容器镜像

有两个方法准备：

- 直接使用官方深度学习容器镜像

比如直接从docker.hub或者阿里云镜像服务中寻找官方的GPU镜像，包括像TensorFlow、Caffe、PyTorch等流行的机器学习框架，都有提供标准的镜像。这样的好处是简单便捷，而且安全可靠。

- 基于Nvidia的CUDA镜像基础构建

当然如果官方镜像无法满足需求时，比如你对TensorFlow框架进行了定制修改，就需要中心编译构建自己的TensorFlow镜像。这种情况下，我们的最佳实践是：依托于Nvidia官方镜像继续构建，而不要从头开始。

如下图中的TensorFlow例子所示，这个就是以CUDA镜像为基础，开始构建自己的GPU镜像。

![](../../assets/images/Kubernetes/attachments/[K8S入门]GPU管理和Device%20Plugin工作机制_image_1.png)

#### 3. GPU容器镜像原理

要了解如何构建GPU容器镜像，先要知道如何要在宿主机上安装GPU应用。

如下图左边所示，最底层是先安装Nvidia硬件驱动；再到上面是通用的CUDA工具库；最上层是PyTorch、TensorFlow这类的机器学习框架。

上两层的CUDA工具库和应用的耦合度较高，应用版本变动后，对应的CUDA版本大概率也要更新；而最下层的Nvidia驱动，通常情况下是比较稳定的，它不会像CUDA和应用一样，经常更新。

![](../../assets/images/Kubernetes/attachments/[K8S入门]GPU管理和Device%20Plugin工作机制_image_2.png)

同时Nvidia驱动需要内核源码编译，如上图右侧所示，英伟达的GPU容器方案是：在宿主机上安装Nvidia驱动，而在CUDA以上的软件较给容器镜像来做。同时把Nvidia驱动里面的链接以Mount Bind的方式映射到容器中。

这样的一个好处是：当你安装了一个新的Nvidia驱动之后，你就可以在同一个机器节点上运行不同版本的CUDA镜像了。

#### 4. 如何利用容器运行GPU程序

有了前面的基础，我们就比较容易理解GPU容器的工作机制。下图是一个使用Docker运行GPU容器的例子。

![](../../assets/images/Kubernetes/attachments/[K8S入门]GPU管理和Device%20Plugin工作机制_image_3.png)

我们可以观察到，在运行时刻一个GPU容器和普通容器之间的差别，仅仅在于需要将宿主机的设备和Nvidia驱动库映射到容器中。

上图右侧反映了GPU容器启动后，容器中的GPU配置。右上方展示的是设备映射的结果，右下方显示的是驱动库以Bind方式映射到容器后，可以看到的变化。

通常大家会使用Nvidia-docker来运行GPU容器，而Nvidia-docker的实际工作就是来自动化做这两个工作。其中挂在设备比较简单，而真正比较复杂的是GPU应用以来的驱动库。

对于深度学习，视频处理等不同场景，所使用的一些驱动库并不相同。这又需要依赖Nvidia的领域知识，而这些领域知识就被贯穿到了Nvidia的容器之中。

### 三、K8S的GPU管理

#### 1. 如何部署GPU K8S

首先看一下如何给一个K8S节点增加GPU能力，我们以CentOS节点为例。

![](../../assets/images/Kubernetes/attachments/[K8S入门]GPU管理和Device%20Plugin工作机制_image_4.png)

如上图所示：

- 首先安装Nvidia驱动

由于Nvidia驱动需要内核编译，所以在安装Nvidia驱动之前需要安装gcc和内核源码。

- 第二步通过yum源，安装Nvidia Docker2

安装完Nvidia Docker2需要重新加载docker，可以检查docker的daemon.json里面默认启动引擎已经被替换成了nvidia，也可以通过docker info命令查看运行时刻使用的runC是不是Nvidia的runC。

- 第三步是部署Nvidia Device Plugin

从Nvidia的git repo下去下载Device Plugin的部署声明文件，并且通过kubectl create命令进行部署。

这里Device Plugin是以daemonset的方式进行部署的。这样我们就知道，如果需要排查一个K8S节点无法调度GPU应用的问题，需要从这些模块开始入手，比如我要查看一下Device Plugin的日志，Nvidia的runC是否配置为docker默认runC以及Nvidia驱动是否安装成功。

#### 2. 验证部署GPU K8S结果

当GPU节点部署成功后，我们可以从节点的状态信息中发现相关的GPU信息。

- 一个是GPU的名称，这里是nvdia.com/gpu

- 另一个是它对应的数量，如下图所示是2，表示在该节点上含有两个GPU

![](../../assets/images/Kubernetes/attachments/[K8S入门]GPU管理和Device%20Plugin工作机制_image_5.png)

#### 3. 在K8S中使用GPU的yaml样例

站在用户的角度，在K8S中使用GPU容器还是非常简单的。

只需要在Pod资源配置的limit字段中指定nvidia.com/gpu使用GPU的数量，如下图样例中我们设置的数量为1；然后再通过kubectl create命令将GPU的Pod部署完成。

![](../../assets/images/Kubernetes/attachments/[K8S入门]GPU管理和Device%20Plugin工作机制_image_6.png)

#### 4. 查看运行结果

部署完成后可以登录到容器中执行nvidia-smi命令观察一下结果，可以看到再该容器中使用了一张T4的GPU卡。说明在该节点重得两张GPU卡其中一张已经能在该容器中使用了，但是节点的另外一张卡对于该容器来说是完全透明的，它是无法访问的，这里就体现了GPU的隔离性。

![](../../assets/images/Kubernetes/attachments/[K8S入门]GPU管理和Device%20Plugin工作机制_image_7.png)

### 四、工作原理

#### 1. 通过扩展的方式管理GPU资源

K8S本身是通过插件扩展的机制来管理GPU资源的，具体来说这里有两个独立的内部机制。

![](../../assets/images/Kubernetes/attachments/[K8S入门]GPU管理和Device%20Plugin工作机制_image_8.png)

- 第一个是Extend Resources，允许用户自定义资源名称。而该资源的度量是整数级别，这样做的目的在于通过一个通用的模式支持不同的异构设备，包括RDMA、FPGA、AMD GPU等等，而不仅仅是为Nvidia GPU设计的

- Device Plugin Framework允许第三方设备提供商以外置的方式对设备进行全生命周期的管理，而Device Plugin Framework建立K8S和Device Plugin模块之间的桥梁。它一方面负责设备信息的上报到K8S，另一方面负责设备的调度选择。

#### 2. Extended Resource的上报

Extend Resources域属Node-level的api，完全可以独立于Device Plugin使用。而上报Extend Resources，只需要通过一个Patch API对Node对象进行status部分更新即可，而这个Patch操作可以通过一个简单的curl命令来完成。这样，再K8S调度器中就能够记录这个节点的GPU类型，它所对应的资源数量是1。

![](../../assets/images/Kubernetes/attachments/[K8S入门]GPU管理和Device%20Plugin工作机制_image_9.png)

当然如果使用的是Device Plugin，就不需要做这个Patch操作，只需要遵从Device Plugin的编程模型，在设备上报的工作中Device Plugin就会完成这个操作。

#### 3. Device Plugin工作机制

介绍一下Device Plugin的工作机制，整个Device Plugin的工作流程可以分成两个部分：

- 一个是启动时刻的资源上报

- 另一个是用户使用时刻的调度和运行

![](../../assets/images/Kubernetes/attachments/[K8S入门]GPU管理和Device%20Plugin工作机制_image_10.png)

Device Plugin的开发非常简单。主要包括最关注与最核心的两个事件方法：

- 其中ListAndWatch赌赢资源的上报，同时还提供健康检查的机制。当设备不健康的时候，可以上报给K8S不健康设备的ID，让Device Plugin Framework将这个设备从可调度设备中移除

- 而Allocate会被Device Plugin再部署容器时调用，传入的参数核心就是容器会使用的设备ID，返回的参数是容器启动时，需要的设备、数据卷以及环境变量。

#### 4. 资源上报和监控

对于每一个硬件设备，都需要它所对应的Device Plugin进行管理，这些Device Plugin以客户端得身份通过GRPC的方式对kubelet重得Device Plugin Manager进行连接，并且将自己监听的Unis socket api的版本号和设备名称比如GPU，上报给kubelet。

我们来看一下Device Plugin资源上报的整个流程。总的来说，整个过程分为四步，其中前三步是发生在节点上，第四步是kubelet和api-server的交互。

![](../../assets/images/Kubernetes/attachments/[K8S入门]GPU管理和Device%20Plugin工作机制_image_11.png)

- 第一步是Device Plugin的注册，需要K8S知道要跟哪个Device Plugin进行交互。这就因为一个节点上可能有多个设备，需要Device Plugin以客户端得身份像Kubelet汇报三件事情：我是谁?就是Device Plugin所管理的设备名称，是GPU还是RDMA；我在哪?就是插件自身监听的unis socket所在的文件位置，让kubelet能够调用自己；交互协议?就是API的版本号

- 第二步是服务启动，Device Plugin会启动一个GRPC的server。在此之后Device Plugin一直以这个服务器的身份提供服务让kubelet来访问，而监听地址和提供API的版本就已经在第一步完成了

- 第三步，当该GRPC server启动之后，kubelet会建立一个到Device Plugin的ListAndWatch的长连接，用来发现设备ID以及设备得健康状态。当Device Plugin检测到某个设备不健康的时候，就会主动通知kubelet。而此时如果这个设备处于空闲状态，kubele会将其移除可分配的列表。但是当这个设备已经被某个Pod所使用的时候，kubelet就不会做任何事情，如果此时杀掉这个Pod是一个很危险的操作

- 第四步，kubelet会将这些设备暴露到Node节点的状态中，把设备数量发送到K8S的api-server中。后续调度器可以根据这些信息进行调度。

需要注意的是kubelet在向api-server进行汇报的时候，只会汇报该GPU对应的数量。而kubelet自身的Device Plugin Manager会对这个GPU的ID列表进行保存，并用来具体的设备分配。而这个对于K8S全局调度器来说，它不掌握这个GPU的ID列表，它只知道GPU的数量。

这就意味着在现有的Device Plugin工作机制下，K8S的全局调度器无法进行更复杂的调度。比如说想做两个GPU的亲和性调度，同一个节点两个GPU可能需要进行通过NVLINK通讯而不是PCIe通讯，才能达到更好的数据传输效果。在这种需求下，目前的Device Plugin调度机制中是无法实现的。

#### 5. Pod的调度和运行的过程

![](../../assets/images/Kubernetes/attachments/[K8S入门]GPU管理和Device%20Plugin工作机制_image_12.png)

Pod想使用一个GPU的时候，他只需要像之前的例子一样，在Pod的Resource下limits字段中声明GPU资源和对应的数量(比如nvidia.com/gpu:1)。K8S会找到满足数量条件的节点，然后将该节点的GPU数量减1，并且完成Pod与Node的绑定。

绑定成功后，自然就会被对应节点的kubelet拿来创建容器。而当kubelet发现这个Pod的容器请求得资源是一个GPU的时候，kubelet就会委托自己内部的Device Plugin Manager模块，从自己持有的GPU的ID列表中选择一个可用的GPU分配给该容器。

此时kubelet就会向本机的Device Plugin发起一个Allocate请求，这个请求所携带的参数，正是即将分配给该容器的设备ID列表。

Device Plugin收到AllocateRequest请求之后，它就会根据kubelet传过来的设备ID，去寻找这个设备ID对应的设备路径、驱动目录以及环境变量，并且以AllocateResponse的形式返还给kubelet。

AllocateResponse中所携带的设备路径和驱动目录信息，一旦返回给kubelet之后，kubelet就会根据这些信息执行为容器分配GPU的操作，这样Docker会根据kubelet的指令去创建容器，而这个容器中就会出现GPU设备。并且把它所需要的驱动目录给挂载进来，至此K8S为Pod分配一个GPU的流程就结束了。

### 五、思考与实践

#### 1. 本文总结

在本文中，我们一起学习了在Docker和K8S上使用GPU。

- GPU的容器化：如何去构建一个GPU镜像；如何直接在Docker上运行GPU容器

- 利用K8S管理GPU资源：如何在K8S支持GPU调度；如何验证K8S下的GPU配置；调度GPU容器的方法

- Device Plugin的工作机制：资源的上报和监控；Pod的调度和运行

- 思考：目前的缺陷；社区常见的Device Plugin

#### 2. Device Plugin机制的缺陷

最后我们来思考一个问题，现在的Device Plugin是否完美无缺?

需要指出的是Device Plugin整个工作机制和流程上，实际上跟学术界和工业界的真实场景有比较大的差异。这里最大的问题在于GPU资源的调度工作，实际上都是在kubelet上完成的。

而作为全局的调度器对这个参与是非常有限的，作为传统的K8S调度器来说，它只能处理GPU数量。一旦你的设备是异构的，不能简单的使用数目去描述需求的时候，比如我的Pod像运行在两个nvlink的GPU上，这个Device Plugin就完全不能处理。

更不用说在许多场景上，我们希望调度器进行调度的时候，是根据整个集群的设备进行全局调度，这种场景是目前的Device Plugin无法满足的。

更为棘手的是在Device Plugin的设计和实现中，像Allocate和ListAndWatch的API去增加可扩展的参数也是没有作用的。这就是当我们使用一些比较复杂的设备使用需求的时候，实际上是无法通过Device Plugin来扩展API实现的。

因此目前的Device Plugin设计涵盖的场景其实是非常单一的，是一个可用但是不好用的状态。这就能解释为什么像Nvidia这些厂商都实现了一个基于K8S上游代码进行fork了自己解决方案，也是不得已而为之。

#### 3. 社区的异构资源调度方案

![](../../assets/images/Kubernetes/attachments/[K8S入门]GPU管理和Device%20Plugin工作机制_image_13.png)

- 第一个是Nvidia贡献的调度方案，这是最常用的调度方案

- 第二个是由阿里云服务团队贡献的GPU共享的调度方案，其目的在于解决用户共享GPU调度的需求，焕应大家一起来使用和改进

- 下面的两个RDMA和FPGA是由具体厂商提供的调度方案