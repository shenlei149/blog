---
title: 'Scheme: 使用Lambda表达式实现排列和组合算法'
date: 2015-08-11 17:20:54
tags: [scheme, 排列, 组合]
categories:
 - 计算机
 - 函数式编程
---
在Scheme定义一个函数，通常写法如下：
``` scheme
(define (sum x y)
  (+ x y))
```
还有一种类似于定义变量的方式：
``` scheme
(define sum (lambda (x y)
  (+ x y)))
```
利用lambda表达式，把sum和一个匿名函数联系在一起。  
这种方式有点类似于JS：
``` js
var sum = function(x, y) { return x + y; }
```

利用lambda表达式，能够简化代码，下面举一个简单的例子。
``` scheme
(define (translate point delta)
  (define (shift-by x)
    (+ x delta))
  (map shift-by point))
```

修改后：
``` scheme
(define (translate point delta)
  (map (lambda (x)
         (+ x delta))
       point))
```
很显然，第二种更简洁直观。

Scheme还内置了let关键字，给人的感觉是创建了变量，其实不然，只是把某个表达式给了个名字罢了。  
(let ((x exp1) (y exp2) (z exp3)) (fn x y z))  
等价于  
((lambda (x y z) (fn x y z)) exp1 exp2 exp3)  
需要注意的是，let里面是个list，不保证求值顺序，所以exp3里面不能使用x和y。

好了，现在开始实现一个方法，返回集合的所有子集。
``` scheme
(define (ps set)
  (if (null? set) '(())
      (let ((ps-rest (ps (cdr set))))
        (append ps-rest
                (map (lambda (subset)
                       (cons (car set) subset))
                     ps-rest)))))
```
现在解释一下整个方法的思路：首先，如果set为空，返回一个由空列表组成的列表；下面运用递归的思想，定义ps-rest是set排除第一个元素的集合的所有子集，比如传入的set是'(1 2 3)，排除第一个元素就是'(2 3)，它的所有子集组成的列表是'(() (3) (2) (2 3))，列表里面共四个元素，这就是ps-rest。我们往ps-rest里面append包含第一个元素1的集合。具体append什么呢？我们mapps-rest，在每一项前面加上第一个元素1，ps-rest第一项是空集，连接完之后是(1)，append到ps-rest里面，第二项是(3)，连接完之后是(1 3)，放到ps-rest里面，第三项第四项类似。最终，我们就得到了结果'(() (3) (2) (2 3) (1) (1 3) (1 2) (1 2 3))。

下面实现算法相对更复杂的全排列。
``` scheme
(define (permute items)
  (if (null? items) '(())
      (apply append
             (map (lambda (ele)
                    (map (lambda (pers)
                           (cons ele pers))
                         (permute (remove ele items))))
                  items))))
```
解释下算法。从apply开始说吧。我们把一系列结果append起来，也就是把后面map里面的求值结果串起来。map的求值结果是什么呢？它遍历了输入的items，比如我们输入'(1 2 3)，第一次遍历使用的是1，也就是向map的第一个参数lambda表达式传入了1，也就是ele是1。接着，对1做了什么呢？又是一个map，遍历了(permute (remove ele items))，remove是从items中移出ele，这个表达式求值结果是'((2 3) (3 2))，遍历它，把1加到每项的前面，得到的结果是'((1 2 3) (1 3 2))。至此，第一次遍历完成，回到外层的map，开始第二次遍历，传入ele为2，后续就不再赘述了。最终得到的结果是'((1 2 3) (1 3 2) (2 1 3) (2 3 1) (3 1 2) (3 2 1))。