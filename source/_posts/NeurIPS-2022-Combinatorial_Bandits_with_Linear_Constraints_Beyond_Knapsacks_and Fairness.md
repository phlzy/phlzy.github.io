---
title: Combinatorial Bandits with Linear Constraints Beyond Knapsacks and Fairness
date: 2026-01-04
tags: [bandits]
categories: [Literature Review]
math: true
---



[本文](https://proceedings.neurips.cc/paper_files/paper/2022/hash/13f17f74ec061f1e3e231aca9a43ff23-Abstract-Conference.html) 率先研究了具有长期线性约束的组合 Bandit 问题，统一了背包约束、公平约束与群体公平约束三种约束形式的问题，提出了 UCB-LP 算法与一个低复杂度版本，并证明了这些算法的 regret bound 与 violation。



<!--more-->

# 问题模型

常规设置，$N$ arms，$T$ rounds，对于 arm $i$，随机奖励 $f_i(t)\sim P_i$。一轮最多选择 $m$ arms，动作集合 
$$
\mathcal{A}=\{\boldsymbol{a}|\boldsymbol{a}\in \{0,1\}^N,\;\|\boldsymbol{a}\|_1\le m\}
$$
反馈形式是 semi-bandit 的。单轮收益 $R_t\triangleq \sum_{i=1}^N f_i(t)a_i(t)$。

每一轮受到线性约束
$$
\boldsymbol{g}(\boldsymbol{a}(t))\triangleq [\boldsymbol{g}_1(\boldsymbol{a}(t)),\boldsymbol{g}_2(\boldsymbol{a}(t)),\dots, \boldsymbol{g}_K(\boldsymbol{a}(t))]^\top
$$
于是得到
$$
\max \sum_{t=1}^T R_t, \quad \text{s.t.} \; \sum_{t=1}^T \boldsymbol{g}(\boldsymbol{a}(t)) \preccurlyeq \boldsymbol{0}
$$
并且定义 regret（dynamic regret）
$$
\text{Reg}_T=\text{OPT}(T)-\mathbb{E}[\sum_{t=1}^T R_t]
$$
以及约束违背
$$
\text{Vio}(T)=\sum_{t=1}^T \boldsymbol{g}(\boldsymbol{a}(t))
$$

## 三种约束类型

**Bandits with Knapsacks**（BwK，背包约束）中，每次 pull 在获得奖励同时产生资源消耗，设有 $K$ 种资源，每种资源 $k$ 的总预算为 $B_k$，定义资源向量
$$
\lambda_i = (\lambda_{1,i},\dots,\lambda_{K,i}), \quad \lambda_{k,i}\ge 0
$$

背包约束

$$
\sum_{t=1}^T \lambda^\top \boldsymbol{a}(t) \le B
$$

从直觉上来看，BwK 约束侧重于限制资源的上界


将预算均摊到每一轮，可写成长期线性约束
$$
\boldsymbol{g}_k(\boldsymbol{a}(t)) = \lambda_k^\top \boldsymbol{a}(t) - \frac{B_k}{T}
$$

**Bandits with Fairness Constraints**（个体公平约束）要求每个臂 $i$ 在长期内至少被拉一定比例 $r_i$，形如
$$
\frac{1}{T} h_i(T) \ge r_i, \quad \forall i
$$
其中 $h_i(T)$ 是臂 $i$ 的累计拉动次数。可行性约束：

$$
\sum_{i=1}^N r_i \le m
$$

从直觉上来看，个体公平约束是对于探索下界的约束

公平约束在本文中被写成：
$$
\boldsymbol{g}(\boldsymbol{a}(t)) = -\boldsymbol{a}(t) + r
$$
**Bandits with Group Fairness Constraints**（群体公平约束）

臂被划分为若干组 $G_1,\dots,G_J$，公平性约束施加在组上，形如

$$
\sum_{i\in G_j} \kappa_{i,j} h_i(T) \ge r_j T
$$

其中 $\kappa_{i,j}$ 是权重。这类约束可以统一写成：

$$
\boldsymbol{g}_j(\boldsymbol{a}(t)) = r_j - \sum_{i\in G_j} \kappa_{i,j} \boldsymbol{a}_i(t)
$$

# UCB-LP

本文提出的算法 UCB-LP 可以处理上述所有问题，并且获得了更优的 regret bound。算法大概是在 UCB 框架中，使用线性规划算法求解长期线性约束下的最优策略。这么做的原理是用 UCB 去学习奖励向量 $\mu$ 的均值，基于 $\hat\mu$ 通过求线性规划问题得到当前最优动作分布。

此外，论文还提出了一个低复杂度版本 UCB-PLLP，通过拉格朗日松弛 + 悲观虚拟队列，避免直接求解高维度带约束线性规划问题，从而将计算复杂度降到 $\tilde{\mathcal{O}}(N)$。（这里的原理好像还是基于 Lyapunov drift 和 Slater 条件的，没仔细看）

# 主要贡献

UCB-LP 在线性约束的组合 Bandit 问题上可以做到 gap-dependent 的 $\tilde{\mathcal{O}}\left(\frac{mN\log T}{\Delta_{\min}}\right)$ 的 regret，并且期望意义下零约束违反。

对于 Bandits with Fairness Constraints，本文发现此类约束要求每个臂持续被拉，导致所有臂的估计误差同时收敛，从而使算法只会有限次选择次优 LP 解。因此，公平约束可以实现 $\mathcal{O}(1)$ regret + 零约束违反。

UCB-PLLP 实现 $\tilde{\mathcal{O}}(N)$ 的低计算复杂度和实现高概率零约束违反。



