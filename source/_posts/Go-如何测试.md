---
title: '[Go] 如何测试'
date: 2018-06-27 10:58:15
categories: Go
tags:
  - Go
  - 测试
---

Go 的标准库中自带 testing 包，用来测试 Go 的代码。下面介绍 Go Test 的使用。

测试的作用：**让代码可以测试，保证代码的正确性**

### Go 的测试代码

使用`testing`包对Go代码进行测试。Go的测试文件形式`code_test.go`。

```
import "testing"

func TestXxx(t *testing.T) {}
func ExampleXxx() {}
func BenchmarkXxx(b *testing.B) {}
```

* `TestXxx` 是测试用例。
* `ExampleXxx` 是代码使用用例。
* `BenchmarkXxx` 是基准测试用例。

<!--more-->

### Test 运行方式

`go test` 来运行后缀 `_test.go`的测试文件。

参数：

`-run` 值运行匹配的测试用例

```
go test -v -timeout 30s ktv_statistic/models -run ^Test_CreateKtvAlarmCheck$
// 只运行匹配 Test_getKtvID 的测试用例
```

`-v`  打印出日志到终端。
`-cover`显示包的测试覆盖率。

单个包运行测试

    go test package_path -v

注意：package_path 是 `GOPATH` 中包的路径，指定包路径的好处是可以测试包中的私有方法(private)

单个文件运行

    go test -v test_filepath

注意：推荐使用 VSCode 或 Goland 这样的 IDE 来运行测试，可以方便的进行单个、整个文件、整个包的测试运行。

### testdata 用来存在测试数据

测试文件夹下可以存放一些测试需要用到的测试数据。

### TestMain测试用例﻿

可以在测试运行前，运行后做一些操作，例如数据的建立和清除。

```
func TestMain(m *testing.M) {
	setUp()
	code := m.Run()
	teardown()
	os.Exit(code)
}
```

`setUp()`在测试前设置数据、配置等信息。
`teardown()`在测试结束后清除信息。

### 表驱动测试（Table driven test）

表测试是通过构建数据表来运行测试的一种方法。可以结合 subtest 来覆盖各种测试情况。

```
func TestOrderSrv_OrderSendToPay(t *testing.T) {
	room := Room{}
	db.First(&room)
	order := test_createOrder(t, room)

	type args struct {
		order *models.Order
	}
	tests := []struct {
		name    string
		args    args
		wantErr bool
	}{
		{
			name: "提交订单",
			args: args{
				order: order,
			},
			wantErr: false,
		},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			s := &OrderSrv{}
			if err := s.OrderSendToPay(tt.args.order); (err != nil) != tt.wantErr {
				t.Errorf("OrderSrv.OrderSendToPay() error = %v, wantErr %v", err, tt.wantErr)
			}
		})
	}
}
```

注意：VSCode 的 Go 插件支持生成测试用例模板。

### 测试并发

通过 `t.Parallel()`来调用函数中的 goroutine。

### 资源

制造假数据库的库：

https://github.com/bxcodec/faker
https://github.com/wawandco/fako
https://github.com/icrowley/fake

创建模型数据的库：

https://github.com/go-testfixtures/testfixtures
https://github.com/bluele/factory-go

测试文章：

https://about.sourcegraph.com/go/advanced-testing-in-go/

[go 测试，go test 工具的具体指令 flag | Deepzz’s Blog](https://deepzz.com/post/the-command-flag-of-go-test.html)

[Advanced Testing in Go](https://about.sourcegraph.com/go/advanced-testing-in-go/)

https://mp.weixin.qq.com/s?src=11&timestamp=1527603848&ver=906&signature=CFSB*cXv-1JlXMSRamF16KQcJn4OF1dZAXfq3yBuFmtxIl2mglp-2SX5a7YwDhE2mQu1qSTjwFXC8oQfQ3avYc-U-g1*vx**L-2UH8jNTW5Nuz-LCRqvI1AxiNOP25zZ&new=1

