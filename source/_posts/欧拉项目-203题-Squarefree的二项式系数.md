---
title: 欧拉项目 | 203题 | Squarefree的二项式系数
date: 2019-06-23 17:29:38
tags: [欧拉项目, Project Euler, 二项式系数]
categories: 数学
---
[Problem 203](https://projecteuler.net/problem=203)

二项式系数可以写成如下帕斯卡三角的形式  
![](/images/ProjectEuler-203.png)

前八排有12个不同的数：1, 2, 3, 4, 5, 6, 7, 10, 15, 20, 21 和 35。  
Squarefree的意思是不能被任何质数的平方整除。前八排的数字里面，4和20不是Squarefree的，因为它们能被质数2的平方整除。  
求帕斯卡三角前51排不重复的Squarefree的数之和。

我们先来估计下最大的数是多少，然后考虑用什么类型表示，第五十一行其实是n=50，那么最大的是数$$C_n^k=C_{50}^{25}=\frac{50!}{25!25!}=126,410,606,437,752$$，long就足够了。  
首先我们拿到前51排的数。
``` csharp
var rows = new List<long[]>();
for (int i = 0; i < 51; i++)
{
    long[] row = new long[i + 1];
    for (int j = 0; j < row.Length; j++)
    {
        if (j == 0 || j == row.Length - 1)
        {
            row[j] = 1;
        }
        else
        {
            var lastRow = rows[i - 1];
            row[j] = lastRow[j - 1] + lastRow[j];
        }
    }

    rows.Add(row);
}
```
下一步去重。
``` csharp
var candidates = rows.SelectMany(l => l).Distinct().ToList();
```
要考虑那些质数的平方呢？仔细想下前51排数的性质，只需要考虑很少几个数就足够了。
``` csharp
var smallPrimeSquare = new int[] { 4, 9, 25, 49 };
```
如果某个数能被下一个质数的平方121整除，那么n至少要是22，这样子二项式系数的分子上才会出现2个11，但是不管k怎么取，分母上都至少有一个11这个因数。那么n至少要是33了，不管k怎么取，分子上都至少有两个11这个因数。那么n至少要是44了，同理，也无法被121整除。总结下，分母上11的因数的个数至少是分子上11的因数的个数减一。其实，结论很容易直接证明的。  
那么为什么需要考虑49呢？很简单，49包含了两个7，那么n取49和50的时候，有可能分子因数7的个数能够比分母多两个。那么该数就能被49整除，进而不是Squarefree的。  
最后，排除Squarefree的数字再求和即可。
``` csharp
candidates = candidates.Where(l =>
{
    foreach (var square in smallPrimeSquare)
    {
        if (l % square == 0)
        {
            return false;
        }
    }

    return true;
}).ToList();

return candidates.Sum();
```
