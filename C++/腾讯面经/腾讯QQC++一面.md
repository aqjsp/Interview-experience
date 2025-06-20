# 腾讯QQ--C++一面面经

> 来源：https://www.nowcoder.com/discuss/734051246809653248

## 1、C++多态有哪些？

#### 静态多态（编译时多态）

静态多态是在编译期间确定函数调用的绑定方式，主要通过函数重载和模板实现。

- 函数重载： 在同一作用域内，可以定义多个同名函数，但它们的参数列表（参数个数、类型、顺序）不同。编译器根据函数调用时传入的参数类型和数量来确定调用哪个函数。

  ```c++
    #include <iostream>
  
    class Printer {
    public:
        void print(int i) {
            std::cout << "Printing integer: " << i << std::endl;
        }
  
        void print(double d) {
            std::cout << "Printing double: " << d << std::endl;
        }
    };
  
    int main() {
        Printer p;
        p.print(5);     // 调用 print(int)
        p.print(3.14);  // 调用 print(double)
        return 0;
    }
  ```

- 函数模板： 通过模板，可以编写泛型函数，使其能够处理多种数据类型。编译器在编译时根据传入的参数类型生成相应的函数实例。

  ```c++
    #include <iostream>
  
    template <typename T>
    void print(T t) {
        std::cout << "Printing: " << t << std::endl;
    }
  
    int main() {
        print(5);     // 打印整数
        print(3.14);  // 打印浮点数
        print("Hello"); // 打印字符串
        return 0;
    }
  ```

#### 动态多态（运行时多态）

动态多态是在运行期间确定函数调用的绑定方式，主要通过虚函数和继承实现。

虚函数： 在基类中使用 `virtual` 关键字声明的成员函数称为虚函数。派生类可以重写这些虚函数。当通过基类指针或引用调用虚函数时，程序会在运行时根据对象的实际类型来决定调用哪个派生类的重写函数。

```c++
  #include <iostream>

  class Animal {
  public:
      virtual void makeSound() {
          std::cout << "Animal makes sound" << std::endl;
      }
  };

  class Dog : public Animal {
  public:
      void makeSound() override {
          std::cout << "Dog barks" << std::endl;
      }
  };

  int main() {
      Animal* animal = new Dog();
      animal->makeSound(); // 输出: Dog barks
      delete animal;
      return 0;
  }
```

## 2、C++异常处理？

C++ 中的异常处理是一种机制，用于处理程序运行时发生的错误或意外情况（如文件打开失败、内存分配不足等）。它通过将错误检测和错误处理分离，提高代码的健壮性和可读性。

#### 基本语法与工作流程

##### 语法

```c++
try {
    // 可能抛出异常的代码
} catch (异常类型 参数) {
    // 处理特定异常
} catch (...) {
    // 捕获所有未处理的异常
}
```

##### 工作流程

1. 正常执行：程序进入 try 块运行代码。
2. 抛出异常：如果 throw 被调用，程序跳出当前作用域，沿调用栈向上寻找匹配的 catch。
3. 捕获异常：找到匹配的 catch 块，执行处理逻辑。
4. 继续执行：处理完异常后，程序从 catch 块后继续运行。

#### 异常处理的关键元素

##### throw

抛出异常对象，通知程序发生错误。

语法：`throw 表达式`;

类型：可以是任意类型（基本类型、类对象、指针等），但推荐使用标准异常类。

##### try

包裹可能抛出异常的代码块。

特点：必须与至少一个 catch 配对。

##### catch

捕获并处理特定类型的异常。

匹配规则：

- 按顺序匹配，类型必须兼容（支持基类捕获派生类）。
- catch (...) 捕获所有异常，作为最后手段。

## 3、C++堆栈大小？

#### 栈（Stack）

栈是一种后进先出（LIFO）的数据结构，用于存储函数调用信息和局部变量（自动变量）。

特点：

- 由编译器自动管理，分配和释放无需程序员干预。
- 生命周期与作用域绑定，函数结束时自动清理。

存储内容：

- 局部变量（如 int x;）。
- 函数调用帧（返回地址、参数、寄存器状态）。

#### 堆（Heap）

堆是一个动态内存区域，用于存储程序运行时手动分配的对象。

特点：

