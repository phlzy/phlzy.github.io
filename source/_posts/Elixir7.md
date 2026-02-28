---
title: Elixir7-GenServer与Supervisor
date: 2026-02-27
tag: [Elixir]
category: [Software Engineering]
math: true
---

Elixir 的OTP 入门（GenServer + Supervisor）

<!--more-->



先来学习 `GenServer` (Generic Server)。

## GenServer

注意： **一个 GenServer 模块里的代码，是跑在两个不同的进程里的！**

为了方便管理，我们通常把这两部分代码写在**同一个文件**里：

1. **Client API（服务员/前台）：** 跑在**调用者**的进程里。负责对外接客，把客人的请求打包成标准信件（发消息）。
2. **Server Callbacks（大厨/后厨）：** 跑在**GenServer 自己的独立进程**里。负责拆信、处理数据、更新状态。

### 实战

来看看用 `GenServer` 写的计数器有多么干净。注意看注释里的“前台”和“后厨”：


```Elixir
defmodule Counter do
  # 这行魔法代码会帮你自动处理隐藏的 receive 死循环和各种底层逻辑
  use GenServer

  # ==========================================
  # 前台 API (Client API) - 跑在调用者的进程
  # ==========================================

  # 1. 开店招人（启动独立进程）
  def start_link(initial_value) do
    # __MODULE__ 就是当前的模块名 Counter
    GenServer.start_link(__MODULE__, initial_value, name: __MODULE__)
  end

  # 2. 异步请求："只管发指令，不傻等结果" (Cast)
  def increment do
    # 服务员把 :inc 这张纸条塞进后厨的信箱
    GenServer.cast(__MODULE__, :inc)
  end

  # 3. 同步请求："发指令，且必须拿到结果才走" (Call)
  def get_count do
    # 服务员把 :get 这张纸条塞给后厨，并站着等回信
    GenServer.call(__MODULE__, :get)
  end


  # ==========================================
  # 后厨回调 (Server Callbacks) - 跑在独立的计数器进程
  # ==========================================

  # 1. 初始化厨房 (对应 start_link)
  @impl true
  def init(initial_value) do
    # 必须返回 {:ok, 初始状态}
    {:ok, initial_value}
  end

  # 2. 处理异步信件 (对应 cast)
  @impl true
  def handle_cast(:inc, state) do
    # 收到 :inc，把状态 + 1
    # 必须返回 {:noreply, 新的状态}
    new_state = state + 1
    {:noreply, new_state}
  end

  # 3. 处理同步信件 (对应 call)
  @impl true
  def handle_call(:get, _from, state) do
    # 收到 :get，把当前状态发回去
    # 必须返回 {:reply, 回复给客人的内容, 新的状态}
    # 这里状态不变，所以回复 state，保持的新状态也是 state
    {:reply, state, state}
  end
end
```

### 如何使用

有了前台 API，外部调用者根本不需要知道底层是不是在发消息，用起来就像调用普通的 Python 类方法一样自然：



```Elixir
# 1. 启动计数器（把它挂在后台）
Counter.start_link(0)

# 2. 疯狂增加访问量（异步，瞬间执行完，不阻塞主程序）
Counter.increment()
Counter.increment()
Counter.increment()

# 3. 获取当前访问量（同步，必须等后厨把结果发回来）
current = Counter.get_count()
IO.puts("当前访问量是：#{current}") # 输出 3
```

### 3 个硬核格式

你发现后厨（Callbacks）返回的格式非常严格了吗？这是有极大好处的。

- **`{:ok, state}`**：告诉系统“我准备好了，这是我的初始状态”。如果这里返回 `{:stop, reason}`，进程根本就不会启动。
- **`{:noreply, new_state}`**：专门用于 `handle_cast`（异步）。意思是“我干完活了，**不需要回信**，这是我更新后的状态，请帮我存好，准备接下一单”。
- **`{:reply, response, new_state}`**：专门用于 `handle_call`（同步）。意思是“我干完活了，**把 response 发给门外等着的客人**，同时把我的内部状态更新为 new_state”。

**这彻底解决了手动写 `receive` 容易出错的问题！** 你不需要自己写死循环，不需要自己传参数，你只要按照 `GenServer` 的格式填空，状态的保存和流转，OTP 框架在底层全帮你做好了。

### 总结：Call vs Cast

这是 GenServer 日常开发中最核心的决策：

1. **`GenServer.call` (同步)**：我需要数据，或者我必须确认操作成功了才能继续往下走。它自带 5 秒超时（还记得上节课防死锁的那个闹钟吗？`call` 底层自动帮你加了）。
2. **`GenServer.cast` (异步)**：我只要下达命令，不在乎结果，也不想等（比如发邮件、写访问日志）。极致的高并发。



## Supervisor

