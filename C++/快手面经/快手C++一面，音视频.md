# 快手C++一面，音视频方向

> 来源：https://www.nowcoder.com/discuss/734531849346613248

## 1、智能指针，解决循环引用

#### unique_ptr

独占所有权的智能指针，同一时刻只有一个 unique_ptr 可以拥有某个对象。

特点：

- 不可复制，只能移动（通过 std::move）。
- 析构时自动释放内存。

用法：

```c++
#include <memory>
#include <iostream>

int main() {
    std::unique_ptr<int> up1 = std::make_unique<int>(10);
    std::cout << *up1 << "\n"; // 输出 10

    // std::unique_ptr<int> up2 = up1; // 错误，不可复制
    std::unique_ptr<int> up2 = std::move(up1); // 转移所有权
    std::cout << (up1 ? "Not null" : "Null") << "\n"; // Null
    return 0;
}
```

#### shared_ptr

共享所有权的智能指针，多个 shared_ptr 可以指向同一对象，通过引用计数管理内存。

特点：

- 引用计数：记录指向对象的 shared_ptr 数量。
- 计数为 0 时释放内存。

用法：

```c++
#include <memory>
#include <iostream>

int main() {
    std::shared_ptr<int> sp1 = std::make_shared<int>(20);
    {
        std::shared_ptr<int> sp2 = sp1; // 引用计数增至 2
        std::cout << *sp2 << " " << sp1.use_count() << "\n"; // 20 2
    } // sp2 析构，计数减至 1
    std::cout << *sp1 << " " << sp1.use_count() << "\n"; // 20 1
} // sp1 析构，计数为 0，内存释放
```

##### 什么是循环引用？

多个 shared_ptr 对象相互引用，形成闭环，导致引用计数永不为 0，内存无法释放。

示例：

```c++
#include <memory>
#include <iostream>

struct Node {
    std::shared_ptr<Node> next;
    ~Node() { std::cout << "Node destroyed\n"; }
};

int main() {
    std::shared_ptr<Node> n1 = std::make_shared<Node>();
    std::shared_ptr<Node> n2 = std::make_shared<Node>();
    n1->next = n2; // n1 引用 n2
    n2->next = n1; // n2 引用 n1
    std::cout << "n1 use_count: " << n1.use_count() << "\n"; // 2
    std::cout << "n2 use_count: " << n2.use_count() << "\n"; // 2
} // 无输出，析构函数未调用
```

问题：

- n1 和 n2 引用计数均为 2。
- 主函数结束时，n1 和 n2 销毁，计数减至 1，但仍不为 0，内存泄漏。

##### 为什么会发生？

引用计数机制：

- 每个`shared_ptr`增加计数，销毁时减少。
- 循环引用形成闭环，计数无法清零。

##### 如何解决？

使用 `weak_ptr`：

- 将循环中的至少一个 `shared_ptr`替换为`weak_ptr`，打破强引用环。
- `weak_ptr`不增加引用计数，仅提供访问。

##### 解决循环引用

```c++
#include <memory>
#include <iostream>

struct Node {
    std::shared_ptr<Node> next;  // 强引用
    std::weak_ptr<Node> prev;    // 弱引用
    ~Node() { std::cout << "Node destroyed\n"; }
};

int main() {
    std::shared_ptr<Node> n1 = std::make_shared<Node>();
    std::shared_ptr<Node> n2 = std::make_shared<Node>();
    n1->next = n2; // n2 的计数增至 2
    n2->prev = n1; // n1 的计数仍为 1，弱引用不计数
    std::cout << "n1 use_count: " << n1.use_count() << "\n"; // 1
    std::cout << "n2 use_count: " << n2.use_count() << "\n"; // 2
} // 正常销毁
```

输出：

```c++
n1 use_count: 1
n2 use_count: 2
Node destroyed
Node destroyed
```

#### weak_ptr

弱引用的智能指针，不控制对象生命周期，用于解决 shared_ptr 的循环引用问题。

特点：

- 不增加引用计数。
- 需通过 lock() 获取 shared_ptr 使用。

示例：

```c++
#include <memory>
#include <iostream>

int main() {
    std::shared_ptr<int> sp = std::make_shared<int>(30);
    std::weak_ptr<int> wp = sp;

    if (auto locked = wp.lock()) { // 检查是否有效
        std::cout << *locked << "\n"; // 30
    }
    sp.reset(); // 释放内存
    if (wp.expired()) {
        std::cout << "Weak pointer expired\n"; // 已失效
    }
    return 0;
}
```

