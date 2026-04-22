+++
date = '2026-04-22T11:10:07+08:00'
draft = false
title = 'Go语言Reflect包的使用'
tags = ['Golang']
showtoc = true
+++

> `reflect` 是 Go 标准库里的反射包，可以在运行时查看变量的类型和值，也可以动态读取和修改对象。它常用于 JSON 序列化、ORM、配置解析、参数校验、依赖注入等框架代码中。

## 什么是反射

正常写 Go 代码时，变量类型在编译期就已经确定：

```go
name := "Tom"
age := 18
```

编译器知道 `name` 是 `string`，`age` 是 `int`，所以可以做类型检查。

反射解决的是另一个问题：如果代码在编译期不知道具体类型，运行时才拿到一个 `interface{}` 或 `any`，还能不能知道它到底是什么类型、有什么字段、能不能调用方法？

答案就是使用 `reflect`。

简单来说：

```text
普通代码：编译期知道类型
反射代码：运行时检查类型和值
```

---

## reflect 的两个核心类型

`reflect` 包最重要的是两个类型：

- `reflect.Type`：表示变量的类型信息。
- `reflect.Value`：表示变量的值信息。

获取方式：

```go
reflect.TypeOf(x)
reflect.ValueOf(x)
```

示例：

```go
package main

import (
	"fmt"
	"reflect"
)

func main() {
	var age int = 18

	t := reflect.TypeOf(age)
	v := reflect.ValueOf(age)

	fmt.Println(t)        // int
	fmt.Println(v)        // 18
	fmt.Println(t.Kind()) // int
	fmt.Println(v.Kind()) // int
}
```

`Type` 更关注“它是什么类型”，`Value` 更关注“它的值是什么”。

---

## Type 和 Kind 的区别

`Type` 表示具体类型，`Kind` 表示底层分类。

```go
type MyInt int

var num MyInt = 10

t := reflect.TypeOf(num)

fmt.Println(t)        // main.MyInt
fmt.Println(t.Kind()) // int
```

`MyInt` 是一个自定义类型，所以 `Type` 是 `main.MyInt`。但它的底层类型是 `int`，所以 `Kind` 是 `int`。

常见 `Kind` 有：

- `reflect.Int`
- `reflect.String`
- `reflect.Bool`
- `reflect.Struct`
- `reflect.Ptr`
- `reflect.Slice`
- `reflect.Map`
- `reflect.Interface`
- `reflect.Func`

---

## 读取基本类型的值

`reflect.Value` 提供了不同方法读取具体值：

```go
func main() {
	name := "Tom"
	age := 18
	active := true

	fmt.Println(reflect.ValueOf(name).String())
	fmt.Println(reflect.ValueOf(age).Int())
	fmt.Println(reflect.ValueOf(active).Bool())
}
```

不同类型要用对应方法：

- 字符串用 `String()`
- 整数用 `Int()`
- 无符号整数用 `Uint()`
- 浮点数用 `Float()`
- 布尔值用 `Bool()`

如果类型不匹配，可能会 panic。所以实际使用时一般先判断 `Kind`。

```go
v := reflect.ValueOf(age)

if v.Kind() == reflect.Int {
	fmt.Println(v.Int())
}
```

---

## 修改变量的值

反射可以修改变量，但必须传入指针。

错误示例：

```go
age := 18
v := reflect.ValueOf(age)

// panic: reflect.Value.SetInt using unaddressable value
v.SetInt(20)
```

原因是 `ValueOf(age)` 拿到的是变量的副本，不是原变量本身。

正确写法：

```go
age := 18

v := reflect.ValueOf(&age)
e := v.Elem()

e.SetInt(20)

fmt.Println(age) // 20
```

这里有两个关键点：

- `ValueOf(&age)`：传入变量地址。
- `Elem()`：拿到指针指向的真实值。

修改前也可以判断是否可修改：

```go
if e.CanSet() {
	e.SetInt(20)
}
```

---

## 反射结构体

反射最常见的使用场景之一是处理结构体。

```go
type User struct {
	Name string
	Age  int
}

func main() {
	u := User{Name: "Tom", Age: 18}

	t := reflect.TypeOf(u)
	v := reflect.ValueOf(u)

	for i := 0; i < t.NumField(); i++ {
		fieldType := t.Field(i)
		fieldValue := v.Field(i)

		fmt.Println(fieldType.Name, fieldType.Type, fieldValue)
	}
}
```

输出类似：

```text
Name string Tom
Age int 18
```

常用方法：

- `NumField()`：字段数量。
- `Field(i)`：根据下标获取字段。
- `FieldByName(name)`：根据字段名获取字段。
- `Name`：字段名。
- `Type`：字段类型。

---

## 读取结构体 Tag

结构体 tag 本质上也是类型信息的一部分，所以需要通过 `reflect.Type` 读取。

```go
type User struct {
	Name string `json:"name" validate:"required"`
	Age  int    `json:"age"`
}

func main() {
	t := reflect.TypeOf(User{})

	for i := 0; i < t.NumField(); i++ {
		field := t.Field(i)

		fmt.Println(field.Name)
		fmt.Println("json:", field.Tag.Get("json"))
		fmt.Println("validate:", field.Tag.Get("validate"))
	}
}
```

