---
layout: post
title:  "C++ Pattern-Composite!"
date:   2016-12-09 15:23:12
categories: C++
tags: C++
excerpt: C++设计模式之组合模式
---

![github](https://github.com/cheng668/image/blob/master/%E7%BB%84%E5%90%88%E6%A8%A1%E5%BC%8F.png) 

## 一.意图：
  

* 将对象组合成树形结构以标识“部分-整体”层次结构。Composite和单个对象接口一致

## 二.适用性：
  
* 表示对象的部分-整体层次结构；
* 用户忽略组合和单个对象的不同，统一使用；

## 三.参与者：
  
### Component(Equipment):
* 为组合中对象声明接口；
* 实现所有类共有接口的缺省行为；
* 声明一个接口用于访问和管理Component的子组件；
* （可选）在递归结构中定义一个接口，用于访问一个父不见，并在合适情况下实现它
### Leaf(FloppyDisk):
* 在组合中表示叶节点，没子节点；
* 在组合中定义图元对象的行为
### Composite(CompositeEquipment,Chassis):
* 定义有子部件的那些部件的行为；
* 存储子部件；
* 在Component接口中实现与子部件有关的操作
### Client:
* 通过Component接口操纵组合部件的对象

## 四.协作：
  
* 用户使用Component类接口与组合结构中的对象进行交互。如果接收者是一个叶节点，则直接处理请求，如果是Composite，
则把请求发送给子部件，并在转发前后执行一些辅助操作。

## 五.效果：
  
### 优点：
* 组合对象和基本对象对客户同名
* 单个对象或者组合对象可递归组成新的组合对象；
* 容易新增新类型组件；
### 缺点：
* 组合中不能限制特定单个对象

## 六.实现：
  
* 子部件引用父部件，在Component类中定义父部件引用，Leaf和Composite类继承这个引用以及管理这个引用的操作，父部件维护一个不变式；
* 共享组件，共享组件可以减少对存储的需求，可以使用Flywight方式来实行共享。
* 最大化Component接口，为Leaf和Composte类尽可能对定义一些缺省公共操作
* 在Component中声明管理子部件的操作，舍弃安全性获取透明性。并可在Leaf类中Add()和Remove()中抛出异常告知客户不可用，
或者定义一个返回Composite对象的GetComposite()函数判断是组合函数叶子。
* Component列表不应在Component类中实现，浪费空间，除非叶子少；
* 子部件排序，必须仔细地设计对子节点的访问和管理接口，以便管理子节点序列。Iterator模式可以在这方面给予一些定的指导；
* 使用高速缓冲存贮对子节点的遍历或者查找的相关信息可以改善性能，但子部件变化时应通知父部件缓存失效，
所以需要在Component中定义一个private接口判断是否有效。
* Composite被销毁时由Composite删除其子节点，但Leaf对象不会改变，因此可以被共享不删除；
* 存贮组件用效率高的数据结构，对每个子节点Composite 都有一个变量与之对应，这就要求Composite的每个子类都要实现自己的管理接口。
    参见Interpreter模式中的例子。

## 七.应用：
  
* InterViews,ET++,Smalltalk中的MVC结构；
 
## 八.相关模式：
  
* Responsibility of Chain模式：实现部件和父部件的连接
* Decoratoe模式：通常和Composite模式一起使用；
* Flyweight模式：共享组件，但不能引用他们的父部件；
* Iterator模式：遍历Composite;
* Visitor模式：将本来应该分布在Composite和Leaf类中的操作和行为局部化。

## 九.源码例子：

**[修饰模式源码例子 🇨🇳](https://github.com/cheng668/Pattern-Composite)**