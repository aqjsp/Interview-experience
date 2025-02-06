# 字节推荐架构C++一面，八股部分

> 来源：https://www.nowcoder.com/discuss/709786059567013888

### 1、智能指针

这里推荐一篇我之前总结过的文章：

### 2、手写一个shared_ptr

一个简单的shared_ptr主要包括：

- 引用计数机制
- 自动释放内存
- 支持拷贝构造、拷贝赋值和移动语义

代码示例：

```c
#include <iostream>
#include <atomic>

template <typename T>
class MySharedPtr {
private:
    T* ptr;  // 原始指针
    std::atomic<int>* ref_count;  // 引用计数

public:
    // 构造函数
    explicit MySharedPtr(T* p = nullptr) : ptr(p), ref_count(new std::atomic<int>(1)) {}

    // 拷贝构造函数
    MySharedPtr(const MySharedPtr& other) : ptr(other.ptr), ref_count(other.ref_count) {
        ++(*ref_count);  // 增加引用计数
    }

    // 移动构造函数
    MySharedPtr(MySharedPtr&& other) noexcept : ptr(other.ptr), ref_count(other.ref_count) {
        other.ptr = nullptr;  // 释放移动后指针的资源
        other.ref_count = nullptr;
    }

    // 拷贝赋值运算符
    MySharedPtr& operator=(const MySharedPtr& other) {
        if (this != &other) {
            release();  // 释放当前资源

            // 复制资源
            ptr = other.ptr;
            ref_count = other.ref_count;
            ++(*ref_count);  // 增加引用计数
        }
        return *this;
    }

    // 移动赋值运算符
    MySharedPtr& operator=(MySharedPtr&& other) noexcept {
        if (this != &other) {
            release();  // 释放当前资源

            // 迁移资源
            ptr = other.ptr;
            ref_count = other.ref_count;
            other.ptr = nullptr;
            other.ref_count = nullptr;
        }
        return *this;
    }

    // 析构函数
    ~MySharedPtr() {
        release();
    }

    // 获取原始指针
    T* get() const { return ptr; }

    // 重载*运算符
    T& operator*() const { return *ptr; }

    // 重载->运算符
    T* operator->() const { return ptr; }

    // 获取引用计数
    int use_count() const { return ref_count ? *ref_count : 0; }

private:
    // 释放资源
    void release() {
        if (ref_count && --(*ref_count) == 0) {
            delete ptr;         // 释放资源
            delete ref_count;   // 删除引用计数对象
        }
    }
};

int main() {
    // 创建 shared_ptr
    MySharedPtr<int> ptr1(new int(10));  // 引用计数为 1

    std::cout << *ptr1 << std::endl;  // 输出 10
    std::cout << ptr1.use_count() << std::endl;  // 输出 1

    // 拷贝构造
    MySharedPtr<int> ptr2 = ptr1;  // 引用计数增加到 2
    std::cout << ptr2.use_count() << std::endl;  // 输出 2

    // 移动构造
    MySharedPtr<int> ptr3 = std::move(ptr2);  // 引用计数依然为 2
    std::cout << ptr3.use_count() << std::endl;  // 输出 2
    std::cout << ptr2.use_count() << std::endl;  // 输出 0，ptr2 已为空

    // 移动赋值
    MySharedPtr<int> ptr4;
    ptr4 = std::move(ptr3);  // ptr4 现在持有资源，ptr3 为空
    std::cout << ptr4.use_count() << std::endl;  // 输出 1

    return 0;  // 最后 ptr4 超出作用域，引用计数为 0，资源被释放
}
```

### 3、TCP UDP区别？

#### 1. TCP（Transmission Control Protocol） 传输控制协议

- 面向连接：在数据传输前，TCP需要通过三次握手建立连接。这确保了通信双方能够确认连接是否成功，并为后续的通信做好准备。
- 可靠性：提供可靠的数据传输，确保数据包按顺序到达目的地。如果数据丢失、出错或乱序，TCP 会进行重传，直到数据成功到达。
- 流量控制：使用滑动窗口机制来控制流量，确保接收方的缓冲区不会被溢出。
- 拥塞控制：可以根据网络状况调整发送数据的速率，避免因网络拥塞导致的数据丢失。
- 数据顺序：确保数据按照发送的顺序到达接收方，即使数据包在传输过程中发生乱序，接收方也会将其重新排序。

#### 2. UDP（User Datagram Protocol） 用户数据报协议

