# 华为C++二面

### 1、算法题25分钟：给定正整数数组和正整数k。数组中只含有0或1，可以最多k次将0翻转成1。求数组中最长连续1的长度。

#### 思路

1. 使用滑动窗口来维护一个区间，区间内最多允许翻转k个0。
2. 使用left和right两个指针表示区间的左右边界，初始时都指向数组的第一个元素。
3. 移动right指针，如果遇到0，则将k减一；如果遇到1，则继续向右移动。
4. 当k小于0时，移动left指针，直到k恢复为0为止，期间不断更新最长连续1的长度。
5. 返回最长连续1的长度。

#### 参考C++代码（ACM模式）

```
#include <iostream>
#include <vector>

using namespace std;

int longestOnes(vector<int>& nums, int k) {
    int left = 0, right = 0;    // 定义滑动窗口的左右边界
    int zeroCount = 0;          // 记录当前窗口内0的个数
    int maxLength = 0;          // 记录最长连续1的长度

    while (right < nums.size()) {
        if (nums[right] == 0) { // 如果右边界为0，增加zeroCount
            zeroCount++;
        }
        while (zeroCount > k) { // 当zeroCount超过k时，移动左边界，直到zeroCount <= k
            if (nums[left] == 0) {
                zeroCount--;
            }
            left++;
        }
        maxLength = max(maxLength, right - left + 1); // 更新最长连续1的长度
        right++; // 右边界向右移动
    }

    return maxLength;
}

int main() {
    vector<int> nums = {1, 0, 0, 1, 1, 0, 1, 1, 0, 1};
    int k = 2;
    int result = longestOnes(nums, k);
    cout << "最长连续1的长度为：" << result << endl;
    return 0;
}
```

### 2、项目20分钟

下面这几个问题，可能是项目里面的，这里我给一些建议答案吧

### 3、epoll在服务器之中如何使用？

1. 创建epoll句柄：使用`epoll_create`函数创建一个epoll句柄。
2. 添加事件：使用`epoll_ctl`函数将需要监听的文件描述符（通常是socket）添加到epoll句柄中，并设置关注的事件类型，如读事件（EPOLLIN）、写事件（EPOLLOUT）等。
3. 等待事件：使用`epoll_wait`函数等待事件的发生，该函数会阻塞直到有事件发生或者超时。
4. 处理事件：当`epoll_wait`返回时，表示有事件发生，此时可以遍历返回的事件列表，根据事件类型进行相应的处理，比如接受连接、读取数据、发送数据等。
5. 循环监听：处理完事件后，可以继续添加新的事件，然后再次调用`epoll_wait`等待新的事件发生。

给个demo

```
#include <iostream>
#include <sys/epoll.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <sys/socket.h>

#define MAX_EVENTS 10
#define PORT 8080

int main() {
    int server_fd, client_fd, epoll_fd, events_count;
    struct sockaddr_in server_addr, client_addr;
    struct epoll_event event, events[MAX_EVENTS];

    // 创建socket
    server_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (server_fd == -1) {
        perror("socket creation failed");
        return 1;
    }

    // 绑定地址和端口
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = INADDR_ANY;
    server_addr.sin_port = htons(PORT);
    if (bind(server_fd, (struct sockaddr*)&server_addr, sizeof(server_addr)) == -1) {
        perror("bind failed");
        return 1;
    }

    // 监听端口
    if (listen(server_fd, 10) == -1) {
        perror("listen failed");
        return 1;
    }

    // 创建epoll句柄
    epoll_fd = epoll_create1(0);
    if (epoll_fd == -1) {
        perror("epoll_create failed");
        return 1;
    }

    // 将server_fd添加到epoll句柄中
    event.events = EPOLLIN;
    event.data.fd = server_fd;
    if (epoll_ctl(epoll_fd, EPOLL_CTL_ADD, server_fd, &event) == -1) {
        perror("epoll_ctl failed");
        return 1;
    }

    while (true) {
        // 等待事件发生
        events_count = epoll_wait(epoll_fd, events, MAX_EVENTS, -1);
        if (events_count == -1) {
            perror("epoll_wait failed");
            return 1;
        }

        // 处理事件
        for (int i = 0; i < events_count; ++i) {
            if (events[i].data.fd == server_fd) {
                // 有新连接请求
                client_fd = accept(server_fd, (struct sockaddr*)&client_addr, (socklen_t*)&addrlen);
                // 将新的client_fd添加到epoll句柄中
                event.events = EPOLLIN;
                event.data.fd = client_fd;
                epoll_ctl(epoll_fd, EPOLL_CTL_ADD, client_fd, &event);
            } else {
                // 读取数据
                // ...
            }
        }
    }

    return 0;
}
```

