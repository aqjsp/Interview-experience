# momenta中间件C++开发一面

> 来源：https://www.nowcoder.com/feed/main/detail/c275081d6f1a4aa5a2c160b97f1eba36

### 1、C++智能指针？

#### 1. `std::unique_ptr`

- 特点：独占所有权，不能被拷贝，只能移动。
- 用途：用于明确一个对象只会有一个所有者的场景，保证独占所有权。
- 用法：不能通过拷贝构造函数或赋值运算符复制 `std::unique_ptr`，但可以通过 `std::move` 转移所有权。

```c++
#include <memory>
#include <iostream>

struct Foo {
    Foo() { std::cout << "Foo created.\n"; }
    ~Foo() { std::cout << "Foo destroyed.\n"; }
};

int main() {
    std::unique_ptr<Foo> ptr1(new Foo());  // 创建 unique_ptr，管理 Foo 对象
    std::unique_ptr<Foo> ptr2 = std::move(ptr1);  // 转移所有权，ptr1 变为空
    return 0;  // ptr2 离开作用域时自动释放内存
}
```

- 优点：内存管理简单，避免内存泄漏。
- 缺点：不支持共享所有权，不能多指针指向同一对象。

#### 2. `std::shared_ptr`

- 特点：共享所有权，多个智能指针可以共享同一对象的所有权，当最后一个指针销毁时，才会释放对象。
- 用途：适用于多个地方需要共同拥有对象的场景，自动管理引用计数。
- 用法：`std::shared_ptr` 支持拷贝和赋值，每一个 `std::shared_ptr` 实例都会维护一个引用计数。当引用计数变为 0 时，自动释放对象。

```c++
#include <memory>
#include <iostream>

struct Foo {
    Foo() { std::cout << "Foo created.\n"; }
    ~Foo() { std::cout << "Foo destroyed.\n"; }
};

int main() {
    std::shared_ptr<Foo> ptr1(new Foo());  // 创建 shared_ptr
    {
        std::shared_ptr<Foo> ptr2 = ptr1;  // ptr2 和 ptr1 共享所有权
        std::cout << "Shared ownership count: " << ptr1.use_count() << std::endl;  // 输出 2
    }  // ptr2 离开作用域，不再持有对象
    std::cout << "Shared ownership count: " << ptr1.use_count() << std::endl;  // 输出 1
    return 0;  // 最后一个 shared_ptr 离开作用域，释放对象
}
```

- 优点：可以在多个地方共享对象的所有权。
- 缺点：维护引用计数开销较大，可能导致循环引用问题（需配合 `std::weak_ptr` 解决）。

#### 3. `std::weak_ptr`

- 特点：不控制对象生命周期，解决 `std::shared_ptr` 循环引用的问题。
- 用途：用于观察 `std::shared_ptr` 管理的对象，但不增加引用计数。它不会影响对象的生命周期，适合用于缓存、回调等场景。
- 用法：`std::weak_ptr` 不能直接访问对象，需要通过 `lock()` 方法将其提升为 `std::shared_ptr` 才能访问对象。

```c++
#include <memory>
#include <iostream>

struct Foo {
    Foo() { std::cout << "Foo created.\n"; }
    ~Foo() { std::cout << "Foo destroyed.\n"; }
};

int main() {
    std::shared_ptr<Foo> sharedPtr = std::make_shared<Foo>();  // 创建 shared_ptr
    std::weak_ptr<Foo> weakPtr = sharedPtr;  // 创建 weak_ptr，不增加引用计数
    
    if (auto tempPtr = weakPtr.lock()) {  // 提升为 shared_ptr
        std::cout << "Object is still alive.\n";
    } else {
        std::cout << "Object has been destroyed.\n";
    }
    return 0;
}
```

- 优点：避免 `std::shared_ptr` 的循环引用问题，不增加引用计数。
- 缺点：不能直接访问对象，需要使用 `lock()` 提升为 `std::shared_ptr`。

#### 4. `std::auto_ptr`（已废弃）

- 特点：早期 C++98 引入的智能指针，单一所有权。
- 缺点：不支持标准的拷贝语义（拷贝时所有权转移），导致使用上的困惑。由于 `std::unique_ptr` 提供了更好的替代方案，`std::auto_ptr` 在 C++11 中被废弃。

```c++
#include <memory>
#include <iostream>

int main() {
    std::auto_ptr<int> ptr1(new int(10));  // 创建 auto_ptr
    std::auto_ptr<int> ptr2 = ptr1;  // 所有权转移，ptr1 变为空，ptr2 持有对象
    std::cout << *ptr2 << std::endl;  // 输出 10
}
```

- 结论：`std::auto_ptr` 已被废弃，应使用 `std::unique_ptr` 来替代。

### 2、unique_ptr有什么特性，底层实现是怎样的？

#### 主要特性

1. 独占所有权：

   资源的所有权只能属于一个 `unique_ptr`，不能被拷贝或共享。当你尝试将一个 `unique_ptr` 赋值给另一个时，编译器会报错。

   ```c++
   std::unique_ptr<int> ptr1(new int(10));
   std::unique_ptr<int> ptr2 = ptr1;  // 错误：不能拷贝
   ```

2. 移动语义：

   `unique_ptr` 支持移动语义，意味着你可以通过 `std::move` 将资源的所有权从一个 `unique_ptr` 转移到另一个。这在函数返回时或需要重新分配所有权时非常有用。

   ```c++
   std::unique_ptr<int> ptr1(new int(10));
   std::unique_ptr<int> ptr2 = std::move(ptr1);  // 所有权从 ptr1 转移到 ptr2，ptr1 变为 nullptr
   ```

