# 字节C++系统工程师二面

> 来源：https://www.nowcoder.com/discuss/353157751373242368

## 1、C++多态底层实现？vtbl存储在哪里？

多态分类：

- 编译时多态：通过函数重载和模板实现。
- 运行时多态：通过虚函数和继承实现。

运行时多态条件：

- 继承关系。
- 基类定义虚函数（virtual 关键字）。
- 通过基类指针或引用调用虚函数。

#### 底层实现

C++的运行时多态是通过虚函数机制实现的，其底层依赖于虚函数表（vtable）和虚指针（vptr）。

##### 虚函数表（vtable）和虚指针（vptr）

当一个类包含虚函数时，编译器会为该类生成一个虚函数表（vtable），其中存储了该类的虚函数的地址。每个包含虚函数的类实例都会包含一个隐藏的指针，称为虚指针（vptr），指向该类的虚函数表。

##### 虚函数表的存储位置

虚函数表通常存储在程序的只读数据段中，因为它在运行时不需要修改。

##### 多态的底层实现

当通过基类指针或引用调用虚函数时，程序会通过对象的虚指针（vptr）找到对应的虚函数表（vtable），然后在虚函数表中查找实际要调用的函数地址，从而实现动态绑定和多态性。

```c++
#include <iostream>

class Base {
public:
    virtual void show() { std::cout << "Base\n"; }
};

class Derived : public Base {
public:
    void show() override { std::cout << "Derived\n"; }
};

int main() {
    Base* obj = new Derived();
    obj->show();  // 输出：Derived
    delete obj;
    return 0;
}
```

####  vtable 的存储位置

vtable 是静态数据，由编译器生成并存储在程序的特定内存区域。

##### 存储区域

只读数据段（.rodata 或 .rdata）：

- vtable 通常存储在程序的只读数据段，与字符串字面量、常量等类似。
- 这是因为 vtable 在程序运行期间不会修改，具有全局唯一性。

链接阶段：编译器为每个类生成 vtable，链接器将其放入可执行文件的只读段。

运行时：加载到内存的只读段，地址固定，所有对象共享。

##### vptr 的存储位置

对象内部：

- vptr 是一个隐藏成员变量，存储在每个对象实例的内存中。
- 通常位于对象开头（偏移 0），但具体位置由编译器决定（非标准）。

大小：

- 32 位系统：4 字节。
- 64 位系统：8 字节。

## 2、type_info存储在哪里以及是否可改写？

type_info 是一个类，声明在 `<typeinfo>` 头文件中，由编译器生成，用于表示类型的元信息。

`type_info`的存储位置：`type_info`对象的存储位置由编译器实现决定，通常存储在程序的只读数据段中，以确保类型信息在运行时保持不变。

`type_info`是否可修改：`type_info`类的设计旨在提供类型信息的只读访问，因此不提供修改类型信息的接口。

例子：

```c++
#include <iostream>
#include <typeinfo>

class Base {
public:
    virtual ~Base() {}
};

class Derived : public Base {};

int main() {
    Derived d;
    Base& b = d;

    // 获取b的实际类型信息
    const std::type_info& ti = typeid(b);
    std::cout << "Type: " << ti.name() << std::endl;

    return 0;
}
```

`typeid(b)`返回一个指向`type_info`对象的引用，该对象包含了`b`的实际类型信息。

## 3、copy ctor中为什么要pass by reference-to-const？

拷贝构造函数是一种特殊的构造函数，用于从同一类型的对象创建新对象。

典型形式：

```c++
class MyClass {
public:
    MyClass(const MyClass& other) {
        // 拷贝 other 的数据
    }
};
```

#### 为什么要 Pass by Reference-to-Const？

##### 避免无限递归

如果拷贝构造函数接受的是按值传递的方式（即非引用方式），那么调用拷贝构造函数时会先创建传入对象的一个副本，这个过程本身又需要调用拷贝构造函数，从而导致无限递归。

示例：

