---
title: 在单链表中找到倒数第k个结点
date: 2014-11-07 15:07:11
tags: [面试题, 链表]
categories:
 - 计算机
 - 算法
---
如果是正数第k个结点，很容易，依次往后遍历就好，现在是倒数第k个，该怎么办呢？  
我们使用两个指针p1和p2，p1指向开始位置，p2指向第k个结点，使得两者之间距离是k。然后同步的移动这两个指针，当p2指向最后一个结点的时候，p1指向的就是倒数第k个结点。  
算法的时间复杂度是O(n)，空间复杂度是O(1)。
``` csharp
private static LinkedListNode<T> KthToLast<T>(LinkedListNode<T> head, int k)
{
    if (k <= 0)
    {
        return null;
    }

    LinkedListNode<T> p1 = head;
    LinkedListNode<T> p2 = head;

    for (int i = 0; i < k - 1; i++)
    {
        if (p2 == null)
        {
            return null;    // Link length is less than k
        }

        p2 = p2.Next;
    }

    if (p2 == null)
    {
        return null;    // Link length is less than k
    }

    while (p2.Next != null)
    {
        p1 = p1.Next;
        p2 = p2.Next;
    }

    return p1;
}
```