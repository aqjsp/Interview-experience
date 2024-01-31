来源：https://www.nowcoder.com/discuss/552634582416248832

### 1、内存对齐你了解吗，作用是什么，怎么样实现的？

指数据在内存中存储时相对于起始地址的偏移量是数据大小的整数倍。

作用：

- 提高访问速度： 许多计算机体系结构要求数据按照特定的边界地址存储，而不是任意地址。当数据被按照这些边界对齐时，处理器能够更快地访问这些数据，提高数据存取速度。
- 硬件要求： 一些硬件平台对于特定类型的数据要求按照一定的对齐方式存储，不遵循这个规则可能导致硬件异常或性能下降。
- 减少浪费： 内存对齐可以减少内存碎片，提高内存利用率。

在C/C++中，内存对齐是由编译器负责的。编译器会按照平台的要求为数据进行对齐，通常会将数据按照其自身大小对齐到特定字节的倍数。这个特定字节的倍数通常由平台决定，例如，在32位系统中可能是4字节，而在64位系统中可能是8字节。

C/C++中可以使用一些特殊的关键字或编译器指令来控制内存对齐，例如：

`alignas`关键字： C++11引入了`alignas`关键字，用于指定对齐方式。

```C++
alignas(16) struct MyStruct {
    // 结构体成员
};
```

`attribute((aligned(n)))`： 在一些编译器中，可以使用`attribute((aligned(n)))`来指定对齐方式。

```C++
struct MyStruct {
    // 结构体成员
} __attribute__((aligned(16)));
```

### 2、C++的this指针了解吗？有什么作用，是如何实现的？

在C++中，`this` 指针是一个指向当前对象的指针，它是成员函数的隐含参数。`this` 指针的主要作用是允许在一个类的成员函数中访问调用这个函数的对象的地址。

特点：

1. 隐含参数： 在每个成员函数内部，编译器都会自动地传递一个额外的参数，即 `this` 指针，作为函数的隐含参数。这个指针指向调用该函数的对象。
2. 指向当前对象： `this` 指针指向调用成员函数的对象，使得成员函数能够访问调用对象的成员变量和其他成员函数。
3. 生命周期： `this` 指针的生命周期与成员函数的调用周期相同。当成员函数被调用时，`this` 指针被创建；当函数执行结束后，`this` 指针也随之销毁。
4. 用法： 通常，`this` 指针在成员函数中用于解决成员变量与局部变量之间的歧义。如果成员变量和局部变量同名，可以使用 `this` 指针来明确地访问成员变量。

简单例子：

```C++
#include <iostream>

class MyClass {
public:
    void printAddress() {
        std::cout << "Object address: " << this << std::endl;
    }
};

int main() {
    MyClass obj1;
    MyClass obj2;

    obj1.printAddress();
    obj2.printAddress();

    return 0;
}
```

### 3、在成员函数中删除this指针出发生什么，之后还可以使用吗？

在成员函数中删除 `this` 指针是一种危险的行为，因为这样做会导致未定义行为。`this` 指针指向当前对象，而删除它可能导致访问无效的内存地址，引发各种问题，包括程序崩溃。

当 `this` 指针被删除后，对象的成员函数可能会试图访问已经无效的地址，这可能导致未知的行为，例如访问冗余数据、崩溃或数据损坏。

简单的示例，说明了删除 `this` 指针的后果：

```C++
#include <iostream>

class MyClass {
public:
    void deleteThis() {
        delete this;
    }

    void printMessage() {
        std::cout << "Hello, this is MyClass!" << std::endl;
    }
};

int main() {
    MyClass* obj = new MyClass;
    obj->deleteThis(); // 删除 this 指针

    // 调用已删除 this 指针的对象的成员函数，可能导致未定义行为
    obj->printMessage();

    return 0;
}
```

