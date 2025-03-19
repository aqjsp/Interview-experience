# 腾讯IEG后端开发面经

> 来源：https://www.nowcoder.com/discuss/469546084872822784?sourceSSR=search

### C++

## 1、map unordered_map底层？

#### `std::map`

##### 底层实现

`std::map` 通常基于平衡二叉搜索树来实现，其中最常见的是红黑树。这是一种自平衡的二叉查找树，能够保证在最坏情况下查找、插入、删除操作的时间复杂度均为 O(log n)。

##### 实现特点

有序性：红黑树会自动维护节点的有序性，因而 `std::map` 中的键值对是按照键的大小排序的。迭代器遍历时会按照升序顺序访问所有元素。

稳定性：红黑树在插入或删除节点时，通过颜色标记和旋转操作来维持树的平衡，这保证了操作的最坏情况性能。

内存分配：每个元素都作为树的节点存储，节点通常包含键、值、左右子指针、父指针以及颜色信息。这意味着每个节点的内存开销相对较大，但可以高效支持有序遍历和区间查找。

#### `std::unordered_map`

##### 底层实现

`std::unordered_map` 通常基于哈希表实现，采用**分离链表法**（Separate Chaining）或开放地址法（Open Addressing）来解决哈希冲突。大部分实现（例如 GNU STL、libc++）采用分离链表，每个桶内存放一个链表或其他链式结构（如前向链表）。

##### 实现特点

无序性：由于哈希表的原理决定了元素的存储位置不具备顺序性，因此 `std::unordered_map` 中的元素是无序的。如果需要按顺序遍历，则需要额外排序。

平均时间复杂度：哈希表能在平均情况下实现 O(1) 的查找、插入和删除操作。但在最坏情况下（例如所有元素都哈希到同一桶），时间复杂度会退化为 O(n)。为了保持良好性能，容器会维持合适的负载因子（load factor），当负载因子过高时会自动进行再散列（rehashing），重新分配桶并将元素重新分布。

内存开销：哈希表需要一个桶数组来存储链表的指针，并且每个元素以节点形式存在于对应桶的链表中。虽然内存利用率依赖于负载因子，但整体上查找速度非常快。

## 2、static

#### 静态局部变量

在函数内部使用 `static` 修饰变量时，该变量具有静态存储周期（static storage duration），即只会被分配一次，程序运行期间一直存在，而不仅仅是函数调用期间。

示例：

```c++
#include <iostream>

void counter() {
    static int count = 0; // 仅初始化一次
    count++;
    std::cout << "Count: " << count << std::endl;
}

int main() {
    counter(); // Count: 1
    counter(); // Count: 2
    counter(); // Count: 3
    return 0;
}
```

#### 静态全局变量

当在全局作用域中使用 `static` 修饰变量或函数时，它会使这些变量或函数具有内部链接。也就是说，这些符号仅在定义它们的翻译单元（通常是一个源文件及其包含的头文件）中可见，不能被其他源文件链接使用。

示例：

```c++
// file1.cpp
#include <iostream>
static int globalVar = 10; // 文件内可见

void printGlobal() {
    std::cout << "Global: " << globalVar << std::endl;
}

int main() {
    printGlobal(); // Global: 10
    return 0;
}

// file2.cpp
// extern int globalVar; // 错误，链接时找不到
```

#### 静态函数

函数前加 static。限制在当前文件内，其他文件无法调用。

示例：

```c++
// file1.cpp
#include <iostream>
static void helper() { // 文件内函数
    std::cout << "Helper called\n";
}

int main() {
    helper(); // Helper called
    return 0;
}

// file2.cpp
// void helper(); // 错误，无法链接
```

#### 类中的静态成员变量

静态成员变量属于整个类，而不属于某个特定对象。所有类的实例共享同一份静态数据成员。

示例：