#### auto_ptr（已废弃）

C++98 引入的早期智能指针，独占所有权，但有严重缺陷。

特点：

- 复制时转移所有权，原指针变空。
- 不支持容器（如 `std::vector<auto_ptr>`）。

问题：

```c++
std::auto_ptr<int> ap1(new int(40));
std::auto_ptr<int> ap2 = ap1; // ap1 变空
// *ap1 会崩溃
```

## 2、虚函数表？

#### 什么是虚函数表？

虚函数表是一个指针数组，存储类的虚函数地址。每个包含虚函数的类都有一个唯一的虚函数表，类的对象通过虚表指针（vptr）访问它。

#### 工作原理

##### 虚函数表的生成

条件：类中至少有一个虚函数。

过程：

1. 编译器为类生成一个虚函数表。
2. 表中按声明顺序存储虚函数的地址。
3. 每个对象实例包含一个隐式指针（vptr），指向类的虚函数表。

位置：虚函数表通常存储在程序的只读数据段（.rodata），是静态的。

##### 虚表指针（vptr）

vptr 是编译器在对象中插入的隐藏成员，指向虚函数表。

初始化：

- 在对象构造时，构造函数设置 vptr 指向类的虚函数表。
- 析构时可能调整（涉及虚析构函数）。

大小：通常占 4 字节（32 位系统）或 8 字节（64 位系统）。

##### 调用过程

对象创建：vptr 初始化为类的虚函数表地址。

虚函数调用：

- 通过对象指针或引用访问虚函数。
- 运行时根据 vptr 查找虚函数表。
- 从表中提取对应函数地址并调用。

动态分派：根据对象的实际类型而非声明类型决定调用。

## 3、为什么析构函数要虚函数？

将析构函数声明为虚函数是为了确保通过基类指针或引用删除派生类对象时，能够正确调用派生类的析构函数，从而避免资源泄漏或未定义行为。

#### 不使用虚析构函数的后果

场景：基类指针指向派生类对象，使用 delete 删除。

后果：

- 如果析构函数非虚，仅调用基类析构函数。
- 派生类析构函数未执行，导致资源泄漏或未定义行为。

#### 原理：虚析构函数与虚函数表

当类有虚函数时，编译器生成一个虚函数表（vtable），并在对象中插入虚表指针（vptr）。

虚析构函数被添加到 vtable 中，运行时通过 vptr 调用正确的析构函数。

#### 调用过程

1. 对象构造：

   - `Derived`对象创建时，`vptr`指向`Derived`的`vtable`。

   - `vtable`包含`Derived::~Derived`的地址。

2. 删除对象：

   - `delete b`通过`b`的 `vptr` 查找`vtable`。

   - 调用`vtable`中的析构函数（`Derived::~Derived`）。

3. 析构顺序：先调用派生类析构函数，再自动调用基类析构函数。

## 4、指针常量和常量指针?

#### 指针常量

指针指向的内容是常量，即通过该指针不能修改所指向的值。

语法：`const Type* pointer;`

const 修饰 * 前的类型，表示指向的数据不可变。

特点：

- 指针本身可变（可以指向其他地址）。
- 指向的数据不可变（不能通过指针修改）。

示例：

```c++
#include <iostream>

int main() {
    int a = 10;
    int b = 20;

    const int* ptr = &a; // 指针常量，指向 a
    // *ptr = 30;       // 错误：不能修改指向的值
    std::cout << *ptr << "\n"; // 10

    ptr = &b; // 正确：指针本身可变，指向 b
    std::cout << *ptr << "\n"; // 20

    return 0;
}
```

#### 常量指针

指针本身是常量，即指针的地址不可变，但指向的内容可以修改。

语法：`Type* const pointer;`

const 修饰 pointer，表示指针变量不可变。

特点：

- 指针本身不可变（不能指向其他地址）。
- 指向的数据可变（可以通过指针修改）。

示例：

```c++
#include <iostream>

int main() {
    int a = 10;
    int b = 20;

    int* const ptr = &a; // 常量指针，固定指向 a
    *ptr = 30;        // 正确：可以修改 a 的值
    std::cout << *ptr << "\n"; // 30

    // ptr = &b;     // 错误：指针本身不可变
    return 0;
}
```

