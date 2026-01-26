---
title: Triple-Optimistic Learning for Stochastic Contextual Bandits with General Constraints
date: 2026-01-25
tags: [bandits]
categories: [Literature Review]
math: true
---



未完待续



<!--more-->

# 问题模型

问题由 $\{ \mathcal X,\mathcal A,\mathcal F,\mathcal G\}$ 描述：上下文集合 $\mathcal X$、有限动作集 $\mathcal A$、奖励函数类 $\mathcal F$、成本函数类 $\mathcal G$。

第 $t$ 轮：观察 $x_t \sim P(\cdot)$（i.i.d.，且论文假设 $P$ 已知），选动作 $a_t\in\mathcal A$，得到随机奖励 $r_t(x_t,a_t)\in[0,1]$ 与随机成本 $c_t(x_t,a_t)\in[-1,1]$。

$$
\Delta(\mathcal A)
=\Big\{ p\in\mathbb R^{|\mathcal A|}\; :\ p_a\ge 0\; \forall a\in\mathcal A,\; \sum_{a\in\mathcal A} p_a = 1 \Big\}
$$
问题建模
$$
\max_\pi \sum_{t=1}^T r_t(x_t,a_t)\\ 
\text{s.t.}\quad \sum_{t=1}^T c_t(x_t,a_t)\le 0
$$
然后谈了一下 BwK 问题和公平性约束问题的转化，大体上和 [Combinatorial Bandits with Linear Constraints Beyond Knapsacks and Fairness](https://phlzy.github.io/2026/01/04/NeurIPS-2022-Combinatorial_Bandits_with_Linear_Constraints_Beyond_Knapsacks_and_Fairness/) 里面讲的是一样的。

Regret 的定义还是 pseudo regret，依赖上下文 $x$ 的静态最优：
$$
\pi^* \triangleq \mathrm{arg}\max_\pi \mathbb E[\langle \pi, f(x)\rangle]\quad 
\text{s.t.}\ \mathbb E[\langle \pi, g(x)\rangle]\le 0
$$
最优值
$$
\nu^*=\mathbb E[\langle\pi^*,f(x)\rangle]
$$
pseudo regret
$$
\mathcal{R}(T)=T\nu^*-\mathbb E\bigg[\sum_{t=1}^T r_t(x_t,a_t)\bigg]
$$
constraint violation
$$
\mathcal{V}(T)=\mathbb E\bigg[\sum_{t=1}^T c_t(x_t,a_t)\bigg]
$$

## assumption

存在两个在线回归 oracle，分别用于 reward 和 cost，它们在每一轮输出估计函数 $\hat f_t(x,a)$ 和 $\check g_t(x,a)$，

以至少 $1-p$ 的概率，同时满足下面两个单边置信界：
$$
0 \le \hat f_t(x,a)-f(x,a)\le 2\varepsilon_t(x,a,p)
$$
$$
0 \le g(x,a)-\check g_t(x,a)\le 2\varepsilon_t(x,a,p)
$$
其中 $p=1/T^2$，还有
$$
\sum_{t=1}^T \|\varepsilon_t(x,a,p)\|_2^2 = O\bigg(\log\Big(\frac{\max(|\mathcal F|,|\mathcal G|)}{p}\Big)\bigg)\tag{\#}
$$


这个东西的用处会在后面的构造中体现

# 主要思想

## optimistic surrogate function

构造
$$
\hat L(x_t,a)=\hat f_t(x_t,a)-Q_t \check g_t(x_t,a)
$$


然后求解带 KL 正则的策略更新
$$
\pi_t=\arg\max_\pi \langle \pi,\hat L(x_t)\rangle-\alpha D(\pi\|\pi_{t-1})
$$


更新
$$
Q_{t+1}=\Big(Q_t-\mathbb{E}\big[\langle\pi_{t-1},\check g_{t-1}(x)\rangle-2\langle\pi_t,\check g_t(x)\rangle\big]\Big)^+
$$

## Regret plus Drift

首先，想办法用一个不等式把 Regret 和 Violation 统一起来，这里先使用 primal-dual / Lagrangian，把约束去掉。这里构造的 $Q(t)$ 就像是 Lagrangian 函数里面那个 $\lambda$，超过约束了就变大，满足约束就变小。

为什么这里的 $q_t$ 是 shift dual variable？而不是直接用 $Q_t$？

这里的 primal 变量是策略 $\pi$，dual 变量是 $Q_t$，但是因为这里的队列构造是特殊的，所以改写成 $q_t$ 进行计算

定义：
$$
q_t = Q_t-\mathbb E_x[\langle \pi_{t-1},\check g_{t-1}(x)\rangle]
$$
因为传统的队列更新是只包含 $\langle\pi_t,\check g_t(x)\rangle$ 的，现在因为他们引入了 $t-1$ 时刻的项用于 smooth the update and accelerate convergence，直接去 drift，展开会出现 $\langle \pi_{t-1},\check g_{t-1}(x)\rangle$ 和 $\langle \pi_{t},\check g_{t}(x)\rangle$ 的积，这就消不掉了，而 $q_t$ 就避免了这个问题。

> 从而更好地预测 violation（避免过于激进），这被认为是能去掉 Slater 依赖的关键之一。

一旦更新里出现 (\pi_{t-1},\pi_t) 两步的量，直接用 (Q_t) 做 Lyapunov drift 会得到很难处理的交叉项。用 (q_t) 做平移后，drift 的代数展开会“对齐”这两个量，让后面 Lemma 5.6 / 5.7 可以把交叉项压住

然后利用 Lyapunov 思想，定义这个 drift：
$$
\Delta_t=\frac12(q_{t+1}^2-q_t^2) \quad\text{}
$$

能在求和时望远镜消掉，只剩末项 $q_{T+1}^2$







然后证明单步上：
$$
\underbrace{\langle \pi^*-\pi_t,f(x_t)\rangle}_{\text{单步 regret}}
\underbrace{\Delta_t}_{\text{约束势能变化}}
\le \text{(学习误差 + 镜像下降项 + 若干可求和项)}
$$
这就是 Lemma 5.3 的核心作用：**把 regret 和“约束压力增长”锁在同一个不等式里**。求和后，左边第一项给你总 regret，第二项望远镜成 (q_{T+1}^2)，再由 (q_{T+1}) 控 violation（见 5.3 的思路）。



## Lemma 5.3

### Lemma 5.5 Pushback lemma

看了半天终于会证明了



### Lemma 5.6 Drift lemma

为什么 drift 会和 $Q_t \mathbb E[\langle\pi_t,\check g_t\rangle]$ 联系在一起？

他们定义 $\Delta_t=\frac12(q_{t+1}^2-q_t^2)$。平方 drift 展开时自然出现线性项 $q_t\cdot(\text{更新增量})$ 和二次项 ((\text{更新增量})^2)。Lemma 5.6 就是在形式上把它写成：
$$
\mathbb E_x[\Delta_t]\le q_t\mathbb E_x[\langle\pi_t,\check g_t(x)\rangle]+
\mathbb E_x[\langle\pi_t,\check g_t(x)\rangle]^2
$$

再把 $q_t=Q_t-\mathbb E_x[\langle\pi_{t-1},\check g_{t-1}\rangle]$ 代回去，就会出现一个麻烦的交叉项

$$
\mathbb E_x[\langle\pi_{t-1},\check g_{t-1}\rangle]\mathbb E_x[\langle\pi_t,\check g_t\rangle]
$$

### Lemma 5.7

处理交叉项：那个交叉项是“两个不同时刻的期望的乘积”，很难直接求和。作者用一个不等式把它下界成“两个平方的平均”减去“变化量惩罚”：
$$
\mathbb E_x[\langle\pi_{t-1},\check g_{t-1}\rangle]\mathbb E_x[\langle\pi_t,\check g_t\rangle]
\ge
\frac12\mathbb E_x[\langle\pi_t,\check g_t\rangle]^2
+\frac12\mathbb E_x[\langle\pi_{t-1},\check g_{t-1}\rangle]^2
-\mathbb E_x|\check g_{t-1}-\check g_t|_2^2
-|\pi_t-\pi_{t-1}|_1^2
$$


直觉是：

- 如果相邻两步的成本估计 $\check g_t$ 变化不大、策略 $\pi_t$ 变化不大，那么交叉项就接近“平方项”，drift 会很好地被控制；
- 代价就是额外出现两项：$|\check g_{t-1}-\check g_t|_2^2$ 与 $|\pi_t-\pi_{t-1}|_1^2$。

而这两项后面都能被上界并求和：

- $\check g_t$ 来自在线回归 oracle，整体变化量可控（在 5.2 里被做成 $\tilde O(1)$ 的和）；
- $|\pi_t-\pi_{t-1}|_1^2$ 则用 Pinsker 把它变成 KL（下一条）。

### Lemma 5.9 Pinsker

为什么要它？为什么要求 $\alpha\ge2$？

Lemma 5.7 留下了 $|\pi_t-\pi_{t-1}|_1^2$。而你的 primal 更新里本来就有 KL 正则项 $D(\pi_t\Vert\pi_{t-1})$。Pinsker 提供：
$$
|\pi_t-\pi_{t-1}|_1^2 \le 2D(\pi_t\Vert\pi_{t-1})
$$

所以只要 $\alpha$ 够大（文中取 $\alpha\ge2$），就能让 Lemma 5.7 带来的 $|\pi_t-\pi_{t-1}|_1^2$ 被 KL 正则“支付掉”，使得最终单步不等式干净闭合。

------

为什么他们要在对偶更新里用 $\mathbb E_x[\cdot]$（假设已知 context 分布）？

这是 5.1 里一个非常关键、也最容易忽略的“工程性”决定。

如果不知道分布，只能用样本 (x_t) 做更新，会出现类似
$|\check g_{t-1}(x_{t-1})-\check g_t(x_t)|^2$
这样的项，它反映的是“不同 context 之间的跳变”，在 contextual setting 下分析会非常难。作者在 Remark 5.8 里明确说：为了绕开这个分析障碍，他们在对偶更新里使用对 context 的期望（并假设分布已知）；否则会引入难以处理的 context-dependent variation 项。

------

把 5.1 的结论用一句话总结：它到底“帮了什么忙”？

- Lemma 5.3 给出单步：
$$
\text{(regret)}+\text{(drift)} \le \text{可控误差项} + \text{可望远镜项}
$$
- 求和后：
  - regret 的和就是总 regret；
  - drift 的和望远镜成 $\frac12 q_{T+1}^2$，这就把“约束压力”压成一个末项；
  - KL 的差分也望远镜；
  - 剩下的学习误差用 oracle 假设给出 $\tilde O(\sqrt T)$。

于是后面 5.2 用它推出 regret；5.3 再用 $q_{T+1}$ 上界累计 violation









