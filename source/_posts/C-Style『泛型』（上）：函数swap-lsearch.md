---
title: C-Style『泛型』（上）：函数swap & lsearch
date: 2015-06-04 14:33:22
tags: C
categories:
 - 计算机
 - 计算机基础
---
在C语言中，没有提供任何泛型能力，但是，有神奇void\*，只要运用恰当，能写出通用的『泛型』函数。在这里，『泛型』打了引号，表示并非真的是C#这种支持泛型语言中的含义，而是表示一种通用的意思。

关于这个主题，打算写上下两篇。上篇有关函数本身，写两个函数swap和lsearch来说明如何利用void\*来写出通用的程序。下篇写一个通用的Stack来说明在C语言里面也能写出通用的数据结构。

首先来看第一个例子，swap函数不难写，很多人在刚开始学习C语言时候就写过。
比如想要交换两个int，可以写一个函数，函数声明大致如下：
``` cpp
void swap(int*, int*)
```

现在你又想交换两个double，交换两个自定义的结构体，怎么办呢？再写两个类似的函数，显然是不合适的。  
<!-- more -->
我们可以写一个通用的函数来做这个事情。  
我们传入两个要交换对象的地址。对于int版本的swap函数，编译器知道拿4个字节做交换，double版本的swap函数是8个字节，但是void\*不包含类似的信息，所以我们要传入一个int来表示要交换对象的大小。  
下面是通用版的swap函数
``` cpp
void swap(void *p1, void *p2, int size)
{
    char buffer[size];
    memcpy(buffer, p1, size);
    memcpy(p1, p2, size);
    memcpy(p2, buffer, size);
}
```
我们先开辟一个大小和对象大小一致的空间用来缓存数据。  
然后就是经典的三步：把对象1放到buffer里面，对象2放到对象1里面，最后把buffer里面的放到对象2里面。  
这里需要注意一下，函数第一句需要编译器的支持，幸好，现代编译器基本都支持了。  
可以写几行代码简单的测试一下：
``` cpp
int x = 17;
int y = 27;
swap(&x, &y, sizeof(int));
std::cout << x << "\t" << y <<std::endl;
```

下面来写个线性搜索函数。  
搜索，要有关键字，所以我们要传入待比较对象的地址，还要传入一个数组和它的大小，和前面说的一样，void\*几乎不包含什么信息，所以要传入待搜索对象的大小，最后，要能比较两个对象，需要传入一个比较函数。对于线性搜索来说，可以不传，因为用memcmp比较就能知道两个对象是否一样，但是对于二分搜索就不行了。虽然只是一个简单的示例，还是要充分的考虑通用性的。
``` cpp
void* lsearch(void *key, void *base, int n, int eleSize, int(*cmp)(void*, void*))
{
    for (int i = 0; i < n; ++i)
    {
        void *eleAddr = (char*)base + i * eleSize;
        if (cmp(key, eleAddr) == 0)
        {
            return eleAddr;
        }
    }

    return NULL;
}
```
首先遍历数据，每一个对象都要和key进行比较。void\*是不能做指针的加减运算的，我们转成char\*，然后根据当前遍历的i和对象大小，向后移动若干个字节，获得待比较对象的地址。  
然后就是比较啦，调用传入的比较函数比较，如果一致，就返回该地址。最后，遍历完成还没有找到则返回NULL。

这次写一个稍微复杂的代码来测试：找相同的字符串。  
我们需要一个比较函数：
``` cpp
int strCmp(void *p1, void *p2)
{
    char *s1 = *(char**)p1;
    char *s2 = *(char**)p2;

    return strcmp(s1, s2);
}
```
需要注意一下，字符串本来就是char\*，对于搜索函数而言，要比较的就是char\*，lsearch需要的对象的地址，那么传入的就是char\*的地址，也就是说，key的类型是char\*\*，base数组里面是char\*。所以我们的比较函数的p1 p2是char\*\*，先强转再解引用。  
如果写成char \*s1 = (char\*)p1;，那么s1的含义就和预期的不一致，把p1当做char\*，解析地址，跳转到对应的地方，鬼知道那里是什么，很可能压根你就没权访问，假定能访问，从那里开始一直找到\0结束，没人知道s1是个什么鬼字符串。  
简单地说，这里需要理解指针，要理解跳一次和跳两次的区别。

``` cpp
char *notes[] = {"Ab", "F#", "B", "Gb", "D"};
char *favoriteNote = "Eb";
char **found = (char**)lsearch(&favoriteNote, notes, 5, sizeof(char*), strCmp);
std::cout << (found == NULL) << std::endl;
favoriteNote = "Gb";
found = (char**)lsearch(&favoriteNote, notes, 5, sizeof(char*), strCmp);
std::cout << *found << std::endl;
```
从测试代码就更容易理解char\*\*了，倒数第二参数是char\*的大小，说明要比较的对象类型是char\*。  
如前所述，第一个参数是对象的地址，favoriteNote是char\*，我们要比较它（char\*），所以再取地址传进去。

『泛型』函数就写到这里，回头有时间继续写『泛型』数据结构 - Stack。