- 由程序员通过 new 或 malloc 分配，通过 delete 或 free 释放。
- 生命周期由程序控制，不受作用域限制。

存储内容：动态对象（如 new int）。

#### 堆和栈的大小限制

##### 栈的大小

默认值：

- Linux：通常 8MB（可以通过 ulimit -s 查看，默认 8192 KB）。
- Windows：Visual Studio 默认 1MB。

特点：

- 栈大小是每个线程独立的，线程创建时分配固定栈空间。
- 栈溢出（Stack Overflow）发生在栈空间不足时（如递归过深）。

影响因素：

- 局部变量大小。
- 函数调用深度。

查看方法：

- Linux：ulimit -s。
- Windows：编译器链接选项（/STACK:size）。

##### 堆的大小

默认值：

- 理论上受可用虚拟内存限制（32 位系统约 2-4GB，64 位系统更大）。
- 实际受物理内存和操作系统限制。

特点：

- 堆是进程共享的，所有动态分配共享同一堆空间。
- 大小动态增长，直到内存耗尽（抛出 std::bad_alloc）。

影响因素：

- 系统可用内存。
- 程序分配行为。

查看方法：运行时通过工具（如 top、Task Manager）观察进程内存使用。

#### 堆栈对比

| 特性     | 栈 (Stack)         | 堆 (Heap)          |
| -------- | ------------------ | ------------------ |
| 分配方式 | 自动（编译器）     | 手动（new/delete） |
| 大小     | 固定（默认 1-8MB） | 动态（受内存限制） |
| 生命周期 | 作用域结束自动释放 | 手动释放           |
| 速度     | 快（连续内存）     | 慢（动态分配）     |
| 溢出风险 | 栈溢出             | 内存耗尽           |

## 4、用户态和内核态的区别？

#### 定义

##### 用户态

用户态是 CPU 执行用户程序（如应用程序）的模式，具有较低的特权级别。

特点：

- 受限访问：无法直接操作硬件或核心系统资源。
- 隔离性：每个用户进程运行在独立的虚拟地址空间中。

保护操作系统，防止用户程序误操作或恶意破坏。

##### 内核态

内核态是 CPU 执行操作系统内核代码的模式，具有最高特权级别。

特点：

- 完全访问：可以直接操作硬件（如 CPU、内存、I/O 设备）和所有系统资源。
- 无限制：运行内核代码（如驱动、调度器）。

提供核心服务，管理硬件和进程。

#### 工作原理

##### 用户态

执行环境：

- 用户程序运行在虚拟内存中，每个进程有独立的地址空间。
- 通过页表映射到物理内存，受操作系统保护。

限制：

- 不能直接执行特权指令（如 I/O 操作、修改页表）。
- 需要通过系统调用请求内核服务。

##### 内核态

执行环境：

- 内核代码运行在统一的内核地址空间，直接访问物理内存。
- 可执行所有指令，包括特权指令。

功能：管理进程、线程、文件系统、设备驱动等。

#### 主要区别

| 特性     | 用户态                 | 内核态                 |
| -------- | ---------------------- | ---------------------- |
| 特权级别 | 低（Ring 3）           | 高（Ring 0）           |
| 权限     | 受限，不能直接访问硬件 | 无限制，可操作所有资源 |
| 运行内容 | 用户应用程序           | 操作系统内核代码       |
| 地址空间 | 虚拟地址，进程隔离     | 物理地址或内核地址空间 |
| 安全性   | 高，防止非法操作       | 低，需内核自身保证     |
| 中断处理 | 不可处理               | 可处理硬件中断         |
| 切换开销 | 通过系统调用进入内核态 | 可直接返回用户态       |

## 5、系统调用和函数调用的开销？

#### 函数调用

函数调用是程序在同一特权级别（通常是用户态）内，从一个函数跳转到另一个函数的执行过程。

在用户空间内完成，无需操作系统干预。

通常由编译器生成直接跳转指令（如 call）。

#### 系统调用

系统调用是用户程序请求操作系统内核服务的过程，涉及从用户态切换到内核态。

需要特权级别切换（User Mode → Kernel Mode）。

通过特定指令（如 int、syscall）触发。

#### 开销对比

