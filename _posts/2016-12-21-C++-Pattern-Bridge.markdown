---
layout: post
title:  "C++ Pattern-Bridge"
categories: C++
tags: Pattern
excerpt: C++设计模式之桥连模式
---

* content
{:toc}

桥连模式
![](https://raw.githubusercontent.com/cheng668/image/master/%E6%A1%A5%E8%BF%9E%E6%A8%A1%E5%BC%8F.png)

### 一.意图：

* 将抽象部分与它的实现部分分离，使它们都可以独立地变化。

### 二.适用性：

* 解除抽象和实现部分的绑定关系。在运行期间实现部分可以被选择或者切换。
* 类的抽象及实现原本都应该通过生成子类的方法进行扩充。Bridge使可以它们分离并可分别进行扩充。
* 对抽象的实现部分的修改不影响客户代码，客户不需重新编译。
* 对客户隐藏抽象的实现部分。
* 本该如图实现的类层次结构：
![](https://raw.githubusercontent.com/cheng668/image/master/%E6%A1%A5%E8%BF%9E%E6%A8%A1%E5%BC%8F2.png)
* 想在多个对象间共享实现（引用计数），一个简单例子就是string类的是实现。博主在github上有
**[引用计数string代码](https://github.com/cheng668/ME29-String/blob/master/String/String.h)**

### 三.参与者：

#### Abstraction(Window)
* 抽象类接口
* 维护指向Implementor类型对象的指针

#### RefinedAbstraction (IconWindow、 ApplicationWindow)
* 扩充Astraction接口

#### Implementor (WindowImp)
* 实现类接口，Abstraction接口定义基于此接口更高层次的操作

#### ConcreteImplementor (XWindowImp、 PMWindowImp)
* 实现Implementor接口并定义具体实现

### 四.协作：

* Abstraction将客户的请求转发给它的Implementor对象。

### 五.效果：

#### 分离接口和实现
* 使抽象的实现可在运行时改变
* 降低编译依赖性
* 分层而产生更好的结构化系统

#### 提高可扩充性
* 可分别扩充接口和实现类

#### 实现对客户透明

### 六.实现：

* Implementor仅有一个实现时，每必要创建 Implementor抽象基类。
* 可以引入一个抽象工厂（Abstract Factoty）对象实现 Implementor对象的正确选择和创建
* 可共享Implementor对象，加引用计数
* 用多重继承以public继承Abstraction,以private继承 concreteImplementor,但这样不能实现动态调用。

### 七.应用：

* Libg++类库定义的一些类，如Set,LinkedSet、HashSet、 LinkedList和HashTable，Set是一个抽象类，LinkedList和 HashTable分别是链表和hash表的具体实现。LinkedSet和 HashSet是Set的实现，它们桥接LinkedList和 HashTable，这里并没有抽象Implementor类。

### 八.相关模式：

* Astract Factory模式：可用于创建和配置一个特定的Bridge模式。
* Adapter模式：用来帮助无关的类协同工作，通常在系统设计完成后才会被使用。Bridge则在系统开始时被使用，使得抽象接口和实现部分可独立进行改变。

### 九.代码：

**[桥接模式源码例子 🇨🇳](https://github.com/cheng668/Pattern-Bridge)**