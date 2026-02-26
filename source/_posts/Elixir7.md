---
title: Elixir7-OTP入门
date: 2026-02-27
tag: [Elixir]
category: [Software Engineering]
math: true
---

Elixir 的OTP 入门（GenServer + Supervisor）

<!--more-->

重点（只抓最常用的 20%）：

- GenServer 的 `init/1`、`handle_call/3`、`handle_cast/2`
- state 的建模方式（就是状态机）
- Supervisor 的 child spec、重启策略（one_for_one 足够你入门）

练习（非常典型，也最能体现价值）：

- `KVStore` GenServer：`put/get/delete`
- Supervisor 启动多个 KV 分片：根据 key hash 路由到不同 GenServer

Supervision Tree
