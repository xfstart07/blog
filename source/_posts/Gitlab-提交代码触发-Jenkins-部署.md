---
title: "[Devops] Gitlab 提交代码触发 Jenkins 部署"
date: 2018-06-10 11:32:30
categories:
  - Devops
tags:
  - Devops
  - Gitlab
  - Jenkins
---


### 背景
在开发过程中我们经常会需要提交代码然后部署，在使用 Gitlab 和 Jenkins 的条件下，可以一步完成代码的提交到部署。

Update(20180511): 之前使用的触发方法存在提交时多个分支同时部署的情况，没有很好的区别分支部署。所以重新修改用“Build when a change is pushed to GitLab” 选项来配置部署。

### Jenkins 的配置
首先需要安装两个插件 `Gitlab Hook Plugin` 和 `Build Authorization Token Root Plugin`

{% img http://pa1so03xn.bkt.clouddn.com/images/0_KN6aqOdu6uPdI-Jc.png %}

#### 创建项目触发器
配置路径：项目-配置-构建触发器

勾选 “Build when a change is pushed to GitLab” 选项。在选项的右边有个 URL，将这个 URL 拷贝到 GitLab 去的。

{% img http://pa1so03xn.bkt.clouddn.com/images/1_KbZxFQjuU_N0ka7sv0_dKg.png %}

##### 指定触发的分支
配置需要注意的是高级选项，点击高级选项，在 `Allowed branches` 的 `Filter branches by name` Include 中填写部署的指定分支。然后点击 `Secret Token` 后面的 Genrate。

{% img http://pa1so03xn.bkt.clouddn.com/images/1_xsQ5athArRE_hd-MfoaBxg.png %}

### 配置 GitLab
首先在 GitLab 项目中创建一个 `webhook`，`webhook`的配置：

将 Jenkins 的 “Build when a change is pushed to GitLab” 中的 URL 拷贝过来。`Secret Token` 把 Jenkins 中生成的拷贝过来。

```
http://192.168.4.71:8080/project/项目名
```

{% img http://pa1so03xn.bkt.clouddn.com/images/1_52hlIr7tlR7PnwJOhMrMSw.png %}

GitLab 提供了配置后 Test 功能，可以使用 Test 测试一下配置是否有效。

### 使用
在配置完 `jenkins` 和 `Gitlab` 后，我们就可以来提交代码然后去`jenkins`看是否部署了。

除了提交代码外，`Merge Request`也能配置来触发部署。
