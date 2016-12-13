---
layout: post
title:  "C++ Pattern-Builder"
categories: C++
tags: Pattern
---

* content
{:toc}

建造者模式就像KFC或者Mcdonalds的点餐取餐过程

![](https://raw.githubusercontent.com/cheng668/image/master/%E5%BB%BA%E9%80%A0%E8%80%85%E6%A8%A1%E5%BC%8F2.jpg)




建造者模式类图如下，在本例代码中将实现如下类图

![image](https://raw.githubusercontent.com/cheng668/image/master/%E5%BB%BA%E9%80%A0%E8%80%85%E6%A8%A1%E5%BC%8F.png)

### 一.意图：

* 将一个复杂对象的构建与它的表现分离，使得同样的构建过程可以创建不同的表示。

### 二.适用性：

* 当创建复杂对象的算法应该独立于该对象的组成部分以及它们的装配方式时（分离）
* 要求构造对象的函数（ParsePtF）可以构造出不同表现的对象（ASCIIText等）时（多功能）

### 三.参与者：

#### Builder(TextConverter)
* 为创建一个Product对象的各个部件指定抽象接口

#### ConcreteBuilder(ASCIIConverter、TeXConverter、TextWidgetConverter)
* 实现Builder	的接口以构造和装配该产品的各个部件
* 定义并明确它所创建的表示
* 提供一个检索产品的接口（例如GetASCIIText）

#### Director(RTFReader)
* 构造一个使用Builder接口的对象

#### Product(ASCIIText、TeXText、TextWidget)
* 表示被构造的复杂对象，ConcreteBuilder创建该产品的内部表示并定义它的装配过程
* 包含定义组成部件的类，包括将这些部件装配成最终产品的接口

### 四.协作：

* 客户创建Director对象，并用它所想要的Builder对象进行配置
* 一旦产品部件被生成，导向器（Director）就会通知生成器（Builder）
* 生成器处理导向器的请求，并将部件添加到该产品中
* 客户从生成器中检索产品

#### 交互图如下：

![image](https://raw.githubusercontent.com/cheng668/image/master/%E5%BB%BA%E9%80%A0%E8%80%85%E6%A8%A1%E5%BC%8F1.png)

### 五.效果：

#### 使你可改变一个产品的内部表现
* Builder隐藏了产品的装配过程和内部结构和表现，它只提供Director一个构造产品的抽象接口，改变产品的内部表现时只需新定义一个生成器。

#### 将构造代码和表现代码分开
* Builder通过封装一个复杂对象的创建和表现方式来提高对象的模块性，ConcreteBuilder代码可复用

#### 可更精细控制构造过程
* Builder于一下子就生成产品的创建型模式相比，它是在导向者控制下一步步构造产品，能反应构造过程

### 六.实现：

* Builder类接口要足够普遍，以便为各种类型的具体生成器构造产品。
* 产品类型相差大，没必要为产品定义抽象类
* Builder类中缺省方法要为空，不能是纯虚成员，便于子类只定义感兴趣的部分

### 七.应用：

* ET++的RTF转换器；
* 自适应通讯环境中的服务配置者框架使用生成器来构造运行时刻动态连接到服务器的网络服务构件。

### 八.相关模式：

* AstractFactory模式：Builder着重于一步步构造复杂对象，产品在最后一步返回；AstractFactory着重于多个系列产品对象的构造，产品立即返回
* Composite通常用Builder生成

### 九.代码：