### 4、服务器SYN-RECV为什么会过多，怎么处理？

1. 网络延迟或拥塞：如果服务器端网络延迟或者网络拥塞严重，会导致服务器端无法及时响应客户端的SYN请求，从而导致SYN-RECV状态过多。
2. 服务器资源不足：服务器端的TCP连接资源（如端口号、内存等）不足时，会导致服务器无法建立新的连接，从而产生大量的SYN-RECV状态。
3. 恶意攻击：某些恶意攻击行为，如SYN洪水攻击等，会大量发送伪造的SYN请求，导致服务器SYN-RECV状态过多。

处理方法：

1. 优化服务器网络：确保服务器端网络畅通，尽量减少网络延迟和拥塞。
2. 优化服务器资源：增加服务器端的TCP连接资源，如调整操作系统的最大连接数、优化内存使用等。
3. 防火墙和入侵检测系统：配置防火墙和入侵检测系统，及时发现并阻止恶意攻击。
4. SYN Cookie技术：使用SYN Cookie技术可以在SYN攻击时动态生成SYN+ACK响应，不占用服务器端资源，有效防止SYN洪水攻击。
5. 负载均衡：使用负载均衡技术分担服务器端压力，提高系统的可靠性和可用性。

### 5、死链和保活机制怎么设计？

死链是指两个或多个进程在执行过程中因争夺资源而造成的一种僵局，使得所有进程都无法继续执行。在网络通信中，死链也可以指两个或多个节点之间的通信因为相互等待对方的消息而无法继续的情况。保活机制（Keep-Alive）是指在网络通信中，为了确保连接的有效性，定期发送一些数据包以维持连接的状态。

设计时需要注意的点：

1. 识别死链：在程序设计中，需要识别可能导致死链的情况，如多个进程互相等待对方释放资源的情况，以及网络通信中可能发生的相互等待消息的情况。
2. 避免死链：可以通过合理的资源分配和请求顺序来避免死链的发生。在网络通信中，可以使用非阻塞IO和超时机制来避免死链。
3. 处理死链：一旦死链发生，需要及时处理，可以通过强制释放资源或者重新请求资源来解除死链。
4. 保活机制：在网络通信中，可以通过保活机制来确保连接的有效性。保活机制通常包括定时发送心跳包或者其他数据包来检测连接是否有效，如果连接超过一定时间没有活动则认为连接已经失效。

### 6、长连接和短连接区别，长连接断开如何恢复状态？

1. 长连接：长连接指客户端与服务器建立的连接在一段时间内保持打开状态，可以用于多次请求和响应。在长连接中，客户端和服务器之间的通信不会频繁地建立和关闭连接，可以减少连接建立和断开的开销，提高通信效率。
2. 短连接：短连接指客户端与服务器建立的连接只用于一次请求和响应，通信完成后立即关闭连接。在短连接中，客户端和服务器之间的通信需要频繁地建立和关闭连接，可能会增加通信的延迟和资源消耗。

长连接断开后如何恢复状态取决于具体的应用场景和需求，一般可以通过以下几种方式来实现：