- 无连接：一个无连接协议，不需要建立连接，直接发送数据。由于没有建立连接的过程，UDP 协议的开销比 TCP 小，传输速度较快。
- 不可靠性：不提供可靠的数据传输保证。它不对数据包进行确认和重传，也不确保数据包按顺序到达目的地。
- 没有流量控制和拥塞控制：不会进行流量控制和拥塞控制，因此它不会根据网络的负载情况调整数据传输的速率。
- 数据顺序：不保证数据包的顺序，接收方接收到的顺序可能与发送方发送的顺序不同。
- 轻量级：头部较小，传输速度快，适用于实时传输和对时延敏感的应用。

#### 3. TCP 和 UDP 的区别

| 特性       | **TCP**                              | **UDP**                        |
| ---------- | ------------------------------------ | ------------------------------ |
| 连接类型   | 面向连接（需要三次握手）             | 无连接（无需建立连接）         |
| 可靠性     | 提供可靠的数据传输，确保数据完整性   | 不保证数据传输的可靠性         |
| 顺序保证   | 保证数据按顺序到达                   | 不保证数据顺序                 |
| 流量控制   | 提供流量控制（滑动窗口机制）         | 不提供流量控制                 |
| 拥塞控制   | 提供拥塞控制                         | 不提供拥塞控制                 |
| 开销       | 较大（有三次握手和四次挥手过程）     | 较小（没有连接和确认过程）     |
| 传输速度   | 较慢，因存在连接建立和维护的开销     | 较快，因没有连接建立的开销     |
| 应用层协议 | HTTP, FTP, SMTP, POP3, IMAP 等       | DNS, VoIP, SNMP, TFTP, IPTV 等 |
| 数据包大小 | 有头部和有效载荷，头部较大（20字节） | 头部较小（8字节）              |
| 错误检测   | 有（校验和、序列号、确认等）         | 有（仅有校验和）               |

### 4、堆和栈的区别？

#### 1. 栈（Stack）

##### 定义

栈是程序运行时用来存储局部变量、函数调用等数据的内存区域。栈内存由操作系统自动管理，采用先进后出（LIFO，Last In First Out）方式分配和释放内存。

##### 特点

- 自动管理：栈内存的分配和释放由编译器和操作系统自动完成，程序员无需手动管理。
- 内存分配：栈内存由操作系统提供，通常用于存储局部变量、函数参数和返回地址等信息。
- 生命周期：栈内存中的数据生命周期较短，仅在函数调用时有效，函数退出时自动销毁。
- 存储方式：栈内存通过连续的内存地址进行存储，分配速度非常快。
- 大小限制：栈的大小通常是有限的，由操作系统或编译器设定。如果栈空间过大，可能导致栈溢出（stack overflow）。

##### 操作流程

1. 入栈：当函数被调用时，局部变量和函数参数会被推入栈中。
2. 出栈：当函数调用结束时，局部变量和函数参数会被销毁，栈指针回退，栈空间被回收。

##### 示例

```c
void function() {
    int x = 10;  // 局部变量 x 存储在栈上
    int y = 20;  // 局部变量 y 存储在栈上
}  // 当函数结束时，x 和 y 自动销毁
```

##### 栈的优缺点

- 优点：
  - 快速分配和释放：栈的分配和回收速度非常快，因为栈操作是连续的，系统只需要移动栈指针即可。
  - 自动管理：栈的内存由系统自动管理，程序员无需手动释放内存。
  - 数据局部性好：栈中存储的数据一般是局部变量，具有较好的局部性（可以提高 CPU 缓存命中率）。
- 缺点：
  - 内存大小有限：栈的内存空间通常比堆小，超过栈的最大限制会导致栈溢出（例如递归过深）。
  - 不适合存储大型数据：栈上的数据通常较小（如局部变量、基本数据类型等），不适合存储大对象或动态分配的内存。

#### 2. 堆（Heap）

##### 定义

堆是用于动态分配内存的区域。程序员需要手动管理堆内存的分配和释放，堆内存通常用于存储程序运行时动态分配的数据，如对象、数组等。

##### 特点

- 手动管理：堆内存的分配和释放由程序员显式控制，需要使用 `new`（C++）或 `malloc`（C）来分配内存，使用 `delete` 或 `free` 来释放内存。如果程序员忘记释放堆内存，可能导致内存泄漏。
- 内存分配：堆内存不是连续的，操作系统在堆区域中根据需要分配内存块。
- 生命周期：堆内存的生命周期由程序员控制，只要程序员不显式释放堆内存，它会一直存在，直到程序结束。
- 大小限制：堆的内存大小通常较大，只有受限于计算机的可用内存。

##### 操作流程

1. 分配内存：程序员通过 `new` 或 `malloc` 请求内存分配，堆内存由操作系统分配并返回指针。
2. 释放内存：程序员通过 `delete` 或 `free` 来释放堆内存。如果没有释放，内存会一直占用，导致内存泄漏。

##### 示例

