---
layout: post
title:  "C++ Pattern-Decorator!"
date:   2016-12-08 12:01:54
categories: C++
tags: Pattern
excerpt: C++设计模式之装饰模式
---

![](https://raw.githubusercontent.com/cheng668/image/master/%E8%A3%85%E9%A5%B0%E6%A8%A1%E5%BC%8F.png)

## 一.意图：

* 动态给对象添加一些额外职责，比生成子类更灵活。

## 二.适用性：

* 不影响其他对象前提下，动态、透明地给单个对象添加职责；
* 处理可撤销的职责；
* 不能采用生成子类方式进行扩充时。

## 三.参与者：

### Component(visualComponent):
* 定义一个对象接口，可给这些对象添加职责
### ConcreteComponent(TextView):
定义一个对象，可以给这个对象添加一些职责
### Decorator:
* 维持一个指向Component对象的指针，并定义一个与Component接口一致的接口
### ConcreteDecorator(BorderDecorator,ScrollDecorator)
* 向组件添加职责

## 四.协作：

* Decorator将请求转发给它的Component对象，并可发送前后作附加动作

## 五.效果：

### 优点：
* 比静态继承更灵活，防止子类爆炸，方便增删
* 避免高层结构的类有太多特征，特征可移至Decorator中
### 缺点：
* Decorator是透明包装，使用时不应依赖对象标识
* 产生很多仅相互连接方式上不同，属性值和类相同的相似小对象，难以学习和维护

## 六.实现：

* 接口一致（Draw()）
* 仅需一个职责，可省略抽象Decorator类
* 保持Component接口类简单性
* 修饰模式：改变外壳，装饰对组件透明，接口需一致
  策略模式：改变内核，引用和维护相应策略（装饰），可以有自己的接口
当Component类原本就很庞大时，使用Decorator模式代价太高， Strategy模式相对更好一些。在 Strategy模式中，组件将它的一些行为转发给一个独立的策略对象，我们可以替换Strategy对象，从而改变或扩充组件的功能。

## 七.应用：

* InterViews,ET++,ObjectWorks\Smalltalk类库；
* Streams类：Component(MemoryStream,FileStream)
             Decorator(ASCII7Stream,CompressingStream)
             接口：HandleBufferFull()

## 八.相关模式：

* Adapter模式： Decorator模式不同于Adapter模式，因为装饰仅改变对象的职责而不改
变它的接口；而适配器将给对象一个全新的接口。
* Composite模式：可以将装饰视为一个退化的、仅有一个组件的组合。然而，装饰仅
给对象添加一些额外的职责—它的目的不在于对象聚集。
* Strategy模式：用一个装饰你可以改变对象的外表；而Strategy模式使得你可以改变
对象的内核。这是改变对象的两种途径。

## 九.源码例子：

**[修饰模式源码例子 🇨🇳](https://github.com/cheng668/Pattern-Decorator)**