---
title: '[DevOps] Ansible 入门介绍'
date: 2018-09-24 13:50:45
categories: Devops
tags:
  - Devops
  - ansible
---


Ansilbe 是一个部署一群远程主机的工具。远程的主机可以是远程虚拟机或物理机， 也可以是本地主机。

**Ansible 能做什么？**

Ansilbe 通过 SSH 协议实现远程节点和管理节点之间的通信。理论上说，只要管理员通过 ssh 登录到一台远程主机上能做的操作，Ansible 都可以做到。

包括：

* 拷贝文件
* 安装软件包
* 启动服务

<!-- more -->

## 安装

```
pip install ansible
```

如果是 centos 系统也可以通过 `yum install ansible -y` 安装。

配置文件路径：

```
/etc/ansible/
```

**注**: 在 mac 中安装，并没有 `/etc/ansible` 文件夹，可以自己创建一个，然后去 github 上下载一个[配置](https://github.com/ansible/ansible/blob/devel/examples/ansible.cfg)。

## 使用

### Host

ansible 管理的主机

```
192.168.1.50
aserver.example.org
bserver.example.org

[webservers]
foo.example.com
bar.example.com
```

host 支持带分类的方式。

### 使用命令

Ansible 提供了一个命令行工具，在官方文档中起给命令行起了一个名字叫 `Ad-Hoc Commands`。

ansible命令的格式是：

```
ansible <host-pattern> [options]
```

下面简单的看一下 ansible 的命令如何使用：

ping 一下 `web` 组的所有主机：

```
ansible web -m ping

192.168.4.51 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

### 使用 Playbook

为了避免重复输入命令，Ansible 提供脚本功能。Ansible 脚本的名字叫 Playbook，使用的是 YAML 的格式，文件以yml结尾。

执行脚本 playbook 的方法

```
ansible-playbook deploy.yml
```

#### playbook的例子

```yml
---
- hosts: web
  vars:
    http_port: 80
    max_clients: 200
  remote_user: root
  tasks:
  - name: ensure apache is at the latest version
    yum: pkg=httpd state=latest

  - name: Write the configuration file
    template: src=templates/httpd.conf.j2 dest=/etc/httpd/conf/httpd.conf
    notify:
    - restart apache

  - name: Write the default index.html file
    template: src=templates/index.html.j2 dest=/var/www/html/index.html

  - name: ensure apache is running
    service: name=httpd state=started
  handlers:
    - name: restart apache
      service: name=httpd state=restarted
```

deploy.yml 的功能为 web主机部署 apache, 其中包含以下部署步骤：

1. 安装apache包；
2. 拷贝配置文件httpd，并保证拷贝文件后，apache服务会被重启；
3. 拷贝默认的网页文件index.html；
4. 启动apache服务；

`playbook deploy.yml` 包含下面几个关键字，每个关键字的含义：

* hosts：为主机的IP，或者主机组名，或者关键字all
* remote_user: 以哪个用户身份执行。
* vars： 变量
* tasks: playbook的核心，定义顺序执行的动作action。每个action调用一个`ansbile module`。
* handers： 是playbook的event，默认不会执行，在action里触发才会执行。多次触发只执行一次。

> action 语法： module： module_parameter=module_value
> 常用的 **module** 有yum、copy、template等，module在ansible的作用，相当于bash脚本中yum，copy这样的命令。下一节会介绍。

### 什么是 Ansible Module？

bash 无论在命令行上执行，还是bash脚本中，都需要调用 `cd、ls、copy、yum` 等命令；
module 就是Ansible的“命令”，module是ansible命令行和脚本中都需要调用的。常用的 Ansible module 有 yum、copy、template等。

在bash，调用命令时可以跟不同的参数，每个命令的参数都是该命令自定义的；同样，ansible中调用module也可以跟不同的参数，每个module的参数也都是由module自定义的。

每个module的用法可以查阅文档。http://docs.ansible.com/ansible/modules_by_category.html

#### Ansible 在命令行里使用 Module

在命令行中

* `-m` 后面接调用module的名字
* `-a` 后面接调用module的参数

```
$ # 使用module copy拷贝管理员节点文件/etc/hosts到所有远程主机 /tmp/hosts
$ ansible all -m copy -a "src=/etc/hosts dest=/tmp/hosts"
$ # 使用module yum在远程主机web上安装httpd包
$ ansible web -m yum -a "name=httpd state=present"
```

#### Ansible 在 Playbook 脚本使用 Module

在 playbook 脚本中，tasks 中的每一个 action 都是对 module 的一次调用。在每个action中：

* 冒号前面是module的名字
* 冒号后面是调用module的参数

```
---
  tasks:
  - name: ensure apache is at the latest version
    yum: pkg=httpd state=latest
  - name: write the apache config file
    template: src=templates/httpd.conf.j2 dest=/etc/httpd/conf/httpd.conf
  - name: ensure apache is running
    service: name=httpd state=started
```

### 常用几个 Module

**调试和测试类的module**

* ping - ping一下你的远程主机，如果可以通过ansible成功连接，那么返回pong
* debug - 用于调试的module，只是简单打印一些消息，有点像linux的echo命令。

**文件类的module**

* copy - 从本地拷贝文件到远程节点
* template - 从本地拷贝文件到远程节点，并进行变量的替换
* file - 设置文件的属性

**linux上常用的操作**

* user - 管理用户账户
* yum - red hat系linux上的包管理
* service - 管理服务
* firewalld - 管理防火墙中的服务和端口

**执行Shell命令**

* shell - 在节点上执行shell命令，支持$HOME和”<”, “>”, “|”, “;” and “&”
* command - 在远程节点上面执行命令，不支持$HOME和”<”, “>”, “|”, “;” and “&”

变量使用 `\{\{\}\}`，例如： `\{\{ http_port \}\}`

## Tasks 列表说明

每一个 play 包含了一个 task 列表（任务列表）。一个 task 在其所对应的所有主机上（通过 host pattern 匹配的所有主机）执行完毕之后，下一个 task 才会执行。
有一点需要明白的是（很重要）,在一个 play 之中,所有 hosts 会获取相同的任务指令,这是 play 的一个目的所在,也就是将一组选出的 hosts 映射到 task.

> Each play contains a list of tasks. Tasks are executed in order, one at a time, against all machines matched by the host pattern, before moving on to the next task. It is important to understand that, within a play, all hosts are going to get the same task directives. It is the purpose of a play to map a selection of hosts to tasks.

## Handlers: 在发生改变时执行的操作

（当发生改动时）’notify’ actions 会在 playbook 的每一个 task 结束时被触发,而且即使有多个不同的 task 通知改动的发生, ‘notify’ actions 只会被触发一次.

Handlers 也是一些 task 的列表,通过名字来引用,它们和一般的 task 并没有什么区别.
Handlers 是由通知者进行 notify, 如果没有被 notify,handlers 不会执行.不管有多少个通知者进行了 notify,等到 play 中的所有 task 执行完成之后,handlers 也只会被执行一次.

### 按Handler的定义顺序执行

handlers是按照在handlers中定义个顺序执行的，而不是安装notify的顺序执行的。

下面的例子定义的顺序是1>2>3，notify的顺序是3>2>1，实际执行顺序：1>2>3.


```yml
---
- hosts: lb
  remote_user: root
  gather_facts: no
  vars:
      random_number1: "\{\{ 10000 | random \}\}"
      random_number2: "\{\{ 10000000000 | random \}\}"
  tasks:
  - name: Copy the /etc/hosts to /tmp/hosts.\{\{ random_number1 \}\}
    copy: src=/etc/hosts dest=/tmp/hosts.\{\{ random_number1 \}\}
    notify:
      - define the 3nd handler
  - name: Copy the /etc/hosts to /tmp/hosts.\{\{ random_number2 \}\}
    copy: src=/etc/hosts dest=/tmp/hosts.\{\{ random_number2 \}\}
    notify:
      - define the 2nd handler
      - define the 1nd handler

  handlers:
  - name: define the 1nd handler
    debug: msg="define the 1nd handler"
  - name: define the 2nd handler
    debug: msg="define the 2nd handler"
  - name: define the 3nd handler
    debug: msg="define the 3nd handler"
```

# 资源

* https://github.com/ansible/ansible
* https://docs.ansible.com/
* https://getansible.com/
* https://ansible-tran.readthedocs.io/en/latest/index.html
* https://blog.csdn.net/minxihou/article/details/53422492