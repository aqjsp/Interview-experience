# 腾讯提前批一面面经

> 来源：https://www.nowcoder.com/discuss/727458381765558272

## 1、线程间同步方式？

#### 互斥锁（Mutex）

用于保护共享资源，确保同一时刻只有一个线程可以访问临界区，从而防止数据竞争和不一致问题。C++11 提供了 `std::mutex` 及其变种（如 `std::recursive_mutex`、`std::timed_mutex`）。

使用方式：

- `std::mutex`： 最常用的互斥锁
- `std::lock_guard`： RAII 风格的封装，自动加锁和解锁，简化异常安全性管理。
- `std::unique_lock`： 更灵活的锁管理方式，支持延迟加锁、提前释放等操作。

示例：

```c++
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx;
int shared_data = 0;

void increment() {
    std::lock_guard<std::mutex> lock(mtx);  // RAII，自动释放锁
    for (int i = 0; i < 100000; ++i) {
        ++shared_data;
    }
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);
    t1.join();
    t2.join();
    std::cout << "Final value: " << shared_data << std::endl;  // 正确输出 200000
    return 0;
}
```

#### 条件变量（Condition Variable）

条件变量允许线程在某个条件不满足时等待，当条件满足时，通过通知机制唤醒等待的线程。它通常与互斥锁一起使用，以确保对共享数据的安全访问。

关键函数：

- `std::condition_variable::wait`：使线程等待直到条件变量被通知，同时释放关联的互斥锁。
- `std::condition_variable::notify_one` 与 `notify_all`：分别唤醒一个或所有等待线程。

生产者--消费者模型：

```c++
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <queue>

std::mutex mtx;
std::condition_variable cv;
std::queue<int> dataQueue;
bool finished = false;

void producer() {
    for (int i = 0; i < 5; ++i) {
        {
            std::lock_guard<std::mutex> lock(mtx);
            dataQueue.push(i);
            std::cout << "Produced: " << i << std::endl;
        }
        cv.notify_one(); // 通知一个等待的消费者
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
    }
    {
        std::lock_guard<std::mutex> lock(mtx);
        finished = true;
    }
    cv.notify_all();
}

void consumer() {
    while (true) {
        std::unique_lock<std::mutex> lock(mtx);
        // 等待直到队列非空或生产结束
        cv.wait(lock, []{ return !dataQueue.empty() || finished; });
        if (!dataQueue.empty()) {
            int value = dataQueue.front();
            dataQueue.pop();
            lock.unlock();
            std::cout << "Consumed: " << value << std::endl;
        } else if (finished) {
            break;
        }
    }
}

int main() {
    std::thread prod(producer);
    std::thread cons(consumer);
    prod.join();
    cons.join();
    return 0;
}
```

`cv.wait(lock, predicate)`：等待条件满足，释放锁，醒来后重新加锁。

`cv.notify_one() / notify_all()`：唤醒一个/所有等待线程。

#### 原子操作

原子操作（atomic）可以在不使用锁的情况下实现线程安全的操作，适用于简单的计数器或状态标记等场景。C++11 提供了 `std::atomic<T>` 模板类，支持原子读写和修改。

示例：无锁计数器

```c++
#include <iostream>
#include <thread>
#include <atomic>

std::atomic<int> counter(0);

void increment() {
    for (int i = 0; i < 1000; ++i) {
        counter.fetch_add(1, std::memory_order_relaxed); // 原子加
    }
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);
    t1.join();
    t2.join();
    std::cout << "Counter: " << counter << std::endl; // 输出 2000
    return 0;
}
```

#### 信号量（Semaphore）

信号量用于控制对共享资源的访问数量。在 C++20 中引入了 `std::counting_semaphore` 和 `std::binary_semaphore`。在旧版本中，可以使用 POSIX 信号量（sem_t）。

示例：限制并发线程

