+++
date = '2026-04-22T10:53:07+08:00'
draft = false
title = 'Go语言Map的使用和底层原理'
tags = ['Golang']
showtoc = true
+++

> `map` 是 Go 语言内置的哈希表，用来保存键值对。它查询、插入、删除都很方便，平均时间复杂度接近 O(1)，但普通 `map` 不是并发安全的。

## 什么是 map

`map` 是一种键值对集合：

- **key**：用来查找数据的键，必须是可比较类型，比如 `string`、`int`、`bool`、指针、数组、结构体等。
- **value**：键对应的值，可以是任意类型。

常见定义方式：

```go
// 声明但未初始化，值为 nil，不能直接写入
var m map[string]int

// 使用 make 初始化
scores := make(map[string]int)

// 字面量初始化
ages := map[string]int{
	"Tom":  18,
	"Jack": 20,
}
```

需要注意，`nil map` 可以读取，但不能写入：

```go
var m map[string]int

fmt.Println(m["Tom"]) // 0

// panic: assignment to entry in nil map
m["Tom"] = 18
```

所以实际使用时，一般用 `make` 或字面量初始化。

---

## 基础用法

### 新增和修改

```go
users := make(map[string]int)

users["Tom"] = 18  // 新增
users["Tom"] = 19  // 修改
users["Jack"] = 20

fmt.Println(users) // map[Jack:20 Tom:19]
```

`map` 的新增和修改使用同一种写法：如果 key 不存在就是新增，如果 key 已存在就是覆盖旧值。

### 查询

```go
age := users["Tom"]
fmt.Println(age)
```

如果 key 不存在，会返回 value 类型的零值：

```go
fmt.Println(users["Unknown"]) // int 的零值 0
```

但只看零值无法判断 key 是否真的存在，所以 Go 提供了 `comma ok` 写法：

```go
age, ok := users["Tom"]
if ok {
	fmt.Println("Tom 的年龄是", age)
} else {
	fmt.Println("Tom 不存在")
}
```

### 删除

```go
delete(users, "Jack")
```

即使 key 不存在，`delete` 也不会报错。

### 遍历

```go
for name, age := range users {
	fmt.Println(name, age)
}
```

需要注意，`map` 的遍历顺序是不固定的。不要依赖 `range map` 的顺序。如果需要稳定顺序，可以先取出 key 排序，再按 key 读取。

```go
keys := make([]string, 0, len(users))
for name := range users {
	keys = append(keys, name)
}

sort.Strings(keys)

for _, name := range keys {
	fmt.Println(name, users[name])
}
```

---

## map 和 JSON 序列化

Go 标准库 `encoding/json` 可以直接把 `map` 序列化成 JSON。

### map 转 JSON

```go
package main

import (
	"encoding/json"
	"fmt"
)

func main() {
	user := map[string]any{
		"name": "Tom",
		"age":  18,
		"tags": []string{"go", "backend"},
	}

	data, err := json.Marshal(user)
	if err != nil {
		panic(err)
	}

	fmt.Println(string(data))
}
```

输出：

```json
{"age":18,"name":"Tom","tags":["go","backend"]}
```

JSON 对象的 key 本质上是字符串，所以最常见的是使用 `map[string]any` 或 `map[string]string`。

### JSON 转 map

```go
package main

import (
	"encoding/json"
	"fmt"
)

func main() {
	raw := []byte(`{"name":"Tom","age":18,"active":true}`)

	var user map[string]any
	err := json.Unmarshal(raw, &user)
	if err != nil {
		panic(err)
	}

	fmt.Println(user["name"])
	fmt.Println(user["age"])
	fmt.Println(user["active"])
}
```

反序列化到 `map[string]any` 时，JSON 类型会被转换成 Go 的默认类型：

- JSON string -> `string`
- JSON number -> `float64`
- JSON boolean -> `bool`
- JSON array -> `[]any`
- JSON object -> `map[string]any`
- JSON null -> `nil`

所以如果要把 `age` 当作整数使用，需要做类型断言或转换：

```go
ageFloat, ok := user["age"].(float64)
if ok {
	age := int(ageFloat)
	fmt.Println(age)
}
```

如果字段结构固定，更推荐使用结构体：

