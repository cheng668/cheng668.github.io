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

这里实现了两种单例模式：

* 一种是多例模式(特殊单例模式)，其中包括用于计数的模板 Counted<>类和用于生成实例的 Printer类；

Counted类：

```c++
template<class BeingCounted>
class Counted{
public:
	class TooManyObject{};  //用于抛出异常
	static int objectCount(){ return numObjects; }
protected:
	Counted();
	Counted(const Counted& rhs);
	~Counted(){ --numObjects; }
private:
	static int numObjects;
	static const size_t maxObjects;
	void init();
};
```

Printer类,其中 makePrinter函数是《More Effective C++》中提到的伪构造函数，用于构造类对象；

```c++
class Printer :
	private Counted<Printer>
{
public:
	static Printer* makePrinter(const Printer& rhs);

	static Printer* makePrinter();
	~Printer();
	void reset();
	using Counted<Printer>::objectCount;
	using Counted<Printer>::TooManyObject;
	
	//单例模式中取实例函数
	/*
	Printer& thePrinter()
	{
		static Printer p;
		return p;
	}
	*/

private:
	Printer();
	Printer(const Printer& rhs);
};
```

```c++
Printer* Printer::makePrinter(const Printer& rhs)
{
	return new Printer(rhs);
}
```

* 另一种是可继承的单例模式，包括用于选择创建子类实例的 SinglePainter父类和用于扩展功能的 ColorSinglePainter子类

而此种可继承的单例模式又分为用注册表方法和父类构造子类方法来管理子类实例，下面会解析。

SinglePainter父类：

```c++
class SinglePainter
{
public:
	//获取单例
	static SinglePainter* Instance();
	virtual ~SinglePainter();
	virtual void print(){}
	//注册，用于注册方法
	static void Register(const char* name, SinglePainter*);
protected:
	SinglePainter();
	//用于注册方法，查看注册表
	static SinglePainter* Lookup(const char* name);
private:
	static SinglePainter* _instance;
	//注册表，用于注册方法
	static map<string, SinglePainter*>* _register;
};
```

ColorSinglePainter子类：

```c++
class ColorSinglePainter :
	public SinglePainter
{
public:
	virtual ~ColorSinglePainter();
	virtual void print() override;

	/*如果不用注册表方法保存子类单例，
	要加这个以便在父类Instance函数
	中调用子类构造函数*/
	//friend SinglePainter;
protected:
	ColorSinglePainter();
private:
	//注册表方法，必须实现饿汉模式，p用于保存子类实例
	static ColorSinglePainter* p;
};

```

* 把构造函数放到保护级别以上( protected, private)保证一个唯一的实例。
例如：因 SinglePainter需要被继承，所以应该把构造函数放在 Protected级别上：

```c++
protected:
	SinglePainter();
```

Printer类不需用于继承,于是构造函数和赋值构造函数应该放在 private级别上：

```c++
private:
	Printer();
	Printer(const Printer& rhs);
```

* 保存 Singleton对象指针的静态数据变量 _instance可以指向 Singleton子类对象。
也就是 _instance静态数据变量类型必须是父类指针：

```c++
private:
	static SinglePainter* _instance;
```

* 不能将单件定义为一个全局或静态的对象，然后依赖于自动初始化，原因： a)不能保证静态对象只有一个实例会被声明； b)没有足够信息初始化每一个单件； c)全局对象构造顺序不知道，单件不能相互依赖； d)无论是否用到都要创建，浪费资源。
* 创建 Singleton类的子类： 
a)在父类 Instance函数中根据条件判断 new哪个子类，此方法需要父类知道所有子类，且为了让父类能调用子类的构造函数，应该在子类中加入`friend SinglePainter`；

```c++
SinglePainter* SinglePainter::Instance()
{
	//懒汉单例
	if (_instance == 0)
	{
		//这里可以运行时确定,或者读取环境变量
		const char* style = "ColorSinglePainter";

		//非注册表方法
		if (strcmp(style, "ColorSinglePainter") == 0)
			_instance = new ColorSinglePainter;
		//else if(strcmp(style, "ColorSinglePainter2") == 0)
		//	_instance = new ColorSinglePainter2;
	}

	return _instance;
}
```

b)在父类中维护一个注册表，子类在创建时进行注册，父类只需在 Instance函数中读取注册表即可，此方法子类需要用饿汉模式。

在`ColorSinglePainter.cpp`中应该有：

```c++
ColorSinglePainter::ColorSinglePainter()
{
	//注册
	SinglePainter::Register("ColorSinglePainter", this);
}

// 饿汉单例
ColorSinglePainter* ColorSinglePainter::p = new ColorSinglePainter;
```

在`SinglePainter.cpp`中应该有：

```c++
SinglePainter* SinglePainter::Instance()
{
	//懒汉单例
	if (_instance == 0)
	{
		//这里可以运行时确定
		const char* style = "ColorSinglePainter";

		//注册表方法
		_instance = Lookup(style);
	}
	return _instance;
}
//查询注册表
SinglePainter* SinglePainter::Lookup(const char* name)
{
	return _register->at(name);
}
//用于子类注册，_register是map<>静态变量
void SinglePainter::Register(const char* name, SinglePainter* sp)
{
	if (!_register)
		_register = new map < string, SinglePainter* > ;
	_register->insert(make_pair(name,sp));
}
```

### 七.应用：

* 数据库读写句柄，打印机读写句柄。

### 八.相关模式：

* Abstract Factory模式、Builder模式、Prototype模式可以用 Singleton模式实现。

### 九.代码：

**[单/多例模式完整代码 🇨🇳](https://github.com/cheng668/ME-26-Singleton.git)**