| 特性       | 函数调用               | 系统调用                 |
| ---------- | ---------------------- | ------------------------ |
| 特权级别   | 无切换（用户态）       | 切换（用户态 ↔ 内核态）  |
| 上下文切换 | 轻量（栈和少量寄存器） | 重量（完整上下文）       |
| 跳转机制   | 直接跳转（call/ret）   | 中断/特权指令（syscall） |
| 执行时间   | 几纳秒                 | 几微秒                   |
| 开销大小   | 低（10-50 周期）       | 高（500-5000 周期）      |
| 依赖       | CPU 和编译器           | 操作系统和硬件           |

## 6、如何理解一切皆文件？

“一切皆文件”是 UNIX 及其衍生操作系统（如 Linux）设计哲学中的核心理念。它指的是系统中许多资源（包括硬件设备、网络连接、进程间通信等）都被抽象为文件，并通过统一的文件操作接口（如 open、read、write、close）进行管理。

#### 核心思想

在 UNIX/Linux 中，文件不仅仅是存储在磁盘上的数据，而是对所有资源的一种抽象表示。

无论是普通文件、目录、设备（键盘、磁盘）、管道、网络套接字，甚至某些系统信息，都可以通过文件系统的接口访问。

#### 实现机制

##### 文件描述符

文件描述符是一个非负整数，标识进程打开的某个资源。

内核通过文件描述符表将用户程序的操作映射到具体的资源。

##### 文件系统抽象

层次结构：

- 用户态：通过标准 C/C++ 库（如 fopen）或系统调用（如 open）访问。
- 内核态：VFS（Virtual File System，虚拟文件系统）统一管理。

VFS 的作用：

- 提供统一的接口，屏蔽底层差异（ext4、NFS、设备驱动）。
- 将不同资源映射到文件操作（如 struct file_operations）。

## 7、内存泄露是什么，如何查找，如何避免？

内存泄漏是指程序在堆上动态分配的内存（通过 new 或 malloc），在不再需要时未能释放（通过 delete 或 free），且失去了对这块内存的引用，导致它无法被程序或操作系统回收。

后果：

- 内存占用逐渐增加。
- 性能下降，最终可能导致程序崩溃（内存耗尽）。

```c++
void leak() {
    int* ptr = new int(10); // 分配内存
    // 未 delete ptr，函数退出后 ptr 丢失
}
```

#### 成因

##### 忘记释放内存

分配内存后未调用 delete 或 free。

```c++
int* ptr = new int[100];
// 缺少 delete[] ptr;
```

##### 指针丢失

对动态内存的指针被重新赋值或超出作用域，导致无法访问原内存。

```c++
int* ptr = new int(5);
ptr = nullptr; // 原内存无法访问
```

##### 异常路径未释放

异常抛出时，内存未在适当位置释放。

```c++
void risky() {
    int* ptr = new int(10);
    throw std::runtime_error("Error"); // 未 delete ptr
}
```

##### 循环引用

对象间相互引用（如智能指针循环），阻止内存释放。

```c++
struct Node {
    std::shared_ptr<Node> next;
};
auto n1 = std::make_shared<Node>();
auto n2 = std::make_shared<Node>();
n1->next = n2;
n2->next = n1; // 循环引用
```

#### 如何查找

##### 手动检查

审查代码，确保每 new 有对应的 delete，每 malloc 有对应的 free。

##### 静态分析工具

- Cppcheck：检测未释放的内存。
- Clang Static Analyzer：分析潜在泄漏。

##### 动态分析工具

Valgrind（Linux）：使用 memcheck 检查泄漏。

```bash
valgrind --leak-check=full ./xxxxx
```

输出：显示泄漏的内存块及其分配点。

#### 如何避免

##### 使用 RAII

资源获取即初始化（Resource Acquisition Is Initialization），通过对象生命周期自动管理资源。

```c++
#include <memory>

void safe() {
    auto ptr = std::unique_ptr<int[]>(new int[10]);
    // 函数结束时自动释放
}
```

##### 异常安全

在异常路径中使用 RAII 确保释放。

##### 避免手动管理

使用容器替代裸指针。

```c++
#include <vector>

std::vector<int> vec(100); // 替代 int* ptr = new int[100]
```

##### 检查返回值

确保分配成功并正确释放。

