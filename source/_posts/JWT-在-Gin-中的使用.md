---
title: JWT 在 Gin 中的使用
date: 2019-04-18 16:37:49
categories: Go
tags:
    - jwt
    - gin
---

## 介绍

JSON Web Token (JWT), 是为了在网络应用环境间传递声明而执行的一种基于JSON的开放标准（(RFC 7519).该 Token 被设计为紧凑且安全的，特别适用于分布式站点的单点登录（SSO）场景。JWT 的声明一般被用来在身份提供者和服务提供者间传递被认证的用户身份信息，以便于从资源服务器获取资源，也可以增加一些额外的其它业务逻辑所必须的声明信息，该 Token 也可直接被用于认证，也可被加密。

## 使用

### 安装

```bash
go get github.com/appleboy/gin-jwt
```

引入

```go
import "github.com/appleboy/gin-jwt"
```

我目前使用的版本是 `v2.5.0`.

<!-- more -->

### 创建中间件

设计 API 对象

```go
type API struct {
	App    *apps.App  // 业务对象
	Router *gin.Engine // 路由
	JWT    *jwt.GinJWTMiddleware // jwt 对象
}
```

中间件对象：

```go
api.JWT = &jwt.GinJWTMiddleware{
		Realm:      "gin jwt",
		Key:        []byte("secret key"),
		Timeout:    time.Hour,
		MaxRefresh: time.Hour,
		PayloadFunc: func(data interface{}) jwt.MapClaims {},
		Authenticator: func(c *gin.Context) (interface{}, error) {},
		Authorizator: func(data interface{}, c *gin.Context) bool {},
		Unauthorized: func(c *gin.Context, code int, message string) {},
		TokenLookup: "header: Authorization, query: token, cookie: jwt",
		// TokenLookup: "query:token",
		// TokenLookup: "cookie:token",
		TokenHeadName: "Bearer",
		TimeFunc: time.Now,
	}
```

* `Realm` JWT标识
* `Key` 服务端密钥
* `Timeout` token 过期时间
* `MaxRefresh` token 更新时间
* `PayloadFunc` 添加额外业务相关的信息
* `Authenticator` 在登录接口中使用的验证方法，并返回验证成功后的用户对象。
* `Authorizator` 登录后其他接口验证传入的 token 方法
* `Unauthorized` 验证失败后设置错误信息
* `TokenLookup` 设置 token 获取位置，一般默认在头部的 `Authorization` 中，或者    query的 token 字段，cookie 中的 jwt 字段。
* `TokenHeadName` Header中 token 的头部字段，默认常用名称 `Bearer`。
* `TimeFunc` 设置时间函数

### 注册阶段

在注册时如果要直接返回 token，那么可以调用 `TokenGenerator` 来生成 token。

```go
token, expire, err := c.JWT.TokenGenerator(strconv.Itoa(user.ID), *user)
```

`TokenGenerator` 的具体实现

```go
func (mw *GinJWTMiddleware) TokenGenerator(userID string, data interface{}) (string, time.Time, error) {
   // 根据签名算法创建 token 对象
	token := jwt.New(jwt.GetSigningMethod(mw.SigningAlgorithm))
	// 获取 claims
	claims := token.Claims.(jwt.MapClaims)

   // 设置业务中需要的额外信息
	if mw.PayloadFunc != nil {
		for key, value := range mw.PayloadFunc(data) {
			claims[key] = value
		}
	}

   // 过期时间
	expire := mw.TimeFunc().UTC().Add(mw.Timeout)
	claims["id"] = userID
	claims["exp"] = expire.Unix()
	claims["orig_iat"] = mw.TimeFunc().Unix()
	// 生成 token 
	tokenString, err := mw.signedString(token) 
	if err != nil {
		return "", time.Time{}, err
	}

	return tokenString, expire, nil
}
```

### 登录阶段

登录时会调用 `Authenticator` 注册的方法。

```go
func (api *API) LoginAuthenticator(ctx *gin.Context) (interface{}, error) {
	var params model.UserParams
	if err := ctx.Bind(&params); err != nil {
		return "", jwt.ErrMissingLoginValues
	}

   // 根据用户名获取用户
	user, err := api.App.GetUserByName(params.Username)
	if err != nil {
		return nil, err
	}

   // 验证密码
	if user.AuthPassword(params.Password) {
		return *user, nil
	}

	return nil, jwt.ErrFailedAuthentication
}
```

### 验证 Token

其他接口在设置了中间件 `Router.Use(api.JWT.MiddlewareFunc())` 后，通过调用 `Authorizator` 方法来验证。

```go
func (api *API) LoginedAuthorizator(data interface{}, c *gin.Context) bool {
	if id, ok := data.(string); ok {
		return api.App.IsExistUser(id)
	}
	return false
}
```

在业务 Hander 中可以通过方法 `jwt.ExtractClaims(ctx)` 来获取 payload 的信息。

## 深入

`gin-jwt` 依赖的 `jwt` 库叫做 `jwt-go`。下面来介绍一下这个库。

核心的 `Token` 结构：

```go
// A JWT Token.  Different fields will be used depending on whether you're
// creating or parsing/verifying a token.
type Token struct {
	Raw       string                 // The raw token.  Populated when you Parse a token
	Method    SigningMethod          // The signing method used or to be used
	Header    map[string]interface{} // The first segment of the token
	Claims    Claims                 // The second segment of the token
	Signature string                 // The third segment of the token.  Populated when you Parse a token
	Valid     bool                   // Is the token valid?  Populated when you Parse/Verify a token
}
```

这个`Token`结构体是用来生成 `jwt` 的 token。其中 `Method` 是用来表示签名使用的算法。`Header` 是头部`jwt`的信息，还有 `Claims` 记录额外的信息。

然后是生成签名的方法，key 是服务端的密钥。

```go
func (t *Token) SignedString(key interface{}) (string, error) {
	var sig, sstr string
	var err error
	// 将 Header 和 Claims 转换成字符串然后 base64 之后拼接在一起。
	if sstr, err = t.SigningString(); err != nil {
		return "", err
	}
	// 使用签名算法加密
	if sig, err = t.Method.Sign(sstr, key); err != nil {
		return "", err
	}
	return strings.Join([]string{sstr, sig}, "."), nil
}
```

解密 token 的对象叫做 `Parser`

```go
type Parser struct {}

// 主要解析方法
func (p *Parser) ParseWithClaims(tokenString string, claims Claims, keyFunc Keyfunc) (*Token, error) {}
```

Parser 除了验证 Token 外，还包括解码 Header 和 Claims 的内容。

## 资源

* https://jwt.io/introduction
* https://github.com/appleboy/gin-jwt
* https://github.com/dgrijalva/jwt-go
