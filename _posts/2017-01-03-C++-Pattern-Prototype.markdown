---
layout: post
title:  "C++ Pattern-Prototype"
categories: C++
tags: Pattern
---

* content
{:toc}

程序员2001年第11期杂志－理解设计模式（2）原型中提到“如果你将工厂模式的UMl图对折，你得到的就是Prototype原型的UML图”。对原型模式，博主的理解是：作为产品类的对象(原型)可以不需要工厂而自己生产出产品(复制自己)，也就是集工厂和产品于一身的一种模式。







原型模式的UML图如下：

![](https://raw.githubusercontent.com/cheng668/image/master/%E5%8E%9F%E5%9E%8B%E6%A8%A1%E5%BC%8F1.png)

### 一.意图：

* 用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。

### 二.适用性：

* 运行时指定要实例化的类。
* 避免创建工厂层次。
* 要实例化的类的组合数量确定，这时拷贝实例比每次创建实例方便。

### 三.参与者：

#### Prototype
* 声明拷贝自身的接口。

#### ConcretePrototype
* 实现拷贝接口。

#### Client
* 让原型拷贝自身从而创建一个新的对象。

### 四.协作：

* 客户请求一个原型拷贝自身。

### 五.效果：

#### 优点

* 将应用相关的类和客户代码分离。
* 运行时增加和删除产品。
* 客户可改变传入值以创建新对象(原型)。
* 改变复合对象(复合原型)结构内部子对象(原型)以指定新对象。
* 用类动态配置应用。

#### 缺点

* 子类必须实现 Clone操作，类已存在或者不支持拷贝和循环引用时，不能加入 Clone操作，这时应该用工厂模式。

### 六.实现：

* 系统中原型数目不固定，可用原型管理器，博主代码中有实现。
* 实现深度拷贝操作(重点)。
* 可以引入 Initialize操作传入不同参数初始化对象。

### 七.应用：

* VISIO工具包工具，如UML工具包的“类”工具（猜想）。

### 八.相关模式：

* Abstract Factory模式：可存储一个被拷贝的原型的集合，并返回产品对象(原型管理类？)。
* Composite模式和Decorator模式：可从Prototype获益。

### 九.代码：

**[原型模式源码例子 🇨🇳](https://github.com/cheng668/Pattern-Prototype.git)**