```c++
class MyClass {
public:
    MyClass(MyClass other) { // 按值传递
        // 拷贝 other
    }
};

int main() {
    MyClass a;
    MyClass b = a; // 无限递归：拷贝 a 到 other 时调用拷贝构造函数
    return 0;
}
```

这样会导致栈溢出。

解决：

- 使用引用（MyClass& 或 const MyClass&），避免形参的拷贝。
- 引用只是别名，不触发拷贝构造函数。

##### 提高效率（避免不必要的拷贝）

按值传递的开销：

- 如果按值传递（MyClass other），即使能避免递归（假设有其他机制），仍需创建形参的副本。
- 对于复杂对象（含动态内存、容器等），拷贝开销大。

引用传递的优势：

MyClass& 或 const MyClass& 仅传递对象的地址（通常 4 或 8 字节），无需复制整个对象。

示例：

```c++
class MyClass {
    int* data;
public:
    MyClass() : data(new int[1000]) {}
    ~MyClass() { delete[] data; }
    MyClass(const MyClass& other) { // 引用传递
        data = new int[1000];
        std::copy(other.data, other.data + 1000, data);
    }
};
```

##### 保护原始对象（为什么要 const）

如果参数是 MyClass&（非 const 引用），拷贝构造函数可能意外修改原始对象。

示例：

```c++
class MyClass {
    int x;
public:
    MyClass(int val) : x(val) {}
    MyClass(MyClass& other) { // 非 const 引用
        x = other.x;
        other.x = 0; // 修改原始对象
    }
};

int main() {
    MyClass a(10);
    MyClass b = a;
    std::cout << a.x << std::endl; // 输出 0，a 被意外修改
    return 0;
}
```

解决：

- 使用 const MyClass&，保证拷贝构造函数只读取 other，不修改它。
- 符合拷贝语义：新对象是原始对象的副本，原始对象保持不变。

##### 支持 const 对象和临时对象

如果参数是 MyClass&（非 const 引用），无法接受 const 对象或临时对象。

示例：

```c++
class MyClass {
public:
    MyClass(MyClass& other) {} // 非 const 引用
};

int main() {
    const MyClass a;
    MyClass b = a; // 错误：不能将 const 对象绑定到非 const 引用

    MyClass c = MyClass(); // 错误：临时对象不能绑定到非 const 引用
    return 0;
}
```

## 4、指针常量和常量指针区别？

#### 指针常量

指针本身是常量，即指针的值（存储的地址）不可修改，但指针指向的数据可以修改。

语法：Type * const pointer = initial_value;

const 位于 * 和指针名之间，修饰指针变量。

##### 特点

- 指针必须初始化（因为地址不可变）。
- 不能重新赋值给指针（改变指向）。
- 可以通过指针修改所指向的数据。

示例：

```c++
#include <iostream>

int main() {
    int a = 10, b = 20;
    int* const ptr = &a; // 指针常量，指向 a

    // *ptr = 30; // 合法：修改指向的数据
    std::cout << *ptr << std::endl; // 输出 30

    // ptr = &b; // 非法：不能改变指针的指向
    // error: assignment of read-only variable 'ptr'

    return 0;
}
```

#### 常量指针

指针指向的数据是常量，即不能通过指针修改数据，但指针本身可以指向其他地址。

语法：const Type * pointer;

const 位于类型前，修饰指针所指向的数据。

##### 特点

- 指针可以不初始化。
- 可以重新赋值给指针（改变指向）。
- 不能通过指针修改所指向的数据。

示例：

```c++
#include <iostream>

int main() {
    int a = 10, b = 20;
    const int* ptr = &a; // 常量指针，指向 a

    // *ptr = 30; // 非法：不能修改指向的数据
    // error: assignment of read-only location '*ptr'

    ptr = &b; // 合法：可以改变指针的指向
    std::cout << *ptr << std::endl; // 输出 20

    return 0;
}
```

#### 两者的区别

