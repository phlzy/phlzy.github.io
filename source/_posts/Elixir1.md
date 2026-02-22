---
title: Elixir1-基础语法
date: 2026-02-20
tag: [Elixir]
category: [Software Engineering]
math: true
---

主要内容：Struct，case/cond/with，匿名函数

<!--more-->

## Struct

Map 很自由，你可以随便塞任何键值对进去，例如 `%{"name" => "Alice", age: 25}`。考虑一个场景，如果把 `age` 拼写成了 `aeg`，Map 不会报错，这就可能会出 bug。

Struct 就是为了解决这个问题而生的。它就是 C++ 里的 `struct` 或者 Python 里带 `@dataclass` 的类。

**定义**：Struct 不能像 Map 那样随便创建，它**必须依附于一个模块（Module）**。模块的名字，就是这个 Struct 的名字。

```Elixir
defmodule User do
  # 使用 defstruct 定义这个结构体允许拥有哪些键，以及它们的默认值
  defstruct name: "匿名用户", age: 0, is_admin: false
end
```

**创建一个 Struct 实例**：创建 Struct 的语法和 Map 几乎一模一样，只是在 `%` 和 `{}` 之间**加上了模块的名字**。

```Elixir
# 1. 使用默认值创建
user1 = %User{}
# 结果: %User{age: 0, is_admin: false, name: "匿名用户"}

# 2. 指定值创建
user2 = %User{name: "Gemini", age: 3}
# 结果: %User{age: 3, is_admin: false, name: "Gemini"}

# 3. 尝试塞入不存在的键（报错）
# user3 = %User{name: "Bob", height: 180} 
# 报错: key :height not found in struct User
```

### Struct 的底层是什么

**Struct 的本质就是一个 Map，只是它多了一个隐藏的叫 `__struct__` 的键。**

你可以用访问 Map 的方式去访问 Struct：

```Elixir
# 访问数据 (因为 Struct 的键强制是原子 Atom，所以可以放心用点语法)
IO.puts(user2.name) # 输出 "Gemini"

# 更新数据 (和 Map 一模一样，返回一个新 Struct)
new_user = %{user2 | age: 4}
```

### 为什么要用 Struct 而不用 Map

假设你写了一个函数，专门用来升级管理员权限。你希望这个函数**只能**接收 `User` 结构体，如果别人传进来一个普通的 Map，或者传了个 `Product` 结构体，你要立刻拒绝。

**在 Python 里：**

```Python
def make_admin(user):
    if not isinstance(user, User):
        raise TypeError("必须是 User 对象")
    # ...
```

**在 Elixir 里（模式匹配）：**

```Elixir
defmodule AdminTools do
  # 这里的 %User{} 就是在做模式匹配！
  # 意思是：只有当你是一个 User 结构体时，才允许进入这个函数。
  def make_admin(%User{} = user) do
    %{user | is_admin: true}
  end
end

# 调用测试
user = %User{name: "Alice"}
AdminTools.make_admin(user) # 成功返回新的 User 结构体

# 如果传个普通 Map 进去
bad_data = %{name: "Alice", age: 20}
# AdminTools.make_admin(bad_data) # 报错！FunctionClauseError (找不到匹配的函数)
```

> Struct 就是有严格字段约束的 Map

## 匿名函数

匿名函数已经很常见了，但在 Elixir 中，匿名函数的地位极高，而且语法更强大（因为支持模式匹配）。

### 标准写法

在 Elixir 里，定义一个匿名的、没有名字的函数，需要用 `fn` 开头，`end` 结尾。

```elixir
# 定义一个加法匿名函数，绑定给变量 add
add = fn a, b -> a + b end

# Python 对比: add = lambda a, b: a + b
```

注意，在 Elixir 里，**调用匿名函数必须加一个点 `.`**！

```elixir
# 报错！Elixir 会以为你在调用一个叫 add 的全局命名函数
# add(1, 2) 

# 正确姿势：变量名 + 点 + 括号
result = add.(1, 2) 
IO.puts(result) # 输出 3
```

**为什么这么反直觉？** 这是因为 Elixir 严格区分了“模块里的命名函数”和“绑定在变量上的匿名函数”。加个点是为了让你一眼就看出来：“哦，这玩意是个存在变量里的函数对象”。

### 模式匹配

Python 的 `lambda` 只能写一行简单的表达式，但在 Elixir 里，匿名函数可以配合模式匹配，变得非常强悍，甚至充当 `if/else` 的角色。

```elixir
# 这个匿名函数可以根据传入数据的不同形状，走不同的逻辑
handle_result = fn
  {:ok, data} -> "成功获取数据: #{data}"
  {:error, reason} -> "糟糕，出错了: #{reason}"
end

# 测试一下
IO.puts handle_result.({:ok, "Elixir 真好玩"})   # 输出: 成功获取数据: Elixir 真好玩
IO.puts handle_result.({:error, "网络断开"})    # 输出: 糟糕，出错了: 网络断开
```

注意：这里不需要写多个 `fn`，只需要在一个 `fn ... end` 内部写多个匹配分支即可。

### Capture Operator

当你的匿名函数非常简单的时候（比如只是加个 1，或者只是调用另一个函数），写 `fn x -> ... end` 显得太啰嗦了。这时候，**`&` (Capture Operator)** 就闪亮登场了。它有两个主要用途：

#### 用途 1：快速创建简写版匿名函数

