**认证（Authentication，简称 Authn）：** 一般指身份验证，指通过一定的手段，完成对用户身份的确认。认证用来证明你是谁。

### JWT 认证流程



![image-20231203174211751](../img/image-20231203174211751.png)

1. 客户端（通常是控制台）使用用户名和密码登录；
2. 服务端收到请求后会去验证用户名和密码，如果用户名和密码跟数据库记录不一致，则验证失败，如果一致则验证通过，服务端会签发一个 Token 返回给客户端；
3. 客户端收到请求后会将 Token 缓存起来，比如放在浏览器 Cookie 中或者本地存储中，之后每次请求都会携带该 Token；
4. 服务端收到请求后会验证请求中携带的 Token，验证通过则进行业务逻辑处理并成功返回数据。

### JWT Token 格式

在 JWT 中，Token 有三部分组成，中间用 `.` 隔开，并使用 Base64 编码：

- header；
- payload；
- signature。

如下是 JWT 中的一个 Token 示例：

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpYXQiOjE1MjgwMTY5MjIsImlkIjowLCJuYmYiOjE1MjgwMTY5MjIsInVzZXJuYW1lIjoiYWRtaW4ifQ.LjxrK9DuAwAzUD8-9v43NzWBN7HXsSLfebw92DKd1JQ
```

#### header

JWT Token 的 header 中，包含两部分信息：

- `type`：Token 的类型；
- `alg`：Token 所使用的加密算法。
- `kid`：密钥id（可选）

例如：

```json
{
  "typ": "JWT",
  "alg": "HS256"
}
```

参数说明：

- `type`：说明Token的类型是JWT
- `alg`：说明Token加密算法，这里是 HS256（alg 算法可以有多种)

这里将Header进行base64编码

```shell
echo -n '{"type":"jwt", "alg":"HS256"}' |base64
eyJ0eXBlIjoiand0IiwgImFsZyI6IkhTMjU2In0=
```

在某些场景下，可能还会有kid选项，用来标识一个密钥ID，例如：

```json
{
	"alg": "HS256",
	"kid": "x7dfa8syf8dysafdsya8fyds8ayfsa",
	"type": "JWT"
}
```



#### Payload 载荷

Payload 中携带 Token 的具体内容，由三部分组成：JWT标准注册的声明（可选）、公共的声明、私有声明，可在公共声明中添加比如记录请求用户名。

标准字段有：

- `iss`：JWT Token 的签发者，其值大小写敏感字符串或者URL；
- `sub`：主题；可以用来鉴别一个用户
- `exp`：JWT Token 过期时间；
- `aud`：接收 JWT Token 的一方，其值应为大小写敏感字符串或URL，一般可以为特定的App，服务或模块。服务端的安全策略在签发时和验证时，aud必须时一致的；
- `iat`：JWT Token 签发时间；
- `nbf`：JWT Token 生效时间 ；
- `jti`：JWT Token ID。令牌唯一标识符，通常用于一次性消费的Token

```json
{
  "id": 2,						// 公共字段：用户id
  "username": "kong", // 公共字段：用户名
  "iss": "iamctl"
  "aud": "iam.authz.marmotedu.com"
  "exp": 1604158987
  "nbf": 1527931805,
  "iat": 1527931805
}

```

在公共的声明中可以添加任何需要的信息，一般添加用户相关的形象或者其他业务需要的信息，注意不要添加敏感信息，私有声明时客户端和服务端所共同定义的声明，因为 base64 是对称加密的， 所以一般不建议存放敏感信息



#### Signature 签名

`Signature` 是 Token 的签名部分，通过如下方式生成：

1. 用 Base64 对 header和payload 进行base64编码后用 "." 连接。
2. 用 Secret 对编码后的内容进行加密，加密后的内容即为 `Signature`。

Secret 相当于一个密码，存储在服务端，一般通过配置文件来配置 Secret 的值:

```shell
# JWT 配置
jwt:
  realm: JWT # jwt 标识
  key: dfVpOK8LZeJLZHYmHdb1VdyRrACKpqoo # 服务端密钥
  timeout: 24h # token 过期时间(小时)
  max-refresh: 24h # token 更新时间(小时)
```

签名后服务端返回生成的token，客户端下次请求会携带该token, 服务端收到token后解析出header.payload 然后用相同的加密算法和密钥对header.payload再进行一次加密得到 Signature ，并对加密后的 Signature 和收到的 Singature 是否相同没如果相同则验证通过，如果不相同则返回 HTTP 401 Unauthorized 错误

注意：不要在Token中存放敏感信息，Payload中的过期时间不要设置太大，一般开发版本设置7小时，线上版本设置为2小时。



### gin-jwt

```
go get -u github.com/appleboy/gin-jwt
```

它使用jwt-go提供JWT身份验证中间件。它提供了额外的处理函数，用于提供`login API`以生成令牌，并提供了一个用于刷新令牌的额外刷新处理程序。

安全问题：

简单的HS256 JWT令牌暴力破解器。仅能有效破解使用弱密钥的JWT令牌。**建议**：使用强大且长的密钥，或者使用RS256令牌。参考[jwt-cracker](https://github.com/lmammino/jwt-cracker)库。该工具库是破解jwt的工具



#### Example

```go
package main

