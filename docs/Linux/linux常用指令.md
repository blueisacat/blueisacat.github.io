总核数 = 物理CPU个数 X 每颗物理CPU的核数

总逻辑CPU数 = 物理CPU个数 X 每颗物理CPU的核数 X 超线程数

查看物理CPU个数

```bash
cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l
```

查看每个物理CPU中core的个数(即核数)

```bash
cat /proc/cpuinfo| grep "cpu cores"| uniq
```

查看逻辑CPU的个数

```bash
cat /proc/cpuinfo| grep "processor"| wc -l
```

启动脚本

```bash
#!/bin/sh
tpid=`cat tpid | awk '{print $1}'`
tpid=`ps -aef|grep $tpid | awk '{print $2}'|grep $tpid`
if [ ${tpid} ]
then  kill -9 $tpid
fi
```

关闭脚本

```bash
#!/bin/sh
tpid=`cat tpid | awk '{print $1}'`
tpid=`ps -aef|grep $tpid | awk '{print $2}'|grep $tpid`
if [ ${tpid} ]
then  kill -9 $tpid
fi
```