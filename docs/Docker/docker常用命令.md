### 容器生命周期管理

#### run

docker run:创建一个新的容器并运行一个命令

```shell
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

#### start/stop/restart

docker start:启动一个或多个已经被停止的容器

```shell
docker start [OPTIONS] CONTAINER [CONTAINER...]
```

docker stop:停止一个运行中的容器

```shell
docker stop [OPTIONS] CONTAINER [CONTAINER...]
```

docker restart:重启容器

```shell
docker restart [OPTIONS] CONTAINER [CONTAINER...]
```

#### kill

docker kill:杀掉一个运行中的容器

```shell
docker kill [OPTIONS] CONTAINER [CONTAINER...]
```

#### rm

docker rm:删除一个或多个容器

```shell
docker rm [OPTIONS] CONTAINER [CONTAINER...]
```

#### pause/unpause

docker pause:暂停容器中所有的进程

```shell
docker pause [OPTIONS] CONTAINER [CONTAINER...]
```

docker unpause:恢复容器中所有的进程

```shell
docker unpause [OPTIONS] CONTAINER [CONTAINER...]
```

#### create

docker create:创建一个新的容器但不启动它

```shell
docker create [OPTIONS] IMAGE [COMMAND] [ARG...]
```

#### exec

docker exec:在运行的容器中执行命令

```shell
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

### 容器操作

#### ps

docker ps:列出容器

```shell
docker ps [OPTIONS]
```

#### inspect

docker inspect:获取容器/镜像的元数据

```shell
docker inspect [OPTIONS] NAME|ID [NAME|ID...]
```

#### top

docker top:查看容器中运行的进程信息,支持ps命令参数

```shell
docker top [OPTIONS] CONTAINER [ps OPTIONS]
```

#### attach

docker attach:连接到正在运行中的容器

```shell
docker attach [OPTIONS] CONTAINER
```

#### events

docker events:从服务器获取实时事件

```shell
docker events [OPTIONS]
```

#### logs

docker logs:获取容器的日志

```shell
docker logs [OPTIONS] CONTAINER
```

#### wait

docker wait:阻塞运行直到容器停止,然后打印出它的退出代码

```shell
docker wait [OPTIONS] CONTAINER [CONTAINER...]
```

#### export

docker export:将文件系统作为一个tar归档文件导出到STDOUT

```shell
docker export [OPTIONS] CONTAINER
```

#### port

docker port:列出指定的容器的端口映射,或者查找将PRIVATE_PORT NAT到面向公众的端口

```shell
docker port [OPTIONS] CONTAINER [PRIVATE_PORT[/PROTO]]
```

### 容器rootfs命令

#### commit

docker commit:从容器创建一个新的镜像

```shell
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```

#### cp

docker cp:用于容器与主机之间的数据拷贝

```shell
docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH
```

#### diff

docker diff:检查容器里文件结构的更改

```shell
docker diff [OPTIONS] CONTAINER
```

### 镜像仓库

#### login

docker login:登录到一个Docker镜像仓库,如果未指定镜像仓库地址,默认为官方仓库Docker Hub

docker logout:登出一个Docker镜像仓库,如果未指定镜像仓库地址,默认为光放仓库Docker Hub

```shell
docker login [OPTIONS] [SERVER]
docker logout [OPTIONS] [SERVER]
```

#### pull

docker pull:从镜像仓库中拉取或者更新指定镜像

```shell
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```

#### push

docker push:将本地的镜像上传到镜像仓库,要先登录到镜像仓库

```shell
docker push [OPTIONS] NAME[:TAG]
```

#### search

docker search:从Docker Hub查找镜像

```shell
docker search [OPTIONS] TERM
```

### 本地镜像管理

#### images

docker images:列出本地镜像

```shell
docker images [OPTIONS] [REPOSITORY[:TAG]]
```

#### rmi

docker rmi:删除本地一个或多个镜像

```shell
docker rmi [OPTIONS] IMAGE [IMAGE...]
```

#### tag

docker tag:标记本地镜像,将其归入某一仓库

```shell
docker tag [OPTIONS] IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]
```

#### build

docker build:命令用于使用Dockerfile创建镜像

```shell
docker build [OPTIONS] PATH | URL | -
```

#### history

docker history:查看指定镜像的创建历史

```shell
docker history [OPTIONS] IMAGE
```

#### save

docker save:将指定镜像保存成tar归档文件

```shell
docker save [OPTIONS] IMAGE [IMAGE...]
```

#### load

docker load:导入使用docker save命令导出的镜像

```shell
docker load [OPTIONS]
```

#### import

docker import:从归档文件中创建镜像

```shell
docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]
```

### 其他

#### info

docker info:显示Docker系统信息,包括镜像和容器数

```shell
docker info [OPTIONS]
```

#### version

docker version:显示Docker版本信息

```shell
docker version [OPTIONS]
```