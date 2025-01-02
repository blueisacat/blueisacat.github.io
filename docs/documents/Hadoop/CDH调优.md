# CDH调优

## 1 YARN相关调优

| 参数 | 描述及建议 |
| -- | -- |
| yarn.nodemanager.resource.memory-mb | 每个NodeManager的可分配内存大小。 |
| yarn.nodemanager.resource.cpu-vcores | 每个NodeManager的可分配CPU核数。 |
| mapreduce.map.memory.mb | 每个Map任务可使用的内存大小。 |
| mapreduce.map.cpu.vcores | 每个Map任务可使用的CPU核数。 |
| mapreduce.reduce.memory.mb | 每个Reduce任务可使用的内存大小。 |
| mapreduce.reduce.cpu.vcores | 每个Reduce任务可使用的CPU核数。 |
| yarn.app.mapreduce.am.resource.mb | 每个ApplicationMaster可使用的内存大小。 |
| yarn.app.mapreduce.am.resource.cpu-vcores | 每个ApplicationMaster可使用的CPU核数。 |
| ApplicationMaster Java 最大堆栈 | 每个ApplicationMaster的Java最大堆栈，建议yarn.app.mapreduce.am.resource.mb * 0.8。 |
| mapreduce.task.io.sort.mb | 每个Map任务输出排序的内存大小，更大的值可以减少排序时的溢出文件个数。 |
| mapreduce.job.reduce.slowstart.completedmaps | 当Map任务完成对应百分比时Reduce开始申请资源，建议资源紧张的情况下设置为1，所有Map任务完成后再启动Reduce任务。 |
| mapreduce.job.reduces | 默认Reduce任务的数量。 |
| 最大进程文件描述符数 | 可使用的最大进程文件描述符数，建议与系统最大文件描述符数一致。 |
| mapreduce.reduce.shuffle.parallelcopies | 每个Reduce任务Shuffle开启的线程数，建议设置为5。 |

!!! warning

    `mapreduce.reduce.shuffle.input.buffer.percent(默认0.7)× mapreduce.reduce.shuffle.memory.limit.percent(默认0.25)× mapreduce.reduce.shuffle.parallelcopies < 1` ，当大于 `1` 时会导致内存溢出。

## 2 HDFS相关优化

| 参数 | 描述及建议 |
| -- | -- |
| 最大进程文件描述符数 | 可使用的最大进程文件描述符数，建议与系统最大文件描述符数一致。 |
| dfs.datanode.handler.count | 每个DataNode中用于处理RPC调用的线程数，默认值为3，建议值为20。 |
| dfs.datanode.max.xcievers | 每个DataNode可使用的最大进程文件描述符数，建议与系统最大文件描述符一致。 |
| ipc.maximum.data.length | NameNode可以接收的最大数据包大小，默认值为64M，当block数量达到一定程度后，需要增大该值，否则无法传输。 |
| dfs.namenode.fs-limits.max-directory-items | 每个目录下子目录或文件的最大数量，默认值为1048576，当文件过多时，需要增大该值，否则无法存储。 |
| dfs.datanode.handler.count | DataNode中用于处理RPC调用的线程数，默认值为3，可适当增加该值提升并发度。 |
| dfs.namenode.handler.count | NameNode中用于处理RPC调用的线程数，默认值为10，可适当增加该值提升并发度。 |
| dfs.namenode.service.handler.count | NameNode中用于处理DataNode上报数据块和心跳的线程数，与dfs.namenode.handler.count保持一致。 |
| dfs.socket.timeout | Socket读超时时间，默认值为60000ms，并发度高时可适当增加该值。 |
| dfs.datanode.socket.write.timeout | Socket写超时时间，默认值为60000ms，并发度高时可适当增加该值。 |
| dfs.client.block.write.locateFollowingBlock.retries | NameNode需要等待Block达到最小副本数变为COMPLETE状态的尝试次数，默认值为5，并发都高时可适当增加该值。 |

## 3 Hive相关优化

| 参数 | 描述及建议 |
| -- | -- |
| 最大进程文件描述符数 | 可使用的最大进程文件描述符数，建议与系统最大文件描述符数一致。 |

## 4 Hue相关优化

| 参数 | 描述及建议 |
| -- | -- |
| 最大进程文件描述符数 | 可使用的最大进程文件描述符数，建议与系统最大文件描述符数一致。 |

## 5 Impala相关优化

| 参数 | 描述及建议 |
| -- | -- |
| 最大进程文件描述符数 | 可使用的最大进程文件描述符数，建议与系统最大文件描述符数一致。 |

## 6 Oozie相关优化

| 参数 | 描述及建议 |
| -- | -- |
| 最大进程文件描述符数 | 可使用的最大进程文件描述符数，建议与系统最大文件描述符数一致。 |

## 7 ZooKeeper相关优化

| 参数 | 描述及建议 |
| -- | -- |
| 最大进程文件描述符数 | 可使用的最大进程文件描述符数，建议与系统最大文件描述符数一致。 |

## 8 Cloudera Management Service相关优化

| 参数 | 描述及建议 |
| -- | -- |
| 最大进程文件描述符数 | 可使用的最大进程文件描述符数，建议与系统最大文件描述符数一致。 |

## 9 linux相关优化

* 系统可分配最大文件数

    ```shell
    # 查看
    sysctl -a | grep 'fs.file-max'
    # 修改
    echo "fs.file-max = 167772166 " >> /etc/sysctl.conf
    # 立即生效
    sysctl -p
    ```

* 进程可分配最大文件数

    ```shell
    # 查看
    sysctl -a | grep 'fs.nr_open'
    # 修改
    echo "fs.nr_open = 167772166 " >> /etc/sysctl.conf
    # 立即生效
    sysctl -p
    ```

    !!! tip

        默认值：1048576

* 用户进程可分配最大文件数

    ```shell
    # 查看
    ulimit -a | grep "open files"
    # 修改
    # 在/etc/security/limits.conf中添加或修改
    * soft nofile 1048576
    * hard nofile 1048576
    * soft nproc 1048576
    * hard nproc 1048576
    # 在/etc/security/limits.d/20-nproc.conf中添加或修改
    * soft nofile 1048576
    * hard nofile 1048576
    * soft nproc 1048576
    * hard nproc 1048576
    # 在/etc/systemd/system.conf中添加或修改
    DefaultLimitNOFILE=1048576
    DefaultLimitNPROC=1048576
    # 在/etc/systemd/user.conf中添加或修改
    DefaultLimitNOFILE=1048576
    DefaultLimitNPROC=1048576
    # 使配置生效
    systemctl daemon-reload
    systemctl daemon-reexec
    ```

    !!! tip

        默认值： `1024` ，如果需要超过 `1048576` ，需要先增大 `nr_open` 值。
