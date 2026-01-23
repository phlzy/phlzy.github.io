---
title: Universal Dynamic Regret and Constraint Violation Bounds for Constrained Online Convex Optimization
date: 2026-01-22
tags: [bandits]
categories: [Literature Review]
math: true
---



一个 OCO 的工作，surrogate cost function 的构造很巧妙



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

## 基础定义

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

**Comparator Sequence**

$u_{1:T}\triangleq \{u_1,u_2,\cdots, u_T\}$ 决策序列

**Path Length**
$$
\mathcal{P}_T(u_{1:T})\triangleq \sum_{t=2}^T\|u_t-u_{t-1}\|
$$

**Feasible Set** 
$$
\mathcal{X}_t^{\star}\triangleq \{x\in \mathcal{X}:\;g_t(x)\le 0\}
$$


这个也是凸的

Feasible Comparators：对于一个决策序列 $u_{1:T}$ 是feasible 的，当且仅当所有的 $u_t\in \mathcal{X}_t^\star$。

**Universal Dynamic Regret**
$$
\text{UD-Reg}(f_{1:T};u_{1:T})=\sum_{t=1}^T f_t(x_t)-\sum_{t=1}^T f_t(u_t)
$$

可以发现，当 Common Feasibility 假设成立时，存在 $x^\star \in \cap_{t=1}^T\mathcal{X}_t^{\star}$，选择 $u_t=x^\star$ 就得到了 static regret。

**Worst-case Dynamic Regret**
$$
\text{WD-Reg}(f_{1:T})=\sum_{t=1}^T f_t(x_t)-\sum_{t=1}^T \min_{x_t^\star \in \mathcal{X}_t^{\star}} f_t(x_t^\star)
$$

从 UDR 到 WDR，存在这样一个不等式：


$$
\sum_{t=1}^T f_t(x_t)\le
\underbrace{\sum_{t=1}^T f_t(u_t)}_{\text{term-I}}
+\underbrace{\psi_T(\mathcal{P}_T(u_{1:T}))}_{\text{term-II}}
$$
其中 $\psi_T(\cdot)$ 是 algorithm-dependent 的、随 comparator path length 单调不减的函数。

> 这里的 term-II 是怎么来的？为什么会有一个和 $\mathcal{P}_T(u_{1:T})$ 相关的东西？
>
> 因为 telescoping 的时候抵消不掉了（这就是 static 和 dynamic regret 的区别）
>
> 以 OGD 算法为例，大部分 OCO 算法的框架是这样的：
>
> 设决策集 $\mathcal{X}$ 直径为 $D$，即任意 $a,b\in \mathcal{X}$，$\|a-b\|\le D$。
>
> OGD 更新：$x_{t+1}=\mathrm{Proj}_\mathcal{X}(x_t-\eta g_t)$，其中 $g_t\in\partial f_t(x_t)$。往负梯度方向走一步之后投影。
>
> Step 1：先用凸性把函数差变成梯度内积（次梯度不等式）
>
> 对凸函数，根据次梯度性质
>
> $$
> f_t(x_t)-f_t(u_t)\le \langle g_t, x_t-u_t\rangle
> $$
>
> 于是
>
> $$
> \mathrm{UD\text{-}Reg}(f_{1:T};u_{1:T})
> =\sum_{t=1}^T\bigl(f_t(x_t)-f_t(u_t)\bigr)
> \le \sum_{t=1}^T \langle g_t, x_t-u_t\rangle
> \tag{1}
> $$
>
> Step 2：用更新规则把这个内积写成距离的差分以便 telescoping
>
> 利用投影的性质
> $$
> \|x_{t+1}-u_t\|^2
> \le \|x_t-\eta g_t-u_t\|^2
> = \|x_t-u_t\|^2+\eta^2\|g_t\|^2-2\eta\langle g_t,x_t-u_t\rangle
> $$
> 移项得到
> $$
> \langle g_t,x_t-u_t\rangle
> \le \frac{\|x_t-u_t\|^2-\|x_{t+1}-u_t\|^2}{2\eta}
> +\frac{\eta}{2}\|g_t\|^2
> \tag{2}
> $$
> 把 (2) 代入 (1)，并对 $t=1,\dots,T$ 求和：
> $$
> \mathrm{UD\text{-}Reg}(f_{1:T};u_{1:T})
> \le
> \underbrace{\sum_{t=1}^T\frac{\|x_t-u_t\|^2-\|x_{t+1}-u_t\|^2}{2\eta}}_{\text{差分项}}
> +\frac{\eta}{2}\sum_{t=1}^T\|g_t\|^2
> \tag{3}
> $$
> Step 3：如果 comparator 是固定的，telescoping 的时候可以抵消，得到经典的 static regret
>
> $$
> &\sum_{t=1}^T\Big(\|x_t-u_t\|^2-\|x_{t+1}-u_t\|^2\Big)\\
> =&
> \|x_1-u_1\|^2-\|x_{T+1}-u_T\|^2
> +\sum_{t=1}^{T-1}\Big(\|x_{t+1}-u_{t+1}\|^2-\|x_{t+1}-u_t\|^2\Big)
> \tag{4}
> $$
>
> 现在令 $\Delta_t \triangleq \|x_{t+1}-u_{t+1}\|^2-\|x_{t+1}-u_t\|^2$，这个东西消不掉了
>
> $$
> \Delta_t
> = \| (x_{t+1}-u_t) - (u_{t+1}-u_t)\|^2 - \|x_{t+1}-u_t\|^2
> = \|u_{t+1}-u_t\|^2 - 2\langle x_{t+1}-u_t, u_{t+1}-u_t\rangle
> $$
> 因此
> $$
> \Delta_t \le \|u_{t+1}-u_t\|^2 + 2\|x_{t+1}-u_t\|\|u_{t+1}-u_t\|
> \le \|u_{t+1}-u_t\|^2 + 2D\|u_{t+1}-u_t\|
> \tag{5}
> $$
> 把 (5) 代回 (4) 和 (3)，并用 $\|x_1-u_1\|\le D$ 和 $-\|x_{T+1}-u_T\|^2\le0$，得到一个典型形态
>$$
> \mathrm{UD\text{-}Reg}(f_{1:T};u_{1:T})\le
> \frac{D^2}{2\eta}
> +\frac{1}{2\eta}\sum_{t=1}^{T-1}\Big(\|u_{t+1}-u_t\|^2+2D\|u_{t+1}-u_t\|\Big)
> +\frac{\eta}{2}\sum_{t=1}^T\|g_t\|^2
> $$
> 再用 $\|u_{t+1}-u_t\|\le D$ 之类的放缩一下，就很容易得到 $\sum\|u_{t+1}-u_t\|$ 这样的东西了。



