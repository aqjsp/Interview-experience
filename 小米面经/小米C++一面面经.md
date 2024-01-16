来源：https://www.nowcoder.com/discuss/557314429990207488

1、对于整个计算机体系的了解？

1. 硬件层：
   - 中央处理器（CPU）：执行指令、进行运算的核心部件。
   - 内存（RAM）：用于存储程序和数据，是CPU可以直接访问的地方。
   - 存储设备（硬盘、固态硬盘等）：用于永久存储数据。
   - 输入设备（键盘、鼠标等）：用于向计算机输入数据。
   - 输出设备（显示器、打印机等）：用于从计算机获取数据。
2. 计算机组成结构：
   - 指令集架构（ISA）：规定了CPU能够执行的指令的集合。
   - 总线系统：连接CPU、内存和其他硬件组件的通信路径。
   - 输入输出系统：处理输入输出设备与计算机之间的通信。
3. 操作系统：
   - 内核：控制计算机硬件，并提供基本的系统服务。
   - 文件系统：管理存储设备上的文件。
   - 进程管理：管理程序的执行。
   - 内存管理：分配和管理计算机内存。
4. 编程语言和应用软件：
   - 编程语言：用于编写应用程序的工具，例如C++、Java、Python等。
   - 应用软件：通过编程语言开发的软件，用于完成各种任务，例如文本编辑器、浏览器、办公套件等。
5. 网络和通信：
   - 计算机网络：允许计算机之间进行通信和数据交换。
   - 协议：规定了计算机之间通信的规则，例如TCP/IP协议。
6. 安全和隐私：
   - 网络安全：防止未经授权的访问和数据泄漏。
   - 加密：保护数据的机制，使其难以被未经授权的人读取。
7. 人机交互：
   - 图形用户界面（GUI）：通过图形元素（窗口、图标等）和用户操作实现交互。
   - 人机界面设计：通过设计用户友好的界面来提高用户体验。

2、进程和线程？

进程（Process）：

1. 定义：进程是计算机中运行中的程序的实例。每个进程都有自己独立的内存空间，包括代码段、数据段、堆和栈。不同进程之间的数据通常不能直接共享。
2. 特点：
   1. 独立性：进程是独立的执行单位，一个进程的崩溃通常不会影响其他进程。
   2. 资源分配：进程拥有独立的资源，如文件描述符、打开的文件等。
3. 通信：进程之间的通信相对复杂，通常需要采用进程间通信（IPC）的机制，如消息队列、共享内存、管道等。
4. 创建和销毁：创建新的进程相对较慢，销毁进程也需要较多的系统资源。

线程（Thread）：

1. 定义：线程是进程的一部分，是一个执行路径，一个进程可以包含多个线程。线程共享进程的资源，包括代码段、数据段、堆和栈，但每个线程拥有独立的栈。
2. 特点：
   1. 轻量级：线程比进程轻量级，创建和销毁线程的开销相对较小。
   2. 资源共享：线程之间共享相同的地址空间和其他资源，通信更容易。
3. 通信：线程之间通信相对容易，可以通过共享内存等方式直接进行数据交换。
4. 创建和销毁：创建新的线程相对较快，销毁线程的开销也较小。

进程和线程的关系：

- 进程内的多个线程共享相同的进程资源，但拥有独立的执行路径。
- 多个进程之间相互独立，拥有各自独立的资源。

选择使用场景：

- 当需要更高的隔离性和安全性时，使用进程。
- 当需要更轻量级的并发和更容易共享数据时，使用线程。

3、生产者消费者怎么同步的？

