---
title: Gorm 模型
date: 2018-06-10 10:48:48
categories:
  - GO
tags:
  - GO
  - ORM
---

模型定义

```
type User struct {
    gorm.Model
    Birthday     time.Time
    Age          int
    Name         string  `gorm:"size:255"` // 默认长度为 255,可以自定义
    Num          int     `gorm:"AUTO_INCREMENT"` // 自增长

    CreditCard        CreditCard
    // 一对一关联关系 (has one - use CreditCard's UserID as foreign key)
    Emails            []Email
    // 一对多关联关系 (has many - use Email's UserID as foreign key)

    BillingAddress    Address
    // 一对一关联关系 (belongs to - use BillingAddressID as foreign key)
    BillingAddressID  sql.NullInt64   // 外键

    ShippingAddress   Address
    // 一对一关联关系 (belongs to - use ShippingAddressID as foreign key)
    ShippingAddressID int

    IgnoreMe          int `gorm:"-"`   // 忽略这个字段
    Languages         []Language `gorm:"many2many:user_languages;"`
    // 多对多关联关系, 'user_languages' is join table
}

type Email struct {
    ID      int
    UserID  int     `gorm:"index"` // 外键，并创建索引
    Email   string  `gorm:"type:varchar(100);unique_index"`
    // `type` 是设置数据库类型的, `unique_index` 是创建唯一索引
    Subscribed bool
}

type Address struct {
    ID       int
    Address1 string         `gorm:"not null;unique_index"`
    // 字段不能为空，并且创建唯一索引
    Address2 string         `gorm:"type:varchar(100);unique_index"`
    Post     sql.NullString `gorm:"not null"`
}

type Language struct {
    ID   int
    Name string `gorm:"index:idx_name_code"`
    // 创建索引和定义索引名称，并且允许创建联合索引，在其他字段定义同样的索引名称，唯一索引也是同样操作
    Code string `gorm:"index:idx_name_code"` // `unique_index` 也是同样
}

type CreditCard struct {
    gorm.Model
    UserID  uint
    Number  string
}
```

### 模型约定
`gorm.Model` 结构体

`gorm.Model` 定义了四个字段，`ID`、`CreatedAt`、`UpdatedAt`、`DeletedAt`，可以将结构体嵌入到自己定义的结构体中，或者在自己定义的结构体中定义自己需要的字段。

```
// 模型的基本定义
type Model struct {
  ID        uint `gorm:"primary_key"`
  CreatedAt time.Time
  UpdatedAt time.Time
  DeletedAt *time.Time
}

// 加入 `ID`, `CreatedAt`, `UpdatedAt`, `DeletedAt`
type User struct {
  gorm.Model
  Name string
}

// 只定义 `ID`, `CreatedAt`
type User struct {
  ID        uint
  CreatedAt time.Time
  Name      string
}
```

**表名**

表名是结构体名称的复数形式

```
type User struct {} // default table name is `users`

// 自定义表名 `profiles`
func (User) TableName() string {
  return "profiles"
}

// 根据判断选择表
func (u User) TableName() string {
    if u.Role == "admin" {
        return "admin_users"
    } else {
        return "users"
    }
}

// 不使用复数形式设置表名
db.SingularTable(true)
```

#### 配置默认的表名
使用 `DefaultTableNameHandler` 来处理表名，设置默认规则

```
gorm.DefaultTableNameHandler = func (db *gorm.DB, defaultTableName string) string  {
    return "prefix_" + defaultTableName;
}
```

#### 字段名称是 snake case 规则
字段名也能自定义

```
type User struct {
  ID uint             // 字段名是 `id`
  Name string         // 字段名是 `name`
  Birthday time.Time  // 字段名是 `birthday`
  CreatedAt time.Time // 字段名是 `created_at`
}

// Overriding Column Name
type Animal struct {
    AnimalId    int64     `gorm:"column:beast_id"`         // 设置字段名为 `beast_id`
    Birthday    time.Time `gorm:"column:day_of_the_beast"`
    Age         int64     `gorm:"column:age_of_the_beast"`
}
```

#### `ID` 默认是主键