```c++
#include <iostream>
#include <thread>
#include <vector>
#include <semaphore>

std::counting_semaphore<2> sem(2); // 最多 2 个线程同时访问

void worker(int id) {
    sem.acquire();
    std::cout << "Thread " << id << " working\n";
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::cout << "Thread " << id << " done\n";
    sem.release();
}

int main() {
    std::vector<std::thread> threads;
    for (int i = 0; i < 5; ++i) {
        threads.emplace_back(worker, i);
    }
    for (auto& t : threads) t.join();
    return 0;
}
```

####  读写锁（Read-Write Lock）

当读操作远多于写操作时，使用读写锁可以提高并发性能。C++17 引入了 `std::shared_mutex`，允许多个线程同时读共享数据，而写操作时要求独占。

示例：读多写少

```c++
#include <iostream>
#include <thread>
#include <shared_mutex>

std::shared_mutex smtx;
int data = 0;

void reader(int id) {
    std::shared_lock<std::shared_mutex> lock(smtx);
    std::cout << "Reader " << id << " sees: " << data << std::endl;
}

void writer() {
    std::unique_lock<std::shared_mutex> lock(smtx);
    data++;
    std::cout << "Writer updated data to: " << data << std::endl;
}

int main() {
    std::vector<std::thread> threads;
    for (int i = 0; i < 5; ++i) threads.emplace_back(reader, i);
    threads.emplace_back(writer);
    for (auto& t : threads) t.join();
    return 0;
}
```

`shared_lock`：允许多个读。

`unique_lock`：独占写。

####  对比

| 方式     | 优点           | 缺点                  | 场景           |
| -------- | -------------- | --------------------- | -------------- |
| 互斥锁   | 简单，通用     | 阻塞，性能开销        | 保护共享资源   |
| 条件变量 | 条件等待，高效 | 复杂，需锁配合        | 生产者-消费者  |
| 原子操作 | 无锁，高性能   | 仅限简单操作          | 计数器、标志位 |
| 信号量   | 资源计数       | C++20，新特性支持有限 | 限流、资源池   |
| 读写锁   | 读并发高       | 实现复杂              | 读多写少       |

## 2、http和https的区别？

#### 基本定义

##### `HTTP`

超文本传输协议，是一种用于在客户端（如浏览器）和服务器之间传输数据的应用层协议。

特点：明文传输，无加密。

端口：默认使用 80。

##### `HTTPS`

安全的超文本传输协议，是 HTTP 的加密版本，通过在 HTTP 下层加入 SSL/TLS（安全套接字层/传输层安全协议）实现。

特点：加密传输，提供安全保障。

端口：默认使用 443。

#### 安全性

##### `HTTP`

- 数据以明文形式传输，任何处于传输链路中的中间人都可能窃取或篡改数据。
- 不具备身份验证机制，无法确认服务器身份。

##### `HTTPS`

- 利用 TLS（或早期的 SSL）对数据进行加密，确保数据在传输过程中被保护，防止窃听和中间人攻击。
- 通过数字证书对服务器进行身份验证，客户端可以验证服务器的真实性，从而防止钓鱼网站和伪造攻击。
- 数据完整性校验，确保数据在传输过程中未被篡改。

#### 加密机制

##### `HTTP`

无加密，仅基于 TCP 连接，建立连接后直接进行 HTTP 请求和响应。

##### `HTTPS`

- 使用 SSL/TLS 加密，建立安全通道。
- 在 TCP 连接之上，还需要经过一系列 TLS 握手过程，协商加密算法、交换密钥等，建立安全的通信信道。
- TLS 握手过程包括客户端发送 ClientHello、服务器返回 ServerHello 及证书、密钥交换、双方验证和 Finished 消息。

#### 协议栈层次

HTTP：应用层协议，直接运行在 TCP/IP 上。

HTTPS：在 HTTP 和 TCP 之间插入 SSL/TLS 层，形成安全通道。

#### 证书与信任链

