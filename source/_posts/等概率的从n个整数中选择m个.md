---
title: 等概率的从n个整数中选择m个
date: 2014-09-29 15:45:17
tags: [概率, 面试题]
categories: 
 - 计算机
 - 算法
---
给一个数组，其中包含n个整数，要求等概率的从中选取m个整数。

这道题挺简单的，在《编程珠玑》这本书中有讲解，想我两年前找工作的时候，认真的看了这本书的大多数内容，受益颇多。  
这本书的作者提到他是从《计算机程序设计艺术》看到的一个解决方法。《计算机程序设计艺术》这本书也是好书，曾经看了第一卷的多数内容，实在太高端，后面想继续看，但是一直没有再去图书馆借，日后有机会，还是多读读。

废话不多说。解题的关键点是，从s个数中，选择r个，那么以概率r/s来选择。整个过程要遍历n个整数，每次都动态的以r/s的概率来要选择当前的这一个。下面是代码：  
``` csharp
public static int[] PickM(int[] a, int m)
{
    List<int> result = new List<int>();
    int n = a.Length;
    Random rand = new Random(DateTime.Now.Millisecond);

    for (int i = 0; i < n; i++)
    {
        if (rand.Next(0, n - i) < m)
        {
            result.Add(a[i]);
            m--;
        }
    }

    return result.ToArray();
}
```
