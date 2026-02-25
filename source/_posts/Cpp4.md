---
title: CPP4-简述RAII与智能指针
date: 2026-02-25
tag: [cpp]
category: [Software Engineering]
math: true
---

C++ 11 内存管理基础

<!--more-->


## RAII (Resource Acquisition Is Initialization)

RAII 的中文直译是“资源获取即初始化”，这个名字起得很糟糕，它真正的含义应该是：**“将资源的生命周期绑定到局部对象的生命周期上”**。

在 C++ 中，**栈（Stack）** 上的对象在离开作用域（大括号 `{}` 结束、函数返回、抛出异常）时，一定会**自动、确定性地**调用析构函数。

RAII 的做法就是：

1. 在类的**构造函数**中申请资源（如 `new` 内存、打开文件、加锁）。
2. 在类的**析构函数**中释放资源（如 `delete` 内存、关闭文件、解锁）。
3. **在栈上创建这个类的对象**。

这样，无论函数以什么方式退出（哪怕是中途抛出异常崩溃），栈展开（Stack Unwinding）机制都会保证析构函数被执行，资源被完美释放。**智能指针，就是 RAII 思想在堆内存管理上的具体实现。**

> 函数调用在内存中是用**栈（Stack）**来维护的。
>
> - **函数调用的本质：** 当你调用一个函数时，操作系统/CPU 会在栈顶为你分配一块连续的内存，叫做**栈帧（Stack Frame）**。你的局部变量（比如 `int a`, `std::unique_ptr p`）就按顺序排列在这块栈帧里。CPU 的一个寄存器（比如 x86 架构下的 `RSP` 栈指针寄存器）会向下移动，标记当前栈顶的位置。
> - **正常返回：** 当函数执行到 `return` 或者大括号 `}` 结束时，CPU 只需要把 `RSP` 寄存器“向上”移回调用前的位置，这块栈帧内存就被视为“释放”了（哪怕里面的数据还在，但随时会被下一个函数覆盖）。
> - **C++ 编译器的魔法（RAII 的底层）：** 仅仅移动指针是不够的，因为堆上的资源还没释放。C++ 编译器会在编译阶段，在函数 `return` 之前，或者在异常抛出导致栈帧弹出的路径上，**硬编码插入所有局部对象的析构函数调用指令**。
> - **栈展开（Stack Unwinding）：** 特指在抛出异常（Exception）时，程序会一层一层地往外跳，寻找 `catch` 块。在这个“向外跳”的过程中，操作系统和 C++ 运行时库会**逆序（后进先出）**销毁当前栈帧上的所有局部变量，逐个调用它们的析构函数，直到找到处理异常的地方。
>
> **总结：** 栈展开就是由编译器和运行时库共同配合的机制，确保在任何退出路径下，栈指针回退之前，对象的析构函数一定会被执行。



## 独占所有权：`std::unique_ptr` (常用)

`std::unique_ptr` 表达的是**独占（Exclusive Ownership）**。同一时刻，只能有一个 `unique_ptr` 指向那块内存。

- **性能：** 它是零开销抽象（Zero-overhead Abstraction）。在内存大小和运行效率上，它和裸指针（Raw Pointer）**完全一样**，没有任何额外开销。
- **规则：** **禁止拷贝，只允许移动**。



```C++
#include <memory>
#include <iostream>

class Widget {
public:
    Widget() { std::cout << "Widget Created\n"; }
    ~Widget() { std::cout << "Widget Destroyed\n"; }
    void doSomething() {}
};

void testUniquePtr() {
    // 现代 C++ 最佳实践：永远使用 std::make_unique (C++14 引入)
    // 绝对不要出现 new 关键字！
    std::unique_ptr<Widget> p1 = std::make_unique<Widget>();
    p1->doSomething();

    // std::unique_ptr<Widget> p2 = p1; // 编译报错！禁止拷贝，防止 Double Free

    // 必须使用移动语义，将所有权“偷”给 p2
    std::unique_ptr<Widget> p2 = std::move(p1); 
    
    // 此时 p1 变成了空指针 (nullptr)，p2 接管了内存。
    if (!p1) {
        std::cout << "p1 is now null\n";
    }
} // 函数结束，p2 离开作用域，自动 delete 堆上的 Widget 对象。
```


## 共享所有权：`std::shared_ptr`

当你真的需要多个地方共享同一个对象（比如一棵树的多个节点共享同一个配置对象），就需要用到 `std::shared_ptr`。

它内部使用的是**引用计数（Reference Counting）**机制：

1. 每次拷贝 `shared_ptr`，引用计数 +1。
2. 每次析构 `shared_ptr`，引用计数 -1。
3. 当引用计数降为 0 时，自动 `delete` 底层内存。

**底层实现与开销：**

`shared_ptr` 的体积是裸指针的 **2 倍**。它不仅包含一个指向实际对象的指针，还包含一个指向**控制块（Control Block）**的指针。控制块是在堆上动态分配的，里面存着强引用计数（Strong Count）和弱引用计数（Weak Count）。



