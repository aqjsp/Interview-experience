# 金山C++一面，强度拉满。。。

> 来源：https://www.nowcoder.com/share/jump/1729599335273

### 1、c++程序的内存分布？

#### 1. 代码段（Text Segment）

- 功能：存储程序的可执行代码，即编译后的机器指令。
- 特点：这部分内存是只读的，通常在程序加载时由操作系统分配，并且在整个程序运行期间保持不变。
- 管理：由操作系统管理，程序员无需干预。

#### 2. 数据段（Data Segment）

数据段可以进一步细分为几个子段：

##### 2.1 已初始化数据段（Initialized Data Segment）

- 功能：存储已初始化的全局变量和静态变量。
- 特点：这些变量在程序启动时已经被赋予初始值。
- 管理：由编译器在编译时分配内存，程序运行期间由操作系统管理。

##### 2.2 未初始化数据段（Uninitialized Data Segment 或 BSS）

- 功能：存储未初始化的全局变量和静态变量。
- 特点：这些变量在程序启动时被初始化为零。
- 管理：由编译器在编译时分配内存，程序运行期间由操作系统管理。

#### 3. 堆区（Heap）

- 功能：用于动态分配内存。
- 特点：
  - 通过 `new` 或 `malloc` 等函数分配内存。
  - 通过 `delete` 或 `free` 等函数释放内存。
  - 动态分配的内存生命周期由程序员管理。
  - 堆内存的分配和释放可能导致内存碎片。
- 管理：由程序员管理，不当的内存管理可能导致内存泄漏。

#### 4. 栈区（Stack）

- 功能：用于存放局部变量和函数参数。
- 特点：
  - 由编译器自动管理，函数调用时分配，函数返回时自动释放。
  - 栈内存分配运算内置在处理器的指令集中，效率高。
  - 栈的大小有限，通常在几 MB 到几十 MB 之间。
- 管理：由编译器自动管理，程序员无需干预。

#### 5. 常量区（Const Segment）

- 功能：存储常量数据，如字符串常量。
- 特点：这部分内存是只读的，通常与代码段一起存储。
- 管理：由编译器在编译时分配内存，程序运行期间由操作系统管理。

### 2、堆和栈的区别？

#### 1. 管理方式

- 栈：由操作系统自动分配和释放。每当函数被调用时，操作系统会在栈上为该函数的局部变量和参数分配内存；当函数返回时，这些内存会被自动释放。栈的管理是自动的，无需程序员手动干预。
- 堆：由程序员手动分配和释放。通过 `new` 或 `malloc` 等函数分配内存，通过 `delete` 或 `free` 等函数释放内存。堆的管理需要程序员谨慎处理，不当的内存管理可能导致内存泄漏。

#### 2. 空间大小

- 栈：每个进程的栈大小相对较小，通常在几 MB 到几十 MB 之间。例如，64 位的 Windows 系统默认栈大小为 1MB，64 位的 Linux 系统默认栈大小为 10MB。
- 堆：堆的大小可以非常大，理论上可以达到虚拟内存的大小。堆可以动态扩展，但实际大小受系统可用内存和配置限制。

#### 3. 生长方向

- 栈：栈的生长方向是从高地址向低地址生长。每次函数调用时，栈顶指针向下移动，为新函数的局部变量和参数分配内存。
- 堆：堆的生长方向是从低地址向高地址生长。每次分配内存时，堆顶指针向上移动，为新分配的内存块分配空间。

#### 4. 分配方式

- 栈：栈有两种分配方式：静态分配和动态分配。静态分配由操作系统完成，如局部变量的分配。动态分配由 `alloca()` 函数完成，但栈的动态分配仍然由操作系统管理，无需手动释放。
- 堆：堆都是动态分配的，没有静态分配的堆。堆的分配和释放由 C/C++ 提供的库函数或运算符完成，如 `malloc()`、`free()`、`new` 和 `delete`。

#### 5. 分配效率

- 栈：栈的分配和释放非常高效。操作系统提供了专门的寄存器存放栈的地址，压栈和出栈操作都有专门的指令执行，这使得栈的效率非常高。
- 堆：堆的分配和释放相对复杂。每次分配内存时，系统需要遍历空闲内存链表，找到合适的内存块，并记录分配的大小。频繁的内存分配和释放可能导致内存碎片，影响性能。

#### 6. 存放内容

- 栈：栈主要用于存放函数的局部变量、参数、返回地址和寄存器内容等。当主函数调用另一个函数时，栈会保存当前函数的执行断点和相关上下文信息，确保函数调用结束后能正确返回。
- 堆：堆主要用于存放动态分配的数据结构，如动态数组、链表、树等。堆中具体存放的内容由程序员决定，可以是任何类型的数据。

#### 7. 生命周期

- 栈：栈中存储的数据的生命周期与函数的执行周期一致。当函数执行完毕后，栈中的数据自动被释放。
- 堆：堆中存储的数据的生命周期由程序员控制。如果分配的内存未被释放，其生命周期等同于程序的生命周期。

### 3、内存泄漏怎么办？

#### 1. 理解内存泄漏的定义和原因

内存泄漏是指程序中已动态分配的堆内存由于某种原因程序未释放或无法释放，导致系统内存的浪费。常见的内存泄漏原因包括：

- 未释放动态分配的内存：通过 `new` 或 `malloc` 分配的内存未通过 `delete` 或 `free` 释放。
- 循环引用：特别是在使用智能指针时，如果存在循环引用，指针之间的引用计数可能永远不会降为零。
- 异常导致的资源泄漏：在发生异常时，如果没有适当地释放资源，就可能发生内存泄漏。
- 虚析构问题：在继承关系中，如果父类的析构函数不是虚函数，派生类的析构函数可能不会被调用，导致内存泄漏。

#### 2. 检测内存泄漏的方法

##### 2.1 使用工具

- Valgrind：Valgrind 是一个强大的内存调试工具，可以检测内存泄漏、内存访问错误等问题。适用于 C/C++ 程序。

```
valgrind --leak-check=full ./your_program
```

- Visual Studio：Visual Studio 提供了内存诊断工具，可以在调试模式下检测内存泄漏。

- LeakSanitizer：LeakSanitizer 是 LLVM 项目的一部分，可以与 AddressSanitizer 一起使用，检测内存泄漏。

```
clang -fsanitize=address -fsanitize=leak your_program.cpp -o your_program
./your_program
```

##### 2.2 手动检测

- 代码审查：定期进行代码审查，检查内存分配和释放是否匹配。
- 日志记录：在关键位置添加日志记录，监控内存分配和释放的情况。
- 单元测试：编写单元测试，确保每个模块在使用完内存后都能正确释放。

#### 3. 解决内存泄漏的方法

##### 3.1 使用智能指针

C++11及以上版本引入了智能指针，如`std::shared_ptr`和`std::unique_ptr`，可以在对象生命周期结束时自动释放内存。

```c++
#include <memory>

void useSmartPointers() {
    std::unique_ptr<int> uniquePtr(new int(10));
    std::shared_ptr<int> sharedPtr(new int(20));

    // 不需要手动释放内存
}
```

##### 3.2 使用 RAII（资源获取即初始化）原则

RAII原则确保资源在对象构造时获得，并在对象析构时释放。通过这种方式，可以减少手动管理资源的需要，降低内存泄漏的风险。

```c++
class FileHandler {
public:
    FileHandler(const std::string& fileName)
        : file(fopen(fileName.c_str(), "r")) {
        if (!file) {
            throw std::runtime_error("Failed to open file");
        }
    }

    ~FileHandler() {
        if (file) {
            fclose(file);
        }
    }

private:
    FILE* file;
};
```

##### 3.3 仔细处理异常

在可能发生异常的地方使用 `try-catch` 块，确保在异常发生时能够正确释放资源。

```c++
void handleException() {
    int* myData = nullptr;
    try {
        myData = new int[100];
        // 可能会发生异常
        // ...
    } catch (...) {
        // 处理异常
    }

    delete[] myData; // 确保释放内存
}
```

##### 3.4 继承关系中的虚析构

确保基类的析构函数是虚函数，这样派生类的析构函数也能被调用。

```c++
class Base {
public:
    virtual ~Base() {
        // 虚析构函数
    }
};

class Derived : public Base {
public:
    ~Derived() {
        // 派生类的析构函数
    }
};
```

### 4、智能指针，哪几种？

#### 1. `std::unique_ptr`

`std::unique_ptr` 是 C++11 引入的一种独占所有权的智能指针。它确保了资源的独占所有权，即一个资源在同一时刻只能被一个 `std::unique_ptr` 对象拥有。当 `std::unique_ptr` 对象被销毁时，它所管理的资源也会被自动释放。

**特点：**

- 独占所有权：一个 `std::unique_ptr` 对象不能被复制，但可以通过移动语义（`std::move`）将所有权转移给另一个 `std::unique_ptr` 对象。
- 自动释放资源：当 `std::unique_ptr` 对象超出作用域或被显式销毁时，它所管理的资源会被自动释放。
- 轻量级：`std::unique_ptr` 的实现非常轻量级，几乎不会带来额外的性能开销。

```c++
#include <memory>
#include <iostream>

void useUniquePtr() {
    std::unique_ptr<int> uniquePtr(new int(10));
    // 使用 uniquePtr
    std::cout << *uniquePtr << std::endl;

    // 移动所有权
    std::unique_ptr<int> anotherUniquePtr = std::move(uniquePtr);
    // uniquePtr 现在为空
    if (!uniquePtr) {
        std::cout << "uniquePtr is now empty" << std::endl;
    }
}
```

#### 2. `std::shared_ptr`

`std::shared_ptr` 是 C++11 引入的一种共享所有权的智能指针。它可以允许多个 `std::shared_ptr` 对象共享同一个资源。资源的生命周期由引用计数管理，当最后一个 `std::shared_ptr` 对象被销毁时，资源会被自动释放。

**特点：**

- 共享所有权：多个 `std::shared_ptr` 对象可以共享同一个资源。
- 引用计数：通过引用计数来管理资源的生命周期。当引用计数降为零时，资源会被自动释放。
- 线程安全：引用计数的增减操作是线程安全的，但对资源的访问需要额外的同步机制。

```c++
#include <memory>
#include <iostream>

void useSharedPtr() {
    std::shared_ptr<int> sharedPtr1(new int(20));
    std::shared_ptr<int> sharedPtr2 = sharedPtr1;

    // 使用 sharedPtr1 和 sharedPtr2
    std::cout << *sharedPtr1 << std::endl;
    std::cout << *sharedPtr2 << std::endl;

    // 引用计数
    std::cout << "Use count: " << sharedPtr1.use_count() << std::endl;
}
```

