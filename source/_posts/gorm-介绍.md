---
title: GORM 介绍
date: 2018-06-09 16:41:28
categories:
  - Go
tags:
  - GO
  - ORM
---

文档路径:

http://gorm.io/

### 数据库使用
MySQL 连接

```
import (
    "github.com/jinzhu/gorm"
    _ "github.com/jinzhu/gorm/dialects/mysql"
)

func main() {
  db, err := gorm.Open("mysql", "user:password@/dbname?charset=utf8&parseTime=True&loc=Local")
  defer db.Close()
}
```

ps. gorm 还支持 `sqlite3`, `postgreSQL`, `SQL server` 等数据库

### 迁移 Migrate
`Auto Migration` 比较简单，目前只支持创建表的操作，不支持在建表后修改字段、索引的操作。

```
db.AutoMigrate(&User{})

db.AutoMigrate(&User{}, &Product{}, &Order{}) // 多个表
```

ps. 建议 `Auto Migration` 只用户创建表，添加修改字段、索引，可以创建 migrate 文件来单独执行。

`Has Table` 判断表是否存在

```
// Check model `User`'s table exists or not
db.HasTable(&User{})
// Check table `users` exists or not
db.HasTable("users")
```

`Create Table` 建表

```
// Create table for model `User`
db.CreateTable(&User{})
```

`Drop table` 删表

```
// Drop model `User`'s table
db.DropTable(&User{})
```

`ModifyColumn` 修改字段

```
db.Model(&User{}).ModifyColumn("description", "text")
```

`DropColumn` 删除字段

```
db.Model(&User{}).DropColumn("description")
```

`Add Foreign Key` 添加外键

```
// 第一个参数 : 外键名称
// 第二个参数 : 对应的外键表
// 第三个参数 : ONDELETE 删除约束
// 第四个参数 : ONUPDATE 更新约束
db.Model(&User{}).AddForeignKey("city_id", "cities(id)", "RESTRICT", "RESTRICT")
```

`Indexes` 索引的添加和删除

```
// Add index for columns `name` with given name `idx_user_name`
db.Model(&User{}).AddIndex("idx_user_name", "name")

// Add index for columns `name`, `age` with given name `idx_user_name_age`
db.Model(&User{}).AddIndex("idx_user_name_age", "name", "age")

// Add unique index
db.Model(&User{}).AddUniqueIndex("idx_user_name", "name")

// Add unique index for multiple columns
db.Model(&User{}).AddUniqueIndex("idx_user_name_age", "name", "age")

// Remove index
db.Model(&User{}).RemoveIndex("idx_user_name")
```