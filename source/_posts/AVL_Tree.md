---
title: AVL tree
date: 2020-02-12
tag: [data structure]
category: [Algorithms]
math: true
---

# 简介：

AVL树是最先发明的自平衡二叉查找树。AVL树高度平衡，其任意节点左右子树的高度差不超过1。同时它也拥有普通二叉查找树的性质，即每个节点左子树上所有节点的值均小于根节点的值，右子树上所有节点的值均大于根节点的值，且不同节点的值均不同。

<!--more-->

# 实现过程
**平衡因子：**每个节点左子树和右子树的高度差。若所有节点平衡因子的绝对值均不超过1则平衡。

使用一个结构体保存节点的键值、频数、以该点为根的树的树高和大小以及左右子树。当树不平衡时，可以通过旋转使树平衡。

```c++
struct AVL_node;
typedef AVL_node *AVL_Tree;
struct AVL_node
{
    int value, height, freq, size;
    AVL_Tree ls, rs;
    AVL_node() : value(0), height(1), freq(1), size(1), ls(nullptr), rs(nullptr) {}
    AVL_node(int n) : value(n), height(1), freq(1), size(1), ls(nullptr), rs(nullptr) {}
};
```

**不平衡情况1（左-左）：**

![1](https://i.loli.net/2020/02/12/BYrixbevTfP8cR4.png)

在插入1之前，树是平衡的，但是9的左子树比右子树高度大。在9的左子树的左子树插入1（或2，4，5）后，树不平衡了，这种左子树更高且向左子树的左子树插入节点的情况可以通过一次右旋来使树平衡。右旋使原来不平衡的根节点的左子树成为根节点，而原来的根节点成为新的右子树，原来左子树的右子树成为原来的根节点的左子树。容易证明，旋转后依旧具有二叉查找树的性质。

![2](https://i.loli.net/2020/02/12/mz46SiMWg5lTcfJ.png)

```c++
inline void rotate_ls_ls(AVL_Tree &p)
{
    AVL_Tree q;
    q = p->ls;
    p->ls = q->rs;
    q->rs = p;
    update(p);
    update(q);
    p = q;
}
```
**不平衡情况2（右-右）：**

![3](https://i.loli.net/2020/02/12/f5eClWkqbJ7XywL.png)

在插入13之前，树是平衡的，但是8的右子树比左子树高度大。在8的右子树的右子树插入13（或11）后，树不平衡了，这种右子树更高且向右子树的右子树插入节点的情况可以通过一次左旋来使树平衡。左旋使原来不平衡的根节点的右子树成为根节点，而原来的根节点成为新的左子树，原来右子树的左子树成为原来的根节点的右子树。同样，旋转后依旧具有二叉查找树的性质。

![4](https://i.loli.net/2020/02/12/vhUgRs8SFj41HmV.png)
```c++
inline void rotate_rs_rs(AVL_Tree &p)
{
    AVL_Tree q;
    q = p->rs;
    p->rs = q->ls;
    q->ls = p;
    update(p);
    update(q);
    p = q;
}
```
**不平衡情况3：（左-右）**

![5](https://i.loli.net/2020/02/12/dRS1WHXAN7MlUhg.png)

这种情况是在左子树更高的树的左子树的右子树插入节点，需要先进行一次左旋，再进行一次右旋。

左旋：

![6](https://i.loli.net/2020/02/12/9S1xL8N2FIvhuZA.png)

右旋：

![7](https://i.loli.net/2020/02/12/WgVKU1dbrEno9Ra.png)
```c++
inline void rotate_ls_rs(AVL_Tree &p)
{
    rotate_rs_rs(p->ls);
    rotate_ls_ls(p);
}
```
**不平衡情况4：（右-左）**

与情况3相反，先进行一次右旋，再进行一次左旋。

![8](https://i.loli.net/2020/02/12/6KXb74TNLQUO1RB.png)

右旋：

![9](https://i.loli.net/2020/02/12/V5JZvClKTzU8qcP.png)

左旋：

![10](https://i.loli.net/2020/02/12/2h8ktCgqNnZR3IW.png)
```c++
inline void rotate_rs_ls(AVL_Tree &p)
{
    rotate_ls_ls(p->rs);
    rotate_rs_rs(p);
}
```

**获取树的大小**

```c++
inline int get_size(AVL_Tree p)
{
    if (p == nullptr)
        return 0;
    return p->size;
}
```

**获取树的高度**

```c++
inline int get_height(AVL_Tree p)
{
    if (p == nullptr)
        return 0;
    return p->height;
}
```

**更新**

```c++
inline void update(AVL_Tree &p)
{
    p->size = get_size(p->ls) + get_size(p->rs) + p->freq;
    p->height = max(get_height(p->ls), get_height(p->rs)) + 1;
}
```

**插入**

```c++
void AVL_insert(AVL_Tree &p, int x)
{
    if (p == nullptr)
    {
        p = new AVL_node(x);
        return;
    }
    if (p->value == x)//已经存在,频数+1
    {
        (p->freq)++;
        update(p);
        return;
    }
    if (p->value > x)
    {
        AVL_insert(p->ls, x);
        update(p);
        if (get_height(p->ls) - get_height(p->rs) == 2)//插到左边,左边更高
        {
            if (x < p->ls->value)
                rotate_ls_ls(p);
            else
                rotate_ls_rs(p);
        }
    }
    else
    {
        AVL_insert(p->rs, x);
        update(p);
        if (get_height(p->rs) - get_height(p->ls) == 2)//插到右边,右边更高
        {
            if (x > p->rs->value)
                rotate_rs_rs(p);
            else
                rotate_rs_ls(p);
        }
    }
    update(p);
}
```

**删除数据**

```c++
void AVL_erase(AVL_Tree &p, int x)
{
    if (p == nullptr)
        return;
    if (p->value > x)
    {
        AVL_erase(p->ls, x);
        update(p);
        if (get_height(p->rs) - get_height(p->ls) == 2)//删左边，右边高
        {
            if (get_height(p->rs->rs) >= get_height(p->rs->ls))//见代码后注释
                rotate_rs_rs(p);
            else
                rotate_rs_ls(p);
        }
    }
    else if (p->value < x)
    {
        AVL_erase(p->rs, x);
        update(p);
        if (get_height(p->ls) - get_height(p->rs) == 2)
        {
            if (get_height(p->ls->ls) >= get_height(p->ls->rs))
                rotate_ls_ls(p);
            else
                rotate_ls_rs(p);
        }
    }
    else
    {
        if (p->freq > 1)
        {
            (p->freq)--;
            update(p);
            return;
        }
        if (p->ls && p->rs)
        {
            AVL_Tree q;
            q = p->rs;
            while (q->ls)
                q = q->ls;
            p->freq = q->freq;
            p->value = q->value;
            q->freq = 1; //删除q
            AVL_erase(p->rs, q->value);
            if (get_height(p->ls) - get_height(p->rs) == 2)
            {
                if (get_height(p->ls->ls) >= get_height(p->ls->rs))
                    rotate_ls_ls(p);
                else
                    rotate_ls_rs(p);
            }
        }
        else
        {
            AVL_Tree q;
            q = p;
            if (p->ls)
                p = p->ls;
            else if (p->rs)
                p = p->rs;
            else
                p = nullptr;
            delete q;
            q = nullptr;
        }
    }
    if (p == nullptr)
        return;
    update(p);
}
```

第11行必须用$\ge$，否则遇到如图的情况：

![11](https://i.loli.net/2020/02/12/XuxmhoWtJvaCqwp.png)

删去13后，先右旋再左旋，树仍然不是平衡的：

![12](https://i.loli.net/2020/02/12/U4IajS9vukohHml.png)

**通过值查找位次**

```c++
int get_rank(AVL_Tree p, int x)
{
    if (p->value == x)
        return get_size(p->ls) + 1;
    if (p->value > x)
        return get_rank(p->ls, x);
    return get_rank(p->rs, x) + get_size(p->ls) + p->freq;
}
```

**通过位次查找值**

```c++
int get_val(AVL_Tree p, int r)
{
    if (get_size(p->ls) >= r)
        return get_val(p->ls, r);
    if (get_size(p->ls) + p->freq >= r)
        return p->value;
    return get_val(p->rs, r - get_size(p->ls) - p->freq);
}
```

**查找前驱**

```c++
int get_pre(AVL_Tree p, int x)
{
    AVL_Tree ans = new AVL_node(-inf);
    while (p)
    {
        if (p->value == x)
        {
            if (p->ls)
            {
                p = p->ls;
                while (p->rs)
                {
                    p = p->rs;
                }
                ans = p;
            }
            break;
        }
        if (p->value < x && p->value > ans->value)
            ans = p;
        p = p->value < x ? p->rs : p->ls;
    }
    return ans->value;
}
```

**查找后继**

```c++
int get_suf(AVL_Tree p, int x)
{
    AVL_Tree ans = new AVL_node(inf);
    while (p)
    {
        if (p->value == x)
        {
            if (p->rs)
            {
                p = p->rs;
                while (p->ls)
                {
                    p = p->ls;
                }
                ans = p;
            }
            break;
        }
        if (p->value > x && p->value < ans->value)
            ans = p;
        p = p->value < x ? p->rs : p->ls;
    }
    return ans->value;
}
```

**中序遍历**

```c++
void output(AVL_Tree p)
{
    if(p==nullptr)
        return;
    output(p->ls);
    for (int i = 1; i <= p->freq;i++)
        printf("%d ", p->value);
    output(p->rs);
}
```

AVL树的插入、删除、查找都是$\log$级别的复杂度，其中AVL树的查询和其它平衡树相比会更有优势，但是AVL树代码量比较大，总体速度也很一般，可以分裂合并，可持久化，但是比较难写。

终于写完了，真是不容易。

![v2-28825ec8201c53892205bfd8b41c97dd_hd.jpg](https://i.loli.net/2020/02/12/1scLYZaiKRt25N3.jpg)