3. 自动释放资源：

   当 `unique_ptr` 离开作用域或被销毁时，管理的对象会自动释放，不需要手动调用 `delete`。这极大减少了内存泄漏的风险。

   ```c++
   {
       std::unique_ptr<int> ptr(new int(10));
   }  // 离开作用域时，自动调用 delete
   ```

4. 自定义删除器：

   `unique_ptr` 支持自定义删除器，你可以为其指定如何删除所管理的对象，尤其适合需要复杂清理逻辑的场景（例如需要调用 `free` 或其他自定义释放函数时）。

   ```c++
   std::unique_ptr<int, void(*)(int*)> ptr(new int(10), [](int* p) {
       std::cout << "Custom deleter called.\n";
       delete p;
   });
   ```

5. 效率高：

   `unique_ptr` 的存储开销非常低，通常只包含一个指针（指向管理的资源），并且不维护引用计数，因此比 `shared_ptr` 更轻量。

#### 底层实现

`std::unique_ptr` 的底层实现并不复杂，主要依赖于**RAII**（Resource Acquisition Is Initialization）的思想来管理资源。它使用了模板和移动语义来确保资源的独占管理。大致上可以分为以下几个关键点：

##### 1. 模板类型管理资源

`std::unique_ptr` 是一个模板类，用于管理动态分配的对象，模板参数指定了它管理的资源类型。

```c++
template <typename T, typename Deleter = std::default_delete<T>>
class unique_ptr {
private:
    T* ptr;  // 资源指针
    Deleter deleter;  // 自定义删除器
};
```

- T：`unique_ptr` 管理的资源的类型。
- Deleter：删除资源的方式，默认使用 `std::default_delete<T>`，即 `delete` 操作，但可以自定义删除器。

##### 2. 构造函数与析构函数

构造函数用于初始化 `unique_ptr`，并且它不会默认创建任何资源，除非显式传递了一个资源指针。析构函数确保当 `unique_ptr` 离开作用域时，调用删除器释放资源。

```c++
unique_ptr(T* ptr = nullptr) : ptr(ptr) {}

~unique_ptr() {
    if (ptr) deleter(ptr);  // 在析构时调用删除器
}
```

##### 3. 禁止拷贝构造和拷贝赋值

为了保证独占所有权，`std::unique_ptr` 禁止拷贝构造和拷贝赋值运算符。这通过将它们声明为 `delete` 来实现。

```c++
unique_ptr(const unique_ptr&) = delete;  // 禁止拷贝构造
unique_ptr& operator=(const unique_ptr&) = delete;  // 禁止拷贝赋值
```

##### 4. 支持移动构造和移动赋值

`unique_ptr` 支持移动语义，这意味着可以通过移动构造或移动赋值将资源的所有权从一个 `unique_ptr` 转移到另一个。

```c++
unique_ptr(unique_ptr&& other) noexcept : ptr(other.ptr) {
    other.ptr = nullptr;  // 转移所有权后，原指针置空
}

unique_ptr& operator=(unique_ptr&& other) noexcept {
    if (this != &other) {
        deleter(ptr);  // 释放现有资源
        ptr = other.ptr;  // 转移所有权
        other.ptr = nullptr;  // 置空原指针
    }
    return *this;
}
```

移动构造函数和移动赋值运算符从另一个 `unique_ptr` 中“偷走”指针，将原来的指针置为空。

##### 5. 访问与释放资源

通过 `operator*` 和 `operator->` 可以访问 `unique_ptr` 管理的资源，而 `release()` 可以手动释放资源但不销毁资源。

```c++
T& operator*() const { return *ptr; }  // 解引用
T* operator->() const { return ptr; }  // 指针访问

T* release() {
    T* temp = ptr;
    ptr = nullptr;  // 释放资源，但不销毁
    return temp;
}
```

- `release()`：手动释放所有权，将内部的指针置空，并返回指针的所有权，用户负责管理返回的指针。
- `reset()`：替换当前管理的资源，释放旧的资源。

### 3、unique_ptr是怎么保证无法赋值构造的？

`std::unique_ptr` 通过删除拷贝构造函数和拷贝赋值运算符来确保它不能被复制。这种机制是 C++11 引入的特性，称为 **删除的函数** (`= delete`)，它用于显式禁止某些操作。

`std::unique_ptr` 中删除拷贝构造和拷贝赋值运算符的方式：

```c++
template<typename T>
class unique_ptr {
public:
    // 删除拷贝构造函数
    unique_ptr(const unique_ptr&) = delete;

    // 删除拷贝赋值运算符
    unique_ptr& operator=(const unique_ptr&) = delete;

    // 支持移动构造函数
    unique_ptr(unique_ptr&& other) noexcept;

    // 支持移动赋值运算符
    unique_ptr& operator=(unique_ptr&& other) noexcept;
    
    // 其他成员函数...
};
```

#### 工作原理

1. 删除拷贝构造函数：

```c++
unique_ptr(const unique_ptr&) = delete;
```

当你尝试使用 `std::unique_ptr` 进行拷贝时，比如：

```c++
std::unique_ptr<int> ptr1(new int(10));
std::unique_ptr<int> ptr2 = ptr1;  // 错误，编译不通过
```