#### 对比

| 特性       | 指针常量 (const Type*) | 常量指针 (Type* const) |
| ---------- | ---------------------- | ---------------------- |
| 定义       | 指向常量数据的指针     | 本身是常量的指针       |
| 语法       | const int* ptr         | int* const ptr         |
| 指针可变性 | 可变（可指向其他地址） | 不可变（固定地址）     |
| 数据可变性 | 不可变（只读）         | 可变（可修改数据）     |
| 初始化要求 | 可不立即初始化         | 必须初始化             |
| 典型用法   | 保护数据不被修改       | 固定指针指向           |

## 5、进程间通信?

进程间通信（Inter-Process Communication，简称 IPC） 是操作系统中多个进程之间交换数据或信号的机制。由于进程是独立的执行单元，拥有各自的地址空间，彼此隔离，因此需要特定的方法实现通信。

#### 管道

单向数据流通道，通常用于父子进程通信。

类型：

- 匿名管道：无名，仅限亲缘进程（如父子）。
- 命名管道（FIFO）：有名，可用于无关进程。

特点：

- 单向（读端和写端）。
- 数据按字节流传输。

示例：匿名管道

```c++
#include <unistd.h>
#include <iostream>

int main() {
    int pipefd[2]; // pipefd[0] 读端，pipefd[1] 写端
    pipe(pipefd);

    pid_t pid = fork();
    if (pid == 0) { // 子进程
        close(pipefd[1]); // 关闭写端
        char buf[32];
        read(pipefd[0], buf, sizeof(buf));
        std::cout << "Child received: " << buf << "\n";
        close(pipefd[0]);
    } else { // 父进程
        close(pipefd[0]); // 关闭读端
        const char* msg = "Hello from parent";
        write(pipefd[1], msg, strlen(msg) + 1);
        close(pipefd[1]);
        wait(nullptr);
    }
    return 0;
}
```

#### 消息队列

进程通过队列发送和接收离散消息。

特点：

- 双向通信。
- 消息有类型和优先级。

示例：POSIX 消息队列

```c++
#include <mqueue.h>
#include <fcntl.h>
#include <iostream>

int main() {
    mqd_t mq = mq_open("/myqueue", O_CREAT | O_RDWR, 0666, nullptr);
    if (mq == (mqd_t)-1) {
        perror("mq_open");
        return 1;
    }

    pid_t pid = fork();
    if (pid == 0) { // 子进程接收
        char buf[64];
        mq_receive(mq, buf, 64, nullptr);
        std::cout << "Received: " << buf << "\n";
    } else { // 父进程发送
        const char* msg = "Hello from parent";
        mq_send(mq, msg, strlen(msg) + 1, 0);
        wait(nullptr);
    }
    mq_close(mq);
    mq_unlink("/myqueue");
    return 0;
}
```

#### 共享内存

多个进程映射同一块物理内存，实现直接数据共享。

特点：

- 最快（无需数据拷贝）。
- 需要同步机制（如信号量）。

示例：POSIX 共享内存

```c++
#include <sys/mman.h>
#include <fcntl.h>
#include <unistd.h>
#include <iostream>

int main() {
    int shm_fd = shm_open("/myshm", O_CREAT | O_RDWR, 0666);
    ftruncate(shm_fd, 1024);
    void* ptr = mmap(0, 1024, PROT_WRITE | PROT_READ, MAP_SHARED, shm_fd, 0);

    pid_t pid = fork();
    if (pid == 0) { // 子进程读
        std::cout << "Child read: " << (char*)ptr << "\n";
    } else { // 父进程写
        const char* msg = "Hello from shared memory";
        memcpy(ptr, msg, strlen(msg) + 1);
        wait(nullptr);
    }
    munmap(ptr, 1024);
    shm_unlink("/myshm");
    return 0;
}
```

#### 信号量

用于进程同步的计数器，控制共享资源访问。

特点：不直接传递数据，用于协调。

示例：POSIX 信号量

```c++
#include <semaphore.h>
#include <fcntl.h>
#include <iostream>

int main() {
    sem_t* sem = sem_open("/mysem", O_CREAT, 0666, 0);

    pid_t pid = fork();
    if (pid == 0) { // 子进程
        std::cout << "Child waiting...\n";
        sem_wait(sem);
        std::cout << "Child proceed\n";
    } else { // 父进程
        sleep(1);
        std::cout << "Parent signal\n";
        sem_post(sem);
        wait(nullptr);
    }
    sem_close(sem);
    sem_unlink("/mysem");
    return 0;
}
```

