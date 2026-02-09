---
title: CPP1-作用域
date: 2026-02-09
tag: [cpp]
category: [Software Engineering]
math: true
---

从一个简单问题出发，理解 C++ 中的作用域

<!--more-->

看一个简单的问题：

``` cpp
#include <bits/stdc++.h>
using namespace std;

int x = 1;
int main(){
	int x = 2;
	{
		int x = 3;
		cout << ::x << endl;
	}
}
```

这个代码会输出什么？

答案显然是 `1`。

Variable Shadowing 机制告诉我们，在内层代码块的作用域里面，`x = 3`。

然而，前面的作用域解析运算符 `::` 发挥了作用，当 `::` 运算符**前面没有任何前缀**时，它指代的是**全局作用域（Global Namespace）**。所以这里 `::x` 是最外面的全局变量。

有趣的是，没有办法在内层的代码块中访问到那个 `x=2`，`::` 只能跳到最外面的全局变量，或者某个指定的命名空间，无法访问被遮蔽的中间层局部变量。

还有，如果把全局变量修改为 `static int x = 1;`，还是可以访问的，这里的 `static` 只是限制了该变量只能在当前 `.cpp` 文件内可见（内部链接性），它依然属于全局作用域，`::x` 依然有效。
