+++
date = '2026-04-22T11:30:07+08:00'
draft = false
title = 'Go语言GORM的介绍和常见用法'
tags = ['Golang']
+++

> GORM 是 Go 语言里常用的 ORM 框架，可以把数据库表映射成 Go 结构体，让我们用面向对象的方式操作 MySQL、PostgreSQL、SQLite 等数据库。

## 什么是 GORM

ORM 的全称是 Object Relational Mapping，也就是对象关系映射。

在 Go 项目里，如果不用 ORM，通常需要手写 SQL：

```go
rows, err := db.Query("select id, name, age from users where id = ?", id)
```

使用 GORM 后，可以用结构体和方法调用来操作数据库：

```go
var user User
db.First(&user, id)
```

GORM 会根据结构体、字段、方法调用生成对应 SQL。

GORM 常见特点：

- 支持 MySQL、PostgreSQL、SQLite、SQL Server 等数据库。
- 支持结构体和表的自动映射。
- 支持增删改查、事务、关联关系、预加载。
- 支持自动迁移表结构。
- 支持 Hook、软删除、日志、连接池等功能。

---

## 安装 GORM

以 MySQL 为例：

```bash
go get gorm.io/gorm
go get gorm.io/driver/mysql
```

如果使用 SQLite：

```bash
go get gorm.io/driver/sqlite
```

如果使用 PostgreSQL：

```bash
go get gorm.io/driver/postgres
```

---

## 连接数据库

### 连接 MySQL

```go
package main

import (
	"gorm.io/driver/mysql"
	"gorm.io/gorm"
)

func main() {
	dsn := "root:password@tcp(127.0.0.1:3306)/test?charset=utf8mb4&parseTime=True&loc=Local"

	db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
	if err != nil {
		panic(err)
	}

	_ = db
}
```

DSN 说明：

- `root:password`：数据库用户名和密码。
- `127.0.0.1:3306`：数据库地址和端口。
- `test`：数据库名。
- `charset=utf8mb4`：支持完整 UTF-8 字符。
- `parseTime=True`：把数据库时间转换成 Go 的 `time.Time`。
- `loc=Local`：使用本地时区。

### 连接 SQLite

```go
db, err := gorm.Open(sqlite.Open("test.db"), &gorm.Config{})
if err != nil {
	panic(err)
}
```

SQLite 适合本地测试、小工具、单机应用。

---

## 定义模型

GORM 用结构体表示数据库表。

```go
type User struct {
	ID        uint
	Name      string
	Email     string
	Age       int
	CreatedAt time.Time
	UpdatedAt time.Time
}
```

默认规则：

- 结构体名 `User` 对应表名 `users`。
- 字段 `Name` 对应列名 `name`。
- `ID` 默认是主键。
- `CreatedAt` 创建时自动填充。
- `UpdatedAt` 更新时自动填充。

如果想使用 GORM 内置基础模型，可以嵌入 `gorm.Model`：

```go
type User struct {
	gorm.Model
	Name  string
	Email string
	Age   int
}
```

`gorm.Model` 包含：

```go
type Model struct {
	ID        uint
	CreatedAt time.Time
	UpdatedAt time.Time
	DeletedAt gorm.DeletedAt
}
```

其中 `DeletedAt` 用于软删除。

---

## 自动迁移表结构

GORM 可以根据结构体自动创建或更新表结构：

```go
err := db.AutoMigrate(&User{})
if err != nil {
	panic(err)
}
```

`AutoMigrate` 会创建表、添加缺少的字段、创建索引和约束，但一般不会主动删除字段，避免误删数据。

开发环境可以使用 `AutoMigrate`，生产环境更推荐使用专门的迁移工具管理表结构。

---

## 使用 tag 控制字段

GORM 支持通过结构体 tag 控制字段属性：

```go
type User struct {
	ID    uint   `gorm:"primaryKey"`
	Name  string `gorm:"size:64;not null"`
	Email string `gorm:"uniqueIndex;size:128"`
	Age   int    `gorm:"default:18"`
}
```

常见 tag：

- `primaryKey`：主键。
- `column:name`：指定列名。
- `type:varchar(100)`：指定数据库类型。
- `size:64`：指定字段长度。
- `not null`：非空。
- `default:18`：默认值。
- `uniqueIndex`：唯一索引。
- `index`：普通索引。
- `-`：忽略该字段，不映射到数据库。

示例：

