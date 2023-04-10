---
title: 设计模式（六）
date: 2023-04-10 19:37:52
tags: 设计模式
---

### 说明

在此是 设计模式-结构型 部分的简单整理

结构型模式主要总结了一些类或对象组合在一起的经典结构

### 代理模式

在不改变原始类（被代理类）的情况下，通过引入代理类来给原始类附加功能。

<details>
    <summary>代理模式示例</summary>

    package proxy

    import "context"
    
    type UserController struct {
    }
    
    func (u UserController) Login(c context.Context) {
        // do something
    }
    
    func (u UserController) Register(c context.Context) {
        // do something
    }
    
    type RequestInfo struct {
        ApiName    string
        Timestamp  int64
        ResponseMs int32
    }
    
    type MetricReporter struct {
    }
    
    func (m MetricReporter) Reporter(info RequestInfo) {
        // reporter info
    }
    
    type UserControllerProxy struct {
        userControllerIns UserController
        metricReporterIns MetricReporter
    }
    
    func (u UserControllerProxy) Login(c context.Context) {
        var info RequestInfo
        defer func() {
            // metric cal
            u.metricReporterIns.Reporter(info)
        }()
    
        u.metricReporterIns.Reporter(info)
    }
    
    func (u UserControllerProxy) Register(c context.Context) {
        var info RequestInfo
        defer func() {
            // metric cal
            u.metricReporterIns.Reporter(info)
        }()
    
        u.userControllerIns.Register(c)
    }

</details>

`动态代理` 没必要那就不用（go 中可以用 go generate生成依赖关系代码）

#### 应用场景

- 业务系统的非功能性开发。如监控、日志、限流等
- RPC、缓存应用


### 桥接模式

“Decouple an abstraction from its implementation so that the two can vary independently”

“将抽象和实现解耦，让它们可以独立变化”

“一个类存在两个（或多个）独立变化的维度，我们通过组合的方式，让这两个（或多个）维度可以独立进行扩展”

ep: `gorm` 只依赖于 `Dialector`接口实现，在使用是 注册不同的实现（如下mysql/pg） 即可完成不同数据库的切换
```go

import (
  "gorm.io/driver/mysql"
  "gorm.io/gorm"
)

dsn := "{mysql_dns}"
DB, err = gorm.Open(mysql.Open(dsn), &gorm.Config{})


import (
"gorm.io/driver/postgres"
"gorm.io/gorm"
)

dsn := "pg_dns"
db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
```


### 装饰器模式

"用组合代替继承"

#### 特点/作用

- 装饰器类和原始类继承同样的父类，这样我们可以对原始类“嵌套”多个装饰器类

- 装饰器类是对功能的增强

示例如下： gzip, buf 均实现了 基于`io.Reader`的扩展（功能增强） 我们何以嵌套使用

<details>
    <summary>demo</summary>

    package decorator
    
    import (
    "bufio"
    "bytes"
    "compress/gzip"
    "io"
    "os"
    )
    
    func main() {
        var count int
        var err error
        var f *os.File
        var g io.Reader
    
        f, err = os.Open("demo.txt") // 返回的*os.File是io.Reader的一个实现 后面的gzip、buf可以说都是委托它来读取的
        if err != nil {
            panic(err)
        }
        defer f.Close()
    
        g, err = gzip.NewReader(f)
        if err != nil {
            panic(err)
        }
    
        reader := bufio.NewReader(g)
        buffer := bytes.NewBuffer(make([]byte, 0))
        part := make([]byte, 1000)
    
        for {
            if count, err = reader.Read(part); err != nil {
                panic(err)
            }
            buffer.Write(part[:count])
        }
    
        return
    }

</details>


### 适配器模式

将不兼容的接口转换成兼容的接口

#### 应用场景

- 封装有缺陷的接口设计
- 同一多个类的接口设计： 将某类功能的不同实现（可能依赖外部系统）封装成同一的接口定义，这样我们就可以利用多态的特性，实现代码的复用
- 替换依赖
- 兼容旧版本接口

