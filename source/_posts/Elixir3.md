---
title: Elixir3-控制流核心
date: 2026-02-22
tag: [Elixir]
category: [Software Engineering]
math: true
---

Elixir 的控制流核心：模式匹配（Pattern Matching）与尾递归（Tail Recursion）

<!--more-->

在 Elixir 里，**模式匹配就是你的 `if/else`，尾递归就是你的 `while` 循环。**

## 模式匹配

### 匹配操作符

重新认识 `=`：它不是赋值，是“模式匹配”。它的本质是解代数方程：**让等号左右两边的形状（Shape）保持一致。**当我们让变量 `=` 一个新的值，进行的是变量的重绑定（Rebinding）。

### Pin Operator

在常见的语言里，经常会写 `if (x == 2)` 这样的语句，Elixir 也提供了 `==` 返回一个布尔值。

但是，特殊的是，Elixir 提供了一个 Pin Operator `^`，它的作用是在模式匹配中**强制要求变量保持它现有的值，而不允许它被重新绑定**。

例如：

```Elixir
x = 1

# 加上 ^，意思是：“我不仅要模式匹配，而且等号右边的值必须匹配 x 当前的值（1）！”
^x = 2
```

这时候，Elixir 就会抛出一个报错： ` (MatchError) no match of right hand side value: 2`。

这个相当于 C++ 里的断言或条件判断 `assert(x == y);` 或者 `if (x == y)`。

这看起来似乎没什么大用，因为我们本身就不习惯模式匹配的写法。但 `^` 真正的威力，在于配合**复杂数据结构的解构**，或者在 `case` 分支语句中进行**动态路由**。

**场景 A：在元组或列表中精准捕获**。假设你在写图论算法，你手里有一个目标节点 `target_node = 5`，你正在遍历一个包含 `{节点, 距离}` 的边列表。

```Elixir
target_node = 5

# 我们只想要那些目标节点正好是 5 的边，顺便提取出它的 distance
# 如果写 {target_node, distance} = {5, 10}，target_node 会被重新绑定，这不是我们想要的。

# 正确做法：钉死 target_node！
{^target_node, distance} = {5, 10}  # 匹配成功！distance 变成了 10
{^target_node, distance} = {8, 15}  # 匹配失败！直接引发报错或跳过（如果在 case 里）
```

**场景 B：在 `case` 语句中当做动态的 `switch-case`**。在 C++ 的 `switch` 里，`case` 后面只能跟常量（比如 `case 1:`）。在 Elixir 里，你可以用 `^` 匹配动态变量：

```Elixir
def process_data(data, expected_status) do
  case data do
    # 只有当 data 里的 status 等于外面传进来的 expected_status 时，才走这条分支
    %{status: ^expected_status, value: val} -> 
      "匹配成功！拿到了值: #{val}"
      
    %{status: other_status} -> 
      "状态不对，期望 #{expected_status}，但收到 #{other_status}"
  end
end
```

所以灵活使用这些特性，可以在面对复杂嵌套的结构时省去无数个繁琐的 `if` 嵌套。

## 尾递归

为什么需要递归？因为没有 `while`，因为变量不可变，你不能写出 `int i = 0; while(i < n) { i++; }` 这样的代码。状态必须通过函数的参数向后传递，这就注定了**循环必须通过递归来实现**。

结合模式匹配，我们可以在函数签名上直接做分支判断（这就是为什么 Elixir 代码里很少见到长篇大论的 `if-else`）。

### 传统递归

假设我们要对一个 List 求和。

```Elixir
defmodule Math do
  # 边界条件：空列表的和是 0
  def sum([]), do: 0
  
  # 递归条件：匹配出 head 和 tail，头元素加上剩余部分的和
  def sum([head | tail]), do: head + sum(tail)
end
```

**为什么这种递归会爆栈？**

当你调用 `sum([1, 2, 3])` 时，计算机会这样展开：

```
1 + sum([2, 3])
1 + (2 + sum([3]))
1 + (2 + (3 + sum([])))
1 + (2 + (3 + 0))
```

每往下一层，计算机都必须把当前的上下文（如当前的 `head`）压入**调用栈（Call Stack）**，因为它得**等待**下一层返回结果后，才能执行那个 `+` 号。如果列表长度很大，就会 Stack Overflow。

### 尾递归

为了解决爆栈问题，函数式编程引入了**尾递归**。

所谓“尾递归”，就是指**函数在执行的最后一步，仅仅只调用了它自己，没有任何其他的附加计算。**

为了做到这一点，我们通常会引入一个叫做 **累加器（Accumulator, 简称 acc）** 的参数，用来沿途记录状态。

来看看尾递归版本的求和：

```Elixir
defmodule Math do
  # 对外暴露的接口，默认初始累加器为 0
  def sum(list), do: do_sum(list, 0)

  # 边界条件：列表为空时，直接返回累加器里已经算好的结果！
  defp do_sum([], acc), do: acc

  # 尾递归条件：把当前的 head 加到 acc 里，传给下一次调用
  defp do_sum([head | tail], acc) do
    # 注意！递归调用是整个函数的绝对最后一步，外面没有任何加减乘除
    do_sum(tail, acc + head) 
  end
end
```

**为什么尾递归不会爆栈？**

当我们执行 `do_sum([1, 2, 3], 0)` 时：

```
do_sum([2, 3], 1)
do_sum([3], 3)
do_sum([], 6)
```
因为当前函数在调用完下一层之后，**没有任何其他事情要做了**（没有未完成的 `+` 操作）。所以，Erlang 虚拟机（BEAM）的编译器会做一件事：**尾调用优化（Tail Call Optimization, TCO）。**

编译器在底层会直接把当前栈帧（Stack Frame）的内存覆盖掉，**复用当前的栈空间**，直接跳回到函数开头执行。这在汇编层面，本质上就是执行了一个 `JMP` 指令！