#### 3. `std::weak_ptr`

`std::weak_ptr` 是 C++11 引入的一种观察者智能指针，它与 `std::shared_ptr` 一起使用。`std::weak_ptr` 不增加资源的引用计数，因此不会影响资源的生命周期。它主要用于解决 `std::shared_ptr` 之间的循环引用问题。

**特点：**

- 观察者：`std::weak_ptr` 不拥有资源，只是观察资源的存在。
- 不增加引用计数：`std::weak_ptr` 不会影响资源的生命周期。
- 解决循环引用：通过 `std::weak_ptr` 可以打破 `std::shared_ptr` 之间的循环引用。

```c++
#include <memory>
#include <iostream>

class B;
class A {
public:
    std::shared_ptr<B> b;
};

class B {
public:
    std::weak_ptr<A> a;
};

void useWeakPtr() {
    std::shared_ptr<A> a = std::make_shared<A>();
    std::shared_ptr<B> b = std::make_shared<B>();

    a->b = b;
    b->a = a;

    // 使用 weak_ptr
    if (auto aPtr = b->a.lock()) {
        std::cout << "a is still alive" << std::endl;
    } else {
        std::cout << "a has been destroyed" << std::endl;
    }
}
```

#### 4. `std::auto_ptr`（已废弃）

`std::auto_ptr` 是 C++98 引入的一种早期的智能指针，但它在 C++11 中已被废弃，不推荐使用。`std::auto_ptr` 也管理独占所有权，但它的实现有一些缺陷，如拷贝和赋值操作会转移所有权，容易导致意外的内存问题。

**特点：**

- 独占所有权：一个 `std::auto_ptr` 对象不能被复制，但可以通过拷贝和赋值操作转移所有权。
- 已废弃：由于其设计上的缺陷，C++11 引入了 `std::unique_ptr` 作为更好的替代方案。

```c++
#include <memory>
#include <iostream>

void useAutoPtr() {
    std::auto_ptr<int> autoPtr(new int(30));
    // 使用 autoPtr
    std::cout << *autoPtr << std::endl;

    // 拷贝操作会转移所有权
    std::auto_ptr<int> anotherAutoPtr = autoPtr;
    // autoPtr 现在为空
    if (!autoPtr) {
        std::cout << "autoPtr is now empty" << std::endl;
    }
}
```

### 5、循环引用计数最后是多少？

循环引用通常发生在两个或多个对象相互引用的情况下，导致它们的引用计数无法降为零，从而无法被自动回收。具体来说，当对象 A 持有对象 B 的引用，而对象 B 同时持有对象 A 的引用时，即使这些对象已经不再被外部引用，它们的引用计数仍然会保持为 1 或更高，因为它们互相引用。

#### 1. 基本示例

假设我们有两个类 `A` 和 `B`，它们分别持有对方的 `std::shared_ptr`，如下所示：

```c++
class A {
public:
    std::shared_ptr<B> b_ptr;
};

class B {
public:
    std::shared_ptr<A> a_ptr;
};
```

当我们创建 `A` 和 `B` 的对象，并使它们互相引用时：

```c++
std::shared_ptr<A> a = std::make_shared<A>();
std::shared_ptr<B> b = std::make_shared<B>();
a->b_ptr = b;
b->a_ptr = a;
```

例子中，`A` 对象持有一个指向 `B` 对象的 `std::shared_ptr`，`B` 对象也持有一个指向 `A` 对象的 `std::shared_ptr`。因此，`A` 和 `B` 的引用计数都为 1。即使 `a` 和 `b` 超出作用域，`A` 和 `B` 对象的引用计数也不会降为零，因为它们互相引用。

#### 2. 引用计数的详细解释

- 初始状态：`a` 和 `b` 的引用计数分别为 1。
- 互相引用：`a->b_ptr = b;` 和 `b->a_ptr = a;` 之后，`A` 和 `B` 的引用计数都增加到 1。
- 超出作用域：当 `a` 和 `b` 超出作用域时，它们的析构函数会被调用，导致 `A` 和 `B` 对象的引用计数各自减 1。但由于 `A` 和 `B` 互相引用，它们的引用计数仍然为 1。

#### 3. 解决循环引用

为了解决循环引用问题，可以使用 `std::weak_ptr`。`std::weak_ptr` 不增加所指向对象的引用计数，因此可以打破循环引用。

```c++
class A {
public:
    std::shared_ptr<B> b_ptr;
};

class B {
public:
    std::weak_ptr<A> a_ptr;
};

std::shared_ptr<A> a = std::make_shared<A>();
std::shared_ptr<B> b = std::make_shared<B>();
a->b_ptr = b;
b->a_ptr = a;

// 在使用 weak_ptr 时，需要先将其转换为 shared_ptr
if (auto a_locked = b->a_ptr.lock()) {
    // 使用 a_locked
}
```

#### 4. 最终引用计数

- **使用 `std::shared_ptr` 时**：在 `a` 和 `b` 超出作用域后，`A` 和 `B` 对象的引用计数仍然为 1，因为它们互相引用，导致内存泄漏。
- **使用 `std::weak_ptr` 时**：在 `a` 和 `b` 超出作用域后，`A` 和 `B` 对象的引用计数会降为 0，因为 `std::weak_ptr` 不增加引用计数，从而允许对象被正确回收。

### 6、shared_ptr线程安全吗？

#### 1. 引用计数的线程安全性

`std::shared_ptr` 的引用计数是线程安全的。这意味着多个线程可以同时增加或减少引用计数，而不会导致竞争条件。引用计数的增加和减少操作是原子的，通常通过原子操作或互斥锁来实现。因此，多个线程可以安全地拷贝同一个 `std::shared_ptr` 对象，而不会引起问题。

#### 2. 对 `std::shared_ptr` 对象本身的修改

虽然引用计数是线程安全的，但对 `std::shared_ptr` 对象本身的修改并不是线程安全的。具体来说，如果多个线程同时修改同一个 `std::shared_ptr` 对象（例如，通过 `reset`、`swap` 或赋值操作），则可能会导致数据竞争和未定义行为。这是因为这些操作涉及到多个步骤，例如释放旧的资源、更新指针和增加新的引用计数，这些步骤不是原子的。

#### 3. 对 `std::shared_ptr` 指向的资源的访问

`std::shared_ptr` 指向的资源本身并不是线程安全的。如果多个线程同时读写 `std::shared_ptr` 指向的资源，可能会导致数据竞争和未定义行为。例如，如果多个线程同时对 `std::shared_ptr<int>` 指向的整数进行自增操作，最终的结果可能不是预期的。

#### 4. 具体示例

##### 示例 1：引用计数更新，线程安全

```c++
std::shared_ptr<int> p = std::make_shared<int>(0);
constexpr int N = 10000;
std::vector<std::shared_ptr<int>> sp_arr1(N);
std::vector<std::shared_ptr<int>> sp_arr2(N);

void increment_count(std::vector<std::shared_ptr<int>>& sp_arr) {
    for (int i = 0; i < N; i++) {
        sp_arr[i] = p;
    }
}

std::thread t1(increment_count, std::ref(sp_arr1));
std::thread t2(increment_count, std::ref(sp_arr2));

t1.join();
t2.join();

std::cout << p.use_count() << std::endl; // always 20001
```

这个示例中，两个线程同时对同一个 `std::shared_ptr` 进行拷贝，引用计数的值总是 20001，这是线程安全的。

##### 示例 2：同时读写内存区域，线程不安全

```c++
std::shared_ptr<int> p = std::make_shared<int>(0);

void modify_memory() {
    for (int i = 0; i < 10000; i++) {
        (*p)++;
    }
}

std::thread t1(modify_memory);
std::thread t2(modify_memory);

t1.join();
t2.join();

std::cout << "Final value of p: " << *p << std::endl; // possible result: 16171, not 20000
```

这个示例中，两个线程同时对同一个 `std::shared_ptr` 指向的内存区域进行自增操作，最终的结果不是预期的 20000，这是线程不安全的。

##### 示例 3：直接修改 `std::shared_ptr` 对象本身的指向，线程不安全

```c++
std::shared_ptr<int> sp = std::make_shared<int>(1);

auto modify_sp_self = [&sp]() {
    for (int i = 0; i < 1000000; ++i) {
        sp = std::make_shared<int>(i);
    }
};

std::vector<std::thread> threads;
for (int i = 0; i < 10; ++i) {
    threads.emplace_back(modify_sp_self);
}

for (auto& t : threads) {
    t.join();
}
```

在这个示例中，多个线程同时修改同一个 `std::shared_ptr` 对象的指向，程序发生了异常终止。这是因为 `reset` 操作不是原子的，导致数据竞争。

#### 5. 解决方案

为了确保 `std::shared_ptr` 在多线程环境下的安全性，可以采取以下措施：

- **使用互斥锁**：在修改 `std::shared_ptr` 对象或其指向的资源时，使用互斥锁（`std::mutex`）来保护临界区。

- **使用原子操作**：对于简单的操作，可以使用 `std::atomic` 来确保原子性。

- **使用 `std::shared_ptr` 的原子操作**：C++ 标准库提供了`std::atomic<std::shared_ptr<T>>`，可以在多线程环境中安全地使用

  `std::shared_ptr`。

### 7、多线程使用shared_ptr如何保护数据安全？

#### 1. 使用互斥锁（Mutex）

互斥锁是最常见的同步机制之一，可以确保同一时间只有一个线程可以访问共享资源。通过在访问 `std::shared_ptr` 指向的数据或修改 `std::shared_ptr` 对象时使用互斥锁，可以有效防止数据竞争。

```c++
#include <iostream>
#include <thread>
#include <memory>
#include <mutex>

std::shared_ptr<int> shared_data = std::make_shared<int>(0);
std::mutex mtx;

void increment_data() {
    std::lock_guard<std::mutex> lock(mtx); // 自动管理锁的获取和释放
    (*shared_data)++;
    std::cout << "Incremented: " << *shared_data << std::endl;
}

int main() {
    std::vector<std::thread> threads;
    for (int i = 0; i < 10; ++i) {
        threads.emplace_back(increment_data);
    }

    for (auto& t : threads) {
        t.join();
    }

    std::cout << "Final value: " << *shared_data << std::endl;
    return 0;
}
```

#### 2. 使用 `std::atomic<std::shared_ptr<T>>`

C++ 标准库提供了 `std::atomic<std::shared_ptr<T>>`，可以在多线程环境中安全地使用 `std::shared_ptr`。这确保了 `std::shared_ptr` 对象的读取和写入操作是原子的。