| 特性       | 指针常量 (Type * const)      | 常量指针 (const Type *)        |
| ---------- | ---------------------------- | ------------------------------ |
| 定义       | 指针本身不可变               | 指针指向的数据不可变           |
| 语法       | int* const ptr               | const int* ptr                 |
| 初始化要求 | 必须初始化                   | 可不初始化                     |
| 指针指向   | 不可修改（固定地址）         | 可修改（指向其他地址）         |
| 指向的数据 | 可通过指针修改               | 不可通过指针修改               |
| 典型用途   | 固定指向的指针（如函数参数） | 保护数据不被修改（如只读访问） |

## 5、esp在CALL时增大还是减小？

#### 栈与 ESP

- 栈：一种后进先出（LIFO）的数据结构，用于函数调用、参数传递和局部变量存储。
- ESP：栈指针寄存器，指向栈顶的当前地址。
- 栈增长方向：
  - 在 x86 中，栈从高地址向低地址增长。
  - 入栈（push）减少 ESP，出栈（pop）增加 ESP。

#### CALL 指令

- 功能：调用子程序（函数），保存返回地址并跳转到目标地址。
- 操作：
  1. 将当前指令指针（EIP，下一条指令的地址）压入栈。
  2. 将 EIP 设置为目标函数的入口地址。

#### ESP 在 CALL 时是减小

##### 原理

- CALL 指令本质上包含一个隐式的 PUSH

   操作：

  - 将返回地址（下一条指令的地址）压入栈。
  - 压栈意味着栈顶向下移动，ESP 的值减小。

- 栈对齐：

  在 32 位 x86 中，每次压栈通常减少 4 字节（一个字长），因为地址和数据按 32 位对齐。

##### 过程

假设当前 ESP = 0x1000，下一条指令地址为 0x401000：

1. 执行 CALL 0x402000：
   - 返回地址 0x401000 被压入栈。
   - ESP 从 0x1000 减小到 0x0FFC（减去 4 字节）。
   - 内存 [0x0FFC] 存储 0x401000。
   - EIP 更新为 0x402000（目标函数地址）。
2. 栈状态：
   - 调用前：ESP = 0x1000（栈顶）。
   - 调用后：ESP = 0x0FFC（栈顶，保存返回地址）。

#### 为什么 ESP 减小而不是增大？

栈增长方向：

- x86 约定栈向下增长（从高地址到低地址）。
- 入栈减少 ESP，出栈增加 ESP。

与 PUSH 一致：CALL 的压栈操作等价于 PUSH EIP，而 PUSH 总是减小 ESP。

## 6、函数CALL的底层实现？

#### CALL 指令的基本功能

指令格式：

- CALL `<target>`：调用目标地址的函数。
- 示例：CALL 0x401000 或 CALL my_function。

主要操作：

1. 将返回地址（下一条指令的地址）压入栈。
2. 将 EIP 设置为目标函数地址。

伪代码:

```
CALL(target):
    push EIP          // 保存返回地址
    EIP = target      // 跳转到目标地址
```

#### 底层实现过程

假设调用者地址为 0x8048000，目标函数地址为 0x8048100。

##### 调用前状态

寄存器：

- EIP = 0x8048000（CALL 指令地址）。
- ESP = 0x1000（栈顶）。

内存：栈为空，或已有其他数据。

##### 执行 CALL 0x8048100

压栈返回地址：

- 返回地址 = 0x8048005（CALL 指令后的地址，假设 CALL 占 5 字节）。
- ESP -= 4（32 位系统，栈减 4 字节）。
- 内存 [0x0FFC] = 0x8048005。

跳转：EIP = 0x8048100（目标函数入口）。

##### 调用后状态

寄存器：

- EIP = 0x8048100。
- ESP = 0x0FFC。

栈：

```
0x0FFC: [0x8048005] <- ESP
0x1000: [之前的栈数据]
```

#####  函数返回（RET）

RET 指令：

- 出栈返回地址：EIP = [ESP]（即 0x8048005）。
- ESP += 4。

恢复调用者上下文。

## 7、C++11 右值引用是什么？

#### 左值与右值

左值：

- 有持久状态的对象，通常有名字，可以取地址。
- 示例：变量 int x = 10; 中的 x。

