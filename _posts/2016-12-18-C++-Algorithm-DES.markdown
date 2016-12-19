---
layout: post
title:  "DES加密算法实现"
categories: C++
tags: Algorithm
---

* content
{:toc}

DES算法为密码体制中的对称密码体制，由美国IBM公司研制的对称密码体制加密算法。 明文按64位进行分组，密钥长64位，密钥事实上是56位参与DES运算（第8、16、24、32、40、48、56、64位是校验位， 使得每个密钥都有奇数个1）分组后的明文组和56位的密钥按位替代或交换的方法形成密文组的加密方法。





### 一、算法实现

#### 密钥的处理

* K1:把64位密钥去除符号位，用KT1表置换为56位密码

```c
const static int
KT1[56] = {
	57, 49, 41, 33, 25, 17, 9, 1,
	58, 50, 42, 34, 26, 18, 10, 2,
	59, 51, 43, 35, 27, 19, 11, 3,
	60, 52, 44, 36, 63, 55, 47, 39,
	31, 23, 15, 7, 62, 54, 46, 38,
	30, 22, 14, 6, 61, 53, 45, 37,
	29, 21, 13, 5, 28, 20, 12, 4 },
```

* K2:将K1生成的56位密码分成左后两份，每份28bits,分别是KL,KR,
* K3:分别将KL,KR循环左移，每次左移量为KEYMOVE表对应数字

```c
KEYMOVE[16] = {
	1, 1, 2, 2, 2, 2, 2, 2,
	1, 2, 2, 2, 2, 2, 2, 1 },
```

* K4:将循环左移后KR和KL，再把KR和KL合并起来进行KT2表的压缩置换，这里注意KT2表中28要选取第55位，而不是（28-1=）27位，最终最终Key,重复K2-K4共16次

```c
KT2[48] = {
	14, 17, 11, 24, 1, 5, 3, 28,
	15, 6, 21, 10, 23, 19, 12, 4,
	26, 8, 16, 7, 27, 20, 13, 2,
	41, 52, 31, 37, 47, 55, 30, 40,
	51, 45, 33, 48, 44, 49, 39, 56,
	34, 53, 46, 42, 50, 36, 29, 32 },
```

#### 密文的生成

* C1:将要加密的8字节明文用CT1表置换位置，并分开前32bits为CL,后32bits为CR

```c
CT1[64] = {
	58, 50, 42, 34, 26, 18, 10, 2,
	60, 52, 44, 36, 28, 20, 12, 4,
	62, 54, 46, 38, 30, 22, 14, 6,
	64, 56, 48, 40, 32, 24, 16, 8,
	57, 49, 41, 33, 25, 17, 9, 1,
	59, 51, 43, 35, 27, 19, 11, 3,
	61, 53, 45, 37, 29, 21, 13, 5,
	63, 55, 47, 39, 31, 23, 15, 7 },
```

* C2:将CR用CRCHG表扩展并置换为48bits，后和K4得到的key进行模2加（异或）生成48bits序列

```c
CRCHG[48] = {
	32, 1, 2, 3, 4, 5, 4, 5,
	6, 7, 8, 9, 8, 9, 10, 11,
	12, 13, 12, 13, 14, 15, 16, 17,
	16, 17, 18, 19, 20, 21, 20, 21,
	22, 23, 24, 25, 24, 25, 26, 27,
	28, 29, 28, 29, 30, 31, 32, 1 },
```

* C3:经CRCHG表扩展置换后的48bits分成8组，用于选择第S盒，每组6bits,第一和最后一bit组成一个数字，用于选择某个S_box中某行，剩下4bits组成一个数字，选择该行某列的数字，
* C4:结果是取得S_box中一个整数，该整数只有低4bits可能置位，也就是该数字最大为15，
* C5:然后组成(8 * 4bits = )32bits初步密文，后面要经过SCHG表进行置换，
* C6:将置换后的结果和CL进行模2加（异或），存到CL，交换CL和CR，重复C2-C6共16次