```c++
#include <iostream>

class MyClass {
public:
    static int count; // 声明
    MyClass() { count++; }
};

int MyClass::count = 0; // 定义并初始化

int main() {
    MyClass obj1;
    MyClass obj2;
    std::cout << "Count: " << MyClass::count << std::endl; // Count: 2
    return 0;
}
```

#### 类中的静态成员函数

静态成员函数同样属于整个类，而不是某个对象。它们没有 `this` 指针，不能访问非静态成员，只能直接访问静态数据成员或调用其他静态成员函数。

示例：

```c++
#include <iostream>

class MyClass {
public:
    static int count;
    static void increment() {
        count++;
        // std::cout << x; // 错误，无 this，无法访问非静态成员
    }
};

int MyClass::count = 0;

int main() {
    MyClass::increment();
    MyClass::increment();
    std::cout << "Count: " << MyClass::count << std::endl; // Count: 2
    return 0;
}
```

## 3、const

#### 常量变量

```c++
const int maxSize = 100;
// maxSize = 200; // 编译错误，不能修改 const 变量的值
```

`const` 变量必须在声明时初始化，否则编译器无法确定其只读值。

#### 常量指针与指针常量

##### 指向常量的指针

指针指向的数据是只读的，但指针本身可以修改。

```c++
const int *p;      // 或 int const *p;
int value = 10;
p = &value;        // 合法：可以修改 p 指向哪个地址
// *p = 20;        // 错误：不能修改 p 指向的数据
```

##### 常量指针

指针本身是只读的，指向的数据可以修改。

```c++
int *const p = &value;
// p = &otherValue; // 错误：p 本身不可修改
*p = 20;           // 合法：可以修改 p 指向的数据
```

##### 指向常量的常量指针

指针和内容都不可变。

```c++
const int *const p = &value;
// p 和 *p 都不能修改
```

#### 函数参数中的 const

修饰参数，防止函数修改传入数据。

##### 示例1：值传递

值传递中 const 作用有限，仅防止函数内修改。

```c++
#include <iostream>

void print(const int x) {
    // x = 20; // 错误：不能修改
    std::cout << "x: " << x << std::endl;
}

int main() {
    int a = 10;
    print(a); // x: 10
    return 0;
}
```

##### 示例2：引用传递

```c++
#include <iostream>

void modify(const int& x) {
    // x = 20; // 错误：不能修改
    std::cout << "x: " << x << std::endl;
}

int main() {
    int a = 10;
    modify(a); // x: 10
    std::cout << "a: " << a << std::endl; // a: 10
    return 0;
}
```

##### 示例3：指针参数

```c++
#include <iostream>

void printPtr(const int* ptr) {
    // *ptr = 20; // 错误：不能修改
    std::cout << "*ptr: " << *ptr << std::endl;
}

int main() {
    int a = 10;
    printPtr(&a); // *ptr: 10
    return 0;
}
```

#### 类中的 const 成员变量

类中声明为 const 的成员变量。

特点：

- 初始化必须在构造函数初始化列表中完成。
- 对象创建后不可修改。

示例：

```c++
#include <iostream>

class MyClass {
    const int id; // 常量成员
public:
    MyClass(int val) : id(val) {} // 初始化列表
    int getId() const { return id; } // const 成员函数
};

int main() {
    MyClass obj(42);
    std::cout << "ID: " << obj.getId() << std::endl; // ID: 42
    return 0;
}
```

#### 类中的 const 成员函数

函数后加 const，表示不修改对象状态。

特点：

- 无 this 修改权限。
- 只能调用其他 const 成员函数。

示例：

```c++
#include <iostream>

class MyClass {
    int value;
public:
    MyClass(int v) : value(v) {}
    int getValue() const { // 不会修改对象
        // value = 100; // 错误
        return value;
    }
    void setValue(int v) { // 非 const 函数
        value = v;
    }
};

int main() {
    const MyClass obj(10); // const 对象
    std::cout << "Value: " << obj.getValue() << std::endl; // Value: 10
    // obj.setValue(20); // 错误：const 对象只能调用 const 函数
    return 0;
}
```