- HTTP：不使用数字证书，也就没有证书链的验证机制，无法提供身份认证。
- HTTPS：
  - 需要由受信任的证书颁发机构（CA）签发数字证书，服务器在建立连接时会将证书发送给客户端。
  - 客户端会验证证书是否有效、是否被信任、证书是否与访问的域名匹配，从而建立信任关系。

## 3、http位于7层模型的哪一层？

应用层。这里讲讲ISO的七层模型吧。

#### 物理层

负责传输原始比特流，是数据通信中最底层的一层。定义了物理介质的电气、机械、过程和功能特性。

作用：将数据转换为电信号、光信号或无线信号。

常见技术：光纤、电缆、无线传输、网卡、调制解调器等。

#### 数据链路层

提供节点间可靠的数据传输，负责在物理链路上对数据帧的封装、传输、差错检测与纠正。它将物理层传输的原始比特流组装成有意义的帧，并提供物理地址（MAC 地址）。

作用：帧同步、流量控制、错误校验（如 CRC）。

常见协议：Ethernet、PPP、HDLC、Wi-Fi 等。

#### 网络层

负责数据包在不同网络间的传输与路由选择，实现逻辑地址（IP 地址）寻址。该层确定数据从源节点到目标节点的最佳路径。

作用：路径选择、分片与重组、地址解析。

常见协议：IP（IPv4/IPv6）、ICMP、IGMP、路由协议（如 OSPF、BGP 等）。

#### 传输层

提供端到端的数据传输服务，确保数据的完整性和顺序。负责分段、重组、差错校验和流量控制。传输层对上层应用提供透明的通信服务。

作用：流量控制、错误恢复、连接管理。

常见协议： TCP（传输控制协议）和 UDP（用户数据报协议）。

#### 会话层

负责建立、管理和终止应用程序之间的会话，提供对话控制和同步机制，确保在通信过程中数据的连续性和状态管理。

作用：会话恢复、同步、对话控制。

常见应用： 会话层的功能通常由一些高级协议和 API 实现，比如 RPC、SQL 连接等。现代网络中，这一层的功能有时由传输层或应用层承担。

#### 表示层

负责数据的语法和语义转换，提供数据加密、解密、压缩和解压缩功能。确保发送方和接收方能够正确理解彼此传输的数据。

作用：数据加密/解密、压缩、格式转换（如 XML 到 JSON）。

常见应用： 数据格式转换（如 ASCII 与 EBCDIC）、加密协议（如 SSL/TLS 的一部分功能）、文件格式标准等。

#### 应用层

提供网络服务和应用接口，直接为用户应用程序提供支持。包括数据展示、用户身份验证、资源请求等。

作用：数据交互、用户接口、应用程序协议。

常见协议： HTTP、FTP、SMTP、DNS、Telnet、SSH 等。

实际网络多采用 TCP/IP 模型（4 层），与 OSI 模型对应如下：

| OSI 层     | TCP/IP 层        |
| ---------- | ---------------- |
| 应用层     | 应用层           |
| 表示层     | （合并到应用层） |
| 会话层     | （合并到应用层） |
| 传输层     | 传输层           |
| 网络层     | 网络层           |
| 数据链路层 | 链路层           |
| 物理层     | （合并到链路层） |

## 4、tcp和udp的区别？

#### 连接性

TCP（传输控制协议）：在数据传输前，TCP 需要建立一个连接（三次握手），确保双方处于通信状态，通信结束时通过四次挥手释放连接。

UDP（用户数据报协议）：UDP 不需要事先建立连接，直接将数据包（称为数据报）发送到目标主机，发送前和接收后都没有握手过程。

#### 可靠性

TCP 提供面向字节流的可靠数据传输。通过序列号、确认应答（ACK）、超时重传和校验和等机制保证数据不丢失、不重复且按序到达。

UDP 是一种无连接协议，发送数据后不保证对方会收到，也不提供重传、顺序保证或数据完整性检查（应用层可自行实现校验）。

#### 数据传输方式

TCP：将数据作为连续的字节流传输，数据没有边界，接收方需要根据协议解析数据。