1. 互斥锁（Mutex）：使用互斥锁确保在同一时刻只有一个线程可以访问共享的数据结构。生产者和消费者在访问共享缓冲区时会使用互斥锁进行保护，防止多个线程同时对缓冲区进行操作。
2. 条件变量（Condition Variable）：条件变量用于在某个条件满足时通知其他线程。在生产者-消费者问题中，可以使用条件变量来通知消费者在缓冲区非空时进行消费，或通知生产者在缓冲区未满时进行生产。
3. 信号量（Semaphore）：信号量是一种更一般化的同步机制，可以用于控制对共享资源的访问。在生产者-消费者问题中，可以使用信号量来表示缓冲区中剩余的空间数量，生产者和消费者在操作缓冲区时需要获取和释放信号量。
4. 阻塞队列（Blocking Queue）：使用阻塞队列作为共享缓冲区，当队列为空时，消费者会阻塞等待直到有新的数据产生；当队列已满时，生产者会阻塞等待直到有空间可以存放数据。
5. 管程（Monitor）：管程是一种高级的同步机制，通过将共享资源和对资源的操作封装在一起，确保只有一个线程能够在任一时刻访问这些资源。生产者和消费者可以通过管程进行同步。

4、无锁队列？

无锁队列（Lock-Free Queue）是一种并发数据结构，它允许多个线程在没有互斥锁的情况下并发地对队列进行操作。无锁队列通常采用一些原子操作，如 CAS（Compare-And-Swap）等，来保证并发的正确性。

来一个简单的无锁队列的示例：

```C++
#include <atomic>
#include <memory>

template <typename T>
class LockFreeQueue {
private:
    struct Node {
        T data;
        std::unique_ptr<Node> next;

        Node(T val) : data(val), next(nullptr) {}
    };

    std::atomic<Node*> head;
    std::atomic<Node*> tail;

public:
    LockFreeQueue() : head(new Node(0)), tail(head.load()) {}

    void enqueue(T value) {
        std::unique_ptr<Node> newNode = std::make_unique<Node>(value);
        Node* oldTail = tail.exchange(newNode.get());
        oldTail->next = std::move(newNode);
    }

    bool dequeue(T& result) {
        Node* oldHead = head.load();
        while (oldHead != tail.load()) {
            if (head.compare_exchange_weak(oldHead, oldHead->next.get())) {
                result = oldHead->data;
                return true;
            }
        }
        return false;
    }
};
```

这个简单的无锁队列实现，其中使用了 `std::atomic` 类型来保证对共享数据的原子操作。enqueue 操作通过 `exchange` 原子操作更新队列的尾部，dequeue 操作通过 `compare_exchange_weak` 原子操作更新队列的头部。

5、C11新特性？

1. 自动类型推导（Auto）： 允许编译器推导变量的类型，使代码更加简洁。

```C++
auto x = 5; // x的类型将被推导为int
```

2. 范围-based for 循环： 简化了对容器元素的遍历。

```C++
std::vector<int> numbers = {1, 2, 3, 4, 5};
for (const auto& num : numbers) {
    // 使用num
}
```

3. 智能指针： 引入了`std::shared_ptr`和`std::unique_ptr`等智能指针，用于管理动态分配的内存，帮助防止内存泄漏。

```C++
std::shared_ptr<int> sharedPtr = std::make_shared<int>(42);
```

4. Lambda 表达式： 允许在函数内部定义匿名函数，提高代码可读性和灵活性。

```C++
auto add = [](int a, int b) { return a + b; };
```

5. nullptr： 引入了空指针常量`nullptr`，用于替代传统的空指针`NULL`。

```C++
int* ptr = nullptr;
```

6. 强制类型转换（Type Casting）： 引入了`static_cast`、`dynamic_cast`、`const_cast`、`reinterpret_cast`等更安全和灵活的类型转换操作符。

```C++
double x = 3.14;
int y = static_cast<int>(x);
```

7. 右值引用和移动语义： 支持通过右值引用实现移动语义，提高了对临时对象的处理效率。

```C++
std::vector<int> getVector() {
    // 返回一个临时vector
    return std::vector<int>{1, 2, 3};
}

std::vector<int> numbers = getVector(); // 使用移动语义
```

8. 新的容器和算法： 引入了新的容器，如`std::unordered_map`、`std::unordered_set`，以及一些新的算法。

```C++
std::unordered_map<int, std::string> myMap = {{1, "one"}, {2, "two"}};
```

