---
title: "[go error] Go 程序报错：using unaddressable value"
date: 2018-06-10 11:19:49
categories:
  - Go
tags:
  - Go
---

一般出现 `using unaddressable value` 错误，表示传递的指针值不对，比如需要传递指针地址的，但是传了值。

例子：

```
func main() {
    db, _ := gorm.Open("mysql", "myproject:myproject@/myproject?charset=utf8&parseTime=True")

    db.AutoMigrate(Lorem{})
}
```

应该：

```
db.AutoMigrate(&Lorem{})
```
