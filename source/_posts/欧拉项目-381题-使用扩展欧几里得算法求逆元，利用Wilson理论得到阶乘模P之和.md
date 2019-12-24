---
title: 欧拉项目 | 381题 | 使用扩展欧几里得算法求逆元，利用Wilson理论得到阶乘模P之和
date: 2015-06-20 15:09:18
tags: [欧几里得, 乘法逆元, 模, 欧拉项目, Project Euler, 数学]
categories: 数学
---
[题目381](https://projecteuler.net/problem=381 "Problem 381 - Project Euler")的描述如下：  
对于一个质数p来说，S(p) = (∑(p-k)!) mod(p) 其中 1 ≤ k ≤ 5  
举个例子  
(7-1)! + (7-2)! + (7-3)! + (7-4)! + (7-5)! = 6! + 5! + 4! + 3! + 2! = 720+120+24+6+2 = 872.  
因为872 mod(7) = 4, 所以S(7) = 4.  
对于5 ≤ p < 10 ^ 8，求∑S(p)

题目给了一个可以检验程序是否写正确的范例：∑S(p) = 480 其中5 ≤ p < 100.

根据[Wilson's theorem](https://en.wikipedia.org/wiki/Wilson%27s_theorem "Wilson's theorem")可以得到下面等式：  
(p−1)!≡−1  
(p−2)!≡(p−1)!(p−1)^−1≡1  
(p−3)!≡(p−2)!(p−2)^−1≡(p−2)^−1  
(p−4)!≡(p−3)!(p−3)^−1≡(p−2)^−1 \* (p−3)^−1  
(p−5)!≡(p−4)!(p−4)^−1≡(p−2)^−1 \* (p−3)^−1 \* (p−4)^−1  
都是模p的，省略了。

(p−2)^−1 (p−3)^−1 (p−4)^−1这玩意如何整成整数呢？  
下面给出逆元的定义：  
对于整数n、p，如果存在整数b，满足nb mod p =1，则说，b是n的模p乘法逆元。并且：  
n存在模p的乘法逆元的充要条件是gcd(n,p) = 1  
显然，p-2 p-3 p-4与p互质，最大公约数就是1

那么如何求逆元呢？考虑[扩展欧几里得算法](http://blog.guozi149.me/%E8%AE%A1%E7%AE%97%E6%9C%BA/%E7%AE%97%E6%B3%95/%E6%89%A9%E5%B1%95%E6%AC%A7%E5%87%A0%E9%87%8C%E5%BE%97%E7%AE%97%E6%B3%95/)  
ap + b(p-2) = 1  
显然，p-2的逆元就是扩展欧几里得算法中的b。

到此为止，基础的数学知识都已经准备妥当，开始写代码吧。

``` csharp
var primes = Utils.GenPrimes(100000000);
for (int i = 2; i < primes.Length; i++)
{
    long p = primes[i];
    long a = 0;
    long p2 = 0;
    long p3 = 0;
    long p4 = 0;
    Utils.GetGcd(p, p - 2, ref a, ref p2);
    Utils.GetGcd(p, p - 3, ref a, ref p3);
    Utils.GetGcd(p, p - 4, ref a, ref p4);

    long s3 = ((p2 % p) + p) % p;
    long s4 = ((s3 * p3) % p + p) % p;
    long s5 = ((s4 * p4) % p + p) % p;
    long s = ((s3 + s4 + s5) % p + p) % p;
    sum += s;
}

return sum;
```
简单的解释一下，primes是2到10 ^ 8所有的质数，利用的是埃拉托斯特尼筛选法。  
由于得到的逆元可能很大，也可能是负数，所以模p后加了p再进行一次模运算，实在懒的判断其是否大于0了，简单粗暴的写法。  
后半段程序利用了点模的运算规则，都很基础，不再赘述。