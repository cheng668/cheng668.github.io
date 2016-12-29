---
layout: post
title:  "对智能指针重载Operator->*"
categories: STL
tags: SmartPoint
---

* content
{:toc}

本文是博主阅读北京《程序员》杂志2001/09文章《为智能指针实现operator->*》一文后的笔记和后续实现。






### 一.原文概述：

#### 初步考虑

原文先从`operator->*`的基本特性的分析入手。最初实现是将此二元运算符`operator->*`右侧传入函数指针，返回另外一个类型用于专门处理函数调用，初步代码为：

```c++
 // template，函数调用的返回类型是 ReturnType
 // 未完成的成员函数调用。
template<typename ReturnType>
class PMFC { 
public:
	ReturnType operator()() const;	
};

// 智能指针类
template<typename T>
class SP {
public:
	template<typename ReturnType>
		const PMFC<ReturnType>
		operator->*( ReturnType(T::*pmf)() ) const;
};
```

SP相当于一个智能指针模板，这里实现了`operator->*`，并把右侧传入的函数指针的返回值`ReturnType`作为函数调用类模板`PMFC`的模板参数，而函数处理类`PMFC`只不过是重载了`operator()`调用操作符，所以`PMFC`类是一个可调用类。至于为什么`PMFC`要用到模板，原因是成员函数`*pmf`的返回值不确定，这里用模板，正体现泛型编程的思想。

#### 无参成员函数入手

接着为进一步实现这两个模板，应该涉及到的考虑有：
* 如果成员函数有入参,`PMFC`如何处理
* 在函数调用类模板`PMFC`中如何得知成员函数`*pmf`所在类对象的类型和成员函数指针的签名
* `PMFC`类应该用什么数据类型保存`SP`调用`operator->*`的返回值，毫无疑问，无论保存什么数据类型，都应该从作为`operator->*`返回值的`PMFC`类的构造函数中赋值

这里作者选择了先把成员函数的入参放一边。作者综合考虑第二第三点，返回给`PMFC`的值应该包括函数指针`pmf`和成员函数`*pmf`所在类对象的指针，于是这里可以用`std::pair`类型保存数据。而且对象类型(`T`,从`SP`中获取)和函数指针的签名(`ReturnType(T::*)()`)可以通过模板类型传入到`PMFC`中。于是就有了代码：

```c++
template<typename ObjectType,     // 提供成员函数的那个类类型。
		 typename ReturnType, 	  // 成员函数的返回值类型。
		 typename MemFuncPtrType> // 函数指针的签名。
class PMFC {
public:
	typedef std::pair<ObjectType*, MemFuncPtrType> CallInfo;

	PMFC(const CallInfo& info) : callInfo(info) {}

	ReturnType operator()() const
	{
		return (callInfo.first->*callInfo.second)();
	}
private:
	CallInfo callInfo;
};

template <typename T>
class SP {
public:
	SP(T *p) : ptr(p) {}

	template <typename ReturnType>
		const PMFC<T, ReturnType, ReturnType(T::*)()>
		operator->*(ReturnType(T::*pmf)()) const
		{
			return std::make_pair(ptr, pmf);
		}
		
	~SP(){ if(ptr) delete ptr; } // 博主补上
	
private:
	T* ptr;
};
```

到这里，这两个模板已经是可以用的了，但是只能处理没有形参的函数指针。
调用方式如下：

```c++
#include <utility>
using namespace std;
class Wombat {
public:
	int dig()
	{
		cout << "Digging..." << endl;
		return 1;
	}
	int sleep()
	{
		cout << "Sleeping..." << endl;
		return 5;
	}
};
int main()
{ 
	typedef int (Wombat::*PWMF)(); // 指向一个Wombat成员函数
	SP<Wombat> pw = new Wombat;
	PWMF pmf = &Wombat::dig;      // 令 pmf 指向 Wombat::dig
	(pw->*pmf)();                 // 调用我们的 operator->*;
	                              // 输出 "Digging..."
	pmf = &Wombat::sleep;         // 令 pmf 指向 Wombat::sleep
	(pw->*pmf)();                 // 调用我们的 operator->*;
}                                 // 输出 "Sleeping...
```

