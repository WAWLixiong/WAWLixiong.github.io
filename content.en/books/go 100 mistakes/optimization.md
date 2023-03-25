---
title: 12 Optimizations
description: ""
date: 2023-03-25
tags:
  - optimization
categories:
  - 2023
---

## not understanding stack vs heap

{{< columns >}}
![gostack](/imgs/gostack.png)

<--->

![govalidstack](/imgs/govalidstack.png)
{{< /columns >}}

<!--more-->

分配在heap的case

1. 全局变量
2. 发送给channel的指针

   ```go
   type Foo struct{ s string }
   ch := make(chan *Foo, 1)
   foo := &Foo{s: "x"} // Foo对象分配在heap
   ch <- foo
   ```

3. 发送给channel的值所引用的对象

   ```go
   type Foo struct{ s *string }
   ch := make(chan Foo, 1)
   s := "x"
   bar := Foo{s: &s} // s对象分配在heap
   ch <- bar
   ```

`go build -gcflags "-m=2"` 查看编译的escape(分配到heap)的情况

## not knowing how to reduce allocations

常见方式

1. 使用strings.Builder
2. 避免 []byte 转为 strings
3. 预先设置slice和map的大小
4. 优化struct对齐方式

### 改变API

```go
type Reader interface {
  Read(p []byte) (n int, err error)
}
```

```go
type Reader interface {
  Read(n int) (p []byte, err error)
}
```

第二种方式设计的API必然会在heap上分配，而第一种方式将决定权放在了调用者一方

### 编译阶段自动优化

```go
type cache struct {
  m map[string]int
}
func (c *cache) get(bytes []byte) (v int, contains bool) {
  key := string(bytes)
  v, contains = c.m[key]
  return
}
```

```go
func (c *cache) get(bytes []byte) (v int, contains bool) {
  v, contains = c.m[string(bytes)]
  return
}
```

方式二, compiler不会做[]byte到string的转化

### 使用sync.Pool

```go
func write(w io.Writer) {
  b := getResponse()
  _, _ = w.Write(b)
}
```

```go
var pool = sync.Pool{
  New: func() any {
    return make([]byte, 1024)
  },
}
func write(w io.Writer) {
  buffer := pool.Get().([]byte)
  buffer = buffer[:0]
  defer pool.Put(buffer)
  getResponse(buffer)
  _, _ = w.Write(buffer)
}
```

对比第一种方式, 频繁创建对象时可以使用sync.Pool的方式

    When are objects drained from the pool? There’s no specific method to do this: it
    relies on the GC. After each GC, objects from the pool are destroyed
  
## not relaying on inling

1.9版本之后支持 mid-stack inline
当一个mid函数体中包含fast-path, 和slow-path时，由于mid整体的复杂度高不会inline,
把slow-path封装到独立函数时，mid函数就可以被compiler执行inline优化

## not using go diagnostics tooling

### profiling

### execution tracer
