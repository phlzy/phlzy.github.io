---
title: S8-FTRL
date: 2026-02-07
tags: [bandits]
categories: [Synapse]
math: true
---

在一个 semi-bandit 反馈的时变图上，使用 FTRL 算法求在线最小费用最大流问题（在线最短路），实现亚线性 regret bound 的定理证明

<!--more-->

# Proof of Theorem 3


这里我们采用标准的在线凸优化（Online Convex Optimization, OCO）证明框架。



## 1. 符号定义与准备 (Preliminaries)

首先定义问题的数学框架，确保符号统一。

> flow polytope 并不相当于 probability simplex over paths
>
> 概率单纯形 $\Delta_{|\mathcal X|}$ 是对所有简单路径取一个分布，$\pi\in\Delta_{|\mathcal X|}$，满足 $\sum_{\rho}\pi(\rho)=1$
> $$
> K=\mathrm{Conv}(\mathcal X)=\mathrm{Conv}{\chi(P):P\in\mathcal X}\subset\mathbb R^m
> $$
> 也就是“边使用向量/分数流”的集合
>
> 给定路径分布 $\pi$，定义边流
> $$
> x=\sum_{\rho\in \mathcal{X}}\pi(\rho)\mathcal{X}(\rho)
> $$
> 有 $\mathrm{Conv}(\mathcal{X})\equiv \{x\}$ 
>
> 选一个分数流 $x\in\mathrm{Conv}(\mathcal X)$ 等价于选某个路径分布 $\pi$，并把它投影成边的期望使用量。
>
> 它们不是同一个集合，原因很简单：
>
> - $\Delta_{|\mathcal X|}$ 的变量是“每条路径的概率”；
> - $\mathrm{Conv}(\mathcal X)$ 的变量是“每条边的期望使用量”。

定义边集 $\mathcal{E}$ 表示所有时隙可能出现的边集合，编号 $e=1,\dots,m$。对于简单路径 $\rho$，定义指示向量 $\mathcal{X}(\rho)\in \{0,1\}^m$，全体路径向量集合记为 $\mathcal{X}$。

定义流多面体 $\Xi=\mathrm{Conv}(\mathcal X)$，FTRL 更新过程为
$$
x_t=\mathrm{arg}\min_{x\in \Xi}\langle x,\hat{L}_{t-1} \rangle + \frac{1}{\eta} \mathscr{R}(x)
$$
其中正则函数
$$
\mathscr{R}(x)=\sum_{j=1}^m x_j\ln x_j
$$
假设每一轮边成本有界，有 $0\le C_e(t)\le C_\max$，因此 $\|\ell_t\|_\infty \le C_\max$

无偏采样，有
$$
\mathbb{E}[X_t\mid x_t]=x_t,X_t\in \mathcal{X}
$$
因此 $\mathbb{E}[\langle \ell_t,X_t\rangle \mid x_t]=x_t$。

semi-bandit 的 importance-weighted 估计器，
$$
\hat{\ell}_e(t)=\frac{C}{q_e(t)}\mathbf{1}\{e\in \rho_t\}
$$
然后 $\hat{L}_t=\hat{L}_{t-1}+\hat{\ell}_t$

条件无偏，基于全部历史（包含采样随机性与观测）
$$
\mathbb{E}[\hat{\ell}_t\mid \mathcal{F}_{t-1}]=\ell_t
$$


目标是和 hindsight 最优，static regret



Let
$$
B:=\max_{x\in \Xi} \| x\|_1
$$

## Proof

