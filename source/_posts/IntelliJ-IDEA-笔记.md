---
title: IntelliJ IDEA 笔记
date: 2019-03-12 16:56:04
categories: Tools
tags: idea
---
# IntelliJ IDEA 笔记

## 常用快捷键

* ⌘ Command
* ⇧ Shift
* ⌥ Option
* ⌃ Control
* ↩︎ Return/Enter
* ⌫ Delete

### 文件相关快捷键：

* `Command + Shift + E`，打开最近浏览过的文件。

### General（通用）

* `Command + 1` Project
* `Command + 6` TODO
* `Command + 7` Structure
* `Command + 9` Version Control
* `Command + ,` 打开IDEA系统设置
* `Option + F12` Terminal

<!--more-->

### Editing（编辑）

* `Command + Option + L` 格式化代码
* `Command + /` 注释/取消注释与行注释
* `Command + +` ， `Command + -` 展开 / 折叠代码块
* `Shift+Enter`，可以向下插入新行，即使光标在当前行的中间。
* `Command + Option + L` 格式化代码
* `Command + Option + T` 包围代码（使用if..else, try..catch, for, synchronized等包围选中的代码）
* `Alt + Enter` 修改功能，对应是 `introduce local variable`

### Find（查找）

* `Command + Ctrl + G` 查找类似代码，并可以编辑

### Search/Replace（查询/替换）

* `Double Shift` Search Everywhere
* `Command + Shift + F` 全局替换（根据路径）
* `Command + Shift + R` 全局替换（根据路径）
* `Command + Shift + A` 搜索快捷键
* `Command + Option + O` 查找声明

### Go To

* `Command + B` **跳转到声明**
* `Command + [` **跳转回去**
* `Command + Option + B` **跳转到实现**
* `Command + Shift + T` **跳转到测试**(选择 Generate Function 可以自动生成 Table 测试)

### Refactor

* `Command + Option + V` 提取成变量
* `Command + Option + C` 提取成常量
* `Command + Option + M` **提取成方法或函数**

### Go 移除表示

在代码中使用 `Deprecated: 描述` 来表示代码需要移除。
 
```
// Deprecated: 将在下个版本删除
```

## 推荐插件

* IdeaVim
* Material Theme
* Protobuf Support
* Makefile support
* .ignore
* Ini
* BashSupport
* Markdown Navigator
* gruvbox Theme
    修改字体大小: Editor -> Color Scheme -> Color Scheme Font
    
## Live Template

在创建完模板之后，还需要配置模板中变量执行什么函数，用来获得想要的值。例如图片中：FIELD_NAME 配置执行的函数是 `snakeCase(fieldName())`，表示获取域中的名称并转换为下划线小写形式。

![屏幕快照 2019-03-12 下午4.51.25](http://blogcdn.weixinote.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-03-12%20%E4%B8%8B%E5%8D%884.51.25.png)

## 自定 TODO 标识

![1553565081656](http://blogcdn.weixinote.com/2020-01-09-1553565081656.jpg)

## Ref

* https://github.com/judasn/IntelliJ-IDEA-Tutorial
* [快捷键文档](http://wiki.jikexueyuan.com/project/intellij-idea-tutorial/keymap-mac-introduce.html)
* https://blog.csdn.net/dc_726/article/details/42784275

update: 2020-01-09