---
title: Elixir2-三个特性
date: 2026-02-21
tag: [Elixir]
category: [Software Engineering]
math: true
---

主要内容：guard，管道，Enum/Stream

<!--more-->

## Guard

在 Elixir 里，模式匹配（比如解构一个 Map）非常强大，但它有一个弱点：**它只能匹配“形状”和“具体的值”，不能做逻辑判断。**

比如，你想写一个函数，判断一个人是否成年：

- 模式匹配可以做到：`def is_adult(18) do ...` （严格匹配 18 岁）
- 但模式匹配做不到：大于等于 18 岁怎么办？

这时候，**Guard（使用 `when` 关键字）** 就出场了。它就像站在模式匹配门后的保安，匹配成功后，还要再查一下条件。

### 1. 在函数定义中使用 Guard

这是 Guard 最常用的场景。你可以定义多个同名函数，通过 Guard 来路由不同的逻辑。

```Elixir
defmodule UserCheck do
  # 场景 A：带有 age 键的 Map，且 age >= 18
  def is_adult(%{age: age}) when age >= 18 do
    "已成年"
  end

  # 场景 B：带有 age 键的 Map，但 age < 18
  def is_adult(%{age: age}) when age < 18 do
    "未成年"
  end

  # 场景 C：兜底（传进来的不是带 age 的 Map，或者格式不对）
  def is_adult(_) do
    "数据无效"
  end
end

IO.puts UserCheck.is_adult(%{age: 20}) # 输出：已成年
IO.puts UserCheck.is_adult(%{age: 12}) # 输出：未成年
```

**对比 C++/Python：** 你不需要在函数体内部写 `if age >= 18`。条件判断直接被提到了函数的“签名”级别，代码极其扁平。

### 2. 在 `case` 中使用 Guard

```Elixir
status_code = 404

case status_code do
  200 -> "成功"
  code when code >= 400 and code < 500 -> "客户端错误: #{code}"
  code when code >= 500 -> "服务器错误: #{code}"
  _ -> "未知状态码"
end
```

### 3. Guard 的“霸王条款”

注意：**在 `when` 后面，你不能调用任何自定义的函数！**

你只能使用 Elixir 提供的一小部分极其高效的内置操作：

- **比较运算符**：`==`, `!=`, `===`, `!==`, `>`, `<`, `>=`, `<=`
- **布尔运算符**：`and`, `or`, `not`
- **数学运算**：`+`, `-`, `*`, `/`
- **类型检查函数（最常用）**：`is_integer/1`, `is_binary/1` (检查是不是字符串), `is_list/1`, `is_map/1`, `is_nil/1` 等。

**为什么有这个限制？** 因为 Elixir 跑在 Erlang 虚拟机（BEAM）上，模式匹配和 Guard 发生在非常底层的虚拟机级别，必须保证**绝对没有副作用**且**执行速度极快**。如果你能在 Guard 里调用一个查数据库的函数，整个虚拟机的调度就崩溃了。

```Elixir
# 错误示范：不能在 Guard 里调用自定义函数
# def check(name) when String.length(name) > 5 do ...

# 正确做法：如果在 Guard 里搞不定，就老老实实放进函数体里用 if/cond，或者查阅系统允许的 Guard 函数（比如 byte_size 是允许的）
def check(name) when byte_size(name) > 5 do
  "名字很长"
end
```

> Guard (`when`) 就是模式匹配的“增强插件”，专门用来做简单的范围判断和类型检查。



## 管道操作符

把左边表达式的结果，作为第一个参数，塞进右边的函数里。即 `a |> f(b)` **完全等价于** `f(a, b)`。