例子中，`deleteThis` 函数试图删除 `this` 指针，而后续对 `printMessage` 函数的调用可能导致程序崩溃或其他不确定的行为。这种用法是不安全和不推荐的，应该避免在成员函数中删除 `this` 指针。正确的做法是在合适的地方使用 `delete` 操作符删除动态分配的对象。

### 4、引用占用内存吗？

引用在 C++ 中并不占用额外的内存。引用是一个别名，它和被引用的对象共享相同的内存地址。因此，引用本身并不占用独立的内存空间，它只是为了方便访问已经存在的对象。

例子：

```C++
#include <iostream>

int main() {
    int x = 42;
    int& ref = x;

    std::cout << "x: " << x << std::endl;   // 输出 x 的值
    std::cout << "ref: " << ref << std::endl; // 输出引用 ref，与 x 的值相同

    ref = 10; // 修改引用的值，也会修改 x 的值

    std::cout << "x after modification: " << x << std::endl;

    return 0;
}
```

例子中，`ref` 是 `x` 的引用，它们共享相同的内存空间。修改 `ref` 的值实际上就是修改 `x` 的值，因为它们指向同一块内存。因此，引用本身并不占用额外的内存。

### 5、C++什么时候会出现越界访问的情况？

C++ 中越界访问是一种程序错误，它可能导致程序崩溃、未定义行为或者产生不可预测的结果。越界访问通常发生在数组、指针、容器等数据结构的操作中。

可能导致越界访问的情况：

1. 数组越界： 访问数组元素时，索引超过数组的有效范围。

```C++
int arr[5];
arr[5] = 42; // 越界访问
```

2. 指针越界： 通过指针访问内存时，超出了指向对象的有效范围。

```C++
int arr[5];
int* ptr = arr;
*(ptr + 5) = 42; // 越界访问
```

3. 迭代器越界： 在使用容器的迭代器时，超出了容器的有效范围。

```C++
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec = {1, 2, 3};
    auto it = vec.begin();
    ++it;
    ++it;
    ++it;
    std::cout << *it << std::endl; // 越界访问
    return 0;
}
```

4. 字符串越界： 对字符串进行操作时，超出了字符串的有效范围。

```C++
#include <iostream>
#include <string>

int main() {
    std::string str = "Hello";
    char ch = str[10]; // 越界访问
    std::cout << ch << std::endl;
    return 0;
}
```

### 6、进程的通信方式有哪些？

1. 管道（Pipes）： 管道是一种半双工通信方式，主要用于具有亲缘关系的进程之间的通信。管道是一种线性数据流，数据只能单向流动。它有两种类型：
   - 无名管道（Anonymous Pipes）： 通常用于父子进程之间的通信。创建管道使用 `pipe()` 系统调用。
   - 命名管道（Named Pipes）： 用于无关联的进程之间的通信，它们以文件系统中的命名管道文件形式存在。
2. 消息队列（Message Queues）： 消息队列允许进程通过消息进行异步通信。消息队列允许多个进程通过将消息发送到队列，然后其他进程从队列中接收消息来进行通信。消息队列通常有操作系统提供的 API 来管理消息的发送和接收。
3. 共享内存（Shared Memory）： 共享内存是一种高效的通信方式，允许多个进程共享相同的物理内存区域。这使得数据在进程之间的传输非常快速，因为它们可以直接读写相同的内存。然而，共享内存需要进行同步以避免数据竞争。
4. 信号（Signals）： 信号是异步通信的一种方式，用于通知进程某些事件的发生，如错误或异常。每个信号都有一个数字标识符，当事件发生时，进程可以注册信号处理程序来处理信号。
5. 套接字（Sockets）： 套接字是一种用于网络通信的通用通信机制，但也可以在同一台计算机上的不同进程之间使用。套接字提供了面向流和面向数据报的通信方式，允许进程通过网络套接字进行通信。
6. 文件（File）： 进程可以通过读写文件来实现通信。一个进程可以将数据写入文件，而另一个进程则可以读取该文件的内容。这种方式不够高效，但是可以应用在不同进程之间的通信需求较少的情况下。
7. 信号量（Semaphores）： 信号量是一种用于控制多个进程对共享资源的访问的同步机制。信号量可以用于避免竞争条件，确保一次只有一个进程可以访问共享资源。
8. 共享文件映射（Memory-Mapped Files）： 共享文件映射允许进程将文件映射到它们的地址空间中，以便多个进程可以访问相同的文件数据。这在共享大量数据时非常有用。