1. 重连机制：客户端在检测到与服务器的连接断开后，可以尝试重新建立连接。重连机制可以根据实际情况设定重连的间隔和次数，以避免频繁地尝试重连导致服务器负载过大。
2. 保活机制：在长连接中，可以通过保活机制来检测连接的有效性。保活机制通常包括定时发送心跳包或者其他数据包来检测连接是否有效，如果连接超过一定时间没有活动则认为连接已经失效，此时可以触发重连机制。
3. 状态同步：在长连接断开后，客户端和服务器可以通过其他手段来同步状态，如客户端可以重新发送之前发送过的请求，服务器可以保存客户端的状态并在重连后恢复状态。
4. 重试机制：在长连接断开后，客户端可以在重连后重新发送之前发送过的请求，以确保请求的可靠性。

### 7、原子性的软件（CAS）和硬件实现方式？

1. 软件实现：在软件层面，原子操作通常通过一些特殊的编程技巧来实现。例如，在多线程编程中，可以使用互斥锁（mutex）或者信号量（semaphore）来保证临界区的原子性操作。另外，一些编程语言提供了原子操作的库函数或者语法，如 Java 中的 `Atomic` 类。
2. 硬件实现：在硬件层面，原子操作通常依赖于处理器提供的特殊指令。例如，x86 架构中提供了一些原子操作指令，如 `CMPXCHG`、`XADD` 等，可以在执行过程中保证操作的原子性。这些指令可以在执行时锁定内存位置，防止其他线程或者处理器同时访问该内存位置。

### 8、互斥锁和自旋锁应用场景和区别？

1. 实现方式：
   - 互斥锁：当一个线程获取到互斥锁后，如果另一个线程尝试获取该锁，它会被阻塞，直到第一个线程释放了锁。互斥锁的实现通常使用操作系统提供的原子操作，涉及用户态和内核态的切换，因此会有一定的开销。
   - 自旋锁：当一个线程尝试获取自旋锁时，如果锁已被其他线程占用，它会不断地循环（自旋）检查锁是否被释放，直到获取到锁为止。自旋锁的实现通常是通过在多核处理器上使用原子操作来实现的，避免了用户态和内核态的切换，因此在短时间内的竞争中效率更高。
2. 适用场景：
   - 互斥锁：适用于对共享资源的访问时间较长的情况，因为在长时间等待时，会将线程置于休眠状态，节省了 CPU 资源。适用于临界区代码执行时间较长的情况，例如文件操作、网络访问等。
   - 自旋锁：适用于对共享资源的访问时间很短的情况，因为在等待锁时会一直占用 CPU 资源，如果等待时间过长，会导致性能下降。适用于临界区代码执行时间很短的情况，例如简单的数学计算、内存操作等。

### 9、信号量如何使用？

信号量是一种用于线程或进程间同步与互斥的机制，常用于控制对共享资源的访问。在操作系统中，信号量可以分为两种类型：二进制信号量和计数信号量。

1. 二进制信号量：
   - 二进制信号量只能取两个值：0和1。通常用于实现互斥锁。
   - 当信号量为1时，表示资源可用，允许访问；当信号量为0时，表示资源已被占用，禁止访问。
   - 使用二进制信号量实现互斥锁时，通常使用P操作和V操作来实现对临界区的互斥访问。
2. 计数信号量：
   - 计数信号量可以取多个非负整数值，表示资源的可用数量。
   - 当计数信号量大于0时，表示还有可用资源；当计数信号量等于0时，表示资源已经全部被占用。
   - 使用计数信号量可以实现限制资源访问的数量，比如线程池中的线程数量限制等。

在C++中，可以使用`std::mutex`和`std::condition_variable`等标准库提供的同步原语来实现信号量的功能。

给个例子：