```c++
int* ptr = new (std::nothrow) int[100];
if (ptr) {
    // 使用 ptr
    delete[] ptr;
}
```

##### 解决循环引用

使用 std::weak_ptr 打破循环。

```c++
struct Node {
    std::shared_ptr<Node> next;
    std::weak_ptr<Node> prev; // 避免循环
};
```

## 8、智能指针的实现？

智能指针

## 9、map为什么用红黑树，好处？

#### 为什么选择红黑树？

std::map 是一个关联容器，要求以下特性：

- 键值对存储：支持键（key）和值（value）的映射。
- 有序性：键按升序（或自定义顺序）排列。
- 高效操作：插入、删除、查找的时间复杂度应稳定。
- 动态调整：支持频繁的动态插入和删除。

红黑树是一种自平衡二叉搜索树，能够很好地满足这些需求。相比其他数据结构（如普通二叉搜索树、AVL 树、哈希表），红黑树在性能、平衡性和实现复杂度之间取得了平衡。

#### 红黑树的性质

1. 节点颜色：每个节点是红色或黑色。
2. 根节点：根节点是黑色。
3. 叶子节点（NIL）：所有空子节点（NIL）是黑色。
4. 红色规则：红色节点的子节点必须是黑色（不能有连续红色节点）。
5. 黑色高度：从任意节点到其所有叶子节点的路径上，黑色节点数量相同（黑色高度相等）。

##### 平衡性

- 这些性质保证红黑树的高度不超过 2 * log₂(n + 1)，其中 n 是节点数。
- 操作时间复杂度稳定在 O(log n)。

#### 好处

##### 时间复杂度稳定（O(log n)）

查找：按键值二分搜索，O(log n)。

插入：插入后通过局部调整（旋转和着色），O(log n)。

删除：删除后调整平衡，O(log n)。

对比：

- 普通 BST：最坏情况退化为 O(n)（链表）。
- 红黑树：始终保持近似平衡，避免退化。

##### 有序性

红黑树是 BST，天然支持中序遍历输出有序序列。

好处：

- std::map 的迭代器按键升序访问。
- 支持范围查询（如 lower_bound、upper_bound）。

##### 自平衡能力

调整开销低：

- 插入和删除只需 O(1) 次旋转和着色（最多 2-3 次）。
- 对比 AVL 树：AVL 更严格平衡（高度差 ≤ 1），调整更频繁。

好处：

- 动态操作（插入/删除）性能稳定。
- 避免普通 BST 的退化问题。

## 10、protocol buffer？

Protocol Buffers（简称 Protobuf）是 Google 开发的一种高效、语言无关、平台无关的数据序列化格式，用于将结构化数据编码为紧凑的二进制形式，以便在网络传输或存储时使用。它类似于 XML 或 JSON，但设计目标是更小、更快、更简单。

#### 工作原理

##### 定义数据结构

使用 .proto 文件定义消息结构，包含字段名、类型和编号。

```protobuf
syntax = "proto3"; // 使用 proto3 语法
message Person {
    string name = 1;      // 字段编号 1
    int32 age = 2;        // 字段编号 2
    repeated string emails = 3; // 可重复字段（数组）
}
```

##### 编译生成代码

使用 protoc 编译器将 .proto 文件生成目标语言的代码。

```bash
protoc --cpp_out=. person.proto
```

输出：

- person.pb.h：头文件，定义类。
- person.pb.cc：实现文件。

##### 序列化与反序列化

序列化：将对象转为二进制字节流。

反序列化：从字节流还原对象。

Protobuf 使用紧凑的变长编码和字段编号，减少冗余。

## 11、SQL注入？

SQL 注入是一种常见的网络安全漏洞，攻击者通过向应用程序的 SQL 查询中注入恶意代码，操控数据库执行非预期操作，从而窃取数据、破坏数据库或绕过身份验证。

#### SQL 注入的原理

#####  漏洞来源

动态拼接 SQL：应用程序将用户输入直接拼接到 SQL 语句。

```c++
std::string username = "admin";
std::string password = "' OR '1'='1";
std::string query = "SELECT * FROM users WHERE username = '" + username + "' AND password = '" + password + "'";
// 执行 query，结果被操控
```

未过滤输入：未检查或转义特殊字符（如 '、--、;）。

##### 执行过程

