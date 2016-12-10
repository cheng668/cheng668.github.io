---
layout: post
title:  "C++ Pattern-Strategy!"
date:   2016-06-08 15:14:54
categories: C++
tags: C++
excerpt: C++设计模式之策略模式
---

![Image](https://github.com/cheng668/image/blob/master/%E7%AD%96%E7%95%A5%E6%A8%A1%E5%BC%8F.png)

## 一、意图：

* 定义一系列算法，把他们一个个封装起来，并使它们可相互替换。使算法独立与客户而变化。

## 二、适用性：

* 许多相关类仅仅是行为有异，用多个行为中一个配置一个类；
* 需要使用一个算法的不同变体时；
* 算法对客户透明；
* 一个类中多种行为以多个条件句形式出现时。

## 三、参与者：

### 1.Strategy(策略接口，Compositor)
* 定义所有支持的算法的公共接口

### 2.ConcreteStrategy(具体策略，SimpleCompositor,TeXCompositor,ArrayCompositor)
* 以Strategy接口实现某具体算法

### 3.Comtext(上下文，Composition)
* 用一个ConcreteStrategy对象来配置；
* 维护一个Strategy对象；
* 可定义一个接口来让Strategy访问它的数据

## 四、协作：

* Strategy和Context相互作用以实现选定的算法，Context可把自身或者所需数据传给Strategy；
* Context将客户请求转发给Strategy，客户可选择ConcreteStrategy；

## 五、效果

### 优点：
* 相关算法系列，Strategy类层次为Context定义了一系列的可供重用的算法或行为。继承有助于析取出这些算法中的公共功能。
* 一个替代继承的方法，更易切换、理解、扩展。
* 消除一些条件语句
* 策略可供客户选择

### 缺点：
* 客户必须了解不同的Strategy，否则只能向客户暴露具体实现
* Strategy和Context之间的通信开销，某些ConcreteStrategy可能不适用Context传过滤的信息，这时需要Strategy和Context之间耦合更紧密
* 增加对象的数目，可将Strategy实现为可供Context共享的无状态的对象来减少这一开销，状态由Context维护，Strategy不应在歌词条用之间维护状态，Flyweighr模式有描述。

## 六、实现

* 定义Strategy和Context接口，Context可传所需数据到ConcreteStrategy中,解耦；也可把自身发给Strategy，紧密耦合。
* 可将Strategy作为模板参数，这样可以不需定义Strategy接口的抽象类，提高效率。但要求（1）可在编译时选择Strategy；（2）不需在运行时改变。
````
template<class AStrategy>
class Context{
	void Operation(){ theStrategy.DoAlgorithm(); }
	//...
private:
	AStrategy theStrategy;
};

class MyStrategy{
public:
	void DoAlgorithm();
};

Context<MyStrategy> aContext;
````

* 使Strategy对象成为可选的，对Context可缺省

## 七、应用：

* ET++ 和 InterViews用Strategy来封装不同的换行算法；
* ObjectWindows使用Validator对象封装验证策略；

## 八、相关模式：

* Flyweight模式：Strategy对象经常是很好的轻量级对象

## 九.源码例子：

**[修饰模式源码例子 🇨🇳](https://github.com/cheng668/Pattern-Strategy)**