```c
void function() {
    int* p = new int(10);  // 在堆上分配内存
    // 使用 p
    delete p;  // 释放堆内存
}
```

##### 堆的优缺点

- 优点：
  - 内存空间大：堆内存通常比栈大，可以用于存储大量数据或需要长时间存在的数据（如大型对象或数组）。
  - 灵活性：堆内存可以动态分配，可以在运行时决定分配的内存大小。
- 缺点：
  - 慢速分配和释放：与栈相比，堆的内存分配和释放较慢，因为需要管理内存的分配、释放和合并操作。
  - 手动管理：堆内存的管理需要程序员手动进行，容易发生内存泄漏（未释放内存）或悬挂指针（指向已释放的内存）。
  - 可能导致碎片化：频繁分配和释放内存可能导致堆内存碎片化，影响性能。

#### 3. 堆与栈的比较

| 特性         | **栈（Stack）**                | **堆（Heap）**                               |
| ------------ | ------------------------------ | -------------------------------------------- |
| 内存分配方式 | 自动分配和释放，由操作系统管理 | 手动分配和释放，由程序员管理                 |
| 生命周期     | 局部变量的生命周期与函数相同   | 由程序员控制，只要不显式释放，内存会一直存在 |
| 内存大小     | 内存有限，通常较小             | 通常较大，受限于系统的可用内存               |
| 分配速度     | 分配速度快                     | 分配速度慢                                   |
| 内存碎片     | 没有碎片问题                   | 可能会出现内存碎片（特别是频繁分配和释放时） |
| 适用场景     | 存储局部变量、函数调用信息等   | 存储动态分配的对象、数组等                   |
| 访问速度     | 访问速度快，局部性好           | 访问速度较慢                                 |
| 错误         | 栈溢出（Stack Overflow）       | 内存泄漏、悬挂指针、堆溢出等                 |

### 5、最新的http协议基于什么？

最新的 HTTP 协议是 **HTTP/3**，它基于 **QUIC**（Quick UDP Internet Connections）协议。HTTP/3 是对早期 HTTP/2 的改进，旨在解决一些在 HTTP/2 中存在的问题，如延迟和连接的多路复用性能问题。HTTP/3 具有显著的性能提升，尤其是在高延迟和不稳定网络环境下。

#### 1. QUIC 协议：HTTP/3 的基础

QUIC 是由 Google 发起的一个协议，最初的设计目的是为了改进 HTTP 协议的性能，尤其是在移动设备和高延迟环境中的表现。它是基于 UDP（用户数据报协议）的，而不是传统的 TCP（传输控制协议）。

##### QUIC 的特点：

- 基于 UDP：QUIC 直接使用 UDP 作为传输层协议，而不是 TCP。虽然 UDP 本身不保证数据可靠传输，但 QUIC 在其上实现了类似 TCP 的可靠性、流量控制和拥塞控制机制，提供了高效的连接管理。
- 多路复用：QUIC 可以实现请求的多路复用，即多个请求可以在同一个连接中同时进行，而不会像 HTTP/2 中那样存在头部阻塞问题。每个流（请求）都可以独立进行管理，避免了 TCP 中的队头阻塞问题。
- 减少握手次数：QUIC 可以在建立连接时通过 0-RTT 或 1-RTT（即零或一次往返时间）来完成连接的建立，这使得连接建立比 TCP + TLS 更快。传统的 HTTP/2 通常需要 2 次往返时间（RTT）来建立安全连接。
- 加密内建：QUIC 自带加密，默认使用 TLS 1.3 进行加密，提供了与传统的 HTTPS 协议相同的安全性，而不需要额外的加密层。这样不仅减少了延迟，还确保了数据的隐私和安全。

#### 2. HTTP/3 的特性

HTTP/3 基于 QUIC 协议，因此它继承了 QUIC 的一些特点，并在此基础上进一步优化了 HTTP 协议的性能和安全性。

##### 2.1 减少延迟

HTTP/3 减少了连接建立的延迟。通过 QUIC 协议的多路复用和快速连接建立机制，HTTP/3 比 HTTP/2 更快地建立连接。QUIC 的 0-RTT 和 1-RTT 握手方式，使得客户端和服务器之间的连接速度大大提高。

##### 2.2 更好的多路复用

在 HTTP/2 中，尽管引入了多路复用技术，但由于 TCP 的队头阻塞问题，多个请求仍可能受到一个慢请求的影响。HTTP/3 基于 QUIC 解决了这个问题，QUIC 在传输多个数据流时可以避免一个流的延迟影响到其他流，因此多个请求可以并行处理。

##### 2.3 改进的头部压缩

