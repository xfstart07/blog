---
title: '[Mac] Mac 常用软件笔记'
date: 2018-09-24 14:21:59
categories: Mac
tags:
  - Mac
  - brew
---

### 终端

Mac 上我常用的终端应用是：iTerm2。[下载地址](https://www.iterm2.com/downloads.html)

### iTerm2 使用

#### 快捷键

* 分屏竖屏：`command + d`
* 分屏横屏：`command + shift + d`
* 自动补全： `⌘;` `command + ;`
* 显示历史命令： `command + shift + h`
* Tab 搜索： `command + option + e`
* 高亮当前鼠标位置：`command + /`
* 显示配置 `command + ,`

<!-- more -->

#### Hotkey Window

设置的快捷键 `command + .`

![屏幕快照 2018-08-16 下午6.10.47](http://pa1so03xn.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-08-16%20%E4%B8%8B%E5%8D%886.10.47.png)

#### Profiles

有时候我们会管理很多机器，那么不可能记住那么多的信息。所有可以将配置记录在 Profiles 中。

profile 的配置：

* `Basis Name` 配置的名字
* `Tags` 配置支持分类
* `Badge` 标识
* `Command - Command` 运行的名字
* `Working Directory` 登录之后的工作目录

![屏幕快照 2018-08-16 下午6.31.50](http://pa1so03xn.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-08-16%20%E4%B8%8B%E5%8D%886.31.50.png)

**快捷键**

显示 Profile：`command + shift + o`

### 安装 brew

安装好终端的第一件事就是安装 brew，brew可以算的上 Mac 上的一个神器了。

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

### 安装 rmtrash

有时候在终端上会删除一些软件，但是可能也会误删除想要恢复，使用 rmtrash 是将文件删除到垃圾桶中，清楚垃圾桶才算是真正的删除。

```
brew install rmtrash
```