#### 信号

操作系统向进程发送的异步通知。

特点：

- 单向，轻量。
- 用于事件通知（如终止）。

实现：

```c++
#include <signal.h>
#include <iostream>

void handler(int sig) {
    std::cout << "Received signal: " << sig << "\n";
}

int main() {
    signal(SIGUSR1, handler);
    pid_t pid = fork();
    if (pid == 0) {
        sleep(1);
    } else {
        kill(pid, SIGUSR1);
        wait(nullptr);
    }
    return 0;
}
```

#### 套接字

网络通信接口，也可用于本地进程间通信。

特点：双向，支持无关进程和跨主机。

#### IPC 对比

| 方法     | 数据传递 | 速度 | 复杂性 | 适用场景             |
| -------- | -------- | ---- | ------ | -------------------- |
| 管道     | 是       | 中等 | 低     | 亲缘进程，单向数据流 |
| 消息队列 | 是       | 中等 | 中等   | 无关进程，离散消息   |
| 共享内存 | 是       | 最高 | 高     | 大数据量，需同步     |
| 信号量   | 否       | 高   | 低     | 同步控制             |
| 信号     | 否       | 高   | 低     | 事件通知             |
| 套接字   | 是       | 中等 | 高     | 跨主机或本地通信     |

## 6、http和https区别？

#### 基本信息

##### HTTP

超文本传输协议。

一种应用层协议，用于在客户端（如浏览器）和服务器之间传输超文本数据（如 HTML、图片）。

明文传输，无加密。

##### HTTPS

安全的超文本传输协议。

是 HTTP 的加密版本，通过在 HTTP 下层添加 SSL/TLS（安全套接字层/传输层安全协议）实现数据加密和身份验证。

加密传输，安全可靠。

#### 工作原理

##### HTTP 工作流程

1. 建立连接：客户端通过 TCP 三次握手连接服务器（默认端口 80）。
2. 发送请求：客户端发送明文请求（如 GET /index.html HTTP/1.1）。
3. 服务器响应：服务器返回明文响应（如 HTTP/1.1 200 OK 和 HTML 数据）。
4. 关闭连接：HTTP/1.0 默认关闭，HTTP/1.1 支持 Keep-Alive。

##### HTTPS 工作流程

1. SSL/TLS 握手：
   - 客户端 Hello：发送支持的加密算法和随机数。
   - 服务器 Hello：返回选定的加密算法、证书和随机数。
   - 证书验证：客户端验证服务器证书（由 CA 签发）。
   - 密钥交换：双方协商会话密钥（通常用非对称加密生成对称密钥）。
2. 建立加密连接：数据通过对称加密（如 AES）传输。
3. HTTP 通信：在加密通道内发送请求和响应。
4. 关闭连接：释放 TLS 连接和 TCP 连接。

#### 主要区别

| 特性       | HTTP               | HTTPS                |
| ---------- | ------------------ | -------------------- |
| 安全性     | 无加密，明文传输   | 使用 SSL/TLS 加密    |
| 端口       | 80                 | 443                  |
| 协议层     | 直接基于 TCP       | HTTP + SSL/TLS + TCP |
| 身份验证   | 无                 | 通过证书验证服务器   |
| 数据完整性 | 无保障，可能被篡改 | 保障，防止篡改       |
| 性能开销   | 低，无加密开销     | 较高，有加密解密开销 |
| URL 前缀   | http://            | https://             |

## 7、非对称加密和对称加密区别？

#### 对称加密

使用单一密钥（称为对称密钥）对数据进行加密和解密的加密方式。加密和解密使用相同的密钥。

发送方和接收方必须共享同一密钥。

常见算法：AES（高级加密标准）、DES、3DES。

#### 非对称加密

使用一对密钥（公钥和私钥）进行加密和解密。公钥加密的数据只能用私钥解密，反之亦然。

公钥公开，私钥保密。

常见算法：RSA、ECC（椭圆曲线加密）、DSA。

#### 工作原理

##### 对称加密

过程：