> 为什么我们需要它？（对比 C++/Python）
>
> 假设我们要对一个字符串 `"  elixir is awesome  "` 做三件事：
>
> 1. 去除首尾空格 (`String.trim/1`)
> 2. 把单词按空格拆成列表 (`String.split/1`)
> 3. 把列表里的元素连起来，中间加下划线 (`Enum.join/2`)
>
> - **Python / C++ 的嵌套写法（反直觉，从里往外读）：**
>
>   ```Python
>   # 你的眼睛需要不断寻找括号的对应关系
>   "_".join("  elixir is awesome  ".strip().split())
>   ```
>
> - **Elixir 的正常写法（同样反直觉）：**
>
>   ```elixir
>   Enum.join(String.split(String.trim("  elixir is awesome  ")), "_")
>   ```
>
> - **Elixir 的管道写法（极度舒适）：**
>
>   ```elixir
>   "  elixir is awesome  "  # 1. 初始数据
>   |> String.trim()         # 2. 去空格 -> "elixir is awesome"
>   |> String.split()        # 3. 拆分 -> ["elixir", "is", "awesome"]
>   |> Enum.join("_")        # 4. 拼接 -> "elixir_is_awesome"
>   ```
>
>   *注意看最后一步：`Enum.join/2` 本来需要两个参数 `(列表, 分隔符)`。管道自动把上一步的列表变成了第一个参数，所以我们只需要写第二个参数 `"_"`。*



## Enum

`Enum` 模块包含了所有处理集合（列表、Map、Range）的方法，一般可以起到 `for` 或 `while` 循环的作用。结合管道和匿名函数简写 (`&`)，我们可以写出非常强大的数据处理流。

```Elixir
# 任务：取 1 到 10 的数字，挑出偶数，每个乘以 10，最后求和
1..10
|> Enum.filter(&(rem(&1, 2) == 0))  # 过滤出偶数: [2, 4, 6, 8, 10]
|> Enum.map(&(&1 * 10))             # 乘以 10: [20, 40, 60, 80, 100]
|> Enum.sum()                       # 求和: 300
```

**Enum 的致命弱点：产生中间垃圾（贪婪/Eager）**

> `Enum` 是**贪婪的（Eager）**。这意味着每经过一个 `|> Enum.xxx()`，它都会在内存里完整地生成一个**新的列表**，然后再传给下一步。
>
> 如果上面的范围不是 `1..10`，而是 `1..10_000_000`（一千万）：
>
> 1. `Enum.filter` 会在内存里先造一个 500 万元素的列表。
> 2. `Enum.map` 会再造一个 500 万元素的列表。
>
> 为了解决处理海量数据（或无限数据流、读取超大文件）时的内存爆炸问题，Elixir 引入了 **Stream**。



## Stream（惰性求值）


Stream 的核心哲学是：**惰性（Lazy）**。

它不会立刻去计算结果，也不会生成中间列表。它只是把你要做的操作**“记在一个小本本上”（生成一个计算配方）**。

我们用 Stream 重写刚才处理一千万数据的代码：

```Elixir
1..10_000_000
|> Stream.filter(&(rem(&1, 2) == 0))  # 记下：我要过滤偶数，但不执行
|> Stream.map(&(&1 * 10))             # 记下：我要乘以10，但不执行
|> Enum.sum()                         # 触发器！开始真正干活！
```

**Stream 的底层是如何运行的？**

1. 拿出一个 `1`。不是偶数，丢弃。
2. 拿出一个 `2`。是偶数（通过 filter），立马传给 map 变成 `20`，立马传给 sum 加到总和里。
3. 拿出一个 `3`...
4. 如此循环。

**结果：** 整个过程占用的额外内存几乎为 0！它是一个元素一个元素地穿过整个管道，而不是一批一批地处理。

> 为什么最后一行必须是 `Enum.sum()`？
>
> 因为 `Stream` 里的函数都是“光说不练”的。如果你整条管道都用 `Stream`，它最后只会返回一个表示数据流的结构体（Recipe），根本不会产生你要的结果。
>
> **必须用一个 `Enum` 模块的函数（比如 `Enum.to_list`, `Enum.sum`, `Enum.reduce`）作为管道的终点，来扣动扳机，强制它把数据全部流完。**



### 总结与对比

| **特性**     | **Enum**                     | **Stream**                           |
| ------------ | ---------------------------- | ------------------------------------ |
| **求值方式** | 贪婪 (Eager) - 立刻执行      | 惰性 (Lazy) - 延迟执行               |
| **内存占用** | 高（每步都生成完整的新列表） | 极低（一个元素走完所有流程）         |
| **适用场景** | 中小型数据集合、简单转换     | 读取 GB 级大文件、海量数据、无限序列 |
| **使用规则** | 随时随地使用                 | 必须用 `Enum` 的函数来“收尾触发”     |



