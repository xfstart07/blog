---
title: "[Go] JSON 的编码和解码"
date: 2018-06-10 11:22:36
categories:
  - GO
tags:
  - GO
  - JSON
---


在我们写 API 时都会碰到需要处理 JSON 的情况，所以总结一下处理 JSON 的方式。

首先定义结构：

``` GO
type User struct {
	ID        int       `json:"id"`
	Name      string    `json:"name"`
	Password  string    `json:"-"`
	CreatedAt time.Time `json:"created_at"`
	UpdatedAt time.Time `json:"updated_at"`
}
```

### 使用 json 包
在 Go 中处理 JSON 是使用 `encoding/json` 包，其中有两个方法 `Marshal` 和 `Unmarshal` 是用来编码和解码结构的。

当我们要将结构编码时是使用 `Marshal()` 方法：

``` Go
func Marshal(v interface{}) ([]byte, error)
```

``` go
data, err := json.Marshal(user)

// 解析出来的值
// data = []byte(`{"id":1,"name":"Leon","created_at":"2018-01-14T16:11:37.991176688+08:00","updated_at":"2018-01-14T16:11:37.991176752+08:00"}`)
```

<!--more-->

可以看到 `created_at` 解析出来的值是 `2018-01-14T16:03:22.009598441+08:00`，这是 Go 默认的时间格式，但是有时候我们需要的时间值可能不是这种格式，而是希望例如 `2018-01-14 16:03:22`格式或者返回一个时间戳值等，所以我们需要为类型重新实现2个接口 `Marshaler` 和 `Unmarshaler` 。

在 `encoding/json` 包中接口的定义：

``` go
type Marshaler interface {
        MarshalJSON() ([]byte, error)
}

type Unmarshaler interface {
        UnmarshalJSON([]byte) error
}
```

### 定义处理的接口方法

#### 编码
为我们的结构体，定义一个 `MarshalJSON()` 方法，用来处理结构体中需要处理的字段。

``` GO
func (u *User) MarshalJSON() (data []byte, err error) {
	type UserAlias User

	userStruct := &struct {
		CreatedAt string `json:"created_at"`
		UpdatedAt string `json:"updated_at"`
		*UserAlias
	}{
		UserAlias: (*UserAlias)(u),
	}

	userStruct.CreatedAt = u.CreatedAt.Format("2006-01-02 15:04:05")
	userStruct.UpdatedAt = u.UpdatedAt.Format("2006-01-02 15:04:05")

	return json.Marshal(userStruct)
}
```

首先我们为 User 结构体声明一个指针方法 `func (u *User) MarshalJSON() (data []byte, err error)`，然后重新声明一个 User的别名类型，目的是为了和 User 类型区别开来，不然我们最后在调用编码方法时会形成一个死循环，又重新调用 `MarshalJSON()` 方法了。

#### 解码
`json` 包中用来解码的方法是 `Unmarshal()`。

解码的方式也是同样的，定义一个结构体的方法 `UnmarshalJSON()`：

``` GO
func (u *User) UnmarshalJSON(data []byte) (err error) {
	type UserAlias User

	userStruct := &struct {
		CreatedAt string `json:"created_at"`
		UpdatedAt string `json:"updated_at"`
		*UserAlias
	}{
		UserAlias: (*UserAlias)(u),
	}

	if err = json.Unmarshal(data, &userStruct); err != nil {
		return err
	}

	u.CreatedAt, err = time.Parse("2006-01-02 15:04:05", userStruct.CreatedAt)
	if err != nil {
		return err
	}
	u.UpdatedAt, err = time.Parse("2006-01-02 15:04:05", userStruct.UpdatedAt)
	if err != nil {
		return err
	}

	return nil
}
```

首先我们定义了一个指针方法 `func (u *User) UnmarshalJSON(data []byte) (err error)`，然后重新定义了一个 User 的别名类型，然后将 User 的数据赋值给别名类型的值。然后处理需要格式化的字段。

**完整代码:**

``` Go
package main

import (
	"encoding/json"
	"fmt"
	"time"
)

const (
	timeStandardLayout = "2006-01-02 15:04:05"
)

type User struct {
	ID        int       `json:"id"`
	Name      string    `json:"name"`
	Password  string    `json:"-"`
	CreatedAt time.Time `json:"created_at"`
	UpdatedAt time.Time `json:"updated_at"`
}

func (u *User) MarshalJSON() (data []byte, err error) {
	type UserAlias User

	userStruct := &struct {
		CreatedAt string `json:"created_at"`
		UpdatedAt string `json:"updated_at"`
		*UserAlias
	}{
		UserAlias: (*UserAlias)(u),
	}

	userStruct.CreatedAt = u.CreatedAt.Format(timeStandardLayout)
	userStruct.UpdatedAt = u.UpdatedAt.Format(timeStandardLayout)

	return json.Marshal(userStruct)
}

func (u *User) UnmarshalJSON(data []byte) (err error) {
	type UserAlias User

	userStruct := &struct {
		CreatedAt string `json:"created_at"`
		UpdatedAt string `json:"updated_at"`
		*UserAlias
	}{
		UserAlias: (*UserAlias)(u),
	}

	if err = json.Unmarshal(data, &userStruct); err != nil {
		return err
	}

	u.CreatedAt, err = time.Parse(timeStandardLayout, userStruct.CreatedAt)
	if err != nil {
		return err
	}
	u.UpdatedAt, err = time.Parse(timeStandardLayout, userStruct.UpdatedAt)
	if err != nil {
		return err
	}

	return nil
}

func main() {
	user := &User{
		ID:        1,
		Name:      "Leon",
		Password:  "",
		CreatedAt: time.Now(),
		UpdatedAt: time.Now(),
	}

	data, err := json.Marshal(&user)
	fmt.Println("编码", string(data), err)

	user2 := &User{}
	err = json.Unmarshal(data, &user2)
	if err != nil {
		fmt.Println(err)
	}
	fmt.Printf("解码 %+v\n", user2)
}
```

**注意**：这里我们为User类型定义的方法都是指针方法，所以需要注意的是User类型的值必须是指针类型，这样才能调用我们自定义的方法。

### 知识点

* json
* 指针方法
* 值方法
* 结构体
* 接口