```
type User struct {
  ID   uint  // 字段 `ID` 默认是主键
  Name string
}

// 也能自定义主键
type Animal struct {
  AnimalId int64 `gorm:"primary_key"` // 设置 AnimalId 是主键
  Name     string
  Age      int64
}
```

`CreatedAt` 存储着记录的创建时间。
`UpdatedAt` 存储着记录的每次更新时间。
`DeletedAt` 存储着记录删除的时间，用于实现软删除的功能。

## 关联

### Belongs To
belongs_to 关联创建两个模型之间一对一的关系。

屏幕快照 2017-12-10 下午9.23.46

```
// `User` belongs to `Profile`, `ProfileID` is the foreign key
type User struct {
  gorm.Model
  Profile   Profile   // belongs_to 声明形式
  ProfileID int // belongs_to 声明形式，关联的外键ID
}

type Profile struct {
  gorm.Model
  Name string
}
```

**指定外键(foreign key)**

```
type Profile struct {
    gorm.Model
    Name string
}

type User struct {
    gorm.Model
    Profile      Profile `gorm:"ForeignKey:ProfileRefer"` // 使用 ProfileRefer 为外键
    ProfileRefer uint
}
```

**指定外键(foreign key)和关联键(association key)**

```
type Profile struct {
    gorm.Model
    Refer int   // 对应 User 表中的ID
    Name  string
}

type User struct {
    gorm.Model
    Profile   Profile `gorm:"ForeignKey:ProfileID;AssociationForeignKey:Refer"`
    ProfileID int
}
```

### Has One
has_one 关联也建立两个模型之间的一对一关系，但有所不同，这种关联表示模型拥有另一个模型。例如一个用户只有一个简述。

屏幕快照 2017-12-11 下午10.33.43

```
// User has one CreditCard, UserID 是外键
type User struct {
    gorm.Model
    CreditCard   CreditCard
}

type CreditCard struct {
    gorm.Model
    UserID   uint
    Number   string
}
```

指定外键

```
type Profile struct {
  gorm.Model
  Name      string
  UserRefer uint    // 外键字段
}

type User struct {
  gorm.Model
  Profile Profile `gorm:"ForeignKey:UserRefer"`
}
```

指定外键和关联键

```
type Profile struct {
  gorm.Model
  Name   string
  UserID uint
}

type User struct {
  gorm.Model
  Refer   uint
  Profile Profile `gorm:"ForeignKey:UserID;AssociationForeignKey:Refer"`
}
```

### Has Many
has_many 关联是建立两个模型的一对多的关系。

屏幕快照 2017-12-11 下午10.31.31

```
// User has many emails, UserID 是外键
type User struct {
    gorm.Model
    Emails   []Email
}

type Email struct {
    gorm.Model
    Email   string
    UserID  uint
}

db.Model(&user).Related(&emails)
//// SELECT * FROM emails WHERE user_id = 111; // 111 is user's primary key
```

ps. 一对多关联同样可以指定外键和关联键

### Many To Many
many_to_many 关联是两个模型通过一张中间表建立多对多关系。

```
// User has and belongs to many languages, use `user_languages` as join table
type User struct {
    gorm.Model
    Languages         []Language `gorm:"many2many:user_languages;"`
}

type Language struct {
    gorm.Model
    Name string
}
```

通过反推断

```
// User has and belongs to many languages, use `user_languages` as join table
// Make sure the two models are in different files
type User struct {
    gorm.Model
    Languages         []Language `gorm:"many2many:user_languages;"`
}

type Language struct {
    gorm.Model
    Name string
    Users               []User     `gorm:"many2many:user_languages;"`
}

db.Model(&language).Related(&users)
//// SELECT * FROM "users" INNER JOIN "user_languages" ON "user_languages"."user_id" = "users"."id" WHERE  ("user_languages"."language_id" IN ('111'))
```

### 多态
has_many 和 has_one 支持多态，belongs_to 和 many_to_many 不支持多态。

```
 type Cat struct {
    Id    int
    Name  string
    Toy   Toy `gorm:"polymorphic:Owner;"`
  }

  type Dog struct {
    Id   int
    Name string
    Toy  Toy `gorm:"polymorphic:Owner;"`
  }

  type Toy struct {
    Id        int
    Name      string
    OwnerId   int
    OwnerType string
  }
```