9. 线程支持（std::thread）： 提供了原生的多线程支持，使得并发编程更加方便。

```C++
#include <thread>

void myFunction() {
    // 线程执行的代码
}

int main() {
    std::thread t(myFunction);
    t.join(); // 等待线程结束
    return 0;
}
```

6、引用计数怎么实现，在哪里？

引用计数（Reference Counting）是一种内存管理技术，它主要用于跟踪对象被引用的次数。在引用计数中，每个对象都有一个计数器，记录着当前对象被引用的次数。当对象被引用时，计数器加1；当引用失效时，计数器减1。当计数器为零时，表示没有任何引用，可以安全地释放对象。

在C++中，引用计数通常通过智能指针实现，特别是`std::shared_ptr`。`std::shared_ptr` 使用引用计数来跟踪共享的对象。计数器存储在一个控制块（control block）中，这个控制块同时还包含了指向实际对象的指针。

给个例子，展示如何使用 `std::shared_ptr` 实现引用计数：

```C++
#include <memory>
#include <iostream>

class MyClass {
public:
    MyClass() {
        std::cout << "Object created." << std::endl;
    }

    ~MyClass() {
        std::cout << "Object destroyed." << std::endl;
    }
};

int main() {
    std::shared_ptr<MyClass> ptr1 = std::make_shared<MyClass>();
    std::shared_ptr<MyClass> ptr2 = ptr1;

    std::cout << "Use count: " << ptr1.use_count() << std::endl;

    // ptr2 被销毁后，计数器减1
    ptr2.reset();

    std::cout << "Use count: " << ptr1.use_count() << std::endl;

    // ptr1 被销毁后，计数器减1，为零时，MyClass 对象被销毁
    ptr1.reset();

    return 0;
}
```

在这个例子中，`std::shared_ptr` 内部维护了一个引用计数，通过 `use_count` 方法可以获取当前的引用计数。在 `ptr2.reset()` 后，`ptr2` 被销毁，引用计数减1。在 `ptr1.reset()` 后，`ptr1` 被销毁，引用计数再次减1，因为此时引用计数为零，`MyClass` 对象会被销毁。

7、弱指针？

弱指针（Weak Pointer）是 C++11 引入的一种智能指针，用于解决 `std::shared_ptr` 可能引发的循环引用问题。与 `std::shared_ptr` 不同的是，弱指针并不增加引用计数，因此不会影响对象的生命周期。

主要特点和用途：

1. 不增加引用计数： 弱指针不会增加所指向对象的引用计数。当最后一个强引用（`std::shared_ptr`）销毁时，无论弱指针是否存在，引用计数都会减少。
2. 不影响对象生命周期： 弱指针不会影响所指向对象的生命周期。当对象被销毁后，弱指针会被自动置空，不再指向任何对象。
3. 用于解决循环引用： 当两个或多个对象相互持有对方的 `std::shared_ptr`，可能形成循环引用，导致对象无法正确释放。使用弱指针可以打破循环引用，防止内存泄漏。

使用示例：

```C++
#include <memory>
#include <iostream>

class MyClass;

int main() {
    std::shared_ptr<MyClass> ptr1 = std::make_shared<MyClass>();
    std::weak_ptr<MyClass> weakPtr = ptr1;

    // 使用弱指针获取共享指针
    if (auto sharedPtr = weakPtr.lock()) {
        std::cout << "Object is still alive." << std::endl;
    } else {
        std::cout << "Object has been destroyed." << std::endl;
    }

    // 释放 ptr1 后，弱指针变为空
    ptr1.reset();

    if (auto sharedPtr = weakPtr.lock()) {
        std::cout << "Object is still alive." << std::endl;
    } else {
        std::cout << "Object has been destroyed." << std::endl;
    }

    return 0;
}
```

8、TCP和UDP区别？

1. 连接性：
   - TCP是面向连接的协议。在建立通信之前，TCP会建立一个连接，确保数据的可靠传输，然后在通信结束后关闭连接。这种连接性保证了数据的可靠性，但会引入一定的延迟。
   - UDP是无连接的协议。它不会建立连接，直接将数据包发送到目标，不保证数据的可靠性，但具有低延迟的优势。