```C++
void testSharedPtr() {
    // 现代 C++ 最佳实践：永远使用 std::make_shared
    // 原因：它可以把对象本身和控制块在堆上分配在一起（一次内存分配），提升性能，避免内存碎片。
    std::shared_ptr<Widget> sp1 = std::make_shared<Widget>(); // 引用计数 = 1
    {
        std::shared_ptr<Widget> sp2 = sp1; // 允许拷贝！引用计数变为 2
    } // sp2 离开作用域，引用计数 -1，变为 1，对象不销毁

} // sp1 离开作用域，引用计数 -1，变为 0，对象自动销毁
```

> 我们在谈论 `std::shared_ptr` 时，它之所以能做到多个指针共享状态，就是因为有控制块的存在。
>
> **控制块是什么？** 控制块是一小块**动态分配在堆（Heap）上的内存结构**。当你创建一个 `shared_ptr` 时，实际上有两块堆内存被分配：
>
> 1. 存储你实际对象（Widget）的内存。
> 2. 存储控制块的内存。
>
> **控制块里面装了什么？**
>
> 1. **强引用计数 (Strong Count)：** 记录有多少个 `shared_ptr` 指向该对象。降为 0 时，销毁 Widget 对象。
> 2. **弱引用计数 (Weak Count)：** 记录有多少个 `std::weak_ptr` 在观察这个对象。当强计数和弱计数**双双降为 0 时**，才真正释放控制块本身的内存。
> 3. **自定义删除器 (Custom Deleter)：** 可选。如果你管理的不是 `new` 出来的内存，而是比如一个文件句柄、一个网络 Socket，你可以把自己写的关闭函数存在这里。
> 4. **分配器 (Allocator)：** 记录用什么方式分配和回收内存。



## 避免循环引用：`std::weak_ptr`

既然 `shared_ptr` 这么好，为什么不全用它？因为引用计数有一个致命缺陷：**循环引用（Circular Dependency）**。

假设你写一个双向链表或图节点：

- 节点 A 有一个 `shared_ptr` 指向节点 B。

- 节点 B 有一个 `shared_ptr` 指向节点 A。

  这会导致 A 和 B 的引用计数永远降不到 0，发生内存泄漏。

**解决方案就是 `std::weak_ptr`：**

它就像是 `shared_ptr` 的“旁观者”。它可以观察 `shared_ptr` 管理的对象，但**不会增加引用计数**。当你要使用对象时，需要调用 `lock()` 方法将它临时提升为 `shared_ptr` 来确保对象存活。



## 经典问题

### 关于裸指针与智能指针的混用

下面这段代码会发生什么严重后果？为什么？

```C++
Widget* raw_p = new Widget();
std::shared_ptr<Widget> sp1(raw_p);
std::shared_ptr<Widget> sp2(raw_p);
```

程序会直接 Crash（崩溃）。 因为 `sp1` 和 `sp2` 是**分别**用 `raw_p` 初始化的，它们根本不知道对方的存在。它们会各自在堆上创建一个“控制块”（Control Block），且引用计数都是 1。 当离开作用域时，`sp1` 发现计数清零，执行 `delete raw_p`；接着 `sp2` 也发现计数清零，**再次执行 `delete raw_p`**。这就是臭名昭著的 **Double Free（重复释放）** 漏洞，会导致严重的内存损坏和安全问题。

**注意：** 永远不要用同一个裸指针初始化多个智能指针。直接用 `std::make_shared` 一步到位。

> **`std::make_shared` 到底好在哪里？** 如果你写：`std::shared_ptr<Widget> p(new Widget());`
>
> - 操作系统要进行**两次**动态内存分配（极度消耗性能，容易产生内存碎片）：一次给 Widget，一次给控制块。 如果你写：`std::shared_ptr<Widget> p = std::make_shared<Widget>();`
> - 编译器会计算出 Widget 和控制块的总大小，然后只呼叫**一次**内存分配，把控制块和 Widget 放在一起（紧凑的连续内存）。这不仅快，而且对 CPU 缓存（Cache Line）极其友好！

### 关于函数传参的性能

假设有一个类对象非常大，并且被 `shared_ptr` 管理。需要把它传给一个只读（不修改它，也不保存它）的函数。以下哪种传参方式最好？为什么？

A. `void printWidget(std::shared_ptr<Widget> p);` (按值传递)

B. `void printWidget(const std::shared_ptr<Widget>& p);` (按常量引用传递)

显然是 B。A 的开销不仅在于多复制了几个字节，更核心的是：`shared_ptr` 的拷贝会触发引用计数的 +1，析构时触发 -1。而为了保证线程安全，这个计数的加减是**原子操作（Atomic Operations）**。频繁的原子操作会触发 CPU 缓存一致性协议同步，造成严重的性能下降。按常引用（`const std::shared_ptr<Widget>&`）传递，底层只是传了一个内存地址，0 拷贝，0 原子操作。

### 关于多线程与线程安全

`std::shared_ptr` 的**引用计数**的增减操作是线程安全的吗？它管理的**底层对象（如上面代码中的 Widget）**本身是线程安全的吗？

`std::shared_ptr` 的**控制块（引用计数）是绝对线程安全的**（内部使用了无锁的原子操作）。你可以安全地在多个线程中拷贝同一个 `shared_ptr`。 但是，它**管理的那个对象本身（Widget）绝对不是线程安全的**。如果多个线程同时通过 `shared_ptr` 去修改 Widget 内部的数据，依然会发生 Data Race（数据竞争），必须自己加锁（如 `std::mutex`）。

