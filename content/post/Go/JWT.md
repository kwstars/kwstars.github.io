---
title: "在Golang中实现基于JWT的身份验证"
date: 2019-10-13T19:53:26+08:00
draft: false
tags: ["JWT"]
categories: ["Go"]
---

本文翻译自[《Implementing JWT based authentication in Golang》](https://www.sohamkamani.com/golang/jwt-authentication/)。全文如下：

认证是让您的应用程序知道`向您的应用程序发送的请求`一定是`请求人`发送的。JSON网络令牌(JWT)是一种允许认证的方法，它不需要在系统本身存储任何关于用户的信息（与基于[session based authentication](https://www.sohamkamani.com/blog/2018/03/25/golang-session-authentication/)相反）。

在这篇文章中，我们将演示基于JWT的身份验证是如何工作的，以及如何在Go中构建一个示例应用程序来实现它。

> 如果你已经知道JWT是如何工作的，只是想看看它的实现，你可以跳过前面的内容，或者在[Github上看看源代码](https://github.com/sohamkamani/jwt-go-example)。



## JWT的格式

假设我们有一个叫user1的用户，他们试图登录一个应用程序或网站。一旦成功，他们会收到一个类似这样的令牌。

```jwt
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InVzZXIxIiwiZXhwIjoxNTQ3OTc0MDgyfQ.2Ye5_w1z3zpD4dSGdRp3s98ZipCNQqmsHRB9vioOx54
```



这是一个JWT，由三部分组成（用.分隔）。

1. 第一部分是`header`(eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9)。头部指定了用于生成签名的算法等信息（第三部分）。这一部分是非常标准的，对于任何使用相同算法的JWT都是一样的。

2. 第二部分是`payload`（eyJ1c2VybmFtZSI6InVzZXIxIiwiZXhwIjoxNTQ3OTc0MDgyfQ），其中包含应用的特定信息（在我们的例子中，这是用户名），以及有关令牌的到期时间和有效性的信息。

3. 第三部分是`signature`（2Ye5_w1z3zpD4dSGdRp3s98ZipCNQqmsHRB9vioOx54）。它是由前两部分和一个秘密密钥组合和散列生成的。

   

现在有趣的是，`header`和`payload`没有加密。它们只是base64编码。这意味着，任何人都可以通过解码来查看它们的内容。

例如，我们可以使用这个[在线工具](https://jwt.io/)，并解码`header`或`payload`。



`header`的内容：

```json
{ "alg": "HS256", "typ": "JWT" }
```



> 如果您使用的是Linux或Mac OS，则还可以在终端上执行以下语句：
>
> echo eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9 | base64 -d



类似地，`payload`的内容为：

```json
{ "username": "user1", "exp": 1547974082 }
```



## JWT签名如何工作

因此，如果JWT的头和签名可以被任何人读取和写入，那么实际上是什么使JWT安全？答案在于最后一部分（签名）是如何生成的。

假设你是一个应用程序，想要向一个已经成功登录的用户（例如，user1）发出JWT。

制作`header`和`payload`是非常简单的。`header`或多或少是固定的，而`payload`是JSON对象是通过设置用户ID和到期时间（以unix毫秒为单位）形成的。

签署令牌的应用还会有一个密钥，这是一个秘密值，只有应用自己知道。然后，`header`和`payload`的base64表示与秘钥相结合，然后通过散列算法（在本例中是`HS256`，正如头中提到的那样

![image-20210413202518511](https://i.imgur.com/Um7Q9Zv.png)

关于算法如何实现的细节不在本篇文章的讨论范围内，但要注意的是，它是单向的，这意味着我们无法逆转算法并获得用于制作签名的组件......所以我们的秘钥仍然是秘密的。



## 验证JWT

为了核实接收的JWT，将再次使用接收的JWT的`header`和`payload`以及秘钥生成的`signature`。如果签名与JWT上的签名相匹配，那么JWT被认为是有效的。

现在让我们假设你是一个黑客，试图发行一个假令牌。您可以轻松地生成`header`和`payload`，但如果不知道密钥，就无法生成有效的签名。如果你试图篡改有效JWT的现有`payload`，`signature`将不再匹配。

![image-20210413202841049](https://i.imgur.com/emQz5ZL.png)

这样，JWT可以以一种安全的方式授权用户，而无需在应用程序服务器上实际存储任何信息（除了密钥）。



## 在Go中实现

现在，我们已经了解了基于JWT的身份验证的工作原理，让我们使用Go来实现它。



### 创建HTTP服务器

首先，使用所需的路由初始化HTTP服务器：

```go
package main

import (
	"log"
	"net/http"
)

func main() {
	// "Signin "和 "Welcome "是我们要实现的处理程序。
	http.HandleFunc("/signin", Signin)
	http.HandleFunc("/welcome", Welcome)
	http.HandleFunc("/refresh", Refresh)

	// 启动8000端口的服务器
	log.Fatal(http.ListenAndServe(":8000", nil))
}
```

现在，我们可以定义`Signin`和`Welcome`routes。



### 处理用户登录

`/signin`路由将获取用户凭据并登录。为简化起见，我们将用户信息存储在内存：

```go
var users = map[string]string{
	"user1": "password1",
	"user2": "password2",
}
```



因此，目前，我们的应用程序中只有两个有效用户：`user1`和`user2`。接下来，我们可以编写`Signin` HTTP处理程序。对于此示例，我们使用`dgrijalva/jwt-go`库来帮助我们创建和验证JWT令牌。

```go
import (
    //...
    // 导入 jwt-go 
	"github.com/dgrijalva/jwt-go"
	//...
)

// 用于signature的JWT密钥
var jwtKey = []byte("my_secret_key")

var users = map[string]string{
	"user1": "password1",
	"user2": "password2",
}

// 创建一个结构来读取请求body中的用户名和密码。
type Credentials struct {
	Password string `json:"password"`
	Username string `json:"username"`
}

// 创建一个将被编码为JWT的结构(payload)。我们添加jwt.StandardClaims作为嵌入式类型，以提供诸如到期时间等字段。
type Claims struct {
	Username string `json:"username"`
	jwt.StandardClaims
}

// 创建Signin处理程序
func Signin(w http.ResponseWriter, r *http.Request) {
	var creds Credentials
	// 获取 JSON body并解码到Credentials
	err := json.NewDecoder(r.Body).Decode(&creds)
	if err != nil {
		// 如果body错误返回http错误
		w.WriteHeader(http.StatusBadRequest)
		return
	}

	// 从内存中获取期望的密码
	expectedPassword, ok := users[creds.Username]

    // 如果给定的用户存在一个密码并且它与我们收到的密码相同，我们可以继续前进，如果不是，那么我们返回一个 "未授权 "状态。
	if !ok || expectedPassword != creds.Password {
		w.WriteHeader(http.StatusUnauthorized)
		return
	}

	// 在这里声明到期时间，我们将其保留为5分钟。
	expirationTime := time.Now().Add(5 * time.Minute)
    // 创建JWT Claims，其中包括用户名和到期时间。
	claims := &Claims{
		Username: creds.Username,
		StandardClaims: jwt.StandardClaims{
			// 在JWT中，到期时间用unix毫秒表示。
			ExpiresAt: expirationTime.Unix(),
		},
	}

    // 通过签名算法和claims创建token(未经过hash的JWT,可以理解成header+payload)。
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
	// 生成的JWT 
	tokenString, err := token.SignedString(jwtKey)
	if err != nil {
		// 如果在创建JWT时出现错误，则返回一个内部服务器错误。
		w.WriteHeader(http.StatusInternalServerError)
		return
	}

	// 最后，我们将 "token "的客户端cookie设置为我们刚刚生成的JWT。
    // 我们还设置了一个到期时间，这个到期时间和令牌本身是一样的
	http.SetCookie(w, &http.Cookie{
		Name:    "token",
		Value:   tokenString,
		Expires: expirationTime,
	})
}
```



如果用户用正确的凭证登录，该处理程序将在客户端设置一个带有JWT值的cookie。一旦在客户端上设置了cookie，它就会随着每一个请求一起发送。现在我们可以编写我们的欢迎处理程序来处理用户的特定信息。



### 处理认证后的路由

现在，所有已登录的客户端都将会话信息以cookie的形式存储在他们的终端，我们可以用它来。

1. 验证后续的用户请求
2. 获取提出请求的用户的信息

让我们写下我们的`Welcome`处理程序来实现这一点。

```go
func Welcome(w http.ResponseWriter, r *http.Request) {
	// 我们可以从每次请求时的请求cookie中获取会话令牌。
	c, err := r.Cookie("token")
	if err != nil {
		if err == http.ErrNoCookie {
			// 如果cookie没有被设置，则返回一个未经授权的状态。
			w.WriteHeader(http.StatusUnauthorized)
			return
		}
		// 对于任何其他类型的错误，返回一个坏的请求状态。
		w.WriteHeader(http.StatusBadRequest)
		return
	}

	// 从cookie中获取JWT字符串
	tknStr := c.Value

	// 初始化一个新的 "Claims "实例。
	claims := &Claims{}

	// 解析JWT字符串，并将结果存储在`claims`.注意，我们在这个方法中也传递了密钥。如果token无效（如果根据我们在登录时设置的到期时间，它已经过期了），或者如果签名不匹配，这个方法将返回一个错误。
	tkn, err := jwt.ParseWithClaims(tknStr, claims, func(token *jwt.Token) (interface{}, error) {
		return jwtKey, nil
	})
	if err != nil {
		if err == jwt.ErrSignatureInvalid {
			w.WriteHeader(http.StatusUnauthorized)
			return
		}
		w.WriteHeader(http.StatusBadRequest)
		return
	}
	if !tkn.Valid {
		w.WriteHeader(http.StatusUnauthorized)
		return
	}

	// 最后，向用户返回欢迎信息，以及用户在token中给出的用户名。
	w.Write([]byte(fmt.Sprintf("Welcome %s!", claims.Username)))
}
```



### 续签您的令牌

在这个例子中，我们设置了一个很短的过期时间，即5分钟。如果用户的token过期，用户就需要每五分钟登录一次。为了解决这个问题，我们将创建另一个`/refresh`路由，该路由使用之前的令牌（仍然有效），并返回一个更新过期时间的新令牌。

> 为了最大限度地减少对JWT的误用，过期时间通常保持在几分钟左右。通常情况下，客户端应用程序会在后台刷新令牌。

```go
func Refresh(w http.ResponseWriter, r *http.Request) {
	// (BEGIN)到此为止的代码与 "Welcome "路径的第一部分相同。
	c, err := r.Cookie("token")
	if err != nil {
		if err == http.ErrNoCookie {
			w.WriteHeader(http.StatusUnauthorized)
			return
		}
		w.WriteHeader(http.StatusBadRequest)
		return
	}
	tknStr := c.Value
	claims := &Claims{}
	tkn, err := jwt.ParseWithClaims(tknStr, claims, func(token *jwt.Token) (interface{}, error) {
		return jwtKey, nil
	})
	if err != nil {
		if err == jwt.ErrSignatureInvalid {
			w.WriteHeader(http.StatusUnauthorized)
			return
		}
		w.WriteHeader(http.StatusBadRequest)
		return
	}
	if !tkn.Valid {
		w.WriteHeader(http.StatusUnauthorized)
		return
	}
	// (END)到此为止的代码与 "Welcome "路径的第一部分相同。

	// 我们确保在足够的时间内才会发出新的令牌。否则，返回一个坏的请求状态
	if time.Unix(claims.ExpiresAt, 0).Sub(time.Now()) > 30*time.Second {
		w.WriteHeader(http.StatusBadRequest)
		return
	}

	// 现在，为当前用途创建一个新的令牌，并更新到期时间。
	expirationTime := time.Now().Add(5 * time.Minute)
	claims.ExpiresAt = expirationTime.Unix()
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
	tokenString, err := token.SignedString(jwtKey)
	if err != nil {
		w.WriteHeader(http.StatusInternalServerError)
		return
	}

	// 将新的token设置为用户的`token`cookie。
	http.SetCookie(w, &http.Cookie{
		Name:    "token",
		Value:   tokenString,
		Expires: expirationTime,
	})
}
```



### 运行我们的应用程序

要运行此应用程序，请构建并运行Go二进制文件。

```bash
go build
./jwt-go-example。
```

现在，使用任何支持cookie的HTTP客户端（如Postman，或你的网络浏览器），用适当的凭证发出登录请求。

```bash
$ curl -i -X POST -H 'Content-Type: application/json' -d '{"username":"user1","password":"password1"}'  http://127.0.0.1:8000/signin                              
HTTP/1.1 200 OK
Set-Cookie: token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InVzZXIxIiwiZXhwIjoxNjE4MzI2NjY0fQ.T2w0b2HLtHyytEmGxvbWKk4grjoyIEsDvKAlNhKOP4o; Expires=Sun, 13 Oct 2019 13:03:31 GMT
Date: Sun, 13 Oct 2019 12:58:31 GMT
Content-Length: 12
Content-Type: text/plain; charset=utf-8
```



现在你可以尝试从同一个客户端打出欢迎路径，来获取欢迎信息。

```bash
$ curl -i -X GET -b "token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InVzZXIxIiwiZXhwIjoxNjE4MzI2NjY0fQ.T2w0b2HLtHyytEmGxvbWKk4grjoyIEsDvKAlNhKOP4o; Expires=Sun, 13 Oct 2019 15:11:04 GMT"  http://127.0.0.1:8000/welcome
HTTP/1.1 200 OK
Date: Tue, 13 Apr 2021 15:08:01 GMT
Content-Length: 14
Content-Type: text/plain; charset=utf-8

Welcome user1!
```



点击刷新路径，然后检查客户端cookie，查看token cookie的新值。

```bash
$ curl -i -X POST -b "token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InVzZXIxIiwiZXhwIjoxNjE4MzI2NjY0fQ.T2w0b2HLtHyytEmGxvbWKk4grjoyIEsDvKAlNhKOP4o; Expires=Sun, 13 Oct 2019 15:11:04 GMT"  http://127.0.0.1:8000/refresh
HTTP/1.1 200 OK
Set-Cookie: session_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InVzZXIxIiwiZXhwIjoxNjE4MzI2OTUzfQ.Nya_cQMX4KsKk7J39F0Cv1KvK4AGgvpfKTSd8DyqUaY; Expires=Tue, 13 Apr 2021 15:15:53 GMT
Date: Tue, 13 Apr 2021 15:10:53 GMT
Content-Length: 0
```