## 4、虚函数、纯虚函数？

#### 虚函数

在基类中用 `virtual` 关键字声明的成员函数称为虚函数。当通过基类指针或引用调用该函数时，实际调用的是派生类中重写的版本，而不是静态绑定到基类的函数版本。这就是所谓的“动态绑定”或“运行时多态”。

##### 实现原理

虚函数表（vtable）：
每个包含虚函数的类在编译时都会生成一个虚函数表（vtable），表中存储着该类所有虚函数的地址。每个对象通常包含一个指向该类 vtable 的指针（称为 vptr），从而在运行时根据对象的实际类型找到正确的函数实现。

动态绑定：
当调用虚函数时，编译器会生成代码，通过对象的 vptr 查找 vtable，然后调用相应的函数指针，从而实现动态绑定。这个过程的额外开销通常是一次指针间接访问，相对于函数调用来说开销较小。

##### 示例

```c++
#include <iostream>

class Base {
public:
    virtual void show() { // 虚函数
        std::cout << "Base show" << std::endl;
    }
};

class Derived : public Base {
public:
    void show() override { // 重写
        std::cout << "Derived show" << std::endl;
    }
};

int main() {
    Base* ptr;
    Base base;
    Derived derived;

    ptr = &base;
    ptr->show(); // Base show

    ptr = &derived;
    ptr->show(); // Derived show（多态）

    return 0;
}
```

#### 纯虚函数

纯虚函数是一种在基类中声明但没有实现的虚函数，其声明形式是在函数原型后加上 `= 0`。如 `virtual void func() = 0;`。

##### 实现原理

抽象类：
包含纯虚函数的类不能被实例化，它仅提供一个接口。编译器在构造抽象类时会生成虚函数表，但其中对应纯虚函数的入口通常指向一个特殊标记（或留空），表明该函数没有实现。

派生类实现：
派生类必须重写所有纯虚函数，否则派生类仍然是抽象类。这样可以确保在运行时调用时总能找到有效的函数实现。

##### 示例

```c++
#include <iostream>

class Abstract {
public:
    virtual void show() = 0; // 纯虚函数
    virtual ~Abstract() = default; // 虚析构函数
};

class Concrete : public Abstract {
public:
    void show() override {
        std::cout << "Concrete show" << std::endl;
    }
};

int main() {
    // Abstract obj; // 错误：抽象类无法实例化
    Concrete concrete;
    Abstract* ptr = &concrete;
    ptr->show(); // Concrete show
    return 0;
}
```

#### 区别对比

| 特性        | 虚函数                 | 纯虚函数                 |
| ----------- | ---------------------- | ------------------------ |
| 语法        | virtual void func();   | virtual void func() = 0; |
| 默认实现    | 有（可调用）           | 无（必须重写）           |
| 类类型      | 可实例化               | 抽象类（不可实例化）     |
| 用途        | 提供默认行为，支持多态 | 定义接口，强制实现       |
| 重写要求    | 可选                   | 必须                     |
| vtable 条目 | 指向基类或子类实现     | 占位，子类填充实现       |

## 5、内存管理

C++ 程序的内存分为以下几个区域：

- 栈（Stack）：
  - 存储局部变量和函数调用信息。
  - 特点：自动分配和释放，生命周期随作用域。
  - 大小有限（通常几 MB，由编译器或系统设置）。
- 堆（Heap）：
  - 存储动态分配的内存。
  - 特点：手动管理，生命周期由程序员控制。
  - 大小受系统可用内存限制。
- 全局/静态区（Data Segment）：
  - 存储全局变量和静态变量。
  - 分为初始化区（.data）和未初始化区（.bss）。
  - 生命周期为程序运行期间。
- 代码区（Text Segment）：
  - 存储编译后的机器指令。
  - 只读，防止篡改。
