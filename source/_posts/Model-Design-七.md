---
title: 设计模式（七）
date: 2023-04-19 19:33:34
tags: 设计模式
---

### 说明

在此是 设计模式-行为型 部分的简单整理

行为型设计模式 主要解决的是 类或对象之间的交互问题


### 观察者模式

发布订阅模式

同步、异步实现方式

EventBus


### 模版模式

模板模式主要是用来解决复用和扩展两个问题。

```
Define the skeleton of an algorithm in an operation, deferring some steps to subclasses. Template Method lets subclasses redefine certain steps of an algorithm without changing the algorithm’s structure.
```

```
模板方法模式在一个方法中定义一个算法骨架，并将某些步骤推迟到子类中实现。模板方法模式可以让子类在不改变算法整体结构的情况下，重新定义算法中的某些步骤。
```

示例如下，注：`go`没有继承的概念 以 抽象方法+模版方法 代替 模版类 （这可能更接近`回调`的定义，回调基于接口/抽象类 是组合关系，模版基于父类 是继承关系）

<details>
    <summary>Demo</summary>

    package main

    import "fmt"
    
    type IFunc interface {
        A1()
        A2()
    }
    
    // Do 模版方法
    func Do(iFunc IFunc) {
        iFunc.A1()
        iFunc.A2()
    }
    
    type C1 struct {
    }
    
    func (c C1) A1() {
        fmt.Println("c1.A1")
    }
    
    func (c C1) A2() {
        fmt.Println("c1.A2")
    }
    
    type C2 struct {
    }
    
    func (c C2) A1() {
        fmt.Println("c2.A1")
    }
    
    func (c C2) A2() {
        fmt.Println("c2.A2")
    }
    
    func main() {
        var c1 = new(C1)
        var c2 = new(C2)
    
        Do(c1)
        Do(c2)
    }

</details>

#### 作用/应用

- 复用。如上述示例中 `Do`模版方法
- 扩展。如我们开发所用到的框架`gin`、`beego`以及底层的`http server`等，可以让用户在不改变源码的情况下，定制化扩展功能。


### 策略模式

利用它来避免冗长的`if-else`或`switch`分支判断。目的是为了拆分简化各部分的逻辑。

<details>
    <summary>Demo</summary>

    package strategy
    
    var m = map[string]Func{
        "f1": F1,
        "f2": F2,
    }
    
    type Func func()
    
    func F1() {
        // do something
    } 
    
    func F2() {
        // do something
    }
    
    func GetStrategy(t string) Func {
        return m[t]
    }

</details>


### 职责链模式

在我们进行业务开发时通常利用`middleware`层来做 认证、限流、监控等工作，在这用的就是职责链模式

<details>
    <summary>Demo</summary>

    package main
    
    import "context"
    
    type Chain struct {
        responses []ResponseFunc
    }
    
    func (c *Chain) AddResponseFunc(f ResponseFunc) {
        c.responses = append(c.responses, f)
    }
    
    func (c *Chain) Do(ctx context.Context) {
        for i := 0; i < len(c.responses); i++ {
            f := c.responses[i]
            if f(ctx) {
                return
            }
        }
    }
    
    type ResponseFunc func(c context.Context) bool
    
    func R1(c context.Context) bool {
        // do something
        return false
    }
    
    func R2(c context.Context) bool {
        // do something
        return true
    }
    
    func main() {
        var c context.Context
    
        var chain = new(Chain)
        chain.AddResponseFunc(R1)
        chain.AddResponseFunc(R2)
    
        chain.Do(c)
    }

</details>


### 状态模式

状态机： 状态、事件、动作。 事件触发状态的转移以及动作的执行

"查表法"： 对于状态多 动作简单的场景 可以考虑查表法

<details>
    <summary>Demo</summary>

    package main
    
    import "fmt"
    
    const (
        StateOfSmall = iota + 1
        StateOfSuper
        StateOfCape
        StateOfFire
    )
    
    func NewMario() Mario {
        return Mario{
            state: newSmallMario(),
        }
    }
    
    type Mario struct {
        state iMarioState
    }
    
    func (m *Mario) GetState() int {
        return m.state.GetState()
    }
    
    func (m *Mario) ObtainMushRoom() {
        m.state = m.state.ObtainMushRoom()
    }
    
    func (m *Mario) MeetMonster() {
        m.state = m.state.MeetMonster()
    }
    
    type iMarioState interface {
        GetState() int
    
        ObtainMushRoom() iMarioState
    
        MeetMonster() iMarioState
    }
    
    func newSmallMario() smallMario {
        return smallMario{
            state: StateOfSmall,
            score: 0,
        }
    }
    
    type smallMario struct {
        state int
        score int
    }
    
    func (s smallMario) GetState() int {
        return StateOfSmall
    }
    
    func (s smallMario) ObtainMushRoom() iMarioState {
        return newSuperMario()
    }
    
    func (s smallMario) MeetMonster() iMarioState {
        // do nothing
    
        return s
    }
    
    func newSuperMario() superMario {
        return superMario{
            state: StateOfSuper,
            score: 100,
        }
    }
    
    type superMario struct {
        state int
        score int
    }
    
    func (s superMario) GetState() int {
        return StateOfSuper
    }
    
    func (s superMario) ObtainMushRoom() iMarioState {
        // do nothing
        return s
    }
    
    func (s superMario) MeetMonster() iMarioState {
        return newSmallMario()
    }
    
    func main() {
        var mario = NewMario()
        fmt.Println(mario.GetState())
    
        mario.ObtainMushRoom()
        fmt.Println(mario.GetState())
    
        mario.MeetMonster()
        fmt.Println(mario.GetState())
    }

</details>


### 迭代器模式


### 访问者模式

把业务操作跟具体的数据结构解耦

支持双分派的语言不需要访问者模式
（支持函数式编程的语言应该也不需要 一个匿名函数就解决了）


### 备忘录模式

主要是用来防丢失、撤销、恢复等


### 命令模式


### 解释器模式


### 中介模式
