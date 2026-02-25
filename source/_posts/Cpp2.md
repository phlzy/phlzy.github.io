---
title: CPP2-面向对象与虚函数
date: 2026-02-10
tag: [cpp]
category: [Software Engineering]
math: true
---

C++ 编译器到底是怎么实现多态的？

<!--more-->


## 多态

C++ 的多态分为两种：

- **静态多态（编译期）：** 函数重载（Overloading）和模板（Templates）。你在编译时就知道要调用哪个函数，速度极快。
- **动态多态（运行期）：** 通过**基类指针或引用**去调用**派生类**重写的函数。编译器在编译时**根本不知道**这个指针究竟指向什么对象，必须等到程序运行到那一刻才能决定。

为了在运行时“动态”找到正确的函数，C++ 编译器在底层悄悄引入了两个机制：**vtbl** 和 **vptr**。

## Virtual Table

**虚函数表是属于“类（Class）”层面的。**

只要你的类里声明了哪怕一个 `virtual` 函数，C++ 编译器就会在这个编译单元的**只读数据段（.rodata）为这个类偷偷生成一个一维数组。这个数组里面存的，就是这个类所有虚函数的内存地址（函数指针）**。

- **基类有自己的 vtbl。**
- **派生类有自己的 vtbl。** 如果派生类重写（Override）了基类的虚函数，派生类 vtbl 中对应的函数指针就会被**替换**为派生类自己的函数地址。

## Virtual Pointer

**虚表指针是属于“对象（Object / Instance）”层面的。**

为了让对象能找到自己对应的虚函数表，编译器会偷偷修改这个类的内存布局：在实例化每一个对象时，在对象的内存模型最开头（通常是起始位置，偏移量为 0），塞入一个隐藏的指针，这就是 **vptr**。

**工作流程（动态绑定的底层本质）：**

假设你有 `Base* ptr = new Derived();` 并调用 `ptr->speak();`（`speak` 是虚函数）。

1. **取指针：** 编译器先通过 `ptr` 找到该对象所在的内存。
2. **找虚表：** 读取该对象内存最开头的隐藏变量 `vptr`，顺藤摸瓜找到属于 `Derived` 类的 `vtbl`。
3. **查表：** 在 `vtbl` 这个数组里，找到 `speak()` 函数对应的索引位置，提取出里面的函数指针。
4. **执行：** 跳转到那个地址去执行代码。

这就解释了为什么“动态多态”会慢一点——因为它比普通函数调用多了**两次寻址（指针解引用）操作**。



### 代码演示与内存验证

我们来看一个直观的代码，感受一下 `vptr` 的存在：

```C++
#include <iostream>

// 普通类：没有虚函数
class NoVirtual {
    int a;
    int b;
};

// 多态类：包含虚函数
class WithVirtual {
    int a;
    int b;
    virtual void speak() {}
};

int main() {
    // 假设在 64 位系统下：int 占 4 字节
    std::cout << "Size of NoVirtual: " << sizeof(NoVirtual) << " bytes\n"; 
    // 输出: 8 bytes (4 + 4)

    std::cout << "Size of WithVirtual: " << sizeof(WithVirtual) << " bytes\n";
    // 输出: 16 bytes (8 字节的 vptr + 4 + 4) 
    // 对象的体积因为多了一个隐藏的 64位指针 而变大了！
    
    return 0;
}
```

### 虚析构函数 (Virtual Destructor)

**注意：只要一个类打算被用作基类（有虚函数），它的析构函数就必须声明为 `virtual`！**

如果你通过一个基类指针来删除一个派生类对象，而基类的析构函数不是虚函数，就会发生局部销毁：程序只会调用基类的析构函数，而不会调用派生类的析构函数。如果派生类在构造时申请了额外内存，就会产生内存泄漏。

> 当基类析构函数没有声明为 `virtual` 时，编译器会采用**静态绑定**，导致删除操作只看指针类型，而不看对象实际类型。

观察下面的例子：

```C++
class Base {
public:
    Base() { /* ... */ }
    ~Base() { std::cout << "Base Destroyed\n"; } // 忘记写 virtual 了！
};

class Derived : public Base {
    int* huge_array;
public:
    Derived() { huge_array = new int[10000]; }
    ~Derived() { 
        delete[] huge_array; 
        std::cout << "Derived Destroyed\n"; 
    }
};

int main() {
    Base* ptr = new Derived();
    delete ptr; 
    // 因为 Base 的析构函数不是虚函数，编译器采用“静态绑定”。
    // 它只看指针的类型是 Base*，于是只调用了 Base::~Base()。
    // Derived 的析构函数根本没执行，huge_array 直接内存泄漏！
    return 0;
}
```



## 经典问题

### 关于构造函数

析构函数通常要是虚函数，那么**构造函数可以是虚函数吗？** 为什么？

