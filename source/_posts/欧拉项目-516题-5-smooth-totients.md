---
title: 欧拉项目 | 516题 | 5-smooth totients
date: 2018-07-27 11:18:39
tags: [欧拉项目, Project Euler, 数论, 质数]
categories: 数学
---
[516题链接](https://projecteuler.net/problem=516 "Problem 516 - Project Euler")

5-smooth数是指质因数不超过5的数。换句话说，质因数只能是2，3，5。  
S(L)是指不超过的L的n求和，其中n满足[欧拉函数，Euler's totient function](https://en.wikipedia.org/wiki/Euler%27s_totient_function)φ(n)是一个5-smooth数。  
S(100)=3728. 题目给出这个提示是进行一个简单的验证，检查自己的算法/程序是否有问题。  
求S(10^12) % 2^32

10^12次方很大，电脑是GHz级别，10^9，所以遍历小于L的每一个数是不现实的。我们要想办法构造出所有满足条件的n，然后求和。  
φ(n)有很多种写法，下面这种对于我们分析这个题目很有帮助。  
$$  
\phi(n)=\prod_{i=1}^k p_i^{r_k-1}(p_i-1)
$$  
$$  
n=\prod_{i=1}^k p_i^{r_k}
$$  

如果质因数只有2，3，5，减一之后是1，2，4，那么这个n是满足条件的。  

质因数能大于5吗？  
可以。比如7，7-1之后是6，质因数是2和3，如果这种质因数的次数r只能是1，r-1之后是0，那么也还是满足要求的。如果多余1次，φ(n)就会包含比5大的因数。再比如11，41等都是满足条件的。  
这种质数是可以任意相乘的，组合之后仍然满足题意。他们和质因数只有235的数相乘，依旧满足题意。顺便说一下，只有这些质数作为质因数的数也是满足题意的。  

如何构造这种质数呢？  
找到所有质因数是235的数，加一，再判断是不是质数。  

让我们先写个程序找到所有小于10^12且质因数都是235的数。
``` csharp
for (int i = 0; i < Math.Log(max) / Math.Log(2); i++)
{
    for (int j = 0; j < Math.Log(max) / Math.Log(3); j++)
    {
        for (int k = 0; k < Math.Log(max) / Math.Log(5); k++)
        {
            if (Math.Pow(2, i) * Math.Pow(3, j) * Math.Pow(5, k) <= max)
            {
                smooths.Add((long)(Math.Pow(2, i) * Math.Pow(3, j) * Math.Pow(5, k)));
            }
        }
    }
}

smooths.Sort();
```
有个微不足道的优化，j和k的上限可以限制的更小一些，但是没必要了，数不大。排序的目的后面会提到。

接下来，生成所有大于5且满足上述分析的质数。
``` csharp
var candidates =
    smooths
    .Select(l => l + 1)
    .Where(Utils.IsPrime)
    .Where(p => p > 5)
    .ToArray();
```
那么如果对这些质数进行组合呢？普通的组合算法是行不通的。因为`candidates`有500多个数据，从中任选5个就达到了10^12这个量级，所以需要合适的裁剪。  
我的做法是分层，第一层都是1个数组成的组合，第二层都是2个数组成的组合，依此类推。同时，后一层是从前一层的组合生成的。对于i层的某一个组合，i+1层就是从`smooths`里面选一个数，然后加上去。`smooths`排序的好处有两个，一是找到i层组合的最后一个数，只需要往上添加比它大的数即可，排序可以利用二分查找；二是如果第j个加上去之后，乘积大于10^12，后面的数就不用看了。
``` csharp
List<List<List<long>>> tiers = new List<List<List<long>>>();
var firstTier = candidates.Select(p => new List<long>() { p }).ToList();
tiers.Add(firstTier);

while (tiers.Last().Count != 0)
{
    var lastTier = tiers.Last();
    var currentTier = new List<List<long>>();
    tiers.Add(currentTier);

    foreach (var combination in lastTier)
    {
        double product = combination.Aggregate((total, next) => total * next);

        int index = Array.BinarySearch(candidates, combination.Last());
        for (int i = index + 1; i < candidates.Length; i++)
        {
            if (product * candidates[i] <= max)
            {
                currentTier.Add(new List<long>(combination)
                {
                    candidates[i]
                });
            }
            else
            {
                break;
            }
        }
    }
}
```
所有的组合都找到了，很容易得到乘积。
``` csharp
var factors = new List<long>();
foreach (var tier in tiers)
{
    foreach (var combination in tier)
    {
        factors.Add(combination.Aggregate((total, next) => total * next));
    }
}
```
我们已经把这种情况得到了。
> 这种质数是可以任意相乘的，组合之后仍然满足题意。

接着处理下一种情况
> 他们和质因数只有235的数相乘，依旧满足题意。
``` csharp
factors.Sort();

foreach (var factor in factors)
{
    foreach (var smooth in smooths)
    {
        if ((double)smooth * factor <= max)
        {
            sum += smooth * factor;
        }
        else
        {
            break;
        }
    }
}

sum += smooths.Sum();
```
排序的好处就是可以提前终止。上面代码的最后一句是把基本case包含上了。
> 如果质因数只有2，3，5，减一之后是1，2，4，那么这个n是满足条件的。

smooths里面包含1（ijk都是0的情况），所以循环也包含了前面提到的这种情况。
> 顺便说一下，只有这些质数作为质因数的数也是满足题意的。

最后，返回结果。
``` csharp
return (uint)sum;
```
sum以开始定义为`long`，`uint`截断后就是模2^32的结果。
