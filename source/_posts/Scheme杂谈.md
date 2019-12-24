---
title: Scheme杂谈
date: 2015-07-28 17:12:06
tags: [scheme]
categories:
 - 计算机
 - 函数式编程
---
假设有这样一个需求，我们需要对列表里面的每个数字都加倍，一种可能的操作是使用car cdr遍历递归，但是Scheme还提供了map这个内置函数，能够简洁快捷的实现。  
先来看看这个函数：

``` scheme
> (map + '(1 2) '(3 4) '(5 6))
'(9 12)
```
map的作用就是遍历每一个列表，然后用第一个参数来对它们进行求值。比如上例，后续的参数是长度为2的列表，那么结果也是一个长度为2的列表，第一项就是把后面每个列表的第一项拿出来，使用+来就行求值，得到了9，类似的，第二项是12。

为了使数字加倍，我们需要定义一个double函数：

``` scheme
(define (double x) (* x 2))
```
使用map和double得到我们的目的：

``` scheme
> (map double '(1 2 3))
'(2 4 6)
```
再介绍两个内置函数apply和eval，关键字就表示出它们的作用：

``` scheme
> (apply + '(1 2 3))
6

> '(+ 1 2 3)
'(+ 1 2 3)
> (eval '(+ 1 2 3))
6
```
来个简单的示例看看它们的应用：

``` scheme
(define (average num-list)
  (/ (apply + num-list)
     (length num-list)))
```
从上面几个例子，我们可以看出，使用apply map使得（至少看起来是的）所有元素平等的共同参与运算，而且，比使用car cdr递归更加直观，简洁。  
那什么时候用eval，按照js经验，可能会有不能用apply的时候，比如define or and这些时候，就只能用eval了。

最后，我们看一个稍微复杂的例子。实现内置函数flatten：  
flatten作用使得列表里面的元素展开，使之平级，都是最外层列表的元素。比如((1 2) 3)flatten之后就是(1 2 3)。  
思路是，如果seq（传入的列表）不是一个列表，就返回一个空列表；否则，我们使用map函数，对seq中的每一个子列表进行flatten操作，最后，使用append把这些结果连接起来。

``` scheme
(define (my-flatten seq)
  (if (not (list? seq)) (list seq)
      (apply append
             (map my-flatten seq))))

> (my-flatten '((1 2) (3 ((4) 5)) 6))
'(1 2 3 4 5 6)
```
最后这个例子稍稍复杂了点，综合应用了apply map和递归的思想。