这也是很多框架能够识别 `json`、`gorm`、`validate` 等 tag 的基础。

例如 `encoding/json` 会读取结构体字段上的 `json` tag，然后决定 JSON 字段名：

```go
type User struct {
	Name string `json:"name"`
}
```

---

## 修改结构体字段

修改结构体字段时，也需要传指针：

```go
type User struct {
	Name string
	Age  int
}

func main() {
	u := User{Name: "Tom", Age: 18}

	v := reflect.ValueOf(&u).Elem()

	nameField := v.FieldByName("Name")
	if nameField.IsValid() && nameField.CanSet() {
		nameField.SetString("Jack")
	}

	ageField := v.FieldByName("Age")
	if ageField.IsValid() && ageField.CanSet() {
		ageField.SetInt(20)
	}

	fmt.Println(u) // {Jack 20}
}
```

需要注意，反射只能直接修改导出字段，也就是首字母大写的字段。

```go
type User struct {
	Name string
	age  int
}
```

这里的 `age` 是未导出字段，普通反射不能直接 `Set`。

---

## 调用方法

反射也可以动态调用方法。

```go
type User struct {
	Name string
}

func (u User) Say(prefix string) string {
	return prefix + ", " + u.Name
}

func main() {
	u := User{Name: "Tom"}

	v := reflect.ValueOf(u)
	method := v.MethodByName("Say")

	args := []reflect.Value{
		reflect.ValueOf("hello"),
	}

	result := method.Call(args)

	fmt.Println(result[0].String()) // hello, Tom
}
```

`Call` 接收 `[]reflect.Value`，返回值也是 `[]reflect.Value`。因为一个函数可能有多个参数，也可能有多个返回值。

调用前最好判断方法是否存在：

```go
method := v.MethodByName("Say")
if method.IsValid() {
	result := method.Call(args)
	fmt.Println(result[0].String())
}
```

---

## 遍历 map 和 slice

### 遍历 slice

```go
nums := []int{1, 2, 3}
v := reflect.ValueOf(nums)

for i := 0; i < v.Len(); i++ {
	fmt.Println(v.Index(i).Int())
}
```

### 遍历 map

```go
m := map[string]int{
	"Tom":  18,
	"Jack": 20,
}

v := reflect.ValueOf(m)

for _, key := range v.MapKeys() {
	value := v.MapIndex(key)
	fmt.Println(key.String(), value.Int())
}
```

如果要写通用工具函数，比如打印任意对象内容，反射就很有用。

---

## 一个简单的通用打印函数

下面是一个简单示例：根据不同类型打印值。

```go
func PrintValue(x any) {
	v := reflect.ValueOf(x)

	switch v.Kind() {
	case reflect.String:
		fmt.Println("string:", v.String())
	case reflect.Int:
		fmt.Println("int:", v.Int())
	case reflect.Bool:
		fmt.Println("bool:", v.Bool())
	case reflect.Struct:
		t := v.Type()
		for i := 0; i < v.NumField(); i++ {
			fmt.Println(t.Field(i).Name, "=", v.Field(i))
		}
	default:
		fmt.Println("unsupported:", v.Kind())
	}
}
```

使用：

```go
PrintValue("Tom")
PrintValue(18)
PrintValue(User{Name: "Tom", Age: 18})
```

---

## 反射的使用场景

反射适合写通用代码，常见场景包括：

- JSON、XML 等序列化和反序列化。
- ORM 框架根据结构体字段生成 SQL。
- 参数校验库读取 `validate` tag。
- 配置库把配置文件映射到结构体。
- RPC 框架动态调用方法。
- 依赖注入框架根据类型创建对象。

比如 JSON 序列化时，标准库需要知道结构体有哪些字段、字段名是什么、有没有 `json` tag，这些都需要依赖反射。

---

## 反射的缺点

反射虽然强大，但不应该滥用。

### 性能更差

反射需要在运行时检查类型、查找字段、做动态调用，相比普通代码性能更低。

### 可读性更差

普通代码：

```go
user.Name = "Tom"
```

反射代码：

```go
v.FieldByName("Name").SetString("Tom")
```

后者更绕，也更容易写错。

### 类型安全变弱

普通代码很多错误可以在编译期发现，反射代码很多错误会变成运行时 panic。

例如：

```go
v.SetInt(10)
```

如果 `v` 不是整数类型，程序运行时才会报错。

---

## 使用建议

- 业务代码里优先使用普通类型和接口。
- 只有在需要处理未知类型、通用框架、动态字段时再使用反射。
- 使用前先判断 `Kind`，避免类型不匹配。
- 修改值时传指针，并使用 `Elem()`。
- 修改字段前判断 `CanSet()`。
- 调用方法前判断 `IsValid()`。
- 对性能敏感的代码谨慎使用反射。

---

## 一句话总结

> **Go 的 reflect 包提供了运行时检查类型和值的能力，核心是 `reflect.Type` 和 `reflect.Value`；它可以读取字段、读取 tag、修改值、调用方法，适合框架和通用工具代码，但性能、可读性和类型安全都不如普通代码，所以业务代码中应该谨慎使用。**
