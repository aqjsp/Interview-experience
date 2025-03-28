# 字节剪映C++一面

> 来源：https://www.nowcoder.com/feed/main/detail/ad15136a02c54c2ebe6224bf2469c995

## 1、tcp和udp区别？

#### 连接性

TCP是面向连接的协议，在数据传输前需要通过三次握手建立连接，确保双方都准备好进行通信。

UDP则是无连接的协议，发送数据之前不需要建立连接，它直接将数据包发送到目标地址，不保证对方能够收到数据。

#### 可靠性

TCP提供可靠的数据传输服务，包括错误检测、重传丢失的数据包、数据包排序等机制来保证数据的完整性和顺序。

UDP不提供可靠性保证，如果一个数据包在网络中丢失或损坏，UDP不会自动重发这个数据包，也不保证数据到达的顺序。

#### 数据处理方式

TCP是面向字节流的协议，应用程序通过TCP发送的数据被视为连续的字节流，接收方需要重新组装这些字节流为原始的消息格式。

UDP是面向报文的协议，每个UDP数据报都是独立的实体，接收方接收到的就是发送方发送的那个完整的数据报，没有拆分或重组的概念。

#### 通信模式

TCP支持一对一的点对点通信，即一次只能在两个端点之间建立一条连接。

UDP支持更灵活的通信模式，包括一对一、一对多、多对一和多对多的通信。

#### 报文头部大小

TCP 的报文首部至少有20个字节，用于包含序列号、确认号等信息。

UDP 的报文首部只有8个字节，主要用于携带源端口、目的端口等基本信息。

## 2、tcp怎么保证可靠传输？

#### 连接管理

TCP是面向连接的协议，在数据传输之前必须通过三次握手建立连接，并在结束时通过四次挥手来终止连接。这确保了双方都准备好进行通信。

#### 序列号和确认应答

每个字节的数据都被分配了一个序列号，这允许接收方将接收到的数据包按正确的顺序重组，并且可以识别重复的数据包。当接收方成功接收到一个数据包后，它会发送一个确认应答（ACK），告知发送方已成功接收该数据包。

#### 校验和

TCP报文段包含一个校验和字段，用于检测数据在传输过程中是否发生了任何改变或损坏。发送方计算校验和并将之放入报文段中，接收方对接收到的数据再次计算校验和并对比。如果结果不一致，则丢弃该数据包。

#### 超时重传

为了处理丢失的数据包，TCP实现了超时重传机制。如果发送方在一个设定的时间内没有收到某个数据包的ACK，它就会重新发送那个数据包。这个时间间隔被称为重传超时（RTO），并且通常基于往返时间（RTT）进行调整。

#### 流量控制

TCP使用滑动窗口机制来进行流量控制。接收方通知发送方其当前的接收窗口大小，即接收方还有多少缓冲区空间可用于接收新的数据。这样，发送方就不会发送超出接收方处理能力的数据量，从而避免了缓冲区溢出导致的数据丢失。

#### 拥塞控制

为了避免网络过载，TCP实施了拥塞控制策略。这包括慢启动、拥塞避免、快重传和快恢复等算法。这些算法共同作用，根据网络状况动态调整发送窗口大小，以减少网络拥堵的风险。

#### 快速重传

除了等待超时重传来应对丢失的数据包，TCP还支持快速重传机制。当发送方连续收到三个重复的ACK时，这意味着中间的一个数据包可能已经丢失，即使还没有达到重传超时时间，发送方也会立即重传该丢失的数据包。

#### 数据排序

由于网络延迟或路由选择的不同，数据包可能会以不同于它们被发送出去的顺序到达。TCP负责对接收到的数据包按照序列号进行排序，然后再传递给上层应用。

## 3、http2.0和3.0区别？

#### 底层传输协议

##### HTTP/2.0

- 基于 TCP 协议。
- TCP 提供可靠传输，但当网络中发生丢包时，会触发 TCP 的重传机制，这可能导致整个连接中的数据都受到影响（即所谓的“队头阻塞”问题）。