HTTP/2 引入了 HPACK 进行头部压缩，而 HTTP/3 使用了新的头部压缩机制 QPACK。QPACK 能更好地适应 QUIC 的流控制和多路复用特性，从而进一步减少了延迟。

##### 2.4 更强的抗丢包能力

QUIC 提供了更强的抗丢包能力。它在丢包时能够保持其他数据流的畅通，即使其中一个流发生丢包，其他流不会受到阻塞，因此具有更好的网络适应性。

##### 2.5 内置加密

与 HTTPS 中的 TLS 不同，QUIC 协议在设计时就内置了加密，这意味着 HTTP/3 也内建加密。这样，HTTP/3 不仅加速了连接建立，还确保了连接的安全性。

#### 3. HTTP/3 与 HTTP/2 的区别

| 特性           | **HTTP/2**                                              | **HTTP/3**                                       |
| -------------- | ------------------------------------------------------- | ------------------------------------------------ |
| **基础协议**   | 基于 TCP 和 TLS                                         | 基于 QUIC 协议（基于 UDP）                       |
| **多路复用**   | 解决了 HTTP/1.x 的队头阻塞问题，但仍受限于 TCP 队头阻塞 | 彻底解决了队头阻塞问题，基于 QUIC 的多路复用     |
| **连接建立**   | 需要 2 次 RTT（往返时间）                               | 通过 QUIC 协议，使用 0-RTT 或 1-RTT 更快建立连接 |
| **加密**       | 必须使用 TLS 加密                                       | 默认内建加密，使用 TLS 1.3                       |
| **头部压缩**   | 使用 HPACK                                              | 使用 QPACK，针对 QUIC 进行了优化                 |
| **丢包容忍性** | 丢包时会影响所有流（TCP 队头阻塞）                      | QUIC 的丢包不影响其他流                          |

### 6、select和poll的区别？

这里将select、poll、epoll都做一个整理。

网络编程中，**`select`**、**`poll`** 和 **`epoll`** 都是常用的 I/O 多路复用机制，它们允许程序在一个线程中同时监视多个文件描述符（如 socket）是否就绪，以实现高效的并发处理。

#### 1. `select`

##### 工作原理：

`select` 是最早出现的 I/O 多路复用机制，它通过检查文件描述符的状态来确定哪些文件描述符可以进行读、写或发生异常。`select` 在应用程序中非常常见，但其性能随着文件描述符数量的增加会显著下降。

##### 特点：

- 阻塞与非阻塞：`select` 可以阻塞在调用中，直到至少一个文件描述符准备好为止，也可以通过设置超时来让它非阻塞。
- 文件描述符数量限制：`select` 的最大文件描述符数量是有限制的。在 Linux 上，通常最大值为 1024（由 `FD_SETSIZE` 限制）。如果需要监视更多的文件描述符，需要重新编译或修改内核参数。
- 性能：当监视的文件描述符数量很多时，`select` 会变得非常低效，因为每次调用都会遍历整个文件描述符集来检查哪些文件描述符就绪。

##### 使用方式：

```c
#include <sys/select.h>
#include <unistd.h>

fd_set read_fds;
FD_ZERO(&read_fds);
FD_SET(sockfd, &read_fds);

struct timeval timeout;
timeout.tv_sec = 5;  // 5 秒超时
timeout.tv_usec = 0;

int ret = select(sockfd + 1, &read_fds, NULL, NULL, &timeout);
if (ret > 0) {
    // 文件描述符就绪，处理数据
} else if (ret == 0) {
    // 超时，处理
} else {
    // 错误处理
}
```

缺点：

- 效率低下：当监视的文件描述符数量很大时，`select` 每次调用都需要遍历整个文件描述符集合，性能随着文件描述符数量增加而下降。
- 文件描述符限制：最大监视的文件描述符数量是固定的，通常为 1024。
- 需要手动管理文件描述符集合：每次调用 `select` 时，都需要重新设置文件描述符集合，管理起来不太方便。

#### 2. `poll`

##### 工作原理：

`poll` 是 `select` 的改进版本，它克服了 `select` 中的一些限制，尤其是文件描述符数量有限的问题。`poll` 使用 `pollfd` 结构体数组来管理文件描述符，并且支持更多的文件描述符。

##### 特点：

- 文件描述符数量不受限制：`poll` 没有 `select` 那样的文件描述符数量限制，可以处理任意数量的文件描述符，受系统内存的限制。
- 每次调用返回的就绪状态更全面：`poll` 返回的 `pollfd` 数组中包含每个文件描述符的就绪状态，应用程序只需根据返回结果处理即可。
- 性能：虽然 `poll` 克服了 `select` 的文件描述符数量限制，但它依然是线性扫描文件描述符列表，随着文件描述符数量的增加，性能会下降。

##### 使用方式：