### 7、无锁队列如何实现？

参考代码：

```C++
#include <atomic>

template <typename T>
class LockFreeQueue {
private:
    // 节点结构
    struct Node {
        T data;                    // 节点存储的数据
        std::atomic<Node*> next;   // 指向下一个节点的原子指针

        Node(const T& val) : data(val), next(nullptr) {}
    };

    // 队列头部和尾部指针
    alignas(64) std::atomic<Node*> head;
    alignas(64) std::atomic<Node*> tail;

public:
    // 构造函数，初始化队列头部和尾部
    LockFreeQueue() : head(new Node(T())), tail(head.load()) {}

    // 析构函数，释放队列中的所有节点
    ~LockFreeQueue() {
        while (Node* temp = head.load()) {
            head.store(temp->next);
            delete temp;
        }
    }

    // 入队操作
    void enqueue(const T& value) {
        Node* newNode = new Node(value);
        Node* currentTail = tail.load(std::memory_order_relaxed);

        while (true) {
            Node* currentNext = currentTail->next.load(std::memory_order_relaxed);
            if (currentNext == nullptr) {
                if (currentTail->next.compare_exchange_weak(currentNext, newNode, std::memory_order_release, std::memory_order_relaxed)) {
                    break;
                }
            } else {
                tail.compare_exchange_weak(currentTail, currentNext, std::memory_order_release, std::memory_order_relaxed);
            }
        }

        tail.compare_exchange_weak(currentTail, newNode, std::memory_order_release, std::memory_order_relaxed);
    }

    // 出队操作
    bool dequeue(T& result) {
        Node* currentHead = head.load(std::memory_order_relaxed);

        while (true) {
            Node* currentTail = tail.load(std::memory_order_relaxed);
            Node* currentNext = currentHead->next.load(std::memory_order_relaxed);

            if (currentHead == currentTail) {
                if (currentNext == nullptr) {
                    return false;  // 队列为空
                }
                tail.compare_exchange_weak(currentTail, currentNext, std::memory_order_release, std::memory_order_relaxed);
            } else {
                if (head.compare_exchange_weak(currentHead, currentNext, std::memory_order_release, std::memory_order_relaxed)) {
                    result = currentNext->data;
                    delete currentHead;
                    return true;
                }
            }
        }
    }
};
```

### 8、ping是在哪一层，实现的原理是什么？

它是一个网络工具，它用于测试两台主机之间是否可以通信，以及在网络上发送数据包的往返时间。Ping工具工作在网络层（第三层）和传输层（第四层），主要通过 ICMP 协议来实现。

实现原理：

1. ICMP协议： Ping主要使用Internet控制报文协议（ICMP）来发送探测消息。ICMP是在网络层（第三层）工作的协议，它通常用于主机和路由器之间的错误报告和网络状况的检测。
2. Echo请求和回应： Ping通过发送Echo请求消息到目标主机，目标主机收到请求后，会返回Echo回应消息。Ping工具根据回应消息的时间来计算往返时间（Round-Trip Time，RTT）。
3. 时间戳： Ping可以使用时间戳来测量往返时间。它在发送的Echo请求消息中包含一个时间戳，接收方在返回的Echo回应消息中将这个时间戳原封不动地返回，发送方通过时间戳的差值计算出往返时间。
4. TTL： Ping可以设置TTL字段，TTL是生存时间，每经过一个路由器，TTL减1。当TTL为0时，路由器会丢弃该数据包并发送ICMP "Time Exceeded"消息给源主机，源主机通过这个消息可以知道到达目标主机所经过的路由器数。
5. 连通性测试： 如果目标主机能够正常收到并响应Echo请求，说明网络通畅。如果目标主机无法响应，Ping会报告超时错误。