右值：

- 临时或即将销毁的对象，无持久状态，通常不可取地址。
- 示例：字面量 10、函数返回值 f()（若返回临时对象）。

#### 右值引用

右值引用是通过`&&`符号定义的，用来绑定到右值上。右值引用允许我们“捕获”即将销毁的对象资源，从而可以转移这些资源而不是复制它们。

##### 语法

```c++
int x = 10;
int& lref = x;       // 左值引用，绑定到左值
// int& rref = 10;   // 错误：左值引用不能绑定右值
int&& rref = 10;     // 右值引用，绑定到右值
```

例子：

```c++
#include <iostream>

void print(int&& r) {
    std::cout << "Rvalue: " << r << std::endl;
}

void print(int& l) {
    std::cout << "Lvalue: " << l << std::endl;
}

int main() {
    int x = 10;
    print(x);      // 输出 "Lvalue: 10"
    print(20);     // 输出 "Rvalue: 20"
    return 0;
}
```

##### 移动语义

传统拷贝构造函数（T(const T&)）复制整个对象，对于动态分配的资源（如 std::vector）效率低。

解决：右值引用允许移动构造函数（T(T&&)）“窃取”临时对象的资源。

示例：

```c++
#include <iostream>
#include <cstring>

class MyString {
    char* data;
public:
    MyString(const char* str) {
        data = new char[strlen(str) + 1];
        strcpy(data, str);
    }

    // 拷贝构造函数
    MyString(const MyString& other) {
        data = new char[strlen(other.data) + 1];
        strcpy(data, other.data);
        std::cout << "Copy" << std::endl;
    }

    // 移动构造函数
    MyString(MyString&& other) noexcept {
        data = other.data;       // 窃取资源
        other.data = nullptr;    // 清空原对象
        std::cout << "Move" << std::endl;
    }

    ~MyString() { delete[] data; }
};

MyString createString() {
    return MyString("Hello");
}

int main() {
    MyString s1 = createString(); // 调用移动构造函数
    return 0;
}
```

输出："Move"。避免了深拷贝，直接转移指针。

##### 完美转发

模板函数传递参数时可能丢失左值/右值属性。

解决：结合 std::forward 和右值引用保留参数属性。

示例：

```c++
#include <iostream>

void process(int& x) { std::cout << "Lvalue: " << x << std::endl; }
void process(int&& x) { std::cout << "Rvalue: " << x << std::endl; }

template<typename T>
void forward(T&& arg) {
    process(std::forward<T>(arg)); // 完美转发
}

int main() {
    int x = 10;
    forward(x);    // 输出 "Lvalue: 10"
    forward(20);   // 输出 "Rvalue: 20"
    return 0;
}
```

##### `std::move`

将左值强制转换为右值引用，触发移动语义。

实现：

```c++
template<typename T>
T&& move(T& obj) {
    return static_cast<T&&>(obj);
}
```

#### 与左值引用的区别

| 特性     | 左值引用 (T&)      | 右值引用 (T&&)         |
| -------- | ------------------ | ---------------------- |
| 绑定对象 | 左值               | 右值                   |
| 语法     | T&                 | T&&                    |
| 用途     | 引用持久对象       | 移动临时对象资源       |
| 修改性   | 可修改（非 const） | 可修改（通常转移资源） |
| 典型场景 | 传递变量           | 移动构造、完美转发     |

## 8、RAII是什么？

RAII：Resource Acquisition Is Initialization，资源获取即初始化。

核心思想：

- 将资源的分配与对象的构造绑定。
- 将资源的释放与对象的析构绑定。

本质：通过 C++ 的自动对象生命周期（栈上对象的构造和析构），管理动态资源（如内存、文件句柄、锁等）。

简单例子：

```c++
#include <iostream>

class Resource {
    int* data;
public:
    Resource() {
        data = new int(42); // 资源获取
        std::cout << "Resource acquired" << std::endl;
    }
    ~Resource() {
        delete data; // 资源释放
        std::cout << "Resource released" << std::endl;
    }
    int get() const { return *data; }
};

int main() {
    Resource r; // 栈上对象
    std::cout << r.get() << std::endl;
    return 0;   // r 析构，自动释放资源
}
```