```go
type User struct {
	ID       uint
	NickName string `gorm:"column:nick_name"`
	Password string `gorm:"-"`
}
```

---

## 新增数据

### 创建一条记录

```go
user := User{
	Name:  "Tom",
	Email: "tom@example.com",
	Age:   18,
}

result := db.Create(&user)

fmt.Println(result.Error)
fmt.Println(result.RowsAffected)
fmt.Println(user.ID)
```

`Create` 成功后，GORM 会把自增主键回填到结构体中。

### 批量创建

```go
users := []User{
	{Name: "Tom", Email: "tom@example.com", Age: 18},
	{Name: "Jack", Email: "jack@example.com", Age: 20},
}

db.Create(&users)
```

---

## 查询数据

### 根据主键查询

```go
var user User

db.First(&user, 1)
```

常见方法：

- `First`：按主键升序查询第一条。
- `Last`：按主键降序查询第一条。
- `Take`：查询一条，不指定排序。

### 查询所有记录

```go
var users []User

db.Find(&users)
```

### 条件查询

```go
var user User

db.Where("name = ?", "Tom").First(&user)
```

多个条件：

```go
db.Where("name = ? and age > ?", "Tom", 18).Find(&users)
```

使用结构体条件：

```go
db.Where(&User{Name: "Tom"}).Find(&users)
```

使用 map 条件：

```go
db.Where(map[string]any{
	"name": "Tom",
	"age":  18,
}).Find(&users)
```

### 排序、分页

```go
db.Where("age > ?", 18).
	Order("created_at desc").
	Limit(10).
	Offset(20).
	Find(&users)
```

这段代码表示查询年龄大于 18 的用户，按创建时间倒序，跳过前 20 条，取 10 条。

### 选择指定字段

```go
db.Select("id", "name", "email").Find(&users)
```

### 查询数量

```go
var count int64

db.Model(&User{}).Where("age > ?", 18).Count(&count)
```

---

## 更新数据

### 更新单个字段

```go
db.Model(&User{}).Where("id = ?", 1).Update("age", 20)
```

### 更新多个字段

```go
db.Model(&User{}).Where("id = ?", 1).Updates(map[string]any{
	"name": "Jack",
	"age":  21,
})
```

也可以使用结构体：

```go
db.Model(&user).Updates(User{
	Name: "Jack",
	Age:  21,
})
```

注意，使用结构体更新时，GORM 默认不会更新零值字段，比如 `0`、`false`、空字符串。

```go
db.Model(&user).Updates(User{
	Age: 0, // 默认不会更新
})
```

如果确实要更新零值，可以使用 map：

```go
db.Model(&user).Updates(map[string]any{
	"age": 0,
})
```

或者使用 `Select`：

```go
db.Model(&user).Select("Age").Updates(User{Age: 0})
```

---

## 删除数据

### 根据主键删除

```go
db.Delete(&User{}, 1)
```

### 条件删除

```go
db.Where("age < ?", 18).Delete(&User{})
```

如果模型里包含 `gorm.DeletedAt`，GORM 默认执行软删除，也就是只更新 `deleted_at` 字段，不会真正删除数据库记录。

```go
type User struct {
	gorm.Model
	Name string
}
```

软删除后，普通查询不会查到这条数据。

如果要查询被软删除的数据：

```go
db.Unscoped().Where("id = ?", 1).First(&user)
```

如果要物理删除：

```go
db.Unscoped().Delete(&user)
```

---

## 错误处理

GORM 的大部分操作都会返回 `*gorm.DB`，错误信息在 `Error` 字段中。

```go
result := db.First(&user, 1)
if result.Error != nil {
	panic(result.Error)
}
```

判断记录不存在：

```go
result := db.First(&user, 1)
if errors.Is(result.Error, gorm.ErrRecordNotFound) {
	fmt.Println("user not found")
}
```

也可以查看影响行数：

```go
result := db.Model(&User{}).Where("id = ?", 1).Update("age", 20)

fmt.Println(result.RowsAffected)
fmt.Println(result.Error)
```

---

## 事务

事务用于保证一组数据库操作要么全部成功，要么全部失败。

### 手动事务

```go
tx := db.Begin()

if err := tx.Error; err != nil {
	panic(err)
}

if err := tx.Create(&user).Error; err != nil {
	tx.Rollback()
	return
}

if err := tx.Create(&order).Error; err != nil {
	tx.Rollback()
	return
}

tx.Commit()
```