```
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

class Semaphore {
public:
    Semaphore(int count) : count_(count) {}

    void acquire() {
        std::unique_lock<std::mutex> lock(mutex_);
        while (count_ == 0) {  // 如果计数为0，等待直到有资源可用
            condition_.wait(lock);
        }
        count_--;  // 获取资源，计数减一
    }

    void release() {
        std::unique_lock<std::mutex> lock(mutex_);
        count_++;  // 释放资源，计数加一
        condition_.notify_one();  // 通知等待的线程有资源可用
    }

private:
    int count_;
    std::mutex mutex_;
    std::condition_variable condition_;
};

Semaphore semaphore(5);  // 创建一个计数为5的信号量

void worker(int id) {
    semaphore.acquire();  // 获取资源
    std::cout << "Worker " << id << " is working" << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(1));  // 模拟工作时间
    semaphore.release();  // 释放资源
}

int main() {
    std::vector<std::thread> threads;
    for (int i = 0; i < 10; ++i) {
        threads.emplace_back(worker, i);
    }

    for (auto& thread : threads) {
        thread.join();
    }

    return 0;
}
```

### 10、Linux内存分布？

1. 内核空间（Kernel Space）：
   - 内核空间是操作系统内核运行的区域，用于存放操作系统内核代码和数据结构。
   - 内核空间通常位于系统的高地址部分，通常是从虚拟地址空间的最高地址开始的一段区域。
   - 用户程序不能直接访问内核空间的内存，需要通过系统调用等方式向内核发起请求。
2. 用户空间（User Space）：
   - 用户空间是用户程序运行的区域，用于存放用户程序的代码和数据。
   - 用户空间通常位于系统的低地址部分，紧邻着内核空间。
   - 用户程序可以直接访问用户空间的内存。
3. 共享库空间：
   - 共享库空间是存放共享库（动态链接库）的区域，用于存放被多个程序共享的代码和数据。
   - 共享库通常位于用户空间，但是可以被多个进程共享，从而节省内存空间。
4. 栈空间（Stack）：
   - 栈空间用于存放函数调用时的局部变量、函数参数、返回地址等数据。
   - 栈空间是由操作系统自动管理的，每个线程都有自己的栈空间。
   - 栈空间通常位于用户空间，是向下增长的，即栈顶地址低于栈底地址。
5. 堆空间（Heap）：
   - 堆空间用于存放动态分配的内存，由程序员手动管理（通过malloc、free等函数）。
   - 堆空间通常位于用户空间，是向上增长的，即堆顶地址高于堆底地址。
6. 内存映射区域（Memory Mapped Region）：
   - 内存映射区域用于将文件映射到内存，从而实现文件的读写操作。
   - 内存映射区域通常位于用户空间，是通过mmap函数实现的。

### 11、Linux虚拟内存如何实现？

1. 虚拟内存概念：
   - 虚拟内存是一种抽象概念，它为每个进程提供了一个独立的、连续的虚拟地址空间。
   - 虚拟内存空间通常比实际物理内存空间大得多，可以让多个进程共享物理内存，并且可以提高内存的利用率和安全性。
2. 页表：
   - Linux使用页表来实现虚拟内存。页表是一个数据结构，用于将虚拟地址映射到物理地址。
   - 每个进程都有自己的页表，用于将其虚拟地址空间映射到物理内存。
   - 页表由多级结构组成，通常包括页目录、页表和页表项。
3. 页面置换：
   - 页面置换是指当物理内存不足时，操作系统需要将部分页面从物理内存换出到磁盘上，以便为新的页面腾出空间。
   - Linux使用LRU（最近最少使用）算法来进行页面置换。当物理内存不足时，操作系统会根据页面的访问时间选择最久未被访问的页面进行置换。
4. 页面调度：
   - Linux使用页面调度算法来管理物理内存中的页面。页面调度算法决定哪些页面应该留在内存中，哪些页面应该置换出去。
   - Linux中常用的页面调度算法包括FIFO（先进先出）、LRU（最近最少使用）、Clock（时钟算法）等。
5. 写时复制：
   - 写时复制是一种优化技术，用于延迟页面的复制操作。
   - 当一个进程尝试修改一个共享页面时，操作系统会先将该页面复制一份，然后再进行修改，从而避免影响其他进程对同一页面的访问。

### 12、Linux内存分配Lazy行为？