编译器会发现 `unique_ptr` 的拷贝构造函数已经被删除，因此编译会失败，报出类似的错误：“use of deleted function”。

2. 删除拷贝赋值运算符：

```c++
unique_ptr& operator=(const unique_ptr&) = delete;
```

类似地，当你尝试给一个 `unique_ptr` 赋值时，比如：

```c++
std::unique_ptr<int> ptr1(new int(10));
std::unique_ptr<int> ptr2;
ptr2 = ptr1;  // 错误，编译不通过
```

编译器也会报错，因为 `operator=` 被标记为 `delete`，因此禁止赋值。

#### 移动语义

虽然 `unique_ptr` 禁止了拷贝操作，但它支持**移动语义**，允许将所有权从一个 `unique_ptr` 转移到另一个。这通过实现移动构造函数和移动赋值运算符来完成：

```c++
// 移动构造函数
unique_ptr(unique_ptr&& other) noexcept : ptr(other.ptr) {
    other.ptr = nullptr;  // 将其他对象的指针置空
}

// 移动赋值运算符
unique_ptr& operator=(unique_ptr&& other) noexcept {
    if (this != &other) {
        delete ptr;       // 删除当前的指针
        ptr = other.ptr;  // 转移所有权
        other.ptr = nullptr;  // 将其他对象的指针置空
    }
    return *this;
}
```

通过移动语义，你可以安全地将 `unique_ptr` 的所有权从一个对象转移到另一个，而不会违反独占所有权的原则。

### 4、shared_ptr怎么实现的，引用计数是什么数据格式？

`shared_ptr` 主要通过引用计数来管理资源的生命周期。每当一个新的 `shared_ptr` 被创建并指向某个资源时，引用计数会增加；每当一个 `shared_ptr` 被销毁或不再指向资源时，引用计数会减少。当引用计数变为 0 时，资源会自动释放。

#### 核心实现

1. 引用计数：`shared_ptr` 使用了一个额外的计数器来记录指向某个资源的 `shared_ptr` 的数量。
2. 控制块：`shared_ptr` 通常管理一个单独的控制块，该块包含了：
   - 资源的指针（即被 `shared_ptr` 管理的对象）。
   - 引用计数（用于记录有多少 `shared_ptr` 指向同一资源）。
   - 弱引用计数（用于 `std::weak_ptr`）。
   - 删除器（自定义删除方式）。

工作方式：

```c++
template <typename T>
class shared_ptr {
private:
    T* ptr;                    // 被管理的资源
    std::shared_ptr_control_block* control_block;  // 控制块，包含引用计数等信息

public:
    // 构造函数
    explicit shared_ptr(T* p = nullptr) : ptr(p) {
        if (ptr) {
            control_block = new std::shared_ptr_control_block(ptr);
            control_block->increment_ref_count();  // 初始化时增加引用计数
        }
    }

    // 拷贝构造函数
    shared_ptr(const shared_ptr& other) noexcept {
        ptr = other.ptr;
        control_block = other.control_block;
        if (control_block) {
            control_block->increment_ref_count();  // 复制时增加引用计数
        }
    }

    // 析构函数
    ~shared_ptr() {
        if (control_block && control_block->decrement_ref_count() == 0) {
            delete ptr;                  // 当引用计数为0时删除资源
            delete control_block;        // 删除控制块
        }
    }

    // 其他操作...
};
```

#### 引用计数的数据格式

引用计数通常是一个 **整型变量**，例如 `std::atomic<int>`，以确保线程安全性。在多线程环境下，多个线程可能同时操作同一个 `shared_ptr`，因此必须使用 **原子操作** 来保证引用计数的更新是安全的。

##### 典型的控制块实现

```c++
struct control_block {
    std::atomic<int> strong_ref_count;  // 强引用计数
    std::atomic<int> weak_ref_count;    // 弱引用计数
    T* resource;                        // 被管理的资源

    control_block(T* ptr)
        : strong_ref_count(1), weak_ref_count(0), resource(ptr) {}

    // 增加强引用计数
    void increment_strong_ref() {
        strong_ref_count.fetch_add(1, std::memory_order_relaxed);
    }

    // 减少强引用计数
    int decrement_strong_ref() {
        return strong_ref_count.fetch_sub(1, std::memory_order_acq_rel);
    }

    // 增加弱引用计数
    void increment_weak_ref() {
        weak_ref_count.fetch_add(1, std::memory_order_relaxed);
    }

    // 减少弱引用计数
    int decrement_weak_ref() {
        return weak_ref_count.fetch_sub(1, std::memory_order_acq_rel);
    }

    // 销毁资源
    void destroy_resource() {
        delete resource;
    }
};
```

**`std::atomic<int>`**：引用计数使用原子操作 (`std::atomic<int>`) 来保证线程安全。原子操作能够确保在多个线程同时修改引用计数时不会产生竞争条件，防止引用计数错误。

#### 引用计数如何管理资源

- 每次创建新的 `shared_ptr` 指向同一个资源时，引用计数加 1。
- 每次一个 `shared_ptr` 离开作用域或被显式销毁时，引用计数减 1。
- 当引用计数减为 0 时，说明没有任何 `shared_ptr` 再管理该资源，资源会被释放，随后控制块也会被销毁。

```c++
{
    std::shared_ptr<int> sp1 = std::make_shared<int>(10);  // 引用计数为1
    {
        std::shared_ptr<int> sp2 = sp1;                    // 引用计数为2
    }  // sp2 离开作用域，引用计数减为1
}  // sp1 离开作用域，引用计数减为0，资源被释放
```