```c
#include <poll.h>
#include <unistd.h>

struct pollfd fds[1];
fds[0].fd = sockfd;
fds[0].events = POLLIN;  // 监听可读事件

int ret = poll(fds, 1, 5000);  // 5000 毫秒超时
if (ret > 0) {
    if (fds[0].revents & POLLIN) {
        // 文件描述符就绪，处理数据
    }
} else if (ret == 0) {
    // 超时，处理
} else {
    // 错误处理
}
```

##### 缺点：

- 效率较低：尽管 `poll` 解决了 `select` 中的文件描述符数量限制，但每次调用时，`poll` 依然需要扫描整个文件描述符数组，性能随着文件描述符数量增加而下降。
- 不支持边缘触发：不像 `epoll`，`poll` 只提供水平触发（Level-Triggered）模式，这意味着即使文件描述符已经就绪，`poll` 也会反复返回，就算数据没有处理。

#### 3. `epoll`

##### 工作原理：

`epoll` 是 Linux 特有的高效 I/O 多路复用机制，设计目的是解决 `select` 和 `poll` 在性能上的问题，尤其是在监视大量文件描述符时的性能瓶颈。`epoll` 采用事件驱动和回调机制，允许操作系统将就绪的文件描述符返回给应用程序，而不需要应用程序每次都扫描整个文件描述符集合。

##### 特点：

- 事件驱动：`epoll` 是事件驱动的，可以通过回调机制直接处理就绪的事件，而不需要遍历文件描述符列表。只有在文件描述符的状态发生变化时，才会通知应用程序。
- 高效性：`epoll` 的性能与文件描述符数量的增加基本无关。它使用内核空间的事件通知机制，通过 `epoll_wait` 只返回就绪的文件描述符，大大提高了效率。
- 支持水平触发和边缘触发：
  - 水平触发（Level-Triggered）：类似于 `poll` 和 `select`，只要文件描述符没有被处理，它就会一直返回。
  - 边缘触发（Edge-Triggered）：只在文件描述符的状态从不可读/不可写变为可读/可写时，才会通知应用程序。适合于高效的事件处理，但需要保证每次都处理数据，以避免丢失事件。

##### 使用方式：

```c
#include <sys/epoll.h>
#include <unistd.h>

int epoll_fd = epoll_create1(0);
struct epoll_event ev;
ev.events = EPOLLIN;  // 监听可读事件
ev.data.fd = sockfd;

epoll_ctl(epoll_fd, EPOLL_CTL_ADD, sockfd, &ev);  // 将文件描述符添加到 epoll 实例

struct epoll_event events[10];
int ret = epoll_wait(epoll_fd, events, 10, 5000);  // 5000 毫秒超时
if (ret > 0) {
    for (int i = 0; i < ret; ++i) {
        if (events[i].events & EPOLLIN) {
            // 文件描述符就绪，处理数据
        }
    }
} else if (ret == 0) {
    // 超时，处理
} else {
    // 错误处理
}
```

##### 优点：

- 性能极高：`epoll` 的性能不会随着文件描述符的增加而降低，适合监控大量文件描述符。
- 事件通知机制：通过内核向应用程序推送就绪事件，避免了每次遍历所有文件描述符的低效操作。
- 边缘触发：可以通过边缘触发减少不必要的事件处理，提高性能。

##### 缺点：

- Linux 特性：`epoll` 仅在 Linux 系统上可用，其他操作系统（如 Windows）并不支持。
- 使用复杂性：与 `select` 和 `poll` 相比，`epoll` 的编程模型相对复杂，尤其是在使用边缘触发模式时，程序员需要确保每次都处理所有可读数据。

#### 4. 总结

| 特性           | **`select`**                     | **`poll`**                       | **`epoll`**                              |
| -------------- | -------------------------------- | -------------------------------- | ---------------------------------------- |
| 文件描述符限制 | 受限（通常为 1024）              | 没有文件描述符限制               | 没有文件描述符限制                       |
| 性能           | 线性扫描所有文件描述符，性能较差 | 线性扫描文件描述符，性能较差     | 高效，性能不受文件描述符数量影响         |
| 实现方式       | 阻塞或非阻塞，轮询检查文件描述符 | 阻塞或非阻塞，轮询检查文件描述符 | 使用事件驱动模型，内核通知               |
| 适用场景       | 文件描述符较少的简单场景         | 文件描述符较多的简单场景         | 需要高性能并且处理大量并发连接的场景     |
| 支持的触发模式 | 水平触发（Level-Triggered）      | 水平触发（Level-Triggered）      | 支持水平触发和边缘触发（Edge-Triggered） |