之前我们实现了一个完美的 `Counter`（计数器大厨）。 但是，万一有人发了一封“毒信件”（比如让大厨去除以零），导致这个大厨崩溃了（进程死亡），我们的计数服务不就彻底宕机了吗？

接下来**只需用 5 行代码**，写一个 `Supervisor`（经理），把这个大厨死死盯住，让他拥有“崩溃瞬间满血复活”的超能力。



### 代码

基于 `Counter` 模块：



```elixir
# 1. 定义经理的“花名册”（告诉他手下有哪些大厨需要盯）
children = [
  {Counter, 0}  # 意思是：你要监控 Counter 模块，且每次重启它时，初始状态设为 0
]

# 2. 经理正式上岗！
{:ok, sup_pid} = Supervisor.start_link(children, strategy: :one_for_one)

IO.puts("经理已就位，计数器大厨已被保护！")
```

原理：

1. **`children` 列表：** 这就是经理的管理清单。你可以往里面塞十几个不同的大厨，经理会按照顺序把他们一个个叫醒（启动）。
2. **`Supervisor.start_link`：** 启动这个经理进程。同时，经理会在底层自动给所有的大厨穿上“防弹衣”（我们之前讲的 `Trap Exit` 和 `Link`）。
3. **`strategy: :one_for_one`（一对一策略）：** 这是最核心的容错策略！ 它的意思是：**“如果清单里某一个大厨挂了，不要惊慌，只把挂掉的那一个大厨原地重启，不要影响其他正在干活的大厨。”**

*(注：除了 `one_for_one`，Elixir 还有 `one_for_all`（一人犯错，全组处决并重启）等策略，用于处理强依赖关系。)*

### 测试

试一下，直接把大厨杀掉，看看会发生什么。



```elixir
# 1. 正常使用大厨
Counter.increment()
IO.puts(Counter.get_count()) # 输出 1

# 2. 找出大厨在操作系统底层的真实 PID (进程ID)
# 因为我们之前给进程起了个名字叫 Counter
chef_pid = Process.whereis(Counter) 
IO.puts("大厨的真实 PID 是: #{inspect(chef_pid)}")

# 3. 物理超度！直接给这个进程发送必定致死的击杀信号
Process.exit(chef_pid, :kill)
IO.puts("砰！大厨被爆头了！")

# 先等个几毫秒，让子弹飞一会儿...
Process.sleep(10) 

# 4. 奇迹发生：我们再去尝试调用计数器
IO.puts(Counter.get_count()) # 竟然输出了 0！没有报错！

# 5. 我们再查一下现在大厨的真实 PID
new_chef_pid = Process.whereis(Counter)
IO.puts("新大厨的 PID 是: #{inspect(new_chef_pid)}") 
# 你会发现，PID 变了！这是一个全新的大厨！
```

**发生了什么？** 在你开枪的一瞬间，旧的 `chef_pid` 死了。但经理（Supervisor）在 1 微秒内收到了死亡通知，经理看了一眼花名册：哦，是 `Counter` 死了。于是经理立刻掏出花名册里的初始配方 `{Counter, 0}`，重新 `start_link` 了一个全新的大厨。 当你再次调用 `Counter.get_count()` 时，请求被无缝路由到了新大厨的手里！

在 C++/Python 里，你可能得写脚本去监控进程，死了再重新拉起来。 但在 Elixir 里，进程的监控和重启是**语言原生提供**的。

### Liveness (存活) vs State (状态)

如果你足够敏锐，你一定会发现一个“致命”问题： **“等一下！大厨虽然复活了，但我之前的访问量变成 0 了啊！我的数据丢了！”**

没错。这就是 Elixir 容错哲学的终极奥义： **Supervisor 保障的是“服务的可用性”（Liveness），绝对不保障“内存状态的持久化”（State）。**

- 经理只管保证你的餐厅**永远有厨师在炒菜，永远不关门**。
- 至于大厨上一锅炒到一半的菜？对不起，直接倒进垃圾桶。

**如果我们要保证数据不丢怎么办？** 在工业界，这就是分工的意义：

1. **短暂状态 / 可丢弃状态：** 放在 `GenServer` 内存里（比如用户的临时 WebSocket 连接状态、游戏里的实时坐标），崩了就崩了，大不了重新连。
2. **关键核心数据：** `GenServer` 在处理完业务后，必须写进 **数据库（PostgreSQL 等）**，或者写进 Elixir 自带的**内存级超快数据库（ETS - Erlang Term Storage）**。

当大厨重启时，他的 `init` 函数里不应该写死返回 0，而是应该去数据库里把上一次的进度读出来，然后继续干活。

**只要把这些 `Supervisor` 一层一层嵌套起来（大经理管小经理，小经理管大厨），这就是所谓的 Supervision Tree（监控树）。** 而整个由监控树构成的巨型应用，就是我们常说的 **OTP Application**。