根据采样的无偏性，
$$
\mathbb{E}[\langle \ell_t,X_t-x^\star \rangle\mid x_t]=\langle \ell_t,x_t-x^\star \rangle
$$
对历史取全期望并求和，得到
$$
\mathbb{E}[\sum_{t=1}^T\langle \ell_t,X_t-x^\star \rangle]=\mathbb{E}[\sum_{t=1}^T\langle \ell_t,x_t-x^\star \rangle]
$$
由于 $x_t$ 是 $\hat{L}_{t-1}$ 的函数，$x_t$ 对 $\mathcal{F}_{t-1}$ 可测，根据条件无偏性质，有
$$
\mathbb{E}[\langle \ell_t,x_t-x^\star \rangle]=\mathbb{E}[\langle \hat{\ell}_t,x_t-x^\star \rangle \mid \mathcal{F}_{t-1}]=\mathbb{E}[\langle \hat{\ell}_t,x_t-x^\star \rangle]
$$
定义累积预测损失 $\hat{L}_t=\sum_{s=1}^t \hat{\ell}_s$，定义
$$
F_t(x)=\langle x,\hat{L}_{t-1} \rangle + \frac{1}{\eta} \mathscr{R}(x)
$$
FTRL 选择
$$
x_t=\mathrm{arg}\min_{x\in \Xi} F_t(x)
$$
根据 FTL-BTL 引理，有
$$
\sum_{t=1}^T \langle \hat{\ell}_t,x_t-x^\star \rangle\le \frac{\mathscr{R}(x^\star)-\mathscr{R}(x_1)}{\eta}+\sum_{t=1}^T \langle \hat{\ell}_t,x_t-x_{t+1} \rangle
$$

### 直径项上界

$\forall x\in \Xi$，有 $0\le x_j\le 1$，即 $\mathscr{R}(x)\le 0$

根据函数 $\phi(u)=u\ln u$ 的凸性，令 $s=\|x\|_1$，有
$$
\mathscr{R}(x)\ge s\ln \frac{s}{m} \ge -s\ln m
$$
所以这块上界是 $B\log m$

### 稳定项上界

Bregman 散度 $B_R(x\| y)$
$$
B_R(x\|y) := \mathscr{R}(x)-\mathscr{R}(y)-\langle \nabla\mathscr{R}(y),x-y \rangle\\
=\sum_{j}(x_j\log\frac{x_j}{y_j}-x_j+y_j)
$$
根据不等式
$$
u\log u - u + 1\ge \frac{(u-1)^2}{2(u+1)}
$$
有
$$
x_j\log\frac{x_j}{y_j}-x_j+y_j\ge \frac{(x_j-y_j)^2}{2(x_j+y_j)}
$$
即
$$
B_R(x\|y)\ge \frac{1}{2}\sum_{j=1}^m\frac{(x_j-y_j)^2}{(x_j+y_j)}
$$
根据带权的 Cauchy–Schwarz，对任意实数 $a_j$ 与正权重 $w_j>0$，有

by weighted Cauchy–Schwarz with $w_j=x_j+y_j$
$$
\Big(\sum_{j=1}^m |a_j|\Big)^2
\le \Big(\sum_{j=1}^m \frac{a_j^2}{w_j}\Big)\Big(\sum_{j=1}^m w_j\Big)
$$
从而得到
$$
\|x-y\|_1^2=\Big(\sum_{j=1}^m |x_j-y_j|\Big)^2
\le \Big(\sum_{j=1}^m \frac{(x_j-y_j)^2}{x_j+y_j}\Big)\Big(\sum_{j=1}^m (x_j+y_j)\Big)
$$
即
$$
B_R(x\|y)\ge \frac{1}{2}\cdot\frac{\|x-y\|_1^2}{\|x\|_1+\|y\|_1}\ge \frac{\|x-y\|_1^2}{4B}
$$


然后推出标准 one-step inequality
$$
B_R(x_t\|x_{t+1})\le \eta\langle \hat{\ell}_t,x_t-x_{t+1} \rangle
$$
holder不等式
$$
\langle \hat{\ell}_t,x_t-x_{t+1} \rangle \le \|\hat{\ell}_t\|_\infty \|x_t-x_{t+1}\|_1
$$
联立，得到
$$
\langle \hat{\ell}_t,x_t-x_{t+1} \rangle\le 4\eta BG^2
$$

### 合并

$$
\sum_{t=1}^T \langle  \rangle \le B(\frac{\log m}{\eta}+4\eta G^2T)
$$

取 $\eta=\sqrt{\frac{\log m}{4TG^2}}$





