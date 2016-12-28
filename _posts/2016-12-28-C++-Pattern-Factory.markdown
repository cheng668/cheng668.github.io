---
layout: post
title:  "C++ Pattern-Factory"
categories: C++
tags: Pattern
---

* content
{:toc}

在博主看来，工厂模式就是简化了的抽象工厂模式，抽象工厂模式经常会用到工厂模式，抽象工厂模式用于产生一系列相关产品，但工厂模式只产出单一产品。工厂模式相对简单，知识点比较少，这节会用上模板代替继承类实现工厂类。






![](https://raw.githubusercontent.com/cheng668/image/master/%E5%B7%A5%E5%8E%82%E6%A8%A1%E5%BC%8F.png)

### 一.意图：

* 定义创建对象的接口，让子类决定实例化哪个类。

### 二.适用性：

* 当一个类不知道它所必须创建的对象的类时。
* 希望由子类指定创建的对象。
* 将创建信息局部化

### 三.参与者：

#### Product (Document)
* 工厂方法所需创建的对象的接口。

#### ConcreteProduct (MyDocument, YourDocument)
* 实现Product接口。

#### Creator (Application)
* 声明工厂方法，返回一个Product对象，可缺省。
* 可调用工厂方法以创建Product对象。

#### ConcreteCreator (MyApplication, YourApplication)
* 实现 Accept操作。

### 四.协作：

* Creator子类定义工厂方法，并返回 ConcreteProduct实例

### 五.效果：

#### 优点

* 将应用相关的类和代码分离。
* 为子类提供挂钩，扩展产品。
* 将应一起工作的工作信息局部化。

#### 缺点

* 客户仅需创建一个特定的 ConcreteProduct时，Creator有点多余。

### 六.实现：

* Creator类不实现工厂方法接口，这样Creator不可实例化；也可缺省，这样Creator的子类作为补充。
* 可以参数化工厂方法，工厂根据产品ID生产产品。
* 注意 Creator类构造函数中不能调用工厂函数，工厂函数可以用 Lazy initialization(《More Effective C++》item M17)。
* 工厂子类可用模板创建
* 命名约定，让客户知道是工厂还是产品。

### 七.应用：

* 工具包和框架。

### 八.相关模式：

* Abstract Factory模式：常用工厂模式实现。
* Template模式：工厂方法常在此模式中被调用。
* Prototypes模式：不需要创建 Creator的子类，但常要求针对 Product类的 Initialize初始化操作。

### 九.代码：

**[建造者模式源码例子 🇨🇳](https://github.com/cheng668/Pattern-Factory.git)**