---
title: 设计模式（五）
date: 2023-04-07 20:03:30
tags: 设计模式
---

### 说明

在此是 设计模式-创建型 部分的简单整理


### 单例模式

#### 解决问题

- 处理资源访问冲突。 例如 写同一文件
- 表示全局唯一类。 例如 配置信息类、唯一自增ID生成类

#### 实现方式

- 饿汉方式： 类加载时 直接创建
- 懒汉方式： 需要时创建

#### 一些问题

- 隐藏了依赖关系， 不利于阅读
- 可扩展性差
- 可测试性差； 在有外部依赖时，很难替换（mock）
- 不支持传参构造（或者说传参会有歧义）


### 工厂模式

- 第一种情况：当代码中存在 if-else 分支判断，动态地根据不同的类型创建不同的对象。针对这种情况，我们就考虑使用工厂模式，将这一大坨 if-else 创建对象的代码抽离出来，放到工厂类中。
- 还有一种情况，尽管我们不需要根据不同的类型创建不同的对象，但是，单个对象本身的创建过程比较复杂，需要组合其他类对象，做各种初始化操作。在这种情况下，我们也可以考虑使用工厂模式，将对象的创建过程封装到工厂类中。

#### 简单工厂

```go
package simple_factory

type Parser interface {
	Parse(filepath string) error
}

// CreateParser 简单工厂
func CreateParser(ext string) Parser {
	switch ext {
	case "json":
		return JsonParser{}
	case "yaml":
		return YamlParser{}
	}

	return nil
}

type JsonParser struct{}

func (j JsonParser) Parse(filePath string) error {
	// do something
	
	return nil
}

type YamlParser struct{}

func (y YamlParser) Parse(filePath string) error {
	// do something
	
	return nil
}
```

#### 工厂方法

当工厂的构建比较复杂时 我们就可以把构造过程单独抽象成类或者方法，避免入口函数过于复杂

```go
package func_factory

type ParseFactory interface {
	CreateParser() Parser
}

var parseFactoryMap = map[string]ParseFactory{
	"json": JsonParseFactory{},
	"yaml": YamlParseFactory{},
}

// GetParseFactory 工厂方法
func GetParseFactory(ext string) ParseFactory {
	return parseFactoryMap[ext]
}

type JsonParseFactory struct {
}

func (j JsonParseFactory) CreateParser() Parser {
	var parse = new(JsonParser)
	// do something

	return parse
}

type YamlParseFactory struct {
}

func (y YamlParseFactory) CreateParser() Parser {
	var parse = new(YamlParser)
	// do something

	return parse
}

type Parser interface {
	Parse(filePath string) error
}

type JsonParser struct{}

func (j JsonParser) Parse(filePath string) error {
	return nil
}

type YamlParser struct{}

func (y YamlParser) Parse(filePath string) error {
	return nil
}
```

#### 抽象工厂

让一个工厂负责创建多个不同类型的对象。为应对多种规则产生不同的类组合而产生的一种解决方案。

提供一个接口，用于创建相关或依赖对象的家族，而不需要明确指定具体类。

#### 后言

其实在支持函数式编程的语言中 完全不用创建工厂类，构造一个 `map[string]func` 就可以了


### 建造者模式

当传递参数过多、参数之间有依赖 或者 需要构建不可变对象（不能创建后 再通过set更新属性值）时，就可以考虑使用 建造者模式。

<details>
  <summary>建造者示例</summary>

    package builder
    
    import "errors"
    
    const (
        DefaultMaxTotal = 8
        DefaultMinIdle  = 0
        DefaultMaxIdle  = 8
    )
    
    func NewBuilder() Builder {
        return Builder{
            name:     "",
            maxTotal: DefaultMaxTotal,
            minIdle:  DefaultMinIdle,
            maxIdle:  DefaultMaxIdle,
        }
    }
    
    type Builder struct {
        name     string
        maxTotal int
        minIdle  int
        maxIdle  int
    }
    
    func (b *Builder) SetName(name string) {
        b.name = name
    }
    
    func (b *Builder) SerMaxTotal(total int) {
        b.maxTotal = total
    }
    
    func (b *Builder) SetMaxIdle(idle int) {
        b.maxIdle = idle
    }
    
    func (b *Builder) SetMinIdle(idle int) {
        b.minIdle = idle
    }
    func (b *Builder) Build() (pool *ResourcePool, err error) {
        if b.name == "" {
            return nil, errors.New("name empty")
        }
        if b.maxTotal < b.maxIdle {
            return nil, errors.New("maxTotal can not be less than maxIdle")
        }
        if b.maxIdle < b.minIdle {
            return nil, errors.New("maxIdle can not be less than minIdle")
        }
        // ... other check
    
        pool = &ResourcePool{
            name:     b.name,
            maxTotal: b.maxTotal,
            minIdle:  b.minIdle,
            maxIdle:  b.maxIdle,
        }
    
        return
    }
    
    type ResourcePool struct {
        name     string
        maxTotal int
        minIdle  int
        maxIdle  int
        // ...
    }


    // use demo
    pool, err := NewBuilder().
		SetName("test").
		SerMaxTotal(20).
		Build()

</details>


### 原型模式

基于一个已有对象创建出一个新的对象。

"深拷贝与浅拷贝"



