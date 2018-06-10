---
title: Go Gin 允许跨域访问
date: 2018-06-10 10:46:55
categories:
  - GO
tags:
  - GO
  - Gin
---

上周五的时候，给接口添加了支持跨域访问，所有做一下跨域方面的笔记。

### HTTP 访问控制(CORS)
当一个资源从与该资源本身所在的服务器**不同的域或端口**请求一个资源时，资源会发起一个跨域 HTTP 请求。

出于安全原因，浏览器限制从脚本内发起的跨域 HTTP 请求。例如，XMLHttpRequest 和 Fetch API 遵循**同源策略**。这意味着使用这些 API 的 Web 应用程序只能从加载应用程序的同一个域请求 HTTP 资源，除非使用 CORS 头文件。

跨域资源共享标准：规范要求，对那些可能对服务器数据产生副作用的 HTTP 请求方法（特别是 GET 以外的 HTTP 请求，或者搭配某些 MIME 类型的 POST 请求），浏览器必须首先使用 `OPTIONS` 方法发起一个预检请求（preflight request），从而获知服务端是否允许该跨域请求。服务器确认允许之后，才发起实际的 HTTP 请求。在预检请求的返回中，服务器端也可以通知客户端，是否需要携带身份凭证（包括 Cookies 和 HTTP 认证相关数据）。

ps. 上面是 MDN 的文档介绍。

### Gin 添加跨域的中间件
Gin 是一个Go 开发的 Web 框架，给 Gin 加 CORS 的支持也是非常简单的。

#### CORS 中间件插件
这是 Gin 官方的 CORS 中间件
https://github.com/gin-contrib/cors

#### 使用 CORS
引入包

```
import "github.com/gin-contrib/cors"
```

允许所有源的配置

```
func main() {
	router := gin.Default()
	// same as
	// config := cors.DefaultConfig()
	// config.AllowAllOrigins = true
	// router.Use(cors.New(config))
	router.Use(cors.Default())
	router.Run()
}
```

自定义源的配置

```
config := cors.DefaultConfig()
config.AllowOrigins = []string{"http://google.com"}
config.AddAllowOrigins("http://facebook.com")
```

自定义其他配置:

```
cors.Config{
		AllowOrigins:     []string{"https://foo.com"},
		AllowMethods:     []string{"PUT", "PATCH"},
		AllowHeaders:     []string{"Origin"},
		ExposeHeaders:    []string{"Content-Length"},
		AllowCredentials: true,
		AllowOriginFunc: func(origin string) bool {
			return origin == "https://github.com"
		},
		MaxAge: 12 * time.Hour,
}
```

主要的几个配置介绍：

* AllowOrigins 允许源列表
* AllowAllOrigins 允许所有源，返回的格式 `Access-Control-Allow-Origin: *`
* AllowMethods 允许的方法列表
* AllowHeaders 允许的头部信息
* AllowCredentials 允许暴露请求的响应，`Access-Control-Allow-Credentials: true`

## 资源
https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS

https://developer.mozilla.org/en-US/docs/Web/HTTP/Server-Side_Access_Control

https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Access-Control-Allow-Credentials

https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Authentication

https://github.com/gin-contrib/cors