1. 发送方使用密钥 K 加密明文 M，生成密文 C。
2. 接收方使用同一密钥 K 解密密文 C，恢复明文 M。

公式：

- 加密：`C = Encrypt(M, K)`
- 解密：`M = Decrypt(C, K)`

##### 非对称加密

过程：

1. 发送方用接收方的公钥加密明文 M，生成密文 C。
2. 接收方用自己的私钥解密密文 C，恢复明文 M。

公式：

- 加密：`C = Encrypt(M, PublicKey)`
- 解密：`M = Decrypt(C, PrivateKey)`

#### 主要区别

| 特性       | 对称加密         | 非对称加密           |
| ---------- | ---------------- | -------------------- |
| 密钥数量   | 1 个（共享密钥） | 2 个（公钥和私钥）   |
| 加密速度   | 快               | 慢                   |
| 密钥分发   | 需安全分发       | 公钥公开，无需分发   |
| 安全性依赖 | 密钥保密         | 私钥保密             |
| 算法复杂度 | 简单（如位运算） | 复杂（如大数运算）   |
| 数据量     | 适合大块数据     | 适合小数据（如密钥） |
| 用途       | 数据加密         | 密钥交换、签名验证   |

## 8、了解过什么设计模式

设计模式

## 9、了解过序列化和反序列化吗？

#### 什么是序列化和反序列化？

##### 序列化

将对象或数据结构（内存中的复杂状态）转换为一种可存储或传输的格式（如字节流、字符串）的过程。

目的：

- 保存到文件（如磁盘存储）。
- 通过网络传输（如客户端-服务器通信）。

输出格式：常见有二进制（如 Protocol Buffers）、文本（如 JSON、XML）。

##### 反序列化

将序列化后的数据（如字节流、字符串）重新还原为内存中的对象或数据结构的过程。

目的：

- 从文件中读取数据。
- 从网络接收数据后重建对象。

关系：序列化和反序列化是一对互逆操作。

#### 工作原理

##### 序列化过程

1. 数据提取：读取对象的字段（如成员变量）。

2. 格式化：

   - 根据目标格式（如 JSON、二进制）组织数据。

   - 可能包括元信息（如字段名、类型）。

3. 编码：

   - 转换为字节流或字符串。

   - 示例：整数 25 可能编码为二进制 00011001。

##### 反序列化过程

1. 解码：从字节流或字符串解析数据。
2. 结构重建：根据格式规则（如 JSON 的键值对）还原字段。
3. 对象填充：将数据填充到新创建的对象中。

## 10、手撕143. 重排链表

#### 思路

直接操作链表节点进行重排较为复杂，通常采用分三步的经典方法：

1. 找到链表中点：将链表分成两半，前半部分是 L0 → L1 → …，后半部分是 Ln-k → … → Ln。
2. 反转后半部分：将后半部分链表反转，变为 Ln → Ln-1 → … → Ln-k。
3. 合并链表：将前半部分和反转后的后半部分交错合并。

##### 详细步骤

###### 步骤 1：找到链表中点

- 方法：使用快慢指针（Floyd 龟兔算法）。
  - 慢指针每次移动 1 步，快指针每次移动 2 步。
  - 快指针到达末尾时，慢指针指向中点（奇数长度指向中间，偶数长度指向前半部分末尾）。
- 示例：
  - `1 → 2 → 3 → 4 → 5 → 6`
  - 慢指针到 3，快指针到末尾，前半部分 `1 → 2 → 3`，后半部分 `4 → 5 → 6`。

###### 步骤 2：反转后半部分

- 方法：迭代法反转链表。

  从中点后的节点开始，逐个调整指针方向。

- 示例：后半部分 `4 → 5 → 6` 反转为 `6 → 5 → 4`。

###### 步骤 3：合并链表

- 方法：交错合并两部分。

  分别从前半部分和反转后的后半部分取节点，依次连接。

- 示例：

  - 前半部分：`1 → 2 → 3`
  - 后半部分：`6 → 5 → 4`
  - 合并：`1 → 6 → 2 → 5 → 3 → 4`。

#### 参考代码（C++）

