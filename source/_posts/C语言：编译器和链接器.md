---
title: C语言：编译器和链接器
date: 2015-07-07 15:36:09
tags: [C, 内存, 编译器, 链接器]
categories:
 - 计算机
 - 计算机基础
---
预处理器把C语言源代码处理完之后，编译器会处理成对象文件（比如.o文件），链接器会找到相关文件，链接组成一个二进制文件。下面看一个例子：

``` cpp
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>

int main()
{
    void *mem = malloc(400);
    assert(mem != NULL);
    printf("HAHA\n");
    free(mem);

    return 0;
}
```
生成的main.o文件大致如下：  
CALL malloc  
CALL printf  
CALL free  
RET  
为啥没有assert呢？因为assert是宏，预处理器处理完就没有了。

链接器把main.o和对应库文件链接生成main.out这个可执行文件。  
运行这个文件，会打印HAHA。

如果我们把include stdio注释掉：  
预处理器处理完成的文件是没有printf的声明的，编译时发现语法都是正确的，gcc会推测printf是一个输入string，返回int的函数，随之产生一个warning，链接器根据编译器给出的结果载入对应的库文件，生成的可执行文件和正常的是一样的。

如果我们把include stdlib注释掉：  
编译器发现malloc free没有声明，推测其函数原型，产生warning，后续和上面情况类似，一切都正常。

如果我们把include assert注释掉：  
预处理不会做对应的宏展开，编译器发现assert，推测原型，.o文件中多了一行CALL assert，链接器在库里面找不到assert函数，报错。

对于上面三种情况的一和二，自己写一个假的原型来欺骗编译器也可以，就去掉了warning。  
其实，声明函数，定义一系列的参数列表，只要是为了能够在Stack上正确的记录上下文活动，而给编译器看是附带效果。  
看下面的示例，程序是诡异的，但是能够正常输出的。

``` cpp
int strlen(char*, int);

int main()
{
    int n = 65;
    int length = strlen((char*)&num, num);
    printf("%d\n", length);

    return 0;
}
```
给了strlen的原型，但是和库里面的不一致，使用gcc编译之后，程序能正常运行，输出多少呢？  
编译器看到strlen需要两个参数，且返回int。链接器去找strlen定义，很开心，找到了，链接，生成文件。这里需要说明下，.o文件没有参数类型，链接器只看名字。  
运行时，栈里面是什么情况呢？  
栈最底下是n，然后分配length需要的空间，再push num和num地址进去，两个参数，SP=SP-8，保存PC上下文，调用strlen，strlen需要一个参数char\*，去SP的地址去拿，幸好，SP指向地方还真是个指针，指向num。strlen把num开始的地址作为char\*去处理，遇到第一个0结束。在小端机器上，num内存是65 0 0 0，遇到第一个0结束，所以它认为这个字符串长度为1。  
所以，这个诡异的程序输出了1！

