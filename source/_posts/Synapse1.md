---
title: S1-关于质数的一些基础概念
date: 2021-08-19
tag: [number theory]
category: [Synapse]
math: true
---

Synapse 是一个专题笔记系列，每一篇的 topic 都是随机的，也许一些看似无关的笔记之间会产生某种联系

<!--more-->


## Prime Number Theorem

**质数定理**：

$[1,N]$ 中质数的个数 $\pi(N)\sim \dfrac{N}{\log(N)}$。

## Dirichlet's theorem on arithmetic progressions

**狄利克雷定理**：

对于任意互质正整数对 $(r,N)$，模 $N$ 余 $r$ 的质数集合相对于质数集合的密度为 $\dfrac{1}{\varphi(N)}$，其中 $\varphi(N)$ 为欧拉函数。

这个定理告诉我们一个有趣的事实：若正整数 $a,b$ 互质，则集合 $\{a+nb \mid n \in N\}$ 中有无穷多个质数。

## Dirichlet series

具有以下形式的无穷级数被称为**狄利克雷级数**：

$$\sum_{n=1}^{\infty}\dfrac{a_n}{n^s}$$

其中 $s$ 是一个复数，$a_n$ 是一个复数列。

很多狄利克雷级数都可以通过莫比乌斯反演和狄利克雷卷积得到。

## Riemann zeta function

黎曼 $\zeta$ 函数是一种特殊的狄利克雷级数。

设一复数 $s$ 满足 $\Re(s)\gt 1$，则定义：

$$\zeta(s)=\sum_{n=1}^{\infty}\dfrac{1}{n^s}$$

$s=1$ 时，右边就是我们熟悉的调和级数。

有这样一个优美的性质：

$$\dfrac{1}{\zeta(s)}=\sum_{n=1}^{\infty}\dfrac{\mu(n)}{n^s}$$

其中 $\mu(n)$ 就是莫比乌斯函数。

## Euler product

**欧拉乘积**：

若 $f$ 为积性函数，则狄利克雷级数 $\sum_n f(n)n^{-s}$ 等于欧拉乘积 $\prod_p P(p,s)$，其中 $P(p,s)$ 为 $\sum_{n=0}^{\infty}f(p^n)p^{-ns}$。

完全形态太复杂了，当 $f$ 为完全积性函数时，$P(p,s)$ 是等比级数，为 $\dfrac{1}{1-f(p)p^s}$。

有趣的是，当 $f(n)=1$ 时，欧拉乘积就变成了黎曼 $\zeta$ 函数，即：

$$\sum_{n=1}^{\infty}\dfrac{1}{n^z}=\prod _p\dfrac{1}{1-p^{-z}}$$

这个等式的证明非常有趣，我们将等式左边记为式 $(0)$，然后不断重复以下操作：

1.  将式 $(i-1)$ 乘上 $p_i^{-1}$，$p_i$ 表示第 $i$ 个质数，记为式 $(*)$；
2.  将式 $(i-1)$ 减去式 $(*)$，记为式 $i$。

容易发现这就是埃氏筛的原理，最后所有质数的倍数都被筛掉了，所以会得到这样的等式：

$$\prod_p(1-p^{-s})\zeta(s)=1^{-s}$$

把 $\prod$ 移到右边就完事了。

现在来看一个问题：任取两个正整数互质的概率是多少？

任取两个正整数，则它们都不是第 $k$ 个质数的倍数的概率应该是 $1-\dfrac{1}{p_k^2}$。

那么这两个数互质的概率就应该是 $\prod(1-\dfrac{1}{p^2})$，根据上面的等式，相当于 $\dfrac{1}{\zeta(2)}$，即 $\dfrac{6}{\pi^2}$。

这个问题还可以加强一下，详见 [这篇博客](https://www.cnblogs.com/pengdaoyi/p/9981997.html)。