Linux内存分配中的Lazy行为指的是延迟分配内存的策略。在Linux系统中，当进程申请内存时，并不会立即分配物理内存给该进程，而是采用一种延迟分配的方式，即只有在进程真正访问这块内存时才会进行物理内存的分配。

优点：

1. 节省内存：如果进程在申请内存后并没有实际使用，那么就不需要分配物理内存，从而节省了内存资源。
2. 提高性能：延迟分配减少了对内存管理的频繁操作，减少了内存管理的开销，提高了系统的性能。
3. 避免浪费：如果进程在申请内存后没有使用，那么分配的内存就会被浪费。延迟分配可以避免这种浪费。

缺点：

1. 内存碎片：延迟分配可能会导致内存碎片问题，因为被释放的内存并没有被及时合并或重用。
2. 性能抖动：如果延迟分配过度，当进程实际需要大量内存时，可能会导致性能抖动，即系统性能急剧下降。

### 13、三级存储体系？

三级存储体系是指由高速缓存、内存和磁盘组成的计算机存储系统。这种存储体系根据存取速度和成本的不同，将数据分别存储在三个不同层次的存储介质中，以实现对数据的高效管理和访问。

1. 高速缓存（Cache）：
   - 特点：高速缓存位于处理器和主内存之间，容量较小但速度极快，能够迅速响应处理器的访问请求。
   - 作用：高速缓存通过存储最近被处理器访问过的数据和指令，减少了处理器访问主内存的次数，提高了系统的整体性能。
2. 内存（Memory）：
   - 特点：内存是计算机中的主要存储介质，容量比高速缓存大，速度介于高速缓存和磁盘之间。
   - 作用：内存存储了当前正在运行的程序和数据，是处理器能够直接访问的存储介质，对系统性能起着至关重要的作用。
3. 磁盘（Disk）：
   - 特点：磁盘是一种容量较大但速度较慢的存储介质，用于长期存储数据和程序。
   - 作用：磁盘通常用于存储操作系统、应用程序和用户数据等，数据在磁盘上以文件形式存储，可以长期保存并且在需要时读取到内存中进行处理。

三级存储体系中，数据的访问速度和成本呈现出明显的差异，高速缓存具有最快的访问速度但成本最高，内存速度次之而成本也适中，磁盘速度最慢但容量最大且成本最低。计算机系统通过合理地利用这三个层次的存储介质，可以在满足性能需求的同时有效地管理存储资源。

### 14、内存屏障？

内存屏障（Memory Barrier）是一种用于控制特定处理器在处理器内部或与外部设备之间的内存操作顺序的指令或者指令序列。它可以确保在多核处理器或多线程环境下，内存操作的顺序符合程序员的预期，避免出现数据竞争和内存一致性问题。

作用：

1. 顺序一致性：内存屏障可以保证在内存屏障之前的内存操作先于内存屏障之后的内存操作执行，从而确保程序的执行顺序符合程序员的预期。
2. 禁止指令重排序：内存屏障可以禁止处理器对内存访问指令进行重排序优化，保证指令的执行顺序不会被改变。
3. 内存可见性：内存屏障可以确保处理器对共享变量的修改在其他处理器或线程中可见，避免缓存不一致性导致的数据读取问题。

根据其作用的范围和影响的内容可以分为几种类型：

1. 全屏障：全屏障会影响到所有内存操作，包括加载和存储操作，在全屏障之前和之后的所有内存操作都不能重排序。
2. 加载屏障：加载屏障用于保证在加载操作之后的内存访问不会被重排序到加载操作之前。
3. 存储屏障：存储屏障用于保证在存储操作之前的内存访问不会被重排序到存储操作之后。
4. Acquire Barrier 和 Release Barrier：用于实现同步操作的屏障。Acquire Barrier 用于确保在之后的读操作不会被重排序到该屏障之前的写操作之前，而 Release Barrier 用于确保在之前的写操作不会被重排序到该屏障之后的读操作之后。

### 15、cmake如何产生二进制文件？