- 常量区：存储字符串字面量和 const 数据（如 .rodata）。

## 6、4种智能指针?

#### `std::auto_ptr`（已弃用）

C++98 标准中引入的智能指针，用于在对象生命周期结束时自动释放内存。然而，由于其所有权语义不明确，可能导致意外的内存释放问题，因此在 C++11 中被标记为弃用，并在 C++17 中被完全移除。

示例：

```c++
#include <iostream>
#include <memory>

int main() {
    std::auto_ptr<int> ptr1(new int(10));
    std::auto_ptr<int> ptr2 = ptr1; // ptr1 变空
    std::cout << (ptr1.get() == nullptr ? "ptr1 null" : "ptr1 valid") << std::endl;
    std::cout << *ptr2 << std::endl;
    return 0;
}
```

输出：

```c++
ptr1 null
10
```

#### `std::unique_ptr`

一种独占所有权的智能指针，确保同一时间内只有一个指针拥有某块资源。它不允许复制，但可以通过 `std::move` 转移所有权。

示例：

```c++
#include <iostream>
#include <memory>

struct MyClass {
    int x;
    MyClass(int val) : x(val) { std::cout << "Constructed\n"; }
    ~MyClass() { std::cout << "Destructed\n"; }
};

int main() {
    std::unique_ptr<MyClass> ptr(new MyClass(10));
    std::cout << "Value: " << ptr->x << std::endl;

    // std::unique_ptr<MyClass> ptr2 = ptr; // 错误，不可复制
    std::unique_ptr<MyClass> ptr2 = std::move(ptr); // 移动所有权
    std::cout << "Moved, ptr is " << (ptr ? "valid" : "null") << std::endl;

    return0; // 自动析构
}
```

输出：

```c++
Constructed
Value: 10
Moved, ptr is null
Destructed
```

#### `std::shared_ptr`

允许多个指针共享同一资源，并通过引用计数来管理资源的生命周期。当最后一个指向该资源的 `std::shared_ptr` 被销毁时，资源会被自动释放。

特性：

- 共享所有权： 多个 `std::shared_ptr` 可以指向同一资源。
- 引用计数： 内部维护一个引用计数，记录有多少个 `std::shared_ptr` 指向同一资源。

示例：

```c++
#include <iostream>
#include <memory>

struct MyClass {
    int x;
    MyClass(int val) : x(val) { std::cout << "Constructed\n"; }
    ~MyClass() { std::cout << "Destructed\n"; }
};

int main() {
    std::shared_ptr<MyClass> ptr1 = std::make_shared<MyClass>(20);
    {
        std::shared_ptr<MyClass> ptr2 = ptr1; // 共享
        std::cout << "Use count: " << ptr1.use_count() << std::endl; // 2
    }
    std::cout << "Use count: " << ptr1.use_count() << std::endl; // 1
    return0; // 析构
}
```

输出：

```c++
Constructed
Use count: 2
Use count: 1
Destructed
```

#### `std::weak_ptr`

一种不控制资源生命周期的智能指针，专门用于解决 `std::shared_ptr` 可能出现的循环引用问题。它是对资源的弱引用，不影响引用计数。

特性：

- 不共享所有权： 不增加引用计数，不影响资源的生命周期。
- 用于打破循环引用： 常用于避免 `std::shared_ptr` 之间的循环引用导致的内存泄漏。

示例：

```c++
#include <iostream>
#include <memory>

struct Node {
    std::shared_ptr<Node> next;
    std::weak_ptr<Node> prev; // 避免循环引用
    ~Node() { std::cout << "Destructed\n"; }
};

int main() {
    std::shared_ptr<Node> n1 = std::make_shared<Node>();
    std::shared_ptr<Node> n2 = std::make_shared<Node>();

    n1->next = n2;
    n2->prev = n1; // 弱引用
    return0; // 正常析构
}
```