##### HTTP/3.0

- 基于 QUIC 协议，QUIC 是运行在 UDP 上的传输协议。
- QUIC 在用户态实现了传输控制逻辑，包括拥塞控制和重传，能够避免 TCP 中的队头阻塞问题，尤其在丢包环境下表现更佳。

#### 连接建立与握手过程

##### HTTP/2.0

- 连接建立依赖于 TCP 三次握手，然后再进行 TLS 握手（如果使用 HTTPS），这增加了延时。
- TLS 握手与 TCP 握手是两个独立的过程，通常需要多个往返时间（RTT）。

##### HTTP/3.0

- QUIC 将连接建立和 TLS 握手结合在一起，通过 0-RTT 或 1-RTT 连接建立过程，大幅降低了连接建立的延迟。
- 这种集成式的握手方式使得在复用连接和重连时速度更快。

#### 多路复用与队头阻塞

##### HTTP/2.0

- 支持多路复用，可以在一个 TCP 连接中并行传输多个流。
- 但由于底层仍是 TCP，当一个数据包丢失时，TCP 的严格顺序交付会导致所有流都受到影响，出现队头阻塞现象。

##### HTTP/3.0

由于 QUIC 是基于 UDP 实现的，它在传输层直接支持多路复用，每个流相互独立，不会因为某个流的丢包而阻塞其他流，从而消除了队头阻塞问题。

#### 头部压缩机制

##### HTTP/2.0

- 使用 HPACK 算法对头部信息进行压缩。
- HPACK 通过静态和动态表存储重复出现的头部字段，有效减少冗余数据的传输。

##### HTTP/3.0

使用 QPACK 算法，这是一种为多路复用环境设计的头部压缩算法，改进了 HPACK 在队头阻塞环境下可能出现的问题，同时保持了压缩效率。

#### 安全性

HTTP/2.0：通常与TLS一起使用来提供加密通信，但并不是强制性的。

HTTP/3.0：集成了TLS 1.3，提供了更强的安全性和隐私保护，默认情况下所有连接都是加密的。

## 4、get和post区别？

#### 语义和用途

##### GET 请求

- 目的：用于向服务器请求资源或数据，主要用于获取信息，而不对服务器上的资源进行修改。
- 幂等性和安全性：GET 请求是幂等的和安全的，即多次发送同一 GET 请求应该返回相同结果，而且不会产生副作用（如修改数据）。

##### POST 请求

- 目的：用于向服务器提交数据（例如表单数据、文件上传等），通常会引起服务器上的资源变化（如创建或更新数据）。
- 非幂等：POST 请求不是幂等的，重复提交可能导致重复创建记录或多次执行相同操作。

#### 数据传输方式

##### GET 请求

- 参数传递：参数通过 URL 的查询字符串传递，如 `http://example.com/api?param1=value1&param2=value2`。
- 数据大小限制：由于 URL 长度限制（各浏览器和服务器的限制不同，通常在 2KB 到 8KB 之间），GET 请求适合传输少量数据。
- 无请求体：GET 请求通常没有请求体（Body）。

##### POST 请求

- 参数传递：数据放在请求体中，可以传输大块数据，格式可以是表单编码（`application/x-www-form-urlencoded`）、多部分表单数据（`multipart/form-data`）、JSON 等。
- 数据量大：没有 URL 长度限制，适合上传大文件或大量数据。

#### 缓存与书签

##### GET 请求

- 缓存：GET 请求可以被浏览器、代理服务器缓存，有助于提高性能和降低服务器压力。
- 书签：URL 中包含所有请求参数，可以直接存储为书签，方便用户再次访问相同内容。

##### POST 请求

- 缓存：POST 请求默认不被缓存，因为其结果通常会改变服务器状态。
- 书签：由于参数不在 URL 中，无法通过书签方式保存请求。

#### 安全性

##### GET 请求