```c++
#include <iostream>
#include <thread>
#include <memory>
#include <atomic>

std::atomic<std::shared_ptr<int>> shared_data(std::make_shared<int>(0));

void increment_data() {
    auto local_data = shared_data.load(); // 原子加载
    (*local_data)++;
    shared_data.store(local_data); // 原子存储
    std::cout << "Incremented: " << *local_data << std::endl;
}

int main() {
    std::vector<std::thread> threads;
    for (int i = 0; i < 10; ++i) {
        threads.emplace_back(increment_data);
    }

    for (auto& t : threads) {
        t.join();
    }

    std::cout << "Final value: " << *shared_data.load() << std::endl;
    return 0;
}
```

#### 3. 使用 `std::shared_mutex`（读写锁）

`std::shared_mutex` 允许多个读线程同时访问共享资源，但在写操作时独占锁。这可以提高读多写少场景下的性能。

```c++
#include <iostream>
#include <thread>
#include <memory>
#include <shared_mutex>

std::shared_ptr<int> shared_data = std::make_shared<int>(0);
std::shared_mutex rw_mtx;

void read_data() {
    std::shared_lock<std::shared_mutex> lock(rw_mtx); // 共享锁
    std::cout << "Read: " << *shared_data << std::endl;
}

void write_data() {
    std::unique_lock<std::shared_mutex> lock(rw_mtx); // 独占锁
    (*shared_data)++;
    std::cout << "Incremented: " << *shared_data << std::endl;
}

int main() {
    std::vector<std::thread> read_threads;
    std::vector<std::thread> write_threads;

    for (int i = 0; i < 10; ++i) {
        read_threads.emplace_back(read_data);
        write_threads.emplace_back(write_data);
    }

    for (auto& t : read_threads) {
        t.join();
    }

    for (auto& t : write_threads) {
        t.join();
    }

    std::cout << "Final value: " << *shared_data << std::endl;
    return 0;
}
```

#### 4. 使用 `std::condition_variable` 进行线程同步

在某些情况下，可能需要更复杂的线程同步机制。`std::condition_variable` 可以用于线程间的通信，确保在特定条件下执行某些操作。

```c++
#include <iostream>
#include <thread>
#include <memory>
#include <mutex>
#include <condition_variable>

std::shared_ptr<int> shared_data = std::make_shared<int>(0);
std::mutex mtx;
std::condition_variable cv;
bool ready = false;

void worker_thread() {
    std::unique_lock<std::mutex> lock(mtx);
    cv.wait(lock, [] { return ready; }); // 等待条件变量
    (*shared_data)++;
    std::cout << "Incremented: " << *shared_data << std::endl;
}

void main_thread() {
    std::this_thread::sleep_for(std::chrono::seconds(1)); // 模拟初始化时间
    std::lock_guard<std::mutex> lock(mtx);
    ready = true;
    cv.notify_all(); // 通知所有等待的线程
}

int main() {
    std::vector<std::thread> threads;
    for (int i = 0; i < 10; ++i) {
        threads.emplace_back(worker_thread);
    }

    std::thread main_th(main_thread);
    main_th.join();

    for (auto& t : threads) {
        t.join();
    }

    std::cout << "Final value: " << *shared_data << std::endl;
    return 0;
}
```

#### 5. 使用 `std::future` 和 `std::promise`

`std::future` 和 `std::promise` 提供了一种简单的异步通信机制，可以在多线程环境中传递数据。

```c++
#include <iostream>
#include <thread>
#include <memory>
#include <future>

std::shared_ptr<int> shared_data = std::make_shared<int>(0);

void increment_data(std::promise<void> done) {
    (*shared_data)++;
    std::cout << "Incremented: " << *shared_data << std::endl;
    done.set_value(); // 通知完成
}

int main() {
    std::vector<std::future<void>> futures;
    for (int i = 0; i < 10; ++i) {
        std::promise<void> promise;
        futures.push_back(promise.get_future());
        std::thread(increment_data, std::move(promise)).detach();
    }

    for (auto& f : futures) {
        f.get(); // 等待所有线程完成
    }

    std::cout << "Final value: " << *shared_data << std::endl;
    return 0;
}
```

### 8、条件变量伪唤醒？

条件变量的伪唤醒（Spurious Wakeup）是指在没有显式地调用 `notify_one()` 或 `notify_all()` 方法的情况下，等待在条件变量上的线程被意外唤醒的现象。尽管这种现象在实际应用中较为罕见，但它确实可能发生，并且需要开发者在设计多线程程序时予以考虑。

#### 伪唤醒的原因

伪唤醒的原因可以分为两类：操作系统层面和应用层代码层面。

**操作系统层面**：

- 硬件中断：某些硬件中断可能导致线程被意外唤醒。
- 内核调度器：操作系统内核调度器的某些行为可能导致线程被唤醒，即使没有显式的唤醒信号。
- 多核处理器：在多核处理器上，`pthread_cond_signal` 可能会激活多个等待线程，导致虚假唤醒。

**应用层代码层面**：

- 编程错误：例如，忘记在 `wait` 之前检查条件，或者在 `wait` 之后没有重新检查条件。
- 竞态条件：多个线程之间的竞争条件可能导致某些线程被意外唤醒。

#### 如何处理伪唤醒

为了避免伪唤醒导致的问题，通常建议在使用条件变量时，将 `wait` 操作放在一个 `while` 循环中，而不是使用 `if` 语句。这样可以确保即使发生伪唤醒，线程也会重新检查条件，从而避免错误的行为。

```c++
#include <iostream>
#include <thread>
#include <condition_variable>
#include <mutex>

std::condition_variable cv;
std::mutex mtx;
bool ready = false;

void consumer() {
    std::unique_lock<std::mutex> lock(mtx);
    while (!ready) {
        cv.wait(lock); // 伪唤醒处理
    }
    std::cout << "Consumer: Data is ready, processing..." << std::endl;
}

void producer() {
    std::this_thread::sleep_for(std::chrono::seconds(1)); // 模拟数据准备时间
    {
        std::lock_guard<std::mutex> lock(mtx);
        ready = true;
    }
    cv.notify_one();
}

int main() {
    std::thread t1(producer);
    std::thread t2(consumer);

    t1.join();
    t2.join();

    return 0;
}
```

### 9、unique_ptr转移所有权？

`std::unique_ptr` 是 C++11 引入的一种智能指针，它实现了独占式所有权，即同一时间内只有一个 `std::unique_ptr` 指向某个动态分配的对象。当 `std::unique_ptr` 被销毁时，所管理的对象也会被自动销毁，避免了内存泄漏。`std::unique_ptr` 的一个重要特性是支持移动语义，可以通过移动语义在对象之间进行所有权的转移，避免了不必要的资源拷贝操作。

#### 移动语义

C++11 引入了右值引用（rvalue reference）和移动语义（move semantics），使得资源的所有权可以在对象之间高效地转移。`std::unique_ptr` 利用这一特性，提供了两种主要的方式来进行所有权的转移：使用 `std::move` 函数和使用 `reset` 方法。

##### 使用 `std::move` 函数

`std::move` 函数将一个左值（lvalue）转换为右值（rvalue），从而允许 `std::unique_ptr` 的移动构造函数或移动赋值运算符被调用，实现所有权的转移。

```c++
#include <iostream>
#include <memory>

int main() {
    // 创建一个 unique_ptr 实例
    std::unique_ptr<int> ptr1(new int(10));
    
    // 使用 std::move 转移所有权
    std::unique_ptr<int> ptr2 = std::move(ptr1);
    
    // 此时 ptr1 已经不再拥有对象的所有权
    // std::cout << *ptr1 << std::endl; // 这行代码会导致编译错误或运行时错误
    
    // ptr2 成为唯一拥有者
    std::cout << *ptr2 << std::endl; // 输出 10
    
    return 0;
}
```

这个例子中，`ptr1` 的所有权被转移到了 `ptr2`，`ptr1` 被置为空（即 `nullptr`）。

##### 使用 `reset` 方法

`reset` 方法可以将 `std::unique_ptr` 指向新的对象，并释放原来管理的对象。如果在 `reset` 方法中传递一个新的原始指针，原来的对象会被销毁，新的对象会被管理。

```c++
#include <iostream>
#include <memory>

int main() {
    // 创建一个 unique_ptr 实例
    std::unique_ptr<int> ptr1(new int(10));
    
    // 使用 reset 方法转移所有权
    std::unique_ptr<int> ptr2;
    ptr2.reset(ptr1.release());
    
    // 此时 ptr1 已经不再拥有对象的所有权
    // std::cout << *ptr1 << std::endl; // 这行代码会导致编译错误或运行时错误
    
    // ptr2 成为唯一拥有者
    std::cout << *ptr2 << std::endl; // 输出 10
    
    return 0;
}
```

这个例子中，`ptr1` 通过 `release` 方法释放了对对象的控制权，并返回了一个裸指针。`ptr2` 使用 `reset` 方法接收这个裸指针，成为新的所有者。

#### 返回 `unique_ptr`

`std::unique_ptr` 还可以作为函数的返回值，这在返回动态分配的对象时非常有用。通过返回 `unique_ptr`，可以确保资源的所有权被正确地转移。

```c++
#include <iostream>
#include <memory>

std::unique_ptr<int> createUniquePtr() {
    return std::unique_ptr<int>(new int(20));
}

int main() {
    std::unique_ptr<int> ptr = createUniquePtr();
    std::cout << *ptr << std::endl; // 输出 20
    
    return 0;
}
```

这个例子中，`createUniquePtr` 函数返回一个 `std::unique_ptr`，主函数中的 `ptr` 接收了这个返回值，成为新的所有者。

### 10、move实现方式？

见9

### 11、完美转发有什么用？

完美转发其核心目的是在函数模板中将参数“完美”地转发给内部调用的其他函数，同时保持参数的类型和值类别（左值或右值）不变。通过完美转发，可以避免不必要的拷贝操作，提高程序的性能和效率。

#### 完美转发的主要用途

1. 避免不必要的拷贝操作：
   - 在传统的函数调用中，传递参数时可能会发生不必要的拷贝，尤其是当参数是大型对象或临时对象时。完美转发通过保持参数的左值或右值属性，可以避免这些不必要的拷贝操作，从而提高性能。
   - 例如，在创建对象时，如果参数是右值，可以使用移动语义（move semantics）来避免拷贝。
2. 提高模板函数的通用性和灵活性：
   - 完美转发使得模板函数能够处理各种类型的参数，包括左值、右值、常量和非常量等。这使得模板函数更加通用，适用于更多的场景。
   - 例如，`std::make_unique` 和 `std::make_shared` 就是利用完美转发来创建 `std::unique_ptr` 和 `std::shared_ptr` 的，它们可以接受任意数量和类型的参数。
