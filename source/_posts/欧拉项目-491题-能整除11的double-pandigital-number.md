---
title: 欧拉项目 | 491题 | 能整除11的double pandigital number
date: 2017-12-08 21:03:29
tags: [欧拉项目, Project Euler, 整除, 排列组合]
categories:
  - 数学
  - 欧拉项目
---
[491题链接](https://projecteuler.net/problem=491 "Problem 491 - Project Euler")

一个数，使用两次0-9这十个数字，被称为double pandigital number，那么有多少个这能被11整除的double pandigital number呢？

首先我们要明白，遍历所有的double pandigital number，然后再看看是否能整除11这种直接的暴力算法肯定是不行的，因为double pandigital number数太多了，大约是20!/(2^10)*0.9。  
那怎么办呢？先考虑能被11整除的数的性质，奇位数的数字之和减去偶位数的数字之和，能被11整除。  
所以，我们不需要精确的知道数字是多少，只要通过排列组合知识，把满足题意的数字的个数计算出来就好。

从20个里面选择10个数字作为奇位数上的数字，C(20, 10) = 184756，这个量级做循环操作还是能搞定的。  
选出了奇位数的数字，很容易就能得到偶位数的数字。  
有了这些数字我们就能通过排列组合的知识来计算个数了。  
如果没有数字重复，结果是10!，每出现一个数字重复，就除以2（重复数字交换，还是同一个数字）。  
这里忽略了两个问题。  
一个是0开头的没有被排除，不管怎么折腾，总会有0在第一位，所以把总结果乘以0.9就好了。  
第二个问题是，选择出来的奇位数可能重复，因为我们传入的参数有两个0，两个1，等等，对于组合算法来说，选择第一个0和选择第二个0是两种不同的组合，但是对于我们组成数字来说，没有区别，所以我们还需要一个集合，记住那些被计算过了的被选择出来的奇位数。  
``` csharp
public static long GetAnswer()
{
    long ret = 0;
    HashSet<string> combinations = new HashSet<string>();
    var allDigits = new List<int> { 0, 0, 1, 1, 2, 2, 3, 3, 4, 4, 5, 5, 6, 6, 7, 7, 8, 8, 9, 9 };
    var odds = Utils.Combination(allDigits, 10);

    foreach (var odd in odds)
    {
        int sum = odd.Sum();
        if (sum == 23 || sum == 34 || sum == 45 || sum == 56 || sum == 67)
        {
            string com = string.Join("_", odd);
            if (!combinations.Contains(com))
            {
                combinations.Add(com);
                var even = GetRemainNumbers(odd);
                ret += GetPCount(even) * GetPCount(odd);
            }
        }
    }

    return ret * 9 / 10;
}

private static long GetPCount(IEnumerable<int> source)
{
    long ret = Factorial10;
    int n = 10 - source.Distinct().Count();
    for (int i = 0; i < n; i++)
    {
        ret /= 2;
    }
    return ret;
}

private static IEnumerable<int> GetRemainNumbers(IEnumerable<int> subtracter)
{
    var bits = new bool[20];
    foreach (var bit in subtracter)
    {
        if (bits[bit * 2])
        {
            bits[bit * 2 + 1] = true;
        }
        else
        {
            bits[bit * 2] = true;
        }
    }

    var ret = new List<int>();
    for (int i = 0; i < bits.Length; i++)
    {
        if (!bits[i])
        {
            ret.Add(i / 2);
        }
    }

    return ret;
}
```