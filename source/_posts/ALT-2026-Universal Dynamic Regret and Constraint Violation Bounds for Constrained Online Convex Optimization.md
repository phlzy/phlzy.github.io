---
title: Universal Dynamic Regret and Constraint Violation Bounds for Constrained Online Convex Optimization
date: 2026-01-21
tags: [bandits]
categories: [Literature Review]
math: true
---



读一个 OCO 的工作。



<!--more-->

# 问题模型

OCO 问题，$T$ rounds，convex decision set $\mathcal{X}\subseteq \mathbb{R}^d$，选择一个动作 $x_t$，回合结束时揭示 convex cost function $f_t : \mathcal{X}\mapsto \mathbb{R}$，learner 从而得到 cost $f_t(x_t)$。优化目标自然是最小化 $\sum_{t=1}^T f_t(x_t)$。

众所周知，在线梯度下降法 OGD 以及各种衍生算法都能做到 $\mathcal{O}\sqrt{T}$ 的 static regret。

## Bandits with Knapsacks

BwK 问题是上述问题的一个特殊版本，在 cost function 的基础上增加了 constraint function $g_t : \mathcal{X}\mapsto \mathbb{R}$，并且需要满足约束 $g_t(x_t)\le 0$，允许一定程度的 violation。于是优化目标新增了最小化 cumulative constraint violations（CCV）。

## Common Feasibility 假设

$$
\exists x^\star \in \mathcal{X} \quad\text{s.t.}\quad g_t(x^\star)\le 0,\forall t\ge 1.
$$

这是一个很经典的假设，意思是说存在一个始终都可行的解。这个假设很强，在时变环境下可能并不成立，一旦抛弃该假设，static regret 这个指标就不再有意义了。

这个研究没有采用该假设，所以后面用的就是 dynamic regret 的指标。

# 主要思想

## surrogate cost function

基于 $\{f_\tau,g_\tau\}_{\tau=1}^{t-1}$ 构建 surrogate cost function $\hat{f}_{t-1}$。



假设决策集合 $\mathcal{X}$ 的直径是 $D$，

定义函数 $g_t^+(x_t)\triangleq \max(0,g_t(x_t))$，则 CCV 就是 

$$
Q(T)=\sum_{t=1}^T g_t^+(x_t)
$$

假设所有的 cost 和 constraint 函数都是 $G$-Lipschitz 连续的，即

$$
|f(x) - f(y)| \le G \cdot \|x - y\|
$$

这里的范数 $\|\cdot\|$ 默认是 Euclidean 范数。

在没有约束的情况下，这就是标准的 OCO 问题。

Comparator Sequence

$u_{1:T}\triangleq \{u_1,u_2,\cdots, u_T\}$ 决策序列

Path Length

$$
\mathcal{P}_T(u_{1:T})\triangleq \sum_{t=2}^T\|u_t-u_{t-1}\|
$$

Feasible Set 

$$
\mathcal{X}_t^{\star}\triangleq \{x\in \mathcal{X}:\;g_t(x)\le 0\}
$$


这个也是凸的

Feasible Comparators：对于一个决策序列 $u_{1:T}$ 是feasible 的，当且仅当所有的 $u_t\in \mathcal{X}_t^\star$。

Universal Dynamic Regret

$$
\text{UD-Reg}(f_{1:T};u_{1:T})=\sum_{t=1}^T f_t(x_t)-\sum_{t=1}^T f_t(u_t)
$$

可以发现，当 Common Feasibility 假设成立时，存在 $x^\star \in \cap_{t=1}^T\mathcal{X}_t^{\star}$，选择 $u_t=x^\star$ 就得到了 static regret。

Worst-case Dynamic Regret

$$
\text{WD-Reg}(f_{1:T})=\sum_{t=1}^T f_t(x_t)-\sum_{t=1}^T \min_{x_t^\star \in \mathcal{X}_t^{\star}} f_t(x_t^\star)
$$

从 UDR 到 WDR



