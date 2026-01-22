---
title: Universal Dynamic Regret and Constraint Violation Bounds for Constrained Online Convex Optimization
date: 2026-01-21
tags: [bandits]
categories: [Literature Review]
math: true
---



读一个 OCO 的工作，未完待续。



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

从 UDR 到 WDR，这一步是怎么来的？看起来像是对 Universal Dynamic Regret 进行了移项和放缩之后得到的：




$$
\sum_{t=1}^T f_t(x_t)\le
\underbrace{\sum_{t=1}^T f_t(u_t)}_{\text{term-I}}
+\underbrace{\psi_T(\mathcal{P}_T(u_{1:T}))}_{\text{term-II}},
$$
其中 $\psi_T(\cdot)$ 是“algorithm-dependent 的、随 comparator path length 单调不减的函数”。

这其实是说，你选的在线算法本身就有一个 universal dynamic regret bound，即对任意可行 comparator 序列 $u_{1:T}$ 都成立的上界 $\mathrm{UD\text{-}Reg}(f_{1:T};u_{1:T})\le \psi_T(\mathcal{P}_T(u_{1:T}))$



那么问题来了，为什么会有这个保证？



为什么动态 regret 的上界会“必然”出现 $P_T(u_{1:T})$：关键在于望远镜求和被 comparator 的移动打断

几乎所有在线一阶算法（OGD / Mirror Descent / AdaGrad / FTRL 变体）的 regret 分析都有同一个骨架：

- 每一步先用凸性把函数差变成梯度内积；
- 再用更新规则把这个内积写成**距离的差分**（这一步制造出“望远镜相消”）；
- 如果 comparator 是固定的，差分项基本都相消，得到静态 regret；
- 如果 comparator 是 (u_t) 在动，相消被“对不齐”打断，残差正好由 $|u_{t+1}-u_t|$ 控制，累加起来就是 $P_T(u_{1:T})=\sum_{t\ge2}|u_t-u_{t-1}|$。

以 OGD 算法为例，可以推导出这样的结果。

