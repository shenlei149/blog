---
title: 神奇的Scheme之强大的递归思想
date: 2015-07-20 16:51:21
tags: [scheme, 递归]
categories:
 - 计算机
 - 函数式编程
---
在写递归函数之前，先简单介绍几个后续会用到的内置函数：  
car 返回列表的第一个元素，列表不能是空'()  
(car '(1 2 3))  
1  
(car '((1 2) 3))  
'(1 2)

cdr 返回一个列表但不包含第一个元素，同样，列表不能为空'()  
(cdr '(1 2 3))  
'(2 3)  
(cdr '((1 2) (2 3) 4))  
'((2 3) 4)

car和cdr可以组合使用，比如cadr表示从不包含第一个元素的列表中返回第一个，即返回第二个元素  
(cadr '(1 (2 3) (3) 4))  
'(2 3)  
为什么可以组合用呢？Scheme的内存模型决定了这一点，以后再详解。

cons 把第一个参数（可以是列表）作为第二个参数的第一个元素，并返回  
(cons '(1) '(2 3))  
'((1) 2 3)

append 把后续的参数加到第一个参数列表后面  
(append '(1) '((2 3) 4))  
'(1 (2 3) 4)

有了以上铺垫开始写第一个简单的函数，求和:

``` scheme
(define (sum-of num-list)
  (if (null? num-list) 0
      (+ (car num-list)
         (sum-of (cdr num-list)))))
```
如果传入的list是null，求值结果是0，否则，求值结果是list第一项加上其余项之和。  
(sum-of '(1 2 3 4 5))  
求值结果是15

再来个求list是否排序的通用函数

``` scheme
(define (sorted? list cmp)
  (or (< (length list) 2)
      (and (cmp (car list)
                (cadr list))
           (sorted? (cdr list) cmp))))
```
这个函数稍稍复杂一点。  
如果说长度小于2，也就是1或者0，那么就是有序的，求值为真；  
否则，比较一下第一个和第二个是否有序，如果有的话，递归地求剩余长度为n-1的list是否有序。  
这里利用了逻辑运算的短路特性。

``` scheme
(sorted? '(1 2 46 7) <)
#f
(sorted? '("a" "b" "c" "d" "e") string<?)
#t
```