暴露数据： 参数直接附加在 URL 上，可能被日志记录、浏览器历史记录和缓存中保存，因此对于敏感数据（如密码、信用卡信息）不适合使用 GET。

##### POST 请求

相对安全： 数据放在请求体中，不会直接显示在 URL 上，虽然这并不意味着数据已加密（传输中仍需 HTTPS 保证安全），但可以减少在 URL 中暴露敏感信息的风险。

## 5、页面置换算法？

#### 最佳置换算法

它选择未来最长时间内不会被访问的页面进行替换。这种算法可以保证获得最低的缺页率，但它是理想化的，因为无法预知未来的页面访问模式。

#### 先进先出算法（FIFO）

最简单的页面置换算法。它根据页面进入内存的时间顺序来选择被淘汰的页面，即最早进入内存的页面最先被淘汰。虽然实现简单，但由于它不考虑页面的实际使用情况，可能导致性能不佳，并且可能出现Belady异常——增加分配给进程的物理块数反而导致缺页次数增加的情况。

#### 最近最久未使用算法（LRU）

基于程序访问的时间局部性原理，选择最近最长时间未被使用的页面进行淘汰。

常见的实现方式：

- 使用链表或栈记录页面访问顺序，每次访问将页面移到链表头部。
- 采用哈希表加双向链表来实现时间复杂度为 O(1) 的操作。

#### 时钟页面置换算法

也称为最近未使用（NRU）算法，这是一种LRU算法的近似实现。每个页面都有一个引用位，当页面被访问时，该位置1。在替换页面时，从当前指针指向的位置开始搜索，直到找到一个引用位为0的页面进行替换。如果所有页面的引用位都是1，则将它们全部置0并重新开始搜索。

#### 最少使用置换算法

记录每个页面的访问频率，选择访问次数最少的页面进行替换。这种方法可能会保留那些曾经频繁访问但现在很少使用的页面，因此有时会采用移位寄存器来计算指数衰减的平均使用计数以改进这一问题。

## 6、互斥锁和信号量区别？

#### 互斥锁（Mutex）

一种用于保护共享资源的同步机制，它确保在同一时间内只有一个线程可以访问临界区（共享资源）。

互斥锁具有所有权的概念，即只有持有该锁的线程才能解锁，其他线程必须等待。

##### 特点

独占性：互斥锁一次只能被一个线程持有，其它试图获取该锁的线程会被阻塞，直到锁被释放。

所有权：线程在加锁后成为该锁的所有者，只有拥有所有权的线程才能释放锁。这有助于防止误释放和死锁问题。

用途：通常用于保护共享变量、数据结构等需要原子性操作的场景，防止数据竞争和并发修改带来的错误。

##### 示例代码

```c++
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx;
int counter = 0;

void increment() {
    for (int i = 0; i < 1000; ++i) {
        mtx.lock();           // 加锁
        counter++;            // 临界区
        mtx.unlock();         // 解锁
    }
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);
    t1.join();
    t2.join();
    std::cout << "Counter: " << counter << std::endl; // 2000
    return 0;
}
```

#### 信号量（Semaphore）

一种更通用的同步机制，用于控制对有限资源的访问。它内部维护一个计数器，表示可用资源的数量。

信号量可以看作是一种计数器，它允许多个线程同时访问共享资源，但总数不能超过设定的上限。

##### 特点

计数机制：信号量有一个内部计数器，每当一个线程成功获得信号量时，计数器减 1；释放信号量时，计数器加 1。计数器值反映了当前可用资源的数量。

无所有权概念：信号量不具备所有权概念，任何线程都可以释放信号量，即使它不是最初获取信号量的线程。这在某些场景下需要小心使用，以免引起逻辑错误。

用途：常用于控制对固定数量资源（如连接池、缓冲区、数据库连接）的并发访问。也可以用来实现生产者-消费者模式，或对并发线程数量进行限制。

##### 基本操作