输出：

```c++
Destructed
Destructed
```

若 prev 用 shared_ptr，引用计数永不为 0，造成泄漏。

## 7、堆、栈区别？

#### 内存分配方式

##### 栈（Stack）

自动分配：栈内存由编译器自动管理，当函数被调用时，局部变量、函数参数和返回地址等数据会自动在栈上分配；函数返回时，这部分内存会自动释放。

分配速度快：分配和释放内存操作仅仅是调整栈指针（SP）的值，速度非常快。

##### 堆（Heap）

动态分配：堆内存由程序员通过 `new`、`malloc` 等显式分配，使用完后需要手动释放（或者使用智能指针等 RAII 技术自动释放）。

分配速度相对较慢：分配和释放堆内存涉及复杂的内存管理算法（如内存池、空闲链表管理等），因此速度比栈慢，而且可能会产生内存碎片。

#### 内存大小和生命周期

##### 栈

大小限制：栈内存通常较小（几百 KB 到几 MB），受限于操作系统和编译器的配置，因此不能存储大量数据。

生命周期短：栈内存中的数据在函数调用结束后自动销毁，不适合存储需要跨函数调用、长期存在的数据。

##### 堆

大小较大：堆内存的大小通常仅受限于系统的物理内存和虚拟内存，可以动态申请大块内存（例如存储大型数据结构或文件内容）。

生命周期灵活：堆内存中分配的内存块的生命周期由程序员控制，可以在需要时长期保留，直到显式释放（或由智能指针自动释放）。

#### 存储管理和访问方式

##### 栈

内存结构：栈是一块连续的内存区域，采用 LIFO（后进先出）结构，这也使得函数调用和返回具有天然的层次性。

局部性好：由于栈内存存储的数据通常是局部变量，它们在函数调用期间被频繁访问，有助于提高 CPU 缓存命中率。

##### 堆

内存结构：堆内存分布在一个较大的内存区域中，分配的内存块可能是不连续的。操作系统需要维护堆内存的空闲和已分配区域，可能会产生内存碎片。

访问效率：堆内存数据的访问速度可能受到内存碎片和分配方式的影响，而且由于内存不连续，缓存局部性可能不如栈内存。

## 8、Linux命令

Linux命令全集

## 9、进程和线程

#### 基本定义

##### 进程

进程是操作系统分配资源的基本单位，是一个正在运行的程序实例。每个进程都有自己独立的地址空间、内存、文件描述符和系统资源。

隔离性：进程之间相互独立，彼此的内存空间是隔离的，一个进程的错误通常不会直接影响其他进程，这种隔离提供了较高的安全性和稳定性。

##### 线程

线程是进程内部的执行单元，一个进程可以包含多个线程，共享进程的内存和资源。线程是操作系统调度和执行的最小单位。

共享性：同一进程内的线程共享全局数据、堆、打开的文件等资源，但每个线程有自己的寄存器状态和栈空间，负责执行独立的任务。

#### 资源分配与开销

##### 进程

每个进程都有独立的虚拟地址空间和资源，因此进程之间的切换需要保存和恢复大量上下文信息，开销相对较大。

由于资源独立，创建新进程时需要复制或分配新的资源（在使用写时复制技术时开销较小），但整体上占用的内存和系统资源较多。

##### 线程

同一进程内的线程共享大部分资源，因此线程切换时只需保存和恢复较少的上下文（例如寄存器、栈指针等），开销较小。

线程是轻量级的执行单元，创建和销毁线程的成本远低于进程，适合高并发场景。

#### 调度与上下文切换

##### 进程

操作系统以进程为基本调度单位。进程切换涉及整个进程的状态（地址空间、资源等）的保存与恢复，时间和资源开销较高。

由于进程间完全隔离，可以避免因为共享数据带来的竞态条件，但进程间通信（IPC）如管道、共享内存、消息队列等相对复杂且效率较低。