2. 数据可靠性：
   - TCP提供数据的可靠传输。它使用确认机制和重传来确保数据的完整性和可靠性。如果数据包在传输过程中丢失或损坏，TCP会重新发送丢失的数据。
   - UDP不提供数据的可靠性。它将数据包发送出去，但不保证它们的到达。丢失、重复或无序的数据包在UDP中是常见的，应用程序需要自行处理。
3. 流量控制：
   - TCP具有流量控制机制，可以根据接收端的处理能力来调整数据的发送速率，以避免过载。
   - UDP没有流量控制机制，发送方将数据包发送出去，不考虑接收方的处理速度，可能导致网络拥塞。
4. 顺序性：
   - TCP保持数据的顺序性。发送的数据包将按照发送顺序在接收端被重建。
   - UDP不保证数据包的顺序性。数据包可能以不同的顺序到达接收端。
5. 头部开销：
   - TCP头部较大，包含许多控制信息，这增加了数据包的大小。
   - UDP头部较小，只包含少量的必要信息。
6. 适用场景：
   - TCP适用于需要可靠数据传输的应用程序，如网页浏览、电子邮件、文件传输等。
   - UDP适用于对数据传输延迟要求较高的应用程序，如音频和视频流、在线游戏等。

9、类中哪些变量占内存，哪些不占内存？

1. 成员变量：类的数据成员（成员变量）会占用内存。每个对象的成员变量在内存中都有一份拷贝。
2. 静态成员变量：静态成员变量是类的所有对象共享的，它们存储在类的静态存储区域而不是对象自己的内存中。
3. 成员函数：类的成员函数不占用对象的内存空间。它们是共享的代码块，不随对象的增多而增加。
4. 静态成员函数：与静态成员变量类似，静态成员函数也不依赖于具体对象，不占用对象的内存。
5. 虚函数和虚函数表指针：如果类包含虚函数，每个对象都包含一个指向虚函数表的指针（通常称为虚函数表指针）。虚函数表本身是静态的，但指针占用每个对象的内存。
6. 虚基类的偏移量：当类包含虚基类时，每个对象可能包含一个指示虚基类位置的偏移量。
7. 内存对齐填充：编译器可能会在成员变量之间填充一些额外的字节，以保证每个成员变量的地址是对齐的。

10、介绍一下虚表？

虚函数表（Virtual Function Table，简称 vtable）是C++中实现多态的关键机制之一，用于支持动态绑定。它是在包含虚函数的类的每个对象的内存布局中的一个指针，指向该类的虚函数表。

以下是虚函数表的基本概念和工作原理：

1. 虚函数：在C++中，通过关键字 `virtual` 声明的成员函数称为虚函数。虚函数允许在运行时动态绑定，即在程序执行过程中确定调用的实际函数。
2. 虚函数表指针（vptr）：每个包含虚函数的类的对象都包含一个指向虚函数表的指针，通常称为虚函数表指针（vptr）。这个指针存储在对象的内存布局的最开始位置。
3. 虚函数表（vtable）：虚函数表是一个数组，包含该类的所有虚函数的地址。每个类只有一个虚函数表，被所有该类的对象所共享。
4. 虚函数调用：当通过指向基类对象的指针或引用调用虚函数时，程序将使用虚函数表指针找到对应的虚函数表，然后在虚函数表中查找并调用相应的虚函数。这确保了正确的函数被调用，实现了多态。
5. 派生类的虚函数表：派生类继承了基类的虚函数表，并在其中添加或重写自己的虚函数。因此，派生类的虚函数表扩展了基类的虚函数表。

给个代码作参考：

```C++
class Base {
public:
    virtual void foo() {
        // Base class implementation
    }
};

class Derived : public Base {
public:
    virtual void foo() override {
        // Derived class implementation
    }
};

int main() {
    Base* ptr = new Derived();  // Upcasting
    ptr->foo();  // 调用的是 Derived 类的虚函数

    delete ptr;
    return 0;
}
```