CMake 是一个开源的跨平台的构建工具，用于管理代码的构建过程。它使用一种称为 CMakeLists.txt 的简单脚本语言来描述项目的构建过程，然后根据这些描述生成相应的构建系统文件（如 Makefile、Visual Studio 项目文件等），最终用于构建项目的二进制文件。

基本步骤：

1. 创建 CMakeLists.txt 文件：在项目的根目录下创建一个名为 CMakeLists.txt 的文件，用于描述项目的构建过程和依赖关系。

2. 编写 CMakeLists.txt 文件：在 CMakeLists.txt 文件中编写描述项目的配置信息，包括项目名称、源文件列表、包含目录、链接库等。

   ```
   cmake_minimum_required(VERSION 3.10)
   project(MyProject)
   
   # 添加可执行文件
   add_executable(MyExecutable main.cpp)
   
   # 添加依赖库
   target_link_libraries(MyExecutable MyLibrary)
   ```

3. 创建 build 目录：在项目根目录下创建一个用于存放生成的中间文件和二进制文件的 build 目录。

4. 进入 build 目录：在命令行中进入 build 目录。

5. 运行 cmake：运行以下命令来配置项目，并生成相应的构建系统文件。

   ```
   cmake ..
   ```

6. 运行构建工具：根据生成的构建系统文件（如 Makefile）来构建项目。

   ```
   make
   ```

7. 生成二进制文件：构建成功后，在 build 目录中会生成项目的可执行文件或库文件，即项目的二进制文件。

   ```
   ./MyExecutable
   ```

### 16、git提交流程和创建新的分支？

很多公司可能都用的git，所以git对于一个程序员来讲还是非常重要的。其中步骤4、5、6、7、8也是我经常用的。

1. 克隆仓库或进入已有仓库：首先需要将远程仓库克隆到本地，或者进入已有的本地仓库。

   ```
   git clone https://github.com/username/repository.git
   cd repository
   ```

2. 创建新分支（可选）：如果需要在新的分支上进行开发，可以使用以下命令创建并切换到新分支。

   ```
   git checkout -b new_branch_name
   ```

3. 进行修改：对文件进行修改、添加、删除等操作。

4. 查看修改状态：可以使用以下命令查看文件的修改状态。

   ```
   git status
   ```

5. 添加修改到暂存区：将修改的文件添加到暂存区。

   ```
   git add .
   ```

6. 提交到本地仓库：将暂存区的修改提交到本地仓库。

   ```
   git commit -m "Commit message"
   ```

7. 推送到远程仓库：如果是在新分支上开发，首次推送需要指定远程仓库和分支。

   ```
   git push -u origin new_branch_name
   ```

   如果是在已有分支上开发，可以直接使用以下命令推送。

   ```
   git push
   ```

8. 合并到主分支（可选）：如果新功能开发完成，可以将新分支合并到主分支。

   ```
   git checkout main
   git merge new_branch_name
   git push
   ```

9. 删除本地分支（可选）：如果新分支已经合并到主分支，可以删除本地分支。

   ```
   git branch -d new_branch_name
   ```

10. 删除远程分支（可选）：如果新分支已经合并到主分支，可以删除远程分支。

    ```
    git push origin --delete new_branch_name
    ```

### 17、gdb如何查看寄存器的值和变量内存？

1. 查看寄存器的值：使用 `info registers` 命令可以查看当前所有寄存器的值。

   ```
   (gdb) info registers
   ```

   若要查看特定寄存器（如 eax）的值，可以使用 `info register` 命令：

   ```
   (gdb) info register eax
   ```

2. 查看变量内存：可以使用 `print` 命令查看变量的值，使用 `x` 命令查看内存内容。

   - 使用 `print` 命令查看变量的值：

     ```
     (gdb) print variable_name
     ```

   - 使用 `x` 命令查看内存内容，语法为 `x/[格式] 内存地址`，例如：

     ```
     (gdb) x/4xw &variable_name
     ```

     其中 `/4xw` 表示以十六进制格式 (`x`) 查看四个字 (`4`)，以字（4字节）为单位 (`w`)。

