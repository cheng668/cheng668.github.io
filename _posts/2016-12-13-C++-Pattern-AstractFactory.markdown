---
layout: post
title:  "C++ Pattern-AstractFactory!"
date:   2016-12-13 11:14:00
categories: C++
tags: Pattern
excerpt: C++设计模式之抽象工厂模式
---

![](https://raw.githubusercontent.com/cheng668/image/master/%E6%8A%BD%E8%B1%A1%E5%B7%A5%E5%8E%82%E6%A8%A1%E5%BC%8F.png)

## 一.意图：

* 提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。

## 二.适用性：

* 一个系统要独立于它的产品的创建、组合和表示时
* 一个系统要由多个产品系列中的一个来配置时
* 强调一系列相关产品对象的设计以便进行联合使用时
* 提供一个产品类库，而只想显示它们的接口而不是实现时

## 三.参与者：

### AbstractFactory(WidgetFactory)
* 声明一个创建抽象产品对象的操作接口。
### ConcreteFactory(MotifWidgetFactory,PMWidgetFactory)
* 实现创建具体产品对象的操作。
### AbstractProduct(Windows,ScrollBar)
* 为一类产品对象声明一个接口。
### ConcreteProduct(MotifWindow,MotifScrollBar)
* 定义一个将被相应的具体工厂创建的产品对象
* 事项AbstractProduct接口
### Client
* 仅使用由AbstractFactory和AbstractProduct类声明的接口

##四.协作：

* 运行时创建一个ConcreteFactory类的实例，具体工厂对应不同产品对象。
* AbstractFactory将产品对象的创建延迟到ConcreteFactory子类。

## 五.效果：

### 优点：
* 分离客户与产品的类
* 一个应用中仅有一次创建具体工厂类的地方，便于交换产品系列
* 利于产品一致性
### 缺点：
* 难以支持新种类的产品

## 六.实现：

* 工厂作为单件，用Singleton模式创建
* .AbstractFactory声明创建产品的接口，ConcreteProduct子类实现。

## 七.应用：

* InterView使用“Kit”后缀来表示AbstractFactory类，WidgetKit,DialogKit；
* ET++使用AbstracFactory模式以打到在不同窗口系统（X Windows和SunView）间的可移植性。

## 八.相关模式：

* AbstractFactory类通常用工厂方法（Factory Method）实现，但它们也可以用Prototype实现。
* 一个具体工厂通常时一个单件（Singleton）。

## 九.源码例子：

**[抽象工厂模式源码例子](https://github.com/cheng668/Pattern-AstractFactory)**