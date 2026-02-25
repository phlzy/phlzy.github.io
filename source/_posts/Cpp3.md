---
title: CPP3-简述右值引用与move
date: 2026-02-25
tag: [cpp]
category: [Software Engineering]
math: true
---

C++ 11 核心概念：右值引用（Rvalue Reference）与移动语义（Move Semantics）

<!--more-->

## Lvalue & Rvalue

首先区别左值和右值：

- **左值 (Lvalue)**：有名字，有内存地址，能用 `&` 取地址。它的生命周期长于当前表达式。
- **右值 (Rvalue)**：没有名字，没有（持久的）内存地址，通常是临时的字面量或表达式求值产生的临时对象。



```C++
int a = 10;           // a 是左值，10 是右值
int b = a;            // a 和 b 都是左值
int c = a + b;        // c 是左值，(a + b) 产生的结果是一个临时对象，是右值

std::string s1 = "Hello";
std::string s2 = s1 + " World"; // s1 和 s2 是左值，(s1 + " World") 是右值
```

### 右值引用 `&&`

在 C++11 之前，只有左值引用（`T&`）和 常量左值引用（`const T&`）。

当一个巨大的临时对象被赋值给另一个变量时，C++ 只能调用**拷贝构造函数**，把这个临时对象复制给变量，然后销毁本体。这在性能上是极大的浪费。

**右值引用 (`T&&`)** 的出现，就是为了**专门绑定到临时对象（右值）上**，避免复制的额外开销。

### 移动构造函数（Move Constructor）

我们来看一个简化的 `MyVector` 例子：

```C++
#include <iostream>

class MyVector {
private:
    int* data;
    size_t size;

public:
    // 1. 普通构造函数
    MyVector(size_t n) : size(n), data(new int[n]) {
        std::cout << "Constructed\n";
    }

    // 2. 析构函数
    ~MyVector() {
        delete[] data;
    }

    // 3. 传统：拷贝构造函数 (深拷贝 - O(N) 昂贵操作)
    MyVector(const MyVector& other) : size(other.size), data(new int[other.size]) {
        for (size_t i = 0; i < size; ++i) {
            data[i] = other.data[i];
        }
        std::cout << "Copied (Deep Copy)\n";
    }

    // 4. 现代：移动构造函数 (浅拷贝 + 掏空源对象 - O(1) 极快)
    // 注意参数是 MyVector&&，代表传入的是一个马上要死掉的右值
    MyVector(MyVector&& other) noexcept : size(other.size), data(other.data) {
        // 关键步骤：偷走它的指针后，必须把源对象的指针置空！
        // 否则 other 析构时，会把我们刚偷来的内存 delete 掉 (Double Free)
        other.data = nullptr; 
        other.size = 0;
        std::cout << "Moved (Resource Stolen)\n";
    }
};

MyVector createVector() {
    return MyVector(100000); // 返回一个临时对象 (右值)
}

int main() {
    MyVector v1(10);             // 调用普通构造
    MyVector v2 = v1;            // v1 是左值，调用【拷贝构造】

    // createVector() 返回的是临时对象(右值)，匹配到 &&
    // C++11 以前这里会引发深拷贝，现在调用【移动构造】，O(1) 转移！
    MyVector v3 = createVector(); 
    return 0;
}
```

这是最基础的内存管理技术。在 C++ 中，如果你不自己写拷贝构造函数，编译器会自动帮你生成一个。但编译器默认生成的总是**浅拷贝**。



### 浅拷贝 (Shallow Copy)：指针的简单复制

浅拷贝只复制对象本身的值，**包括指针的内存地址**，但不会去复制指针所指向的堆内存。

- **现象：** 对象 A 和对象 B 的指针指向了同一块内存。
- **致命灾难 (Double Free)：** 当 A 和 B 的生命周期结束时，它们的析构函数都会被调用。A 会 `delete` 掉这块内存，接着 B 又会去 `delete` 同一块已经被释放的内存，导致程序直接崩溃 (Segmentation Fault)。

```C++
// 浅拷贝示意
class Shallow {
    int* data;
public:
    Shallow(int d) { data = new int(d); }
    // 编译器默认生成的拷贝构造就是这样的：
    // Shallow(const Shallow& other) : data(other.data) {} 
    ~Shallow() { delete data; } 
};
// Shallow a(10); Shallow b = a; // 灾难！a 和 b 指向同一块内存，析构时 Double Free。
```

