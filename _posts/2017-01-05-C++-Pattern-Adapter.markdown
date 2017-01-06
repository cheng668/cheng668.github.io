---
layout: post
title:  "C++ Pattern-Adapter"
categories: C++
tags: Pattern
---

* content
{:toc}

* 适配器模式分为类适配器模式和对象适配器模式，类适配器用的是 private继承方法，而对象适配器用的是持有对象指针的方法。
* 根据《Efective C++》--Item 39 Use private inheritance judiciously-- private继承和持有对象指针有某些异同点，此文顺便会对这些异同点和各自优缺点作一个总结。
* 对 private继承会出现齐位需求这一概念，对这个概念，可参考**[另一博文](http://www.cnblogs.com/graphics/archive/2010/08/12/1797953.html)**
* 类适配器模式可能用到多重继承，至于多重继承的概念和要注意的地方，可以参考《C++ Primer(第五版)》--18.3节 多重继承与虚继承--，本文不会把重点放在这。







类适配器模式的UML图如下：

![](https://raw.githubusercontent.com/cheng668/image/master/%E7%B1%BB%E9%80%82%E9%85%8D%E5%99%A8%E6%A8%A1%E5%BC%8F.png)

对象适配器模式的UML图如下：

![](https://raw.githubusercontent.com/cheng668/image/master/%E5%AF%B9%E8%B1%A1%E9%80%82%E9%85%8D%E5%99%A8%E6%A8%A1%E5%BC%8F.png)

### 一.意图：

* 将一个类的接口转换成客户希望的另一个接口。使接口兼容。

### 二.适用性：

* 想使用一个已存在的类，但接口不符合需求。
* 想创建一个可复用的类，可和接口不兼容的类协同工作。
* (仅使用于对象 Adapter)想使用一些已存在类的子类，适配器可适配它们的父接口(如代码中的 VirRequest()接口)。

### 三.参与者：

#### Target
* 定义 Client使用的相关接口。

#### Client
* 与符合 Target接口的对象协同工作。

#### Adaptee ( CAdaptee, OAdaptee, DerivedOAdaptee)
* 定义一个已存在的接口，接口需适配。

#### Adapter
* 对 Adaptee接口与 Target接口适配。

### 四.协作：
* Client向 Adapter实例上发送请求，Adapter调用 Adaptee的操作实现请求。

### 五.效果：

与之前的博文不同，这里在讨论适配器模式能打到的效果前，博主在这里先结合本设计模式总结下《Effective C++》Item 39 私继承和持有对象指针的关系。

#### 私继承和对象指针

私继承，也就是 Adapter对 CAdaptee的继承方式：

```c++
class Adapter :
	public Target, private CAdaptee
	{
	};
```

持有对象指针，也就是 OAdaptee*作为 Adapter的成员变量：

```c++
private:
	// 对象适配器
	OAdaptee* _oa;
```

这两种方式都能看作 Adapter调用了 CAdaptee或者 OAdaptee，但也是有某些区别的：

1. 私继承让 Adapter能重写或实现私基类( OAdaptee)的虚成员函数，而对象指针( OAdaptee* _oa)的虚成员函数不能被重写或实现。
2. 私继承让 Adapter能使用私基类( OAdaptee)的 protected保护级别的成员，而对象指针( _oa)只能被使用 public级别的成员。
3. 私继承或者说全部类型的继承`CAdaptee`都会使父类和子类耦合，即要在子类的头文件`Adapter.h`中加入`#include "CAdaptee.h"` ；而持有一个对象指针`OAdaptee*`只需加入`class OAdaptee;`，实现解耦。至于为什么要实现解耦，可以参考《Effective C++》Item 31: Minimize compilation dependencies between files 。
4. 如果一个类没有非静态成员变量，没有虚函数，没有虚基类。那么私继承这种类不会占用空间( EBO, empty base optimization);而对象指针需要占用4或者8字节大小的空间。

引用书中栗子：
对于对象指针：

```c++
class Empty{};   //空类

class HoldsAnInt{
private:
	int x;
	Empty* e;    //这里是指针
};
```

这时因为多了一个对象指针，使`sizeof(HoldsAnInt)>sizeof(int)`。
实际上书中的栗子时这样的：

```c++
class Empty{};   //空类

class HoldsAnInt{
private:
	int x;
	Empty e;    //这里不是指针
};
```

这时 Empty对象会根据齐位需求占用0-1个char大小的空间。

而私继承的话：

```c++
class Empty{};   //空类

class HoldsAnInt
: Empty
{
private:
	int x;
};
```

因为有编译器的空白基类最优化(EBO)，让`sizeof(HoldsAnInt)=sizeof(int)`。

脑补了这些知识后，在来看两类适配器模式达到的效果：

#### 类适配器

* 用 Adapter类对 Adaptee和 Target进行匹配(继承)，但要匹配 Adaptee的子类时，类模式不能胜任。
* Adapter类可重定义 Adaptee的部分行为，如上1。

下面是栗子，在 CAdaptee类中有虚函数：

```c++
virtual void VirRequest() = 0;
```

Adapter对 CAdaptee私继承并实现 VirRequest接口。

```c++
class Adapter :
	public Target, private CAdaptee
{
...
private:
	// 类适配器
	virtual void VirRequest();
...
};
```

* 仅引入一个对象，不需额外指针得到 Adaptee，这里可归结为上4。
 
#### 对象适配器

* 允许一个 Adapter类对 Adaptee及其子类进行适配，也就是可以动态装载 Adaptee及其子类到 Adapter类，这点好处在《Effective C++》没提：

```c++
...
// Adaptee子类
DerivedOAdaptee* drvO = new DerivedOAdaptee;
// 动态装载
Adapter* adapter = new Adapter(drvO);
...
```

而在 Adapter应该有：

```c++
...
public:
	Adapter(OAdaptee* oa = nullptr);
private:
	OAdaptee* _oa;
...
```

* 难以重定义 Adaptee的行为，与类适配器相对。

#### 还需考虑

* Adapter类的匹配程度，博主代码里的匹配工作量是最低的了，都是直接调用 Adaptee的接口，但实际应用上并没有这么简单：

```c++
void Adapter::Request()
{
	// 类适配器，直接运用私继承下来的接口
	CSpecificRequest();
	VirRequest();
	
	// 对象适配器，可调用已存在接口的子类
	if (_oa) 
		_oa->OSpecificRequest();
}
```

* 使 Adapter类接口尽量小已实现高可复用性和可插入适配器。

* 双向适配器让两个系统用不兼容的接口进行协作。

### 六.实现：

* 类适配器模式，公共继承 Target类，私继承 Adaptee类。
* 实现可插入适配器(博主的理解是，一个 Adapter适配多个接口互不兼容的 Adaptee类族对象)： 1）继承方式，用窄接口更优； 2）持有对象指针方式(代理对象)，客户可控制适配； 3）参数化适配器

### 七.应用：

* c++标准库中的 stack, queue和 priority_queue容器适配器。

### 八.相关模式：

* Bridge模式：结构和 Adapter模式类似，但出发点不同，前者目的是将实现和接口分离，后者是改变已有对象的接口。
* Decorator模式：增强其他对象的功能而不改变接口，透明性比较好，而且支持递归组合。
* Proxy模式：不改变接口前提下，为另一对象定义一个代理。

### 九.代码：

**[类/对象适配器模式完整代码 🇨🇳](https://github.com/cheng668/Pattern-Adapter.git)**