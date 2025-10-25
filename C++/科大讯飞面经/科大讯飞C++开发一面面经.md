# 科大讯飞C++开发一面面经

> 来源：https://www.nowcoder.com/share/jump/1727183347618

### 1、内存泄漏和内存溢出的概念？

#### 内存泄漏

定义：内存泄漏是指程序在动态分配内存后，失去了对该内存的引用，但该内存并没有被释放。也就是说，程序无法再访问这块内存区域，导致这部分内存被“遗忘”而无法被回收。

原因：

- 忘记释放动态分配的内存（如使用 `new` 分配内存后没有使用 `delete`）。
- 在分配内存后，改变了指向该内存的指针，使得原指针失效。
- 循环引用（在使用智能指针时，两个对象相互持有对方，导致引用计数永远不为零）。

影响：

- 随着时间推移，内存泄漏会导致程序占用越来越多的内存，最终可能导致系统性能下降甚至崩溃。
- 对于长时间运行的程序（如服务器），内存泄漏问题尤为严重。

#### 内存溢出

定义：内存溢出（或称为堆栈溢出）是指程序在运行过程中，试图使用超出其可用内存范围的内存。这通常发生在栈空间耗尽或堆空间耗尽的情况下。

原因：

- 递归调用太深，导致栈空间耗尽。
- 动态分配过多内存，超过了系统或程序的堆内存限制。
- 大数组或数据结构的分配超出了可用内存。

影响：

- 内存溢出通常会导致程序异常终止，并抛出特定的异常（如 C++ 中的 `std::bad_alloc`）。
- 程序在运行过程中可能会出现不可预测的行为，甚至导致系统不稳定。

#### 预防措施

- 内存泄漏：使用智能指针（如 `std::unique_ptr` 和 `std::shared_ptr`），确保动态分配的内存能自动释放；定期检查和测试代码，使用工具（如 Valgrind）检测内存泄漏。
- 内存溢出：优化算法，避免过深的递归；合理分配和释放内存，监控程序内存使用情况；在处理大数据时，使用合适的数据结构和算法。

### 2、new的底层原理？

#### 1. 内存管理

在 C++ 中，内存主要分为两种区域：

- 栈（Stack）：用于存储局部变量和函数调用时的参数，内存由编译器自动管理。
- 堆（Heap）：用于动态分配的内存，程序可以根据需要分配和释放。内存的管理由程序员控制，使用 `new` 和 `delete`。

#### 2. `new` 的工作流程

当使用 `new` 分配内存时，实际的过程包括以下几个步骤：

1. 请求内存：
   - `new` 运算符首先会请求一定大小的内存块。通常使用操作系统的内存管理接口（如 `malloc` 或 `sbrk`）向堆申请所需的内存。
   - 在 C++ 中，`new` 可以接受一个类型参数，确定所需内存的大小。
2. 内存对齐：为了提高内存访问的效率，操作系统通常要求内存地址是特定对齐的（如 4 字节、8 字节）。`new` 会确保分配的内存块遵循这些对齐要求。
3. 调用构造函数：
   - 在分配完内存后，`new` 会调用对象的构造函数，初始化对象的状态。
   - 这一步骤确保对象在使用之前处于有效状态。
4. 返回指针：`new` 返回指向已分配内存的指针。这个指针可以用来访问对象。

#### 3. 例子

```c++
class MyClass {
public:
    MyClass() { /* 构造函数 */ }
    ~MyClass() { /* 析构函数 */ }
};

MyClass* obj = new MyClass(); // 1. 请求内存
                             // 2. 对齐内存
                             // 3. 调用 MyClass 的构造函数
```

#### 4. 内存释放

与 `new` 对应的是 `delete`，用于释放动态分配的内存。`delete` 的工作流程如下：

1. 调用析构函数：在释放内存之前，`delete` 会调用对象的析构函数，执行必要的清理操作（如释放资源、关闭文件等）。
2. 释放内存：`delete` 会将内存返回给操作系统，通常通过 `free` 或其他内存管理函数。

```c
delete obj; // 1. 调用 MyClass 的析构函数
            // 2. 释放内存
```