- **`select`** 和 **`poll`** 都是传统的 I/O 多路复用机制，它们的性能会随着文件描述符数量增加而显著下降，适用于文件描述符数量较少的场景。
- **`epoll`** 是现代高效的 I/O 多路复用机制，尤其适用于高并发场景，能够处理大量文件描述符而不会出现性能瓶颈。

### 7、select支持的最大数量？

### 8、文件句柄的底层实现？

操作系统中，**文件句柄（File Handle）** 是操作系统用于标识和管理打开文件的唯一标识符。它提供了一个抽象层，允许用户程序通过它进行文件操作（如读取、写入、关闭等）。

先从概念说起，文件句柄通常在用户级别表示为一个整数值，操作系统使用它来表示和追踪一个文件的状态、位置、权限等信息。文件句柄提供了一种访问文件的接口，它使得程序无需关心文件存储的具体位置、硬件设备等底层细节。

再来说说文件描述符是什么？

文件句柄通常与文件描述符紧密相关，尤其是在类 Unix 操作系统（如 Linux 和 macOS）中。文件描述符是一个整数值，用于唯一标识一个打开的文件或资源（如管道、网络连接等）。当程序调用 `open()` 打开一个文件时，操作系统为文件分配一个文件描述符，并将该描述符作为文件句柄返回给应用程序。

在类 Unix 操作系统中，文件描述符是一个索引，指向一个内核数据结构，称为“文件描述符表”（File Descriptor Table）。

最后就说说它的底层实现？

主要依赖于操作系统内核对文件描述符的管理。操作系统为每个进程维护一个**文件描述符表**，用于追踪该进程打开的文件及其相关信息。

#### 文件句柄在操作系统中的底层实现过程：

##### 1. 文件描述符表

在操作系统内核中，每个进程都有一个文件描述符表，该表存储了该进程打开的所有文件的状态信息。文件描述符表是一个指向内核数据结构的索引表，每个文件描述符对应一个内核中的“文件对象”（File Object）。这些文件对象保存了文件的具体信息，例如文件的位置、读写位置、权限等。

在 Linux 中，这些文件描述符表通常存在于进程的任务结构体中，每个任务结构体都有一个指向文件描述符表的指针。

```c
struct task_struct {
    ...
    struct files_struct *files;
    ...
};
```

每个文件描述符在文件描述符表中是一个索引，指向一个内核级的 `struct file` 结构体，这个结构体包含了文件的各种状态信息。

##### 2. `struct file` 结构体

在 Linux 中，`struct file` 是内核表示打开文件的关键数据结构，它保存了文件的各类信息。它包含了文件的指针、文件的操作方法、文件状态等信息。

一个简单的 `struct file` 结构可能看起来如下：

```c
struct file {
    unsigned int f_flags;           // 文件的标志位
    struct file_operations *f_op;   // 文件操作的函数指针
    void *private_data;             // 私有数据
    struct inode *f_inode;          // 与文件相关的 inode
    off_t f_pos;                    // 当前文件位置（用于读写）
};
```

- `f_flags`：表示文件的访问模式，如只读、只写、读写等。
- `f_op`：指向文件操作结构体 `file_operations`，它包含了文件的各种操作方法，如 `read`、`write`、`open`、`close` 等。
- `private_data`：存储文件的私有数据，通常用于特定类型的文件，如设备文件。
- `f_inode`：指向文件的 `inode` 结构体，`inode` 保存了文件的元数据，如文件大小、权限等。
- `f_pos`：当前文件的读写位置。

##### 3. 内核中的 I/O 调度

操作系统的内核通过 I/O 调度机制管理文件操作。每个打开的文件都与文件描述符和 `struct file` 关联，而每个文件描述符又关联到特定的内核资源（如磁盘上的块）。当应用程序执行读写操作时，内核会根据文件描述符的状态，读取或写入磁盘上的数据，并根据 `f_pos` 更新文件的读写位置。

##### 4. 文件描述符表的管理

文件描述符表是进程级别的，进程打开文件时，内核为每个文件分配一个文件描述符，并在文件描述符表中为其创建一条记录。文件描述符表通常包含三个标准的文件描述符：

- **0**：标准输入（stdin）
- **1**：标准输出（stdout）
- **2**：标准错误（stderr）

当应用程序通过 `open()` 打开一个文件时，操作系统会为其分配一个文件描述符，并将文件描述符指向一个文件对象。此后，应用程序可以通过这个文件描述符来执行文件操作，如读取文件内容或写入文件。

##### 5. 文件句柄与文件描述符的映射

在某些操作系统中，文件句柄与文件描述符是一一对应的。例如，在类 Unix 操作系统中，文件句柄就是文件描述符。然而在 Windows 中，文件句柄的概念更为广泛，它不仅限于文件，也包括管道、套接字等资源。尽管在不同操作系统中对文件句柄的定义和使用略有不同，但基本原理是相似的：文件句柄通过内核数据结构与实际的文件对象进行映射，提供了访问和管理文件的接口。

