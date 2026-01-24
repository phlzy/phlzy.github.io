---
title: S6-并非博弈
date: 2025-07-15
tag: [linear algebra]
category: [Synapse]
math: true
---

Synapse 是一个专题笔记系列，每一篇的 topic 都是随机的，也许一些看似无关的笔记之间会产生某种联系

<!--more-->

这次来看一个有些奇怪的问题

# 问题

Alice 和 Bob 在玩一个奇怪的游戏：有 $n$ 堆石子，每堆石子的个数为 $s_i$，游戏开始时，Bob 先构造 $n$ 个集合，记为 $B_1\dots B_n$，每个集合的元素均为不小于 $1$ 且不大于 $n$ 的自然数，集合不能为空，元素不重复。Bob 构造完成后向 Alice 揭示所有的集合，Alice 看过这些集合后也按照相同的规则构造一个集合（集合的元素均为不小于 $1$ 且不大于 $n$ 的自然数，集合不能为空，元素不重复），记为 $A$。随后，遍历所有的堆，对于第 $i$ 堆石子，如果集合 $B_i$ 和 $A$ 的交集的大小是偶数，则 Bob 取走第 $i$ 堆石子，否则 Alice 取走第 $i$ 堆石子。游戏结束后，获得更多的石子的一方获胜。问 Alice 是否存在必胜策略。



# 答案

考虑将每个集合表示成 $\{0,1\}^n$ 的向量，例如，将集合 $A$ 转化为向量 $a=(a_1,a_2,\dots,a_n)^\intercal$，如果 $j\in A$，则 $a_j=1$，否则 $a_j=0$。用同样的方法将集合 $B_i$ 转化为向量 $b_i$。

那么，交集大小就是：
$$
|A\cap B_i|=a^\intercal \cdot b=\sum_{j=1}^n a_j b_{ij}
$$
假设 Alice 采取了某种策略 $a$，定义她的收益（即获得的石子数量）为
$$
R(a)\triangleq \sum_{i=1}^n s_i(a^\intercal \cdot b\pmod 2)
$$
算上不被允许的空集在内，Alice 一共有 $2^n$ 种策略，用 $\mathcal{A}$ 表示所有这些策略的集合。这所有策略产生的总收益是

$$
\sum_{a\in \mathcal{A}}R(a)=\sum_{a\in \mathcal{A}}\sum_{i=1}^n s_i\bigg(a^\intercal \cdot b_i\pmod 2\bigg)\\
=\sum_{i=1}^n s_i\bigg(\sum_{a\in \mathcal{A}}a^\intercal \cdot b_i\pmod 2\bigg)
$$

这里的 $\sum_{a\in \mathcal{A}}a^\intercal \cdot b\pmod 2$ 就有很好的性质了。注意到 $b_i$ 是一个非零向量，所以它的秩为 $1$，解空间的维度就是 $n-1$，即，总共有 $2^{n-1}$ 个向量 $a$ 满足 $a^\intercal \cdot b_i= 0\pmod 2$。相应的，有 $2^{n-1}$ 个向量 $a$ 满足 $a^\intercal \cdot b_i= 1\pmod 2$。

因此，可以求出所有策略的总收益为
$$
\sum_{a\in \mathcal{A}}R(a)=2^{n-1}\sum_{i=1}^n s_i
$$

显然，如果 Alice 选择空集，她的收益是 $0$。那么，如果她从合法的策略中随机抽取一个，她获得的平均收益是
$$
\bar{R}(a)=\frac{2^{n-1}}{2^n-1}\sum_{i=1}^n s_i\\
=\frac{2^n}{2^n-1}\bigg(\frac{1}{2}\sum_{i=1}^n s_i\bigg)
$$
这说明，至少存在一种策略 $a$，使得 Alice 的收益超过了石子总数的一半。因此，Alice 是必胜的。