```c
S_box[8][4][16] = {
	//S1
	{ { 14, 4, 13, 1, 2, 15, 11, 8, 3, 10, 6, 12, 5, 9, 0, 7 },
	{ 0, 15, 7, 4, 14, 2, 13, 1, 10, 6, 12, 11, 9, 5, 3, 8 },
	{ 4, 1, 14, 8, 13, 6, 2, 11, 15, 12, 9, 7, 3, 10, 5, 0 },
	{ 15, 12, 8, 2, 4, 9, 1, 7, 5, 11, 3, 14, 10, 0, 6, 13 } },
	//S2
	{ { 15, 1, 8, 14, 6, 11, 3, 4, 9, 7, 2, 13, 12, 0, 5, 10 },
	{ 3, 13, 4, 7, 15, 2, 8, 14, 12, 0, 1, 10, 6, 9, 11, 5 },
	{ 0, 14, 7, 11, 10, 4, 13, 1, 5, 8, 12, 6, 9, 3, 2, 15 },
	{ 13, 8, 10, 1, 3, 15, 4, 2, 11, 6, 7, 12, 0, 5, 14, 9 } },
	//S3
	{ { 10, 0, 9, 14, 6, 3, 15, 5, 1, 13, 12, 7, 11, 4, 2, 8 },
	{ 13, 7, 0, 9, 3, 4, 6, 10, 2, 8, 5, 14, 12, 11, 15, 1 },
	{ 13, 6, 4, 9, 8, 15, 3, 0, 11, 1, 2, 12, 5, 10, 14, 7 },
	{ 1, 10, 13, 0, 6, 9, 8, 7, 4, 15, 14, 3, 11, 5, 2, 12 } },
	//S4
	{ { 7, 13, 14, 3, 0, 6, 9, 10, 1, 2, 8, 5, 11, 12, 4, 15 },
	{ 13, 8, 11, 5, 6, 15, 0, 3, 4, 7, 2, 12, 1, 10, 14, 9 },
	{ 10, 6, 9, 0, 12, 11, 7, 13, 15, 1, 3, 14, 5, 2, 8, 4 },
	{ 3, 15, 0, 6, 10, 1, 13, 8, 9, 4, 5, 11, 12, 7, 2, 14 } },
	//S5
	{ { 2, 12, 4, 1, 7, 10, 11, 6, 8, 5, 3, 15, 13, 0, 14, 9 },
	{ 14, 11, 2, 12, 4, 7, 13, 1, 5, 0, 15, 10, 3, 9, 8, 6 },
	{ 4, 2, 1, 11, 10, 13, 7, 8, 15, 9, 12, 5, 6, 3, 0, 14 },
	{ 11, 8, 12, 7, 1, 14, 2, 13, 6, 15, 0, 9, 10, 4, 5, 3 } },
	//S6
	{ { 12, 1, 10, 15, 9, 2, 6, 8, 0, 13, 3, 4, 14, 7, 5, 11 },
	{ 10, 15, 4, 2, 7, 12, 9, 5, 6, 1, 13, 14, 0, 11, 3, 8 },
	{ 9, 14, 15, 5, 2, 8, 12, 3, 7, 0, 4, 10, 1, 13, 11, 6 },
	{ 4, 3, 2, 12, 9, 5, 15, 10, 11, 14, 1, 7, 6, 0, 8, 13 } },
	//S7
	{ { 4, 11, 2, 14, 15, 0, 8, 13, 3, 12, 9, 7, 5, 10, 6, 1 },
	{ 13, 0, 11, 7, 4, 9, 1, 10, 14, 3, 5, 12, 2, 15, 8, 6 },
	{ 1, 4, 11, 13, 12, 3, 7, 14, 10, 15, 6, 8, 0, 5, 9, 2 },
	{ 6, 11, 13, 8, 1, 4, 10, 7, 9, 5, 0, 15, 14, 2, 3, 12 } },
	//S8
	{ { 13, 2, 8, 4, 6, 15, 11, 1, 10, 9, 3, 14, 5, 0, 12, 7 },
	{ 1, 15, 13, 8, 10, 3, 7, 4, 12, 5, 6, 11, 0, 14, 9, 2 },
	{ 7, 11, 4, 1, 9, 12, 14, 2, 0, 6, 10, 13, 15, 3, 5, 8 },
	{ 2, 1, 14, 7, 4, 10, 8, 13, 15, 12, 9, 0, 3, 5, 6, 11 } } },
//S_box处理后置换用SCHG表
SCHG[32] = {
	16, 7, 20, 21, 29, 12, 28, 17,
	1, 15, 23, 26, 5, 18, 31, 10,
	2, 8, 24, 14, 32, 27, 3, 9,
	19, 13, 30, 6, 22, 11, 4, 25 },
```

