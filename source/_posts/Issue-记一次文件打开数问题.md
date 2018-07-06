---
title: Issue 记一次文件打开数问题
date: 2018-07-06 14:40:08
categories: 项目
tags:
  - Linux
  - Golang
---

最近新上线了一个应用，上线之后跑了2天发现出现错误，一直的报 `too many open files`。

那么程序设置的最大文件数是多少呢？可以通过 `proc limit` 查看：

```
# cat /proc/<pid>/limits
Limit                     Soft Limit           Hard Limit           Units
Max cpu time              unlimited            unlimited            seconds
Max file size             unlimited            unlimited            bytes
Max data size             unlimited            unlimited            bytes
Max stack size            8388608              unlimited            bytes
Max core file size        unlimited            unlimited            bytes
Max resident set          unlimited            unlimited            bytes
Max processes             100000               100000               processes
Max open files            1024                 4096                 files
Max locked memory         65536                65536                bytes
Max address space         unlimited            unlimited            bytes
Max file locks            unlimited            unlimited            locks
Max pending signals       29837                29837                signals
Max msgqueue size         819200               819200               bytes
Max nice priority         0                    0
Max realtime priority     0                    0
Max realtime timeout      unlimited            unlimited            us
```

其中`Max open files`中最大的软连限制是`1024`，硬连限制是`4096`，用命令查了一下进程的文件打开数，发现超过了 1024。

```
# ls -l /proc/<pid>/fd |wc -l
1025
```

还可以查看程序都打开了哪些文件：

```
# ls -l /proc/<pid>/fd
```

通过修改在 `/etc/security/limits.d/` 中的配置：

```
*     soft   nofile    65535
*     hard   nofile    65535
*     soft   nproc     65535
*     hard   nproc     65535
*     soft   core      65535
*     hard   core      65535
```

然后重启，查看系统的配置：

```shell
$ulimit -n
65535

$ulimit -a

# ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 15090
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 65535
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 15090
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

配置已经生效，这个时候重现跑了一下连接测试，发现程序还是超过 1024 之后报错，使用命令 `cat /proc/<pid>/limits` 查看程序的文件数还是 1024。这是为什么呢？

原来程序是跑在 centos7 上的，而上面的方法是针对 centos5/6 的，在 centos7 使用的初始化系统的守护进程工具是 `systemd`，所有需要修改 `systemd` 的配置。

`systemd` 的配置文件路径 `/etc/systemd/system.conf`：

```
DefaultLimitCORE=infinity
DefaultLimitNOFILE=65535
DefaultLimitNPROC=65535
```

然后重启。再查看 `cat /proc/<pid>/limits` 发现程序的最大文件数已经生效为 65535，这个时候在跑程序就没有报 `too many open files` 了。

### 查问题时使用到的命令

#### lsof

根据程序的端口列出打开文件信息：

```
lsof -i :8008
```

-i 根据端口查询
-p 根据pid 进程号查询

```
lsof -p 15008
```

```
lsof -n|awk '{print $2}'|sort|uniq -c|sort -nr|more
```

查看每个进程PID的句柄数

#### wc

统计行数，通过管道配合其他命令一起使用。

```
lsof -i :8008 |wc -l
```

#### netstat

```
$netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
ESTABLISHED 5
SYN_SENT 1
TIME_WAIT 12
```

查看每种连接状态的连接数


```
netstat -nat | grep ip |grep TIME_WAIT
```

查看具体IP的 TIME_WAIT 情况


## 资源

https://huoding.com/2015/08/02/460
https://www.jianshu.com/p/2e6748c4c0b7

