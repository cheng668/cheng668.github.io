---
layout: post
title:  "C++ Pattern-Iterator"
categories: C++
tags: Pattern
---

* content
{:toc}

本文是博主学习的用C++实现的迭代器模式，除了理论知识外，还会贴出代码链接。





这里先贴迭代器模式通用UML类图，稍后再贴C++代码实现UML类图
![](https://raw.githubusercontent.com/cheng668/image/master/%E8%BF%AD%E4%BB%A3%E5%99%A8%E6%A8%A1%E5%BC%8F1.png)

### 一.意图：

* 顺序访问一个聚合对象中各个元素而不暴露该对象内部表示。

### 二.适用性：

* 访问一个聚合对象的内容而不暴露它的内部实现。
* 支持多种遍历。
* 为遍历不同聚合结构提供一个统一接口（多态迭代）。

### 三.参与者：

这里提供下待实现的UML类图，本类图的c++实现代码放在博主github上，文章最后会提供链接，博主很懒，这里就不实现 SkipList和 SkipListIterator了。

![](https://raw.githubusercontent.com/cheng668/image/master/%E8%BF%AD%E4%BB%A3%E5%99%A8%E6%A8%A1%E5%BC%8F2.png)

#### Iterator
* 定义访问和遍历元素的接口。

#### ConcreteIterator (ReverseListIterator、 ListIterator、 SkipListIterator)
* 实现迭代器接口。
* 跟踪聚合遍历的当前位置。

#### Aggregate (AbstractList)
* 定义创建相应迭代器对象的接口。

#### ConcreteAggregate (List、 SkipList)
* 实现聚合接口， CreateIterator返回 ConcreteIterator的一个实例。

### 四.协作：

* ConcreteIterator跟踪聚合中的当前对象，并计算出后继对象。

### 五.效果：

* 支持多种方式遍历一个聚合。
* 简化聚合接口
* 同一聚合可有多个遍历。

### 六.实现：

* 根据控制迭代器的用户的不同，可分为外部迭代器和内部迭代器，外部迭代器由客户控制迭代并对当前对象进行操作，而对于内部迭代器，客户只需提供一个操作函数，迭代器会对每个元素实施该操作。外部迭代器更灵活。
* 定义算法位置，可以是聚合定义的遍历算法，此时迭代器只需作为游标（ cursor）标记遍历的当前位置，如果由迭代器负责遍历算法，则比较灵活且可重用。
* 迭代器健壮性的考虑，即当聚合需要增加或删除元素时，遍历依然正确。
* 附加的迭代器操作，最小接口应由如First()、 Next()、 IsDone()、 CurrentItem() 等操作。
* 多态迭代器，要求Factory Method 动态分配迭代器对象，但需要客户负责删除指针，但可以用 Proxy模式实现资源回收。
* 迭代器访问权限，迭代器可作为聚合的友元。
* 用于复合对象（Composite模式）的迭代器，外部迭代器需要维护Composite路径。
* 空迭代器，IsDone()操作总返回true，用于提供统一处理树形结构的遍历的解决方案。

### 七.应用：

* C++标准库的迭代器

### 八.相关模式：

* Composite模式：迭代器常用于复合对象这样的递归结构上。
* Factory Method模式：多态迭代器用此模式实例化子类。
* Memento模式：用于迭代器捕获一个状态。

### 九.代码：

**[建造者模式源码例子 🇨🇳](https://github.com/cheng668/Pattern-Iterator)**