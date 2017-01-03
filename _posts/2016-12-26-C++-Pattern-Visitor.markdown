---
layout: post
title:  "C++ Pattern-Visitor"
categories: C++
tags: Pattern
---

* content
{:toc}

本文是博主学习的用C++实现的访问者模式，其中有个很重要的概念是双分派（double-dispatch）就是用访问者模式实现的，双分派在《More Effective C++》--第31章：让函数根据一个以上的对象来决定怎么虚拟--中有详细讨论，这里只简单实现基本的访问者模式，本给出代码链接。





这里先贴访问者模式通用UML类图，稍后再贴C++代码实现UML类图
![](https://raw.githubusercontent.com/cheng668/image/master/%E8%AE%BF%E9%97%AE%E8%80%85%E6%A8%A1%E5%BC%8F2.png)

### 一.意图：

* 将对象结构中的对元素的操作和元素本身解耦。

### 二.适用性：

* 对不同接口的类对象实施依赖该对象的操作。
* 对一个对象结构中的对象进行不同的操作。
* 对象结构的类很少改变。

### 三.参与者：

#### Visitor (访问者)
* 为每个 ConcreteElement声明一个Visit操作。

#### ConcreteVisitor (具体访问者)
* 实现Visitor声明的操作或算法。

#### Element (元素)
* 定义一个 Accept操作，入参为访问者。

#### ConcreteElement (具体元素)
* 实现 Accept操作。

#### ObjectStructure (对象结构)
* 能枚举元素。
* 可提供接口让访问者访问它的元素。
* 可以是一个复合（Composite）或者一个集合。

### 四.协作：

下图是访问者模式的协作图：

![](https://raw.githubusercontent.com/cheng668/image/master/%E8%AE%BF%E9%97%AE%E8%80%85%E6%A8%A1%E5%BC%8F%E6%97%B6%E5%BA%8F%E5%9B%BE.png)

* 客户用ConcreteVisitor对象访问对象结构中的每个元素。
* 当元素被访问时，要调用对应它的类的Visitor操作。

### 五.效果：

#### 优点

* 易于添加新操作。
* 访问者集中相关操作而分离无关操作。父类集中相关操作，子类存放无关操作。
* 增加新的 ConcreteElement困难。

#### 缺点

* 与Iterator模式相比，元素并不一定要继承统一父类。是通过类层次进行的访问。
* 访问者访问对象结构的每个元素时可能会积累状态。
* 访问者可能需要操作 ConcreteElement中的内部结构，要破坏封装。

### 六.实现：

* 每个对象结构有一个对应的Visitor类。如 VisitorConcreteElementA(), VisitorConcreteElementB()
* 每个 ConcreteElement类实现一个 Accept() 操作，调用访问者中相应本类的 Visit...操作。
* 实现双分派问题 （Double-dispatch）：执行的操作决定与请求的种类 (Accept)和两个接收者的类型 (Visitor和 Element),而C++本来是单分派语言（操作决定于请求的种类和一个接收者的类型）。
* 遍历对象结构可以是对象结构本身，也可以用迭代器对象（内和外），或者由访问者实现遍历算法（可实现依赖对象结构操作结果的复杂遍历）。

### 七.应用：

* 游戏中的对象之间的碰撞。

### 八.相关模式：

* Composite模式：访问者可以对一个由Composite模式定义的对象结构进行操作。
* Interpreter模式：访问者可用于解析。

### 九.代码：

**[访问者模式源码例子 🇨🇳](https://github.com/cheng668/Pattern-Visitor)**