* C7:最后，将处理后的8字节密文（CL+CR）用CT2表进行置换

```c
CT2[64] = {
	40, 8, 48, 16, 56, 24, 64, 32,
	39, 7, 47, 15, 55, 23, 63, 31,
	38, 6, 46, 14, 54, 22, 62, 30,
	37, 5, 45, 13, 53, 21, 61, 29,
	36, 4, 44, 12, 52, 20, 60, 28,
	35, 3, 43, 11, 51, 19, 59, 27,
	34, 2, 42, 10, 50, 18, 58, 26,
	33, 1, 41, 9, 49, 17, 57, 25 };
```

### 二、代码实现

#### 相关函数简介

##### 用户接口

```c
enum Cipher
{
	DECRYPT = 0,
	ENCRYPT = 1
};
//明文入参in(8bits),钥匙入参key(8bits)，密文缓存入参out(8bits),功能选择入参flag
void des_encrypt(const char* in, const char* key, char* out, Cipher flag);
```

##### 功能函数（系统内部调用）

```c
bool DES_KEY[16][48];
/*密钥处理，经处理的密钥存于DES_KEY中*/
void generate_key(const char* in);

/*置换函数，入参：置换用表，   置换前数据缓存， 置换后数据缓存， 置换后数据缓存大小*/
void Tramsfer(const int inTable[], const bool in[], bool out[], size_t outlenght);

/*循环左移，入参：需要左移的数据缓存，缓存大小，左移步数*/
void RotateLeft(bool* in, size_t inlenght, size_t count);

/*比特数组转变为char数组，入参：需要转换的bit数组，bit数组大小，char数组*/
void Bit2Byte(const bool* in, size_t inlenght, char* out);

/*char数组转变为bit数组，入参：需要转换的char数组，bit数组，bit数组大小*/
void Byte2Bit(const char* in, bool* out, size_t outlenght);

/*模2加，异或，结果存于inoutA中*/
void Xor(bool* inoutA, const bool* inB, size_t lenght);

/*S_box运算，入参：左队列，     右队列，                生成的密钥，              运算次数，  功能参数*/
void SFunc(bool* CL/*32bits*/, bool* CR/*32bits*/, const DES_KEY key/*16*48bits*/, int count, Cipher flag);
```

#### 相关函数实现

##### 用户接口

```c
/*加密解密*/
	void des_encrypt(const char* in, const char* key, char* out, Cipher flag)
	{
		generate_key(key);
		bool afterFchg[64];
		bool tmp[64];
		Byte2Bit(in, tmp, 64);
		/*
		C1:将要加密的8字节明文用CT1表置换位置，
		并分开前32bits为CL,后32bits为CR  
		*/
		Tramsfer(CT1, tmp, afterFchg, 64);
		bool CL[32], CR[32];
		for (int i = 0; i < 32; i++)
		{
			CL[i] = afterFchg[i];
			CR[i] = afterFchg[i + 32];
		}
		//进行S盒运算
		SFunc(CL, CR,  16, flag);
		/*C7:最后，将处理后的8字节密文（CL+CR）用CT2表进行置换*/
		memset(afterFchg, 0, 64);
		for (int i = 0; i < 32; i++)
		{
			afterFchg[i] = CL[i];
			afterFchg[i + 32] = CR[i];
		}
		bool afterLchg[64];

		Tramsfer(CT2, afterFchg, afterLchg, 64);
		Bit2Byte(afterLchg, 64, out);
	}
```

