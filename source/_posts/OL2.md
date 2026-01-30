---
title: Pushback Lemma
date: 2026-01-26
tag: [online learning]
category: [Mathematical Foundation]
math: true
---

一个常见的 Lemma，源于强凸性性质，并保证优化过程固有地抵抗偏离最优解

<!--more-->

# Pushback lemma

令 $\Pi$ 为凸集，$h$ 在 $\Pi$ 上凸，$\pi_{\text{opt}}$ 是下面正则化问题的全局最小解：
$$
\pi_{\text{opt}} \in \arg\min_{\pi\in\Pi}\; h(\pi)+\alpha D(\pi\|\pi_{t-1}),
$$
则对任意 $\pi\in\Pi$，有
$$
h(\pi_{\text{opt}})+\alpha D(\pi_{\text{opt}}\|\pi_{t-1})
\;\le\;
h(\pi)+\alpha D(\pi\|\pi_{t-1})-\alpha D(\pi\|\pi_{\text{opt}}).
$$
这就是“pushback”：任何别的点 $\pi$ 的目标值，比最优点 $\pi_{\text{opt}}$ 至少**大**了 $\alpha  D(\pi\|\pi_{\text{opt}})$ 这么多。

## 基础定义

### Bregman divergence

three-point expansion 本质上是 **Bregman divergence 的一个代数恒等式**。

给定可微严格凸函数 $\Phi$，Bregman divergence 定义为
$$
D_\Phi(u,v) \triangleq \Phi(u)-\Phi(v)-\langle \nabla\Phi(v),u-v\rangle.
$$

这就是 $\Phi(u)$ 与 $\Phi$ 在 $v$ 处切线近似的差，因为 $\Phi(v)+\langle\nabla\Phi(v),u-v\rangle$ 正是 $\Phi$ 在 $v$ 处的**一阶泰勒近似**。
由于 $\Phi$ 凸，函数图像永远在切线之上，所以这个差永远非负：
$$
D_\Phi(u,v)\ge 0,\quad =0 \iff u=v.
$$
因为 Bregman divergence 衡量的是**“用 $v$ 的局部线性模型去近似 $u$ 的误差”**，这天然是**有方向的**（用谁的切线去近似谁）。

### three-point expansion 恒等式

对任意三点 $u,v,w$，有恒等式：
$$
D_\Phi(u,w) - D_\Phi(v,w) - D_\Phi(u,v)
= \langle \nabla\Phi(v)-\nabla\Phi(w),u-v\rangle.
\tag{TP}
$$
这就是为什么叫“三点”：同一个式子里同时出现了 $u,v,w$ 三个点。

#### 用定义证明

把三项按定义写出来：

$$
\begin{aligned}
D_\Phi(u,w) &= \Phi(u)-\Phi(w)-\langle \nabla\Phi(w),u-w\rangle,\\
D_\Phi(v,w) &= \Phi(v)-\Phi(w)-\langle \nabla\Phi(w),v-w\rangle,\\
D_\Phi(u,v) &= \Phi(u)-\Phi(v)-\langle \nabla\Phi(v),u-v\rangle.
\end{aligned}
$$
现在算左边：

$$
\begin{aligned}
& D_\Phi(u,w) - D_\Phi(v,w) - D_\Phi(u,v)\\
=&\Big(\Phi(u)-\Phi(w)-\langle \nabla\Phi(w),u-w\rangle\Big)
-\Big(\Phi(v)-\Phi(w)-\langle \nabla\Phi(w),v-w\rangle\Big)\\
&\quad-\Big(\Phi(u)-\Phi(v)-\langle \nabla\Phi(v),u-v\rangle\Big).
\end{aligned}
$$
右边的 $\Phi(\cdot)$ 项全都抵消掉，剩下只有内积项：