```c++
#include <iostream>

// 单链表节点定义
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};

// 打印链表
void printList(ListNode* head) {
    while (head) {
        std::cout << head->val << " ";
        head = head->next;
    }
    std::cout << "\n";
}

// 重排链表
void reorderList(ListNode* head) {
    if (!head || !head->next) return; // 空或单个节点无需重排

    // 步骤 1：找中点
    ListNode* slow = head;
    ListNode* fast = head;
    ListNode* prev = nullptr;
    while (fast && fast->next) {
        prev = slow;
        slow = slow->next;
        fast = fast->next->next;
    }
    prev->next = nullptr; // 分割链表

    // 步骤 2：反转后半部分
    ListNode* second = slow;
    ListNode* prevNode = nullptr;
    while (second) {
        ListNode* nextNode = second->next;
        second->next = prevNode;
        prevNode = second;
        second = nextNode;
    }
    second = prevNode; // 反转后的头节点

    // 步骤 3：合并链表
    ListNode* first = head;
    while (second) {
        ListNode* temp1 = first->next;
        ListNode* temp2 = second->next;
        first->next = second;
        if (temp1) second->next = temp1;
        first = temp1;
        second = temp2;
    }
}

int main() {
    // 创建链表：1 → 2 → 3 → 4 → 5 → 6
    ListNode* head = new ListNode(1);
    head->next = new ListNode(2);
    head->next->next = new ListNode(3);
    head->next->next->next = new ListNode(4);
    head->next->next->next->next = new ListNode(5);
    head->next->next->next->next->next = new ListNode(6);

    std::cout << "Original list: ";
    printList(head);

    reorderList(head);

    std::cout << "Reordered list: ";
    printList(head);

    while (head) {
        ListNode* temp = head;
        head = head->next;
        delete temp;
    }
    return 0;
}
```

## 11、二叉树中和为目标值的路径

#### 思路

1. 递归遍历：

   - 从根节点开始，深度优先访问每个节点。

   - 维护当前路径和路径上的节点值。

2. 路径记录：

   - 用一个向量（vector）记录当前路径的节点值。

   - 到达叶子节点时检查路径和。

3. 回溯：递归结束后移除当前节点，回溯到上一层继续探索其他路径。

#### 参考代码（C++）

```c++
#include <iostream>
#include <vector>

// 二叉树节点定义
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class Solution {
public:
    std::vector<std::vector<int>> pathSum(TreeNode* root, int targetSum) {
        std::vector<std::vector<int>> result; // 存储所有有效路径
        std::vector<int> path;                // 当前路径
        dfs(root, targetSum, 0, path, result);
        return result;
    }

private:
    void dfs(TreeNode* root, int targetSum, int currSum, 
             std::vector<int>& path, std::vector<std::vector<int>>& result) {
        // 终止条件：空节点
        if (!root) return;

        // 更新当前路径和
        currSum += root->val;
        path.push_back(root->val);

        // 叶子节点检查
        if (!root->left && !root->right && currSum == targetSum) {
            result.push_back(path); // 路径和匹配，加入结果
        }

        // 递归左右子树
        dfs(root->left, targetSum, currSum, path, result);
        dfs(root->right, targetSum, currSum, path, result);

        // 回溯：移除当前节点
        path.pop_back();
    }
};

void printResult(const std::vector<std::vector<int>>& result) {
    for (const auto& path : result) {
        std::cout << "[";
        for (int i = 0; i < path.size(); ++i) {
            std::cout << path[i];
            if (i < path.size() - 1) std::cout << ",";
        }
        std::cout << "]\n";
    }
}

int main() {
    // 构建二叉树 [5,4,8,11,null,13,4,7,2,null,null,5,1]
    TreeNode* root = new TreeNode(5);
    root->left = new TreeNode(4);
    root->right = new TreeNode(8);
    root->left->left = new TreeNode(11);
    root->left->left->left = new TreeNode(7);
    root->left->left->right = new TreeNode(2);
    root->right->left = new TreeNode(13);
    root->right->right = new TreeNode(4);
    root->right->right->left = new TreeNode(5);
    root->right->right->right = new TreeNode(1);

    Solution solution;
    int targetSum = 22;
    auto result = solution.pathSum(root, targetSum);

    std::cout << "Paths with sum " << targetSum << ":\n";
    printResult(result);

    // 释放内存
    delete root->right->right->right;
    delete root->right->right->left;
    delete root->right->right;
    delete root->right->left;
    delete root->left->left->right;
    delete root->left->left->left;
    delete root->left->left;
    delete root->left;
    delete root->right;
    delete root;

    return 0;
}
```

