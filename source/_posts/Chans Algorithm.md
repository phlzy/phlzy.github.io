---
title: Chan's Algorithm
date: 2024-05-20
tag: [convex optimization, geometry]
category: [Algorithm and DataStructure]
math: true
---

理论最优的求凸包算法 Chan's Algorithm。

<!--more-->

# Chan's Algorithm

## 简介

以往常见的求凸包的算法复杂度多为 $\mathcal{\Theta}(n\log n)$（如 Graham Scan 算法、Andrew 算法等），其中 $n$ 是平面内的点数。

当事先已知大多数点位于凸包内部，只有少数点位于边界上时，也有更高效的算法，如 Jarvis March 算法，其复杂度为 $\mathcal{\Theta}(nh)$，其中 $h$ 是凸包的顶点数。

理论上，存在 $\mathcal{O}(n\log h)$ 的算法，如 Kirkpatrick-Seidel 算法（Marriage-before-Conquest），但是非常复杂，不容易实现。Chan's Algorithm 结合了 Graham Scan 算法和 Jarvis March 算法，同样达到了 $\mathcal{O}(n\log h)$ 的复杂度，且更容易实现。

## Graham Scan

简单提一下 Graham Scan 算法的流程：

1. 选最左下角的点作为基准，因为这个点必然在凸包上
2. 对剩下的点进行极角排序
3. 遍历平面上所有点，将凸包视为一个单调栈，如果加入当前点的方向是逆时针的，就加入；否则不断弹出栈顶，直到新加入的点与原有的点形成的方向是逆时针的为止。

``` python
def graham_scan(points):
    n = len(points)
    if n < 3:
        return

    # Step 1: Find the point with the lowest y-coordinate (and the leftmost in case of a tie)
    p0 = min(points, key=lambda p: (p.y, p.x))
    points.remove(p0)

    # Step 2: Sort points based on the polar angle with p0
    points = sorted(points, key=cmp_to_key(lambda p1, p2: compare(p0, p1, p2)))

    # Step 3: Initialize the convex hull with the first three points
    hull = [p0, points[0], points[1]]

    # Step 4: Process the remaining points
    for p in points[2:]:
        while len(hull) > 1 and orientation(hull[-2], hull[-1], p) != 2:
            hull.pop()
        hull.append(p)

    return hull
```

## Jarvis March

简单提一下 Jarvis March 算法的流程：

1. 先找一个基准点，同样可以找最左下角的
2. 从所有点中找出和凸包已有的最后一条边形成的极角最小的点（最靠近凸包外侧的点），将其加入凸包，直至凸包闭合

``` python
def jarvis_march(points):
    n = len(points)
    if n < 3:
        return

    hull = []

    # Find the leftmost point
    l = 0
    for i in range(1, n):
        if points[i].x < points[l].x:
            l = i

    p = l
    while True:
        hull.append(points[p])
        q = (p + 1) % n

        for i in range(n):
            if orientation(points[p], points[i], points[q]) == 2:
                q = i

        p = q

        if p == l:
            break

    return hull
```

## Chan's Algorithm

假设平面上一共有 $n$ 个不同的点，最后得到的凸包有 $h$ 个顶点，Chan's Algorithm 的主要流程如下：

1. 先将所有点 $m$ 个一组，分为 $\lceil \frac{n}{m}\rceil$ 组
2. 对每一组使用 Graham Scan 求凸包，得到 $\lceil \frac{n}{m}\rceil$ 个凸包，复杂度 $\mathcal{O}(n\log m)$
3. 对上一步中得到的 $\lceil \frac{n}{m}\rceil$ 个凸包使用 Jarvis March 算法，求整体的凸包

前两步没什么可说的，算法的核心在于第三步：

注意到，使用 Jarvis March 算法求凸包的时候每次都需要找出极角最小的点。在一般的 Jarvis March 算法中，这样做是 $\mathcal{O}(n)$ 的。然而，由于第二步中求出的是 $\lceil \frac{n}{m}\rceil$ 个凸包，而在凸包上求极角最小的顶点，可以二分，复杂度是 $\mathcal{O}(\log m)$，因此这里每次找一个点的复杂度就是 $\mathcal{O}(\frac{n}{m}\log m)$。这样找到 $h$ 个点，第三步的复杂度是 $\mathcal{O}(\frac{hn}{m}\log m)$。当 $m=h$ 的时候，总复杂度就变成了我们想要的 $\mathcal{O}(n\log h)$。

此时我们就遇到了第二个问题：如果直接像 Jarvis March 算法那样找 $h$ 次，总复杂度是 $\mathcal{O}(\frac{hn}{m}\log m)$，因为 $h$ 是未知的，如果 $m$ 的取值过小，就会退化成 $\mathcal{O}(nh)$，而 $m$ 过大就会退化成 $\mathcal{O}(n\log n)$。这里也不能用二分来确定 $m$ 的值，因为当确定了二分的上界是 $n$ 的时候，一切就全完了。此时就需要引入第二个 trick：倍增。

由于无法事先知道 $h$ 是多少，我们干脆就假设 $m=h$，在第三步的 Jarvis March 中，我们只找至多 $m$ 个点，如果这些点形成的图形闭合了，那么答案自然就已经找到了；如果没有闭合，就说明 $m$ 取小了。这样做之后，对 $m$ 做一次尝试的复杂度就是 $\mathcal{O}(n\log m)$。显然，我们可能会做很多次尝试，直到 $m\ge h$ 为止，假设总共试了 $k$ 次，最后整个算法的复杂度是 
$$
n\sum_{i=1}^{k}\log m_i
$$
这个复杂度取决于 $m_i$ 的取值策略：

若直接遍历，如 $m_i=2,3,4,\cdots$，复杂度为 $\mathcal{O}(nh\log h)$

若使用常用的倍增技术，如 $m_i=2^1,2^2,2^3,\cdots$，复杂度为 $\mathcal{O}(n\log^2 h)$，此时 $k=\lceil \log_2 h\rceil$

这个倍增虽然没有成功将复杂度变成我们想要的样子，但它多少能给人带来一些启发：

注意到 $2^1+2^2+\cdots +2^k=2^{k+1}-1$，如果想让这些 $\log m_i$ 加起来是 $\mathcal{O}(\log h)$ 的，我们可以试着让 $k=\log \log h$，同时 $\log m_i=2^i$，即 $m_i=2^{2^i}$。

现在再来算一下复杂度：

$$
n\sum_{i=1}^{\log \log h}\log 2^{2^i}
=n\sum_{i=1}^{\log \log h}2^i
<n2^{1+\log \log h}
=2n\log h=\mathcal{O}(n\log h)
$$




## 后记

顺便一提，该算法也可以拓展到三维的情况。

以后有空再说了。