3. 支持委托构造函数：
   - 在类的设计中，委托构造函数（delegating constructors）可以将一部分初始化工作委托给另一个构造函数。完美转发可以确保在委托过程中参数的类型和值类别保持不变。
   - 例如，一个类可能有多个构造函数，其中一个构造函数可以将参数转发给另一个构造函数，而不需要进行额外的拷贝。
4. 实现高效的可变参数模板函数：
   - 完美转发可以用于实现接受任意数量和类型参数的函数，如 `std::bind` 和 `std::tuple` 的构造函数。这些函数需要能够处理各种类型的参数，并且在内部调用其他函数时保持参数的原始状态。
   - 例如，`std::bind` 可以将参数完美地转发给绑定的函数，确保参数的类型和值类别不变。

#### 实现机制

完美转发的实现依赖于两个关键的 C++11 特性：模板参数推导和右值引用（包括通用引用）。

1. 模板参数推导：模板参数推导允许编译器根据传入的参数类型自动推导模板参数。这使得模板函数可以接受各种类型的参数。
2. 右值引用和通用引用：
   - 右值引用（rvalue reference）通过 `T&&` 表示，可以绑定到右值。当结合模板参数推导时，`T&&` 可以绑定到左值和右值，称为通用引用（universal reference）。
   - 通用引用的类型推导规则如下：
     - 如果 `T` 是一个引用类型，`T&&` 会折叠为 `T`。
     - 如果 `T` 不是一个引用类型，`T&&` 会保持为右值引用。
3. `std::forward` 函数：

`std::forward`是 C++11 标准库提供的一个函数模板，用于在转发参数时保持其左值或右值属性。`std::forward`的使用方法如下：

```
template <typename T>
void forward(T&& param) {
    someFunction(std::forward<T>(param));
}
```

`std::forward<T>(param)` 会根据 `T` 的类型推导结果，将 `param` 转换为相应的左值引用或右值引用。

一个简单的示例，展示了如何使用完美转发来实现一个通用的日志记录函数：

```c++
#include <iostream>
#include <utility>

// 处理左值
void process(int& i) {
    std::cout << "处理左值: " << i << std::endl;
}

// 处理右值
void process(int&& i) {
    std::cout << "处理右值: " << i << std::endl;
}

// 完美转发的函数模板
template <typename T>
void logAndProcess(T&& param) {
    // 调用 process 函数，同时保持 param 的左值/右值特性
    process(std::forward<T>(param));
}

int main() {
    int a = 5;
    logAndProcess(a);      // a 是左值，将调用处理左值的重载
    logAndProcess(10);     // 10 是右值，将调用处理右值的重载
    return 0;
}
```

这个示例中，`logAndProcess` 函数模板通过 `std::forward` 实现了完美转发。无论传递给 `logAndProcess` 的是左值还是右值，`std::forward` 都能确保参数以其原始值类别传递给 `process` 函数。

#### 引用折叠

引用折叠（Reference Collapsing）是 C++ 中的一个规则，与右值引用和完美转发相关，尤其是在模板编程和类型推导中尤为重要。引用折叠规则决定了当模板参数和函数参数中的引用类型相互作用时，最终的类型应该是什么。引用折叠的规则如下：

- `T& &` 折叠为 `T&`
- `T& &&` 折叠为 `T&`
- `T&& &` 折叠为 `T&`
- `T&& &&` 折叠为 `T&&`

引用折叠确保了引用的最终类型要么是左值引用（`T&`），要么是右值引用（`T&&`）。

### 12、模板的特化和偏特化？

#### 模板特化

模板特化是指为特定类型提供一个专门的模板实现。特化可以分为全特化（Full Specialization）和偏特化（Partial Specialization）。

##### 全特化

全特化是指为模板的所有参数提供具体的类型，从而生成一个完全确定的模板实例。全特化通常用于为特定类型提供优化的实现。

假设我们有一个通用的模板类 `MyClass`，我们可以为 `int` 类型提供一个全特化的实现：

```c++
// 泛化的模板类
template <typename T>
class MyClass {
public:
    void print() {
        std::cout << "泛化版本" << std::endl;
    }
};

// 为 int 类型提供全特化
template <>
class MyClass<int> {
public:
    void print() {
        std::cout << "int 特化版本" << std::endl;
    }
};
```

这个例子中，`MyClass<int>` 是 `MyClass` 的全特化版本，当模板参数为 `int` 时，将使用这个特化版本。

##### 成员函数的全特化

除了全特化整个类，还可以为类的某个成员函数提供全特化：

```c++
template <typename T>
class MyClass {
public:
    void print() {
        std::cout << "泛化版本" << std::endl;
    }
};

template <>
void MyClass<int>::print() {
    std::cout << "int 特化版本" << std::endl;
}
```

#### 偏特化（Partial Specialization）

偏特化是指为模板的部分参数提供具体的类型，而其他参数仍然保持泛型。偏特化通常用于类模板，因为函数模板不支持偏特化。

假设我们有一个模板类 `MyClass`，它有两个模板参数 `T` 和 `U`，我们可以为 `T` 提供一个具体的类型，而 `U` 保持泛型：

```c++
// 泛化的模板类
template <typename T, typename U>
class MyClass {
public:
    void print() {
        std::cout << "泛化版本" << std::endl;
    }
};

// 为第一个参数为 int 提供偏特化
template <typename U>
class MyClass<int, U> {
public:
    void print() {
        std::cout << "int, U 偏特化版本" << std::endl;
    }
};
```

这个例子中，`MyClass<int, U>` 是 `MyClass` 的偏特化版本，当第一个模板参数为 `int` 时，将使用这个偏特化版本。

#### 特化的优先级

当同时存在泛化版本、全特化版本和偏特化版本时，C++ 编译器会选择最具体的版本。优先级顺序如下：

1. 全特化版本
2. 偏特化版本
3. 泛化版本

一个综合示例，展示了模板的全特化和偏特化：

```c++
#include <iostream>

// 泛化的模板类
template <typename T, typename U>
class MyClass {
public:
    void print() {
        std::cout << "泛化版本" << std::endl;
    }
};

// 为 int 类型提供全特化
template <>
class MyClass<int, int> {
public:
    void print() {
        std::cout << "int, int 全特化版本" << std::endl;
    }
};

// 为第一个参数为 int 提供偏特化
template <typename U>
class MyClass<int, U> {
public:
    void print() {
        std::cout << "int, U 偏特化版本" << std::endl;
    }
};

int main() {
    MyClass<double, double> obj1;
    MyClass<int, double> obj2;
    MyClass<int, int> obj3;

    obj1.print(); // 输出: 泛化版本
    obj2.print(); // 输出: int, U 偏特化版本
    obj3.print(); // 输出: int, int 全特化版本

    return 0;
}
```

### 13、c++和c申请内存方式的区别？

#### 1. 关键字 vs 库函数

##### C 语言

在 C 语言中，内存的申请和释放主要通过标准库函数 `malloc`、`calloc`、`realloc` 和 `free` 来完成。这些函数都是库函数，需要包含相应的头文件（如 `<stdlib.h>` 或 `<malloc.h>`）。

- **`malloc`**：用于分配指定大小的内存块。

  ```c
  void* malloc(size_t size);
  ```

- **`calloc`**：用于分配指定数量和大小的内存块，并初始化为零。

  ```c
  void* calloc(size_t num, size_t size);
  ```

- **`realloc`**：用于调整已分配内存的大小。

  ```c
  void* realloc(void* ptr, size_t size);
  ```

- **`free`**：用于释放已分配的内存。

  ```c
  void free(void* ptr);
  ```

##### C++

在 C++ 中，内存的申请和释放主要通过 `new` 和 `delete` 关键字来完成。这些关键字是 C++ 语言的一部分，不需要包含额外的头文件。

- **`new`**：用于在堆上动态分配内存，并调用对象的构造函数。

  ```c++
  T* ptr = new T;
  T* arr = new T[N];
  ```

- **`delete`**：用于释放通过 `new` 分配的内存，并调用对象的析构函数。

  ```c++
  delete p;
  delete[] arr;
  ```

#### 2. 类型安全性

##### C 语言

在 C 语言中，`malloc` 和 `calloc` 返回的是 `void*` 类型的指针，需要显式地进行类型转换。这种做法容易导致类型不匹配的错误。

```c
int* p = (int*)malloc(10 * sizeof(int));
```

##### C++

在 C++ 中，`new` 返回的是指定类型的指针，不需要显式类型转换，因此更符合类型安全性的要求。

```c++
int* p = new int[10];
```

#### 3. 构造函数和析构函数

##### C 语言

在 C 语言中，`malloc` 和 `calloc` 只负责分配内存，不涉及对象的构造和析构。如果需要初始化对象，需要手动调用初始化函数。

##### C++

在 C++ 中，`new` 不仅分配内存，还会调用对象的构造函数进行初始化。同样，`delete` 会调用对象的析构函数，确保对象的正确清理。

```c++
class MyClass {
public:
    MyClass() { std::cout << "构造函数" << std::endl; }
    ~MyClass() { std::cout << "析构函数" << std::endl; }
};

MyClass* obj = new MyClass();
delete obj;
```

#### 4. 异常处理

##### C 语言

在 C 语言中，如果内存分配失败，`malloc` 和 `calloc` 会返回 `NULL`。需要手动检查返回值以判断内存分配是否成功。

```c
int* p = (int*)malloc(10 * sizeof(int));
if (p == NULL) {
    // 处理内存分配失败
}
```

##### C++

在 C++ 中，如果内存分配失败，`new` 会抛出 `std::bad_alloc` 异常。可以通过捕获异常来处理内存分配失败的情况。

```c++
try {
    int* p = new int[1000000000];
} catch (const std::bad_alloc& e) {
    // 处理内存分配失败
}
```

#### 5. 内存管理的灵活性

##### C 语言

C 语言的内存管理相对简单，但灵活性较低。`malloc` 和 `free` 只能用于基本的内存分配和释放，不支持复杂的内存管理需求。

##### C++

C++ 提供了更多的内存管理机制，如自定义的内存分配器、智能指针等，使得内存管理更加灵活和安全。

### 14、c++释放数组和普通对象的区别？

#### 1. 释放普通对象

##### 分配内存

使用 `new` 分配普通对象的内存时，`new` 会调用对象的构造函数来初始化对象。

```c++
MyClass* obj = new MyClass;
```

##### 释放内存

