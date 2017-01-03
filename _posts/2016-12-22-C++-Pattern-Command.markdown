---
layout: post
title:  "C++ Pattern-Command"
categories: C++
tags: Pattern
---

* content
{:toc}

命令模式有点复杂，因为涉及的类和对象有点多，但对命令或者操作的封装解耦起到很好的作用，而且能轻松实现撤销和重做等操作。





这里先贴命令模式通用UML类图，稍后再贴C++代码实现UML类图
![](https://raw.githubusercontent.com/cheng668/image/master/%E5%91%BD%E4%BB%A4%E6%A8%A1%E5%BC%8F.png)

### 一.意图：

* 用对象封装请求，使请求对客户参数化，可实现记录请求日志和撤销操作。

### 二.适用性：

* 抽象出待执行的动作以使动作对象参数化，Command模式是回调机制的一个面向对象的替代品。
* 在不同时刻指定、排列和执行请求。
* 支持取消操作，要维护一个历史列表。
* 支持日志恢复系统。
* 提供对一个事务进行建模的方法，因为Command提供原语的公共接口。

### 三.参与者：

这里提供下待实现的UML类图，本类图的c++实现代码放在博主github上，文章最后会提供链接。

![](https://raw.githubusercontent.com/cheng668/image/master/%E5%91%BD%E4%BB%A4%E6%A8%A1%E5%BC%8F2.png)

#### Command
* 执行操作的接口

#### ConcreteCommand (ReadCommand、 MacroCommand)
* 把一个接收者对象绑定于一个动作。
* 实现Execute()，调用接收者操作

#### Cilent (Application)
* 创建具体命令对象并设定接收者，发送命令

#### Invoker (MenuItem)
* 要求命令执行请求

#### Receiver (String)
* 请求接收者，执行相应操作，可以是任何类，这里为了方便，用了string类型

### 四.协作：

下图为此模式的时序图,用于理解各个参与对象的协作关系：

![](https://raw.githubusercontent.com/cheng668/image/master/%E5%91%BD%E4%BB%A4%E6%A8%A1%E5%BC%8F%E6%97%B6%E5%BA%8F%E5%9B%BE.png)

* Cilent创建一个ConcreteCommand对象并指向它的Receiver对象。
* Invoker对象存储该ConcreteCommand对象。
* Invoker对象调用 Command对象的 Execute操作提交一个请求，若可撤销，则在 ConcreteCommand执行 Execute前存储当前状态。
* ConcreteCommand对象调用Receiver的一些操作实现请求。

### 五.效果：

* 将调用操作的对象（cilent）和实现操作的对象（receiver）解耦。
* Command子类可被操纵和扩展。
* 可用Composite模式实现符合命令MacroCommand类
* Command类扩展性强。

### 六.实现：

* 命令对象智能程度不定。
* 可实现撤销和重做操作， ConcreteCommand类需要存储额外状态信息：接收者对象、执行操作的参数、操作执行前的状态等，可把命令用 Prototype模式放进历史列表。
* 防止取消和重做操作积累错误。可用 Memento模式将状态信息封装起来。
* 对不能被取消和不需参数的命令可使用c++模板实现以减少Command子类。

### 七.应用：

* 类似于C++中的functors（仿函数类、可调用类）通过重载 operator()函数得到透明性，但 Command模式着重于接收者和函数（动作）之间的绑定，而不仅是维护一个函数。

### 八.相关模式：

* Composite模式：实现命令组合
* Memento模式：保存状态参数，用于取消操作
* Prototyte模式：放进历史列表前必须被拷贝的命令

### 九.代码：

**[命令模式源码例子 🇨🇳](https://github.com/cheng668/Pattern-Command)**