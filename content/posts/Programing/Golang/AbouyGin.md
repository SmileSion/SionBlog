+++
date = '2026-04-15T11:23:07+08:00'
draft = false
title = 'Gin的使用及优缺点'
tags = ['Golang']

+++

> Gin 是一个轻量级 HTTP Web 框架，封装了路由、参数绑定、中间件等功能，性能较高，适合开发 RESTful API。

## 优点

**性能高**

- 基于 `httprouter`

- 零内存分配路由（zero allocation）

- 比较接近原生 `net/http`

**中间件机制完善**

- 支持全局 / 路由级 / 分组中间件

**支持参数绑定和验证**

- JSON / form / query 自动解析

## 缺点

**不适合复杂大型项目**

- 没有Java Spring 那样的规范

**Context过于强劲**

- 不利于分层

### 基础用法

**最小demo示例：引入库+启动一个监听接口**

```go
import (
  "net/http"
  "github.com/gin-gonic/gin"
)
func main() {
  // 用默认中间件创建路由
  r := gin.Default()

  // 定义一个简单的get请求接口
  r.GET("/ping", func(c *gin.Context) {
    // 返回JSON响应
    c.JSON(http.StatusOK, gin.H{
      "message": "pong",
    })
  })
  // 默认启动在8080端口
  r.Run()
}
```

**最基本用法**

```go
r := gin.Default()
r.Run(":8080")
```

**路由**

```go
//基础路由
r.GET("/user", handler)
r.POST("/user", handler)
r.PUT("/user/:id", handler)
r.DELETE("/user/:id", handler)
//分组路由
api := r.Group("/api")
{
	api.GET("/users", getUsers)
	api.POST("/users", createUser)
}
```

**参数获取**

```go
//Query参数
name := c.Query("name")
//Path参数
id := c.Param("id")
//Json参数
type User struct {
	Name string `json:"name"`
}

var u User
c.ShouldBindJSON(&u)
```

**返回数据**

```go
c.JSON(200, gin.H{
	"msg": "ok",
})
```

**中间件**

```go
//日志为例
func Logger() gin.HandlerFunc {
	return func(c *gin.Context) {
		fmt.Println("before")
		c.Next()
		fmt.Println("after")
	}
}//c.Abort()中断请求
```

**错误处理**

```go
c.AbortWithStatusJSON(400, gin.H{
	"error": "bad request",
})
```

**绑定校验**

```go
type User struct {
	Name string `json:"name" binding:"required"`
}

if err := c.ShouldBindJSON(&u); err != nil {
	c.JSON(400, gin.H{"error": err.Error()})
	return
}
```

**自定义中间件**

```go
func Auth() gin.HandlerFunc {
	return func(c *gin.Context) {
		token := c.GetHeader("Authorization")
		if token == "" {
			c.AbortWithStatus(401)
			return
		}
		c.Next()
	}
}
```

**文件上传**

```go
file, _ := c.FormFile("file")
c.SaveUploadedFile(file, "./"+file.Filename)
```

