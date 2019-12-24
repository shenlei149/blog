---
title: C++中vector的增长因子
date: 2015-08-24 17:26:09
tags: [C++, vector, 内存]
categories:
 - 计算机
 - 计算机原理
---
我们都知道C++的vector的容量会随着添加的元素而自动增长，但是每次增长多少呢？原来的两倍？三倍？还是多少？下面，让我们来研究下增长因子是如何确定的。

首先，要阐述一个vector实现相关的事实：  
vector使用allocator，而不是 realloc，所以，不管你增长因子是多少，必然需要重新copy-cons(或move-cons)一遍。

废话不多讲了，下面进入正题。  
假设，我们用了一个vector，其占用内存为M，内存布局如下图所示：
<table>
<tr>
<td>Free</td><td style="width:100px">M</td><td>Free</td>
</tr>
</table>

我们一直使用这个vector，当元素太多导致内存不够的时候，要重新给这个vector分配内存，新分配的内存大小为f \* M。  
首先，要先分配f \* M的空间，注意，这个时候M那块内存还没有释放掉：  
<table>
<tr>
<td>Free</td><td style="width:100px">M</td><td style="width:150px">f \* M</td><td>Free</td>
</tr>
</table>

然后把之前的M空间释放掉：  
<table>
<tr>
<td>Free</td><td style="width:100px">Free(M)</td><td style="width:150px">f \* M</td><td>Free</td>
</tr>
</table>

我们继续使用这个vector，内存又不够了，再次分配的内存大小为f \* f \* M。  
和上面类似，分配内存时：
<table>
<tr>
<td>Free</td><td style="width:100px">Free(M)</td><td style="width:150px">f \* M</td><td style="width:225px">f \* f \* M</td><td>Free</td>
</tr>
</table>
然后释放掉前一块地址：    
<table>
<tr>
<td>Free</td><td style="width:100px">Free(M)</td><td style="width:150px">Free(f \* M)</td><td style="width:225px">f \* f \* M</td><td>Free</td>
</tr>
</table>

以此类推，第n次重新分配内存时，需要新分配f ^ n \* M大小的内存，内存分布如下所示(带括号说明已经释放了)：  
<table>
<tr>
<td>Free</td><td style="width:50px">(M)</td><td style="width:70px">(f \* M)</td><td style="width:70px">(...)</td><td style="width:120px">f ^ (n-1) \* M</td><td style="width:130px">f ^ n \* M</td><td>Free</td>
</tr>
</table>
然后再把之前的f ^ (n-1) \* M大小的内存回收:  
<table>
<tr>
<td>Free</td><td style="width:50px">(M)</td><td style="width:70px">(f \* M)</td><td style="width:70px">(...)</td><td style="width:120px">(f^(n-1) \* M)</td><td style="width:130px">f ^ n \* M</td><td>Free</td>
</tr>
</table>

我们思考这样一个问题：如果当第n次进行内存扩展时，前面n-2次操作释放的内存（包括第n-2次的内存和最开始占用的内存M）之和大于第n次所需要的内存，那么内存分配器就可以用之前留下来的内存而不用再往后去寻找Free的块。按照这个想法得到的内存分布如下：  
<table>
<tr>
<td>Free</td><td style="width:150px">f ^ n \* M</td><td style="width:70px">(GAP)</td><td style="width:130px">(f ^ (n-1) \* M)</td><td>Free</td>
</tr>
</table>

这样做的好处有两个，一是可以提高内存分配器的效率，更重要的是，\*\*内存的使用更加紧凑，局部性更好，能够更好的利用缓存机制\*\*。

上述想法包含一个约束条件，即下面的不等式（两边已同除了M）：  
f^n <= 1 + f + f^2 + ... + f^(n-2)  
当f=1.5时，n最小为5，也就是说，第五次进行内存扩展的时候，可以利用前面释放的内存。  
当f=2时，2^n <= 2^(n-1) -1，不等式永假，也就是说，vector再也不能利用之前释放的内存。  

从不等式可以得到，f越小，能满足不等式的n也会随之减小，但是如果f很小，会频繁的遇到内存不足进行扩展的情况，也就是说，会经常性的copy-cons或者move-cons vector里面的元素，得不偿失。而且5也足够小了，所以1.5是一个不错的选择。

读到这里，可能很多人会说，内存分配器是按照First-Fit来进行内存分配的吗？你怎么能假设“内存管理器就可以用之前留下来的内存而不用再往后去寻找Free的块”呢？而且，你这示例图都是连续的内存，实际情况未必如此啊。其实现代内存分配器很复杂，可能会把内存先分成若干块，每一块接受不同大小的内存分配，比如第一块都是内存需求量小于64B的，第二块都是内存需求量64B至128B，第三块都是需求量128B-1KB的等等；同时每一块又可能有不同的分配回收机制。所以实际情况远比我们想象的复杂，想要真实性能，做性能测试才是靠谱的途径。

排除这些外界因素，仅从本文分析的角度看来，增长因子1.5是个不错的选择，比2要好，因为使用2永远不会利用到之前释放的内存。

Facebook利用上述分析的原理（增长因子是1.5），配合jemalloc，开发了一套fbVector（已开源），性能比GCC的要好。  
Microsoft的VC++ STL中的vector实现，增长因子也是1.5以获取更好的性能。
而GCC和CLang还停留在古老的增长因子为2的阶段。

下面这段小程序，能够帮你测试出当前实现的增长因子：  
``` cpp
#include <vector>
#include <iostream>

int main()
{
	std::vector<int> arr;

	for (int i = 1; i < 10; i++)
	{
		arr.push_back(i \* i);
		std::cout <<
			"Size: " << arr.size() << "; " <<
			"Capacity: " << arr.capacity() << "\n";
	}

	return 0;
}
```