构造函数不能是虚函数，虚函数调用依赖于对象的 vptr（虚函数指针）。而 vptr 是在构造函数执行期间才被初始化的。如果构造函数本身是虚的，那么在调用它之前，对象还没初始化完成，也就没有 vptr，程序根本无法正常查找虚表。只有当构造函数开始执行时，编译器才会悄悄把 `vptr` 塞进去并赋值。

> 构造函数的目的是创建一个对象。在创建对象时，你必须明确知道你要创建的是哪个类。虚函数是为了在“不知道具体类型”的情况下实现多态，这与“创建一个特定类型对象”的初衷相悖。
>
> 既然构造函数不能是虚的，那我如何实现根据输入动态创建对象？考虑工厂模式。



### 关于危险的操作

如果我在一个类的**构造函数**或者**析构函数**里面，调用了一个虚函数，会发生多态吗？（例如，Base 的构造函数里调用了虚函数 `speak()`，由于我在 `new Derived()`，它会去执行 Derived 重写的 `speak()` 吗？）

**注意：这种情况绝对不会发生多态！**

**为什么？（极其核心的底层逻辑）：** 想象一下对象的构造顺序是从基类到派生类：`Base` 构造 -> `Derived` 构造。 当程序正在执行 `Base` 的构造函数时，`Derived` 的那部分成员变量**根本还没有被初始化**！如果这时候触发了多态，调用了 `Derived` 重写的虚函数，而这个虚函数里碰巧访问了 `Derived` 特有的指针或数组，程序就会直接崩溃（访问未初始化内存）。

**编译器是怎么实现“屏蔽多态”的？** 这是一个非常惊艳的设计：**`vptr` 的指向是动态变化的！**

1. 当进入 `Base` 构造函数时，编译器把 `vptr` 指向 `Base` 的虚函数表。此时不管你调什么虚函数，都只能调到 `Base` 自己的。
2. `Base` 构造完成后，进入 `Derived` 构造函数，编译器再次偷偷修改 `vptr`，让它指向 `Derived` 的虚函数表。
3. 析构函数的顺序反过来，`vptr` 也会跟着“退化”。

### 关于纯虚函数与抽象类

如果我在基类中写了 `virtual void doSomething() = 0;`（纯虚函数），这个类的实例化规则是什么？它的虚函数表里存的是什么？

包含纯虚函数（形如 `virtual void func() = 0;`）的类叫**抽象类（Abstract Class）**，绝对**不能被实例化**。 而在它的虚函数表（vtbl）中，那个对应的函数指针通常**不是真正的空指针 (`nullptr`)**，而是指向一个由编译器提供的特殊错误处理函数（比如 GCC 下的 `__cxa_pure_virtual`）。如果因为某些极其极端的野指针操作强行调用到了它，程序会直接打印错误并 abort（终止），而不是引发不可预知的段错误。

## 纯虚函数

在工业界，纯虚函数最大的作用是**定义接口（Contract / Interface）**。 C++ 没有像 Java 或 C# 那样的 `interface` 关键字，所以 C++ 工程师约定俗成：**如果一个类没有任何成员变量，且所有函数都是纯虚函数，那它就是一个接口类。**

```C++
// 这是一个纯粹的接口
class ILogger {
public:
    virtual ~ILogger() = default; // 永远记得虚析构！
    virtual void log(const std::string& msg) = 0; // 纯虚函数
};

// 派生类必须重写（实现）纯虚函数，否则它自己也会变成抽象类
class FileLogger : public ILogger {
public:
    void log(const std::string& msg) override {
        // 写入文件的具体实现
    }
};
```

## 多重继承与菱形继承 (The Diamond Problem)

早期 C++ 库或复杂的业务线中偶尔会遇到这类情况。

### 多重继承的内存布局

如果类 `C` 同时继承了类 `A` 和类 `B`，那么 `C` 的对象内存里会有**两个** `vptr`！

当你把 `C` 类型的指针转换成 `A` 类型或 `B` 类型的指针时，编译器会在底层自动帮你做**指针偏移（Pointer Adjustment）**。这也是为什么在 C++ 中随意使用 C 风格的强转 `(B*)c_ptr` 是极其危险的，必须使用 `static_cast` 或 `dynamic_cast`，让编译器帮你算好偏移量。

### 菱形继承 (Diamond Inheritance)

这是多重继承带来的灾难。假设：

- `Base` 是最底层的基类（有一个成员变量 `int data`）。
- `Left` 和 `Right` 都继承自 `Base`。
- `Bottom` 同时继承自 `Left` 和 `Right`。

**痛点：** 此时，`Bottom` 的对象内存中，会包含**两份** `Base` 的拷贝！一份在 `Left` 里，一份在 `Right` 里。当你尝试访问 `bottom_obj.data` 时，编译器会报错：**二义性（Ambiguity）**，它不知道你要访问哪一个 `data`。

