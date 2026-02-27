---
title: CPP6-匿名函数基础
date: 2026-02-27
tag: [cpp]
category: [Software Engineering]
math: true
---

简单复习一下 Lambda

<!--more-->


## 匿名函数并不是一个函数

C++ 的匿名函数不是一个函数，而是一个仿函数，一个重载了 `operator()` 的类。举个例子：

```C++
int x = 10;
int y = 20;
auto f = [x, &y](int z) { 
    return x + y + z; 
};
```

编译器生成的类似下面这样一个 `struct`：

```C++
// 编译器偷偷生成的类（类名是编译器随便起的乱码，你无法知道）
class __Lambda_Uniquely_Generated_Name_123 {
private:
    int x;       // 因为你用了按值捕获 [x]
    int& y;      // 因为你用了按引用捕获 [&y]

public:
    // 构造函数，用来初始化捕获的变量
    __Lambda_Uniquely_Generated_Name_123(int _x, int& _y) : x(_x), y(_y) {}

    // 重载了 operator() 函数，让你能像调用函数一样调用它
    // 注意：默认情况下它是 const 的！
    int operator()(int z) const {
        return x + y + z;
    }
};

// 你的 auto f = ... 其实等价于：
__Lambda_Uniquely_Generated_Name_123 f(x, y);
```

> 在面向对象设计中，仿函数完美契合了**策略模式 (Strategy Pattern)** 和 **命令模式 (Command Pattern)**。
>
> - 当你需要把一个“动作”当成参数传递，并且这个动作还需要**自带状态**（比如携带一些成员变量）时，普通的函数指针根本做不到。
> - 仿函数本质上就是**“自带状态的函数”**。Lambda 表达式只不过是 C++ 编译器帮你自动写仿函数的一种“语法糖”而已。



## `std::function`：类型擦除的代价

既然 Lambda 只是编译器偷偷生成的匿名类，而且**每一个 Lambda 表达式的类型都是独一无二的**（就算捕获列表和代码完全一样，编译器也会生成两个不同的类名），那问题来了：

**如果你要把一个 Lambda 存进 `std::vector` 里，或者把它作为类的成员变量，你怎么写类型？** 你不能写 `std::vector<auto>`，因为 C++ 必须在编译期确定容器的明确大小。

为了解决这个问题，C++11 引入了万能的多态包装器：**`std::function`**。 它可以装下任何签名匹配的东西：普通函数指针、Lambda、重载了 `operator()` 的类，甚至类的成员函数。



```C++
#include <functional>

// 它的类型签名非常清晰：返回 int，接收两个 int 参数
std::function<int(int, int)> my_func;

my_func = [](int a, int b) { return a + b; }; // 装入 Lambda
my_func = std::plus<int>();                   // 装入标准库仿函数
```

**注意：`std::function` 的性能开销极高！** 既然 `std::function` 能装下各种乱七八糟、大小不一的对象，那么，代价是什么？ 它的底层使用了**“类型擦除 (Type Erasure)”**技术，代价是：

1. **动态内存分配（Heap Allocation）：** 如果你传入的 Lambda 捕获了很多变量，导致它的匿名类体积太大，`std::function` 内部会偷偷呼叫 `new`，在堆上分配内存来存你的 Lambda。（不过现代标准库有 SOO 小对象优化，如果捕获很少，就不需要 `new`）。
2. **阻断内联优化（Inline）且包含虚函数调用：** `std::function` 内部其实维护了一套极其复杂的面向对象多态机制（类似于虚函数）。当你调用 `my_func()` 时，CPU 会进行一次间接跳转（Indirect Branch）。对于那些需要在 `for` 循环里执行上亿次的极小函数，千万不要用 `std::function`，会慢几个数量级！

## `std::bind`：时代的眼泪

**不要再使用 `std::bind`，使用 Lambda 替代！**

先来看看 `std::bind` 是什么：假如时光倒流回 C++98 时代，你有一个 `std::vector<int> v`，你想用标准库的 `std::count_if` 统计里面**大于 5** 的数字个数。 `std::count_if` 需要你传一个接收一个参数并返回 `bool` 的回调函数。但在 C++98 里，根本没有 Lambda 表达式，你只能**手写一个仿函数**：

