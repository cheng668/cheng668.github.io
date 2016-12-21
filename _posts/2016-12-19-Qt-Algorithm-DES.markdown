---
layout: post
title:  "QT中运用Openssl DES加密算法"
categories: Qt
tags: Openssl
---

* content
{:toc}

QT并没有自带的加解密工具，网上有用Qt开源加解密插件，有自己实现加解密算法功能的。
因博主之前的文章已经用C++编写过一个DES加密算法，这里就不再用QT重复一遍了。
本文介绍了在QT中运用OPENSSL开源加解密库的DES算法，和着重提醒几个容易出错的地方，其他算法的运用与此类似。





### 一、配置环境
* 运用VS2014平台配置Openssl环境已经在之前讲过，如有不懂，可以
**[点我](http://cheng668.com/2016/12/14/OPENSSL-Develop-Config/)** 

### 二、选择加解密模式
* 电子密码本模式ECB
* 加密块链模式CBC
* 加密反馈模式CFB
* 输出反馈模式OFB

根据加密模式的不同，安全性和容错性会有差异，详细介绍请看
**[WIKI](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Electronic_Codebook_.28ECB.29)** 

本文只介绍ECB的用法。

### 先贴出加密代码

```c++
CRYPTO_STATUS ECBCipher::DES_Encrypt(const QString cleartext, const QString key, QString& ciphertext)
{
	if (cleartext.isEmpty())
	{
		return ENCRYPT_SUCCESS;
	}
	DES_cblock keyEncrypt;
	memset(keyEncrypt, 0, 8);
	const char* ch = key.toLatin1().data();
	if (key.length() <= 8)
		memcpy(keyEncrypt, key.toLatin1().data(), key.length());
	else
		memcpy(keyEncrypt, key.toLatin1().data(), 8);

	DES_key_schedule keySchedule;
	DES_set_key_unchecked(&keyEncrypt, &keySchedule);

	const_DES_cblock inputText;
	DES_cblock outputText;
	string Ciphertext;

	for (int i = 0; i < cleartext.length() / 8; i++) {
		memcpy(inputText, cleartext.toLatin1().data() + i * 8, 8);
		DES_ecb_encrypt(&inputText, &outputText, &keySchedule, DES_ENCRYPT);
		for (int j = 0; j < 8; j++)
			Ciphertext.push_back(outputText[j]);
	}

	if (cleartext.length() % 8 != 0) {
		int tmp1 = cleartext.length() / 8 * 8;
		int tmp2 = cleartext.length() - tmp1;
		memset(inputText, 0, 8);
		memcpy(inputText, cleartext.toLatin1().data() + tmp1, tmp2);

		DES_ecb_encrypt(&inputText, &outputText, &keySchedule, DES_ENCRYPT);

		for (int j = 0; j < 8; j++)
			Ciphertext.push_back(outputText[j]);
	}

	ciphertext.clear();
	ciphertext = QString::fromLatin1(Ciphertext.c_str(), Ciphertext.length());

	return ENCRYPT_SUCCESS;
}
```

### 接口介绍

#### DES_cblock别名

```c
typedef unsigned char DES_cblock[8];
```

`DES_cblock`用于存放原始密码或者原始数据，是`unsigned char[8]`的别名。从这里可看出DES算法只用到了用户密钥的前8个字节的数据作为原始密钥和原始数据；另一个问题是Qt常用的是QString类型存放字符串，所以这里需要进行转换，转换前记得先把`DES_cblock`进行`memset`初始化，防止当用户输入的密钥key不足8位时出现未定义行为，下面会详细说明。

#### const_DES_cblock别名

```c
typedef /* const */ unsigned char const_DES_cblock[8];
/*
 * With "const", gcc 2.8.1 on Solaris thinks that DES_cblock * and
 * const_DES_cblock * are incompatible pointer types.
 */
```

看注释可知，现在版本`const_DES_cblock`与`DES_cblock`相同

#### DES_key_schedule别名

```c
typedef struct DES_ks {
    union {
        DES_cblock cblock;
        /*
         * make sure things are correct size on machines with 8 byte longs
         */
        DES_LONG deslong[2];
    } ks[16];
} DES_key_schedule;
```

`DES_key_schedule`用于存放处理后的密钥

#### 初始化钥匙函数

```c
void DES_set_key_unchecked(const_DES_cblock *key, DES_key_schedule *schedule);
```

这里只要根据类型别名对号入座就行，如用`keySchedule`存放处理后的密钥：

```c
DES_key_schedule keySchedule;
DES_set_key_unchecked(&keyEncrypt, &keySchedule);
```

#### 加解密函数

```c
void DES_ecb_encrypt(const_DES_cblock *input, DES_cblock *output,
                     DES_key_schedule *ks, int enc);
```

其中，`input`是明文，`output`是密文缓存，`ks`是处理后的密钥，`enc`为1时调用加密、0为解密

```c
# define DES_ENCRYPT     1
# define DES_DECRYPT     0
```

### 四、数据处理

#### 密钥的处理

因为Qt常用QString存放字符串数据，无论是明文还是密钥，但在算法接口中，只提供了`unsigned char*`的类型接口，而且只能填入最多8字节数据和密钥，所以这里就需要数据类型的转换，并要注意对缓存的初始化，即运用`memset`，对密钥`QString key`的初始化如下:

```c
DES_cblock keyEncrypt;
memset(keyEncrypt, 0, 8);
const char* ch = key.toLatin1().data();
if (key.length() <= 8)
	memcpy(keyEncrypt, key.toLatin1().data(), key.length());
else
	memcpy(keyEncrypt, key.toLatin1().data(), 8);
```

#### 明文处理

因为待加密的数据很少会刚好是8倍数字节的，所以这里就需要对最后一段不足8字节的数据进行处理

```c++
const_DES_cblock inputText;           //存放8字节明文
DES_cblock outputText;				  //存放8字节密文
string Ciphertext;					  //存放全部密文

//对完整8字节的明文进行每8个字节加密
for (int i = 0; i < cleartext.length() / 8; i++) {
	memcpy(inputText, cleartext.toLatin1().data() + i * 8, 8);
	DES_ecb_encrypt(&inputText, &outputText, &keySchedule, DES_ENCRYPT);

	for (int j = 0; j < 8; j++)
		Ciphertext.push_back(outputText[j]);
}

//对余下不足8字节的明文进行扩展并加密
if (cleartext.length() % 8 != 0) {
	int tmp1 = cleartext.length() / 8 * 8;
	int tmp2 = cleartext.length() - tmp1;
	memset(inputText, 0, 8);
	memcpy(inputText, cleartext.toLatin1().data() + tmp1, tmp2);

	DES_ecb_encrypt(&inputText, &outputText, &keySchedule, DES_ENCRYPT);

	for (int j = 0; j < 8; j++)
		Ciphertext.push_back(outputText[j]);
}
```

#### 对密文类型转换

将密文保存在`Ciphertext`内后，要注意从string转换为QString，string解码后存放的字符不一定是原来的Ascii编码字符，用`QString::fromStdString()`函数转换时先判断每个字节是否大于127，如果不大于，就按Ascii逐字节存进QString的字节中，如果大于127，则按照UTF8转换为Unicode编码，但问题会出在UTF8转换为Unicode编码过程中，`QString::fromStdString()`内部会检查编码规则，则当一个字节为196（`1100 0100`）时，代码会认为这个UTF8编码的是双字节字符，但实际传进去的是整个string的长度，显然不符合UTF8编码规则，所以会返回`QChar::ReplacementCharacter`即`0xfffd`，简单来说：

假设`outputText`中第一个字符的2进制数为`10000001`,即129，对于这个大于127的字符，`QString::fromStdString()`函数会因为UTF8编码转Unicode编码失败而翻译成65533，这样就出错了。

所以这里应该用`QString::fromLatin1()`函数进行转换，

```c
 static inline QString fromLatin1(const char *str, int size = -1)
```

函数会先创建size个位置（如果size不填，则取char数组长度），然后把char数组逐个字节转变为uchar存储在QString的相应位置，即会把129（无符号）或者-127（有符号）都作为无符号的129。

对于编码规则如有不懂，请**[点我进入WIKI](https://en.wikipedia.org/wiki/UTF-8)** 或者**[百度百科](http://baike.baidu.com/link?url=cq7K5TZZ_s0bp9YohoJ14e-QVzTL06fa-AUHSLxpx4x2EpoYJUIWk9JgYJEhbAkcQMyZBVyuh9Qz9y4LsQKB_K)**

```c
ciphertext.clear();
ciphertext = QString::fromLatin1(Ciphertext.c_str(), Ciphertext.length());
```

这里还要注意string的转化过程中，QString会识别`\0`空字符，当解密后字符串中，如在得到的`Ciphertext`8字节字符串中，`Ciphertext[2] = '\0'`，那么转换为QString就会只剩下`Ciphertext[0]`和`Ciphertext[1]`了，显然是不符合的。
所以`QString::fromLatin1`中第二个参数必须指定为`Ciphertext`的长度。

### 五、完整加密解密代码

**[ECB加密代码](https://github.com/cheng668/QT-SQLiteStudio/blob/master/ecbcipher.cpp)** 

解密过程和加密类似，这里不再累赘，但要注意以下一点：
* 当调用`QString::fromLatin1()`从string密文向明文QString转换时，第二个参数不要填，原因是解密后得到的明文以每8字节保存在`Ciphertext`(假设是`Ciphertext`,也可以为`Cleartext`,自定义)中，不足8字节的会用`\0`补齐，如果这时用

```c
	ciphertext = QString::fromLatin1(Ciphertext.c_str(), Ciphertext.length());
```

进行转换的话，QString会把`\0`带上，造成不必要的错误。