### 9、HTTPS的流程说一下？

1. 客户端发起请求： 与HTTP一样，HTTPS通信始于客户端向服务器发起请求。这个请求是明文的，因为在此阶段还没有进行加密。
2. 服务器证书： 服务器在响应中返回自己的数字证书，该证书通常由可信的第三方机构（CA，Certificate Authority）签发。证书中包含了服务器的公钥和相关信息。
3. 客户端验证证书： 客户端收到服务器的证书后，会验证证书的有效性。验证包括检查证书的签发机构是否受信任，证书是否过期，以及域名是否匹配。
4. 生成随机密钥： 一旦证书被验证，客户端生成一个用于后续加密通信的随机对称密钥。这个密钥称为"Pre-master secret"。
5. 用服务器公钥加密密钥： 客户端使用服务器的公钥对生成的随机密钥进行加密，然后将加密后的密钥发送给服务器。
6. 服务器解密密钥： 服务器使用自己的私钥对接收到的密文进行解密，得到客户端生成的随机密钥。
7. 建立加密通道： 双方现在都拥有了相同的随机密钥，可以用它来加密和解密后续的通信内容。通信的数据将使用对称密钥进行加密，提供保密性。
8. SSL握手完成： 至此，SSL握手完成，双方可以开始使用加密通道进行安全的通信。客户端和服务器会共享密钥来保护数据的机密性，并确保通信的完整性。

### 10、使用过GDB吗，如何使用的；如果出现CPU高使用，应该如何定位调试？

使用GDB调试：

1. 编译时添加调试信息： 在编译程序时，使用 `-g` 选项生成调试信息。例如：

```C++
g++ -g -o my_program my_program.cpp
```

2. 启动GDB： 在命令行中启动GDB，并指定可执行文件的路径：

```C++
gdb ./my_program
```

3. 设置断点： 在GDB中，使用 `break` 命令设置断点。例如，在 `main` 函数处设置断点：

```C++
break main
```

4. 运行程序： 使用 `run` 命令运行程序，GDB会在断点处停止。

```C++
run
```

5. 查看变量值： 使用 `print` 命令查看变量的值。

```C++
print my_variable
```

6. 单步执行： 使用 `step` 命令单步执行程序，逐行查看代码的执行情况。

```C++
step
```

7. 其他调试命令： GDB提供了许多调试命令，如 `next`（下一步）、`continue`（继续执行）、`info variables`（查看变量信息）等。根据需要使用相应的命令。

定位高CPU使用问题：

1. 使用系统工具： 使用系统工具（如`top`、`htop`等）查看哪个进程占用了高CPU。确定具体是哪个进程引起的问题。
2. 性能分析工具： 使用性能分析工具（如`perf`）来检查程序的性能瓶颈。例如，可以使用以下命令：

```C++
perf record -g -p <pid>
perf report
```

2. 核心转储文件： 如果程序崩溃或占用高CPU，可以生成核心转储文件，然后使用GDB进行分析。

```C++
gcore <pid>
gdb -c core.<pid>
```

3. 分析代码： 使用GDB分析导致高CPU使用的代码部分，查看哪些函数或代码路径消耗了大量的CPU时间。

### 11、说一下智能指针？

1. `std::shared_ptr`：

   - 原理：`std::shared_ptr`是基于引用计数的智能指针，用于管理动态分配的对象。它维护一个引用计数，当计数为零时，释放对象的内存。

   - 使用场景：适用于多个智能指针需要共享同一块内存的情况。例如，在多个对象之间共享某个资源或数据。

   - ```C++
     std::shared_ptr<int> sharedInt = std::make_shared<int>(42);
     std::shared_ptr<int> anotherSharedInt = sharedInt; // 共享同一块内存
     ```