<details>
<summary>demo</summary>

    package adaptor
    
    type ITarget interface {
        F1()
        F2()
    }
    
    type Adaptee struct {
    }
    
    func (a Adaptee) A1() {
    }
    
    func (a Adaptee) A2() {
    }
    
    type Adaptor struct {
        a Adaptee
    }
    
    func (a Adaptor) F1() {
        a.a.A1()
    }
    
    func (a Adaptor) F2() {
        a.a.A2()
    }


</details>


### 门面模式

```
Provide a unified interface to a set of interfaces in a subsystem. Facade Pattern defines a higher-level interface that makes the subsystem easier to use.
```

"为子系统提供一组统一的接口，定义一组高层接口让子系统更易用。"

分装下层接口，让使用方更加方便（易用性）。


### 组合模式

主要是用来处理树形结构数据（一组对象集合）。

```
Compose objects into tree structure to represent part-whole hierarchies.Composite lets client treat individual objects and compositions of objects uniformly.
```

```
将 一组对象 组织（Compose）成树形结构，以表示一种“部分-整体”的层次结构。组合让客户端（代码的使用者）可以统一单个对象和组合对象的处理逻辑（递归）。

将 一组对象 组织成树形结构，将单个对象和组合对象都看做树中的节点，以统一处理逻辑，并且它利用树形结构的特点，递归地处理每个子树，依次简化代码实现。
```

与其说是一种设计模式，倒不如说是对业务场景的一种数据结构和算法的抽象。

示例：`Dir` `File` 实现了统一接口定义 `FileSystemNode`，组成树状结构，递归的方式完成逻辑计算。

<details>
    <summary>demo</summary>

    package main

    import "fmt"
    
    type FileSystemNode interface {
        CountNumOfFiles() int
        GetPath() string
    }
    
    func NewFile(path string) *File {
        return &File{path: path}
    }
    
    type File struct {
        path string
    }
    
    func (f File) GetPath() string {
        return f.path
    }
    
    func (f File) CountNumOfFiles() int {
        return 1
    }
    
    func NewDir(path string) *Dir {
        return &Dir{path: path}
    }
    
    type Dir struct {
        path  string
        nodes []FileSystemNode
    }
    
    func (d Dir) GetPath() string {
        return d.path
    }
    
    func (d Dir) CountNumOfFiles() int {
        var count int
        for _, node := range d.nodes {
            count += node.CountNumOfFiles()
        }
    
        return count
    }
    
    func (d *Dir) AddNode(node FileSystemNode) {
        d.nodes = append(d.nodes, node)
    }
    
    func (d *Dir) RemoveNode(path string) {
        var index int = 0
        for ; index < len(d.nodes) && d.nodes[index].GetPath() != path; index++ {
        }
    
        if index < len(d.nodes) {
            d.nodes = append(d.nodes[:index], d.nodes[index+1:]...)
        }
    }
    
    func main() {
        f1 := NewFile("a.txt")
        f2 := NewFile("t2/b.txt")
    
        d2 := NewDir("t2")
        d2.AddNode(f2)
        fmt.Println(d2.CountNumOfFiles())
    
        d1 := NewDir("")
        d1.AddNode(d2)
        d1.AddNode(f1)
        fmt.Println(d1.CountNumOfFiles())
    
        d2.RemoveNode("t2/b.txt")
        fmt.Println(d2.CountNumOfFiles())
        fmt.Println(d1.CountNumOfFiles())
    }

</details>


### 享元模式

"被共享的单元"

享元模式的意图是复用对象，节省内存，前提是享元对象是不可变对象。

例如： 棋牌游戏中，我们用`编号`来表示`棋子`的信息（名称、颜色, 创建好后 固定不变），然后`棋盘`数据结构中只需要维护`编号`+`坐标`信息即可