只有作者还考虑到const成员函数的调用，也该加`const`的地方加上或者重载，这里就不再叙述了，有兴趣的自己研究一下`const`在类函数中的运用。

#### 有入参的成员函数

到这一步就简单得多了，作者是直接扩充`PMFC`类模板的模板参数以实现增加入参的.
即在`SP`类模板中重载`operator->*`：

```c++
	// 支持 1 个入参
	template <typename ReturnType, typename Param1Type>
	const PMFC<T, ReturnType, ReturnType(T::*)(Param1Type)>
		operator->*(ReturnType(T::*pmf)(Param1Type)) const
	{
		return std::make_pair(ptr, pmf);
	}
```

在`PWMF`中重载`operator()`:

```c++
	// 支持 1 个入参
	template<typename Param1Type>
	ReturnType operator()(Param1Type p1) const
	{
		return (callInfo.first->*callInfo.second)(p1);
	}
```

到这里基本功能就完成了，但作者还运用了
**[偏特化技术](https://msdn.microsoft.com/en-US/library/3967w96f(v=vs.80).aspx)**进行优化。

#### 偏特化优化模板

因为成员函数所在对象类型`ObjectType`和函数返回值类型`ReturnType`都可以从函数指针签名`MemFuncPtrType`中获取，所以这里可以用偏特化技术，从而减少`PMFC`的模板参数，而且增加灵活性，于是有：

```c++
template<typename T>
struct MemFuncTraits{};

template<typename R ,typename O>
struct MemFuncTraits<R (O::*)()>
{
	typedef R ReturnType;
	typedef O ObjectType;
};

template<typename R, typename O>
struct MemFuncTraits < R(O::*)() const >
{
	typedef R ReturnType;
	typedef O ObjectType;
};

//支持一个入参
template<typename R, typename O,typename P1>
struct MemFuncTraits < R(O::*)(P1) >
{
	typedef R ReturnType;
	typedef O ObjectType;
};

//支持一个入参
template<typename R, typename O, typename P1>
struct MemFuncTraits < R(O::*)(P1) const >
{
	typedef R ReturnType;
	typedef O ObjectType;
};
```

在`PWMF`类模板中加入

```c++
	typedef typename MemFuncTraits<MemFuncPtrType>::ObjectType ObjectType;
	typedef typename MemFuncTraits<MemFuncPtrType>::ReturnType ReturnType;
```

获取`ObjectType`和`ReturnType`，这样使代码更清晰易读，而且使用户代码和智能指针代码解耦。

### 二、剩下的问题和解决

作者留下了暂未解决的两个问题：
* 无法使用数据成员指针（与函数指针相对）
* 无法使用用户自定义的成员指针。

博主实现上面两个问题的代码如下：
* 增加偏特化模板`MemFuncTraits`:

```c++
//new
template<typename R, typename O>
struct MemFuncTraits < R O::* >
{
	typedef R ReturnType;
	typedef O ObjectType;
};
```

在`PWMF`中加入重载类型转换运算符：

```c++
	//new
	operator ReturnType() const
	{
		return (_callInfo.first->*_callInfo.second);
	}
```

测试代码：

```c++
class Wombat{
public:
	/*data member*/
	int _data = 999;
	int* _pdata = &_data;

};

/*pointer to data member*/
typedef int Wombat::*PD;
typedef int* (Wombat::*PPD);

void main()
{

	PD pd = &Wombat::_data;
	cout << (pw->*pd) << endl;//999

	PPD ppd = &Wombat::_pdata;
	cout << *(pw->*ppd) << endl;//999

	system("pause");
	_CrtCheckMemory();
	return;
}
```

### 三、总结

本来博主想一并实现智能指针类模板，但百度一下，一堆一堆的，而且在《More Effectiive C++》的结尾也有完整的`auto_ptr`的实现代码，所以博主觉得也没必要再在这里累赘了。还因为博主连阅读加写本博文有点小累，并没有考虑到上述问题的解决方法的健壮性，所以希望读者多多包涵并最好提出意见，大家交流学习，谢谢！

### 四、github完整代码链接

**[点我带你飞](https://github.com/cheng668/ME-SmartPtrBase-Operator--)**