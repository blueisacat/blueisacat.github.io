# Linux指令

## 1 基础指令

| 指令 | 英文 | 中文 |
| -- | -- | -- |
| man | Manual | 意思是手册，可以用这个命令查询其他命令的用法。 |
| pwd | Print working directory | 显示当前工作路径。 |
| su | Swith user | 切换用户，切换到root用户 |
| cd | Change directory | 切换目录 |
| ls | List files | 列出目录下的文件 |
| ps | Process Status | 进程状态 |
| mkdir | Make directory | 建立目录 |
| rmdir | Remove directory | 移动目录 |
| mkfs | Make file system | 建立文件系统 |
| fsck | File system check | 文件系统检查 |
| cat | Concatenate | 串联 |
| uname | Unix name | 系统名称 |
| df | Disk free | 空余硬盘 |
| du | Disk usage | 硬盘使用率 |
| lsmod | List modules | 列表模块 |
| mv | Move file | 移动文件 |
| rm | Remove file | 删除文件 |
| cp | Copy file | 复制文件 |
| ln | Link files | 链接文件 |
| fg | Foreground | 前景 |
| bg | Background | 背景 |
| chown | Change owner | 改变所有者 |
| chgrp | Change group | 改变用户组 |
| chmod | Change mode | 改变模式 |
| umount | Unmount | 卸载 |
| dd | Convert an copy | 转换和复制 |
| tar | Tape archive | 解压文件 |
| ldd | List dynamic dependencies | 列出动态相依 |
| insmod | Install module | 安装模块 |
| rmmod | Remove module | 删除模块 |
| lsmod | List module | 列表模块 |

## 2 进阶指令

!!! tip

    `总核数 = 物理CPU个数 X 每颗物理CPU的核数`
    
    `总逻辑CPU数 = 物理CPU个数 X 每颗物理CPU的核数 X 超线程数`

### 2.1 查看物理CPU个数

```shell
cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l
```

### 2.2 查看每个物理CPU中core的个数(即核数)

```shell
cat /proc/cpuinfo| grep "cpu cores"| uniq
```

### 2.3 查看逻辑CPU的个数

```shell
cat /proc/cpuinfo| grep "processor"| wc -l
```