输出：

```c++
Resource acquired
42
Resource released
```

#### RAII 的工作原理

##### 对象的生命周期

构造：对象创建时，构造函数执行，分配资源。

析构：对象离开作用域时，析构函数自动调用，释放资源。

自动性：C++ 保证栈上对象的析构按逆序执行（LIFO），无需手动干预。

##### 资源绑定

RAII 将资源封装到类中：

- 资源（如指针、文件句柄）作为类的私有成员。
- 构造函数分配资源，析构函数释放资源。

异常安全：即使抛出异常，栈展开确保析构函数被调用。

## 9、shared_ptr实现原理，引用计数的存储方式？

`std::shared_ptr` 是 C++11 引入的智能指针，用于管理动态分配资源的共享所有权。它通过引用计数（Reference Counting）机制确保资源在最后一个指针销毁时被正确释放。

#### 实现原理

`std::shared_ptr` 的核心在于维护一个控制块（Control Block），该控制块通常包含以下信息：

- 引用计数： 记录当前有多少个 `shared_ptr` 实例共享拥有该对象。
- 弱引用计数： 记录有多少个 `std::weak_ptr` 指向该对象。弱引用不会影响对象的生命周期，但需要通过弱引用计数来管理控制块的销毁。
- 指向实际对象的指针： 即被管理的对象本身。
- 自定义删除器： 用于在对象销毁时执行特定的清理操作。

当创建一个新的 `shared_ptr` 时，会分配一个新的控制块，并将引用计数初始化为 1。每当复制一个 `shared_ptr`，引用计数加 1；每当一个 `shared_ptr` 被销毁或重置，引用计数减 1。当引用计数减为零时，表示没有 `shared_ptr` 再指向该对象，此时对象会被销毁。随后，若弱引用计数也为零，控制块本身也会被释放。

#### 引用计数的存储方式

1. 独立分配控制块： 控制块与实际对象分开存储。每次创建 `shared_ptr` 时，控制块会被单独分配在堆内存中。这种方式的优点是灵活性高，适用于对象的内存布局无法修改的情况。
2. 对象内嵌控制块： 控制块与实际对象一起分配在同一块内存中。这通常通过 `std::make_shared` 实现，它在分配对象时，同时分配控制块。这种方式可以减少内存分配次数，提升性能，但要求对象的内存布局允许这种嵌入。

由于 `shared_ptr` 采用引用计数机制，如果存在循环引用（即两个或多个对象互相持有 `shared_ptr` 指向对方），会导致引用计数无法降为零，从而造成内存泄漏。为了解决这个问题，可以使用 `std::weak_ptr`，它是一种不增加引用计数的弱引用，可以打破循环引用，确保对象能被正确销毁。

示例：

```c++
struct Node {
    std::shared_ptr<Node> next;
    ~Node() { std::cout << "Destroyed\n"; }
};

int main() {
    auto n1 = std::make_shared<Node>();
    auto n2 = std::make_shared<Node>();
    n1->next = n2;
    n2->next = n1; // 循环引用
    return 0; // 不会析构
}
```

use_count 永不为 0，导致内存泄漏。

解决：

使用 std::weak_ptr 打破循环：

```c++
struct Node {
    std::weak_ptr<Node> next;
};
```

## 10、make_shared和new区别？

#### 内存分配方式

##### make_shared

单次分配：同时为对象和控制块（shared_ptr 的引用计数管理结构）分配一块连续内存。

内存布局：

```c++
[T 对象 | ControlBlock]
```

控制块包含引用计数（use_count 和 weak_count）和其他元数据。

##### new

两次分配：

1. new T 分配对象内存，返回裸指针。
2. shared_ptr 构造函数再分配控制块内存。

内存布局：

```c++
[T 对象]        <- 裸指针
[ControlBlock]   <- 独立分配
```