```go
type User struct {
	Name   string `json:"name"`
	Age    int    `json:"age"`
	Active bool   `json:"active"`
}

var user User
err := json.Unmarshal(raw, &user)
```

结构固定用 `struct`，结构动态或字段不确定时再用 `map[string]any`。

---

## map 的底层原理

Go 的 `map` 底层是哈希表。它的核心思想是：通过 key 计算 hash 值，再根据 hash 值找到对应的桶，然后在桶里查找具体的 key。

可以简单理解为：

```text
key -> hash(key) -> bucket -> key/value
```

### 桶 bucket

Go 的 `map` 会把数据分散到多个 bucket 中。每个 bucket 可以保存多个键值对。查找一个 key 时，流程大概是：

1. 对 key 计算 hash。
2. 根据 hash 的低位找到 bucket。
3. 在 bucket 里比较 key。
4. 找到后返回对应 value。

如果多个 key 计算后落到同一个 bucket，就会发生哈希冲突。Go 会在 bucket 内继续比较 key，如果 bucket 放不下，还会使用溢出桶。

### 扩容

随着元素越来越多，哈希冲突会增加，查询效率会下降。Go 会在合适时机对 `map` 扩容。

扩容不是一次性把所有数据搬完，而是渐进式迁移：

- 新建更大的 bucket 数组。
- 后续读写过程中，逐步把旧 bucket 的数据迁移到新 bucket。
- 这样可以避免一次扩容造成明显卡顿。

这种设计让 `map` 在大多数场景下保持较好的性能。

### key 为什么必须可比较

因为 `map` 查找时不只依赖 hash，还需要判断 bucket 里的 key 是否等于目标 key。

所以 key 必须支持 `==` 比较。下面这些类型不能直接作为 key：

```go
map[[]int]string      // slice 不能比较
map[map[string]int]int // map 不能比较
map[func()]string     // func 不能比较
```

结构体可以作为 key，但结构体里的所有字段都必须可比较：

```go
type Point struct {
	X int
	Y int
}

points := map[Point]string{
	{X: 1, Y: 2}: "A",
}
```

---

## 为什么 map 并发不安全

普通 `map` 支持并发读，但不支持并发读写或并发写写。

下面代码可能直接 panic：

```go
m := make(map[string]int)

go func() {
	for {
		m["count"]++
	}
}()

go func() {
	for {
		fmt.Println(m["count"])
	}
}()
```

常见报错：

```text
fatal error: concurrent map read and map write
```

原因是 `map` 的写操作可能会修改内部结构，比如：

- 插入新 key。
- 删除 key。
- 触发扩容。
- 迁移 bucket。
- 修改 bucket 或溢出桶里的数据。

如果另一个 goroutine 同时读取或写入，就可能读到不一致的内部状态，导致数据竞争，甚至破坏哈希表结构。

Go 的 `map` 没有像 `channel` 那样在内部给每次操作都加锁。这样设计是为了保证单 goroutine 或外部已同步场景下的性能。如果所有 `map` 操作都内置锁，很多普通使用场景都会付出不必要的开销。

---

## 并发场景怎么使用 map

### 使用 sync.Mutex

读写都加同一把锁：

```go
type SafeMap struct {
	mu   sync.Mutex
	data map[string]int
}

func NewSafeMap() *SafeMap {
	return &SafeMap{
		data: make(map[string]int),
	}
}

func (s *SafeMap) Set(key string, value int) {
	s.mu.Lock()
	defer s.mu.Unlock()
	s.data[key] = value
}

func (s *SafeMap) Get(key string) (int, bool) {
	s.mu.Lock()
	defer s.mu.Unlock()
	v, ok := s.data[key]
	return v, ok
}
```

### 使用 sync.RWMutex

读多写少时，可以使用读写锁：

```go
type SafeMap struct {
	mu   sync.RWMutex
	data map[string]int
}

func (s *SafeMap) Set(key string, value int) {
	s.mu.Lock()
	defer s.mu.Unlock()
	s.data[key] = value
}

func (s *SafeMap) Get(key string) (int, bool) {
	s.mu.RLock()
	defer s.mu.RUnlock()
	v, ok := s.data[key]
	return v, ok
}
```

### 使用 sync.Map

Go 标准库还提供了 `sync.Map`，适合读多写少、key 相对稳定、缓存类的场景。

