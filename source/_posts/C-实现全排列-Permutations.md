---
title: 'C#实现全排列 | Permutations'
date: 2013-11-19 16:01:22
tags: [算法, C#, 排列]
categories: 
 - 计算机
 - 算法
---
今天需要用到一个全排列算法，Google了一下，找了好几个，觉得下面的写法最NB，最ZB，最简洁。  
最核心的地方在于那个for循环和foreach那里。  
遍历每个元素，把这个元素从source里面取出来，递归的求得剩余元素的全排列，把这个取出来的元素放到剩余元素全排列的前面。  
That's all!  

``` csharp
/// <summary>
/// Returns all permutations of the input <see cref="IEnumerable{T}"/>.
/// </summary>
/// <param name="source">The list of items to permute.</param>
/// <returns>A collection containing all permutations of the input <see cref="IEnumerable<T>"/>.</returns>
public static IEnumerable<IEnumerable<T>> Permutations<T>(this IEnumerable<T> source)
{
    if (source == null)
        throw new ArgumentNullException("source");
    // Ensure that the source IEnumerable is evaluated only once
    return permutations(source.ToArray());
}

private static IEnumerable<IEnumerable<T>> permutations<T>(IEnumerable<T> source)
{
    var c = source.Count();
    if (c == 1)
        yield return source;
    else
        for (int i = 0; i < c; i++)
            foreach (var p in permutations(source.Take(i).Concat(source.Skip(i + 1))))
                yield return source.Skip(i).Take(1).Concat(p);
}
```