UDP：每个数据包都有明确的边界，消息在发送时以独立的数据报形式发送，接收方一次性接收一个完整的数据报。

#### 头部结构

TCP 头部（20 字节，选项除外）：

- 源端口、目标端口。
- 序列号、确认号。
- 窗口大小、校验和等。

UDP 头部（8 字节）：

- 源端口、目标端口。
- 长度、校验和。

## 5、讲讲tcp三次握手的原理？

#### 为什么需要三次握手？

可靠性：

- TCP 是面向连接的协议，通信前需确认双方准备就绪。
- 网络中可能存在延迟、丢包或重复包，三次握手确保连接可靠。

目标：

1. 确认双方的发送能力（能发数据）。
2. 确认双方的接收能力（能收数据）。
3. 同步序列号（Sequence Number），为后续数据传输做准备。

#### 三次握手的具体过程

```
客户端                          服务器
  |               SYN (Seq=x) --> |
  | <-- SYN+ACK (Seq=y, Ack=x+1)  |
  |    ACK (Seq=x+1, Ack=y+1) --> |
```

##### 第一次（客户端行为）

客户端发起连接请求，向服务器发送一个 SYN 包（SYN 标志置 1），并在包中包含一个初始序列号（比如称为 ISN，即 Initial Sequence Number）。

目的就是告诉服务器：“我想建立连接，这里是我的初始序列号。”

##### 第二次（服务端行为）

收到客户端的 SYN 包后，服务器确认客户端的连接请求，向客户端回复一个 SYN+ACK 包：

- SYN 标志置 1，表示服务器同意建立连接。
- ACK 标志置 1，同时确认号（ACK number）设为客户端的 ISN 加 1，表示已经收到客户端的请求。
- 同时服务器选择自己的初始序列号（服务器 ISN），作为后续数据传输的起始点。

目的 告知客户端：“我收到了你的连接请求，并且我也准备好了连接，这里是我的初始序列号。”

##### 第三次（客户端行为）

客户端收到服务器的 SYN+ACK 包后，发送一个 ACK 包给服务器：ACK 标志置 1，确认号设为服务器 ISN 加 1。

目的： 告知服务器：“我收到了你的确认信息，连接建立完成。”

#### 为什么不是两次或四次？

两次不足：

如果仅用两次握手（比如客户端发 SYN，服务器回复 SYN+ACK），可能会存在旧的重复连接请求导致混乱。第三次握手确保客户端确认了服务器的应答，避免旧的、延迟的连接请求影响当前连接。

四次冗余：三次已足够确认双向通信，多余的步骤增加开销。

## 6、长连接短连接了解吗？

#### 基本定义

##### 短连接

每一次请求/响应周期都会建立一个新的 TCP 连接，用完即断。即客户端向服务器发出请求，服务器响应后立即关闭连接。

早期的 HTTP/1.0 默认就是短连接，除非在请求头中指定 `Connection: keep-alive` 来要求保持连接。

##### 长连接

在同一个 TCP 连接上可以传输多个 HTTP 请求和响应。连接建立后保持一段时间，不会在每次请求完成后立即关闭。

HTTP/1.1 默认使用长连接，即使不显式声明 `keep-alive`，连接也会保持，直到超时或客户端/服务器主动关闭。

#### 工作原理

##### 短连接

1. 客户端发起 TCP 三次握手，建立连接。
2. 发送请求（如 HTTP GET），服务器响应。
3. 数据传输完成后，四次挥手断开连接。
4. 下次请求重复上述过程。

##### 长连接

1. 客户端发起 TCP 三次握手，建立连接。
2. 发送请求，服务器响应，连接保持打开。
3. 后续请求复用同一连接，无需重复握手。
4. 连接空闲超时或显式关闭时断开。

#### 优缺点

##### 短连接

优点：

- 每个连接结束后资源被释放。
- 无须考虑长时间连接带来的状态维护问题。

缺点：

- 每次请求都要重新进行三次握手，增加延时。
- 频繁建立和断开连接会导致 TCP/IP 协议栈反复处理连接状态，效率较低。

