---
title: Matrix Operation
date: 2022-07-27
tag: [linear algebra]
category: [Mathematical Foundation]
math: true
---

矩阵的基本运算

<!--more-->

矩阵的加法和标量乘法很简单，就不多说了。

## 矩阵乘法

来看一个线性变换：对于 $\mathbb{R}^k$ 中的一个向量 $\boldsymbol{v}$，用一个 $n\times k$ 的矩阵 $B$ 乘上这个 $\boldsymbol{v}$，就会得到一个 $\mathbb{R}^n$ 中的向量 $B\boldsymbol{v}$。容易发现，向量的维度发生了改变。所以如果此时我们再需要对其进行一次变换，必须用一个 $n$ 列的矩阵。假设用一个 $m\times n$ 矩阵 $A$ 乘上新向量 $B\boldsymbol{v}$，就得到了向量 $A(B\boldsymbol{v})$。

由于线性变换的标准矩阵是唯一的，所以存在一个矩阵 $AB$ 满足 $(AB)\boldsymbol{v}=A(B\boldsymbol{v})$。矩阵乘法的概念就是这么来的。

对于一个 $n\times k$ 的矩阵 $B$，可以将其每一列看作一个 $n$ 维向量，即 $B=[\boldsymbol{b_1},\boldsymbol{b_2},\cdots ,\boldsymbol{b_k}]$。那么矩阵 $AB$ 就应该是 $[A\boldsymbol{b_1},A\boldsymbol{b_2},\cdots ,A\boldsymbol{b_k}]$，其实就相当于拆成矩阵和向量的乘积以后再拼回去。显然，$A$ 的列数必须等于 $B$ 的行数。

这也就很好理解为什么两个矩阵相乘必须是前者的每一行乘上后者的每一列。

结合矩阵加法和标量乘法的性质，容易发现矩阵的乘法满足结合律和分配律，但是不满足交换律。

定义矩阵的主对角线为所有行列下标相等的元素，单位矩阵 $I_n$ 为主对角线元素均为 $1$、其余元素均为 $0$ 的 $n\times n$ 矩阵。容易证明，$I_mA=A=AI_n$。

值得注意的是，即便 $AB=AC$ 且 $A$ 不为零矩阵，也不能推出 $B=C$。

## 转置

对于 $m\times n$ 矩阵 $A$，它的转置 $A^T$ 是一个 $n\times m$ 矩阵，它的列由 $A$ 的对应行构成。

显然，矩阵的转置有以下的性质：

- $(A^T)^T=A$
- $(A+B)^T=A^T+B^T$
- $(rA)^T=rA^T$（$r$ 为常数）
- $(AB)^T=B^TA^T$

前三个很显然。下面证明第四个性质：对于 $(AB)^T$ 中位于 $(i,j)$ 的元素，在 $AB$ 中位于 $(j,i)$，等于 $\sum_{t=1}^n a_{jt}b_{ti}$。而 $B^T$ 的第 $i$ 行，就是 $B$ 的第 $i$ 列，即 $b_{1i},b_{2i},\cdots,b_{ni}$；$A^T$ 的第 $j$ 列，就是 $A$ 的第 $j$ 行，即 $a_{j1},a_{j2},\cdots,a_{jn}$，对应位置相乘后求和就是 $\sum_{t=1}^n a_{jt}b_{ti}$。

事实上，有

$$
(\prod_{i=1}^n A_i)^T=\prod_{i=n}^1 A_i^T
$$

这可以通过数学归纳法来证明。

## 矩阵的逆

定义了加减乘三种运算后，我们不禁会想，能不能给矩阵也定义一个除法？

参考离散数学里逆元的概念，所谓除法，就相当于对于一个给定的元素 $A$，求 $A$ 的逆元 $A^{-1}$，满足 $A^{-1}\times A=A\times A^{-1}=\epsilon$。对于矩阵而言，幺元 $\epsilon$ 就是单位矩阵。由于 $A^{-1}\times A$ 和 $A\times A^{-1}$ 都必须有意义，$A$ 必须是一个方阵。我们称 $A^{-1}$ 为 $A$ 的逆矩阵。

由此可见，并非所有的矩阵都有逆矩阵。如果一个矩阵有逆矩阵，就称其为可逆矩阵，或者**非奇异矩阵**；否则称其为不可逆矩阵，或**奇异矩阵**。

