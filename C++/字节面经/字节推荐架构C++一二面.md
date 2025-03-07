# 字节推荐架构C++一面，各种拷打时间复杂度，喜提转组，开始下一轮拷打。。。

> 来源：https://www.nowcoder.com/discuss/709786059567013888

### 一面

## 1、智能指针

智能指针

## 2、手写一个shared_ptr

shared_ptr的核心是引用计数机制，确保对象在最后一个指针销毁时自动释放。实现需要：

- 一个控制块（ControlBlock）存储对象的指针和引用计数。
- shared_ptr类管理构造、拷贝、移动和销毁操作。
- 支持空指针和异常安全。

```c++
template<typename T>
struct ControlBlock {
    int ref_count;
    T* ptr;
};

template<typename T>
class shared_ptr {
private:
    ControlBlock<T>* ctrl;
public:
    // 默认构造函数，初始化为空
    shared_ptr() : ctrl(nullptr) {}

    // 从原始指针构造，创建新控制块
    shared_ptr(T* p) {
        if (p == nullptr) {
            ctrl = nullptr;
        } else {
            ctrl = new ControlBlock<T>{1, p};
        }
    }

    // 拷贝构造函数，增加引用计数
    shared_ptr(const shared_ptr& other) {
        if (other.CTRL == nullptr) {
            ctrl = nullptr;
        } else {
            ctrl = other.CTRL;
            ctrl->ref_count++;
        }
    }

    // 移动构造函数，转移所有权
    shared_ptr(shared_ptr&& other) noexcept {
        ctrl = other.CTRL;
        other.CTRL = nullptr;
    }

    // 析构函数，减少引用计数，计数为0时释放
    ~shared_ptr() {
        if (ctrl != nullptr) {
            ctrl->ref_count--;
            if (ctrl->ref_count == 0) {
                delete ctrl->ptr;
                delete ctrl;
            }
        }
    }

    // 拷贝赋值运算符，使用拷贝-交换习惯用法
    shared_ptr& operator=(const shared_ptr& other) {
        if (this != &other) {
            shared_ptr temp(other);
            std::swap(ctrl, temp.CTRL);
        }
        return *this;
    }

    // 移动赋值运算符，直接转移
    shared_ptr& operator=(shared_ptr&& other) noexcept {
        if (this != &other) {
            if (ctrl != nullptr) {
                ctrl->ref_count--;
                if (ctrl->ref_count == 0) {
                    delete ctrl->ptr;
                    delete ctrl;
                }
            }
            ctrl = other.CTRL;
            other.CTRL = nullptr;
        }
        return *this;
    }

    // 获取原始指针
    T* get() const {
        return (ctrl != nullptr) ? ctrl->ptr : nullptr;
    }

    // 解引用运算符
    T& operator*() const {
        if (ctrl == nullptr || ctrl->ptr == nullptr) {
            throw std::runtime_error("Null pointer");
        }
        return *ctrl->ptr;
    }

    // 箭头运算符
    T* operator->() const {
        if (ctrl == nullptr || ctrl->ptr == nullptr) {
            throw std::runtime_error("Null pointer");
        }
        return ctrl->ptr;
    }

    // 比较运算符
    bool operator==(const shared_ptr& other) const {
        return get() == other.get();
    }

    bool operator!=(const shared_ptr& other) const {
        return get() != other.get();
    }

    // 获取引用计数
    int use_count() const {
        if (ctrl == nullptr) {
            return 0;
        }
        return ctrl->ref_count;
    }

    // 检查是否唯一
    bool unique() const {
        if (ctrl == nullptr) {
            return true;
        }
        return ctrl->ref_count == 1;
    }
};

int main() {
    shared_ptr<int> sp1(new int(5));
    std::cout << "sp1 use_count: " << sp1.use_count() << std::endl; // 输出 1
    shared_ptr<int> sp2 = sp1;
    std::cout << "sp2 use_count: " << sp2.use_count() << std::endl; // 输出 2
    sp1 = nullptr;
    std::cout << "sp2 use_count after sp1=nullptr: " << sp2.use_count() << std::endl; // 输出 1
    return 0;
}
```