##### 线程

线程是操作系统调度的基本单位，同一进程的线程之间切换速度快，因为它们共享大部分上下文。

线程之间可以直接通过共享内存进行通信，无需像进程间通信那样进行数据拷贝，因此效率更高，但这也要求在多线程程序中注意同步和互斥问题，防止数据竞争和死锁。

## 10、七层模型和四层模型？

#### OSI 七层模型

##### 物理层

负责传输原始比特流，是数据通信中最底层的一层。定义了物理介质的电气、机械、过程和功能特性。

作用：将数据转换为电信号、光信号或无线信号。

常见技术：光纤、电缆、无线传输、网卡、调制解调器等。

##### 数据链路层

提供节点间可靠的数据传输，负责在物理链路上对数据帧的封装、传输、差错检测与纠正。它将物理层传输的原始比特流组装成有意义的帧，并提供物理地址（MAC 地址）。

作用：帧同步、流量控制、错误校验（如 CRC）。

常见协议：Ethernet、PPP、HDLC、Wi-Fi 等。

##### 网络层

负责数据包在不同网络间的传输与路由选择，实现逻辑地址（IP 地址）寻址。该层确定数据从源节点到目标节点的最佳路径。

作用：路径选择、分片与重组、地址解析。

常见协议：IP（IPv4/IPv6）、ICMP、IGMP、路由协议（如 OSPF、BGP 等）。

##### 传输层

提供端到端的数据传输服务，确保数据的完整性和顺序。负责分段、重组、差错校验和流量控制。传输层对上层应用提供透明的通信服务。

作用：流量控制、错误恢复、连接管理。

常见协议： TCP（传输控制协议）和 UDP（用户数据报协议）。

##### 会话层

负责建立、管理和终止应用程序之间的会话，提供对话控制和同步机制，确保在通信过程中数据的连续性和状态管理。

作用：会话恢复、同步、对话控制。

常见应用： 会话层的功能通常由一些高级协议和 API 实现，比如 RPC、SQL 连接等。现代网络中，这一层的功能有时由传输层或应用层承担。

##### 表示层

负责数据的语法和语义转换，提供数据加密、解密、压缩和解压缩功能。确保发送方和接收方能够正确理解彼此传输的数据。

作用：数据加密/解密、压缩、格式转换（如 XML 到 JSON）。

常见应用： 数据格式转换（如 ASCII 与 EBCDIC）、加密协议（如 SSL/TLS 的一部分功能）、文件格式标准等。

##### 应用层

提供网络服务和应用接口，直接为用户应用程序提供支持。包括数据展示、用户身份验证、资源请求等。

作用：数据交互、用户接口、应用程序协议。

常见协议： HTTP、FTP、SMTP、DNS、Telnet、SSH 等。

#### TCP/IP 四层模型

##### 链路层

负责在物理网络上发送数据帧，涵盖了 OSI 模型中的物理层和数据链路层的功能。

设备： 网卡、交换机、局域网协议（如以太网、Wi-Fi）。

##### 网际层

负责在不同网络之间传输数据包，定义了逻辑地址（IP 地址）和路由选择。

协议： IP 协议（IPv4、IPv6）、ICMP、IGMP 等。

设备： 路由器。

##### 传输层

负责端到端数据传输，提供可靠（TCP）和不可靠（UDP）的数据传输服务。

协议： TCP、UDP。

##### 应用层

提供应用程序之间的数据通信接口，直接为用户应用提供服务。

协议： HTTP、FTP、SMTP、DNS 等。

## 11、四次挥手的两种wait？

这里先讲一下四次挥手的流程：

第一次挥手（FIN）：主动关闭方发送一个 FIN 报文，表示它已经没有数据发送了，但仍能接收数据。

第二次挥手（ACK）：被动关闭方收到 FIN 后，发送一个 ACK，确认收到 FIN。主动关闭方此时进入 **FIN_WAIT_2** 状态。

