---
title: 算法数论摘录
date: 2023-05-08
tag: [math]
category: [math]
math: true
---

数学太差了，很多东西都不会，只能摘录一点能理解的。

# 质数

## Mersenne 质数

先来看一个引理：

若 $n\gt 1$，且 $a^n-1$ 为质数，则 $a=2$，$n$ 为质数。

这个太显然了，证明过程就不写了。

$M_n=2^n-1$ 称为第 $n$ 个 Mersenne 数。当 $p$ 为质数且 $M_p$ 为质数时，$M_p$ 称为 Mersenne 质数。

## Fermat 质数

先证明一个引理：

若 $2^m+1$ 为质数，则 $m=2^n$。

容易发现，若 $m$ 有奇因子 $q$，令 $m=qr$，则 

$$
2^{qr}+1=(2^r+1)\sum_{i=0}^{q-1}(-1)^i2^{ri}
$$

显然不是质数。

$F_n=2^{2^n}+1$ 称为第 $n$ 个 Fermat 数，$F_0$ 到 $F_4$ 均为质数，所以 Fermat 猜测所有这样的数都是质数。然而接下来的几个都不是质数，下一个质数直到 $F_{10}$ 才会出现，而这个数已经有三百多位了。

## Euler 函数

### Euler 定理

若 $\gcd(k,m) = 1$，则 

$$k^{\varphi(m)}\equiv 1 \pmod m$$

证明十分巧妙：

从模 $m$ 的 $\varphi(m)$ 个剩余类中各取一个代表元，组成一个缩剩余系，简称缩系。

用 $a_1,a_2,\cdots,a_{\varphi(m)}$ 表示一个缩系，由于 $\gcd(k,m)=1$，则 $\lbrace ka_i\rbrace$ 也是一个缩系。于是

$$
\prod_{i=1}^{\varphi(m)} ka_i \equiv\prod_{i=1}^{\varphi(m)} a_i \pmod m
$$

根据 $\gcd(a_i,m) = 1$，可得 $k^{\varphi(m)}\equiv 1 \pmod m$。

### Fermat 小定理

若 $p$ 为质数，$\forall a\in \mathbb{N}$，有

$$
a^p \equiv a \pmod p
$$

使用 Euler 定理即可证明。

# 同余

## Wilson 定理

若 $p$ 为质数，则

$$
(p-1)!\equiv -1\pmod p
$$

## 原根

### 重要结论

若 $p$ 为质数，则同余方程

$$
x^k \equiv 1\pmod p
$$

的解数为 $\gcd(k, p - 1)$。

### 阶

设 $h$ 为整数，$\gcd(h,n)=1$，满足 

$$
h^l \equiv 1\pmod n
$$

的最小正整数 $l$，称为 $h$ 模 $n$ 的阶。

设 $l|p-1$，则模 $p$ 的阶为 $l$ 的互不同余的整数个数为 $\varphi(l)$。

### 原根

阶为 $p-1$ 的数称为模 $p$ 的一个原根。

$p$ 有 $\varphi(p-1)$ 个原根，若 $g$ 为 $p$ 的原根，则

$$
g^0,g^1,\cdots,g^{p-2}\pmod p
$$

为模 $p$ 的一个缩系。

### 指数

任一整数 $p(p\nmid n)$，必有一数 $a$，满足

$$
n=g^a \pmod p,0\le a\lt p-1
$$

$a$ 称为 $n$ 模 $p$ 的指数，记为 $a=\text{ind}_g n$。

指数的性质与对数类似：

$$
\text{ind}(ab) \equiv \text{ind}(a) + \text{ind}(b)\pmod {p-1} \\
\text{ind}(a^l)\equiv l\cdot \text{ind}(a) \pmod {p-1}
$$

### 构造

$m$ 有原根存在的充要条件：$m=2,4,p^l,2p^l$（$p$ 为奇质数）。