你可以用 `&(...)` 来代替 `fn ... -> ... end`。在括号里，用 **`&1`, `&2`, `&3`** 来代表第 1 个、第 2 个、第 3 个参数。

```Elixir
# 啰嗦写法：
add_one = fn x -> x + 1 end
sum = fn a, b -> a + b end

# 极简写法 (完全等价)：
add_one_fast = &(&1 + 1)
sum_fast = &(&1 + &2)

# 调用方式还是一样，加点！
IO.puts add_one_fast.(5)  # 输出 6
IO.puts sum_fast.(10, 20) # 输出 30
```

#### 用途 2：函数指针

如果你只想把一个已经存在的函数作为参数传给别人（比如传给 `Enum.map`），你可以直接用 `&模块名.函数名/参数个数` 来捕获它。

```Elixir
# 比如我们要把列表里的字符串都转成大写
names = ["alice", "bob"]

# 啰嗦写法：
Enum.map(names, fn name -> String.upcase(name) end)

# 优雅写法 (直接捕获 String 模块里的 upcase 函数，它接收 1 个参数)：
Enum.map(names, &String.upcase/1)
```

## 控制流

case/cond/with 的用法。

### case

`case` 就像 C++ 的 `switch`，或者是 Python 3.10 引入的 `match...case`，但它不仅比对值，还能**解构数据**。

```Elixir
# 假设我们调用了一个网络请求函数
response = {:ok, %{name: "Gemini", age: 3}}

# 使用 case 处理不同的情况
case response do
  {:ok, user} ->
    # 匹配成功！并且顺便把 Map 里的数据提取到了 user 变量里
    "请求成功，用户名字是：#{user.name}"

  {:error, "timeout"} ->
    "请求超时，请重试。"

  {:error, reason} ->
    # 这个 reason 相当于一个通配符，捕获其他所有错误
    "发生了未知错误：#{reason}"
    
  _ ->
    # 下划线是 Elixir 的“垃圾桶”，表示匹配任何之前没匹配到的东西（相当于 default）
    "这是啥玩意？"
end
```

**进阶技巧：配合 Guard (卫语句)** 你甚至可以在 `case` 的分支里加上条件判断：

```Elixir
case age do
  x when x >= 18 -> "成年人"
  x when x < 18 -> "未成年"
end
```

### cond

如果你有一堆**互不相关**的条件需要判断，用 `case` 就不合适了。 在 Python 里你会写 `if ... elif ... elif ... else`。在 Elixir 里，我们用 `cond`。

**使用场景：** 依次评估多个条件，执行**第一个为真（truthy）**的分支。

```Elixir
# 经典的 FizzBuzz 游戏
number = 15

cond do
  rem(number, 3) == 0 and rem(number, 5) == 0 ->
    "FizzBuzz"

  rem(number, 3) == 0 ->
    "Fizz"

  rem(number, 5) == 0 ->
    "Buzz"

  true ->
    # 这是 cond 的灵魂！相当于最后的 else。
    # 因为 true 永远为真，如果上面的条件都不满足，就会走到这里。
    "数字是 #{number}"
end
```

*注：在 Elixir 中，除了 `false` 和 `nil` 之外，所有东西都是“真”（truthy）。*

### with

这是 Elixir 最闪亮的特性之一，甚至超越了大部分现代语言。

**痛点：** 假设你要完成一个连贯的业务逻辑：

1. 验证用户 token 是否有效。
2. 根据 token 获取用户 ID。
3. 根据用户 ID 获取数据库里的数据。

在 C++ 或 Python 里，如果中途任何一步失败，你要写很深的嵌套，或者抛出一堆异常（"Pyramid of Doom"）：

```Python
# Python 里的噩梦嵌套
token_data = verify_token(token)
if token_data:
    user_id = get_user_id(token_data)
    if user_id:
        user = get_user_from_db(user_id)
        if user:
            return user
        else:
            return "用户不存在"
    else:
        return "解析ID失败"
else:
    return "Token 无效"
```

**Elixir 的解药：`with`**

 `with` 语句只走“快乐路径”：只要模式匹配成功，就接着往下走；**一旦某一步匹配失败，立刻终止，并返回那个失败的值。**

```Elixir
# 假设这三个函数都会返回 {:ok, data} 或 {:error, reason}
with {:ok, token_data} <- verify_token(token),
     {:ok, user_id} <- get_user_id(token_data),
     {:ok, user} <- get_user_from_db(user_id) do
  # 只有上面三步全部匹配 {:ok, ...} 成功，才会执行这里的代码
  "完美获取用户数据: #{user.name}"
else
  # 如果上面任何一步失败了（比如返回了 {:error, "Token expired"}），
  # 就会直接跳到这里的 else 块进行模式匹配。
  {:error, "Token expired"} -> "登录已过期，请重新登录"
  {:error, _reason} -> "获取失败，发生了系统错误"
  nil -> "找不到数据"
end
```

**注意：** `with` 里面用的是 `<-` 而不是 `=`。`<-` 的意思是：“如果匹配成功，就赋值并继续；如果失败，就把失败的值扔给最后的 `else` 块（如果没有写 else 块，就直接返回失败的值）”。

### 总结

1. **`case`**：盯着**一个值**，看它的形状来分叉。
2. **`cond`**：盯着**一堆条件**，哪个为 `true` 就执行哪个。
3. **`with`**：像流水线一样执行**一系列容易失败的操作**，一旦出错立刻退出。