第三次挥手（FIN）：被动关闭方处理完数据后，也发送一个 FIN，表示它也没有数据发送了。

第四次挥手（ACK）：主动关闭方收到被动关闭方的 FIN 后，发送 ACK，确认收到对方的终止请求，然后进入 **TIME_WAIT** 状态一段时间，确保对方收到了最后的 ACK 后再彻底关闭连接。

#### FIN_WAIT_2

##### 出现时机

当主动关闭方发送完 FIN（第一次挥手）并收到被动关闭方对该 FIN 的 ACK（第二次挥手）后，就进入 FIN_WAIT_2 状态。

##### 作用与意义

此时主动关闭方已经完成自己发送数据的部分，等待被动关闭方发送 FIN 表示其也完成数据发送。

在 FIN_WAIT_2 状态中，主动关闭方保持半关闭状态（只发送不接收的关闭），等待对方终止连接。如果被动关闭方延迟发送 FIN，则连接会长时间停留在 FIN_WAIT_2 状态，直到超时或 FIN 到达。

#### TIME_WAIT

##### 出现时机

当主动关闭方收到被动关闭方的 FIN（第三次挥手）后，发送最后的 ACK（第四次挥手）并进入 TIME_WAIT 状态。

##### 作用与意义

确保最后的 ACK 能被对方接收：如果最后的 ACK 丢失，被动关闭方可能会重传 FIN，主动关闭方在 TIME_WAIT 状态下仍能重发 ACK。

防止旧数据干扰新连接：TIME_WAIT 状态通常持续 2MSL（Maximum Segment Lifetime，最大报文段生存时间）的时间，这个等待期可以确保所有网络中延迟传输的旧报文都被丢弃，从而避免这些数据影响后续新连接。

保证协议的正确性：TIME_WAIT 还保证了 TCP 四次挥手的可靠性，确保连接的双方都对对方的关闭行为有充分的确认。

## 12、最近公共祖先节点

#### 解题思路

利用递归遍历二叉树，判断当前节点是否为 p 或 q 或者在左右子树中分别找到 p 和 q，从而确定最近公共祖先。

递归过程：

1. 终止条件：
   - 如果当前节点为空，返回空指针。
   - 如果当前节点等于 p 或 q，则直接返回当前节点，因为一个节点可以是它自己的祖先。
2. 递归调用：
   - 递归遍历当前节点的左子树，记为 left。
   - 递归遍历当前节点的右子树，记为 right。
3. 合并结果：
   - 如果 left 和 right 均不为空，说明 p 和 q 分别位于当前节点的左右子树中，因此当前节点即为最近公共祖先，返回当前节点。
   - 如果 left 为空，说明 p、q 都在右子树中，返回 right。
   - 如果 right 为空，说明 p、q 都在左子树中，返回 left。

#### 参考代码（C++）

```c++
#include <iostream>

// 定义二叉树节点结构体
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

// 递归函数，返回当前子树中 p 和 q 的最近公共祖先
TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
    // 终止条件：若当前节点为空或等于 p 或 q，则直接返回当前节点
    if (root == nullptr || root == p || root == q) {
        return root;
    }
    
    // 递归遍历左子树和右子树
    TreeNode* left = lowestCommonAncestor(root->left, p, q);
    TreeNode* right = lowestCommonAncestor(root->right, p, q);
    
    // 如果左右子树均不为空，说明 p 和 q 分别在左右子树中，因此 root 为最近公共祖先
    if (left != nullptr && right != nullptr) {
        return root;
    }
    
    // 否则，返回非空的那一侧
    return (left != nullptr) ? left : right;
}

// 测试函数
int main() {
    // 构造示例二叉树
    //         3
    //        / \
    //       5   1
    //      / \ / \
    //     6  2 0  8
    //       / \
    //      7   4
    TreeNode* root = new TreeNode(3);
    root->left = new TreeNode(5);
    root->right = new TreeNode(1);
    root->left->left = new TreeNode(6);
    root->left->right = new TreeNode(2);
    root->right->left = new TreeNode(0);
    root->right->right = new TreeNode(8);
    root->left->right->left = new TreeNode(7);
    root->left->right->right = new TreeNode(4);
    
    // 假设要查找节点 5 和节点 1 的最近公共祖先，结果应为 3
    TreeNode* p = root->left;   // 节点 5
    TreeNode* q = root->right;  // 节点 1
    
    TreeNode* lca = lowestCommonAncestor(root, p, q);
    
    if (lca) {
        std::cout << "最近公共祖先的值为: " << lca->val << std::endl;
    } else {
        std::cout << "没有找到公共祖先。" << std::endl;
    }
    
    // 此处为了简化示例，没有释放所有动态分配的内存
    return 0;
}
```

