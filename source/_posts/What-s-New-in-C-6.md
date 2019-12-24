---
title: 'What''s New in C# 6'
date: 2016-12-02 16:29:38
tags: C#
categories: 计算机
---
C# 6.0发布很多小功能，帮助开发者提高生产力。

整体来说，这些功能让我们编写的代码更简洁，也更具可读性。了解这些功能，工作效率会更高，写的代码的可读性更好，能够更好的专注于实现自己想要的功能，而不是其他的什么东西。

自动属性的增强

只读属性  
其实C#里面的自动属性已经很好用了，简洁，提供多样的访问性控制。  
不过长久以来，我们申明只读属性本质上都不是只读的，因为类内部可以修改，只是对于使用者来说是只读的。
``` csharp
public string FirstName { get; private set; }
public string LastName { get; private set; }
```
新的版本提供了真正意义上的只读属性：
``` csharp
public string FirstName { get; }
public string LastName { get; }
```
你只能在构造函数里面赋值，其他地方赋值都会编译错误。

自动属性初始化  
这个很简单，就像是这样：
``` csharp
public ICollection<double> Grades { get; } = new List<double>();
```
只会被初始化一次，这样就没有必要在构造函数里面写一遍了，对于有些开发者，构造函数不会重载使用，那样子可以少写很多次。

Expression bodied函数  
如果一个函数只有一行，不用大括号括起来了，直接=>就可以了。
``` csharp
public override string ToString() => $"{LastName}, {FirstName}";
public string FullName => $"{FirstName} {LastName}";
```
第一个是重载了以函数；第二个是只读属性，同样好用。

using static  
这个feature让我们写代码的时候能少敲几个字母吗？估计不行，因为反正有智能提示。  
好处是让代码更简洁一些。
``` csharp
using static System.String;
if (IsNullOrWhiteSpace(lastName))
    throw new ArgumentException(message: "Cannot be blank", paramName: nameof(lastName));
```

?. Null条件操作符  
这个挺好用的，先判断前面的对象是否是空，如果是null，就不进行后面的操作。对于event的调用，很好用，因为最常用的pattern就是先看event是不是null，不是null的话Invoke。
``` csharp
var handler = this.SomethingHappened;
if (handler != null)
    handler(this, eventArgs);

this.SomethingHappened?.Invoke(this, eventArgs);
```
现在只需要一行就搞定了。对于某些公司要求if必须有大括号（我个人也喜欢打）的话，能省掉更多的行数。

String interpolation  
这个在前面的代码里面出现了，比如FullName这个只读属性，以前要这么写：
``` csharp
public string FullName
{
    get
    {
        return string.Format("{0} {1}", FirstName, LastName);
    }
}
```
现在只需要一行就能搞定。  
这个里面可以写很复杂的表达式，调用函数等等。  
我个人觉得，打日志拼接字符串的时候很有用，以前string.Format里面{}能高达十个以上的参数，都跟在后面，鬼知道哪个对应数字编号几，有了这个feature，可读性大大提高了。

异常过滤  
个人感觉是一个简化代码的小feature，相比C# 7.0而言，提高并不是太多，这里就不举例子了。

nameof  
超级有用的feature。  
以前写MVVM SDK的时候，太多地方会用PropertyChangedEventArgs，写一个string，有的时候变量名换了，这里忘记换了，导致运行结果不正确，debug半天，有了这个功能，直接rename重构，就能全部换掉了。  
还有就是抛异常的时候，需要写入参变量名，这样子就能自动rename而不用自己修改了。

Index Initializers  
以前List这种集合可以方便的再后面{}初始化，现在Dictionary也可以了。
``` csharp
private Dictionary<int, string> webErrors = new Dictionary<int, string>
{
    [404] = "Page not Found",
    [302] = "Page moved, but left a forwarding address.",
    [500] = "The web server can't come out to play today."
};
```
新版本还提供了通过扩展Add方法，使得自定义的集合类也可以使用类似的初始化语法。