1. 用户提交输入（合法或恶意）。
2. 应用程序将输入拼接到 SQL。
3. 数据库解析并执行完整 SQL。
4. 攻击者利用结果实现目的。

#### SQL 注入的类型

##### 经典 SQL 注入

通过应用程序返回的数据直接获取结果。

```sql
输入：1; DROP TABLE users; --
查询：SELECT * FROM orders WHERE id = 1; DROP TABLE users; --
结果：删除表。
```

##### 盲注

无直接回显，通过条件推断结果。

布尔盲注：基于真假判断。

- 输入：`1' AND SUBSTRING((SELECT database()), 1, 1) = 'm' --`
- 判断：返回正常则数据库名首字母为 'm'。

时间盲注：通过延迟推断。

- 输入：`1' AND IF(1=1, SLEEP(5), 0) --`
- 判断：响应延迟 5 秒则条件成立。

##### 联合查询注入

使用 UNION 合并查询结果。

```sql
输入：' UNION SELECT username, password FROM users --
查询：SELECT name FROM products WHERE id = '' UNION SELECT username, password FROM users --
结果：返回用户表数据。
```

##### 错误注入

通过数据库错误信息泄露数据。

```sql
输入：1' AND EXTRACTVALUE(1, CONCAT(0x7e, (SELECT database()))) --
结果：错误信息显示数据库名。
```

## 12、NoSQL？KV存储？

NoSQL（Not Only SQL）是一类非关系型数据库的总称，区别于传统的关系型数据库（RDBMS，如 MySQL、PostgreSQL）。

#### 与 RDBMS 的对比

| 特性     | RDBMS           | NoSQL               |
| -------- | --------------- | ------------------- |
| 数据模型 | 表和关系        | 键值、文档、列等    |
| 模式     | 固定 Schema     | 无 Schema 或灵活    |
| 扩展性   | 垂直扩展        | 水平扩展            |
| 一致性   | 强一致性 (ACID) | 最终一致性 (BASE)   |
| 查询语言 | SQL             | 专用 API 或查询语言 |

#### NoSQL 的类型

##### 键值存储

数据以键值对形式存储。

示例：Redis、LevelDB、RocksDB。

##### 文档存储

数据以文档形式存储（通常是 JSON 或 BSON）。

示例：MongoDB、CouchDB。

##### 列存储

数据按列存储，适合分析型查询。

示例：Cassandra、HBase。

##### 图数据库

数据以节点和边存储，适合关系复杂的数据。

示例：Neo4j。

#### 什么是键值存储（KV 存储）？

是 NoSQL 的一种类型，数据以简单的键值对（Key-Value Pair）形式组织。

键（Key）：唯一标识符，通常是字符串或整数。

值（Value）：任意数据（如字符串、整数、对象、二进制）。

操作：基本包括 put(key, value)、get(key)、delete(key)。

##### 底层实现

内存存储：如 Redis，使用哈希表。

磁盘存储：如 LevelDB，使用 LSM 树。

特性：

- 简单：无复杂查询，仅键值操作。
- 高效：O(1) 或 O(log n) 的访问时间。

## 13、HTTP2.0？

#### HTTP/1.1 的局限

队头阻塞（Head-of-Line Blocking）：每个 TCP 连接一次只能处理一个请求-响应，多个请求排队等待。

多连接开销：浏览器通过多个 TCP 连接并行请求资源，增加服务器负担。

头部冗余：每个请求/响应携带大量重复的头部信息（如 User-Agent）。

延迟：文本协议解析慢，握手和传输效率低。

#### HTTP/2 的目标

- 减少延迟。
- 提高吞吐量。
- 兼容 HTTP/1.1 的语义（方法、状态码、URI）。
- 优化现代 Web 需求（如多资源加载）。

#### HTTP/2 的核心特性

##### 二进制协议

HTTP/2 使用二进制格式替代 HTTP/1.x 的文本格式。

结构：

- 数据以帧（Frame）为单位传输。
- 帧类型：HEADERS（头部）、DATA（数据）、SETTINGS（设置）等。

好处：

- 解析更快，减少错误。
- 压缩效率更高。

##### 多路复用

在单一 TCP 连接上并行传输多个请求和响应。

