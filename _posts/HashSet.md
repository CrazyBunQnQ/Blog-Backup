---
title: HashSet 散列集
date: 2017-03-19 22:22:22
categories: Java
tags: 
- 数据结构
- 集合
- HashSet
- HashTable
- Set
- 哈希表
---

链表和数组可以按照人们的意愿排列元素的次序，但是如果想要查看某个指定的元素，却又忘记了它的位置，就需要访问所有元素，直到找到为止。如果集合中包含的元素很多，将会消耗很多时间。

<!-- more -->

## Hash Table 散列表
**散列表（hash table）**是一种以 **键-值（key-indexed）** 存储数据的结构，**可以快速的查找所需要的对象**。散列表为每个对象计算一个整数，成为[散列码（hash code）](/2017/03/18/HashCode 散列码/)。

在 Java 中，散列表用链表数组（又叫**桶数组**...）实现。每个列表被称为**桶（bucket）**。

桶数组其实就是一个容量为 N 的普通数组 A[0, 1, 2...N-1]，只不过在这里我们将每个单元都想象为一个“桶”，每个桶单元中可以存放一个条目。

比如，倘若所有的关键码（key）都是小于 N 的非负整数，我们就可以直接将 key 为关键码的那个条目（如果存在的话）存放在桶单元 A[key]内，为了节省空间，空闲的单元都被置为 null。由于 HashSet 底层是[映射](/2017/03/19/Map 映射/)结构，所以**每个桶单元中最多只能存放一个条目。**如此一来，查找、插入和删除操作都可以快速完成。

但是，这只是一种极为特殊的情况，通常往往不会这么碰巧。比如，这里很难确定数组的最佳容量 N，所以往往选用远大于映射实际规模 n 的某个 N：如果关键码都是 int 类型的非负整数，则至少需要 2 147 483 648 个桶单元，这远远超出了一般映射结构本身的规模，造成了空间的巨大浪费。

为了解决这个问题，就需要一个方法，可以将任意关键码转换为介于 0 与 N-1 之间的整数——这个函数就是散列函数。

</br>
## 散列函数
- 散列函数 h 将关键码 key 映射为一个整数 h(key) ∈ [0...N-1]，并将对应的条目存放到 h(key) 号桶内，其中 N 为桶数组的容量。
- 如果将桶数组记作 A[]，这一技术就可以总结为 “将条目 e = (key, value) 存放至 A[h(key)] 中”。
- 反过来，为了查找关键码为 key 的条目，只需要取出桶单元 A[h(key)] 中存放的对象。
- 因此，h(key) 也被称作 e 的散列地址。

>好吧，这里有点乱。如图，散列函数的计算划分为两步：

![Java 计算散列函数的过程](http://wx2.sinaimg.cn/mw690/a6e9cb00ly1fds4zpdrf5j20l10ca3zf.jpg)

1. 首先将一般性的关键码 key 转换为一个称作 “散列码” 的整数。
2. 通过所谓的 “压缩函数” 将该整数 映射至区间 [0...N-1] 中。

>不过，若要对线上述构思，还需要满足一个条件——不同的关键码 key1 ≠ key2 必须要对应于不同的散列地址 h(key1) ≠ h(key2)。
>但是还是可能会无法满足该条件，如果不同关键码的散列地址相同，就会发生**[散列冲突（hash collision）](/2017/03/19/HashSet 散列集/#散列冲突)**

</br>
### 散列码
Java 可以帮助我们将任意类型的关键码 key 转换为一个整数，称作 key 的散列码（Hash Code）。超类 Object 提供了一个默认的散列码转换方法 [hashCode()](/2017/03/18/HashCode 散列码/)，利用它可以将任意对象实例映射为 “代表” 该对象的某个整数。

>**注意:**散列码距离我们最终所需的散列地址还有很大距离——它不一定落在区间 [0...N-1] 内，甚至不一定是**正**整数。

</br>
### 压缩函数
如果直接将散列码作为桶数组的胆原地址，则桶的容量将达到 2^32 = 4 G！即使能够提供如此大的空间，其利用率也是极低的。因此就需要将散列码进一步压缩至我们希望的 [0...N-1] 区间内。

压缩函数有两种：【模余法】和【MAD 法】

**模余法压缩:**
最简单的压缩办法，就是**取 N 为素数，并将散列码 i 映射为： `|i| mod N`**。

之所以将 N 选为素数，是为了最大程度的将散列码均匀地映射至 [0...N-1] 区间内。若所有关键码都是在 [0...N-1]内随机均匀分布的，则其中每一对关键码发生冲突的概率都是 1/N。

因此，月石能够使得这个概率接近于 1/N，我们选用的散列函数就越好。

**MAD 法：**
模余法是一个简单易行的策略，但如果关键码的分布具有 aN + b 的模式(啊~这模式我也没懂，先了解吧，以后明白了回来补上...)，则仍然会发生不少冲突。
因此，可以采用另一种方法，将乘法（Mutiply）、加法（Add）和除法（Divide）结合起来的方法，该方法也因此得名。
**对于散列码 i，MAD 法会将 i 映射为 `|a*i + b| mod N`**。
其中 N 仍为素数，a > 0，b > 0，a mod N ≠ 0，它们都是在确定压缩函数时随机选取的常数。

</br>
## 散列冲突
在插入元素时，两个不同对象的散列码一样，导致不同关键码的散列地址相同，就会导致散列冲突。不管怎样，散列冲突的可能性总是存在的。

如果大致知道最终会有多少个元素要插入到散列表中，就可以设置桶数，一般将桶数设置为预计元素个数的 75%~150%。
但是，并不是总能够知道需要存储多少个元素的。也可能最初的估计过低。

如果散列表太满，就需要**再散列**，就需要创建一个桶数更多的表，并将所有元素插入到这个新表中，然后丢弃原来的表。

**装填因子（load factor）**决定何时对散列表进行再散列。例如，如果装填因子为 0.75(默认值)，而表中超过 75% 的位置已经填入元素，这个表就会用**双倍**的桶数自动的进行再散列。

