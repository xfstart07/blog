---
title: GO HTTP测试
date: 2019-12-19 16:55:01
categories: Go
tags:
    - Go
    - test
---

当我们在写接口的时候避免不了的需要对接口进行测试。

### 标准库 net/http/httptest

这个包提供了 HTTP 测试使用工具。

#### Server 

通过`NewServer(handler http.Handler)`方法可以创建HTTP服务对象 `Server`。这样在创建时将需要测试的接口 `handler` 传入，我们就可以测试这个接口了。

还有 `NewTLSServer` 创建使用TLS的服务器。

<!--more-->

```go
func Get(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "Hello, client")
}

func Test_Get(t *testing.T) {
    ts := httptest.NewServer(http.HandlerFunc(Get))
    defer ts.Close()
    
    res, err := http.Get(ts.URL)
    if err != nil {
    	t.Fatal(err)
    }
    greeting, err := ioutil.ReadAll(res.Body)
    res.Body.Close()
    if err != nil {
    	t.Fatal(err)
    }
}
```

#### ResponseRecorder 和 NewRequest()

当使用类似 Gin 这类框架提供接口，而非 `net/http`。就需要使用 `ResponseRecorder`  配合 `NewRequest()` 来创建请求服务。


```go
router := gin.Default()

params := `{"code":1,"codemsg":"success"}`
w := httptest.NewRecorder()
req := httptest.NewRequest("POST", "/products", bytes.NewBuffer(params))
router.ServeHTTP(w, req)

result, _ := ioutil.ReadAll(w.Body)
t.Log("Resp Body", string(result))
if w.Code != http.StatusCreated {
	t.Errorf("状态不对, got = %v, want = %v", w.Code, http.StatusCreated)
}
```

### 外部接口请求

有时我们会调用到外部提供的接口，但是外部接口我们假设是不稳定的，如果直接在测试时发起请求不一定返回正确信息。这时我们就需要模拟这个外部接口。

使用库：

```go
import "github.com/jarcoal/httpmock"
```

例子：

```go
httpmock.Activate()
	defer httpmock.DeactivateAndReset()

	// our database of articles
	articles := make([]map[string]interface{}, 0)

	// mock to list out the articles
	httpmock.RegisterResponder("GET", "https://api.mybiz.com/articles",
		func(req *http.Request) (*http.Response, error) {
			resp, err := httpmock.NewJsonResponse(200, articles)
			if err != nil {
				return httpmock.NewStringResponse(500, ""), nil
			}
			return resp, nil
		},
	)
```

需要注意的`httpmock.DeactivateAndReset()` 声明中默认使用的是 `http.Client`，在测试发起请求时也同样需要使用 `http.Client`，否则请求无法拦截。

当然也可以自定义 `http.Client`，例如：`httpmock.ActivateNonDefault(http_api.GetClient())`。

### Ref

* https://github.com/jarcoal/httpmock