- 每个请求/响应分配一个**流（Stream）**，流有唯一 ID。
- 帧交错发送，客户端和服务器按流 ID 重组。

好处：

- 消除队头阻塞。
- 减少 TCP 连接数。

##### 头部压缩（HPACK）

使用 HPACK 算法压缩 HTTP 头部。

原理：

- 维护动态表，记录常见头部字段（如 Accept）。
- 只传输索引或差异数据。

好处：减少冗余，节省带宽。

##### 服务器推送

服务器主动推送客户端可能需要的资源。

机制：

- 通过 PUSH_PROMISE 帧通知客户端。
- 示例：请求 index.html，服务器推送 style.css。

好处：减少客户端请求次数，降低延迟。

## 14、最长非递减子序列

这里使用动态规划做个分析

#### 思路

##### 动态规划

定义状态：`dp[i]` 表示以第 i 个元素结尾的最长非递减子序列的长度。

递推公式：对于每个位置`i`，遍历之前的元素`j`（`0 ≤ j < i`）：

- 如果 `nums[i] >= nums[j]`，则`dp[i] = max(dp[i], dp[j] + 1)`。
- 否则，`dp[i]` 保持默认值 1。

初始化：每个位置的初始值 dp[i] = 1（单个元素本身是长度为 1 的子序列）。

结果：最终答案是dp数组中的最大值。

##### 贪心 + 二分查找

贪心策略：

- 维护一个数组 tail，tail[i] 表示长度为 i+1 的非递减子序列的末尾最小值。
- 尽量让末尾值尽可能小，以增加后续连接的可能性。

二分查找：对于每个新元素，在 tail 中找到第一个大于等于它的位置，替换掉（保持非递减）。

#### 参考代码

##### 动态规划

输出长度和子序列

```c++
#include <iostream>
#include <vector>
#include <algorithm>

std::vector<int> longestNonDecreasingSubsequence(const std::vector<int>& nums) {
    int n = nums.size();
    if (n == 0) return {};

    // dp[i] 表示以 i 结尾的最长非递减子序列长度
    std::vector<int> dp(n, 1);
    // prev[i] 记录前驱节点，用于重建路径
    std::vector<int> prev(n, -1);

    int maxLen = 1, maxEnd = 0;

    // 计算 dp
    for (int i = 1; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (nums[i] >= nums[j] && dp[j] + 1 > dp[i]) {
                dp[i] = dp[j] + 1;
                prev[i] = j;
            }
        }
        if (dp[i] > maxLen) {
            maxLen = dp[i];
            maxEnd = i;
        }
    }

    // 重建子序列
    std::vector<int> result;
    for (int i = maxEnd; i >= 0; i = prev[i]) {
        result.push_back(nums[i]);
        if (prev[i] == -1) break;
    }
    std::reverse(result.begin(), result.end());

    std::cout << "Length: " << maxLen << "\n";
    return result;
}

int main() {
    std::vector<int> nums = {1, 2, 2, 3, 1, 4};
    auto result = longestNonDecreasingSubsequence(nums);
    std::cout << "Subsequence: ";
    for (int x : result) std::cout << x << " ";
    std::cout << "\n";
    return 0;
}
```

输出：

```c++
Length: 5
Subsequence: 1 2 2 3 4
```

##### 贪心 + 二分查找

仅求长度

```c++
#include <iostream>
#include <vector>
#include <algorithm>

int longestNonDecreasingSubsequenceLength(const std::vector<int>& nums) {
    int n = nums.size();
    if (n == 0) return 0;

    // tail[i] 表示长度为 i+1 的非递减子序列的末尾最小值
    std::vector<int> tail;
    tail.push_back(nums[0]);

    for (int i = 1; i < n; i++) {
        if (nums[i] >= tail.back()) {
            tail.push_back(nums[i]); // 扩展
        } else {
            // 找到第一个大于等于 nums[i] 的位置
            auto it = std::lower_bound(tail.begin(), tail.end(), nums[i]);
            *it = nums[i]; // 替换
        }
    }

    return tail.size();
}

int main() {
    std::vector<int> nums = {1, 2, 2, 3, 1, 4};
    int len = longestNonDecreasingSubsequenceLength(nums);
    std::cout << "Length: " << len << "\n";
    return 0;
}
```

输出：

```c++
Length: 5
```