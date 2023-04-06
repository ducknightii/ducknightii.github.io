---
title: 设计模式（三）
date: 2023-04-04 11:17:33
tags: 设计模式
---


### 说明

在此是 设计原则部分的简单整理


### 单一职责原则（SRP）

`Single Responsibility Principle`

### 开闭原则（OCP）

`Open Close Principle`

"对扩展开放、对修改关闭"

这条原则的设计初衷：只要它没有破坏原有的代码的正常运行，没有破坏原有的单元测试，我们就可以说，这是一个合格的代码改动。

该原则是衡量扩展性的重要标准

### 里氏替换原则（LSP）

`Liskov Substitution Principle`

```
If S is a subtype of T, then objects of type T may be replaced with objects of type S, without breaking the program。
```

```
Functions that use pointers of references to base classes must be able to use objects of derived classes without knowing it。
```

```
子类对象（object of subtype/derived class）能够替换程序（program）中父类对象（object of base/parent class）出现的任何地方，并且保证原来程序的逻辑行为（behavior）不变及正确性不被破坏。
```

里氏替换可以说是用了多态的特性，但是注重的是替换后要`保证程序的逻辑行为不变` （例如 同样的输入 父类正常 在替换成子类后抛出异常 那就违反了该规则）

"Design By Contract" （按照协议来设计）子类在设计的时候要完全遵循父类的行为约定

### 接口隔离原则（ISP）

`Interface Segregation Principle`

```
Clients should not be forced to depend upon interfaces that they not use.
```

不要让使用者（调用者）依赖它们不需要的功能

### 依赖反转原则（DIP）

`Dependency Inversion Principle`

```
High-level modules shouldn’t depend on low-level modules. Both modules should depend on abstractions. In addition, abstractions shouldn’t depend on details. Details depend on abstractions.

高层模块（high-level modules）不要依赖低层模块（low-level）。高层模块和低层模块应该通过抽象（abstractions）来互相依赖。除此之外，抽象（abstractions）不要依赖具体实现细节（details），具体实现细节（details）依赖抽象（abstractions）。
```

IOC：控制反转

DI：依赖注入   提高了代码的扩展性，我们可以灵活地替换依赖的类。 它是编写可测试性代码最有效的手段。



------

### KISS原则 && YAGNI原则

"Keep It Simple And Stupid"

"You Ain't Gonna Need It"  不要过度设计（不需要不必要的实现 但是扩展点或者说可扩展性还是要考虑的）


### DRY原则

"Don't Repeat Yourself"

写出可复用性好的代码

"我们可以不写可复用的代码，但一定不能写重复的代码"


### 迪米特法则（LOD）

`Law Of Demeter`

`The Least Knowledge Principle`

```
不该有直接依赖关系的类之间，不要有依赖；有依赖关系的类之间，尽量只依赖必要的接口（也就是定义中的“有限知识”）。
```

"高内聚、松耦合"


### 总结

“创造的一大秘诀是要懂得如何隐藏你的来源”

系统设计的一个基本流程：

1. 合理地将功能划分到不同的模块
2. 设计模块与模块之间的交互关系
3. 设计模块的接口、数据库、业务模型


对于一个非业务通用框架的开发要考虑的点：

1. 易用性
2. 性能
3. 扩展性： 这里的扩展性 是指使用方的扩展性，在不改变源码的情况下 定制需求
4. 容错性
5. 通用性