#### 5. 重载 `new` 和 `delete`

C++ 允许程序员重载 `new` 和 `delete` 运算符，以提供自定义的内存管理策略。例如，可以在分配内存时记录分配的数量，或者实现自己的内存池。

```c
void* operator new(size_t size) {
    std::cout << "Allocating " << size << " bytes" << std::endl;
    return malloc(size);
}

void operator delete(void* pointer) {
    std::cout << "Deallocating memory" << std::endl;
    free(pointer);
}
```

### 3、this指针的原理 如果把this delete，还能用吗，什么场景下还能用？

#### 1. `this` 指针的原理

- 定义：在类的成员函数内部，`this` 指针指向调用该成员函数的对象实例。它是一个隐含参数，所有非静态成员函数都有一个 `this` 指针。
- 类型：`this` 的类型是 `ClassName*`，其中 `ClassName` 是当前类的名称。

#### 2. 使用 `this` 指针

- 访问成员：通过 `this` 指针可以访问当前对象的成员变量和成员函数。

- 链式调用：在某些情况下，使用 `this` 可以实现链式调用，例如：

```c++
class MyClass {
public:
    MyClass& setValue(int value) {
        this->value = value;
        return *this; // 返回当前对象的引用
    }
private:
    int value;
};
```

#### 3. 删除 `this` 指针

如果在成员函数中执行 `delete this`，会导致以下结果：

- 对象销毁：`delete this` 会销毁当前对象，释放其占用的内存。
- 未定义行为：在调用 `delete this` 后，`this` 指针所指向的内存将被标记为可用，但指针仍然存在。因此，试图继续使用该指针将导致未定义行为，可能会导致程序崩溃或错误的结果。

#### 4. 何时可以安全使用 `this`

- 特殊场景：在某些情况下，可能需要使用 `delete this`，例如在实现单例模式或自我管理的对象时。
- 成员函数的结束：在类的成员函数中调用 `delete this` 后，不应再使用任何该对象的成员或方法，除非在调用 `delete` 之前确认不再需要使用。

### 4、进程间的通信方式，线程间的通信方式？

#### 进程间通信（IPC）

进程间通信是指在不同进程之间传递数据的机制。由于进程具有独立的内存空间，因此需要特定的方法来进行数据共享。

1. **管道**（Pipe）
   - 描述：管道提供一种单向的数据流。可以是匿名管道或命名管道。
   - 使用场景：适用于父子进程之间的通信。
2. **消息队列（Message Queue）**
   - 描述：允许进程以消息的形式进行通信，消息可以在队列中存储，接收进程可以从队列中读取消息。
   - 优点：可以支持优先级，消息可以被异步发送和接收。
3. **共享内存（Shared Memory）**
   - 描述：允许多个进程访问同一块内存区域，数据的读写非常高效。
   - 使用场景：适用于需要频繁交换大量数据的情况。
   - 注意：需要额外的同步机制（如信号量）来避免竞争条件。
4. **信号（Signal）**
   - 描述：用于通知进程某些事件的发生。信号是异步的，可以用来通知某个进程需要处理某个特定事件。
   - 优点：简单有效，但通常用于简单的通知。
5. **套接字（Socket）**
   - 描述：套接字是一种通用的 IPC 机制，可以用于不同主机间的通信。分为流式套接字（TCP）和数据报套接字（UDP）。
   - 使用场景：适用于网络通信。

#### 线程间通信（TIC）

线程间通信是在同一进程内的多个线程之间传递数据。由于线程共享同一进程的内存空间，因此通信相对简单。

1. **共享变量**
   - 描述：线程可以直接访问同一进程中的共享变量，进行数据的读写。
   - 注意：需要使用同步机制（如互斥锁、读写锁）来避免数据竞争和不一致性。
2. **条件变量（Condition Variable）**
   - 描述：用于线程间的同步和通信，可以让线程在某个条件不满足时等待，并在条件满足时通知其他线程继续执行。
   - 使用场景：适用于生产者-消费者模型。