使用 `delete` 释放普通对象的内存时，`delete` 会调用对象的析构函数来清理资源，然后再释放内存。

```c++
delete obj;
```

#### 2. 释放数组

##### 分配内存

使用 `new[]` 分配数组的内存时，`new[]` 会为数组中的每个元素调用构造函数来初始化对象。

```c++
MyClass* arr = new MyClass[10];
```

##### 释放内存

使用 `delete[]` 释放数组的内存时，`delete[]` 会为数组中的每个元素调用析构函数来清理资源，然后再释放整个数组的内存。

```c++
delete[] arr;
```

#### 主要区别

1. **构造函数和析构函数的调用**
   - 普通对象：`new` 为单个对象调用构造函数，`delete` 为单个对象调用析构函数。
   - 数组：`new[]` 为数组中的每个元素调用构造函数，`delete[]` 为数组中的每个元素调用析构函数。
2. **内存管理**
   - 普通对象：`new` 分配的内存块只需 `delete` 释放。
   - 数组：`new[]` 分配的内存块需要 `delete[]` 释放，以确保所有元素的析构函数都被调用。

#### 为什么不能混用 `new` 和 `delete[]` 或 `new[]` 和 `delete`？

##### 错误示例 1：`new` + `delete[]`

```c++
MyClass* obj = new MyClass;
delete[] obj;  // 错误
```

**问题**：`new` 分配的内存只包含一个对象，而 `delete[]` 期望释放一个数组的内存。这会导致未定义行为，因为 `delete[]` 会尝试调用多个析构函数，而实际上只有一个对象。

##### 错误示例 2：`new[]` + `delete`

```c++
MyClass* arr = new MyClass[10];
delete arr;  // 错误
```

**问题**：`new[]` 分配的内存包含多个对象，而 `delete` 只会调用一个析构函数。这会导致内存泄漏，因为数组中的其他对象没有被正确析构。

### 15、动态多态虚表的位置在哪？

#### 虚函数表的位置

1. **只读数据段（.rdata）**：

   - 虚函数表（vtable）通常位于只读数据段（.rdata）中。这个段在程序的可执行文件中是预先定义好的，由编译器在编译时生成。
   - 虚函数表是类的一部分，而不是特定对象的一部分。所有该类的实例共享同一个虚函数表。

2. **对象的内存布局**：

   - 每个包含虚函数的类的实例在内存中都会有一个隐式的虚函数表指针（vptr）。这个指针通常位于对象的最前面，以便快速访问虚函数表。

   - 对象的内存布局大致如下：

     ```
     +-----------------+
     | vptr            |  -> 指向虚函数表
     +-----------------+
     | 数据成员1        |
     +-----------------+
     | 数据成员2        |
     +-----------------+
     | ...             |
     +-----------------+
     ```

#### 虚函数表的生成和使用

1. **编译时生成**：
   - 虚函数表在编译时由编译器生成。编译器会为每个包含虚函数的类生成一个虚函数表，并将其放置在只读数据段中。
   - 虚函数表中存储的是虚函数的地址，这些地址指向类中定义的虚函数。
2. **运行时使用**：
   - 当对象被创建时，编译器会在对象的内存中初始化虚函数表指针（vptr），使其指向该类的虚函数表。
   - 当通过基类指针或引用调用虚函数时，程序会通过虚函数表指针找到虚函数表，然后从虚函数表中获取虚函数的地址并调用相应的函数。

假设有一个基类 `Base` 和一个派生类 `Derived`，其中 `Base` 包含一个虚函数 `virtual void func()`，`Derived` 重写了这个虚函数。

```c++
class Base {
public:
    virtual void func() { std::cout << "Base::func()" << std::endl; }
};

class Derived : public Base {
public:
    void func() override { std::cout << "Derived::func()" << std::endl; }
};
```

#### 内存布局

- `Base` 对象的内存布局：

  ```
  +-----------------+
  | vptr            |  -> 指向 Base 的虚函数表
  +-----------------+
  ```

- `Derived` 对象的内存布局：

  ```
  +-----------------+
  | vptr            |  -> 指向 Derived 的虚函数表
  +-----------------+
  ```

#### 虚函数表

- `Base` 的虚函数表：

  ```
  +-----------------+
  | &Base::func     |
  +-----------------+
  ```

- `Derived` 的虚函数表：

  ```
  +-----------------+
  | &Derived::func  |
  +-----------------+
  ```

代码示例：

```c++
#include <iostream>

class Base {
public:
    virtual void func() { std::cout << "Base::func()" << std::endl; }
};

class Derived : public Base {
public:
    void func() override { std::cout << "Derived::func()" << std::endl; }
};

int main() {
    Base* basePtr = new Derived();
    basePtr->func();  // 输出 "Derived::func()"
    delete basePtr;
    return 0;
}
```

### 16、有序数组去重不用额外空间？

对于数组的一些处理，通常情况下比较建议使用**双指针**。

#### 思路

1. 初始化两个指针，`slow` 和 `fast`，均从数组的起始位置开始。
2. 遍历数组：
   - 如果 `nums[fast]` 不等于 `nums[slow]`，则将 `nums[fast]` 的值赋给 `nums[++slow]`，即慢指针前进一位，并将快指针指向的值复制到慢指针位置。
   - 如果 `nums[fast]` 等于 `nums[slow]`，则仅移动快指针 `fast`。
3. 遍历结束后，慢指针 `slow` 的位置即为去重后数组的新长度。

#### 示例代码

```c++
int removeDuplicates(int* nums, int numsSize) {
    if (numsSize == 0) return 0;

    int slow = 0;
    for (int fast = 1; fast < numsSize; fast++) {
        if (nums[fast] != nums[slow]) {
            nums[++slow] = nums[fast];
        }
    }
    return slow + 1;
}
```

### 17、二叉树度为0和度为2的数量关系？

**度为0的节点数（n0）总是等于度为2的节点数（n2）加1**，即 `n0 = n2 + 1`。

#### 推导过程

##### 1. 节点和边的关系

首先，我们知道在任何树结构中，节点数总是比边数多1。这是因为树是一个连通的无环图，从根节点到任一其他节点都有一条唯一的路径。因此，如果一个树有`n`个节点，那么它就有`n-1`条边。

##### 2. 度的定义

在二叉树中，节点的度定义为该节点的子节点数量。根据度的不同，可以将节点分为三类：

- 度为0的节点：没有子节点的节点，也称为叶子节点。
- 度为1的节点：只有一个子节点的节点。
- 度为2的节点：有两个子节点的节点。

##### 3. 边数的另一种表达方式

对于度为1的节点，它贡献1条边；对于度为2的节点，它贡献2条边。因此，树中所有边的数量也可以表示为所有度为1的节点数加上两倍的度为2的节点数，即 `n1 + 2 * n2`。

##### 4. 综合上述关系

结合上述两点，我们可以得出以下等式：

- 节点总数 `n = n0 + n1 + n2`
- 边数 `n - 1 = n1 + 2 * n2`

将第一个等式中的 `n` 代入第二个等式，得到：

- `(n0 + n1 + n2) - 1 = n1 + 2 * n2`
- `n0 + n1 + n2 - 1 = n1 + 2 * n2`
- `n0 + n2 - 1 = 2 * n2`
- `n0 = n2 + 1`

### 18、哈夫曼树构建过程？

哈夫曼树（Huffman Tree）是一种带权路径长度最短的二叉树，也称为最优二叉树。它广泛应用于数据压缩领域，尤其是文本数据的压缩，通过构造哈夫曼编码来减少数据的存储空间和传输时间。哈夫曼树的构建过程遵循贪心算法的思想，确保每次合并的都是权值最小的两个节点，从而逐步构建出一棵带权路径长度最小的树。

#### 哈夫曼树的构建步骤

1. **初始化**：
   - 将给定的每个权值（通常是字符出现的频率）作为一个单独的节点，每个节点都是一棵树，形成一个森林。
   - 例如，给定权值集合 `{5, 3, 8, 2, 7}`，则初始森林中有5棵树，每棵树只有一个节点。
2. **选择最小权值的两个节点**：
   - 从森林中选择权值最小的两个节点。如果有多组权值相同的节点，任意选择一组即可。
   - 例如，从 `{5, 3, 8, 2, 7}` 中选择权值最小的两个节点 `2` 和 `3`。
3. **创建新节点**：
   - 创建一个新的节点，其权值为两个最小节点的权值之和。
   - 将这两个最小节点作为新节点的左右子节点。
   - 例如，创建一个新节点，其权值为 `2 + 3 = 5`，并将 `2` 和 `3` 作为其左右子节点。
4. **更新森林**：
   - 从森林中移除这两个最小节点。
   - 将新创建的节点加入森林。
   - 例如，森林更新为 `{5, 8, 7, 5}`。
5. **重复上述步骤**：
   - 重复上述步骤，直到森林中只剩下一棵树，这棵树即为哈夫曼树。
   - 例如，继续选择最小的两个节点 `5` 和 `5`，创建新节点 `10`，森林更新为 `{8, 7, 10}`。
   - 再选择最小的两个节点 `7` 和 `8`，创建新节点 `15`，森林更新为 `{10, 15}`。
   - 最后选择最小的两个节点 `10` 和 `15`，创建新节点 `25`，森林中只剩下一棵树 `{25}`，即为哈夫曼树。

#### 详细示例

假设给定权值集合 `{5, 3, 8, 2, 7}`，我们按照上述步骤构建哈夫曼树：

1. 初始化：森林：`{5, 3, 8, 2, 7}`
2. 选择最小权值的两个节点：选择 `2` 和 `3`。
3. 创建新节点：
   - 新节点权值：`2 + 3 = 5`
   - 森林更新：`{5, 5, 8, 7}`
4. 选择最小权值的两个节点：选择 `5` 和 `5`。
5. 创建新节点：
   - 新节点权值：`5 + 5 = 10`
   - 森林更新：`{8, 7, 10}`
6. 选择最小权值的两个节点：选择 `7` 和 `8`。
7. 创建新节点：
   - 新节点权值：`7 + 8 = 15`
   - 森林更新：`{10, 15}`
8. 选择最小权值的两个节点：选择 `10` 和 `15`。
9. 创建新节点：
   - 新节点权值：`10 + 15 = 25`
   - 森林更新：`{25}`

最终，森林中只剩下一棵树，即为哈夫曼树。

#### 哈夫曼编码

哈夫曼编码是基于哈夫曼树生成的一种前缀编码，用于数据压缩。具体步骤如下：