11、哪些地方会用到this指针？

`this` 指针是 C++ 中的一个隐含指针，它指向当前对象。在类的成员函数中，可以通过 `this` 指针访问当前对象的成员变量和成员函数。

使用场景：

1. 区分成员变量和局部变量： 在成员函数中，如果有一个局部变量和一个成员变量同名，可以使用 `this` 指针来明确指出访问的是成员变量还是局部变量。

```C++
class MyClass {
private:
    int x;

public:
    void setX(int x) {
        this->x = x;  // 使用this指针区分成员变量和局部变量
    }
};
```

2. 在成员函数中返回对象自身： 可以在成员函数中返回对象自身，以便支持链式调用。

```C++
class MyClass {
private:
    int value;

public:
    MyClass& setValue(int val) {
        this->value = val;
        return *this;  // 返回对象自身，支持链式调用
    }
};
```

3. 在构造函数和析构函数中： 在构造函数和析构函数中，`this` 指针指向被构造或销毁的对象。

```C++
class MyClass {
public:
    MyClass(int x) {
        this->x = x;
        // 构造函数中的this指针
    }

    ~MyClass() {
        // 析构函数中的this指针
    }
private:
    int x;
};
```

4. 在类的静态成员函数中： 静态成员函数是与类关联而不是与对象关联的，因此在静态成员函数中不可以使用 `this` 指针。

```C++
class MyClass {
public:
    static void staticFunction() {
        // 在静态成员函数中不能使用this指针
    }
};
```

总的来说，`this` 指针用于在类的成员函数中指向调用该函数的对象，提供对当前对象的访问。

12、单例模式有几个锁？

1. 懒汉式： 在第一次使用时才创建实例，可能涉及到多线程环境，需要考虑线程安全。

双重检查锁（Double-Checked Locking）： 通过两次检查锁定来确保只有一个线程创建实例，避免了不必要的锁开销。这种方式通常使用两个锁，外层锁用于减小锁的粒度，内层锁用于在实例为 null 时创建实例。

给个例子：

```C++
class Singleton {
private:
    static Singleton* instance;
    static std::mutex mtx;

    Singleton() {}

public:
    static Singleton* getInstance() {
        if (instance == nullptr) {
            std::unique_lock<std::mutex> lock(mtx);
            if (instance == nullptr) {
                instance = new Singleton();
            }
        }
        return instance;
    }
};

Singleton* Singleton::instance = nullptr;
std::mutex Singleton::mtx;
```

2. 饿汉式： 在类加载时就创建实例，不存在多线程环境时天然是线程安全的。

```C++
class Singleton {
private:
    static Singleton* instance;

    Singleton() {}

public:
    static Singleton* getInstance() {
        return instance;
    }
};

Singleton* Singleton::instance = new Singleton();
```

13、内联函数用来解决什么问题

内联函数是一种在编译器编译阶段将函数的代码嵌入到调用处的机制。内联函数的主要目的是提高程序的执行效率，解决函数调用带来的一些开销。

内联函数的主要作用和解决的问题：

1. 减少函数调用开销：函数调用会带来一定的开销，包括压栈、跳转等操作。对于短小的函数，这些开销可能比函数体执行的开销还要大。内联函数将函数的代码嵌入到调用处，避免了这些额外的开销。
2. 提高执行效率：减少函数调用开销可以提高程序的执行效率，特别是在频繁调用的情况下。内联函数通常用于一些简单的、频繁调用的函数，例如获取某个对象的长度、执行简单的数学运算等。
3. 省去函数调用的栈操作：函数调用需要在调用时将一些寄存器的值保存到栈上，然后在返回时再将这些值还原，而内联函数的代码直接嵌入到调用处，省去了这些栈操作。
4. 适用于小型函数：内联函数适用于小型函数，即函数体较小，执行时间短。如果函数体较大，内联可能会导致代码膨胀，反而降低了缓存的效率。

14、内联和宏定义的区别？

