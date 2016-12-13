---
layout: post
title:  "C++ Pattern-AstractFactory!"
date:   2016-12-13 11:14:00
categories: C++
tags: Pattern
excerpt: C++���ģʽ֮���󹤳�ģʽ
---

![](https://raw.githubusercontent.com/cheng668/image/master/%E6%8A%BD%E8%B1%A1%E5%B7%A5%E5%8E%82%E6%A8%A1%E5%BC%8F.png)

## һ.��ͼ��

* �ṩһ������һϵ����ػ��໥��������Ľӿڣ�������ָ�����Ǿ�����ࡣ

## ��.�����ԣ�

* һ��ϵͳҪ���������Ĳ�Ʒ�Ĵ�������Ϻͱ�ʾʱ
* һ��ϵͳҪ�ɶ����Ʒϵ���е�һ��������ʱ
* ǿ��һϵ����ز�Ʒ���������Ա��������ʹ��ʱ
* �ṩһ����Ʒ��⣬��ֻ����ʾ���ǵĽӿڶ�����ʵ��ʱ

## ��.�����ߣ�

### AbstractFactory(WidgetFactory)
* ����һ�����������Ʒ����Ĳ����ӿڡ�
### ConcreteFactory(MotifWidgetFactory,PMWidgetFactory)
* ʵ�ִ��������Ʒ����Ĳ�����
### AbstractProduct(Windows,ScrollBar)
* Ϊһ���Ʒ��������һ���ӿڡ�
### ConcreteProduct(MotifWindow,MotifScrollBar)
* ����һ��������Ӧ�ľ��幤�������Ĳ�Ʒ����
* ����AbstractProduct�ӿ�
### Client
* ��ʹ����AbstractFactory��AbstractProduct�������Ľӿ�

##��.Э����

* ����ʱ����һ��ConcreteFactory���ʵ�������幤����Ӧ��ͬ��Ʒ����
* AbstractFactory����Ʒ����Ĵ����ӳٵ�ConcreteFactory���ࡣ

## ��.Ч����

### �ŵ㣺
* ����ͻ����Ʒ����
* һ��Ӧ���н���һ�δ������幤����ĵط������ڽ�����Ʒϵ��
* ���ڲ�Ʒһ����
### ȱ�㣺
* ����֧��������Ĳ�Ʒ

## ��.ʵ�֣�

* ������Ϊ��������Singletonģʽ����
* .AbstractFactory����������Ʒ�Ľӿڣ�ConcreteProduct����ʵ�֡�

## ��.Ӧ�ã�

* InterViewʹ�á�Kit����׺����ʾAbstractFactory�࣬WidgetKit,DialogKit��
* ET++ʹ��AbstracFactoryģʽ�Դ��ڲ�ͬ����ϵͳ��X Windows��SunView����Ŀ���ֲ�ԡ�

## ��.���ģʽ��

* AbstractFactory��ͨ���ù���������Factory Method��ʵ�֣�������Ҳ������Prototypeʵ�֡�
* һ�����幤��ͨ��ʱһ��������Singleton����

## ��.Դ�����ӣ�

**[���󹤳�ģʽԴ������](https://github.com/cheng668/Pattern-AstractFactory)**