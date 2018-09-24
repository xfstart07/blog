---
title: '[Devops] 基于 Gitlab+Jenkins+Ansible+Supervisor实现Go项目自动化部署'
date: 2018-09-24 14:01:24
categories: Devops
tags:
  - Devops
  - gitlab
  - jenkins
  - ansible
  - supervisor
  - go
---

### GitLab 安装

GitLab 安装方式可以参考官方给出的教程：https://about.gitlab.com/installation/

注：推荐使用 Docker 安装。

### GitLab CI/CD

参考：[搭建GitLab-CI](http://blog.weixinote.com/2018/06/10/%E6%90%AD%E5%BB%BAGitLab-CI/)

### GitLab 通知 Jenkins 部署

触发器参考：[Gitlab-提交代码触发-Jenkins-部署](http://blog.weixinote.com/2018/06/10/Gitlab-%E6%8F%90%E4%BA%A4%E4%BB%A3%E7%A0%81%E8%A7%A6%E5%8F%91-Jenkins-%E9%83%A8%E7%BD%B2/)

### Jenkins 安装 GitLab、Ansible 插件

* 安装插件的路径：系统管理 --> 插件管理

插件信息：

```
* GitLab
This plugin integrates GitLab to Jenkins by faking a GitLab CI Server.
* Ansible plugin
Invoke Ansible Ad-Hoc commands and playbooks.
```

![屏幕快照 2018-09-21 下午4.27.46](http://pa1so03xn.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-09-21%20%E4%B8%8B%E5%8D%884.27.46.png)


### Jenkins 创建部署项目

在 Jenkins 页面点击 `新建任务` 来创建一个需要部署的项目任务。

![屏幕快照 2018-09-21 下午4.31.00](http://pa1so03xn.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-09-21%20%E4%B8%8B%E5%8D%884.31.00.png)

#### 配置项目的功能工作路径(workspace)

点击 `General` 标签中的高级选项，使用自定义的工作空间，填入自定义工作空间路径：`/data/go/src/project`。

#### 源码管理

选择 Git 将 Gitlab 的项目路径和密钥信息填写好，并制定部署的分支信息。

#### 构建触发器

触发器参考：[Gitlab-提交代码触发-Jenkins-部署](http://blog.weixinote.com/2018/06/10/Gitlab-%E6%8F%90%E4%BA%A4%E4%BB%A3%E7%A0%81%E8%A7%A6%E5%8F%91-Jenkins-%E9%83%A8%E7%BD%B2/)

#### 构建

构建部署只要的作用可以是构建源码的二进制程序，发送到目标服务器，并在目标服务器运行起来程序。构建和运行程序可以直接将 shell 写在Jenkins或写在部署服务器的文件中，然后运行这个文件。或者通过 Ansible 来管理这些文件。

**使用 Ansible**

在`增加构建步骤` 选择 `Invoke Ansible Playbook` 选项，然后配置 playbook 文件的路径。

![屏幕快照 2018-09-21 下午4.45.47](http://pa1so03xn.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-09-21%20%E4%B8%8B%E5%8D%884.45.47.png)

注：构建步骤也支持多个部署，同时执行 Shell 和 Ansible，按照配置的顺序执行。

### Ansible 编写配置

我习惯将项目相关的配置文件统一放在 `/data/devops/projectname` 目录下。

模版

```yaml
---
- name: handlers projectname
  hosts: dev
  user: root
  tasks:
    - name: Creates directory
      file: path=/data/website/project/ state=directory
    - name: Creates directory config
      file: path=/data/website/project/config/ state=directory
    - name: Creates directory log
      file: path=/data/logs/project/ state=directory
    - name: copy app
      copy: src=/data/go/src/project/projectname dest=/data/website/project/ owner=root group=root mode=0755
    - name: copy config
      copy: src=/data/devops/project/config.dev.ini dest=/data/website/project/config/config.ini owner=root group=root mode=0644
    - name: copy supervisor config
      copy: src=/data/devops/project/supervisor.ini dest=/etc/supervisord.d/supervisor_project.ini owner=root group=root mode=0644
      notify: reread config
    - name: copy run
      copy: src=/data/devops/project/run.sh dest=/data/init_start/project.sh owner=root group=root mode=0755
    - name: copy supervisor sh
      copy: src=/data/devops/project/supervisor.sh dest=/data/sh/supervisor_project.sh owner=root group=root mode=0755
    - name: restart app
      supervisorctl: name=project state=restarted
    - name: copy lograte
      copy: src=/data/devops/project/logratate dest=/etc/logrotate.d/project owner=root group=root mode=0644
      notify: run lograte
  handlers:
    - name: run lograte
      shell: logrotate -f /etc/logrotate.d/project
```

一个项目包含的配置文件：

* supervisor.ini supervisor 的配置文件
* supervisor.sh supervisor 重启
* lograte 日志切割的配置文件

### Supervisor 配置

[[Linux] 应用进程管理工具 supervisord](mweblib://15306751696104)