但是需要注意的是：

1. 避免从同一原始指针创建多个shared_ptr，可能导致双重释放。
2. 循环引用需配合weak_ptr解决。
3. 确保析构函数正确，防止资源泄漏。

## 3、TCP UDP区别？

TCP注重可靠性，适合数据完整性要求高的场景；UDP注重速度，适合实时性要求高的应用。

#### 基本概念

TCP：连接导向的传输层协议，确保数据可靠、有序传输。

UDP：无连接的传输层协议，快速但不保证交付。

两者均位于OSI模型的第四层（传输层），基于IP协议。

#### 连接方式

- TCP：
  - 使用三次握手建立连接：客户端发送SYN，服务器回复SYN-ACK，客户端确认ACK。
  - 确保双方收发能力正常，初始化序列号。
  - 示例：浏览器访问[Google](https://www.google.com)，先建立TCP连接。
- UDP：
  - 无连接，直接发送数据报，无需预先通信。
  - 每个数据报独立，接收端不需确认连接。
  - 示例：DNS查询用UDP，快速解析域名如[Cloudflare DNS](https://1.1.1.1)。

#### 可靠性

- TCP：
  - 保证数据按序到达，通过序列号和确认机制。
  - 若数据丢失，触发重传（如超时或ACK未收到）。
  - 拥塞控制（如慢启动）防止网络过载。
- UDP：
  - 不保证交付，数据可能丢失或失序。
  - 无重传机制，接收端需自行处理（如应用层重发）。
  - 适合实时应用，少量丢失可接受。

#### 数据传输

- TCP：
  - 流式传输，数据分段后连续发送，接收端重组。
  - 无消息边界，需应用层解析（如HTTP分隔请求）。
- UDP：
  - 数据报传输，每个数据报独立，保持消息边界。
  - 接收端不保证顺序，需应用层排序（如RTP视频帧）。

#### 错误处理

- TCP：
  - 使用校验和检测错误，错误数据重传。
  - 提供可靠传输，适合数据完整性要求高场景。
- UDP：
  - 有校验和但不重传，错误数据丢弃。
  - 适合实时性高、丢包可接受的场景。

#### 速度与效率

- TCP：
  - 因连接管理、ACK和重传，速度较慢。
  - 适合大文件传输，吞吐量高。
- UDP：
  - 无连接开销，头部仅8字节（TCP20字节），速度快。
  - 适合低延迟场景，如游戏服务器。

#### 使用场景

- TCP：
  - 网页浏览：HTTP/HTTPS，确保页面完整加载。
  - 邮件：SMTP/IMAP，需可靠交付。
  - 文件传输：FTP，数据完整性关键。
- UDP：
  - 在线游戏：实时性高，少量丢包可接受。
  - 视频流：如YouTube实时流，延迟敏感。
  - VoIP：语音通信，丢包影响小。
  - DNS：查询快速，数据小。

#### 表格区别

| 特性     | TCP                  | UDP                        |
| -------- | -------------------- | -------------------------- |
| 连接     | 连接导向，三次握手   | 无连接                     |
| 可靠性   | 保证交付，按序，重传 | 不保证，失序可能，丢包可能 |
| 传输方式 | 流式，连续           | 数据报，独立               |
| 错误处理 | 校验+重传            | 校验，无重传               |
| 头部开销 | 20字节+              | 8字节                      |
| 拥塞控制 | 有，慢启动等         | 无，应用层处理             |
| 典型应用 | HTTP, FTP, SMTP      | DNS, 游戏, 视频流          |

## 4、堆和栈的区别？

#### 定义

##### 堆（Heap）

 堆是程序运行时动态分配的内存区域，由程序员手动管理。可以通过new（C++）或malloc（C）申请内存，使用delete或free释放内存。堆适合存储需要灵活控制生命周期的数据。

##### 栈（Stack）

 栈是自动管理的内存区域，遵循“先进后出”（LIFO, Last In First Out）的原则，由编译器负责分配和释放。主要用于存储局部变量、函数参数和函数调用信息。

#### 特点

##### 堆（Heap）

手动管理：程序员需要显式分配和释放内存，容易出现内存泄漏或悬空指针问题。

生命周期：内存从分配到释放由程序员控制，适合长期存在的数据。

分配速度：较慢，因为需要操作系统介入寻找可用内存块。

内存布局：非连续，频繁分配和释放可能导致内存碎片。

线程安全：多线程访问堆内存时需要同步机制（如锁）。

##### 栈（Stack）

自动管理：编译器自动分配和释放内存，无需程序员干预。

生命周期：内存随函数作用域的开始和结束自动分配和释放，适合临时数据。

分配速度：极快，仅需调整栈指针即可完成分配。

内存布局：连续存储，无碎片问题。

线程安全：每个线程拥有独立的栈，天然线程安全。

#### 核心区别

| 特性     | 堆（Heap）             | 栈（Stack）                 |
| -------- | ---------------------- | --------------------------- |
| 管理方式 | 手动（new/delete）     | 自动（编译器管理）          |
| 分配速度 | 慢（操作系统分配）     | 快（移动栈指针）            |
| 生命周期 | 手动控制，存活至释放   | 自动，随作用域结束释放      |
| 内存布局 | 非连续，易产生碎片     | 连续，无碎片                |
| 线程安全 | 需要同步机制           | 线程私有，无需同步          |
| 大小限制 | 较大（受物理内存限制） | 较小（操作系统设定，如8MB） |
| 溢出风险 | 内存不足（分配失败）   | 栈溢出（递归过深或大变量）  |
| 典型应用 | 动态对象、复杂数据结构 | 局部变量、函数调用          |

#### 代码示例

```c++
#include <iostream>

void func() {
    // 栈上分配
    int stackVar = 10;         // 局部变量，自动分配在栈上
    // 堆上分配
    int* heapPtr = new int(20); // 动态分配在堆上
    std::cout << "栈变量: " << stackVar << ", 堆变量: " << *heapPtr << "\n";
    delete heapPtr;            // 手动释放堆内存
    // stackVar 在函数结束时自动释放
}

int main() {
    func();
    return 0;
}
```

#### 优缺点

##### 堆（Heap）

- 优点：
  - 灵活性高，支持运行时大小不确定的对象。
  - 可分配大块内存，适合动态数据结构（如链表、树）。
  - 支持跨函数或线程共享。
- 缺点：
  - 管理复杂，容易发生内存泄漏或悬空指针。
  - 分配速度慢，内存碎片可能降低性能。

##### 栈（Stack）

- 优点：
  - 分配和释放高效，性能优越。
  - 自动管理，无需担心内存泄漏。
  - 内存连续，线程安全。
- 缺点：
  - 大小受限（通常几MB），不适合大对象。
  - 生命周期固定，无法长期存储数据。

#### 使用场景

##### 堆（Heap）

- 动态数据结构：如动态数组（std::vector）、链表、树等。
- 运行时大小未知的对象：如用户输入决定的大小。
- 共享资源：多线程或跨函数访问的对象。

##### 栈（Stack）

- 局部变量：函数内的临时变量。
- 函数调用：存储参数、返回地址和调用帧。
- 递归调用：栈保存每次调用的状态（注意栈溢出风险）。

## 5、最新的http协议基于什么？

最新的 HTTP 协议版本是 HTTP/3。与之前的版本不同，HTTP/3 不再基于传输控制协议（TCP），而是构建在用户数据报协议（UDP）之上，采用了快速 UDP 互联网连接（QUIC）协议作为其传输层。

##### HTTP/3 与 QUIC 协议：

- QUIC 协议： 由谷歌开发，旨在提供更快、更可靠的互联网连接。QUIC 集成了传输层和安全层功能，利用 UDP 实现了类似于 TCP 的可靠传输，同时减少了连接建立时间。
- HTTP/3： 作为 HTTP 协议的最新版本，HTTP/3 使用 QUIC 作为其传输层协议，取代了之前版本中的 TCP。

##### HTTP/3 的主要优势：

1. 减少连接建立时间： QUIC 将传输层和安全层的握手过程合并，减少了连接建立所需的往返时间（RTT），从而加快了连接速度。
2. 消除队头阻塞： 在 TCP 中，如果一个数据包丢失，后续的数据包必须等待丢失的数据包被重新传输后才能处理，这会导致队头阻塞。QUIC 通过在应用层实现多路复用，确保单个数据流的丢包不会影响其他数据流的传输，提升了传输效率。
3. 支持连接迁移： QUIC 使用连接 ID 来标识连接，使其能够在客户端 IP 地址变化（例如从 Wi-Fi 切换到蜂窝数据）时保持连接的连续性，无需重新建立连接。

## 6、select和poll的区别？select支持的最大数量？

#### 1. select

##### 工作原理与数据结构：

- 数据结构：使用 `fd_set` 位图来保存所有待监听的文件描述符。在调用前需要用 `FD_SET` 将感兴趣的描述符添加到集合中。
- 调用方式：调用时会传入读、写、异常的 `fd_set` 集合，内核返回时会修改这些集合，保留就绪的文件描述符。每次调用前都必须重新设置集合。

##### 性能特点：

- 遍历方式：内核每次调用 select 都需要遍历整个 `fd_set` 数组（其大小固定为 FD_SETSIZE），因此时间复杂度为 O(n)。
- 适用场景：适用于文件描述符数量较少的场景，但在大并发场景下性能较差。

##### 最大支持的文件描述符数量：

- 限制：受限于宏定义 `FD_SETSIZE`，在大多数系统中默认值为 1024。这意味着最多只能监控 1024 个文件描述符（实际数值可能还需要预留一些系统使用的描述符）。
- 调整方法：可以在编译前通过 `#define FD_SETSIZE 2048` 等方式增大该值，但这样做可能会带来性能和兼容性问题。

#### 2. poll

##### 工作原理与数据结构：

- 数据结构：使用 `struct pollfd` 的数组，每个结构体保存一个文件描述符及其所关注的事件（如 POLLIN、POLLOUT）和内核返回的就绪事件（revents）。
- 调用方式：传入 pollfd 数组后，内核遍历该数组，检测每个描述符的状态，并在返回时设置相应的 `revents` 字段。

##### 性能特点：

- 遍历方式：实现同样是遍历传入的数组，时间复杂度为 O(n)；不过由于数组的长度是动态的（只包含实际需要监控的描述符），在某些情况下比 select 更灵活。
- 适用场景：适用于文件描述符数量较多的场景，但随着监控数量增多，性能依然会受到遍历开销的影响。

##### 最大支持的文件描述符数量：

- 限制：本身没有像 FD_SETSIZE 那样的固定上限，其最大支持的文件描述符数量主要受限于系统资源（如内存）以及操作系统对单个进程最大打开文件数（ulimit -n）的限制。

#### 3. epoll

##### 工作原理与数据结构：

- 数据结构：是 Linux 特有的 I/O 多路复用机制，通过创建一个 epoll 实例（文件描述符）来管理多个待监听的文件描述符。
- 调用方式：通过 `epoll_ctl` 添加、修改或删除监听的文件描述符，然后使用 `epoll_wait` 等待事件。内核通过事件通知机制将就绪事件传递给用户态。
- 触发模式：支持水平触发（Level Triggered）和边缘触发（Edge Triggered），边缘触发模式可以在高并发场景下进一步减少系统调用次数。

##### 性能特点：

- 时间复杂度：在事件发生时只返回实际就绪的文件描述符，时间复杂度接近 O(1)（与监听的文件描述符总数无关），非常适合大量文件描述符的场景。
- 适用场景：特别适合大规模并发连接（比如高并发的网络服务器），性能远超 select 和 poll。

##### 最大支持的文件描述符数量：

- 限制：本身没有固定的文件描述符上限，理论上可以支持非常大量的描述符，最大数量取决于系统允许的最大打开文件数（ulimit -n）。例如，在 Linux 系统中，ulimit -n 可能默认是 1024 或 4096，但可以通过系统配置和内核参数进行调整，从而支持上万个甚至更多的文件描述符。

## 7、文件句柄的底层实现？

文件句柄（file handle）是程序与操作系统交互的桥梁，表示打开的文件、管道、套接字等资源。在 POSIX 系统中，文件句柄通常表现为文件描述符（file descriptor），一个非负整数，由内核分配和管理。用户通过文件描述符调用系统函数（如 read、write、close）操作资源，内核则负责底层实现。

#### 底层实现步骤与机制

##### 文件描述符的分配

- 用户空间调用：
  - 程序通过系统调用 open（如 int fd = open("file.txt", O_RDONLY);）请求打开文件。
  - open 返回一个文件描述符（整数，例如 3），作为文件句柄。
- 内核处理：
  - 内核在当前进程的文件描述符表中分配一个空闲条目。
  - 文件描述符表是每个进程私有的数组，索引为文件描述符值，内容指向文件表条目。
  - 示例：若进程已有标准输入（0）、输出（1）、错误（2），新文件描述符从 3 开始。

##### 内核数据结构

文件句柄的底层实现依赖三个核心数据结构：

- 文件描述符表（File Descriptor Table）：
  - 位置：进程的进程控制块（PCB，如 Linux 的 task_struct）中。
  - 结构：数组，每个条目包含指向文件表的指针和标志（如 O_CLOEXEC）。
  - 作用：将文件描述符（如 3）映射到文件表条目。
  - 大小：受系统限制（如 Linux 默认 1024，可通过 ulimit 或 setrlimit 调整）。
- 文件表（File Table）：
  - 位置：内核全局维护，多个进程可共享。
  - 结构：每个条目包含文件状态（如读写偏移量、打开模式）和指向 inode 表的指针。
  - 作用：管理文件的操作状态，例如当前读写位置（offset）。
  - 实例：若两个进程打开同一文件，可能共享文件表条目，但偏移量独立。
- inode 表（Inode Table）：
  - 位置：内核维护，通常对应文件系统（如 ext4）的 inode。
  - 结构：包含文件元数据（如大小、权限）和数据块指针。
  - 作用：链接到物理文件数据，处理实际 I/O。
  - 实例：磁盘上的文件通过 inode 表示，文件表通过 inode 访问数据。

##### 系统调用与 I/O 操作

- 读操作（如 read(fd, buffer, size)）：
  1. 用户调用 read，传递文件描述符 fd。
  2. 内核检查文件描述符表，找到对应的文件表条目。
  3. 从文件表获取偏移量和 inode 指针。
  4. 通过 inode 读取文件数据到缓冲区，更新偏移量。
  5. 返回读取字节数。
- 写操作（如 write(fd, buffer, size)）：
  - 类似流程，数据从缓冲区写入文件，更新 inode 和文件表。
- 关闭操作（如 close(fd)）：
  1. 内核释放文件描述符表中的条目。
  2. 减少文件表条目的引用计数，若为 0，则释放。
  3. 若 inode 引用计数为 0，释放 inode 或标记为可重用。

##### 特殊资源处理

- 文件句柄不仅限于文件，还包括：
  - 管道：文件描述符指向内核缓冲区，而非磁盘 inode。
  - 套接字：文件描述符关联网络协议栈（如 TCP/IP）。
  - 设备文件：指向设备驱动程序（如 /dev/tty）。
- 实现上，内核通过 struct file_operations 定义不同资源的操作函数。

## 8、红黑树？

红黑树（Red-Black Tree）是一种自平衡二叉搜索树，它通过节点颜色（红或黑）和五条性质保持平衡，确保树高不超过 2log(n+1)，从而保证查找、插入和删除的时间复杂度为 O(log n)。在 C++ 中，STL 的关联容器（如 std::map 和 std::set）使用红黑树实现。

#### 红黑树的性质

红黑树通过以下五条性质维持平衡：

1. 节点颜色：每个节点是红色或黑色。
2. 根节点：根节点始终为黑色。
3. 叶子节点（NIL）：所有叶子节点（空节点）为黑色。
4. 红色规则：红色节点的子节点必须为黑色（即无连续红色节点）。
5. 黑色高度：从任一节点到其叶子节点的所有路径包含相同数量的黑色节点。

这些性质确保树的高度平衡，防止退化为线性结构。

## 9、算法题：二叉树最大路径和，但要求从头到尾实现（包括构造树和测试用例）

```c++
#include <iostream>
#include <algorithm>
#include <limits>
using namespace std;

// 定义二叉树节点结构
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    // 构造函数（C++98 写法，使用 NULL 而非 nullptr）
    TreeNode(int x) : val(x), left(NULL), right(NULL) {}
};

// 求二叉树最大路径和的类实现
class Solution {
public:
    // 主函数：返回二叉树中的最大路径和
    int maxPathSum(TreeNode* root) {
        // 使用 int 的最小值初始化最大和
        int max_sum = std::numeric_limits<int>::min();
        // 递归计算每个节点贡献的最大和
        maxGain(root, max_sum);
        return max_sum;
    }
    
private:
    // 递归函数，返回以 node 为起点沿单边延伸时能获得的最大贡献值
    int maxGain(TreeNode* node, int &max_sum) {
        if (node == NULL) {
            return 0;
        }
        // 递归计算左子树和右子树的最大贡献值
        // 如果贡献为负，则舍弃（即取 0）
        int left_gain = max(maxGain(node->left, max_sum), 0);
        int right_gain = max(maxGain(node->right, max_sum), 0);
        
        // 经过当前节点的最大路径和
        int current_path_sum = node->val + left_gain + right_gain;
        
        // 更新全局最大值
        max_sum = max(max_sum, current_path_sum);
        
        // 返回节点沿单边延伸时的最大贡献值
        return node->val + max(left_gain, right_gain);
    }
};

// 辅助函数：释放二叉树占用的内存
void deleteTree(TreeNode* root) {
    if (root == NULL) return;
    deleteTree(root->left);
    deleteTree(root->right);
    delete root;
}

int main() {
    // 测试用例 1：构造二叉树 [1, 2, 3]
    //       1
    //      / \
    //     2   3
    TreeNode* root1 = new TreeNode(1);
    root1->left = new TreeNode(2);
    root1->right = new TreeNode(3);
    
    Solution sol;
    cout << "测试用例 1 最大路径和为: " << sol.maxPathSum(root1) << endl; // 期望结果：6 (2+1+3)
    
    // 清理内存
    deleteTree(root1);
    
    // 测试用例 2：构造二叉树 [-10, 9, 20, NULL, NULL, 15, 7]
    //         -10
    //         /  \
    //        9   20
    //            /  \
    //           15   7
    TreeNode* root2 = new TreeNode(-10);
    root2->left = new TreeNode(9);
    root2->right = new TreeNode(20);
    root2->right->left = new TreeNode(15);
    root2->right->right = new TreeNode(7);
    
    cout << "测试用例 2 最大路径和为: " << sol.maxPathSum(root2) << endl; // 期望结果：42 (15+20+7 或 -10+9+20+15+7 取较大者)
    
    // 清理内存
    deleteTree(root2);
    
    return 0;
}
```