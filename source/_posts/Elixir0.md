---
title: Elixir0-常识
date: 2026-02-19
tag: [Elixir]
category: [Software Engineering]
math: true
---

Elixir 可能是最容易上手的函数式语言，从 Elixir 开始培养函数式手感。

<!--more-->

# 安装

Windows11 系统，[官方下载页面](https://elixir-lang.org/install.html#windows)。

没有 Erlang，[Download and run the Erlang installer](https://www.erlang.org/downloads.html) 下载，目前的版本是 `otp_win64_28.3.1.exe`。

然后 [Elixir 1.19.5 on Erlang 28](https://github.com/elixir-lang/elixir/releases/download/v1.19.5/elixir-otp-28.exe) 下载 `elixir-otp-28.exe`，一路 next 会自动配置环境变量。

装完以后在 cmd 输入 `iex -v` 检查，不成功就重启电脑。

```
Erlang/OTP 28 [erts-16.2] [source] [64-bit] [smp:16:16] [ds:16:16:10] [async-threads:1] [jit:ns]

IEx 1.19.5 (compiled with Erlang/OTP 28)
```

## Learning

[官方页面](https://elixir-lang.org/learning.html) 给出了一些书籍和课程。

[Joy of Elixir](https://joyofelixir.com/) 可以在线阅读，[exercism](https://exercism.org/tracks/elixir) 可以在线练习。

# 基本概念

## 基础类型

参考 [官方文档](https://hexdocs.pm/elixir/basic-types.html)：

``` elixir
1          # integer
0x1F       # integer
1.0        # float
true       # boolean
:atom      # atom / symbol
"elixir"   # string
[1, 2, 3]  # list
{1, 2, 3}  # tuple
```

基础代数运算中只有除法是特殊的：In Elixir, the operator [`/`](https://hexdocs.pm/elixir/Kernel.html#//2) always returns a float. If you want to do integer division or get the division remainder, you can invoke the [`div`](https://hexdocs.pm/elixir/Kernel.html#div/2) and [`rem`](https://hexdocs.pm/elixir/Kernel.html#rem/2) functions:

``` elixir
10 / 2
5.0
div(10, 2)
5
div 10, 2
5
rem 10, 3
1
```

浮点数都是 64 位的：Floats in Elixir are 64-bit precision.

You can invoke the [`round`](https://hexdocs.pm/elixir/Kernel.html#round/1) function to get the closest integer to a given float, or the [`trunc`](https://hexdocs.pm/elixir/Kernel.html#trunc/1) function to get the integer part of a float.

Finally, we work with different data types, we will learn Elixir  provides several predicate functions to check for the type of a value.  For example, [`is_integer`](https://hexdocs.pm/elixir/Kernel.html#is_integer/1) can be used to check if a value is an integer or not:

```elixir
is_integer(1)
true
is_integer(2.0)
false
```

You can also use [`is_float`](https://hexdocs.pm/elixir/Kernel.html#is_float/1) or [`is_number`](https://hexdocs.pm/elixir/Kernel.html#is_number/1) to check, respectively, if an argument is a float, or either an integer or float.

没有隐式类型转换，布尔运算符有短路性质：
``` elixir
false and raise("This error will never be raised")
false
true or raise("This error will never be raised")
true
```

### nil

Elixir also provides the concept of `nil`, to indicate the absence of a value, and a set of logical operators that also manipulate `nil`: [`||/2`](https://hexdocs.pm/elixir/Kernel.html#||/2), [`&&/2`](https://hexdocs.pm/elixir/Kernel.html#&&/2), and [`!/1`](https://hexdocs.pm/elixir/Kernel.html#!/1). For these operators, `false` and `nil` are considered "falsy", all other values are considered "truthy":

```elixir
# or
1 || true
1
false || 11
11

# and
nil && 13
nil
true && 17
17

# not
!true
false
!1
false
!nil
true
```

Similarly, values like `0` and `""`, which some other programming languages consider to be "falsy", are also "truthy" in Elixir.

As a rule of thumb, use `and`, `or` and `not` when you are expecting booleans. If any of the arguments are non-boolean, use `&&`, `||` and `!`.

## Atom

Atom 就和 Enum 差不多，常用于枚举状态、标签、错误原因：

```elixir
{:error, :invalid_input}
```

> `true`，`false` 以及 `nil` 也是 atom。

## String

拼接：You can concatenate two strings with the [`<>`](https://hexdocs.pm/elixir/Kernel.html#<>/2) operator.

插入：String concatenation requires both sides to be strings but  interpolation supports any data type that may be converted to a string:

```elixir
number = 42
"i am #{number} years old!"
"i am 42 years old!"
```

## Structural comparison

其他的差不多，看下面这个就行：

Integers and floats compare the same if they have the same value:

```elixir
1 == 1.0
true
1 == 2.0
false
```

However, you can use the strict comparison operator [`===`](https://hexdocs.pm/elixir/Kernel.html#===/2) and [`!==`](https://hexdocs.pm/elixir/Kernel.html#!==/2) if you want to distinguish between integers and floats:

```elixir
1 === 1.0
false
```

The comparison operators in Elixir can compare across any data type. We say these operators perform *structural comparison*. For more information, you can read our documentation on [Structural vs Semantic comparisons](https://hexdocs.pm/elixir/Kernel.html#module-structural-comparison).

## Collections

### Lists

单向链表，用法就和广义表差不多，`hd` 和 `tl`。

> List operators never modify the existing list. Concatenating to or  removing elements from a list returns a new list. We say that Elixir  data structures are *immutable*.

Two lists can be concatenated or subtracted using the [`++`](https://hexdocs.pm/elixir/Kernel.html#++/2) and [`--`](https://hexdocs.pm/elixir/Kernel.html#--/2) operators respectively:

```elixir
[1, 2, 3] ++ [4, 5, 6]
[1, 2, 3, 4, 5, 6]
[1, true, 2, false, 3, true] -- [true, false]
[1, 2, 3, true]
```

注意单向链表实现这些操作的复杂度。

### Tuples

大括号，索引从 0 开始，内存连续。

```elixir
tuple = {:ok, "hello"}
{:ok, "hello"}
elem(tuple, 1)
"hello"
tuple_size(tuple)
2
```

It is also possible to put an element at a particular index in a tuple with [`put_elem`](https://hexdocs.pm/elixir/Kernel.html#put_elem/3):

```elixir
tuple = {:ok, "hello"}
{:ok, "hello"}
put_elem(tuple, 1, "world")
{:ok, "world"}
tuple
{:ok, "hello"}
```

> Notice that [`put_elem`](https://hexdocs.pm/elixir/Kernel.html#put_elem/3) returned a new tuple. The original tuple stored in the `tuple` variable was not modified. Like lists, tuples are also immutable. Every operation on a tuple returns a new tuple, it never changes the given  one.

### Maps

Map 的 Key 可以是任何类型，但是如果是 atom，有语法糖。

通用写法，使用 `=>`（火箭符号）：

```elixir
# 注意外面的 % 号！没有 % 就是元组 (Tuple) 了。
user = %{
  "name" => "Alice",
  1      => "one",
  :age   => 25
}
```

语法糖写法，使用 `key: value`，如果你的键**全部都是原子 (Atom)**，Elixir 允许你省略掉火箭符号，写成类似 JSON 的样子：

```elixir
# 语法糖
user = %{name: "Alice", age: 25}

# user = %{:name => "Alice", :age => 25}
```

> 如果你在一个 Map 里既有字符串键，又有原子键（语法糖），**语法糖必须放在最后面**。
>
> 点 `.` 只能访问原子键：`user.name`
>
> 注意一切都是不变的，不要用其他语言的赋值语法去修改 map：
>
> ```elixir
> user = %{name: "Alice", age: 25}
> 
> # 报错！Elixir 里没有这种赋值语法
> user["age"] = 26 
> user.age = 26
> 
> # 如果要修改【已存在】的键，用特定的更新语法 %{原map | 键 => 新值}
> new_user = %{user | age: 26}
> 
> # 如果要增加【新】的键，必须调用 Map 模块的函数，并产生一个新 Map
> new_user = Map.put(user, :city, "Beijing")
> ```
>
> **`map[key]` 是安全的**：无论 key 存不存在，都不会崩溃。如果不存在，返回 `nil` (类似 Python 的 `dict.get('key')`)。
>
> **`map.key` 是严格的**：**只能用于原子键**。如果这个键不存在，程序会直接抛出 `KeyError` 崩溃。这在确定某个键必须存在时非常好用。

在任何时候都要记得，不同于常见的语言， elixir 里面的 `=` 是一种匹配的关系，是把一个值绑定到一个“变量”上，而不是赋值。