- P操作（wait/signal down）：减少信号量的计数值。如果计数值大于零，则操作成功；如果计数值为零，则调用线程将被阻塞，直到有其他线程增加信号量的值。
- V操作（signal up）：增加信号量的计数值，并唤醒一个正在等待的线程（如果有）。

##### 示例代码

C++20 `std::counting_semaphore`

```c++
#include <iostream>
#include <thread>
#include <vector>
#include <semaphore>

std::counting_semaphore<2> sem(2); // 最多 2 个线程并发
int counter = 0;

void worker(int id) {
    sem.acquire();       // 获取信号量
    std::cout << "Thread " << id << " working\n";
    std::this_thread::sleep_for(std::chrono::seconds(1));
    counter++;
    std::cout << "Thread " << id << " done\n";
    sem.release();       // 释放信号量
}

int main() {
    std::vector<std::thread> threads;
    for (int i = 0; i < 5; ++i) {
        threads.emplace_back(worker, i);
    }
    for (auto& t : threads) t.join();
    std::cout << "Counter: " << counter << std::endl; // 5
    return 0;
}
```

输出：

```c++
Thread 0 working
Thread 1 working
Thread 0 done
Thread 2 working
Thread 1 done
Thread 3 working
Thread 2 done
Thread 4 working
Thread 3 done
Thread 4 done
Counter: 5
```

## 7、介绍下虚拟内存，为什么能提供比实际内存更大的空间？

#### 工作原理

##### 分页

虚拟内存通常被分割成固定大小的块，称为“页”（Page）。同样地，物理内存也被分成相同大小的块，称为“页框”（Page Frame）或“页帧”。虚拟页可以映射到任意一个物理页框上，这使得即使物理内存不是连续的，虚拟地址空间也可以表现为连续的。

##### 页表

为了管理虚拟页与物理页框之间的映射关系，操作系统维护了一张页表。每当CPU需要访问某个虚拟地址时，它会通过内存管理单元（Memory Management Unit, MMU）查询页表来找到对应的物理地址。如果该虚拟页已经在物理内存中，则直接访问；如果不在，则发生缺页异常，操作系统负责将相应的虚拟页从磁盘加载到物理内存中。

##### 页面调度算法

当物理内存不足以容纳所有需要驻留的虚拟页时，操作系统会使用某种页面调度算法选择一个或多个现有的物理页进行替换。常见的页面调度算法包括最近最少使用（LRU）、先进先出（FIFO）、随机替换等。

##### 按需调页

虚拟内存系统采用按需调页的方式工作，即只有当程序实际尝试访问某一页时，才会将其从磁盘加载到物理内存中。这种方式减少了不必要的I/O操作，提高了效率。

#### 为什么能提供比实际内存更大的空间？

##### 地址空间隔离

每个进程都有自己的独立虚拟地址空间，这意味着即使两个进程都请求了大量内存，它们也不会相互干扰，因为它们看到的是不同的虚拟地址空间。

##### 非连续分配

虚拟内存允许逻辑地址空间和物理地址空间是非连续的。这样，即便物理内存碎片化严重，虚拟内存仍可呈现给应用程序一个连续的地址空间。

##### 交换技术

当物理内存不足时，操作系统可以将暂时不用的数据页写入磁盘上的交换文件（如Windows中的pagefile.sys），从而腾出空间用于新的页面。这种机制允许系统运行超过物理内存总量的应用程序集合。

##### 局部性原理

大多数程序表现出时间和空间局部性，即它们倾向于反复访问一小部分数据集。基于这一特性，虚拟内存只需在物理内存中保持活跃数据即可满足大部分需求，而其余数据则保存于磁盘上等待后续加载。

## 8、进程和线程区别？

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

## 9、vector扩容机制？

#### 内部数据结构

`vector` 内部使用一块连续的内存区域来存储数据，这样能保证随机访问时间为 O(1)。

三个关键属性：