3. **信号量（Semaphore）**
   - 描述：信号量是一种计数信号机制，控制对共享资源的访问。可以用于限制同一时间内访问某个资源的线程数。
   - 使用场景：适用于控制并发访问的场景。
4. **消息队列（Message Queue）**
   - 描述：线程也可以使用消息队列进行通信，发送和接收消息以传递数据。
   - 优点：消息队列可以实现异步通信。
5. **事件（Event）**
   - 描述：事件机制允许线程之间的同步。当一个线程设置事件时，其他等待该事件的线程可以被唤醒。
   - 使用场景：用于线程间的协调和状态通知。

### 5、怎么避免死锁？

- 资源分配策略：确保进程在请求资源之前，先释放不必要的资源。可以采用银行家算法动态分配资源。

- 资源排序：为所有资源分配一个固定的顺序，进程按照这个顺序请求资源，避免循环等待。

- 避免持有并等待:

  不允许持有资源时请求新资源：进程在需要额外资源时，必须先释放所有已持有的资源。

- 超时机制

​	设置请求超时：如果进程在一定时间内未能获得资源，则放弃请求并释放所有持有的资源。

- 进程优先级

​	优先级策略：给重要进程优先获取资源的权利，减少资源的竞争。

- 死锁检测与恢复

​	检测：定期检查系统状态，检测死锁的发生，并采取措施（如终止某些进程或回滚操作）来恢复系统。

- 使用非阻塞算法

​	乐观锁和无锁数据结构：使用这些方法可以减少资源占用时间，从而降低死锁发生的概率。

### 6、四种强制类型转换？

#### 1. `static_cast`

- 用途：用于执行编译时类型转换，适用于基本数据类型、指针和引用类型之间的转换。
- 特点：
  - 不会进行运行时检查，因此不适用于需要验证类型安全的情况。
  - 可以进行基本类型（如 `int` 和 `float`）之间的转换。
  - 可以将基类指针转换为派生类指针，但不安全，如果不确定类型，可能会导致未定义行为。

```c++
class Base {};
class Derived : public Base {};

Base* b = new Derived();
Derived* d = static_cast<Derived*>(b);  // 安全转换，假设 b 实际上指向一个 Derived 对象
```

#### 2. `dynamic_cast`

- 用途：用于安全地转换指针或引用，尤其是在多态类型之间进行转换。
- 特点：
  - 仅适用于类层次结构中的指针和引用。
  - 在运行时进行类型检查，如果转换不合法，则返回 `nullptr`（对于指针）或抛出 `std::bad_cast` 异常（对于引用）。
  - 必须至少有一个虚函数，通常是基类的虚析构函数，以支持 RTTI（运行时类型信息）。

```c++
class Base {
    virtual ~Base() {}  // 必须有虚函数
};
class Derived : public Base {};

Base* b = new Base();
Derived* d = dynamic_cast<Derived*>(b);  // 转换失败，d 为 nullptr
```

#### 3. `const_cast`

- 用途：用于去除对象的常量性，主要用于 const 和 volatile 的转换。
- 特点：
  - 允许修改一个 `const` 或 `volatile` 对象的值。
  - 应用场景包括需要调用一个不能接受 `const` 参数的函数时。

```c++
const int a = 10;
int* b = const_cast<int*>(&a);  // 去除 const 限定符
*b = 20;  // 不安全，a 的原始值不可修改
```

#### 4. `reinterpret_cast`

- 用途：用于执行低级别的类型转换，如指针的二进制转换。
- 特点：
  - 可以将任何指针类型转换为任何其他指针类型，不检查安全性。
  - 通常用于系统级编程或底层操作，不推荐在高层次应用中使用。
  - 结果未定义，除非你非常清楚正在做什么。

```c++
long p = 0x12345678;
int* ip = reinterpret_cast<int*>(p);  // 将 long 转换为 int 指针
```

### 7、右值引用和完美转发？

#### 1. 右值引用

定义：右值引用是一种新的引用类型，用于绑定到临时对象（右值），即不再被使用的对象。右值引用使用 `&&` 表示。

用途：