##### 长连接

优点：

- 只建立一次连接，多次请求共享该连接。
- 消除了多次三次握手过程，适合高频请求场景。

缺点：

- 长时间保持大量空闲连接会占用服务器资源（如文件描述符、内存等）。
- 需要处理连接超时、断线重连、心跳检测等问题。
- 长连接可能成为攻击者维持持久连接的一种手段，需要额外管理。

## 7、MySQL索引底层用什么数据结构，为什么要用B+树？

#### MySQL 索引底层数据结构

##### B+ 树（B+ Tree）：

InnoDB 存储引擎的聚簇索引和大多数二级索引都是基于 B+ 树实现的。B+ 树是一种平衡树，每个节点可以存放多个键值对，通过多路分支使得树的高度较低，从而减少磁盘 I/O 次数。

##### 哈希索引

基于哈希表实现。系统会对索引列的每个键值调用哈希函数，将其映射为一个哈希值，然后将该键值及对应的记录指针存储在哈希表中。

##### 全文索引

主要基于倒排索引（Inverted Index）实现。系统在对文本数据进行索引时，会先对文本进行分词处理，然后将每个单词（或词项）映射到包含该词的记录列表中。

##### 空间索引

用于处理地理空间数据或几何数据，常采用 R-Tree（或其变种，如 R*-Tree）结构。R-Tree 通过将多维数据划分为最小边界矩形（MBR），构建出一棵多叉树，使得每个节点存储一个空间范围。

#### 为什么要用B+树？

##### 高扇出性，降低树高

B+ 树的每个内部节点通常可以存储大量的键值和指针（在磁盘块大小限制下），这就使得树的高度非常低。树的高度低意味着从根节点到叶节点所需的磁盘读取次数较少，从而大大提高查找速度。

##### 磁盘 I/O 效率高

B+ 树设计时考虑了磁盘块（page）的大小。节点中的数据通常被存储在一个磁盘块中，当从磁盘读取一个节点时，可以一次性读取大量数据，降低随机 I/O 次数。

##### 支持范围查询

在 B+ 树中，所有数据都存放在叶子节点，并且这些叶子节点通常通过链表相连。这种设计使得范围查询非常高效：一旦定位到范围起始位置，就可以顺序扫描后续叶节点，快速读取连续的数据。

##### 内存利用率与缓存友好性

由于 B+ 树的内部节点占用较大空间且树高较低，更多的节点可以缓存于内存中，这提高了数据的访问速度。内存中缓存的 B+ 树节点能够减少磁盘访问，提升整体性能。

##### 更新与维护效率

平衡性：B+ 树始终保持平衡状态，所有叶节点到根的距离相同。这使得插入、删除和更新操作具有较为稳定的性能，不会因局部数据变化而导致性能急剧下降。

节点拆分和合并： 当数据插入或删除时，B+ 树会自动调整结构（例如节点拆分或合并）以保持树的平衡，这保证了即使在频繁更新的场景下也能维持较高的查询性能。

##### 与 B 树的区别

| 特性     | B 树                   | B+树               |
| :------- | :--------------------- | :----------------- |
| 数据存储 | 所有节点存储键和数据   | 仅叶子节点存储数据 |
| 叶子节点 | 无链表连接             | 双向链表连接       |
| 查询路径 | 数据分散，可能提前终止 | 必须到叶子节点     |
| 范围查询 | 需回溯遍历             | 顺序遍历链表       |

## 8、Redis的性能很高，它是怎么实现的，redis为什么快？

#### (1) 基于内存存储

原理：

- Redis 是一个内存数据库，默认将所有数据存储在 RAM 中。
- 内存访问速度（纳秒级）远超磁盘（毫秒级）。

实现：

- 数据以键值对（Key-Value）形式存储，直接操作内存。
- 支持持久化（RDB 和 AOF），但主要操作不涉及磁盘。

优势：相比 MySQL 等磁盘数据库，Redis 避免了 I/O 瓶颈。