### Transaction 方法

更推荐使用 `Transaction`，代码更简洁：

```go
err := db.Transaction(func(tx *gorm.DB) error {
	if err := tx.Create(&user).Error; err != nil {
		return err
	}

	if err := tx.Create(&order).Error; err != nil {
		return err
	}

	return nil
})

if err != nil {
	fmt.Println("transaction failed:", err)
}
```

只要回调函数返回错误，GORM 就会自动回滚。返回 `nil` 则自动提交。

---

## 关联关系

GORM 支持常见表关系：一对一、一对多、多对多。

### 一对多

一个用户有多篇文章：

```go
type User struct {
	ID    uint
	Name  string
	Posts []Post
}

type Post struct {
	ID     uint
	Title  string
	UserID uint
}
```

创建：

```go
user := User{
	Name: "Tom",
	Posts: []Post{
		{Title: "GORM 入门"},
		{Title: "Go Web 开发"},
	},
}

db.Create(&user)
```

预加载关联数据：

```go
var user User

db.Preload("Posts").First(&user, 1)
```

如果不使用 `Preload`，默认只查询 `users` 表，不会自动查 `posts`。

### 多对多

用户和角色是多对多关系：

```go
type User struct {
	ID    uint
	Name  string
	Roles []Role `gorm:"many2many:user_roles;"`
}

type Role struct {
	ID   uint
	Name string
}
```

GORM 会通过中间表 `user_roles` 维护关联。

查询时预加载：

```go
db.Preload("Roles").First(&user, 1)
```

---

## 原生 SQL

有些复杂查询直接写 SQL 更清晰，GORM 也支持原生 SQL。

### Raw 查询

```go
var users []User

db.Raw("select id, name, age from users where age > ?", 18).Scan(&users)
```

### Exec 执行

```go
db.Exec("update users set age = age + 1 where id = ?", 1)
```

ORM 不是必须替代所有 SQL。复杂报表、复杂 join、性能敏感 SQL，直接写原生 SQL 反而更清楚。

---

## 日志和调试

调试时可以使用 `Debug()` 打印 SQL：

```go
db.Debug().Where("name = ?", "Tom").First(&user)
```

也可以在初始化时配置日志级别：

```go
db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{
	Logger: logger.Default.LogMode(logger.Info),
})
```

这样可以看到 GORM 实际执行的 SQL，排查条件、字段、关联查询问题会方便很多。

---

## 连接池配置

GORM 底层使用的是标准库 `database/sql`，可以配置连接池：

```go
sqlDB, err := db.DB()
if err != nil {
	panic(err)
}

sqlDB.SetMaxIdleConns(10)
sqlDB.SetMaxOpenConns(100)
sqlDB.SetConnMaxLifetime(time.Hour)
```

常见含义：

- `SetMaxIdleConns`：最大空闲连接数。
- `SetMaxOpenConns`：最大打开连接数。
- `SetConnMaxLifetime`：连接最大复用时间。

线上项目一般需要根据数据库性能、服务并发量和部署实例数量设置连接池，避免连接数过多压垮数据库。

---

## 常见注意点

### 不要忽略 Error

```go
if err := db.Create(&user).Error; err != nil {
	return err
}
```

数据库操作一定要检查错误，否则可能出现写入失败但业务继续执行的问题。

### 结构体更新默认忽略零值

```go
db.Model(&user).Updates(User{Age: 0})
```

这类写法不会更新 `Age`，如果要更新零值，使用 `map` 或 `Select`。

### 查询单条记录要处理 ErrRecordNotFound

```go
err := db.First(&user, id).Error
if errors.Is(err, gorm.ErrRecordNotFound) {
	return nil
}
```

### 复杂 SQL 不要强行 ORM

ORM 适合常规 CRUD，但复杂统计、复杂 join、窗口函数等场景，直接写 SQL 通常更直观，也更容易优化。

### 生产环境谨慎使用 AutoMigrate

`AutoMigrate` 很适合开发阶段快速建表，但生产环境表结构变更应该可审计、可回滚，建议使用迁移工具。

---

## 一句话总结

> **GORM 是 Go 语言常用的 ORM 框架，通过结构体映射数据库表，提供连接数据库、自动迁移、CRUD、事务、关联查询、软删除、原生 SQL 等能力；它能提高常规业务开发效率，但使用时要注意错误处理、零值更新、连接池配置和复杂 SQL 的性能问题。**