- Size（大小）： 当前已存储的元素个数。
- Capacity（容量）： 当前分配的内存可以容纳的最大元素个数。
- Data Pointer： 指向分配内存块的指针。

#### 扩容时机

当向 `vector` 中添加元素（如调用 `push_back` 或 `emplace_back`）时，如果当前的 `size` 等于 `capacity`，就需要扩容。

扩容意味着要分配一块更大的内存区域，将已有的元素复制或移动到新区域，再释放旧的内存。

#### 扩容策略（增长因子）

大多数标准库实现采用“指数增长”策略，通常是 “倍增”（doubling），即新容量为旧容量的 2 倍。例如，初始容量可能为 1 或 0，当第一个元素插入时分配 1，当容量不足时扩容到 2、4、8、16 等。

这种策略保证了扩容操作的次数与元素个数呈对数关系，从而使得连续插入操作的平均时间复杂度保持在摊销 O(1)。

有的实现可能采用其他增长因子（例如 1.5 倍），以平衡内存利用率和扩容次数，但倍增策略最常见。

#### 扩容过程

1. 分配新内存：当现有容量不足时，计算新的容量（如 `new_capacity = old_capacity * 2`），并分配一块新内存区域。

2. 复制/移动元素：将旧内存中的所有元素复制或移动到新内存中。如果元素类型支持移动语义（如具有移动构造函数），那么移动操作通常比复制更高效。
3. 释放旧内存：复制或移动完成后，释放旧的内存块，并更新内部指针、容量和大小等属性。

#### 内存碎片

扩容时，由于旧内存被释放，新内存可能不连续分配，尤其在长期频繁扩容和缩减后可能导致内存碎片问题，但由于 vector 使用连续内存，在大多数情况下仍能获得较高缓存命中率。

## 10、快速排序的时间复杂度？

#### 基本思想

1. 选择基准（Pivot）：从数组中选取一个元素作为基准。这个基准可以是数组的第一个元素、最后一个元素、随机选取的元素或者采用“三数取中法”等方法选取。

2. 分区（Partition）：将数组划分为两个子数组：

   - 左侧子数组中所有元素都不大于基准；

   - 右侧子数组中所有元素都不小于基准。
      分区过程实际上是通过不断交换元素来实现的。

3. 递归排序：对左右两个子数组递归执行上述步骤，直到子数组的长度为 0 或 1，此时数组已经有序。
4. 合并结果：快速排序是原地排序，不需要额外的合并过程，整个数组在递归结束后就已经排序完成。

#### 时间复杂度

##### 平均情况

当每次分区都比较均衡时，快速排序的平均时间复杂度为 O(n log n)。这是因为分区操作需要 O(n) 时间，而递归树的高度大约为 log n。

##### 最优情况

和平均情况一致，时间复杂度为 O(n log n)。

##### 最坏情况

当每次分区都极不均衡（例如，数组已经近乎有序，且每次选取的基准是最小或最大值），递归深度可能达到 n，时间复杂度退化为 O(n²)。
为了避免最坏情况，通常采用随机化基准或三数取中法来选取基准。

#### 示例代码（C++）

