---
layout: post
title:  "C++ Resource Management!"
date:   2016-06-08 15:14:54
categories: C++
tags: C++
excerpt: C++资源管理轻谈
---

* 先来看一下背景：在C++98的语言机制中，对象在超出作用域的时候其析构函数会被自动调用。接着，Bjarne Stroustrup 在TC++PL里面定义了RAII（Resource Acquisition is Initialization ） 范式（即：对象构造的时候其所需的资源便应该在构造函数中初始化，而对象析构的时候则释放这些资源）。RAII意味着我们应该用类来封装和管理资源，对于内存管理而言，Boost第一个实现了工业强度的智能指针，如今智能指针（`shared_ptr`和`unique_ptr`）已经是C++11的一部分，简单来说有了智能指针意味着你的C++代码基中几乎就不应该出现`delete`了。
* 对于C++98的内存管理，我们可以建立一个资源管理类，举个例子：
```
class A
{
public:
    A(){}
};

void rsrlek()
{
    A* a = new A();
    //do something    /*<-------------这里可能出现异常导致程序退出，即指针a还没delete就退出了，于是就会出现内存泄露现象*/
    delete a;
}
``` 

* 现在运用资源管理类进行内存管理：
```
class A
{
public:
    A(){}
};
class B
{
public:
    B() :a(new A()){}
    ~B(){ delete a; }
private:
    A* a;
};
void nonrsrlek()
{
    B b;
    //do something
    //delete a;   /*<----------这里不需要显式删除指针*/
}   /*<------------------函数退出，对象b自动析构，并删除指针a，避免了内存泄露*/
```   

* 在C++11中引入了智能指针（shared_ptr,unique_ptr等），现在可以这样写代码：
```
#include <memory>
using namespace std;
class A
{
public:
    A(){}
};
void nonrsrlek()
{
    shared_ptr<A> a;
    //do something
} /*<----------函数退出，a自动释放内存*/
```  

* 智能指针`shared_ptr< A >`相当于资源管理类B，但就如《More effective C++》一书中所说，智能指针并不是普通指针；在《C++ PRIMER(第五版)》中也提过，不能将普通指针和智能指针不能混用，否则会出现悬挂指针现象。
现在引入一种新处理方法：
```
class A
{
public:
    A(){}
};

void nonrsrlek()
{
    A* a = new A();
    ON_SCOPE_EXIT(
        [&a](){ delete a; }  //lambda可调用对象
    );
    //do something
} /*<----------函数退出，a通过ON_SCOPE_EXIT释放内存*/
```   

* 而ON_SCOPE_EXIT实际上类似于一个资源管理类，其实现如下：
```
#define SCOPEGUARD_LINENAME_CAT(name, line) name##line
#define SCOPEGUARD_LINENAME(name, line) SCOPEGUARD_LINENAME_CAT(name, line)

#define ON_SCOPE_EXIT(callback) ScopeGuard SCOPEGUARD_LINENAME(EXIT, __LINE__)(callback
``` 

* `ON_SCOPE_EXIT`是`ScopeGuard xxx（callback）`的宏定义，而为了为`ScopeGuard ` 对象起不重复的名字，这里用了`SCOPEGUARD_LINENAME` 这个宏实现把行号混入变量名xxx,实际上`ScopeGuard ` 这个类才是类资源管理的类，其实现如下：
```
class ScopeGuard
{
public:
    explicit ScopeGuard(std::function<void()> onExitScope)
        : onExitScope_(onExitScope), dismissed_(false)
    { }

    ~ScopeGuard()
    {
        if (!dismissed_)
        {
            onExitScope_();
        }

    void Dismiss()
    {
        dismissed_ = true;
    }

private:
    std::function<void()> onExitScope_;
    bool dismissed_;

private: // noncopyable /*《Effencient C++》中有接收，防止复制*/
    ScopeGuard(ScopeGuard const&);
    ScopeGuard& operator=(ScopeGuard const&);
};
```  


* 这个类的使用很简单，你交给它一个`std::function`，它负责在析构的时候执行， 绝大多数时候这个`function`就是`lambda`。
当然，它处理实现资源管理外，用法还很灵活，例如：
```
HANDLE h = CreateFile(...);
ScopeGuard onExit([&] { CloseHandle(h); });
```

* 在代码块内打开一个文件，然后直接调用`ScopeGuard ` 或者调用`ON_SCOPE_EXIT`宏，实现在代码块退出时自动调用`CloseHandle（）`完成所需要的工作（并不一定是释放指针）