##### 功能函数（系统内部调用）

* 置换函数Tramsfer

```c
	void Tramsfer(const int inTable[],const bool* in, bool* out, size_t outlenght)
	{
		memset(out, 0, outlenght);
		for (size_t i = 0; i < outlenght; i++)
		{
			out[i] = in[inTable[i] - 1];
		}
	}
```

* 循环左移RotateLeft

```c
	void RotateLeft(bool* in,size_t inlenght,size_t count)
	{
		bool tmp[30];
		memset(tmp, 0, 30);
		memcpy(tmp,in,count);
		memcpy(in, in + count, inlenght - count);
		memcpy(in+inlenght-count, tmp, count);
	}
```

* 比特数组转变为char数组Bit2Byte

```c
	void Bit2Byte(const bool* in, size_t inlenght, char* out)
	{
		memset(out, 0, (inlenght + 7) / 8);
		for (size_t i = 0; i < inlenght; i++)
			out[i / 8] |= in[i] << (7-(i % 8));
	}
```

* char数组转变为bit数组Byte2Bit

```c
	void Byte2Bit(const char* in, bool* out, size_t outlenght)
	{
		memset(out, 0, outlenght);
		for (size_t i = 0; i < outlenght; i++)
			out[i] = (in[i / 8] >> (7-(i % 8))) & 1;
	}
```

* 模2加，异或

```c
	void Xor(bool* inoutA , const bool* inB , size_t lenght)
	{
		for (size_t i = 0; i < lenght; i++)
		{
			inoutA[i] ^= inB[i];
		}
	}
```

* S_box运算，核心

```c
	void SFunc(bool* CL/*32bits*/, bool* CR/*32bits*/,
		 int count, Cipher flag)
	{
		/*
		C2:将CR用CRCHG表扩展并置换为48bits，
		后和K4得到的key进行模2加（异或）生成48bits序列 
		*/
		bool tmp[48];		
		Tramsfer(CRCHG, CR, tmp, 48);		
		if (flag)
			Xor(tmp, DES_KEY[16 - count], 48);
		else
			Xor(tmp, DES_KEY[count - 1], 48);
		/*
		C3:经CRCHG表扩展置换后的48bits分成8组，用于选择第S盒，
		每组6bits,第一和最后一bit组成一个数字，用于选择某个S盒中某行，
		剩下4bits组成一个数字，选择该行某列的数字
		*/		
		int S = 0,a = 0,b = 0;
		int result;
		bool afterS[32];
		memset(afterS, 0, 32);
		for (int i = 0; i < 8;i++)
		{
			S = 6 * i;
			a = (tmp[S] << 1) + tmp[S + 5];
			b = (tmp[S + 1] << 3) + (tmp[S + 2] << 2) + (tmp[S + 3] << 1) + tmp[S + 4];
			/*C4:结果是取得S盒中一个整数，
			该整数只有低4bits可能置位，
			也就是该数字最大为15*/
			result = S_box[i][a][b];
			//将所有的结果保存在afterS中
			for (int j = 0; j < 4;j++)
				afterS[i * 4 + j] |= (result >> (3 - j)) & 1;
		}
		/*
		C5:然后组成(8 * 4bits = )32bits初步密文，
		后面要经过SCHG表进行置换
		*/
		bool afterTram[32];
		Tramsfer(SCHG, afterS, afterTram, 32);
		/*C6:将置换后的结果和CL进行模2加（异或），
		存到CL，交换CL和CR，重复C2-C6共16次*/
		Xor(CL, afterTram, 32);
		if (count == 1)
		{
			/*进行到第16次时要把CR,CL交换*/
			bool temp[32];
			memset(temp, 0, 32);
			memcpy(temp, CL, 32);
			memcpy(CL, CR, 32);
			memcpy(CR, temp, 32);
			return;
		}
		//交换CR和CL并进行递归，共执行16次S_BOX运算
		SFunc(CR, CL,  count - 1,flag); 
	}
```	

* 生成钥匙