### 虚继承 (Virtual Inheritance)

为了解决菱形问题，C++ 引入了**虚继承** `virtual public`。



```C++
class Base { public: int data; };
class Left : virtual public Base {};  // 注意 virtual 关键字
class Right : virtual public Base {}; // 注意 virtual 关键字
class Bottom : public Left, public Right {};
```

**底层魔法：`vbptr` (Virtual Base Pointer)** 当你使用了虚继承，编译器会改变对象的内存布局：

1. `Left` 和 `Right` 内部不再直接包含 `Base` 的数据，而是多了一个隐藏指针叫 **`vbptr`（虚基类表指针）**。
2. 这个指针指向一张表，表里记录了真正的 `Base` 数据相对于当前位置的**偏移量**。
3. 这样，`Bottom` 实例化时，内存里就**只有唯一的一份** `Base` 数据，`Left` 和 `Right` 都是通过各自的 `vbptr` 加上偏移量去共享这一份数据。

*(注：由于多重继承和虚继承会导致内存布局极度复杂且降低性能，现代工业界的最佳实践是：**尽量避免多重继承，如果非要用，只能多重继承“纯虚函数接口类”**。)*

### 二进制不兼容（Binary Incompatibility）

C++ 底层最复杂、最容易引发“跨平台兼容性灾难”的终极黑洞——**ABI（Application Binary Interface，应用程序二进制接口）差异**。

> **C++ 标准委员会（ISO C++）从来没有规定过对象的内存到底该怎么排布！** 标准只规定了“行为”（比如多态应该怎么运作），但具体怎么在内存里塞入这些数据和指针，完全交由编译器自己决定。

目前世界上主要有两大流派，它们在简单情况下（单继承）内存布局差不多，但一碰到**多重继承**和**虚继承**，两者就会分道扬镳：

#### MSVC vs GCC

- **MSVC 阵营（微软 Visual Studio）：** 遵循 Microsoft C++ ABI。它的设计理念是“各司其职”，倾向于增加专用的指针来解决复杂问题。
- **GCC / Clang 阵营（Linux / macOS / Android 默认）：** 遵循 Itanium C++ ABI。它的设计理念是“极简与复用”，尽量压缩对象体积，把复杂的逻辑全塞进虚函数表（vtable）里。

#### 虚继承（Virtual Inheritance）怎么实现？

这就是 GCC 和 MSVC 分歧最大的地方。假设有一个类 `Derived` 虚继承自 `Base`：

**在 MSVC 下（微软的做法）：**

微软觉得既然是虚继承，那我就专门给对象加一个新指针。

- 对象内存里会有一个 `vptr`（虚函数表指针，用来找虚函数）。
- **还会多出一个专门的 `vbptr`（Virtual Base Pointer，虚基类表指针）**。
- 这个 `vbptr` 指向一张全新的表（`vbtable`），表里面存着真正的 `Base` 数据在内存中的偏移量。
- **结果：** 对象的体积变大了，因为多了指针。

**在 GCC / Clang 下（Itanium ABI 的做法）：**

GCC 觉得微软那样太浪费内存了！一个对象塞那么多指针干嘛？

- 对象内存里**只有**一个 `vptr`，**没有**所谓的 `vbptr`。
- 那怎么找虚基类的偏移量呢？GCC 的做法极其硬核：它把虚基类的偏移量（vbase_offset）记录在了**虚函数表（vtbl）的“负索引”位置**！
- 也就是说，`vptr` 指向虚函数表的正中间。往下读（正索引）是虚函数指针，往上读（负索引）就是虚基类偏移量和类型信息（RTTI）。
- **结果：** 对象体积更小，但编译器在底层生成的寻址代码更加复杂。

#### 不兼容的后果

这个内存布局的差异，导致了一个工业界极其头疼的问题：**二进制不兼容（Binary Incompatibility）**。

假设你所在的公司有一个核心的算法库，是用 MSVC 在 Windows 下编译成 DLL 的，里面的接口类用到了虚函数或 STL（比如 `std::string` 在不同编译器下的内存布局也完全不同）。

现在，客户用 MinGW（也就是 Windows 下的 GCC）写了个程序，试图去调用你这个 DLL 里的 C++ 对象。

- **结局：** 客户的程序瞬间崩溃（Segmentation Fault）。因为 GCC 编译出的程序，以为对象的偏移量是按 Itanium ABI 排布的，结果跑到 MSVC 编译的内存里一读，全读到了错乱的数据。

**工业界解决方案：**

为了跨越编译器的 ABI 鸿沟，C++ 工程师在提供跨平台动态链接库时，通常会彻底抛弃 C++ 的面向对象特性，**把接口全部扁平化为纯 C 接口（`extern "C"`）**，或者使用更底层的 COM（组件对象模型）技术。因为 C 语言的内存布局（struct）在所有编译器下都是完全一致的！



