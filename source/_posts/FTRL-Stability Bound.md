---
title: FTRL-Stability Bound
date: 2026-02-09
tag: [Online Learning]
category: [Mathematical Foundation]
math: true
---

强凸性，Lipschitz性质，Hölder不等式

<!--more-->



# 基础概念

## strong convexity

说函数 $f(\cdot)$ 是凸的，意味着任意两点连线上的点不低于函数值：
$$
f(\lambda x+(1-\lambda) y)\le \lambda f(x)+(1-\lambda) f(y),\quad \lambda \in [0,1]
$$
强凸函数描述的是函数凸的程度，例如说函数 $f(\cdot)$ 在某个范数 $\|\cdot\|$ 下是 $\alpha$-强凸的，意思是说
$$
f(\boldsymbol{y})\le f(\boldsymbol{x})+\nabla f(\boldsymbol{x})^\top(\boldsymbol{y}-\boldsymbol{x}) + \frac{\alpha}{2} \|\boldsymbol{y}-\boldsymbol{x}\|^2
$$

后面先咕了