```go
var m sync.Map

m.Store("Tom", 18)

value, ok := m.Load("Tom")
if ok {
	fmt.Println(value.(int))
}

m.Delete("Tom")
```

`sync.Map` 的 API 和普通 `map` 不一样，它通过 `Store`、`Load`、`Delete` 等方法操作。一般业务代码里，如果 key 和 value 类型明确，`map + Mutex/RWMutex` 通常更直观。

### 使用 channel 串行化访问 map

除了加锁，也可以用 `channel` 把所有 map 操作交给同一个 goroutine 处理。

这种方式的核心思想是：不要让多个 goroutine 直接访问同一个 `map`，而是让它们把读写请求发送到一个 channel，由一个专门的 goroutine 顺序处理。

```go
type getReq struct {
	key  string
	resp chan getResp
}

type getResp struct {
	value int
	ok    bool
}

type setReq struct {
	key   string
	value int
	done  chan struct{}
}

type MapServer struct {
	getCh chan getReq
	setCh chan setReq
}

func NewMapServer() *MapServer {
	s := &MapServer{
		getCh: make(chan getReq),
		setCh: make(chan setReq),
	}

	go s.run()

	return s
}

func (s *MapServer) run() {
	data := make(map[string]int)

	for {
		select {
		case req := <-s.getCh:
			v, ok := data[req.key]
			req.resp <- getResp{value: v, ok: ok}
		case req := <-s.setCh:
			data[req.key] = req.value
			close(req.done)
		}
	}
}

func (s *MapServer) Get(key string) (int, bool) {
	resp := make(chan getResp)
	s.getCh <- getReq{
		key:  key,
		resp: resp,
	}

	r := <-resp
	return r.value, r.ok
}

func (s *MapServer) Set(key string, value int) {
	done := make(chan struct{})
	s.setCh <- setReq{
		key:   key,
		value: value,
		done:  done,
	}

	<-done
}
```

使用：

```go
func main() {
	m := NewMapServer()

	m.Set("Tom", 18)

	age, ok := m.Get("Tom")
	if ok {
		fmt.Println(age)
	}
}
```

这种方式本质上是用 channel 做消息传递，让 map 只被一个 goroutine 持有和修改。因为同一时间只有 `run` 这个 goroutine 直接访问底层 `data`，所以不会出现并发读写 map 的问题。

优点：

- 不需要显式使用锁。
- 很适合把 map 当作一个状态管理器。
- 可以把读写、删除、统计等操作统一收口。

缺点：

- 所有操作都会经过同一个 goroutine，高并发下可能成为瓶颈。
- 写法比 `Mutex` 更复杂。
- 需要考虑 goroutine 退出、channel 关闭、超时等生命周期问题。

所以 channel 方案更适合“通过消息管理状态”的场景。如果只是简单保护一个 map，`sync.RWMutex` 通常更直接。

---

## 常见注意点

### value 是结构体时，不能直接修改字段

```go
type User struct {
	Name string
	Age  int
}

users := map[string]User{
	"Tom": {Name: "Tom", Age: 18},
}

// 编译错误
// users["Tom"].Age = 20
```

因为 `users["Tom"]` 取出来的是一个临时值，不是可寻址变量。

可以这样写：

```go
u := users["Tom"]
u.Age = 20
users["Tom"] = u
```

或者把 value 定义成指针：

```go
users := map[string]*User{
	"Tom": {Name: "Tom", Age: 18},
}

users["Tom"].Age = 20
```

### map 是引用类型

`map` 赋值或作为函数参数传递时，不会复制底层数据：

```go
func update(m map[string]int) {
	m["Tom"] = 20
}

func main() {
	users := map[string]int{"Tom": 18}
	update(users)
	fmt.Println(users["Tom"]) // 20
}
```

所以函数里修改 `map`，外部也能看到变化。

---

## 一句话总结

> **Go 的 map 是内置哈希表，适合保存键值对，支持高效查询、插入和删除；JSON 场景中常用 `map[string]any` 处理动态结构；底层通过 hash、bucket、溢出桶和渐进式扩容实现；普通 map 为了性能不内置并发锁，所以并发读写时必须使用 `Mutex`、`RWMutex`、`sync.Map` 或 channel 串行化访问。**