可逆矩阵也具有几个基本性质：

- 可逆矩阵的逆矩阵是唯一的
- 若 $A$ 可逆，则 $A^{-1}$ 也可逆，且 $(A^{-1})^{-1}=A$
- $(AB)^{-1}=B^{-1}A^{-1}$
- 若 $A$ 可逆，则 $A^T$ 也可逆，且 $(A^{T})^{-1}=(A^{-1})^T$

利用定义，很容易证明前三个性质。第四个需要结合转置的性质来证明：

$$
(A^{-1})^TA^T=(AA^{-1})^T=I^T=I
$$

类似于转置的性质，$(AB)^{-1}=B^{-1}A^{-1}$ 也可以进行推广（其中 $A_i$ 都是维度相同的可逆矩阵）：

$$
(\prod_{i=1}^n A_i)^{-1}=\prod_{i=n}^1 A_i^{-1}
$$

逆矩阵的求法较为复杂，需要先做一些准备。

### 初等矩阵

把单位矩阵做一次初等行变换，就得到**初等矩阵**。

如果对矩阵乘法的流程比较熟悉，就可以发现，如果对一个矩阵 $A$ 进行一次初等行变换，就相当于对 $A$ 左乘一个初等矩阵 $E$，其中 $E$ 是对单位矩阵进行同样的初等行变换得到的。

由于初等行变换是可逆的，所以初等矩阵也是可逆的。这就产生了一种求逆矩阵的思路：

若 $n$ 阶方阵 $A$ 是可逆的，那么 $A$ 必须行等价于 $I_n$，把 $A$ 化简为 $I_n$ 的一系列初等行变换作用于 $I_n$ 就得到了 $A^{-1}$。

这就是把 $A$ 和 $I$ 排在一起构成增广矩阵 $[A\quad I]$，然后把 $A$ 变成 $I$ 来求逆矩阵的原理。

也可以换一个角度看待这种方法，对于 $A^{-1}$ 中的第 $i$ 列，将其取出命名为向量 $\boldsymbol{x_i}$，那么相当于有向量方程组

$$
A\boldsymbol{x_i}=\boldsymbol{e_i}
$$

其中 $\boldsymbol{e_i}$ 是单位矩阵的第 $i$ 列形成的向量。

## 分块矩阵

分块的思想十分简单，在我们将一个矩阵看作若干向量的组合时，就已经是一种对矩阵的分块了。

分块矩阵的乘法和普通的矩阵乘法差不多，注意不要写错顺序就行了。

例如：

$$
\left[ \begin{array}{ccc}
A_{11} & A_{12}\\
A_{21} & A_{22} 
\end{array} \right]\times 
\left[ \begin{array}{ccc}
B_{11} & B_{12}\\
B_{21} & B_{22} 
\end{array} \right]
\\ \quad
\\
=

\left[ \begin{array}{ccc}
A_{11}B_{11}+A_{12}B_{21} & A_{11}B_{12}+A_{12}B_{22}\\
A_{21}B_{11}+A_{22}B_{21} & A_{21}B_{12}+A_{22}B_{22}
\end{array} \right]
$$

而分块矩阵求逆就很复杂。具体这里就不写了，只提一个小结论：分块对角矩阵是一个分块矩阵，除了主对角线上的分块外，其余全是零分块。这样的矩阵可逆当且仅当对角线上所有矩阵均可逆。

分块矩阵和矩阵的因式分解在数值计算中都十分重要，以后有空写一下。

## 秩

### 子空间

我们已经知道，一个 $m\times n$ 矩阵 $A$ 可以看作 $n$ 个 $\mathbb{R}^m$ 空间内的向量的集合。当 $A$ 乘上一个向量 $\boldsymbol{x}$ 的时候，就相当于是将这 $n$ 个向量进行了一定长度的伸缩变换后重新加到了一起。那么对于 $\boldsymbol{x}$ 的所有取值，产生的所有新向量 $A\boldsymbol{x}$ 就形成了一个 $\mathbb{R}^m$ 的子空间 $H$。

显然，$H$ 具有以下性质：