$$
\begin{aligned}
=&-\langle \nabla\Phi(w),u-w\rangle + \langle \nabla\Phi(w),v-w\rangle + \langle \nabla\Phi(v),u-v\rangle\\
=&\langle \nabla\Phi(w), (v-w)-(u-w)\rangle + \langle \nabla\Phi(v),u-v\rangle\\
=&\langle \nabla\Phi(w), v-u\rangle + \langle \nabla\Phi(v),u-v\rangle\\
=&\langle \nabla\Phi(v)-\nabla\Phi(w),u-v\rangle.
\end{aligned}
$$
证毕。



## 证明

先写一阶最优性，再用 three-point expansion，最后利用凸性。

### Step 0：定义总目标

令
$$
F(\pi)=h(\pi)+\alpha D(\pi\|\pi_{t-1}).
$$
$\pi_{\text{opt}}$ 是 $F$ 在 $\Pi$ 上的最小值点。

### Step 1：最优性条件（KKT / 一阶条件）

若 $\pi_{\text{opt}}$ 在可行域“内部”，则有
$$
\nabla h(\pi_{\text{opt}})+\alpha \nabla_\pi D(\pi_{\text{opt}}\|\pi_{t-1})=0.
$$

更一般地，即便在边界，也可以写成 $\nabla F(\pi_{\text{opt}})$ 落在 normal cone 里，结论仍成立；这里只是用最直观的“等于 0”版本。

### Step 2：Bregman 的 three-point expansion

对任意 Bregman divergence，利用 three-point expansion，有： 
$$
D(\pi\|\pi_{t-1})-D(\pi_{\text{opt}}\|\pi_{t-1})=
D(\pi\|\pi_{\text{opt}})
+\left\langle\nabla D(\pi_{\text{opt}}\|\pi_{t-1}),\pi-\pi_{\text{opt}}\right\rangle.
$$
KL 散度是负熵函数的 Bregman 散度，一样的。

### Step 3：把差值 $F(\pi)-F(\pi_{\text{opt}})$ 拆开

$$
\begin{aligned}
&F(\pi)-F(\pi_{\text{opt}})\\
=&
\big(h(\pi)-h(\pi_{\text{opt}})\big)
+\alpha\big(D(\pi|\pi_{t-1})-D(\pi_{\text{opt}}|\pi_{t-1})\big)\\
=&
\big(h(\pi)-h(\pi_{\text{opt}})\big)
+\alpha D(\pi|\pi_{\text{opt}})
+\alpha\left\langle\nabla D(\pi_{\text{opt}}|\pi_{t-1}),\pi-\pi_{\text{opt}}\right\rangle.
\end{aligned}
$$

现在把 $h(\pi)-h(\pi_{\text{opt}})$ 也按“凸函数一阶下界”拆：
$$
h(\pi)-h(\pi_{\text{opt}})=
\underbrace{\big(h(\pi)-h(\pi_{\text{opt}})-\langle\nabla h(\pi_{\text{opt}}),\pi-\pi_{\text{opt}}\rangle\big)}_{\ge 0\;\text{(凸性)}}
+\langle\nabla h(\pi_{\text{opt}}),\pi-\pi_{\text{opt}}\rangle.
$$

于是 

$$
F(\pi)-F(\pi_{\text{opt}})=
\underbrace{\big(h(\pi)-h(\pi_{\text{opt}})-\langle\nabla h(\pi_{\text{opt}}),\pi-\pi_{\text{opt}}\rangle\big)}_{\ge 0}\\
+\alpha D(\pi|\pi_{\text{opt}})
+\left\langle \nabla h(\pi_{\text{opt}})+\alpha\nabla D(\pi_{\text{opt}}|\pi_{t-1}),\pi-\pi_{\text{opt}}\right\rangle.
$$

### Step 4：用 Step 1 把最后一项消掉 + 用凸性给下界

由最优性条件 $\nabla h(\pi_{\text{opt}})+\alpha\nabla D(\pi_{\text{opt}}|\pi_{t-1})=0$，最后那个内积项为 0。因此：
$$
F(\pi)-F(\pi_{\text{opt}})\ge \alpha D(\pi|\pi_{\text{opt}}).
$$
即
$$
F(\pi_{\text{opt}})\le F(\pi)-\alpha D(\pi|\pi_{\text{opt}}).
$$