#### (2) 高效的数据结构

Redis 提供多种数据结构，每种都针对特定场景优化：

字符串（String）：

- 使用 SDS（Simple Dynamic String），优于 C 字符串。
- 特性：动态分配，预留空间，减少内存重分配。

哈希表（Hash）：

- 底层用字典（dict）实现，渐进式 rehash 避免阻塞。
- 时间复杂度：O(1) 平均查找。

列表（List）：双向链表（quicklist），支持快速头尾操作。

集合（Set）：整数集合（intset）或哈希表，节省空间。

有序集合（ZSet）：跳表（skiplist）+哈希表，高效排序和查找。

为何快：

- 数据结构针对操作优化，如 ZSet 的跳表支持 O(log n) 范围查询。
- 内存管理高效，减少碎片和拷贝。

#### (3) 单线程模型

原理：Redis 主线程是单线程，处理所有客户端请求。

实现：

- 事件循环（Event Loop）顺序执行命令。
- 无锁设计，避免多线程的上下文切换和同步开销。

优势：

- 无竞争：无需互斥锁（如 mutex），节省 CPU 资源。
- 简单性：逻辑清晰，易优化。

限制：单线程依赖 CPU 单核性能，多核利用需集群。

为何快：上下文切换（约微秒级）在高并发下开销大，单线程避免此问题。

#### (4) 事件驱动与 I/O 多路复用

原理：Redis 使用 I/O 多路复用（如 epoll、kqueue）处理网络连接。

实现：

- 单线程监听多个客户端 socket。
- 事件循环基于 libevent 或自实现（如 ae.c）。
- 步骤：
  1. 注册事件（读/写）。
  2. 多路复用器（如 epoll）通知就绪事件。
  3. 处理请求并响应。

优势：

- 一个线程处理数千连接，扩展性强。
- 非阻塞 I/O，避免线程阻塞。

为何快：传统多线程模型每连接一个线程，Redis 单线程复用节省资源。

## 9、c++左值引用和右值引用？

#### 左值引用

左值（lvalue）表示具有持久存储的对象，可以出现在赋值表达式的左侧。例如变量、数组元素、解引用的指针等。

左值引用使用符号 `&` 定义。例如：`int &ref = someInt;`

左值引用必须绑定到一个左值上，不能绑定到右值上（除非使用 const 修饰）。

```c++
int a = 10;
int &lref = a; // 正确：a 是左值
lref = 20;     // 修改 a 的值，a 变成 20

// 错误示例（非 const 左值引用不能绑定到右值）
// int &badRef = 30; // 编译错误
```

##### const 左值引用

使用 `const` 修饰的左值引用可以绑定到右值上，因为它承诺不修改该值。

```c++
const int &cref = 30; // 合法：右值可以绑定到 const 左值引用
```

#### 右值引用

右值（rvalue） 通常表示临时对象或将亡值（即即将销毁的对象），它们没有持久存储。右值不能出现在赋值表达式的左侧。

右值引用使用符号 `&&` 定义。例如：`int &&rref = 20;`

右值引用主要在 C++11 中用于实现移动语义和完美转发：

移动语义：
当一个对象即将被销毁时，可以“窃取”其资源（如动态分配的内存、文件句柄等），避免不必要的深拷贝，提高性能。
例如，一个类可以定义移动构造函数和移动赋值运算符，从右值引用中窃取资源。

完美转发：
结合模板和 `std::forward`，可以将参数的值类别（左值或右值）原封不动地传递给另一个函数，确保效率与正确性。

示例：

