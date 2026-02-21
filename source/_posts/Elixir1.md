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

在上一节我们讲了 Map `%{"name" => "Alice", age: 25}`。 Map 很自由，你可以随便塞任何键值对进去。但是，当你在写一个大项目时（比如处理几百个用户的请求），太自由就意味着**容易写出 Bug**。

比如，你把 `age` 拼写成了 `aeg`，Map 不会报错，但后面的逻辑全乱了。

**Struct 就是为了解决这个问题而生的。它就是 C++ 里的 `struct` 或者 Python 里带 `@dataclass` 的类。**

#### 1. 怎么定义一个 Struct？

Struct 不能像 Map 那样随便创建，它**必须依附于一个模块（Module）**。模块的名字，就是这个 Struct 的名字。

```Elixir
defmodule User do
  # 使用 defstruct 定义这个结构体允许拥有哪些键，以及它们的默认值
  defstruct name: "匿名用户", age: 0, is_admin: false
end
```

#### 2. 怎么创建一个 Struct 实例？

创建 Struct 的语法和 Map 几乎一模一样，只是在 `%` 和 `{}` 之间**加上了模块的名字**。

Elixir

```
# 1. 使用默认值创建
user1 = %User{}
# 结果: %User{age: 0, is_admin: false, name: "匿名用户"}

# 2. 指定值创建
user2 = %User{name: "Gemini", age: 3}
# 结果: %User{age: 3, is_admin: false, name: "Gemini"}

# ❌ 3. 尝试塞入不存在的键（编译器会直接打手板，报错！）
# user3 = %User{name: "Bob", height: 180} 
# 报错: key :height not found in struct User
```

#### 3. Struct 的底层到底是什么？

记住这句话：**Struct 的本质就是一个 Map，只是它多了一个隐藏的叫 `__struct__` 的键。**

你可以用访问 Map 的方式去访问 Struct：

Elixir

```
# 访问数据 (因为 Struct 的键强制是原子 Atom，所以可以放心用点语法)
IO.puts(user2.name) # 输出 "Gemini"

# 更新数据 (和 Map 一模一样，返回一个新 Struct)
new_user = %{user2 | age: 4}
```

#### 4. 为什么要用 Struct 而不用 Map？ (结合模式匹配的杀手锏)

假设你写了一个函数，专门用来升级管理员权限。你希望这个函数**只能**接收 `User` 结构体，如果别人传进来一个普通的 Map，或者传了个 `Product` 结构体，你要立刻拒绝。

**在 Python 里：**

Python

```
def make_admin(user):
    if not isinstance(user, User):
        raise TypeError("必须是 User 对象")
    # ...
```

**在 Elixir 里（模式匹配大显神威）：**

Elixir

```
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

------

### 小结与测试

**总结：** Struct 就是有严格字段约束的 Map，它必须跟一个

- `Enum`（立即求值） vs `Stream`（惰性）
- `|>` 管道，把代码写成“数据流”
- 常用：`map/filter/reduce/group_by/frequencies`