1. **从根节点到叶子节点的路径**：
   - 从根节点出发，到达每个叶子节点的路径上的左分支用 `0` 表示，右分支用 `1` 表示。
   - 例如，假设 `2` 对应的叶子节点路径为 `00`，`3` 对应的叶子节点路径为 `01`，以此类推。
2. **生成编码**：
   - 每个叶子节点的路径上的 `0` 和 `1` 序列即为该叶子节点的哈夫曼编码。
   - 例如，`2` 的编码为 `00`，`3` 的编码为 `01`，`5` 的编码为 `10`，`7` 的编码为 `110`，`8` 的编码为 `111`。

#### 代码实现

以下是用 C++ 实现哈夫曼树构建的示例代码：

```c++
#include <iostream>
#include <vector>
#include <queue>

struct TreeNode {
    int weight;
    char value;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int w, char v) : weight(w), value(v), left(nullptr), right(nullptr) {}
};

// 用于比较节点权值的比较器
struct Compare {
    bool operator()(TreeNode* l, TreeNode* r) {
        return l->weight > r->weight;
    }
};

TreeNode* buildHuffmanTree(const std::vector<std::pair<char, int>>& frequencies) {
    std::priority_queue<TreeNode*, std::vector<TreeNode*>, Compare> pq;

    // 将每个节点加入优先队列
    for (const auto& freq : frequencies) {
        pq.push(new TreeNode(freq.second, freq.first));
    }

    // 构建哈夫曼树
    while (pq.size() > 1) {
        TreeNode* left = pq.top();
        pq.pop();
        TreeNode* right = pq.top();
        pq.pop();

        TreeNode* newNode = new TreeNode(left->weight + right->weight, '\0');
        newNode->left = left;
        newNode->right = right;

        pq.push(newNode);
    }

    return pq.top();
}

void printHuffmanCodes(TreeNode* root, const std::string& code, std::unordered_map<char, std::string>& huffmanCodes) {
    if (!root) return;

    if (!root->left && !root->right) {
        huffmanCodes[root->value] = code;
    }

    printHuffmanCodes(root->left, code + "0", huffmanCodes);
    printHuffmanCodes(root->right, code + "1", huffmanCodes);
}

int main() {
    std::vector<std::pair<char, int>> frequencies = {{'A', 5}, {'B', 3}, {'C', 8}, {'D', 2}, {'E', 7}};
    TreeNode* huffmanTree = buildHuffmanTree(frequencies);

    std::unordered_map<char, std::string> huffmanCodes;
    printHuffmanCodes(huffmanTree, "", huffmanCodes);

    for (const auto& code : huffmanCodes) {
        std::cout << "Character: " << code.first << ", Code: " << code.second << std::endl;
    }

    // 释放内存
    // 这里省略了释放内存的代码，实际应用中需要递归释放所有节点的内存

    return 0;
}
```

### 19、快排最坏情况发生？

其核心思想是通过“分治”策略将一个数组分成较小和较大的两个子数组，然后递归地对这两个子数组进行排序。尽管快速排序在大多数情况下表现优异，但其最坏情况下的时间复杂度为 O(n2)*O*(*n*2)，这主要取决于选择的基准值（pivot）是否能有效地将数组分割成两个大小相近的子数组。

#### 快速排序最坏情况的发生

快速排序的最坏情况通常发生在以下几种情形：

1. **数组已经是正序排列**：

   当数组已经是正序排列时，如果每次选择的基准值都是当前子数组的第一个或最后一个元素，那么每次分割后，一个子数组将为空，另一个子数组将包含除了基准值之外的所有元素。这种情况下，快速排序的递归深度将达到n，每次分割需要n−1次比较，因此总的时间复杂度为O(n^2)。

2. **数组已经是逆序排列**：

   类似于正序排列，当数组是逆序排列时，如果每次选择的基准值都是当前子数组的第一个或最后一个元素，也会导致每次分割后一个子数组为空，另一个子数组包含除了基准值之外的所有元素。同样，这种情况下的时间复杂度也是O(n^2)。

3. **所有元素都相同**：

   当数组中的所有元素都相同时，如果每次选择的基准值都是当前子数组的第一个或最后一个元素，那么每次分割后，一个子数组将包含所有相同的元素，另一个子数组为空。这种情况下，快速排序的递归深度也将达到n，每次分割需要n−1次比较，因此总的时间复杂度为O(n^2)。

#### 避免最坏情况的策略

为了避免快速排序陷入最坏情况，可以采取以下几种优化策略：

1. **随机选择基准值**：

   通过随机选择基准值，可以大大减少最坏情况发生的概率。随机选择基准值使得每次分割更有可能将数组均匀地分成两个子数组，从而提高算法的平均性能。

2. **三数取中法**：

   三数取中法是指从当前子数组的首、尾和中间位置选择三个元素，然后选择这三个元素的中位数作为基准值。这种方法可以有效避免选择极端值作为基准值，从而减少最坏情况的发生。

3. **小数组优化**：

   对于小数组，可以使用其他排序算法（如插入排序）来替代快速排序。这是因为插入排序在小数组上的性能通常优于快速排序。当子数组的大小小于某个阈值时，切换到插入排序可以显著提高整体性能。

#### 示例

假设我们有一个数组 `[1, 2, 3, 4, 5]`，这是一个正序排列的数组。如果我们使用第一个元素作为基准值，那么快速排序的过程如下：

1. 初始数组：`[1, 2, 3, 4, 5]`
2. 选择基准值 `1`，分割后得到：`[1]` 和 `[2, 3, 4, 5]`
3. 选择基准值 `2`，分割后得到：`[2]` 和 `[3, 4, 5]`
4. 选择基准值 `3`，分割后得到：`[3]` 和 `[4, 5]`
5. 选择基准值 `4`，分割后得到：`[4]` 和 `[5]`
6. 选择基准值 `5`，分割后得到：`[5]` 和 `[]`

在这个过程中，每次分割后，一个子数组为空，另一个子数组包含除了基准值之外的所有元素，因此递归深度为 n，每次分割需要 n−1次比较，总的时间复杂度为 O(n^2)。

### 20、递归算法对比循环的问题？

#### 1. 代码可读性

**递归**：

- 优点：递归算法通常代码更加简洁、清晰，能够直观地反映出问题的本质。对于一些具有天然递归结构的问题（如树的遍历、图的搜索、分治算法等），递归的实现方式更加自然，易于理解。
- 缺点：对于初学者来说，递归的逻辑可能较为抽象，理解起来有一定难度，特别是在递归层次较深时，调试和追踪错误也比较困难。

**循环**：

- 优点：循环算法通常代码结构更加线性，执行流程更加显式，易于理解和调试。对于一些简单的重复操作，循环的实现方式更加直观。
- 缺点：对于一些复杂的递归问题，使用循环实现可能会使代码变得冗长且难以理解，尤其是当需要手动管理状态时。

#### 2. 运行效率

**递归**：

- 优点：在某些情况下，递归算法可以通过尾递归优化（如果编译器支持）来提高性能，使其在性能上接近循环算法。
- 缺点：递归算法通常比循环算法慢，因为每次递归调用都会产生额外的函数调用开销，包括保存和恢复上下文、参数传递等。递归深度较大时，这些开销会累积，导致性能下降。

**循环**：

- 优点：循环算法通常具有更好的性能，因为它们没有函数调用的开销，可以在同一个栈帧内完成所有操作，执行效率较高。
- 缺点：对于某些问题，循环算法可能需要更多的代码来实现，且逻辑可能更为复杂，尤其是在需要模拟递归行为时。

#### 3. 内存使用

**递归**：

- 优点：对于一些问题，递归算法可以通过尾递归优化来减少内存使用，避免深度递归导致的栈溢出。
- 缺点：递归算法通常需要更多的内存，因为每次递归调用都会在栈上分配新的栈帧。递归深度较大时可能会导致栈溢出，尤其是在没有尾递归优化的情况下。

**循环**：

- 优点：循环算法通常占用较少的内存，因为它们在一个固定的栈帧内操作，不需要为每次迭代分配新的栈帧。这对于需要处理大量数据或深度较大的问题尤为重要。
- 缺点：对于某些问题，循环算法可能需要额外的数据结构来模拟递归的行为，从而增加内存使用。

#### 4. 适用场景

**递归**：

- 数据结构具有递归性质：如树、图等数据结构的遍历和操作。
- 问题具有递归结构：如分治算法、动态规划等。
- 代码简洁优先：当问题的递归结构明显，且代码简洁性更重要时。
- 深度适中：递归深度不太深，不会导致栈溢出的情况。

**循环**：

- 简单重复操作：如数组遍历、累加求和等。
- 性能要求高：当性能是首要考虑因素时，循环通常比递归更高效。
- 内存限制严格：当可用内存有限，需要尽量减少内存使用时。
- 递归结构不明显：当问题的递归结构不明显，使用循环更直观时。

### 21、优先队列的实现？

优先队列（Priority Queue）是一种特殊的队列，其中的元素具有优先级，当访问元素时，优先级最高的元素最先被删除。优先队列通常用于实现调度、资源分配、事件驱动系统等应用场景。在C++中，优先队列通常使用堆（Heap）来实现，堆是一种完全二叉树，可以高效地维护元素的优先级关系。

#### 基本概念

1. 优先级：每个元素都有一个优先级，优先级高的元素会被优先处理。
2. 最大堆和最小堆：
   - 最大堆：堆顶元素是堆中最大的元素。
   - 最小堆：堆顶元素是堆中最小的元素。
3. 完全二叉树：堆是一种完全二叉树，可以用数组来表示，这样可以节省空间并提高操作效率。

#### 优先队列的实现

##### 1. 使用C++ STL中的`priority_queue`

C++ STL 提供了一个现成的优先队列实现 `std::priority_queue`，它默认是一个最大堆。可以通过自定义比较器来实现最小堆或其他特定的优先级规则。

###### 基本用法

```c++
#include <iostream>
#include <queue>
#include <vector>

int main() {
    // 默认最大堆
    std::priority_queue<int> maxHeap;
    
    // 插入元素
    maxHeap.push(10);
    maxHeap.push(30);
    maxHeap.push(20);
    
    // 输出堆顶元素
    std::cout << "Max element: " << maxHeap.top() << std::endl;
    
    // 删除堆顶元素
    maxHeap.pop();
    
    // 输出新的堆顶元素
    std::cout << "New max element: " << maxHeap.top() << std::endl;
    
    // 判断堆是否为空
    if (!maxHeap.empty()) {
        std::cout << "Heap is not empty" << std::endl;
    }
    
    // 获取堆的大小
    std::cout << "Heap size: " << maxHeap.size() << std::endl;
    
    return 0;
}
```

###### 最小堆