## 13、LRU

#### 解题思路

双向链表：

- 用于记录缓存数据的使用顺序。链表头部表示最近使用的元素，尾部表示最久未使用的元素。
- 每次访问或插入数据时，将对应节点移动到链表头部。
- 当缓存容量超出限制时，移除链表尾部的节点，即最久未使用的元素。

哈希表：

- 存储键值对，其中键为缓存数据的键，值为指向双向链表中对应节点的指针。
- 通过哈希表可以在 O(1) 时间内找到对应的链表节点，方便进行插入、删除和移动操作。

#### 参考代码（C++）

```c++
#include <unordered_map>

// 双向链表节点结构
struct DLinkedNode {
    int key;
    int value;
    DLinkedNode* prev;
    DLinkedNode* next;
    DLinkedNode() : key(0), value(0), prev(nullptr), next(nullptr) {}
    DLinkedNode(int _key, int _value) : key(_key), value(_value), prev(nullptr), next(nullptr) {}
};

class LRUCache {
private:
    std::unordered_map<int, DLinkedNode*> cache; // 哈希表
    DLinkedNode* head; // 虚拟头节点
    DLinkedNode* tail; // 虚拟尾节点
    int size;
    int capacity;

    // 添加节点到头部
    void addToHead(DLinkedNode* node) {
        node->prev = head;
        node->next = head->next;
        head->next->prev = node;
        head->next = node;
    }

    // 移除节点
    void removeNode(DLinkedNode* node) {
        node->prev->next = node->next;
        node->next->prev = node->prev;
    }

    // 移动节点到头部
    void moveToHead(DLinkedNode* node) {
        removeNode(node);
        addToHead(node);
    }

    // 移除尾部节点
    DLinkedNode* removeTail() {
        DLinkedNode* node = tail->prev;
        removeNode(node);
        return node;
    }

public:
    // 构造函数
    LRUCache(int _capacity) : capacity(_capacity), size(0) {
        // 使用虚拟头尾节点简化操作
        head = new DLinkedNode();
        tail = new DLinkedNode();
        head->next = tail;
        tail->prev = head;
    }

    // 获取值
    int get(int key) {
        if (!cache.count(key)) {
            return -1;
        }
        DLinkedNode* node = cache[key];
        moveToHead(node);
        return node->value;
    }

    // 插入或更新值
    void put(int key, int value) {
        if (cache.count(key)) {
            DLinkedNode* node = cache[key];
            node->value = value;
            moveToHead(node);
        } else {
            DLinkedNode* node = new DLinkedNode(key, value);
            cache[key] = node;
            addToHead(node);
            ++size;
            if (size > capacity) {
                DLinkedNode* removed = removeTail();
                cache.erase(removed->key);
                delete removed;
                --size;
            }
        }
    }

    // 析构函数，释放内存
    ~LRUCache() {
        DLinkedNode* current = head;
        while (current != nullptr) {
            DLinkedNode* next = current->next;
            delete current;
            current = next;
        }
    }
};
```