### 9、红黑树？

**红黑树**（Red-Black Tree）是一种自平衡的二叉查找树（Binary Search Tree, BST），其每个节点都有一个颜色（红色或黑色），并且满足一组特定的规则。红黑树的核心优势在于它可以在插入和删除节点时保持平衡，从而保证最坏情况下的时间复杂度为 O(log n)。

#### 1. 红黑树的性质

红黑树的节点有两种颜色，红色和黑色。为了保持树的平衡，红黑树需要满足以下五个性质：

1. **每个节点要么是红色，要么是黑色**。
2. **根节点必须是黑色**。
3. **所有叶子节点（NIL节点，表示空子树）都是黑色**。
4. **如果一个红色节点的子节点存在，那么其子节点必须是黑色**（即，红色节点不能连续出现）。
5. **从任意节点到其所有叶子节点的路径上，必须有相同数目的黑色节点**。

这些规则保证了红黑树的高度不会过大，从而保持了操作的对数时间复杂度。

#### 2. 红黑树的结构

红黑树是一种**二叉查找树**，因此它有以下基本结构：

- **每个节点包含**：
  - `key`：存储的数据值（用于比较大小，通常是键值）。
  - `color`：节点的颜色（红色或黑色）。
  - `left`：指向左子树的指针。
  - `right`：指向右子树的指针。
  - `parent`：指向父节点的指针。
- **特殊的叶子节点**：红黑树中存在虚拟的“叶子节点”或“外部节点”（也叫 NIL 节点），它们是黑色的，并且不包含数据。

#### 3. 红黑树的基本操作

##### 3.1 插入操作

插入新节点时，红黑树首先会将其作为普通的二叉查找树插入（按照键值大小进行插入），然后会对树进行调整，以确保红黑树的性质依然成立。

插入步骤：

1. 插入新节点作为红色节点。
2. 如果父节点是黑色，则无需调整树的平衡，插入操作完成。
3. 如果父节点是红色，违反了规则 4（两个连续的红色节点），则需要进行调整。
4. 调整过程分为旋转和重新着色。具体的调整操作包括：
   - 旋转：单旋转（左旋或右旋）或双旋转（左旋+右旋或右旋+左旋）。
   - 重新着色：将某些节点的颜色改为黑色或红色。

##### 3.2 删除操作

删除节点的操作相对复杂。与插入操作不同，删除操作会破坏红黑树的平衡，需要通过旋转和重新着色来修复。删除节点的步骤如下：

1. 查找并删除节点。
2. 如果删除的节点是红色的，直接删除，不需要调整。
3. 如果删除的是黑色节点，可能会破坏从根到叶子的黑色节点计数，需要进行平衡操作。
4. 调整过程中可能会进行旋转、着色和兄弟节点的替换。

##### 3.3 旋转操作

旋转操作是保持红黑树平衡的核心。旋转有两种类型：

- 左旋（Left Rotate）：将当前节点向左旋转，父节点变为左子节点，当前节点变为父节点的左子节点。
- 右旋（Right Rotate）：将当前节点向右旋转，父节点变为右子节点，当前节点变为父节点的右子节点。

旋转操作的目的是通过局部的结构调整，恢复树的平衡。

代码示例：

