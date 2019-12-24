---
title: C++基本类型在内存中的表示
date: 2015-05-13 15:50:41
tags: [内存, C++]
categories:
 - 计算机
 - 计算机原理
---
C/C++不外乎以下几种基本类型：bool，char，short，int，long，float，double，还有不常用的long long。

char占用一个字节，而short占用两个字节，在char转化为short时，全部复制到short占用内存的低八位里面。
``` cpp
#include <iostream>

int main()
{
	char ch = 'A';
	short s = ch;
	std::cout << s << std::endl;
	return 0;
}
```
A对应的ASCII码是65，所以输出是65。

反之，short转化为char时，把高位的一个字节直接扔掉，只复制低字节。
``` cpp
short s = 67;
char ch = s;
std::cout << ch << std::endl;
```
输出是一个大写的C。

int一般占用4个字节，和short转化和上面char和short转化类似。下面看一个不太一样的例子。
``` cpp
int i = 1 << 15;
short s = i;
std::cout << s << std::endl;
```
i在内存中的表示是0000 0000，0000 0000，1000 0000，0000 0000  
short占用两个字节，直接把低位两个字节复制过来，s在内存中的表示是1000 0000，0000 0000  
第一位是符号位，表示负数，根据补码表示的意义，输出应该是负2的15次方，即-32768

float也占用四个字节，但是每一位的含义不一样，第一位是符号位s，接着的八位是exp，接着23位是尾数，表示0.ddd  
float的值等于(-1)^s * (1.ddd) * 2 ^ (exp - 127)  
正常的int转float很简单，比如：
``` cpp
int i = 5;
float f = i;
std::cout << f << std::endl;
```
就是正常的输出5，只是i和f占用的内存对应的bit位上0/1差别很大

来看个不正常的转化
``` cpp
int i = (1 << 22) + (1 << 30);
float f = * (float*) &i;
std::cout << f << std::endl;
```
i在内存中的表示是0100 0000，0100 0000，0000 0000，0000 0000   
f没有分配新的空间，而是使用了i的内存，以float的方式去读而已  
符号位是0，表示正数  
exp是1000 0000，也就是128  
再后面是一个1，22个0，尾数就是1.1  
所以f等于(-1)^0 * (1.1) * 2 ^ (128 - 127) = 1.5 * 2 = 3，即输出3  
注意，上面等式第一步把二进制变成了十进制