### 18、堆的实现？

堆是一种数据结构，通常用来实现优先队列。在堆中，每个节点的值都大于等于（或小于等于）其子节点的值，且根节点是最大（或最小）值。堆通常使用数组来表示，具有以下特性：

1. 完全二叉树结构：堆是一棵完全二叉树，即除了最后一层节点可能不满外，其他层节点都是满的，且最后一层的节点都靠左排列。
2. 最大堆和最小堆：最大堆的每个节点的值都大于等于其子节点的值；最小堆的每个节点的值都小于等于其子节点的值。

堆的常见操作包括插入元素、删除堆顶元素、建立堆和堆排序。以下是堆的实现细节：

1. 堆的表示：通常使用数组来表示堆，数组中的第一个元素（索引为 0）不存储元素，堆中的元素从索引 1 开始存储。
2. 插入元素：将新元素插入到堆的末尾，然后向上调整堆，直到满足堆的性质。
3. 删除堆顶元素：将堆顶元素（根节点）删除，并将堆的最后一个元素移到根节点位置，然后向下调整堆，直到满足堆的性质。
4. 建立堆：从最后一个非叶子节点开始，依次向上调整每个节点，使得以该节点为根的子树成为一个堆，直到根节点。
5. 堆排序：利用最大堆（或最小堆）的特性，首先建立一个堆，然后每次将堆顶元素与堆的最后一个元素交换，并缩小堆的范围，再次调整堆，直到排序完成。

这里给大家一个最大堆的示例代码。

```
#include <iostream>
#include <vector>

using namespace std;

// 定义最大堆类
class MaxHeap {
private:
    vector<int> heap;

    // 获取父节点索引
    int parent(int i) {
        return (i - 1) / 2;
    }

    // 获取左子节点索引
    int leftChild(int i) {
        return 2 * i + 1;
    }

    // 获取右子节点索引
    int rightChild(int i) {
        return 2 * i + 2;
    }

    // 向上调整堆
    void heapifyUp(int i) {
        while (i > 0 && heap[i] > heap[parent(i)]) {
            swap(heap[i], heap[parent(i)]);
            i = parent(i);
        }
    }

    // 向下调整堆
    void heapifyDown(int i) {
        int maxIndex = i;
        int l = leftChild(i);
        int r = rightChild(i);
        if (l < heap.size() && heap[l] > heap[maxIndex]) {
            maxIndex = l;
        }
        if (r < heap.size() && heap[r] > heap[maxIndex]) {
            maxIndex = r;
        }
        if (i != maxIndex) {
            swap(heap[i], heap[maxIndex]);
            heapifyDown(maxIndex);
        }
    }

public:
    // 插入元素
    void insert(int value) {
        heap.push_back(value);
        heapifyUp(heap.size() - 1);
    }

    // 删除堆顶元素
    void deleteMax() {
        if (heap.empty()) {
            return;
        }
        heap[0] = heap.back();
        heap.pop_back();
        heapifyDown(0);
    }

    // 堆排序
    void heapSort() {
        vector<int> sortedHeap;
        while (!heap.empty()) {
            sortedHeap.insert(sortedHeap.begin(), heap[0]);
            deleteMax();
        }
        heap = sortedHeap;
    }

    // 打印堆元素
    void printHeap() {
        for (int i = 0; i < heap.size(); i++) {
            cout << heap[i] << " ";
        }
        cout << endl;
    }
};

int main() {
    MaxHeap maxHeap;
    maxHeap.insert(5);
    maxHeap.insert(3);
    maxHeap.insert(8);
    maxHeap.insert(1);
    maxHeap.insert(6);

    cout << "原始堆：";
    maxHeap.printHeap();

    maxHeap.deleteMax();
    cout << "删除堆顶元素后的堆：";
    maxHeap.printHeap();

    maxHeap.heapSort();
    cout << "堆排序后的结果：";
    maxHeap.printHeap();

    return 0;
}
```