## surrogate cost function

auxiliary cost function 

$$
\tilde{f}_t(x)=f_t(x)+2G \mathrm{dist}(x,\mathcal{X}_t^\star)
$$

注意这里这个 $\mathrm{dist}(x,\mathcal{X}_t^\star)$ 的部分，这个是有必要的，后面会多次用到相关的性质。

> 后面作者专门写了一个 Remark 16，说单纯用 $f_t+g_t^+(x)$ 去构造 surrogate cost function 在理论上是不成立的，并且构造了一个简单的反例，直观地来说就是让 surrogate 的最小点在可行域的外面，并且让约束函数 $g$ 的梯度太小而被 $f$ 所支配。为了避免这种情况，他们才设计了 $2G \mathrm{dist}(x,\mathcal{X}_t^\star)$ 这一项来放大惩罚。

Lemma 11
$$
\sum_{t=1}^T(f_t(x_t)-f_t(u_t))\le \sum_{t=1}^T(\tilde{f}_t(x_t)-\tilde{f}_t(u_t))
$$
为什么？因为 $\tilde{f}_t(x)\ge f_t(x)$，而 $u_t$ 是可行解，所以 $\mathrm{dist}(x,\mathcal{X}_t^\star)=0$

Lemma 12
$$
\mathrm{arg}\min_{x\in \mathcal{X}} \tilde{f}_t(x)\in \mathcal{X}_t^\star, \quad t\ge 1
$$
这个引理用反证法证的，后面会用到。之前的 $\tilde{f}_t(\cdot)$ 里面的距离在这里就起到用处了。

然后就是重点，构造 surrogate cost function $\hat{f}_t(x)=g_t^+(x)+\tilde{f}_t(x)$，这是 $4G$-Lipschitz 的，Lipschitz 性质在后面也有用。

## 投影算法

接下来 Theorem 13 给了一个 ADER 算法的 UD-Regret 的 bound。

Theorem 14 证明了 UDR 和 CCV 的 bound。这里有一些关键的性质需要提一下
$$
u_t=x_t^\star &=& \mathrm{arg}\min_{x\in \mathcal{X}} \tilde{f}_t(x)\\
&=& \mathrm{arg}\min_{x\in \mathcal{X}_t^\star} \tilde{f}_t(x) \\
&=& \mathrm{arg}\min_{x\in \mathcal{X}_t^\star} f_t(x)
$$
第一步就是 Lemma 12，第二步是因为对于可行解 $x\in \mathcal{X}_t^\star$ 来说 $\tilde{f}_t(x)=f_t(x)$。

然后还有一个看起来有点奇怪的 $\zeta=\mathcal{O}(\log \log T)$，这个东西在后面的 bound 里面也出现了，似乎是调用的那些 OCO 算法产生的，不是本文的重点，先略过了。

然后提了一下可以扩展到约束函数是 quasi-convex 的情况，定义一个 $h_t(x)=G \mathrm{dist}(x,\mathcal{X}_t^\star)$ 可以转化过去。

## 一阶算法

之后就是说设计一个 projection-free 的一阶算法，提升效率。

Theorem 18 是 Dynamic Regret Bounds for *AdaGrad*。

Theorem 19 是 universal dynamic regret bound for *AdaHedge* with *AdaGrad*（AHAG）。

然后从这个 AHAG 过渡到第二个算法，AHAG 已经是一阶算法了，只需要每一轮去算那些梯度。

这里他们先构造 Lyapunov function $\Phi(x)=x^2$，然后设计了一个新的 surrogate cost function $\hat{f}_t(x)=Vf_t(x)+\Phi'(Q(t))g_t^+(x)$，这一个构造很巧妙：之后可以把 CCV 和 Regret 整合起来，一起 bound regret 和 violation。

这里有两个关键的式子：
$$
\Phi(Q(t))-\Phi(Q(t-1))\le \Phi'(Q(t))g_t^+(x_t)
$$
这个平方以后展开是显然的。

还有一个是：
$$
\Phi(Q(t))+V\sum_{t=1}^T\left( f_t(x_t)-f_t(u_t)\right)\le 
\sum_{t=1}^T\Phi'(Q(t))g_t^+(x_t)+V\sum_{t=1}^T\left( f_t(x_t)-f_t(u_t)\right)
$$
这其实就是 telescoping 一下。之后把这个 RHS 继续放缩，就得到了 $\sum_{t=1}^T\left( \hat{f}_t(x_t)-\hat{f}_t(u_t)\right)$，有
$$
\Phi(Q(t))+V\;\text{UD-Reg}(f_{1:T};u_{1:T})\le \text{UD-Reg}(\hat{f}_{1:T};u_{1:T})
$$
主要构造的就是这些，后面就是利用各种性质进行放缩了。