> 设决策集 $\mathcal{X}$ 直径为 $D$，即任意 $a,b\in \mathcal{X}$，$\|a-b\|\le D$。
>
> OGD 更新：$x_{t+1}=\mathrm{Proj}_\mathcal{X}(x_t-\eta g_t)$，其中 $g_t\in\partial f_t(x_t)$。往负梯度方向走一步之后投影。
>
> Step 1：凸性把函数差变成内积（次梯度不等式）
>
> 对凸函数，根据次梯度性质
> $$
> f_t(x_t)-f_t(u_t)\le \langle g_t, x_t-u_t\rangle.
> $$
> 于是
> $$
> \mathrm{UD\text{-}Reg}(f_{1:T};u_{1:T})
> =\sum_{t=1}^T\bigl(f_t(x_t)-f_t(u_t)\bigr)
> \le \sum_{t=1}^T \langle g_t, x_t-u_t\rangle.
> \tag{S1}
> $$
>
> Step 2：用投影更新把内积写成“距离差分 + 梯度范数项”
>
> 投影的非扩张性给出（标准不等式）：
> $$
> |x_{t+1}-u_t|^2
> \le |x_t-\eta g_t-u_t|^2
> = |x_t-u_t|^2+\eta^2|g_t|^2-2\eta\langle g_t,x_t-u_t\rangle.
> $$
> 移项得到
> $$
> \langle g_t,x_t-u_t\rangle
> \le \frac{|x_t-u_t|^2-|x_{t+1}-u_t|^2}{2\eta}
> +\frac{\eta}{2}|g_t|^2.
> \tag{S2}
> $$
> 把 (S2) 代入 (S1)，并对 $t=1,\dots,T$ 求和：
> $$
> \mathrm{UD\text{-}REGRET}
> \le
> \underbrace{\sum_{t=1}^T\frac{|x_t-u_t|^2-|x_{t+1}-u_t|^2}{2\eta}}_{\text{差分项}}
> +\frac{\eta}{2}\sum_{t=1}^T|g_t|^2.
> \tag{S3}
> $$
> Step 3：关键点——差分项无法完全望远镜相消
>
> 把差分项拆开： 
> $$
> \sum_{t=1}^T\Big(|x_t-u_t|^2-|x_{t+1}-u_t|^2\Big)
> |x_1-u_1|^2-|x_{T+1}-u_T|^2
> +\sum_{t=1}^{T-1}\Big(|x_{t+1}-u_{t+1}|^2-|x_{t+1}-u_t|^2\Big).
> \tag{S4}
> $$
>
> - 前两项是边界项，不是问题。
> - 真正新出现的，就是最后那串“同一个 $x_{t+1}$ 配两个不同的 comparator：$u_{t}$ vs $u_{t+1}$”的差：
>
> $$
> \Delta_t \triangleq |x_{t+1}-u_{t+1}|^2-|x_{t+1}-u_t|^2.
> $$
>
> 展开：
> $$
> \Delta_t
> = | (x_{t+1}-u_t) - (u_{t+1}-u_t)|^2 - |x_{t+1}-u_t|^2
> = |u_{t+1}-u_t|^2 - 2\langle x_{t+1}-u_t,; u_{t+1}-u_t\rangle.
> $$
> 因此
> $$
> \Delta_t \le |u_{t+1}-u_t|^2 + 2|x_{t+1}-u_t|,|u_{t+1}-u_t|
> \le |u_{t+1}-u_t|^2 + 2D|u_{t+1}-u_t|,
> \tag{S5}
> $$
> 其中最后一步用的是 $|x_{t+1}-u_t|\le D$。
>
> 把 (S5) 代回 (S4)(S3)，并用 $|x_1-u_1|\le D$、$-|x_{T+1}-u_T|^2\le0$，得到一个典型形态：
> $$
> \mathrm{UD\text{-}Reg}(f_{1:T};u_{1:T})\le
> \frac{D^2}{2\eta}
> +\frac{1}{2\eta}\sum_{t=1}^{T-1}\Big(|u_{t+1}-u_t|^2+2D|u_{t+1}-u_t|\Big)
> +\frac{\eta}{2}\sum_{t=1}^T|g_t|^2.
> \tag{S6}
> $$
> 现在可以看到，路径来自 (S4) 里“望远镜相消对不齐”的差，最终变成 $\sum|u_{t+1}-u_t|$。
>
> 进一步，把 $\sum|u_{t+1}-u_t|$ 识别为 $P_T(u_{1:T})$，并用 $|u_{t+1}-u_t|\le D$ 使平方项也被线性项吸收（例如 $|d|^2\le D|d|$，就得到更干净的形式：
> $$
> \mathrm{UD\text{-}REGRET}\le
> \frac{D^2}{2\eta}
> +\underbrace{\frac{cD}{\eta}P_T(u_{1:T})}_{\text{tracking 代价}}
> +\frac{\eta}{2}\sum_{t=1}^T|g_t|^2,
> \tag{S7}
> $$
> 其中 (c) 是常数（从上面的系数合并得到）。
>
> 为什么 regret 会放缩成 path length 的形式：因为动态 comparator 让望远镜求和留下的残差，恰好由每一步 comparator 的移动量 $|u_{t+1}-u_t|$ 控制。
>
> 



auxiliary cost function 
$$
\tilde{f}_t(x)=f_t(x)+2G \mathrm{dist}(x,\mathcal{X}_t^\star)
$$
为什么要这样定义，后面就会知道



Lemma 11
$$
\sum_{t=1}^T(f_t(x_t)-f_t(u_t))\le \sum_{t=1}^T(\tilde{f}_t(x_t)-\tilde{f}_t(u_t))
$$
为什么？因为 $\tilde{f}_t(x)\ge f_t(x)$，而 $u_t$ 是可行解，所以 $\mathrm{dist}(x,\mathcal{X}_t^\star)=0$

然后反证法证明，不重要。

然后构造 $\hat{f}_t(x)=g_t^+(x)+\tilde{f}_t(x)$，这是 $4G$-Lip 的

这里给了一个 ADER 算法，

(Universal Dynamic Regret and CCV for Algorithm 2) 

用这个 ADER 算法去做之前的 OCO 问题，会得到一个新的 bound