### 深拷贝 (Deep Copy)：完全独立的分支

为了解决浅拷贝的灾难，我们需要自定义拷贝构造函数，重新分配一块同样大小的内存，并把内容一个一个复制过去。

- **优点：** 绝对安全，A 和 B 互不干扰。
- **缺点：** 极其耗时。如果里面是一个 $10^6$ 大小的数组，每次拷贝都是 $O(N)$ 的时间复杂度。

```C++
// 深拷贝示意
class Deep {
    int* data;
public:
    Deep(int d) { data = new int(d); }
    // 自定义深拷贝：
    Deep(const Deep& other) {
        data = new int(*other.data); // 开辟新内存，拷贝真实的值
    }
    ~Deep() { delete data; }
};
```

**移动构造的本质，就是一种被控制的“浅拷贝”**。

它执行了浅拷贝（只复制指针，极快 $O(1)$），但为了避免 Double Free，它多做了一步至关重要的操作：**把源对象的指针置空（赋为 `nullptr`）**。因为在 C++ 中 `delete nullptr` 是安全的空操作。



## `std::move` 

**注意：`std::move` 什么都没有 move（不移动任何东西）！**

它的本质仅仅是一个**无条件的类型转换**，把一个左值强制转换为右值引用 (`static_cast<T&&>`)。

假设你有一个左值，但你明确知道**我以后再也不用它了**，你想强行触发移动语义，把它“榨干”：

```C++
std::string s1 = "Competitive Programming";
std::string s2 = s1;             // 发生了拷贝，s1 和 s2 现在一样

// 我告诉编译器：s1 我不要了，把 s1 当作右值来看待，把资源偷给 s3
std::string s3 = std::move(s1);  

// 此时：
// s3 变成了 "Competitive Programming"
// s1 变成了 "有效但未定义的状态" (对于 std::string 通常是空串 "")
// 继续使用 s1 的值是非常危险的工程坏习惯！
```



## 两个经典问题

### 关于局部变量返回

假设我要写一个函数返回一个局部变量。为了避免拷贝，使用 move，是不是更高效了？

```C++
std::vector<int> getVector() {
    std::vector<int> local_vec = {1, 2, 3};
    return std::move(local_vec); // 我用了 move，是不是更高效了？
}
```

并非如此。永远不要 `std::move` 一个准备按值返回的局部变量。

首先，在这里移动没有任何意义，在外面调用函数的时候得到的本来就是右值。

其次，现代 C++ 编译器有 RVO / NRVO（返回值优化 / 具名返回值优化）机制：

- 当你写 `return local_vec;` 时，编译器发现你要返回一个局部变量，它会直接在**调用方**（也就是接收返回值的变量）的内存地址上构造这个 `vector`。**0 次拷贝，0 次移动，开销完全为 0！**
- 当你画蛇添足写了 `return std::move(local_vec);` 时，你骗过了编译器，阻止了 NRVO 的发生。编译器只能退而求其次，执行一次**移动构造**。虽然移动 $O(1)$ 很快，但再快也比不上 RVO 的 0 开销。

### 关于引用的性质

在刚才的移动构造函数内部：

```C++
MyVector(MyVector&& other) {
    // 此时的 other，在函数体内，它自己是左值还是右值？
    // 如果我在函数体里调用 std::vector<int> v = other.some_vector; 是拷贝还是移动？
}
```

无论一个变量的类型是什么，只要它有名字，它本身就是一个左值。

这里的 `other` 当然是左值，因此，在函数体内，如果想转移 `other` 里的成员，必须显式写 `std::move(other.some_vector)`，否则触发的依然是拷贝。

> 需要注意的是，这里的 `&&` **不是**万能引用（Universal / Forwarding Reference）。万能引用**必须发生在类型推导（Type Deduction）的上下文中**。比如模板参数 `template<typename T> void f(T&& x);` 或者 `auto&& x = ...;`。 而在 `MyVector(MyVector&& other)` 中，`MyVector` 是一个已经确定的具体类型，没有类型推导，所以这里的 `&&` 就是纯粹的**右值引用**。