对象和控制块在堆上的不同区域。

#### 性能

**make_shared**

优点：

- 单次内存分配，减少分配开销（通常 malloc 或 operator new 一次）。
- 连续内存布局，改善缓存局部性（Cache Locality），提升访问效率。

开销：O(1) 分配，构造一次。

##### new

缺点：

- 两次内存分配，增加开销。
- 对象和控制块分散，缓存命中率可能降低。

开销：O(1) 分配两次，构造一次。

#### 异常安全性

使用 `new` 时，如果在对象创建后但在将其传递给 `std::shared_ptr` 之前发生异常，可能导致内存泄漏。而 `std::make_shared` 将对象的创建和 `shared_ptr` 的构造合并为一个原子操作，避免了这种潜在的内存泄漏问题，提高了异常安全性。

#### 总结

| 特性     | std::make_shared    | new + shared_ptr     |
| -------- | ------------------- | -------------------- |
| 内存分配 | 单次（对象+控制块） | 两次（对象，控制块） |
| 性能     | 更高（缓存友好）    | 较低（多次分配）     |
| 异常安全 | 强（原子操作）      | 弱（可能泄漏）       |
| 语法     | 简洁优雅            | 稍显冗长             |
| 删除器   | 不支持自定义        | 支持自定义           |
| 适用场景 | 常规构造            | 复杂构造或特殊释放   |

## 11、malloc底层实现原理？

C 语言中，`malloc`（memory allocation 的缩写）函数用于在运行时动态分配指定大小的内存块。它返回一个指向已分配内存的指针，如果分配失败，则返回 `NULL`。与静态或自动内存分配不同，`malloc` 提供了灵活的内存管理方式，使程序能够根据需要在运行时请求和释放内存。

#### 底层实现原理

1. 内存池初始化： 内存分配器会维护一个内存池（heap），用于管理动态分配的内存块。初始化时，分配器会向操作系统请求一块连续的内存区域作为内存池的起始。
2. 内存块管理： 内存池被划分为多个内存块，每个内存块包含两部分：元数据和实际数据区。元数据通常存储内存块的大小、状态（已分配或空闲）等信息。
3. 内存分配： 当调用 `malloc` 请求分配内存时，分配器会在内存池中查找足够大的空闲块。如果找到合适的块，则将其标记为已分配，并返回指向该块数据区的指针。
4. 内存释放： 当调用 `free` 释放内存时，分配器会将对应的内存块标记为空闲，并可能与相邻的空闲块合并，以减少内存碎片。
5. 内存扩展： 如果内存池中没有足够大的空闲块满足 `malloc` 请求，分配器可能会向操作系统请求更多的内存，以扩展内存池。

#### 内存分配算法

不同的 `malloc` 实现可能采用不同的内存分配算法，以平衡内存利用率和分配速度。

常见的算法：

- 首次适配： 从内存池的起始位置开始，找到第一个足够大的空闲块进行分配。
- 最佳适配： 在所有空闲块中找到最接近所需大小的块进行分配，以减少内存碎片。
- 最差适配： 选择最大的空闲块进行分配，理论上可以保留较大的连续空闲空间。

## 12、虚拟内存、MMU、TLB细节？

#### 虚拟内存

虚拟内存是一种内存管理技术，为每个进程提供独立的虚拟地址空间，使其认为自己独占整个内存。

目的：

- 隔离：进程间互不干扰。
- 扩展：通过磁盘扩展物理内存。
- 简化：程序使用连续地址，无需关心物理内存布局。

##### 工作原理

虚拟地址（VA）：

- 进程看到的地址，由程序生成。
- 示例：0x8048000。

物理地址（PA）：

- 实际硬件内存地址。
- 示例：0x1000。

映射：通过页表（Page Table）将虚拟地址映射到物理地址。

分页：

- 内存按固定大小的页面（Page）划分，通常 4KB。
- 虚拟页（VPN，Virtual Page Number）映射到物理帧（PFN，Physical Frame Number）。

##### 虚拟地址空间

32 位示例