import (
  "log"
  "net/http"
  "os"
  "time"

  jwt "github.com/appleboy/gin-jwt/v2"
  "github.com/gin-gonic/gin"
)

type login struct {
  Username string `form:"username" json:"username" binding:"required"`
  Password string `form:"password" json:"password" binding:"required"`
}

var identityKey = "id"

func helloHandler(c *gin.Context) {
  claims := jwt.ExtractClaims(c)
  user, _ := c.Get(identityKey)
  c.JSON(200, gin.H{
    "userID":   claims[identityKey],
    "userName": user.(*User).UserName,
    "text":     "Hello World.",
  })
}

// User demo
type User struct {
  UserName  string
  FirstName string
  LastName  string
}

func main() {
  port := os.Getenv("PORT")
  r := gin.Default()

  if port == "" {
    port = "8000"
  }

  // the jwt middleware
  authMiddleware, err := jwt.New(&jwt.GinJWTMiddleware{
    Realm:       "test zone",
    Key:         []byte("secret key"),
    Timeout:     time.Hour,
    MaxRefresh:  time.Hour,
    IdentityKey: identityKey,
    PayloadFunc: func(data interface{}) jwt.MapClaims {
      if v, ok := data.(*User); ok {
        return jwt.MapClaims{
          identityKey: v.UserName,
        }
      }
      return jwt.MapClaims{}
    },
    IdentityHandler: func(c *gin.Context) interface{} {
      claims := jwt.ExtractClaims(c)
      return &User{
        UserName: claims[identityKey].(string),
      }
    },
    Authenticator: func(c *gin.Context) (interface{}, error) {
      var loginVals login
      if err := c.ShouldBind(&loginVals); err != nil {
        return "", jwt.ErrMissingLoginValues
      }
      userID := loginVals.Username
      password := loginVals.Password

      if (userID == "admin" && password == "admin") || (userID == "test" && password == "test") {
        return &User{
          UserName:  userID,
          LastName:  "Bo-Yi",
          FirstName: "Wu",
        }, nil
      }

      return nil, jwt.ErrFailedAuthentication
    },
    Authorizator: func(data interface{}, c *gin.Context) bool {
      if v, ok := data.(*User); ok && v.UserName == "admin" {
        return true
      }

      return false
    },
    Unauthorized: func(c *gin.Context, code int, message string) {
      c.JSON(code, gin.H{
        "code":    code,
        "message": message,
      })
    },
    // TokenLookup is a string in the form of "<source>:<name>" that is used
    // to extract token from the request.
    // Optional. Default value "header:Authorization".
    // Possible values:
    // - "header:<name>"
    // - "query:<name>"
    // - "cookie:<name>"
    // - "param:<name>"
    TokenLookup: "header: Authorization, query: token, cookie: jwt",
    // TokenLookup: "query:token",
    // TokenLookup: "cookie:token",

    // TokenHeadName is a string in the header. Default value is "Bearer"
    TokenHeadName: "Bearer",

    // TimeFunc provides the current time. You can override it to use another time value. This is useful for testing or if your server uses a different time zone than your tokens.
    TimeFunc: time.Now,
  })

  if err != nil {
    log.Fatal("JWT Error:" + err.Error())
  }

  // When you use jwt.New(), the function is already automatically called for checking,
  // which means you don't need to call it again.
  errInit := authMiddleware.MiddlewareInit()

  if errInit != nil {
    log.Fatal("authMiddleware.MiddlewareInit() Error:" + errInit.Error())
  }

  r.POST("/login", authMiddleware.LoginHandler)

  r.NoRoute(authMiddleware.MiddlewareFunc(), func(c *gin.Context) {
    claims := jwt.ExtractClaims(c)
    log.Printf("NoRoute claims: %#v\n", claims)
    c.JSON(404, gin.H{"code": "PAGE_NOT_FOUND", "message": "Page not found"})
  })

  auth := r.Group("/auth")
  // Refresh time can be longer than token timeout
  auth.GET("/refresh_token", authMiddleware.RefreshHandler)
  auth.Use(authMiddleware.MiddlewareFunc())
  {
    auth.GET("/hello", helloHandler)
  }

  if err := http.ListenAndServe(":"+port, r); err != nil {
    log.Fatal(err)
  }
}
```







