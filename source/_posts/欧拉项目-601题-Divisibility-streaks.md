---
title: 欧拉项目 | 601题 | Divisibility streaks
date: 2018-07-28 15:33:58
tags: [欧拉项目, Project Euler, 整除, 数论]
categories: 数学
---
[601题链接](https://projecteuler.net/problem=601 "Problem 601 - Project Euler")

对于每个整数n我们可以定义一个函数streak(n)=k，其中k是最小的n+k不能被k+1整除的数。  
比如  
13 可以被 1 整除  
14 可以被 2 整除  
15 可以被 3 整除  
16 可以被 4 整除  
17 不可以被 5 整除  
所以streak(13)=4  
同样的  
120 可以被 1 整除  
121 不可以被 2 整除  
所以streak(120)=1  

P(s,N)是满足下面两个条件的n的和  
1. 1 < n < N  
2. streak(n)=s  

题目给了两个值P(3,14)=1 和 P(6,106)=14286帮助验证自己的想法。

求$$\sum_{i=1}^{31} P(i, 4^i)$$  

首先看下N的数量级，4^31=2^62，都接近long的数量级了，遍历小于N的每个数显然是不合适的。

整除k+1，其实就是被2，3，4等整除。手算了i=1，2，3，4的情况，发现有规律的：  
比如i=4，那么到了2 3 4 5的最小公倍数往后的情况和之前的情况是一样的，循环起来了。  
那么如果我能计算最小公倍数以内的情况，那么就能很容易得到全部的情况。  
不过事与愿违，从2到32这31个数的最小公倍数是一个长达15的数字，虽然比2^62小好几个数量级，但是还是不能遍历。  
这条路走不通。  

各种想了2个小时之后，找到了一种直接计算的方式。  

什么样的n满足streak(n)=s呢？  
n+1 % 2 == 0，否则streak(n)=1  
同理，n+2 % 3 == 0  
类推到 n+s-1 % s == 0  
最后 n+s % s+1 != 0  
对于前面s-1个等式被整除的数都减去k，k=2,3...s，得到一个统一的式子  
n-1 % k == 0  
也就是说，n-1是2,3...s最小公倍数的若干倍。利用这一点，我们可以容易的找到可能的候选n，n的数量不会很多，对于i等于31的情况，粗略估计也只有四位数个吧。  
最后利用n+s % s+1 !=0得到n，然后求和。

下面是P函数的代码
``` csharp
static long P(int s, long N)
{
    long lcm = Enumerable.Range(1, s)
        .Select(Convert.ToInt64)
        .Aggregate((cur, next) => Utils.GetLcm(cur, next));

    return Enumerable.Range(1, (int)((N - 2) / lcm))
        .Select(i => i * lcm + 1)
        .Count(c => (c + s) % (s + 1) != 0);
}
```
Enumerable.Range(1, s)，多了个1，对于最小公倍数没有影响。  
N-2的原因是
> 1. 1 < n < N  

调用P函数得到和
``` csharp
Enumerable.Range(1, 31).Select(i => P(i, 1L << 2 * i)).Sum();
```
