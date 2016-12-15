---
layout: post
title:  "Openssl开发环境配置"
categories: C++
tags: Openssl
---

* content
{:toc}

OpenSSL 是一个安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及SSL协议，并提供丰富的应用程序供测试或其它目的使用。
它采用c语言开发，有很好的跨平台性能。要运用openssl进行开发，先要配置好环境变量，此种配置方法适用于绝大部分开源开发库。




#### 下载openssl
第一步肯定是要先下载openssl库：
**[openssl官方下载地址](http://www.openssl.com/download.html)**
**[openssl github 源码](https://github.com/cheng668/openssl)**

#### 系统环境配置
下载完成后编译（或者安装）得到OpenSSL-Win32文件夹，于是打开“环境变量”，按“添加”按钮，根据自己的安装路径添加一条变量，博主安装在D:\目录下，所以添加变量值为D:\OpenSSL-Win32，变量名为OPENSSL_DIR或者根据自己需要填写：
![]()

#### 附加包含目录
接着打开你要用到openssl库的工程，博主的是自己的编写的SQLiteStudio工具[代码](https://github.com/cheng668/QT-SQLiteStudio)的工程，右击选择“属性”：
![]()

在“配置属性”-“C/C++”-“常规”的“附加包含目录”中添加`$(OPENSSL_DIR)\include`，其中，`OPENSSL_DIR`就是之前配置环境变量的变量名，`$(OPENSSL_DIR)\include`会用变量值翻译过来变为`D:\OpenSSL-Win32\include`:
![]()

#### 附加库目录
同理，在“配置属性”-“链接器”-“常规”的“附加库目录”中添加`$(OPENSSL_DIR)\lib`：
![]()

#### 附加依赖项
在“配置属性”-“链接器”-“输入”的“附加依赖项”中添加“libssl.lib”、“libcrypto.lib”两项，其中此两项按照版本的不同而不同，博主用的是1_1_0c：
![]()

#### 最后
最后，重启IDE，如果需要在QT上使用DES算法，请看博主文章；或想进一步指导OPENSSL算法的使用，可以[点我](https://www.openssl.org/docs/man1.1.0/)