- 资源管理：允许转移资源的所有权，避免不必要的复制，从而提高性能。
- 实现移动语义：通过移动构造函数和移动赋值运算符，能够高效地管理动态分配的资源。

```c++
#include <iostream>
#include <vector>

class MyClass {
public:
    MyClass() { std::cout << "Constructor\n"; }
    MyClass(const MyClass&) { std::cout << "Copy Constructor\n"; }
    MyClass(MyClass&&) noexcept { std::cout << "Move Constructor\n"; }
};

void process(MyClass obj) {
    // Do something with obj
}

int main() {
    MyClass a;                  // 调用构造函数
    MyClass b = std::move(a);  // 调用移动构造函数
    process(b);                // 调用复制构造函数
}
```

`std::move` 将对象 `a` 转换为右值引用，从而调用移动构造函数，避免了复制的开销。

#### 2. 完美转发

定义：完美转发是指在模板中以原始的值类别（左值或右值）转发参数，确保保持参数的值类别。它使用 `std::forward` 函数实现。

用途：当我们希望将参数传递给另一个函数时，能够保留其原始的左值或右值属性。

```c++
#include <iostream>
#include <utility>  // For std::forward

template<typename T>
void wrapper(T&& arg) {
    process(std::forward<T>(arg));  // 完美转发
}

void process(MyClass obj) {
    // Do something with obj
}

int main() {
    MyClass a;
    wrapper(a);               // 将 a 作为左值传递
    wrapper(MyClass());       // 将临时对象作为右值传递
}
```

`wrapper` 函数使用 `T&&` 作为参数，既可以接受左值也可以接受右值。使用 `std::forward<T>(arg)` 将保持 `arg` 的值类别，从而在调用 `process` 时能正确区分左值和右值。

### 8、怎么利用C++新特性实现一个无锁的并发？

实现无锁并发的关键在于使用原子操作和合适的数据结构，以避免传统的锁机制导致的线程竞争和性能瓶颈。

#### 1. 原子操作

C++11 引入了 `std::atomic`，它提供了一种原子类型，可以安全地在多个线程间共享数据而不需要加锁。原子操作是不可中断的操作，保证了在多个线程同时访问时的一致性。

```c++
#include <iostream>
#include <atomic>
#include <thread>
#include <vector>

std::atomic<int> counter(0); // 原子计数器

void increment() {
    for (int i = 0; i < 1000; ++i) {
        counter.fetch_add(1); // 原子地增加计数器
    }
}

int main() {
    std::vector<std::thread> threads;
    for (int i = 0; i < 10; ++i) {
        threads.emplace_back(increment);
    }

    for (auto& t : threads) {
        t.join(); // 等待所有线程完成
    }

    std::cout << "Final counter value: " << counter.load() << std::endl; // 应该是10000
    return 0;
}
```

示例中，多个线程同时增加 `counter`，使用 `fetch_add` 方法保证操作的原子性。

### 2. 无锁数据结构

无锁数据结构通常依赖于原子操作和 CAS（Compare-And-Swap）算法。可以实现如无锁队列、无锁栈等。

示例：无锁栈（使用单向链表实现）：

```c++
#include <iostream>
#include <atomic>
#include <thread>

template <typename T>
class LockFreeStack {
private:
    struct Node {
        T data;
        Node* next;
        Node(T value) : data(value), next(nullptr) {}
    };

    std::atomic<Node*> head;

public:
    LockFreeStack() : head(nullptr) {}

    void push(T value) {
        Node* newNode = new Node(value);
        do {
            newNode->next = head.load();
        } while (!head.compare_exchange_weak(newNode->next, newNode));
    }

    bool pop(T& result) {
        Node* oldHead = head.load();
        while (oldHead && !head.compare_exchange_weak(oldHead, oldHead->next)) {
            // 循环直到成功
        }
        if (!oldHead) return false; // 栈为空
        result = oldHead->data;
        delete oldHead;
        return true;
    }
};

int main() {
    LockFreeStack<int> stack;

    std::thread t1([&]() {
        for (int i = 0; i < 10; ++i) {
            stack.push(i);
        }
    });

    std::thread t2([&]() {
        for (int i = 0; i < 10; ++i) {
            int value;
            if (stack.pop(value)) {
                std::cout << "Popped: " << value << std::endl;
            }
        }
    });

    t1.join();
    t2.join();

    return 0;
}
```