```c++
#include <iostream>
#include <queue>
#include <vector>
#include <functional>

int main() {
    // 最小堆
    std::priority_queue<int, std::vector<int>, std::greater<int>> minHeap;
    
    // 插入元素
    minHeap.push(10);
    minHeap.push(30);
    minHeap.push(20);
    
    // 输出堆顶元素
    std::cout << "Min element: " << minHeap.top() << std::endl;
    
    // 删除堆顶元素
    minHeap.pop();
    
    // 输出新的堆顶元素
    std::cout << "New min element: " << minHeap.top() << std::endl;
    
    return 0;
}
```

##### 2. 自定义优先队列

如果需要更复杂的优先级规则，可以自定义比较器。

###### 自定义比较器

```c++
#include <iostream>
#include <queue>
#include <vector>

struct Compare {
    bool operator()(const std::pair<int, std::string>& a, const std::pair<int, std::string>& b) {
        return a.first > b.first; // 小顶堆
    }
};

int main() {
    // 自定义优先队列
    std::priority_queue<std::pair<int, std::string>, std::vector<std::pair<int, std::string>>, Compare> customPQ;
    
    // 插入元素
    customPQ.push({10, "Alice"});
    customPQ.push({30, "Bob"});
    customPQ.push({20, "Charlie"});
    
    // 输出堆顶元素
    std::cout << "Top element: " << customPQ.top().second << " with priority: " << customPQ.top().first << std::endl;
    
    // 删除堆顶元素
    customPQ.pop();
    
    // 输出新的堆顶元素
    std::cout << "New top element: " << customPQ.top().second << " with priority: " << customPQ.top().first << std::endl;
    
    return 0;
}
```

#### 堆的实现细节

##### 1. 堆的插入操作

插入操作包括将新元素添加到堆的末尾，然后通过向上调整（Percolate Up）确保堆的性质。

```c++
void insert(std::vector<int>& heap, int value) {
    heap.push_back(value);
    int index = heap.size() - 1;
    while (index > 0 && heap[(index - 1) / 2] < heap[index]) {
        std::swap(heap[(index - 1) / 2], heap[index]);
        index = (index - 1) / 2;
    }
}
```

##### 2. 堆的删除操作

删除操作包括移除堆顶元素，将堆的最后一个元素移到堆顶，然后通过向下调整（Percolate Down）确保堆的性质。

```c++
void remove(std::vector<int>& heap) {
    if (heap.empty()) return;
    heap[0] = heap.back();
    heap.pop_back();
    int index = 0;
    while (true) {
        int leftChild = 2 * index + 1;
        int rightChild = 2 * index + 2;
        int largest = index;
        
        if (leftChild < heap.size() && heap[leftChild] > heap[largest]) {
            largest = leftChild;
        }
        
        if (rightChild < heap.size() && heap[rightChild] > heap[largest]) {
            largest = rightChild;
        }
        
        if (largest != index) {
            std::swap(heap[index], heap[largest]);
            index = largest;
        } else {
            break;
        }
    }
}
```

### 22、有一个超大文件，无法一次性加载到内存，如何排序？

#### 概述

外部排序是一种处理大量数据排序的方法，当数据量超过内存容量时，无法一次性加载到内存中。多路归并排序是外部排序中常用的方法，其基本思想是将大文件分割成多个小文件，对每个小文件进行内部排序，然后将这些已排序的小文件合并成一个有序的大文件。

#### 具体步骤

1. **文件分割**：
   - 将大文件分割成多个小文件，每个小文件的大小应适合内存容量，以便可以一次性加载到内存中进行排序。
   - 分割时可以使用文件读写操作，每次读取一定大小的数据块，将其写入一个小文件中。
2. **内部排序**：
   - 对每个小文件进行内部排序。可以使用任何高效的内部排序算法，如快速排序（Quick Sort）、归并排序（Merge Sort）等。
   - 排序后的每个小文件都是有序的，可以将其写回到磁盘上。
3. **多路归并**：
   - 将所有已排序的小文件合并成一个有序的大文件。这是通过多路归并算法实现的。
   - 多路归并的基本思想是从每个已排序的小文件中读取一部分数据（例如，每个文件读取1MB的数据），将这些数据放入一个优先队列（最小堆）中。
   - 从优先队列中取出最小的元素，将其写入最终的输出文件中，然后从对应的文件中读取下一个元素放入优先队列中。
   - 重复上述过程，直到所有小文件中的数据都被处理完毕。

#### 实现细节

##### 1. 文件分割

```c++
#include <iostream>
#include <fstream>
#include <vector>
#include <string>

void split_file(const std::string& input_file, const std::string& output_dir, size_t chunk_size) {
    std::ifstream infile(input_file);
    if (!infile) {
        std::cerr << "无法打开输入文件: " << input_file << std::endl;
        return;
    }

    size_t chunk_num = 0;
    std::string line;
    std::vector<std::string> chunk;

    while (std::getline(infile, line)) {
        chunk.push_back(line);
        if (chunk.size() * line.size() >= chunk_size) {
            std::ofstream outfile(output_dir + "/chunk_" + std::to_string(chunk_num) + ".txt");
            for (const auto& l : chunk) {
                outfile << l << "\n";
            }
            chunk.clear();
            ++chunk_num;
        }
    }

    if (!chunk.empty()) {
        std::ofstream outfile(output_dir + "/chunk_" + std::to_string(chunk_num) + ".txt");
        for (const auto& l : chunk) {
            outfile << l << "\n";
        }
    }
}
```

##### 2. 内部排序

```c++
#include <algorithm>

void sort_chunk(const std::string& input_file, const std::string& output_file) {
    std::ifstream infile(input_file);
    if (!infile) {
        std::cerr << "无法打开输入文件: " << input_file << std::endl;
        return;
    }

    std::vector<long long> numbers;
    std::string line;
    while (std::getline(infile, line)) {
        numbers.push_back(std::stoll(line));
    }

    std::sort(numbers.begin(), numbers.end());

    std::ofstream outfile(output_file);
    for (const auto& num : numbers) {
        outfile << num << "\n";
    }
}

void sort_chunks(const std::string& output_dir, const std::vector<std::string>& chunk_files) {
    for (const auto& file : chunk_files) {
        sort_chunk(file, file);
    }
}
```

##### 3. 多路归并

```c++
#include <queue>
#include <memory>
#include <functional>

struct FileIterator {
    std::ifstream* file;
    std::string current_line;
    bool has_next;

    FileIterator(std::ifstream* file) : file(file), has_next(true) {
        read_next();
    }

    void read_next() {
        if (std::getline(*file, current_line)) {
            current_line = std::to_string(std::stoll(current_line));
        } else {
            has_next = false;
        }
    }
};

struct Compare {
    bool operator()(const std::shared_ptr<FileIterator>& a, const std::shared_ptr<FileIterator>& b) {
        return a->current_line > b->current_line;
    }
};

void merge_sorted_files(const std::vector<std::string>& chunk_files, const std::string& output_file) {
    std::priority_queue<std::shared_ptr<FileIterator>, std::vector<std::shared_ptr<FileIterator>>, Compare> min_heap;

    std::vector<std::unique_ptr<std::ifstream>> files;
    for (const auto& file : chunk_files) {
        files.emplace_back(new std::ifstream(file));
        if (files.back()->is_open()) {
            min_heap.push(std::make_shared<FileIterator>(files.back().get()));
        }
    }

    std::ofstream outfile(output_file);
    while (!min_heap.empty()) {
        auto top = min_heap.top();
        min_heap.pop();

        outfile << top->current_line << "\n";

        top->read_next();
        if (top->has_next) {
            min_heap.push(top);
        }
    }
}
```

##### 主函数

```c++
int main() {
    const std::string input_file = "large_file.txt";
    const std::string output_dir = "chunks";
    const std::string output_file = "sorted_file.txt";
    const size_t chunk_size = 1024 * 1024 * 100; // 100MB

    // 创建输出目录
    std::filesystem::create_directory(output_dir);

    // 文件分割
    split_file(input_file, output_dir, chunk_size);

    // 获取所有分割文件
    std::vector<std::string> chunk_files;
    for (const auto& entry : std::filesystem::directory_iterator(output_dir)) {
        if (entry.is_regular_file()) {
            chunk_files.push_back(entry.path().string());
        }
    }

    // 内部排序
    sort_chunks(output_dir, chunk_files);

    // 多路归并
    merge_sorted_files(chunk_files, output_file);

    std::cout << "排序完成，结果保存在: " << output_file << std::endl;

    return 0;
}
```

### 23、B+树对比普通树，红黑树的区别，为什么不用B树？

#### 1. 普通树

普通树是一种基本的树形数据结构，每个节点可以有多个子节点，没有特定的平衡要求。普通树的主要特点是：

- 结构简单：每个节点可以有任意数量的子节点。
- 无平衡要求：可能导致树的不平衡，影响查找、插入和删除操作的效率。
- 应用场景：适用于简单的层次结构数据，如文件系统目录结构。

#### 2. 红黑树

红黑树是一种自平衡二叉查找树，通过在每个节点上增加一个存储位来表示节点的颜色（红色或黑色），并通过一系列的约束条件来保持树的平衡。红黑树的主要特点是：

- 自平衡：通过旋转和重新着色操作，确保树的高度始终保持在对数级别。
- 高效操作：查找、插入和删除操作的时间复杂度均为 O(log⁡n)*O*(log*n*)。
- 应用场景：广泛应用于需要频繁动态更新的场景，如符号表、集合和映射等数据结构。

#### 3. B+树

B+树是一种多路平衡查找树，广泛应用于数据库和文件系统中。B+树的主要特点是：

- 多路查找：每个节点可以有多个子节点，减少了树的高度，提高了查找效率。
- 非叶子节点只存储索引：非叶子节点不存储实际数据，只存储索引信息，使得每个节点可以存储更多的索引项。
- 叶子节点链表：所有叶子节点通过指针链接在一起，便于范围查询和顺序遍历。
- 高效磁盘访问：由于每个节点可以存储大量索引项，减少了磁盘I/O次数，适合磁盘存储系统。

#### 为什么不用B树？

虽然B树也是一种多路平衡查找树，但B+树在某些方面表现更好，特别是在数据库和文件系统中。以下是B+树相对于B树的优势：

1. **更好的磁盘读写性能**：

   - 索引集中：B+树的非叶子节点只存储索引信息，不存储实际数据，这意味着每个节点可以存储更多的索引项。相比之下，B树的每个节点既存储索引也存储数据，导致每个节点存储的索引项较少。
   - 减少I/O次数：由于B+树的每个节点可以存储更多的索引项，减少了磁盘I/O次数，提高了磁盘访问效率。