```c++
#include <iostream>
#include <vector>
#include <cstdlib>

void swap(int &a, int &b) {
    int temp = a;
    a = b;
    b = temp;
}

// 分区函数：选择基准并将数组分区，返回基准的最终位置
int partition(std::vector<int> &arr, int low, int high) {
    // 这里选择最后一个元素作为基准
    int pivot = arr[high];
    int i = low - 1; // i 用于标记小于或等于 pivot 的区域边界

    for (int j = low; j < high; ++j) {
        if (arr[j] <= pivot) {
            ++i;
            swap(arr[i], arr[j]);
        }
    }
    // 将基准元素放到正确的位置
    swap(arr[i + 1], arr[high]);
    return i + 1; // 返回基准位置
}

// 快速排序递归函数
void quickSort(std::vector<int> &arr, int low, int high) {
    if (low < high) {
        // 分区操作，获得基准元素位置
        int pi = partition(arr, low, high);

        // 对基准左侧子数组递归排序
        quickSort(arr, low, pi - 1);
        // 对基准右侧子数组递归排序
        quickSort(arr, pi + 1, high);
    }
}

int main() {
    std::vector<int> data = {10, 7, 8, 9, 1, 5};

    std::cout << "原数组：";
    for (int num : data) {
        std::cout << num << " ";
    }
    std::cout << std::endl;

    quickSort(data, 0, data.size() - 1);

    std::cout << "排序后：";
    for (int num : data) {
        std::cout << num << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

## 11、手撕 四进制转三进制（输入输出都是字符串）

#### 解题思路

先将四进制（基数为4）的数转换成十进制，再将十进制数转换成三进制（基数为3）。

##### 四进制转十进制

假设有一个四进制数，它的各位从左到右为
$$
𝑎
𝑛
,
𝑎
𝑛
−
1
,
…
,
𝑎
0
a 
n
​
 ,a 
n−1
​
 ,…,a 
0
​
$$
（每位 ai 的取值范围是 0~3）。其对应的十进制数可以表示为：
$$
N 
10
​
 =a 
n
​
 ×4 
n
 +a 
n−1
​
 ×4 
n−1
 +⋯+a 
0
​
 ×4 
0
$$
这种转换通过遍历各位数字，根据其位权求和即可完成。

##### 十进制转三进制

将得到的十进制数 N10转换成三进制，可以使用反复除以3的方法：

1. 对 N10N_{10}N10 进行除以 3 得到商和余数，余数即为三进制的最低位数字。
2. 将商继续除以 3，得到新的余数，依次重复，直到商为0。
3. 最后，将得到的余数序列逆序排列，即为三进制数的表示。

给个例子帮助大家理解：

假设输入的四进制数为字符串形式 "1230"（四进制中各位的数值均为 0-3）：

第一步：四进制转十进制
$$
1×4^3
 +2×4^2
 +3×4^1
 +0×4^0
 =1×64+2×16+3×4+0=64+32+12=108
$$
第二步：十进制转三进制 对 108 进行除 3：

108 ÷ 3 = 36，余数 0

36 ÷ 3 = 12，余数 0

12 ÷ 3 = 4，余数 0

4 ÷ 3 = 1，余数 1

1 ÷ 3 = 0，余数 1
 得到余数序列（从最低位到最高位）：0, 0, 0, 1, 1
 逆序排列后得到三进制表示为 "11000"。

#### 参考代码

```c++
#include <iostream>
#include <string>
#include <algorithm>
#include <cmath>

// 四进制字符串转换为十进制整数
int base4ToDecimal(const std::string &base4Str) {
    int decimalValue = 0;
    for (char ch : base4Str) {
        // 假设输入字符都是 '0' 到 '3'
        int digit = ch - '0';
        decimalValue = decimalValue * 4 + digit;
    }
    return decimalValue;
}

// 十进制整数转换为三进制字符串
std::string decimalToBase3(int decimalValue) {
    if (decimalValue == 0) return "0";
    
    std::string base3Str;
    while (decimalValue > 0) {
        int remainder = decimalValue % 3;
        base3Str.push_back('0' + remainder);
        decimalValue /= 3;
    }
    // 余数得到的数字顺序是低位到高位，逆序排列后得到正确的三进制表示
    std::reverse(base3Str.begin(), base3Str.end());
    return base3Str;
}

int main() {
    // 将四进制数 "1230" 转换为三进制
    std::string base4Number = "1230";
    int decimalValue = base4ToDecimal(base4Number);
    std::string base3Number = decimalToBase3(decimalValue);
    
    std::cout << "四进制数 " << base4Number << " 对应的十进制数为: " << decimalValue << std::endl;
    std::cout << "十进制数 " << decimalValue << " 对应的三进制数为: " << base3Number << std::endl;
    
    return 0;
}
```