#### 3. 使用 C++ 标准库的无锁功能

C++17 引入了 `std::shared_mutex` 和 `std::shared_lock`，也可以利用一些 STL 容器（如 `std::atomic`、`std::optional` 等）来简化并发控制，但这些通常依赖于传统锁。

### 9、设计模式，使用单例要注意什么，双检锁是绝对安全的吗？

#### 使用单例模式时需要注意：

1. 线程安全：如果单例实例会在多线程环境中被访问，需要确保线程安全。在 C++ 中，可以通过使用互斥锁（`std::mutex`）来保护实例的创建。
2. 延迟初始化：单例的实例通常在第一次被请求时创建。要避免在多线程环境中出现多个实例，常用的策略是使用双重检查锁定（Double-Checked Locking，DCL）。
3. 静态局部变量：在 C++11 及以上版本中，可以利用静态局部变量的初始化特性。编译器会保证静态局部变量的初始化是线程安全的。
4. 防止拷贝和赋值：为了确保单例特性，应该禁用拷贝构造函数和赋值运算符。
5. 全局访问点：提供一个静态方法来获取单例实例。

#### 双重检查锁定（Double-Checked Locking）

双重检查锁定是一种在多线程环境中使用的优化方案。基本思路是：

1. 在访问实例之前先检查实例是否已经存在。
2. 如果不存在，再加锁以创建实例。
3. 在加锁之后，再次检查实例是否已经存在，以防在锁定期间其他线程创建了实例。

```
#include <iostream>
#include <mutex>

class Singleton {
private:
    static Singleton* instance;
    static std::mutex mtx;

    // 私有构造函数
    Singleton() {}

public:
    static Singleton* getInstance() {
        if (instance == nullptr) { // 第一重检查
            std::lock_guard<std::mutex> lock(mtx); // 加锁
            if (instance == nullptr) { // 第二重检查
                instance = new Singleton();
            }
        }
        return instance;
    }
};

// 静态成员初始化
Singleton* Singleton::instance = nullptr;
std::mutex Singleton::mtx;

int main() {
    Singleton* singleton1 = Singleton::getInstance();
    Singleton* singleton2 = Singleton::getInstance();

    std::cout << (singleton1 == singleton2) << std::endl; // 输出 1，表明两个指针指向同一个实例
    return 0;
}
```

双重检查锁定在多线程环境下并不是绝对安全的，特别是在某些编译器和 CPU 架构中，可能会出现“指令重排”导致的初始化问题。

指令重排：在某些情况下，编译器或 CPU 可能会对代码进行重排，导致 `new Singleton()` 还没有完成，但指针已经被设置为非空值。这种情况下，其他线程可能会获取到一个尚未完全构造的对象。

为了避免这种问题，可以使用 C++11 中的 `std::atomic` 来确保在构造完成之前，指针不会被其他线程看到。

```c++
#include <iostream>
#include <atomic>
#include <mutex>

class Singleton {
private:
    static std::atomic<Singleton*> instance;
    static std::mutex mtx;

    Singleton() {}

public:
    static Singleton* getInstance() {
        Singleton* tmp = instance.load(std::memory_order_acquire);
        if (tmp == nullptr) {
            std::lock_guard<std::mutex> lock(mtx);
            tmp = instance.load(std::memory_order_relaxed);
            if (tmp == nullptr) {
                tmp = new Singleton();
                instance.store(tmp, std::memory_order_release);
            }
        }
        return tmp;
    }
};

std::atomic<Singleton*> Singleton::instance{nullptr};
std::mutex Singleton::mtx;

int main() {
    Singleton* singleton1 = Singleton::getInstance();
    Singleton* singleton2 = Singleton::getInstance();
    std::cout << (singleton1 == singleton2) << std::endl; // 输出 1
    return 0;
}
```