```c
	void generate_key(const char* in)
	{
		/*
		K1:把64位密钥去除符号位，用KT1表置换为56位密码
		*/
		bool tmp[64];
		Byte2Bit(in, tmp, 64);
		bool afterFchg[56];
		Tramsfer(KT1, tmp, afterFchg, 56);
		/*K2:将K1生成的56位密码分成左后两份，每份28bits,分别是KL,KR*/
		bool* KL = afterFchg, *KR = &afterFchg[28];
		/*
		K4:将循环左移后KR和KL，
		再把KR和KL合并起来进行KT2表的压缩置换，
		这里注意KT2表中28要选取第55位，而不是（28-1=）27位，
		最终最终Key,
		重复K2-K4共16次
		*/
		for (int i = 0; i < 16; i++)
		{
			/*K3:分别将KL,KR循环左移，每次左移量为KEYMOVE表对应数字 */
			RotateLeft(KL, 28, KEYMOVE[i]);
			RotateLeft(KR, 28, KEYMOVE[i]);
			memset(DES_KEY[i], 0, 48);
			for (size_t j = 0; j < 48; j++)
			{
				if (KT2[j] == 28)
					DES_KEY[i][j] = afterFchg[55];
				else
					DES_KEY[i][j] = afterFchg[KT2[i] - 1];
			}
		}
	}
```

### 最后

编写时应该注意以下几点：
* 经常用memset初始化数组，原因有两个：
（1）本算法运用的是bool数组模拟bit数组，bool占8bits，而我们只希望用到其中的第一个bit，除了bool为false时是确定每bit为0外，当bool为true时每bit是什么就难以确定了，而且算法中会对bool数组进行模2加，会使增加结果的不确定性；
（2）如果原始key用的是不足8字节的字符串，也要对原始key进行初始化，因为生成key会用到原始key的8字节，你除了能确定自己输入的那几个字节外，剩下的字节是不确定的。

* K4步骤:将循环左移后KR和KL，再把KR和KL合并起来进行KT2表的压缩置换，这里注意KT2表中28要选取第55位，而不是（28-1=）27位；

* 从char数组和bit数组转换时注意转移顺序
如char数组是`10000111 00010001`按照转换成bit数组的话：

```c
	for (size_t i = 0; i < outlenght; i++)
		out[i] = (in[i / 8] >> (7-(i % 8))) & 1;
```

得到bit数组:`10000111 00010001`
而如果按照：

```c
	for (size_t i = 0; i < outlenght; i++)
		out[i] = (in[i / 8] >> (i % 8)) & 1;
```

会得到bit数组为`11100001 10001000`

* 测试代码

```c
int main()
{	
	string key  = "cheng";					//钥匙
	string text = "Welcome to cheng's blog! www.cheng668.com";    //明文
	string ent  = "";							//密文
	string destring = "";						//解密明文

	//修改密钥格式
	char key11[10];
	memset(key11, 0, 10);
	if (key.size() < 8)
		memcpy(key11, key.c_str(), sizeof(key.c_str()));
	else
		memcpy(key11, key.c_str(), 8);

	//将明文分开每8个字节进行加密
	for (size_t i = 0;i<(text.length()+7)/8;i++)
	{
	    char entext[9],intext[9];
		memset(entext, 0, 9);
		memset(intext, 0, 9);
		_memccpy(intext, text.c_str() + 8 * i, NULL, 8);
		des_encrypt(intext, key11, entext, ENCRYPT);
		ent += entext;
	}
	cout << ent << endl;

	//将密文转变为每8个字节进行解密
	for (size_t i = 0; i < (ent.length() + 7) / 8; i++)
	{
		char outtext[9],detext[9];
		memset(outtext, 0, 9);
		memset(detext, 0, 9);
		_memccpy(detext, ent.c_str() + 8 * i, NULL, 8);
		des_encrypt(detext, key11, outtext, DECRYPT);
		destring += outtext;
	}
	cout << destring << endl;
	system("pause");
	return 0;
}
```

### github代码：

**[DES算法代码 🇨🇳](https://github.com/cheng668/Algorithm-DES)**