内联函数和宏定义都有类似的作用，可以减少函数调用的开销，提高代码执行效率。

一些区别：

1. 类型检查：
   - 内联函数： 内联函数在编译时进行类型检查，因为它们是真正的函数。
   - 宏定义： 宏定义是简单的文本替换，不进行类型检查。这可能导致一些潜在的错误，因为它不会像内联函数那样进行参数类型的检查。
2. 作用域：
   - 内联函数： 内联函数是有作用域的，具有独立的命名空间。
   - 宏定义： 宏定义是简单的文本替换，没有独立的作用域，可能会引发命名冲突。
3. 调试信息：
   - 内联函数： 内联函数会保留调试信息，可以在调试时提供更好的代码可读性。
   - 宏定义： 宏定义只是简单的文本替换，不会保留与原始代码之间的关系，可能在调试时不太友好。
4. 错误处理：
   - 内联函数： 内联函数遵循函数的错误处理机制，可以在函数内使用 `return` 语句返回错误码。
   - 宏定义： 宏定义不能像函数那样使用 `return` 语句，可能需要其他的错误处理机制。
5. 语法：
   - 内联函数： 内联函数使用函数语法，具有函数的结构和语法。
   - 宏定义： 宏定义使用预处理器的文本替换，不具备函数的结构，可能会导致一些意外的行为。
6. 类型安全：
   - 内联函数： 内联函数提供类型安全，编译器会进行参数类型检查。
   - 宏定义： 宏定义没有类型检查，容易引入类型不匹配的错误。

15、const？

1. 声明常量：

```C++
const int MAX_VALUE = 100;  // 常量值
const double PI = 3.14159;  // 常量值
```

`MAX_VALUE` 和 `PI` 被声明为常量，它们的值不能在后续的代码中修改。

2. 声明常量指针/引用：

```C++
int number = 42;
const intptr = &number;  // 常量指针，指向的值不可修改
const int& ref = number;   // 常量引用，引用的值不可修改
```

在上述代码中，`ptr` 是一个指向常量整数的指针，意味着通过 `ptr` 不能修改所指向的整数的值。同样，`ref` 是一个常量引用，不能通过 `ref` 修改 `number` 的值。

3. 常量成员函数：

```C++
class MyClass {
public:void regularFunction() {
        // 可以修改成员变量
        variable = 10;
    }
void constFunction() const {
        // 不能修改成员变量
        // variable = 10;  // 编译错误
    }

private:
    int variable;
};
```

在类中，如果一个成员函数被声明为 `const`，它意味着在这个函数中不能修改任何类成员的值（除非该成员被声明为 `mutable`）。

16、C和C++的特点

C 语言的特点：

1. 过程性编程：是一种过程性的编程语言，程序主要由函数组成，它按照线性的流程执行。
2. 面向过程：主要是面向过程的，强调的是函数、数据和流程的结构化。
3. 低级语言：提供了对硬件的较低层次的控制，允许直接访问内存和硬件，这使得 C 成为系统级编程的首选语言。
4. 轻量级：语言的设计追求简洁、高效，没有提供很多高级特性，这使得它非常适合编写高效的系统级代码。

C++ 语言的特点：

1. 面向对象：一种面向对象的编程语言，支持类、继承、多态等面向对象的概念。
2. 扩展性： 是在 C 语言基础上扩展而来的，保留了 C 的所有特性，并引入了一些新的特性，使得它更加灵活和功能丰富。
3. 封装性： 支持封装，可以将数据和操作数据的方法封装在一起形成类，提高了代码的模块化和可维护性。
4. 继承性：支持继承，允许通过建立类的层次结构来共享和扩展代码。
5. 多态性： 具有多态性，可以通过函数重载和虚函数实现多态性，提高了代码的灵活性和可扩展性。
6. 模板：引入了模板机制，支持泛型编程，使得数据类型更加灵活。
7. 标准模板库（STL）：提供了一个强大的标准模板库，其中包含了很多通用的数据结构和算法，方便开发人员使用。

17、join和detach？

