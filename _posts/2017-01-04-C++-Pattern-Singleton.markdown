---
layout: post
title:  "C++ Pattern-Singleton"
categories: C++
tags: Pattern
---

* content
{:toc}

* 单例模式可谓最简单的设计模式，但又是灵活多变的模式，在《More Effective C++》--item M26：限制某个类所能产生的对象数量-- 中就已多种方式实现单例模式，其中剖析了取实例函数作为全局函数和成员函数之间的区别，和为解决取实例函数作为全局函数的命名冲突问题解决方法(用命名域)，还有用模板实现计数和多例模式的方法。
* 因为网上一堆实现懒汉和饿汉、多线程的单例模式，博主在这里就不把重点放在它们身上，但代码中为了实现可继承单例(可和工厂模式一起用)，父类 SinglePainter类用了懒汉模式，子类 ColorSinglePainter用了饿汉模式，代码还实现了 item M26的多例模式 Counted模板和 Printer类，但本博文的概念还是出自《设计模式》。







单例模式的UML图如下：

![](https://raw.githubusercontent.com/cheng668/image/master/%E5%8D%95%E4%BE%8B%E6%A8%A1%E5%BC%8F1.png)

### 一.意图：

* 保证一个类仅有一个实例，并提供一个访问它的全局访问点。

### 二.适用性：

* 当类只能有一个实例且客户可从一个众所周知的访问点访问它。
* 当唯一实例应该通过子类化可扩展的，且客户无需更改代码就能使用一个扩展的实例。

### 三.参与者：

#### Singleton( Printer, SinglePainter, ColorSinglePainter)
* 定义一个 Instance操作，允许客户访问它的唯一实例。Instance是静态成员函数。
* 可能负责创建它自己的唯一实例。

### 四.协作：

* 客户只能通过 Singleton的 Instance操作访问一个 Singleton的实例。

### 五.效果：

#### 优点

* 对唯一实例的受控访问。
* 用static缩小了命名空间。
* Singleton可以有子类并在运行时配置( SinglePainter, ColorSinglePainter)。
* 允许可变数目实例( Printer)。

#### 缺点

* 用static静态成员函数封装单件功能，难以实现多实例和子类重定义。

### 六.实现：

代码dgml图：
![](https://raw.githubusercontent.com/cheng668/image/master/%E5%8D%95%E4%BE%8B%E6%A8%A1%E5%BC%8F2.png)

* 把构造函数放到保护级别以上保证一个唯一的实例。
* 保存 Singleton对象指针的静态数据变量 _instance可以指向 Singleton子类对象。
* 不能将单件定义为一个全局或静态的对象，然后依赖于自动初始化，原因： a)不能保证静态对象只有一个实例会被声明； b)没有足够信息初始化每一个单件； c)全局对象构造顺序不知道，单件不能相互依赖； d)无论是否用到都要创建，浪费资源。
* 创建 Singleton类的子类： a)在父类 Instance函数中根据条件判断 new哪个子类，此方法需要父类知道所有子类； b)在父类中维护一个注册表，子类在创建时进行注册，父类只需在 Instance函数中读取注册表即可，此方法子类需要用饿汉模式。

### 七.应用：

* 数据库读写句柄，打印机读写句柄。

### 八.相关模式：

* Abstract Factory模式、Builder模式、Prototype模式可以用 Singleton模式实现。

### 九.代码：

**[单/多例模式源码例子 🇨🇳](https://github.com/cheng668/ME-26-Singleton.git)**