2. `std::unique_ptr`：

   - 原理：`std::unique_ptr`是独占式智能指针，意味着它独占拥有所管理的对象，当其生命周期结束时，对象会被自动销毁。

   - 使用场景：适用于不需要多个指针共享同一块内存的情况，即单一所有权。通常用于资源管理，例如动态分配的对象或文件句柄。

   - ```C++
     std::unique_ptr<int> uniqueInt = std::make_unique<int>(42);
     // uniqueInt 的所有权是唯一的
     ```

3. `std::weak_ptr`：

   - 原理：`std::weak_ptr`是一种弱引用指针，它不增加引用计数。它通常用于协助`std::shared_ptr`，以避免循环引用问题。

   - 使用场景：适用于协助解决`std::shared_ptr`的循环引用问题，其中多个`shared_ptr`互相引用，导致内存泄漏。

   - ```C++
     std::shared_ptr<int> sharedInt = std::make_shared<int>(42);
     std::weak_ptr<int> weakInt = sharedInt;
     ```

4. `std::auto_ptr`（已废弃）：

   - 原理：`std::auto_ptr`是C++98标准引入的智能指针，用于独占地管理对象。但由于其存在潜在的问题，已在C++11中被废弃。

   - 使用场景：在C++98标准中，可用于独占性地管理动态分配的对象。不推荐在现代C++中使用。

   - ```C++
     std::auto_ptr<int> autoInt(new int(42)); // 已废弃
     ```

### 12、右值引用和移动语义的差别？

右值引用：

- 表示形式： 使用 `&&` 表示，例如 `int&&`.
- 作用： 主要用于引用临时对象（右值），即将要销毁的临时对象。
- 生命周期： 只能引用临时对象，不会延长对象的生命周期。
- 例子：

```C++
int&& x = 5;  // x是一个右值引用，引用了临时对象5
```

移动语义：

- 作用： 允许将资源（如内存）从一个对象“移动”到另一个对象，而不是传统的拷贝。
- 优势： 避免了昂贵的深拷贝操作，提高了性能。
- 实现： 通过移动构造函数和移动赋值运算符实现。
- 例子：

```C++
// 移动构造函数
MyClass(MyClass&& other) noexcept {
    // 进行资源的移动，而不是拷贝
}

// 移动赋值运算符
MyClass& operator=(MyClass&& other) noexcept {
    // 进行资源的移动，而不是拷贝
}
```

应用关系：

- 右值引用是基础： 移动语义依赖于右值引用，它允许我们获取对右值的引用。
- 移动语义的高级应用： 移动语义在容器、智能指针等方面发挥了巨大作用，例如通过 `std::move` 转移对象的所有权。

总的来说，右值引用是语言层面提供的一种引用方式，而移动语义是一种利用右值引用来实现资源高效传递的编程技巧。

### 13、coredump 和 minidump 的差别是什么？

1. Core Dump:
   - 格式： `coredump` 是一种标准的操作系统级别的崩溃信息。通常以 `core` 文件的形式存在。
   - 内容： 包含了程序崩溃时的内存内容、寄存器状态等详细信息。
   - 使用： 可以使用调试器（如`gdb`）来分析 `core` 文件，从而了解程序崩溃的原因。
   - 适用范围： 主要用于本地开发环境，对于生产环境可能泄露敏感信息。
2. Minidump:
   - 格式： `minidump` 是一种跨平台的、轻量级的崩溃报告格式。
   - 内容： 包含了程序崩溃时的关键信息，但相较于 `coredump` 来说，它的体积较小。
   - 使用： 可以使用专门的工具或库来分析 `minidump` 文件，例如 Google Breakpad 或 Microsoft Crashpad。
   - 适用范围： 更适用于生产环境，可以更轻便地传递并在没有源代码的情况下进行分析。