2. **更稳定的查询效率**：

   - 固定高度：B+树的所有叶子节点都在同一层，确保了查询操作的时间复杂度始终为 O(log⁡n)*O*(log*n*)。相比之下，B树的查询路径可能较长，尤其是在树不平衡的情况下。
   - 范围查询友好：B+树的叶子节点通过指针链接在一起，使得范围查询和顺序遍历更加高效。

3. **更好的空间利用率**：

   紧凑存储：B+树的非叶子节点只存储索引信息，使得节点的存储更加紧凑，提高了空间利用率。

#### 应用场景

- 普通树：适用于简单的层次结构数据，如文件系统目录结构。
- 红黑树：适用于需要频繁动态更新的场景，如符号表、集合和映射等数据结构。
- B+树：广泛应用于数据库和文件系统中，特别是需要高效磁盘访问和范围查询的场景。

### 24、HTTP 1/2/3版本的区别？

#### 1. HTTP 1.0

HTTP 1.0 是最早的 HTTP 协议版本之一，它在1996年被标准化。HTTP 1.0 的主要特点包括：

- 无状态和无连接：每次请求都需要建立一个新的 TCP 连接，请求完成后立即断开连接。这种无连接的特性导致网络利用率较低，每次请求都需要经历 TCP 三次握手和四次挥手的过程，增加了延迟。
- 队头阻塞：HTTP 1.0 规定下一个请求必须在前一个请求响应到达之前才能发送，导致队头阻塞问题。
- 缓存机制：HTTP 1.0 引入了基本的缓存机制，使用`Expires`和`Cache-Control`头部来控制缓存行为。
- 请求方法：支持`GET`、`POST`和`HEAD`等基本请求方法。

#### 2. HTTP 1.1

HTTP 1.1 在1997年发布，对 HTTP 1.0 进行了多项改进，主要特点包括：

- 默认持久连接：HTTP 1.1 默认使用持久连接（Persistent Connections），允许在一个 TCP 连接上发送多个请求和响应，减少了连接建立和关闭的开销。可以通过`Connection: keep-alive`头部显式启用持久连接。
- 管道化：允许在一个持久连接上同时发送多个请求，但服务器必须按照请求的顺序返回响应，这可能导致队头阻塞问题。
- 响应分块：引入了范围请求（Range Requests）机制，支持断点续传和部分数据请求。
- 头部压缩：虽然 HTTP 1.1 没有引入头部压缩，但通过`Content-Encoding`和`Transfer-Encoding`头部支持数据压缩。
- 内容协商：允许客户端请求特定格式和语言的内容。

#### 3. HTTP 2.0

HTTP 2.0 在2015年发布，主要基于 Google 的 SPDY 协议，重点在于提高性能和减少延迟。主要特点包括：

- 二进制分帧：HTTP 2.0 采用二进制分帧层，将所有传输的信息分割为更小的消息和帧，使用二进制格式编码，提高了解析效率。
- 多路复用：允许多个请求和响应在同一连接上同时传输，避免了队头阻塞问题。
- 头部压缩：使用 HPACK 算法压缩请求头，减少了数据传输量。
- 服务器推送：服务器可以主动推送资源到客户端的缓存中，减少客户端请求的延迟。
- 优先级：允许客户端为请求设置优先级，确保重要资源优先加载。

#### 4. HTTP 3.0

HTTP 3.0 在2022年发布，主要基于 QUIC 协议，进一步提高了网络传输性能。主要特点包括：

- 基于 UDP 的 QUIC 协议：HTTP 3.0 使用基于 UDP 的 QUIC 协议，而不是传统的 TCP 协议。QUIC 具有更快的连接建立时间和更好的拥塞控制，支持快速的连接迁移和零 RTT 握手。
- 避免队头阻塞：QUIC 协议通过多路复用和流控制机制，避免了 TCP 协议中的队头阻塞问题。
- 改进的连接迁移：QUIC 协议支持连接迁移，即使 IP 地址发生变化，连接也不会中断，特别适合移动网络环境。
- 更高的安全性：QUIC 协议内置了 TLS 1.3 加密，确保了数据传输的安全性。

### 25、HTTP Cookie作用？

HTTP Cookie 是服务器发送到用户浏览器并保存在本地的一小块数据，它会在浏览器下次向同一服务器再次发起请求时被携带并发送到服务器上。Cookie 的主要作用包括：

1. **会话状态管理**

   - 保持用户登录状态：当用户登录网站后，服务器可以设置一个包含用户身份信息的 Cookie，浏览器在后续请求中会自动携带这个 Cookie，服务器通过验证 Cookie 中的信息来识别用户，从而保持用户的登录状态。

   - 购物车：在电子商务网站中，Cookie 可以用来记录用户添加到购物车的商品信息，即使用户关闭浏览器后再次访问网站，购物车中的商品信息仍然保留。

2. **个性化设置**

   - 用户偏好：网站可以根据用户在 Cookie 中存储的偏好设置（如语言、主题、字体大小等）来定制页面内容，提供个性化的用户体验。

   - 用户行为记录：网站可以使用 Cookie 记录用户的浏览历史、搜索记录等信息，以便提供相关推荐或优化用户体验。

     14

3. **浏览器行为跟踪**

   - 用户行为分析：通过在 Cookie 中存储用户的行为数据，网站可以进行用户行为分析，了解用户的访问模式、兴趣偏好等，从而优化网站内容和营销策略。

   - 广告跟踪：广告网络可以使用 Cookie 来跟踪用户在不同网站上的行为，以便展示相关的广告内容。

4. **安全性**

   - HttpOnly 标志：通过设置`HttpOnly`标志，可以防止 JavaScript 访问 Cookie，从而减少跨站脚本攻击（XSS）的风险。

   - Secure 标志：通过设置`Secure`标志，可以确保 Cookie 只在 HTTPS 连接中传输，提高数据传输的安全性。

   - SameSite 属性：通过设置`SameSite`属性，可以防止跨站请求伪造攻击（CSRF），确保 Cookie 只在同站请求中发送。

5. **跨域资源共享**

​	Domain 和 Path 属性：通过设置`Domain`和`Path`属性，可以控制 Cookie 的作用域，确保 Cookie 只在特定的子域名和路径下发送，从而实现跨域资源共享。

6. **会话劫持防护**

​	Cookie 前缀：通过使用 `__Host-` 和 `__Secure-` 前缀，可以进一步增强 Cookie 的安全性，防止子域上的恶意应用访问敏感 Cookie。

### 26、TCP拥塞控制方法？

#### 1. 慢启动（Slow Start）

- 初始窗口大小：在连接建立初期，发送方的拥塞窗口（Congestion Window, cwnd）通常被初始化为 1 个最大报文段大小（MSS）。
- 指数增长：每当发送方收到一个确认（ACK），拥塞窗口 cwnd 就增加 1 个 MSS。这意味着在每个往返时间（RTT）内，cwnd 的大小会翻倍。
- 慢启动阈值：当 cwnd 达到慢启动阈值（Slow Start Threshold, ssthresh）时，TCP 会从慢启动模式切换到拥塞避免模式。

#### 2. 拥塞避免（Congestion Avoidance）

- 线性增长：在拥塞避免模式下，每当发送方收到一个 ACK，cwnd 增加 1/cwnd 个 MSS。这意味着在每个 RTT 内，cwnd 的增长速度较慢，呈线性增长。
- 减半阈值：如果发生丢包（通常是通过超时或接收到三个冗余 ACK 来检测），cwnd 会被减半，并且 ssthresh 也被设置为减半后的 cwnd 值。

#### 3. 快重传（Fast Retransmit）

- 冗余 ACK：当接收方收到失序的报文段时，会立即发送一个冗余 ACK，告知发送方哪些报文段已经收到，哪些还没有收到。
- 重传条件：如果发送方在没有超时的情况下连续收到三个冗余 ACK，就会立即重传丢失的报文段，而不是等待超时。

#### 4. 快恢复（Fast Recovery）

- 进入条件：当发送方连续收到三个冗余 ACK 时，会进入快恢复模式。
- 调整 cwnd：在快恢复模式下，cwnd 被设置为 ssthresh 的值，然后每收到一个冗余 ACK，cwnd 增加 1 个 MSS。
- 退出条件：当发送方收到一个新的 ACK（即非冗余 ACK）时，退出快恢复模式，重新进入拥塞避免模式。

#### 详细过程

![拥塞控制流程](https://cdn.jsdelivr.net/gh/aqjsp/Pictures/image-20241022231225581.png)

1. **慢启动**：
   - 初始时，cwnd = 1 MSS。
   - 每收到一个 ACK，cwnd += 1 MSS。
   - 当 cwnd >= ssthresh 时，切换到拥塞避免模式。
2. **拥塞避免**：
   - 每收到一个 ACK，cwnd += 1/cwnd MSS。
   - 如果发生丢包（超时或三个冗余 ACK），cwnd = cwnd / 2，ssthresh = cwnd，切换到慢启动或快恢复模式。
3. **快重传**：
   - 如果连续收到三个冗余 ACK，立即重传丢失的报文段。
4. **快恢复**：
   - 当进入快恢复模式时，cwnd = ssthresh。
   - 每收到一个冗余 ACK，cwnd += 1 MSS。
   - 收到新的 ACK 时，退出快恢复模式，进入拥塞避免模式。

#### 示例

假设初始时 cwnd = 1 MSS，ssthresh = 32 MSS：

1. **慢启动**：
   - 第一个 RTT：cwnd = 1 + 1 = 2 MSS
   - 第二个 RTT：cwnd = 2 + 2 = 4 MSS
   - 第三个 RTT：cwnd = 4 + 4 = 8 MSS
   - 第四个 RTT：cwnd = 8 + 8 = 16 MSS
   - 第五个 RTT：cwnd = 16 + 16 = 32 MSS
2. **拥塞避免**：
   - 第六个 RTT：cwnd = 32 + 1 = 33 MSS
   - 第七个 RTT：cwnd = 33 + 1 = 34 MSS
   - 第八个 RTT：cwnd = 34 + 1 = 35 MSS
3. **发生丢包**：
   - cwnd = 35 / 2 = 17.5 ≈ 18 MSS
   - ssthresh = 18 MSS
4. **快重传**：
   - 连续收到三个冗余 ACK，立即重传丢失的报文段。
5. **快恢复**：
   - cwnd = 18 MSS
   - 每收到一个冗余 ACK，cwnd += 1 MSS
   - 收到新的 ACK 时，cwnd = 18 MSS，进入拥塞避免模式。