1. `std::thread::join()`：

   - `join` 方法用于等待线程的结束。当一个线程调用 `join` 时，它会阻塞当前线程，直到被调用 `join` 的线程执行完成。这是一种线程同步的机制，确保主线程等待子线程执行完毕再继续执行。

   - 如果不调用 `join` 而直接销毁拥有线程的 `std::thread` 对象，程序会调用 `std::terminate` 终止程序。因此，`join` 是确保线程安全结束的一种方式。

```C++
#include <iostream>
#include <thread>

void myFunction() {
    // 线程的具体工作
}

int main() {
    std::thread myThread(myFunction);

    // 等待线程执行完毕
    myThread.join();

    std::cout << "Main thread continues..." << std::endl;

    return 0;
}
```

2. `std::thread::detach()`：

   - `detach` 方法用于将线程和 `std::thread` 对象分离，使得线程成为后台线程，不再和 `std::thread` 对象有关联。

   - 当调用 `detach` 后，线程在运行时独立执行，不再受 `std::thread` 对象的控制。这样的线程通常在执行完任务后自行销毁，不需要等待主线程。

```C++
#include <iostream>
#include <thread>

void myFunction() {
    // 线程的具体工作
}

int main() {
    std::thread myThread(myFunction);

    // 分离线程，使得线程成为后台线程
    myThread.detach();

    // 主线程继续执行，不等待子线程

    return 0;
}
```

18、父类和子类构造顺序？

1. 基类构造函数：
   - 首先，基类的构造函数会被调用。如果有多个基类，它们的构造函数按照它们在派生类中的声明顺序被调用。
   - 基类的构造函数负责初始化基类部分的成员。
2. 派生类构造函数：
   - 接着，派生类的构造函数被调用。派生类的构造函数在基类构造函数执行完毕后才开始执行。
   - 派生类构造函数负责初始化派生类新增的成员和执行派生类自己的构造逻辑。

给个例子：

```C++
#include <iostream>

class Base {
public:
    Base() {
        std::cout << "Base Constructor" << std::endl;
    }
};

class Derived : public Base {
public:
    Derived() {
        std::cout << "Derived Constructor" << std::endl;
    }
};

int main() {
    Derived derivedObject;

    return 0;
}
```

输出结果：

```C++
Base Constructor
Derived Constructor
```

19、构造/析构可以为虚么？

构造函数：

1. 构造函数不能声明为虚函数：

   构造函数不能被声明为虚函数。虚函数的调用是通过对象的虚函数表（vtable）来实现的，而在构造函数的执行过程中，对象的虚函数表还没有被构造，因此无法使用虚函数机制。

2. 虚构函数的使用：

   如果需要在析构对象时调用派生类的析构函数，可以将基类的析构函数声明为虚函数。这是为了确保在通过基类指针删除指向派生类对象的对象时，会调用派生类的析构函数。

析构函数：

1. 虚析构函数：

   如果基类的析构函数不声明为虚函数，当通过基类指针删除指向派生类对象的对象时，只会调用基类的析构函数，而不会调用派生类的析构函数。这可能导致资源泄漏或不正确的清理。

2. 手动调用析构函数：

   在某些情况下，如果不将析构函数声明为虚函数，也可以通过显式调用派生类的析构函数来达到正确清理资源的目的，但这通常不是推荐的做法。

```C++
class Base {
public:
    // 构造函数不能声明为虚函数
    Base() {
        std::cout << "Base Constructor" << std::endl;
    }

    // 虚析构函数
    virtual ~Base() {
        std::cout << "Base Destructor" << std::endl;
    }
};

class Derived : public Base {
public:
    // 构造函数
    Derived() {
        std::cout << "Derived Constructor" << std::endl;
    }

    // 虚析构函数
    virtual ~Derived() {
        std::cout << "Derived Destructor" << std::endl;
    }
};

int main() {
    Base* obj = new Derived(); // 通过基类指针指向派生类对象

    delete obj; // 通过基类指针删除对象，应该调用派生类的析构函数

    return 0;
}
```