3. 用途：
   - Core Dump： 主要用于本地开发、调试和故障排查，通常用于研究问题的根本原因。
   - Minidump： 主要用于产品发布环境，以便在不暴露敏感信息的情况下进行崩溃分析。
4. 安全性：
   - Core Dump： 可能包含敏感信息，需要小心处理以防泄漏。
   - Minidump： 通常设计成不包含敏感信息，以增加安全性。

### 14、说一下设计模式？

单例模式（Singleton Pattern）：

思想： 保证一个类仅有一个实例，并提供一个访问它的全局访问点。

1. 饿汉式（Eager Initialization）

在类加载的时候就创建实例，因此在使用时已经存在一个实例。

实现代码：

```C++
class SingletonEager {
private:
    // 私有的构造函数，防止外部实例化
    SingletonEager() {}

    // 静态实例，在类加载时初始化
    static SingletonEager instance;

public:
    // 公共的访问点
    static SingletonEager* getInstance() {
        return &instance;
    }
};

// 初始化静态实例
SingletonEager SingletonEager::instance;
```

优点：

- 实现简单，线程安全（C++11之前，C++11及之后需要添加一些关键字保证线程安全）。

缺点：

- 如果程序中未使用该单例，会造成资源浪费。

2. 懒汉式（Lazy Initialization）

在需要使用时才创建实例，避免了不必要的资源浪费。

实现代码：

```C++
class SingletonLazy {
private:
    // 私有的构造函数，防止外部实例化
    SingletonLazy() {}

    // 静态实例，使用时初始化
    static SingletonLazy* instance;

public:
    // 公共的访问点，使用时创建实例
    static SingletonLazy* getInstance() {
        if (!instance) {
            instance = new SingletonLazy();
        }
        return instance;
    }
};

// 初始化静态实例为nullptr
SingletonLazy* SingletonLazy::instance = nullptr;
```

优点：

- 节省了资源，只有在需要时才创建实例。

缺点：

- 需要处理线程安全问题，否则可能导致多个线程同时创建实例。
- 在多线程环境下，需要使用双重检查锁定或者其他机制来保证线程安全性。

工厂模式（Factory Pattern）：

- 思想： 定义一个创建对象的接口，但由子类决定实例化哪个类。工厂方法使一个类的实例化延迟到其子类。
- 代码示例：

```C++
// 抽象产品
class Product {
public:
    virtual void create() = 0;
};

// 具体产品 A
class ConcreteProductA : public Product {
public:
    void create() override {
        std::cout << "Product A created.\n";
    }
};

// 具体产品 B
class ConcreteProductB : public Product {
public:
    void create() override {
        std::cout << "Product B created.\n";
    }
};

// 抽象工厂
class Factory {
public:
    virtual Product* createProduct() = 0;
};

// 具体工厂 A
class ConcreteFactoryA : public Factory {
public:
    Product* createProduct() override {
        return new ConcreteProductA();
    }
};

// 具体工厂 B
class ConcreteFactoryB : public Factory {
public:
    Product* createProduct() override {
        return new ConcreteProductB();
    }
};
```

观察者模式（Observer Pattern）：

- 思想：定义了一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖它的对象都得到通知并自动更新。
- 代码示例：

```C++
// 抽象观察者
class Observer {
public:
    virtual void update() = 0;
};

// 具体观察者 A
class ConcreteObserverA : public Observer {
public:
    void update() override {
        std::cout << "Observer A received update.\n";
    }
};

// 具体观察者 B
class ConcreteObserverB : public Observer {
public:
    void update() override {
        std::cout << "Observer B received update.\n";
    }
};

// 主题
class Subject {
private:
    std::vector<Observer*> observers;

public:
    void attach(Observer* observer) {
        observers.push_back(observer);
    }

    void notify() {
        for (auto observer : observers) {
            observer->update();
        }
    }
};
```