### 5、引用计数的线程安全怎么保证的，底层怎么实现？

1. 原子操作：原子操作保证了在多线程环境下对共享变量（如引用计数）的修改不会被中途打断或发生竞争。例如，在递增或递减引用计数时，线程 A 和线程 B 不会同时修改这个计数器并导致数据不一致。
2. 内存模型：C++ 的原子操作提供了几种内存序模型，例如 `memory_order_relaxed`、`memory_order_acquire` 和 `memory_order_release`，以控制操作的内存可见性顺序。`shared_ptr` 通常使用 `memory_order_acquire` 和 `memory_order_release` 来确保引用计数的线程安全，同时减少内存屏障的开销。

### 6、动态链接和静态链接具体有什么区别，各有什么优势？

#### 1. 静态链接

静态链接是在**编译**或**链接**阶段，将所需的库文件直接整合到可执行文件中。在最终生成的可执行文件中，库的代码已经成为其中的一部分，不需要额外的库文件来运行。

##### 工作流程

- 编译器在编译时，将库文件的代码嵌入到目标文件中，并在链接时合并生成一个完整的可执行文件。
- 生成的可执行文件自包含所有必要的库，因此不需要外部依赖来运行。

##### 优点

1. 独立性：生成的可执行文件包含了所有依赖的库，因此无需依赖外部的库文件，特别是在库版本不兼容的情况下，不会受到影响。
2. 分发简单：由于可执行文件中包含所有依赖，分发时只需要提供一个独立的文件即可，适合没有共享库环境的系统。
3. 加载速度快：因为不需要在运行时查找和加载外部库，启动速度较快。

##### 缺点

1. 文件体积大：由于所有的库代码都被嵌入到可执行文件中，生成的可执行文件体积会较大，尤其当多个程序使用相同的库时，每个程序都包含库的副本，导致冗余。
2. 更新不便：如果库有了更新或者需要修复漏洞，必须重新编译整个程序，而不仅仅是替换库文件。
3. 内存占用增加：多个程序静态链接同一个库时，每个程序都包含库的副本，会导致内存中的库代码被多次加载，浪费资源。

#### 2. 动态链接

动态链接是在**运行时**，将所需的库文件动态加载到内存中，并链接到程序的执行过程中。库文件不包含在可执行文件中，而是存储在外部，程序在运行时通过操作系统加载所需的共享库（如 `.dll`、`.so` 文件）。

##### 工作流程

- 在编译阶段，生成的可执行文件中只包含对外部库的引用符号，而不包含库的实际代码。
- 在程序运行时，操作系统负责查找并加载库文件，将它们链接到程序的执行地址空间。

##### 优点

1. 节省内存：多个程序可以共享相同的库文件，只需加载一份库代码到内存中，多个进程可以共享相同的库代码段，减少内存占用。
2. 可升级性强：库文件可以单独升级或修复漏洞，无需重新编译整个程序。只需更新库文件，所有依赖该库的程序自动使用最新版本的库。
3. 文件体积小：可执行文件不需要包含库代码，因此文件体积更小。
4. 灵活性：允许在运行时替换库文件，便于调试和功能扩展。

##### 缺点

1. 运行时依赖性：程序的运行依赖于外部库，如果所需的库文件缺失、版本不兼容或路径不正确，程序可能无法运行。
2. 加载速度较慢：由于库文件在运行时动态加载，程序启动时需要额外的时间查找和加载库，启动速度可能比静态链接的程序慢。
3. 兼容性问题：多个程序使用相同的动态库版本时，可能出现版本不兼容问题，导致“动态链接库地狱”问题。

#### 3. 比较总结

| 特性           | 静态链接                           | 动态链接                             |
| -------------- | ---------------------------------- | ------------------------------------ |
| 链接时间       | 编译/链接时                        | 运行时                               |
| 库代码包含方式 | 库代码直接嵌入到可执行文件         | 库代码在运行时由操作系统加载         |
| 文件大小       | 较大（每个可执行文件包含库代码）   | 较小（库代码存储在外部共享库文件中） |
| 内存占用       | 每个程序包含库的副本，占用较多内存 | 库文件共享，内存利用率高             |
| 升级难易度     | 必须重新编译整个程序               | 只需替换库文件                       |
| 启动速度       | 较快（无需动态加载库）             | 较慢（需要查找并加载库文件）         |
| 分发方式       | 独立的可执行文件，分发简单         | 必须确保目标系统有正确的库文件       |
| 运行时依赖     | 无外部依赖                         | 依赖于外部库文件，库版本必须兼容     |

### 7、动态链接库被加载到什么位置，这个位置是怎么寻址的？

动态链接库（Dynamic Link Library, DLL 或者 `.so` 文件）在程序运行时被加载到**虚拟内存**的一个特定位置。操作系统负责将动态链接库映射到程序的地址空间中，具体加载到哪里以及如何寻址取决于操作系统和库文件的格式。

#### 1. 动态链接库的加载位置

当程序运行时，操作系统的**动态链接器**（linker/loader）负责将动态库加载到进程的虚拟地址空间中。加载的位置通常有两种策略：

##### 1.1. 预定义的加载地址

在某些情况下，动态库可以指定一个**预定义的基地址**，也就是说库文件希望被加载到某个特定的虚拟地址。如果没有地址冲突，操作系统会尝试将库加载到这个位置。预定义加载地址的库在早期系统中比较常见，但现代操作系统更倾向于使用地址随机化技术。