```c++
// 为了这么个简单的逻辑，你得在外面写一大坨代码：
struct GreaterThan {
    int limit;
    GreaterThan(int l) : limit(l) {}
    bool operator()(int x) const { return x > limit; }
};

int count = std::count_if(v.begin(), v.end(), GreaterThan(5));
```

这太恶心了！为了避免发生这种情况，C++ 委员会在标准库里提供了一些基础运算组件（比如 `std::greater<int>()`，它是个收两个参数的仿函数，执行 `a > b`），并且引入了 `bind1st`, `bind2nd`，以及后来的 **`std::bind`**。

有了 `bind`，哪怕没有 Lambda，你也可以这么写（不用手写结构体了）：

```c++
// 意思是：拿一个“大于”函数，把它的第二个参数“绑定”为 5。
// 这样它就变成了一个只需要一个参数的新函数！
int count = std::count_if(v.begin(), v.end(), std::bind(std::greater<int>(), std::placeholders::_1, 5));
```

但是这仍然很恶心：`std::bind` 语法实在是太怪了，尤其是那个占位符 `std::placeholders::_1`。

好在现在有了 Lambda，只需要写：

```c++
int count = std::count_if(v.begin(), v.end(), [](int x) { return x > 5; });
```

所以，一般来说没必要再用奇怪的 `std::bind` 了，除了看起来很丑陋，它还有下面的缺陷：

1. 它很难处理重载函数。
2. 它在内部强制按值拷贝，很难精确控制引用。
3. **Lambda 编译器能完美内联展开，`std::bind` 通常无法内联，性能不如 Lambda。**



## 经典问题

### 关于悬垂引用

下面这个代码有什么问题？

``` c++
auto makeLambda() {
    int local_val = 100;
    return [&local_val]() { std::cout << local_val << "\n"; }; 
}
```

Lambda 在物理上只是个存了指针/引用的 `struct`，这里的 Lambda 引用捕获了局部变量，离开函数时，`local_val` 所在的那块栈内存已经被回收了。因此，返回的 Lambda 里的 `int&` 成员变量，变成了一个野指针/野引用。

这就是常见 Bug：**悬垂引用 (Dangling Reference)**。

### 关于 mutable 的物理含义

假设我写了下面这段代码，编译器会报错说不能修改 `x`：

```C++
int x = 0;
auto f = [=]() { x++; }; // 编译报错！
```

为了让它能编译，我们需要加上 `mutable` 关键字：`auto f = [=]() mutable { x++; };`。 

为什么会编译报错？`mutable` 起到了什么作用？

从底层来看，当你写按值捕获的 Lambda `[=]() { x++; }` 时，编译器生成的匿名类的 `operator()` 默认是带有 `const` 修饰符的（即 `int operator()() const { ... }`）。

> **为什么 C++ 要默认把它设为 const？** 因为按值捕获代表你拷贝了一份**副本**。C++ 委员会认为，如果你在一个看似普通的函数里修改了副本，而外部的原本却没有改变，这极易引发逻辑 Bug。所以干脆默认禁止修改副本。

 当你在 Lambda 上加上 `mutable` 时，编译器仅仅是**去掉了 `operator()` 后面的那个 `const` 关键字**。此时你就可以合法地修改匿名类内部的成员变量副本了。

> 被 mutable 修饰的变量（mutable 只能用于修饰类中的非静态数据成员），将永远处于可变的状态，即使在一个 const 函数中。

### 关于性能抉择 

既然 `std::function` 有性能损耗，假设我在写一个需要极高效率的排序算法框架，需要接收一个比较函数作为参数。下面两种写法，在工业界追求极致性能时，应该选哪种？为什么？

```C++
// 写法 A：使用 std::function
void mySortA(std::vector<int>& v, const std::function<bool(int, int)>& cmp);

// 写法 B：使用模板
template <typename Compare>
void mySortB(std::vector<int>& v, Compare cmp);
```



使用模板更好，编译期确定，多态是运行期开销大。使用模板时，编译器会进行**单态化 (Monomorphization)**，也就是为你传入的每一个不同的 Lambda，专门生成一份独立的 `mySortB` 机器码。因为函数地址在编译期是绝对已知的，编译器会把 Lambda 的代码**直接内联 (Inline) 展开**到 `for` 循环里，函数调用的开销降为 **0**。而 `std::function` 涉及到指针间接寻址，不仅有调用开销，还会彻底阻断编译器的内联优化。