- $\boldsymbol{0}\in H$
- $\forall \boldsymbol{u},\boldsymbol{v}\in H,\boldsymbol{u}+\boldsymbol{v}\in H$
- $\forall \boldsymbol{u}\in H, c\boldsymbol{u}\in H$（$c$ 为常数）

所以说，子空间对于加法和标量乘法运算是封闭的。

定义矩阵 $A$ 的**列空间**为 $A$ 的各列的线性组合的集合，记作 $\text{Col }A$。

不难发现，$\text{Col }A$ 就是 $\text{Span}\lbrace \boldsymbol{a_1}\cdots\boldsymbol{a_n} \rbrace$ 的另一种表示。显然，$\text{Col }A\subseteq \mathbb{R}^m$，并且只有在 $A$ 的列生成 $\mathbb{R}^m$ 时取到等号。

定义矩阵 $A$ 的**零空间**为齐次方程 $A\boldsymbol{x}=\boldsymbol{b}$ 的所有解的集合，记作 $\text{Nul }A$。

不难发现，$\text{Nul }A$ 一定是 $\mathbb{R}^n$ 的一个子空间。

子空间一般含有无穷个向量，所以最好能通过一个有限的向量集合来表示一个子空间。定义 $\mathbb{R}^n$ 的子空间 $H$ 的一组**基**是 $H$ 中的一个线性无关集，它生成 $H$。

显然，$n$ 阶单位矩阵中的 $n$ 个列向量 $\boldsymbol{e_1}\cdots\boldsymbol{e_n}$ 就构成了 $\mathbb{R}^n$ 的一组基。$\lbrace\boldsymbol{e_1}\cdots\boldsymbol{e_n}\rbrace$ 称为 $\mathbb{R}^n$ 的**标准基**。

矩阵 $A$ 的主元列构成 $A$ 的列空间的基。

有一点需要特殊说明，零子空间没有基。

### 秩

假设 $\mathcal{B}=\lbrace \boldsymbol{b_1}\cdots\boldsymbol{b_p}  \rbrace$ 是子空间 $H$ 的一组基，对 $H$ 中的每一个向量 $\boldsymbol{x}$，其相对于 $H$ 的坐标是使 $\boldsymbol{x}=c_1\boldsymbol{b_1}+\cdots+c_p\boldsymbol{b_p}$ 成立的一组权值 $c_i$，且 $\mathbb{R}^p$ 中的向量

$$
[\boldsymbol{x}]_{\mathcal{B}}=\left[ 
\begin{array}{ccc}
c_1\\
\vdots\\
c_p
\end{array}
\right]
$$

称为 $\boldsymbol{x}$ 相对于 $\mathcal{B}$ 的坐标向量，或 $\boldsymbol{x}$ 的 $\mathcal{B}$-坐标向量。

#### 同构

如果映射 $\boldsymbol{x}\to [\boldsymbol{x}]_{\mathcal{B}}$ 是 $H$ 和 $\mathbb{R}^p$ 之间保持线性关系的一一映射，则称这种映射是**同构**的，且 $H$ 与 $\mathbb{R}^p$ 同构。

如果 $\mathcal{B}=\lbrace \boldsymbol{b_1}\cdots\boldsymbol{b_p}  \rbrace$ 是 $H$ 的基，则映射 $\boldsymbol{x}\to [\boldsymbol{x}]_{\mathcal{B}}$ 是使 $H$ 和 $\mathbb{R}^p$ 之间保持线性关系的一一映射。

#### 维数

可以证明，一个子空间的所有基都拥有相同数量的向量。

定义子空间 $H$ 的维数 $\dim H$ 为 $H$ 的任意一个基的向量个数。$\lbrace \boldsymbol{0}\rbrace$ 的维数为零。

定义矩阵 $A$ 的**秩**为 $A$ 的列空间的维数，记为 $\text{rank }A$。

如果一个矩阵 $A$ 有 $n$ 列，那么 $\text{rank }A+\dim \text{Nul } A=n$。（秩定理）

虽然这里产生了很多新的概念，但它们都是围绕着列向量的线性相关性展开的。如果 $A$ 是一个 $n$ 阶可逆矩阵，那么必然有：

- $A$ 的列向量是 $\mathbb{R}^n$ 的基
- $\text{rank }A=n$
- $\dim \text{Col }A=n$