##### 1.2. 地址空间布局随机化

现代操作系统（如 Linux、Windows、macOS）通常使用 ASLR 技术。ASLR 在程序运行时，随机选择一个虚拟地址加载动态库，以提高安全性，防止攻击者通过预测库的地址来进行攻击。这意味着同一动态库每次被加载时，可能会映射到不同的虚拟地址。

在 ASLR 机制下，操作系统会动态决定库的加载地址，确保每个库的加载地址都不与其他部分（如程序代码、堆栈、其他库）发生冲突。

#### 2. 寻址方式

库加载到虚拟地址空间后，程序如何访问这些动态库的函数和变量呢？这涉及到**动态链接和动态重定位**的机制。

##### 2.1. 基址重定位

当动态库被编译时，编译器并不知道库最终会被加载到内存的哪个位置，因此库中的函数和变量地址通常是**相对地址**。在库文件中，存在一个**重定位表**，记录了需要在加载时修正的地址。加载器会根据库被加载的实际基址，对库中的符号地址进行修正，使得程序能够正确访问库中的内容。

如果库在编译时指定了一个基址，但在加载时无法加载到该地址（因为地址被其他模块占用），操作系统会通过重定位机制，将库加载到其他地址，并更新所有的指针和符号引用。

##### 2.2. 全局偏移表

动态库中的全局变量和函数的地址通常通过全局偏移表（GOT）来访问。GOT 是一个特殊的数据结构，它保存了全局变量和函数的实际地址。程序中使用的每个外部函数或变量调用，都会先通过 GOT 查找实际的地址，然后再执行调用。

- 当库被加载到虚拟地址空间中，操作系统会将函数和变量的实际地址填充到 GOT 表中。
- 程序在执行时通过 GOT 查找对应的符号地址，这样即使库的基地址发生变化，程序依然可以通过 GOT 进行正确的函数调用或变量访问。

##### 2.3. 动态符号解析

对于动态库中的函数调用，常常使用**过程链接表**（PLT）来进行符号解析。PLT 是一个表，它保存了每个外部函数的入口地址。当程序首次调用动态库中的函数时，PLT 会触发动态链接器去解析该函数的实际地址，并将该地址缓存下来。

- 当程序第一次调用一个库函数时，PLT 会将调用转发给动态链接器。
- 动态链接器解析这个符号，并将实际地址写入 PLT 表中。
- 后续的调用就直接通过 PLT 进行跳转，无需再次解析。

#### 3. 动态库加载过程中的步骤

1. 程序启动：可执行文件启动时，操作系统会根据文件中的信息查找并加载依赖的动态库。
2. 地址映射：动态链接器会为动态库分配虚拟地址空间，并将库文件映射到该地址。
3. 符号解析与重定位：动态链接器会根据库的实际加载地址，修正库中的符号引用，并填充 GOT 和 PLT 表，以确保函数和变量能够被正确调用。
4. 执行程序：程序开始运行时，库中的代码和数据通过上述表结构被正确访问，执行函数调用和数据操作。

### 8、虚拟内存里文件映射区在什么位置，位置信息是怎么维护的？

在虚拟内存中，**文件映射区**（Memory-Mapped Files）指的是将一个文件的内容映射到进程的虚拟地址空间，使得文件的内容能够像访问内存一样被读取或修改。文件映射区的位置和管理方式由操作系统和内存管理单元（MMU）共同决定。

#### 1. 文件映射区的位置

文件映射区在虚拟内存中的位置并没有固定的地址，它的具体位置取决于以下几个因素：

- 操作系统的内存管理策略：不同操作系统会有不同的虚拟内存布局。例如，Linux 系统和 Windows 系统的虚拟内存布局有所不同，但通常文件映射区位于**进程的用户地址空间**中。
- 内存分配方式：当进程请求文件映射时，操作系统会根据当前的虚拟内存分配情况，选择一个空闲的内存区域进行映射。映射位置可能是由操作系统自动分配的，也可以由程序指定。
- 地址空间布局随机化（ASLR）：现代操作系统通过 ASLR 技术使得每次运行的程序在不同的地址空间中加载模块和映射文件，这有助于防御内存攻击。

在 Linux 系统中，文件映射区通常位于堆区域（heap）和堆栈区域（stack）之间。虚拟内存的典型布局如下：

