---
title: C-Style『泛型』（下）：通用的Stack
date: 2015-06-09 14:44:06
tags: C
categories:
 - 计算机
 - 计算机基础
---
内容简介：定义Stack头文件，实现Stack；引出为什么会内存泄漏；修复这个问题。

首先，我们定义头文件。

``` cpp
struct Stack {
    void *elements;
    int eleSize;
    int logicalLength;
    int allocLength;
};

void StackNew(Stack *s, int eleSize);

void StackDispose(Stack *s);

void StackPush(Stack *s, void *eleAddr);

void StackPop(Stack *s, void *eleAddr);
```
我们需要用void\*来保存内容；由于void\*不包含其他信息，所以需要eleSize来指定要进出栈的对象的大小；内部两个量，用于记录有多少个元素在栈里面，分配了多少空间来存放这些对象。  
接下来定义的几个函数很容易理解，新初始化一个栈，销毁处理栈，进栈、出栈操作，eleAddr有着不同的含义，进栈操作指的是待进栈对象的地址，出栈操作指的是会把待出栈对象拷贝到的地址。
<!-- more -->
下面我们来实现New这个操作，初始化各个变量，为elements分配默认大小的空间。

``` cpp
void StackNew(Stack *s, int eleSize)
{
    assert(s->eleSize > 0);
    s->eleSize = eleSize;
    s->logicalLength = 0;
    s->allocLength = 4;
    s->elements = malloc(s->allocLength * eleSize);
    assert(s->elements);
}
```
Dispose这个函数很简单，释放elements占用的内存。

``` cpp
void StackDispose(Stack *s)
{
    free(s->elements);
}
```
Push函数稍微复杂一点，把要进栈的对象拷贝到栈空间里面，同时，要考虑到分配的空间可能已经不够大了，要动态增加。

``` cpp
void StackPush(Stack *s, void *eleAddr)
{
    if (s->logicalLength == s->allocLength)
    {
        StackGrow(s);
    }

    void *target = (char*)s->elements + s->eleSize * s->logicalLength;
    memcpy(target, eleAddr, s->eleSize);
    s->logicalLength++;
}
```
其中，Grow函数如下：

``` cpp
static void StackGrow(Stack *s)
{
    s->allocLength *= 2;
    s->elements = realloc(s->elements, s->allocLength * s->eleSize);
    assert(s->elements);
}
```
这里，使用了realloc函数重新分配内存，它的基本实现是如果当前内存后面有足够的空间，直接扩大内存，修改这块内存前的四字节或者八字节的标记块，返回与传入指针相同的地址；如果不够，开辟另一块内存，拷贝内容过去，释放前一块内存，返回新内存地址。

最后是Pop函数，要先判断栈里面是不是有东西，如果有，把最上面的对象拷贝到给定的地址去。

``` cpp
void StackPop(Stack *s, void *eleAddr)
{
    assert(s->logicalLength > 0);
    s->logicalLength--;
    void *source = (char*)s->elements + s->eleSize * s->logicalLength;
    memcpy(eleAddr, source, s->eleSize);
}
```
目前为止，这个Stack实现像模像样，我们写个简单的函数来测试一下：

``` cpp
    Stack s;
    StackNew(&amp;s, 4);
    
    int x = 12;
    int y = 1;
    int z = 15;

    StackPush(&amp;s, &amp;x);
    StackPush(&amp;s, &amp;z);

    int output;

    StackPop(&amp;s, &amp;output);
    std::cout << output << std::endl;

    StackPush(&amp;s, &amp;y);

    StackPop(&amp;s, &amp;output);
    std::cout << output << std::endl;
    StackPop(&amp;s, &amp;output);
    std::cout << output << std::endl;

    StackDispose(&amp;s);
```
输出就是15 1 12，符合预期。

但是这有一个严重的问题，比如传入的是char\*，指针对应的是在Stack外malloc的内存，绝大多数情况，当时指向的指针早就过了生命周期或者指向了其他地方，那么，栈里面这个指针副本是唯一指向当初被分配的那块内存的指针。如果外部没有把所有指针对象都Pop出来并且释放（外部调用程序往往也没有这个义务做这些事情），那么，内存泄露就此产生了。所以，我们Stack实现要负责这个事情。

我们给Stack增加一个字段，同时修改以下New接口，让外界传入释放内存的函数，然后保存起来。

``` cpp
void (*freeFn)(void*);

void StackNew(Stack *s, int eleSize, void (*freeFn)(void*));
```
重新实现Dispose函数，如果Stack里面还有对象，且freeFn不是NULL的话，释放内存，以防内存泄漏。

``` cpp
void StackDispose(Stack *s)
{
    if (s->freeFn != NULL)
    {
        for (int i = 0; i < s->logicalLength; ++i)
        {
            s->freeFn((char*)s->elements + i * s->eleSize);
        }
    }

    free(s->elements);
}
```
