### [Space/time trade-offs in hash coding with allowable errors](../assets/pdfs/bloom_filter_1970.pdf)

> Communications of the ACM, July 1970
>
> https://dl.acm.org/doi/10.1145/362686.362692

逐一测试一组消息（messages）是否为给定消息集（a given set of messages）成员时，如果允许少量测试消息被错误识别为消息集成员，则可以在不增加拒绝时间（reject time）的前提下减少哈希空间（hash area）。

> If a particular case is rejected, as would happen most 
> of the time, the simple process would be used. 
>
> If a case is not rejected, it could then be subjected to a 
> follow-up test to determine if it is actually a member of 
> the minority set or an "allowable error."

论文提出的两种 hash-coding 方法，适用于绝大部分被测试消息不属于给定的集合，且允许少量的 **allowable error** 的场景。算法考虑了三种计算因素:

1. reject time
2. space (i.e. hash area size)
3. allowable fraction of errors

> It will be shown that **allowing** a small number of test messages to 
> be **falsely identified** as members of the given set will permit a 
> much **smaller hash area** to be used without increasing the **reject time**.

在一些实际应用中，减少 hash 空间能够加速处理速度（当它需要存储在磁盘等介质时）。

论文中**自动断句**的例子，10% 的单词需要通过访问磁盘上的 dictionary，对这 10% 的单词构造 hash 空间，其它 90% 的单词中的绝大部分通过查询 hash 空间被 reject，使用简单规则就可以断句，但 90% 中的一小部分会被错误的被 hash 空间 accept，进而需要查询磁盘上的 dictionary，但这些单词并不在磁盘上，因而最后还是会使用简单规则。

#### Method 1

方法一类似传统的 hash-coding 方法，使用多个 hash cell 来保存 hash code，测试是按照 cell 维度进行比对。

> The code is generated from the message, and its size depends on the permissible fraction of errors.

当可接受的错误比例 P 越小，所需要的 hash cell 的个数越多。

#### Method 2

方法二则将 hash area 当做 N 个独立编址的初始值为 0 的比特位，每个 message set 中的消息的 hash code 对应一组比特位，a1, a2, ..., ad，都在 hash area 中被置为 1。当测试一个消息是否属于消息集时，如果消息 hash code 对应的每一个比特位在 hash area 中的位置都为 1，则该消息被 accept，否则被 reject。

> To test a new message a sequence of *d* bit addresses, say a1', a2', ..., ad', is generated 
> in the same manner as for storing a message. 
> 
> If all *d* bits are 1, the new message is
> accepted. If any of these bits is zero, the message is rejected.

#### Bloom filter hash 函数个数的选择

对于一个 m 位、n 个插入元素的布隆过滤器，需要 hash 函数的个数 k = (m/n)ln(2)，Bloom filter 各个变量的选择可使用如下流程:

1. Choose a ballpark value for n
2. Choose a value for m
3. Calculate the optimal value of k
4. Calculate the error rate for our chosen values of n, m, and k. If it's unacceptable, return to step 2 and change m; otherwise we're done.


#### References:

[1] [Bloom Filters by Example](https://llimllib.github.io/bloomfilter-tutorial/)