```c
#include <iostream>
using namespace std;

enum Color { RED, BLACK };

template <typename T>
struct Node {
    T data;
    Node* parent;
    Node* left;
    Node* right;
    Color color;

    Node(T data) : data(data), parent(nullptr), left(nullptr), right(nullptr), color(RED) {}
};

template <typename T>
class RedBlackTree {
private:
    Node<T>* root;
    Node<T>* TNULL;  // NIL节点

    // 左旋操作
    void leftRotate(Node<T>* x) {
        Node<T>* y = x->right;
        x->right = y->left;
        if (y->left != TNULL) {
            y->left->parent = x;
        }
        y->parent = x->parent;
        if (x->parent == nullptr) {
            root = y;
        } else if (x == x->parent->left) {
            x->parent->left = y;
        } else {
            x->parent->right = y;
        }
        y->left = x;
        x->parent = y;
    }

    // 右旋操作
    void rightRotate(Node<T>* x) {
        Node<T>* y = x->left;
        x->left = y->right;
        if (y->right != TNULL) {
            y->right->parent = x;
        }
        y->parent = x->parent;
        if (x->parent == nullptr) {
            root = y;
        } else if (x == x->parent->right) {
            x->parent->right = y;
        } else {
            x->parent->left = y;
        }
        y->right = x;
        x->parent = y;
    }

    // 插入修复函数，确保红黑树的性质
    void fixInsert(Node<T>* k) {
        Node<T>* u;
        while (k->parent != nullptr && k->parent->color == RED) {  // 确保父节点不为空
            if (k->parent == k->parent->parent->right) {
                u = k->parent->parent->left;
                if (u->color == RED) {
                    u->color = BLACK;
                    k->parent->color = BLACK;
                    k->parent->parent->color = RED;
                    k = k->parent->parent;
                } else {
                    if (k == k->parent->left) {
                        k = k->parent;
                        rightRotate(k);
                    }
                    k->parent->color = BLACK;
                    k->parent->parent->color = RED;
                    leftRotate(k->parent->parent);
                }
            } else {
                u = k->parent->parent->right;
                if (u->color == RED) {
                    u->color = BLACK;
                    k->parent->color = BLACK;
                    k->parent->parent->color = RED;
                    k = k->parent->parent;
                } else {
                    if (k == k->parent->right) {
                        k = k->parent;
                        leftRotate(k);
                    }
                    k->parent->color = BLACK;
                    k->parent->parent->color = RED;
                    rightRotate(k->parent->parent);
                }
            }
            if (k == root) {
                break;
            }
        }
        root->color = BLACK;
    }


    // 插入节点
    void insert(T key) {
        Node<T>* node = new Node<T>(key);
        Node<T>* y = nullptr;
        Node<T>* x = root;

        while (x != TNULL) {
            y = x;
            if (node->data < x->data) {
                x = x->left;
            } else {
                x = x->right;
            }
        }
        node->parent = y;
        if (y == nullptr) {
            root = node;
        } else if (node->data < y->data) {
            y->left = node;
        } else {
            y->right = node;
        }
        node->left = TNULL;
        node->right = TNULL;
        node->color = RED;

        fixInsert(node);
    }

public:
    RedBlackTree() {
        TNULL = new Node<T>(0);
        TNULL->color = BLACK;
        root = TNULL;
    }

    // 插入一个新节点
    void insertNode(T key) {
        insert(key);
    }

    // 打印树的内容（仅供调试）
    void printTree(Node<T>* node) {
        if (node != TNULL) {
            printTree(node->left);
            cout << node->data << " ";
            printTree(node->right);
        }
    }

    // 打印红黑树
    void print() {
        printTree(root);
        cout << endl;
    }
};

int main() {
    RedBlackTree<int> tree;
    tree.insertNode(7);
    tree.insertNode(3);
    tree.insertNode(18);
    tree.insertNode(10);
    tree.insertNode(22);
    tree.insertNode(8);
    tree.insertNode(11);

    tree.print();  // 打印树

    return 0;
}
```

### 10、算法题：二叉树最大路径和，但要求从头到尾实现（包括构造树和测试用例）

这里就使用递归的方式给大家做一个代码展示

```c
#include <iostream>
#include <climits>
#include <algorithm>
using namespace std;

// 二叉树节点结构体
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;

    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class Solution {
public:
    int maxPathSum(TreeNode* root) {
        int maxSum = INT_MIN;  // 初始化全局最大路径和为最小值，方便后续更新

        // 调用递归函数计算最大路径和
        maxPathSumHelper(root, maxSum);

        // 返回全局最大路径和
        return maxSum;
    }

private:
    // 递归函数，返回当前节点的最大路径和，同时更新全局最大路径和
    int maxPathSumHelper(TreeNode* node, int &maxSum) {
        // 如果当前节点为空，返回0
        if (node == nullptr) return 0;

        // 递归计算左子树的最大路径和，负数路径不计入
        int leftGain = max(0, maxPathSumHelper(node->left, maxSum));

        // 递归计算右子树的最大路径和，负数路径不计入
        int rightGain = max(0, maxPathSumHelper(node->right, maxSum));

        // 计算当前节点的路径和（包括当前节点和左右子树的路径）
        int currentMaxPathSum = node->val + leftGain + rightGain;

        // 更新全局最大路径和
        maxSum = max(maxSum, currentMaxPathSum);

        // 返回当前节点的最大贡献值，只能选择一个子树继续向上传递
        return node->val + max(leftGain, rightGain);
    }
};

// 主函数
int main() {
    // 构造测试用的二叉树
    TreeNode* root = new TreeNode(-10);
    root->left = new TreeNode(9);
    root->right = new TreeNode(20);
    root->right->left = new TreeNode(15);
    root->right->right = new TreeNode(7);

    // 创建 Solution 对象并调用 maxPathSum 函数
    Solution solution;
    int result = solution.maxPathSum(root);

    // 输出最大路径和
    cout << "Maximum Path Sum: " << result << endl;

    return 0;
}
```