![虚拟内存布局](https://cdn.jsdelivr.net/gh/aqjsp/Pictures/image-20240912232226670.png)

#### 2. 位置信息的维护

操作系统通过内核的数据结构和页面表（Page Table）来维护虚拟内存区域的位置信息，具体如下：

##### 2.1. 内存区域描述符 (Memory Region Descriptors)

操作系统使用内存区域描述符来维护进程虚拟地址空间中各个内存区域的信息。每个内存区域（如文件映射区、堆、栈等）都有对应的描述符，描述符包含以下信息：

- 起始地址和结束地址：用于标记该内存区域在虚拟地址空间中的位置和大小。
- 权限：定义该区域的访问权限，如可读、可写、可执行等。
- 映射的文件信息：对于文件映射区，描述符还包含被映射的文件的相关信息，如文件句柄、文件偏移量等。

在 Linux 系统中，虚拟地址空间的区域信息通过 `vm_area_struct` 数据结构来表示，它包含了每个内存区域的起始地址、大小、访问权限等元数据。

##### 2.2. 页面表 (Page Table)

页面表是虚拟内存管理的核心数据结构，负责维护虚拟地址和物理地址之间的映射关系。对于文件映射区，页面表将虚拟地址与文件内容所在的物理存储页进行关联：

- 当进程访问文件映射区中的某个地址时，如果该地址的页面还没有映射到物理内存中，会触发**页面缺失**（page fault）异常。
- 操作系统捕捉到页面缺失后，会从文件系统读取对应的文件内容，加载到物理内存的某个页中，并更新页面表，使该虚拟地址映射到新的物理页。

通过页面表的管理，操作系统可以高效地将文件的内容加载到内存中，按需提供映射文件的内存页。

##### 2.3. 虚拟内存管理单元（VMA）

操作系统内核使用虚拟内存区域（Virtual Memory Areas, VMA）来管理进程的内存布局。VMA 是用于描述进程的连续虚拟地址区域的结构体。对于文件映射的区域，操作系统会将该区域分配为一个 VMA。

- VMA 包含文件映射区的起始地址和大小。
- VMA 还记录了映射文件的路径、偏移量和访问权限（如 `PROT_READ`, `PROT_WRITE`）。

在 Linux 中，可以通过命令 `cat /proc/[pid]/maps` 来查看某个进程的内存映射情况，其中列出了每个 VMA 的起始地址、结束地址、权限和映射文件路径等信息。

例如：

```
7f4d88c00000-7f4d88c21000 r-xp 00000000 08:01 13172 /lib/x86_64-linux-gnu/ld-2.27.so
```

表示文件 `/lib/x86_64-linux-gnu/ld-2.27.so` 被映射到虚拟地址空间 `0x7f4d88c00000` 到 `0x7f4d88c21000`，并具有只读和可执行权限。

#### 3. 文件映射区的维护机制

- 内存映射文件的创建：通过系统调用如 `mmap`，可以请求操作系统将文件映射到虚拟地址空间。操作系统在内部会创建 VMA 和对应的页面表条目，维护该映射的虚拟地址到物理页的关系。
- 映射区的回收：当映射的文件不再使用时（例如进程结束或调用 `munmap` 解除映射），操作系统会释放对应的 VMA 和物理内存资源。
- 分页机制：如果文件映射区中的某些内容长时间没有被访问，操作系统可能会将这些页面调出内存，等到再次访问时再重新从磁盘加载。

### 9、进程间有哪些通信方式？

#### 1. 管道

- 无名管道：
  - 特点：用于父子进程之间的通信，通常是单向的，父进程创建管道并与子进程共享。
  - 实现：管道实际上是内核缓冲区，数据从一端写入，另一端读取。
  - 优点：简单、效率高，常用于进程间的流式数据传输。
  - 缺点：只能在有亲缘关系的进程（父子进程）之间通信，且单向通信。
- 命名管道：
  - 特点：支持任意不相关进程之间的双向通信。
  - 实现：通过文件系统中的特殊文件来实现，使用 `mkfifo` 命令创建命名管道。
  - 优点：可以在无亲缘关系的进程之间通信，双向通信。
  - 缺点：仍然是基于内核缓冲区，速度上有一定的限制。

#### 2. 信号

- 特点：用于通知进程某个事件发生，信号是一种异步通信机制。每个进程可以对某些信号进行处理或忽略。
- 实现：信号由操作系统内核发送给进程，进程通过捕捉、处理或忽略信号来响应。
- 优点：简单、系统开销小，用于通知或控制进程（如终止进程、暂停进程等）。
- 缺点：传递的信息量有限（通常是信号编号），不适合传递大量数据。

#### 3. 消息队列

- 特点：消息队列是可以让进程之间通过消息进行通信的机制，支持有序、非阻塞地传递消息。
- 实现：消息队列存储在内核中，通过消息队列ID标识。进程可以向消息队列发送和接收消息。
- 优点：支持消息的优先级、无亲缘关系进程的通信、传递结构化数据。
- 缺点：需要内核管理，且存在系统限制（如消息队列长度和大小的限制）。

#### 4. 共享内存

- 特点：共享内存是进程间通信中最快的一种方式，它允许多个进程直接访问相同的内存区域。
- 实现：操作系统分配一个共享内存段，多个进程通过映射到相同的内存区域实现通信。
- 优点：速度极快，因为无需通过操作系统内核传递数据，直接访问内存即可。
- 缺点：需要进程自己实现同步机制（如信号量或互斥锁）来防止数据竞争，容易引发并发问题。

#### 5. 信号量

- 特点：信号量是一种进程同步机制，主要用于控制进程对共享资源的访问，通常与共享内存等通信机制配合使用。
- 实现：信号量是一个计数器，表示可用资源的数量，进程可以通过 P 操作（等待）和 V 操作（释放）对资源进行申请和释放。
- 优点：可以用于进程同步和控制共享资源的访问。
- 缺点：实现和使用相对复杂，无法直接传递数据。

#### 6. 套接字

- 特点：套接字是一种更为通用的进程间通信机制，支持同一台计算机上的进程通信，也支持网络上的进程通信。
- 实现：基于 TCP 或 UDP 协议，通过套接字接口进行通信。
- 优点：支持跨网络通信，进程间通信的范围广泛。
- 缺点：传输效率相对较低，特别是在本地进程通信中。

#### 7. 管道文件

- 特点：这是 UNIX 域套接字的一种形式，专用于同一台计算机上的进程间通信，不使用网络协议。
- 实现：通过创建特殊的套接字文件用于本地通信，效率比传统的网络套接字高。
- 优点：不依赖网络栈，速度更快，支持双向通信。
- 缺点：只能用于本地进程通信。

#### 8. 内存映射文件

- 特点：将文件或设备的内容映射到进程的虚拟地址空间，进程可以像访问内存一样直接操作文件内容，从而实现通信。
- 实现：通过 `mmap` 系统调用将文件映射到内存中，多个进程可以通过映射同一文件来共享数据。
- 优点：传输效率高，适合大数据量的共享和交换。
- 缺点：需要处理数据同步问题，使用较为复杂。

#### 各种方式的对比与应用场景：

| 通信方式     | 传递数据   | 同步   | 适用场景                           |
| ------------ | ---------- | ------ | ---------------------------------- |
| 无名管道     | 字节流     | 不支持 | 父子进程间的数据流传输             |
| 命名管道     | 字节流     | 不支持 | 无亲缘关系进程间的数据流传输       |
| 消息队列     | 结构化消息 | 支持   | 多个进程间有序、异步通信           |
| 共享内存     | 任意数据   | 不支持 | 高速的数据共享，需同步机制         |
| 信号量       | 无数据     | 支持   | 进程同步和共享资源的访问控制       |
| 套接字       | 字节流     | 不支持 | 网络通信和本地进程间通信           |
| 内存映射文件 | 任意数据   | 不支持 | 进程间共享文件数据，适合大数据传输 |

### 10、共享内存怎么创建映射的，怎么知道映射内存的地址？

#### 1. 创建共享内存

1. 使用 `shm_open` 创建共享内存对象：

   - `shm_open` 函数用于创建或打开一个 POSIX 共享内存对象。

   - 函数原型：

     ```c++
     int shm_open(const char *name, int oflag, mode_t mode);
     ```

   - `name` 是共享内存对象的名字（以 `/` 开头），`oflag` 是打开模式标志（如 `O_CREAT` 和 `O_RDWR`），`mode` 是权限（如 `0666`）。

2. 使用 `ftruncate` 设置共享内存大小：

   - `ftruncate` 函数用于设置共享内存对象的大小。

   - 函数原型：

     ```c++
     int ftruncate(int fd, off_t length);
     ```

   - `fd` 是通过 `shm_open` 获取的文件描述符，`length` 是共享内存的大小（以字节为单位）。

#### 2. 映射共享内存

使用 `mmap` 映射共享内存到进程的虚拟地址空间：

- `mmap` 函数将共享内存对象映射到进程的地址空间。

- 函数原型：

  ```c++
  void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);
  ```

- `addr` 是建议的映射地址（通常为 `NULL`），`length` 是映射的大小，`prot` 指定内存保护（如 `PROT_READ | PROT_WRITE`），`flags` 指定映射类型（如 `MAP_SHARED`），`fd` 是共享内存对象的文件描述符，`offset` 是映射的偏移量。

#### 3. 获取共享内存的地址

调用 `mmap` 函数后，它会返回映射区域的起始地址。这个地址就是进程访问共享内存的起始地址。

简单实现：

```c++
#include <sys/mman.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main() {
    const char *name = "/my_shm";
    size_t size = 4096;

    // 创建共享内存对象
    int shm_fd = shm_open(name, O_CREAT | O_RDWR, 0666);
    if (shm_fd < 0) {
        perror("shm_open");
        return 1;
    }

    // 设置共享内存大小
    if (ftruncate(shm_fd, size) == -1) {
        perror("ftruncate");
        return 1;
    }

    // 映射共享内存
    void *ptr = mmap(NULL, size, PROT_READ | PROT_WRITE, MAP_SHARED, shm_fd, 0);
    if (ptr == MAP_FAILED) {
        perror("mmap");
        return 1;
    }

    // 写数据到共享内存
    strcpy((char *)ptr, "Hello, Shared Memory!");

    // 解除映射
    if (munmap(ptr, size) == -1) {
        perror("munmap");
        return 1;
    }

    // 关闭共享内存对象
    close(shm_fd);
    shm_unlink(name);

    return 0;
}
```

### 11、共享内存的互斥访问具体怎么实现，锁和信号量怎么在两个进程间共享？

使用互斥锁（Mutex）和信号量（Semaphore）进行同步。

#### 互斥锁（Mutex）

互斥锁（mutex）是一种同步机制，用于保证同一时刻只有一个线程（或进程）能够访问共享资源。使用互斥锁可以防止多个进程同时对共享内存进行修改，从而保证数据一致性。

POSIX 互斥锁（pthread_mutex_t）：

- 通过 `pthread_mutexattr_t` 和 `pthread_mutex_init` 初始化互斥锁。
- 互斥锁的内存区域可以放在共享内存区域中。
- 通过 `pthread_mutex_lock` 和 `pthread_mutex_unlock` 操作互斥锁。

```c++
#include <pthread.h>
#include <sys/mman.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
    const char *name = "/my_shm";
    size_t size = sizeof(pthread_mutex_t) + 4096;

    // 创建共享内存对象
    int shm_fd = shm_open(name, O_CREAT | O_RDWR, 0666);
    if (shm_fd < 0) {
        perror("shm_open");
        return 1;
    }

    // 设置共享内存大小
    if (ftruncate(shm_fd, size) == -1) {
        perror("ftruncate");
        return 1;
    }

    // 映射共享内存
    void *ptr = mmap(NULL, size, PROT_READ | PROT_WRITE, MAP_SHARED, shm_fd, 0);
    if (ptr == MAP_FAILED) {
        perror("mmap");
        return 1;
    }

    // 初始化互斥锁
    pthread_mutex_t *mutex = (pthread_mutex_t *)ptr;
    pthread_mutexattr_t attr;
    pthread_mutexattr_init(&attr);
    pthread_mutexattr_setpshared(&attr, PTHREAD_PROCESS_SHARED);
    pthread_mutex_init(mutex, &attr);

    // 使用互斥锁进行同步
    pthread_mutex_lock(mutex);
    // 访问和修改共享内存区域的代码
    pthread_mutex_unlock(mutex);

    // 解除映射和关闭文件描述符
    if (munmap(ptr, size) == -1) {
        perror("munmap");
        return 1;
    }
    close(shm_fd);

    return 0;
}
```

#### 信号量（Semaphore）

信号量是一种同步机制，用于控制访问共享资源的数量。信号量可以用于同步多个进程对共享内存的访问。

POSIX 信号量（`sem_t`）：

- 通过 `sem_open` 创建命名信号量，`sem_wait` 和 `sem_post` 操作信号量。
- 信号量的内存区域可以放在共享内存区域中。

```c++
#include <semaphore.h>
#include <sys/mman.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
    const char *name = "/my_shm";
    size_t size = sizeof(sem_t) + 4096;

    // 创建共享内存对象
    int shm_fd = shm_open(name, O_CREAT | O_RDWR, 0666);
    if (shm_fd < 0) {
        perror("shm_open");
        return 1;
    }

    // 设置共享内存大小
    if (ftruncate(shm_fd, size) == -1) {
        perror("ftruncate");
        return 1;
    }

    // 映射共享内存
    void *ptr = mmap(NULL, size, PROT_READ | PROT_WRITE, MAP_SHARED, shm_fd, 0);
    if (ptr == MAP_FAILED) {
        perror("mmap");
        return 1;
    }

    // 初始化信号量
    sem_t *sem = (sem_t *)ptr;
    sem_init(sem, 1, 1); // 第二个参数为1表示进程间共享，第三个参数是初始值

    // 使用信号量进行同步
    sem_wait(sem);
    // 访问和修改共享内存区域的代码
    sem_post(sem);

    // 解除映射和关闭文件描述符
    if (munmap(ptr, size) == -1) {
        perror("munmap");
        return 1;
    }
    close(shm_fd);

    return 0;
}
```

### 12、手撕：岛屿数量

#### 思路

1. 遍历网格：我们需要遍历整个二维网格 `grid` 中的每一个位置。

2. 深度优先搜索 (DFS)：

   - 每当遇到一个 `'1'`，即表示找到一个新的岛屿。

   - 从这个 `'1'` 开始，使用 DFS 将与其相连的所有 `'1'` 都标记为 `'0'`，表示这些部分已经被访问过。

   - 继续遍历下一个位置，直到网格遍历完毕。

3. 计数：每次执行 DFS 后，岛屿的数量加 1。

4. 最终结果：遍历完整个网格后，返回岛屿的总数量。

#### 参考代码

##### C++

```c++
#include <vector>
#include <iostream>

using namespace std;

class Solution {
public:
    // 计算岛屿的数量
    int numIslands(vector<vector<char>>& grid) {
        if (grid.empty()) return 0;

        int num_islands = 0;  // 记录岛屿数量
        int rows = grid.size();  // 网格的行数
        int cols = grid[0].size();  // 网格的列数

        // 遍历网格中的每一个位置
        for (int i = 0; i < rows; ++i) {
            for (int j = 0; j < cols; ++j) {
                // 如果当前位置是'1'，则表示找到一个新的岛屿
                if (grid[i][j] == '1') {
                    ++num_islands;  // 岛屿数量加1
                    dfs(grid, i, j);  // 使用DFS将与此'1'相连的所有'1'都标记为'0'
                }
            }
        }

        return num_islands;  // 返回岛屿的数量
    }

private:
    // 深度优先搜索，将相连的'1'全部标记为'0'
    void dfs(vector<vector<char>>& grid, int i, int j) {
        int rows = grid.size();
        int cols = grid[0].size();

        // 如果越界或者当前位置不是'1'，直接返回
        if (i < 0 || i >= rows || j < 0 || j >= cols || grid[i][j] != '1') {
            return;
        }

        grid[i][j] = '0';  // 将当前'1'标记为'0'，表示已访问

        // 递归访问当前位置的上下左右四个方向
        dfs(grid, i + 1, j);  // 下
        dfs(grid, i - 1, j);  // 上
        dfs(grid, i, j + 1);  // 右
        dfs(grid, i, j - 1);  // 左
    }
};

// 测试函数
int main() {
    Solution solution;

    vector<vector<char>> grid1 = {
        {'1','1','1','1','0'},
        {'1','1','0','1','0'},
        {'1','1','0','0','0'},
        {'0','0','0','0','0'}
    };

    vector<vector<char>> grid2 = {
        {'1','1','0','0','0'},
        {'1','1','0','0','0'},
        {'0','0','1','0','0'},
        {'0','0','0','1','1'}
    };

    cout << "Grid 1岛屿数量: " << solution.numIslands(grid1) << endl;  // 输出: 1
    cout << "Grid 2岛屿数量: " << solution.numIslands(grid2) << endl;  // 输出: 3

    return 0;
}
```
