---
title: 'Scheme: 实现map'
date: 2015-09-06 11:12:54
tags: [scheme, map]
categories:
 - 计算机
 - 函数式编程
---
map操作本身就不多介绍了，直奔主题。

首先，我们实现一个一元的map，也就是说，传给map的函数只接受一个参数。
``` scheme
(define (unary-map fn seq)
  (if (null? seq) '()
      (cons (fn (car seq))
            (unary-map fn (cdr seq)))))
```
很简单，就是一个简单的递归即可，就不详细的说了。我们举一个例子：
``` scheme
> (unary-map (lambda (x) (+ x 1)) '(1 2 3))
'(2 3 4)
```

在继续讲解之前，我们需要知道一些其他知识。Scheme内置的map可以处理任意多的参数，显然，我们需要知道如何定义任意参数的函数。  
Scheme使用.来表示把后面的参数打包起来。  
举个简单的例子说明（需要强调下，小点和字母d之间有一个空格）：
``` scheme
(define (bar a b c . d)
  (list a b c d))

> (bar 1 2 3 4 5 6)
'(1 2 3 (4 5 6))
> (bar 1 2 3)
'(1 2 3 ())
```

下面考虑实现自己的map函数了。  
思路其实也很简单，把每一个list里面的第一个元素拿出来传入fn让其运算，然后呢，把每一个list里面的第一个元素删除，得到一系列的list，然后递归调用。  
说起来容易，如何设计参数呢？  
第一个参数是一个list，第二个参数是.，也就是可变参数，把后面的所有list打包传进来。  
拿第一个list的第一个元素简单，如何把后面每个list的第一个元素取出呢？利用先开始写的一元map，(unary-map car other-list)，就得到其余list的第一个元素，和第一个list的第一个元素连接起来，传给fn去做计算。  
后续的移除第一个元素之后递归也是类似的。  
最后把结果连接成一个list。
``` scheme
(define (my-map fn first-list . other-list)
  (if (null? first-list) '()
      (cons (apply fn
                   (cons (car first-list)
                         (unary-map car other-list)))
            (apply my-map
                   (cons fn
                         (cons (cdr first-list)
                               (unary-map cdr other-list)))))))
```

简单的写两个case测试一下：
``` scheme
> (my-map + '(1 2 3) '(4 5 6) '(7 8 9))
'(12 15 18)

> (my-map (lambda (x) (+ x 1)) '(1 2 3))
'(2 3 4)
```