```c++
#include <iostream>
#include <vector>

// 移动构造函数示例
class MyString {
public:
    char *data;

    // 普通构造函数
    MyString(const char *str) {
        std::cout << "Constructing MyString\n";
        if (str) {
            size_t len = strlen(str) + 1;
            data = new char[len];
            memcpy(data, str, len);
        } else {
            data = nullptr;
        }
    }

    // 拷贝构造函数
    MyString(const MyString &other) {
        std::cout << "Copy constructing MyString\n";
        if (other.data) {
            size_t len = strlen(other.data) + 1;
            data = new char[len];
            memcpy(data, other.data, len);
        } else {
            data = nullptr;
        }
    }

    // 移动构造函数
    MyString(MyString &&other) noexcept : data(other.data) {
        std::cout << "Move constructing MyString\n";
        other.data = nullptr;
    }

    ~MyString() {
        delete[] data;
    }
};

int main() {
    MyString s1("Hello, World!");
    MyString s2 = std::move(s1); // 调用移动构造函数
    std::cout << (s1.data ? s1.data : "s1 is empty") << std::endl;
    std::cout << (s2.data ? s2.data : "s2 is empty") << std::endl;
    return 0;
}
```

#### 左值引用 vs 右值引用

| 特性       | 左值引用 (T&)          | 右值引用 (T&&)     |
| ---------- | ---------------------- | ------------------ |
| 绑定对象   | 左值                   | 右值               |
| 语法       | T&                     | T&&                |
| 用途       | 别名、参数传递         | 移动语义、完美转发 |
| 修改性     | 可修改原对象           | 通常转移资源       |
| const 版本 | 可绑定右值（const T&） | 无意义             |

## 10、c++智能指针？

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

    return 0; // 自动析构
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
    return 0; // 析构
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
    return 0; // 正常析构
}
```

输出：

```c++
Destructed
Destructed
```

若 prev 用 shared_ptr，引用计数永不为 0，造成泄漏。

## 11、手撕：二叉树层序遍历

#### 解题思路

广度优先搜索（BFS）：层序遍历本质上是 BFS，使用队列（Queue）逐层访问节点。

1. 如果根节点为空，返回空结果。
2. 初始化队列，将根节点入队。
3. 循环直到队列为空：
   - 获取当前层节点数（队列大小）。
   - 遍历当前层所有节点：出队，记录节点值。将左子节点和右子节点（若存在）入队。
   - 将当前层结果加入总结果。
4. 返回结果。

#### 参考代码（C++）

```c++
#include <vector>
#include <queue>

struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
};

class Solution {
public:
    std::vector<std::vector<int>> levelOrder(TreeNode* root) {
        // 结果容器，按层存储节点值
        std::vector<std::vector<int>> result;

        // 根节点为空，直接返回
        if (root == nullptr) {
            return result;
        }

        // 队列，用于 BFS
        std::queue<TreeNode*> q;
        q.push(root); // 根节点入队

        // 当队列不为空，继续处理
        while (!q.empty()) {
            // 获取当前层节点数
            int levelSize = q.size();
            // 当前层的值
            std::vector<int> currentLevel;

            // 遍历当前层所有节点
            for (int i = 0; i < levelSize; ++i) {
                // 取出队首节点
                TreeNode* node = q.front();
                q.pop();

                // 记录节点值
                currentLevel.push_back(node->val);

                // 将左右子节点入队（若存在）
                if (node->left) {
                    q.push(node->left);
                }
                if (node->right) {
                    q.push(node->right);
                }
            }

            // 当前层处理完毕，加入结果
            result.push_back(currentLevel);
        }

        return result;
    }
};

void printResult(const std::vector<std::vector<int>>& result) {
    for (const auto& level : result) {
        std::cout << "[";
        for (int val : level) {
            std::cout << val << " ";
        }
        std::cout << "]\n";
    }
}

int main() {
    // 创建测试树：[3,9,20,null,null,15,7]
    TreeNode* root = new TreeNode(3);
    root->left = new TreeNode(9);
    root->right = new TreeNode(20);
    root->right->left = new TreeNode(15);
    root->right->right = new TreeNode(7);

    Solution solution;
    auto result = solution.levelOrder(root);
    printResult(result);

    // 清理内存（手动释放）
    delete root->right->right;
    delete root->right->left;
    delete root->right;
    delete root->left;
    delete root;

    return 0;
}
```