```
0xFFFFFFFF +-----------------+
           | 内核空间         |
0xC0000000 +-----------------+
           | 栈              |
           | ...             |
           | 堆              |
           | 数据段 (.data)   |
           | 代码段 (.text)   |
0x00000000 +-----------------+
```

##### 实现机制

页表：

- 存储虚拟页到物理帧的映射。
- 每进程一张页表，内核维护。

换页（Swapping）：当物理内存不足，将不活跃页面换到磁盘（Swap 分区）。

按需分页（Demand Paging）：页面初次访问时加载，触发页面错误（Page Fault）。

#### MMU

MMU 是 CPU 中的硬件组件，负责将虚拟地址转换为物理地址。

##### 工作原理

输入：虚拟地址（由程序生成）。

输出：物理地址（送往内存控制器）。

过程：

分解虚拟地址：

- 页号（VPN）：高位，表示虚拟页面。

- 偏移（Offset）：低位，表示页面内位置。

- 示例（4KB 页，32 位地址）：

  ```
  31        12 11        0
  +-----------+----------+
  | VPN       | Offset   |
  +-----------+----------+
  ```

查找页表：

- MMU 使用页表基址寄存器（CR3 在 x86）定位页表。
- 根据 VPN 查找对应表项（PTE，Page Table Entry）。

生成物理地址：

- PTE 包含 PFN（物理帧号）。
- 物理地址 = PFN << 12 | Offset。

检查权限：PTE 中的标志位（如读/写、可执行）决定访问是否合法。

#### TLB

TLB 是 MMU 中的高速缓存，用于加速虚拟地址到物理地址的转换。

目的：

- 减少访问页表的时间开销。
- 提高内存访问效率。

##### 工作原理

缓存原理：

- 存储最近使用的虚拟页到物理帧的映射。
- 类似 CPU 缓存，但专用于地址翻译。

结构：

- 键值对：VPN → PFN + 属性。
- 条目数：有限（几十到几千，视 CPU）。
- 组织方式：
  - 全相联（Fully Associative）。
  - 组相联（Set Associative）。

查找：

1. 输入 VPN。
2. TLB 命中：直接返回 PFN。
3. TLB 未命中：访问页表，更新 TLB。

## 13、LC207 课程表

#### 解题思路

构建有向图和入度数组：

- 使用邻接表表示有向图，`graph[i]` 存储课程 `i` 的后续课程。
- 使用数组 `inDegree` 记录每个课程的入度，即需要先修课程的数量。

初始化队列：

将所有入度为 0 的课程加入队列，这些课程可以直接开始学习。

执行拓扑排序：

- 从队列中取出一个课程，记录已完成课程的数量。
- 对于取出的课程的每个后续课程，将其入度减 1。
- 如果某个后续课程的入度减为 0，将其加入队列。

判断结果：

如果已完成课程的数量等于总课程数，说明可以完成所有课程；否则，存在环路，无法完成所有课程。

#### 参考代码（C++）

```c++
#include <vector>
#include <queue>

class Solution {
public:
    bool canFinish(int numCourses, std::vector<std::vector<int>>& prerequisites) {
        // 构建邻接表和入度数组
        std::vector<std::vector<int>> graph(numCourses);
        std::vector<int> inDegree(numCourses, 0);
        for (const auto& pre : prerequisites) {
            graph[pre[1]].push_back(pre[0]);
            ++inDegree[pre[0]];
        }
        
        // 初始化队列，将所有入度为 0 的课程加入队列
        std::queue<int> q;
        for (int i = 0; i < numCourses; ++i) {
            if (inDegree[i] == 0) {
                q.push(i);
            }
        }
        
        // 执行拓扑排序
        int completedCourses = 0;
        while (!q.empty()) {
            int course = q.front();
            q.pop();
            ++completedCourses;
            for (int nextCourse : graph[course]) {
                if (--inDegree[nextCourse] == 0) {
                    q.push(nextCourse);
                }
            }
        }
        
        // 判断是否可以完成所有课程
        return completedCourses == numCourses;
    }
};
```