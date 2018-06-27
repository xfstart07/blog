---
title: "[笔记] Docker 操作容器"
date: 2018-06-27 09:40:38
categories: Docker
tags: docker
---

容器是 Docker 的一个核心概念。

**容器是什么？**

容器是独立运行的一个或一组应用，以及它们的运行态环境。

### 启动容器
主要使用 `docker run` 。

```
docker run -it centos /bin/bash
```

`-i`让容器的标准输入保持打开。
`-t`让 Docker 分配一个伪终端(pseudo-tty)并绑定到标准输入上。

当利用 `docker run` 来创建容器时，Docker 在后台运行的标准操作包括：

* 检查本地是否存在指定的镜像，不存在就从公有仓库下载
* 利用镜像创建并启动一个容器
* 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
* 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
* 从地址池配置一个 ip 地址给容器
* 执行用户指定的应用程序
* 执行完毕后容器被终止

### 后台运行
使用 `-d` 参数。

**为什么要在后台运行？**

在不使用`-d`参数运行容器。

```
$ docker run ubuntu:17.10 /bin/sh -c "while true; do echo hello world; sleep 1; done"
hello world
hello world
hello world
hello world
```
容器会把输出的结果(STDOUT) 打印到宿主上。

使用 `-d`运行容器。

```
$ docker run -d ubuntu:17.10 /bin/sh -c "while true; do echo hello world; sleep 1; done"
77b2dc01fe0f3f1265df143181e7b9af5e05279a884f4776ee75350ea9d8017a
```
使用 `-d`参数启动会返回一个唯一id，表示容器的ID。

**注**： 容器是否会长久运行，是和 docker run 指定的命令有关，和 -d 参数无关。

要获取容器的输出信息，可以通过 `docker container logs` 命令。

### 开启和终止
用 `docker container start` 来启动一个终止的容器。
用 `docker container stop` 来终止一个运行中的容器。
`docker container restart` 命令会将一个运行态的容器终止，然后再重新启动它。

终止状态的容器可以用 `docker ps -a` 命令看到。

### 进入容器
某些时候需要进入容器进行操作，包括使用 `docker attach` 命令或 `docker exec` 命令，推荐大家使用 `docker exec` 命令，原因会在下面说明。

**`attach`命令：**

```
docker attach container
root@243c32535da7:/#
```
注意：如果从这个 stdin 中 exit，会导致容器的停止。

**`exec`命令**

```
docker exec -it 69d1 bash
root@69d137adef7a:/#
```
如果从这个 stdin 中 exit，不会导致容器的停止。**这就是为什么推荐大家使用 docker exec 的原因。**

### 导入导出容器

**导出容器**

使用命令 `docker export`

```
$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                    PORTS               NAMES
7691a814370e        ubuntu:14.04        "/bin/bash"         36 hours ago        Exited (0) 21 hours ago                       test
$ docker export 7691a814370e > ubuntu.tar
```

**导入容器**

使用命令 `docker import`

### 删除容器

使用命令 `docker container rm` 来删除一个终止的容器。例如：

```
$ docker container rm  trusting_newton
trusting_newton
```
如果要删除一个运行中的容器，需要添加参数 `-f`。Docker 会发送一个 `SIGKILL`信号给容器。

**清理所有处于终止状态的容器**

用 `docker container ls -a` 命令可以查看所有已经创建的包括终止状态的容器，如果数量太多要一个个删除可能会很麻烦，用下面的命令可以清理掉所有处于终止状态的容器。

```
$ docker container prune
```
