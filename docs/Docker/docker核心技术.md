---
layout: default
title: docker核心技术
parent: Docker
---

Docker底层实现主要基于LINUX技术，包含LINUX上的命名空间（Namespaces）、控制组（Control groups）、Union文件系统（Union file system）。

1. 命名空间。权限隔离控制，保证虽然在同一个宿主机上，但是相互是透明的。

1. 控制组。资源分配，保证各个容器资源的分配管理。

1. 联合文件系统。主要使用到cow技术（Copy on write），提高磁盘利用率。docker镜像，镜像可以通过分层来进行继承。支持将不同目录挂载到同一个虚拟文件系统下，Docker 中使用的 AUFS（AnotherUnionFS）就是一种联合文件系统。 AUFS 支持为每一个成员目录（类似 Git 的分支）设定只读（readonly）、读写（readwrite）和写出（whiteout-able）权限, 同时 AUFS 里有一个类似分层的概念, 对只读权限的分支可以逻辑上进行增量地修改(不影响只读部分的)。