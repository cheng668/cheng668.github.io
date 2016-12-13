---
layout: post
title:  "C++ fstream as a returned value"
date:   2016-05-08 11:14:54
categories: C++
tags: C++
excerpt: fstream引用作为函数返回值问题
---

* content
{:toc}

* 以`ofstream`的引用和一个销售数据类的引用作为入参，函数中返回输出数据后的流引用，以下是我代码：

```
ofstream& print(ofstream& o,const Sale_data& rhs)
{
    return o<<rhs.isbn()<<":"<<rhs.units_sold<<" "<<rhs.revenue<<" "<<rhs.avg_price<<endl;
}
```

* 代码编译时会报错：

```
error C2440: “return”: 无法从“std::basic_ostream<_Elem,_Traits>”转换为“std::ofstream &”
```

* 众所周知，`fstream`继承自`istream`和`ofstream`。
* 在我查阅了很多资料和查看`fstream.h`,`ostream.h`头文件后发现， 在`fstream`中并没有重写父类的流操作符，即`ostream`中的`<<`没被`fstream`重写，于是当`return`后面跟上有`<<`操作符的语句， 默认返回父类类型`basic_ostream`，而不是`ofstream` 类型，而且基类`ostream`不能赋值给派生类`fstream`，所以会报错。
要解决这个问题，可以把带`<<`操作符的语句放在`return`外，即：

```
ofstream& print(ofstream& o,const Sale_data& rhs){
    o<<rhs.isbn()<<":"<<rhs.units_sold<<" "<<rhs.revenue<<" "<<rhs.avg_price()<<endl;
    return o;
}
```    

* 或者把`ostream&`作为函数返回值：

```
ostream& print(ofstream& o,const Sale_data& rhs){
    return o<<rhs.isbn()<<":"<<rhs.units_sold<<" "<<rhs.revenue<<" "<<rhs.avg_price()<<endl;     
}
```    

* `fstream`除了没重载父类流操作符`<<`和`>>`外，还有很多函数之类的没重载，这里望大家自己从学习过程中整理出来，更望整理后以评论告诉小编，相互学习。 