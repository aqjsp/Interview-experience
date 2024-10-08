# 拼多多服务端C++开发一面面经

> 来源：https://www.nowcoder.com/share/jump/1728393254369

### 1、手撕：给一个数组，输出索引m和n，只要m到n之间的数有序，则整个数组有序。

如果数组本身就有序，则输出-1，-1。例如数组1，3，5，9，8，7，4，6，13，应输出2，7

#### 思路

1. 初始化变量：
   - `start` 和 `end` 分别用于记录不满足有序部分的起始和结束索引，初始值设为 -1。
   - `maxSoFar` 和 `minSoFar` 分别用于记录从左到右遍历时的最大值和从右到左遍历时的最小值。
2. 从左到右遍历：
   - 如果当前元素小于 `maxSoFar`，说明当前元素不满足升序，更新 `end` 为当前索引。
   - 否则，更新 `maxSoFar` 为当前元素。
3. 从右到左遍历：
   - 如果当前元素大于 `minSoFar`，说明当前元素不满足升序，更新 `start` 为当前索引。
   - 否则，更新 `minSoFar` 为当前元素。
4. 调整边界：
   - 确保在 `start` 和 `end` 之间的所有元素都能满足有序的条件。如果 `start` 和 `end` 仍然是 -1，说明数组本身是有序的，返回 `[-1, -1]`。

#### 参考代码

```c++
#include <iostream>
#include <vector>
using namespace std;

vector<int> findSubarrayToSort(vector<int>& nums) {
    int n = nums.size();
    if (n <= 1) return {-1, -1};

    int start = -1, end = -1;
    int maxSoFar = INT_MIN, minSoFar = INT_MAX;

    for (int i = 0; i < n; ++i) {
        if (nums[i] < maxSoFar) {
            end = i;
        } else {
            maxSoFar = nums[i];
        }
    }

    for (int i = n - 1; i >= 0; --i) {
        if (nums[i] > minSoFar) {
            start = i;
        } else {
            minSoFar = nums[i];
        }
    }

    if (start == -1 && end == -1) {
        return {-1, -1};
    }

    for (int i = 0; i <= start; ++i) {
        if (nums[i] > nums[start]) {
            start = i;
            break;
        }
    }

    for (int i = n - 1; i >= end; --i) {
        if (nums[i] < nums[end]) {
            end = i;
            break;
        }
    }

    return {start, end};
}

int main() {
    vector<int> nums = {1, 3, 5, 9, 8, 7, 4, 6, 13};
    vector<int> result = findSubarrayToSort(nums);
    cout << "Start: " << result[0] << ", End: " << result[1] << endl;
    return 0;
}
```

### 2、c++为什么要进行结构体的内存对齐？

结构体的内存对齐是指编译器为了提高数据访问效率，确保结构体成员在内存中的地址满足特定的对齐要求而进行的一种内存布局调整。这种对齐不仅影响单个结构体的大小，还可能影响包含该结构体的其他数据结构的大小，进而影响程序的整体内存使用效率。

#### 1. 提高性能

内存对齐的主要原因之一是提高CPU访问内存的速度。现代CPU通常一次可以读取多个字节的数据，例如4个或8个字节。如果一个数据项没有正确对齐，那么CPU可能需要多次内存访问才能完成对该数据项的加载或存储操作，这会显著降低程序的运行速度。例如，在32位系统上，如果一个4字节的整数没有4字节对齐，那么访问这个整数时可能会跨越两个不同的内存块，导致性能下降。

此外，内存对齐还可以优化内存带宽的利用。内存带宽是有限的，对齐数据可以减少因读取未对齐数据而产生的额外开销，使内存带宽得到更有效的利用。

#### 2. 平台兼容性

不同的硬件平台对内存访问有着不同的限制。某些硬件平台可能无法访问任意地址上的任意数据，而只能从特定的内存地址开始存取特定类型的数据，否则可能会引发硬件异常或错误。内存对齐可以确保数据在存储时满足这些硬件要求，从而提高程序的兼容性和稳定性。

例如，某些架构的CPU要求特定类型的数据必须从特定的内存地址开始存取，如果数据没有正确对齐，可能会导致CPU无法正确读取数据，进而引发错误。因此，内存对齐有助于减少因不同平台间硬件差异而导致的兼容性问题，使得同一份代码能够在不同的硬件平台上正常运行。

#### 3. 内存对齐的具体规则

C++标准规定了每个数据类型的最小对齐要求，这些要求通常是该类型大小的倍数。例如，`int`类型（假设为4字节）应该至少4字节对齐，而`double`类型（假设为8字节）则应8字节对齐。具体规则如下：

- 第一个成员在与结构体偏移量为0的地址处：这意味着结构体的第一个成员总是从结构体的起始地址开始存储。
- 其他成员变量要对齐到某个数字（对齐数）的整数倍的地址处：对齐数是编译器默认的一个对齐数与该成员大小的较小值。例如，在Visual Studio中，默认的对齐数为8。
- 结构体总大小为最大对齐数的整数倍：结构体的总大小必须是其内部最大成员的对齐数的整数倍，不足的部分会通过填充字节来补齐。
- 如果嵌套了结构体的情况，嵌套的结构体对齐到自己的最大对齐数的整数倍处：结构体的整体大小就是所有最大对齐数（含嵌套结构体的对齐数）的整数倍。

#### 4. 实际示例

```c++
struct S1 {
    char c1;
    int i;
    char c2;
};

struct S2 {
    char c1;
    char c2;
    int i;
};
```

在32位系统上，`int`类型占用4个字节，`char`类型占用1个字节。根据内存对齐规则，`S1`和`S2`的内存布局如下：

- `S1`的内存布局：
  - `c1`：1字节，从偏移量0开始。
  - `i`：4字节，从偏移量4开始（因为`i`需要4字节对齐）。
  - `c2`：1字节，从偏移量8开始。
  - 填充3个字节，使结构体总大小为12字节（12是4的倍数）。
- `S2`的内存布局：
  - `c1`：1字节，从偏移量0开始。
  - `c2`：1字节，从偏移量1开始。
  - 填充2个字节，使`i`从偏移量4开始（因为`i`需要4字节对齐）。
  - `i`：4字节，从偏移量4开始。
  - 结构体总大小为8字节（8是4的倍数）。

#### 5. 控制内存对齐

虽然编译器默认会进行内存对齐，但在某些情况下，我们可能希望手动控制内存对齐，以达到更紧凑的内存布局。C++提供了一些机制来实现这一点，例如使用`#pragma pack`预处理指令。通过设置`#pragma pack`的参数，可以改变编译器的默认对齐数，从而影响结构体的内存布局。

例如，使用`#pragma pack(1)`可以取消结构体的内存对齐，使得结构体的成员变量紧密排列，不再有填充字节。这在某些嵌入式系统中可能非常有用，因为这些系统通常对内存使用非常敏感。

### 3、http与https的差别，刷短视频是用的哪个？

#### 1. 安全性

- HTTP：HTTP（超文本传输协议）是一种明文传输协议，不提供数据加密。这意味着在HTTP连接中传输的所有数据都是以明文形式发送的，容易被第三方截取或篡改。因此，HTTP不适合传输敏感信息，如信用卡号、密码等。
- HTTPS：HTTPS（超文本传输安全协议）是HTTP的加密版本，通过SSL/TLS协议进行加密，确保数据传输的安全性。HTTPS使用证书来验证服务器的身份，并为数据传输提供加密通道，保护数据不被窃取或篡改。因此，HTTPS更适合处理敏感信息，如在线银行、电子商务、隐私信息处理等。

#### 2. 证书

- HTTP：HTTP不需要证书。
- HTTPS：HTTPS需要网站安装SSL证书。证书通常由受信任的证书颁发机构（CA）签发，如Symantec、Comodo、GoDaddy和GlobalSign等。证书的费用因机构和功能的不同而有所差异。

#### 3. 端口

- HTTP：默认使用80端口。
- HTTPS：默认使用443端口。

#### 4. 速度

- HTTP：HTTP连接建立过程相对简单，只需要TCP三次握手，客户端和服务器交换3个包。
- HTTPS：HTTPS除了TCP三次握手外，还需要进行SSL握手，客户端和服务器需要交换额外的9个包，因此页面加载时间可能会稍长，功耗也会增加。

#### 5. 链接形式

- HTTP：URL前缀为`http://`。
- HTTPS：URL前缀为`https://`，并且浏览器通常会在地址栏显示一个小锁图标，表示连接是安全的。

#### 刷短视频时使用的协议

在刷短视频时，大多数主流平台（如抖音、快手、B站等）通常使用HTTPS协议。主要原因如下：

1. 数据安全：短视频平台涉及大量的用户数据和个人信息，使用HTTPS可以确保这些数据在传输过程中的安全，防止被第三方截取或篡改。
2. 用户体验：HTTPS协议通过加密传输，可以增强用户对平台的信任感，提升用户体验。
3. 搜索引擎优化：Google等搜索引擎对使用HTTPS的网站给予更高的排名，这有助于提高平台的可见性和流量。

#### 实际应用

- 抖音：抖音是一个非常流行的短视频平台，它使用HTTPS协议来保护用户数据的安全。抖音的官方文档和开发者指南中明确指出，所有API请求和数据传输都应使用HTTPS。
- 快手：快手也是一个大型的短视频平台，同样使用HTTPS协议来确保数据的安全传输。
- B站：B站（哔哩哔哩）是一个以二次元文化为主的视频分享平台，它也采用了HTTPS协议来保护用户数据和隐私。

### 4、epoll的ET与LT？

#### 1. 水平触发（Level Triggered, LT）

水平触发是 `epoll` 的**默认工作模式**，类似于传统的 `poll` 和 `select`。在这种模式下，只要文件描述符上有待处理的事件（例如可读或可写），`epoll_wait` 就会持续返回该事件，直到该事件被处理完毕。

##### 特点：

- 持续触发：只要文件描述符处于就绪状态（例如有数据可读或可写），`epoll_wait` 就会一直返回该事件。
- 易于实现：开发者可以按需处理事件，不需要一次性处理完所有数据。
- 适合场景：适用于对延迟不敏感的应用，如简单的 I/O 操作。

##### 示例代码：

```c++
#include <sys/epoll.h>
#include <sys/socket.h>
#include <unistd.h>
#include <iostream>

int main() {
    int epoll_fd = epoll_create(1);
    if (epoll_fd == -1) {
        perror("epoll_create");
        return -1;
    }

    int listen_fd = socket(AF_INET, SOCK_STREAM, 0);
    // 设置监听 socket 为非阻塞
    fcntl(listen_fd, F_SETFL, O_NONBLOCK);

    // 绑定和监听
    struct sockaddr_in serv_addr;
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = INADDR_ANY;
    serv_addr.sin_port = htons(8080);
    bind(listen_fd, (struct sockaddr *)&serv_addr, sizeof(serv_addr));
    listen(listen_fd, 5);

    struct epoll_event event;
    event.events = EPOLLIN;
    event.data.fd = listen_fd;
    epoll_ctl(epoll_fd, EPOLL_CTL_ADD, listen_fd, &event);

    while (true) {
        struct epoll_event events[10];
        int num_events = epoll_wait(epoll_fd, events, 10, -1);
        for (int i = 0; i < num_events; ++i) {
            if (events[i].data.fd == listen_fd) {
                // 处理新连接
                int conn_fd = accept(listen_fd, NULL, NULL);
                fcntl(conn_fd, F_SETFL, O_NONBLOCK);
                event.data.fd = conn_fd;
                epoll_ctl(epoll_fd, EPOLL_CTL_ADD, conn_fd, &event);
            } else {
                // 处理已连接的客户端
                char buffer[1024];
                int n = read(events[i].data.fd, buffer, sizeof(buffer));
                if (n > 0) {
                    write(events[i].data.fd, buffer, n);
                } else if (n == 0) {
                    // 客户端断开连接
                    epoll_ctl(epoll_fd, EPOLL_CTL_DEL, events[i].data.fd, NULL);
                    close(events[i].data.fd);
                } else {
                    // 错误处理
                    epoll_ctl(epoll_fd, EPOLL_CTL_DEL, events[i].data.fd, NULL);
                    close(events[i].data.fd);
                }
            }
        }
    }

    close(epoll_fd);
    close(listen_fd);
    return 0;
}
```

#### 2. 边缘触发（Edge Triggered, ET）

边缘触发模式下，`epoll_wait` 只在文件描述符的状态从非就绪变为就绪时触发一次事件。一旦事件被触发，即使文件描述符仍然处于就绪状态，`epoll_wait` 也不会再次返回该事件，直到文件描述符的状态再次发生变化。

##### 特点：

- 一次性触发：文件描述符的状态从非就绪变为就绪时触发一次事件，之后不会再次触发，直到状态再次变化。
- 高效：减少了事件的触发次数，适合高并发场景。
- 复杂性：需要开发者确保一次性处理完所有数据，否则可能会错过后续的数据。

##### 示例代码：

```c++
#include <sys/epoll.h>
#include <sys/socket.h>
#include <unistd.h>
#include <iostream>

int main() {
    int epoll_fd = epoll_create(1);
    if (epoll_fd == -1) {
        perror("epoll_create");
        return -1;
    }

    int listen_fd = socket(AF_INET, SOCK_STREAM, 0);
    // 设置监听 socket 为非阻塞
    fcntl(listen_fd, F_SETFL, O_NONBLOCK);

    // 绑定和监听
    struct sockaddr_in serv_addr;
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = INADDR_ANY;
    serv_addr.sin_port = htons(8080);
    bind(listen_fd, (struct sockaddr *)&serv_addr, sizeof(serv_addr));
    listen(listen_fd, 5);

    struct epoll_event event;
    event.events = EPOLLIN | EPOLLET; // 边缘触发模式
    event.data.fd = listen_fd;
    epoll_ctl(epoll_fd, EPOLL_CTL_ADD, listen_fd, &event);

    while (true) {
        struct epoll_event events[10];
        int num_events = epoll_wait(epoll_fd, events, 10, -1);
        for (int i = 0; i < num_events; ++i) {
            if (events[i].data.fd == listen_fd) {
                // 处理新连接
                int conn_fd = accept(listen_fd, NULL, NULL);
                fcntl(conn_fd, F_SETFL, O_NONBLOCK);
                event.data.fd = conn_fd;
                event.events = EPOLLIN | EPOLLET; // 边缘触发模式
                epoll_ctl(epoll_fd, EPOLL_CTL_ADD, conn_fd, &event);
            } else {
                // 处理已连接的客户端
                char buffer[1024];
                while (true) {
                    int n = read(events[i].data.fd, buffer, sizeof(buffer));
                    if (n > 0) {
                        write(events[i].data.fd, buffer, n);
                    } else if (n == 0) {
                        // 客户端断开连接
                        epoll_ctl(epoll_fd, EPOLL_CTL_DEL, events[i].data.fd, NULL);
                        close(events[i].data.fd);
                        break;
                    } else if (n == -1 && errno == EAGAIN) {
                        // 没有更多数据可读
                        break;
                    } else {
                        // 错误处理
                        epoll_ctl(epoll_fd, EPOLL_CTL_DEL, events[i].data.fd, NULL);
                        close(events[i].data.fd);
                        break;
                    }
                }
            }
        }
    }

    close(epoll_fd);
    close(listen_fd);
    return 0;
}
```

#### 3. 选择合适的模式

- 水平触发（LT）：适用于对延迟不敏感的应用，如简单的 I/O 操作。LT 模式下，事件会持续触发，直到处理完毕，因此实现起来相对简单。
- 边缘触发（ET）：适用于高并发场景，如高性能服务器。ET 模式下，事件只在状态变化时触发一次，减少了事件的触发次数，提高了效率，但需要确保一次性处理完所有数据，否则可能会错过后续的数据。

### 5、进程、线程、协程的区别？

#### 1. 进程（Process）

- 定义：进程是操作系统分配资源的基本单位，是正在运行的程序的实例。每个进程都有自己的地址空间、内存、数据栈和其他辅助数据。
- 特性：
  - 独立性：进程之间是独立的，内存不共享，通常通过进程间通信（IPC）进行数据交换。
  - 资源开销：创建和销毁进程的开销较大，进程上下文切换的成本高。
  - 调度：进程调度由操作系统内核负责，通常通过时间片轮转等算法。

##### 示例：

```c++
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int main() {
    pid_t pid = fork();
    if (pid < 0) {
        perror("fork failed");
        return 1;
    } else if (pid == 0) {
        // 子进程
        printf("Child process: PID = %d\n", getpid());
    } else {
        // 父进程
        printf("Parent process: PID = %d, Child PID = %d\n", getpid(), pid);
        wait(NULL); // 等待子进程结束
    }
    return 0;
}
```

#### 2. 线程（Thread）

- 定义：线程是进程的执行单元，是进程内部的一个调度单位。一个进程可以包含多个线程，这些线程共享进程的资源（如内存空间）。
- 特性：
  - 共享资源：线程间共享同一进程的资源，导致更轻量级的通信。
  - 低开销：创建和销毁线程的开销相对较小，线程上下文切换的成本低于进程。
  - 同步问题：由于共享内存，线程间的同步问题（如竞争条件）更为复杂，需要使用锁等机制来解决。

##### 示例：

```c
#include <stdio.h>
#include <pthread.h>

void* thread_function(void* arg) {
    printf("Thread running: ID = %ld\n", pthread_self());
    return NULL;
}

int main() {
    pthread_t thread_id;
    pthread_create(&thread_id, NULL, thread_function, NULL);
    pthread_join(thread_id, NULL);
    printf("Main thread: ID = %ld\n", pthread_self());
    return 0;
}
```

#### 3. 协程（Coroutine）

- 定义：协程是一种轻量级的用户级线程，允许在同一线程中进行多任务处理。它们通过手动控制的方式进行切换，不需要上下文切换的开销。
- 特性：
  - 调度灵活：协程的切换由程序员控制，通常在协程内部通过 `yield` 或 `await` 关键字实现。
  - 内存效率：协程通常使用更少的内存，因为它们在同一线程中共享上下文。
  - 非阻塞 I/O：协程在处理 I/O 操作时可以非阻塞，允许在等待的同时执行其他任务，适用于高并发场景。

##### 示例

```python
import asyncio

async def coroutine_function():
    print("Coroutine running")
    await asyncio.sleep(1)  # 模拟IO操作
    print("Coroutine done")

async def main():
    task = asyncio.create_task(coroutine_function())
    await task

asyncio.run(main())
```

#### 对比总结

| 特性     | 进程                       | 线程                 | 协程                   |
| -------- | -------------------------- | -------------------- | ---------------------- |
| 资源     | 独立，拥有独立的内存空间   | 共享同一进程的内存   | 共享同一线程的上下文   |
| 开销     | 创建和切换开销较大         | 创建和切换开销较小   | 创建和切换开销极小     |
| 调度     | 操作系统内核调度           | 操作系统内核调度     | 用户代码控制调度       |
| 通信     | 通过进程间通信（IPC）      | 共享内存、信号量、锁 | 通过函数调用、协作切换 |
| 适用场景 | 需要隔离性和资源管理的应用 | 多任务处理、并发任务 | 高并发、I/O密集型操作  |

### 6、联合索引的匹配原则？

联合索引（Composite Index）是数据库中一种重要的索引类型，它允许在一个索引中结合多个列，从而提高多条件查询的效率。联合索引的匹配原则主要是**最左前缀原则**，这一原则规定了联合索引在哪些情况下可以被有效利用。

#### 1. 联合索引的定义

联合索引是指在多个字段上创建的索引。例如，假设有一个表 `orders`，包含字段 `user_id`、`status` 和 `order_date`，可以创建一个联合索引：

```sql
CREATE INDEX idx_user_status_date ON orders (user_id, status, order_date);
```

这个联合索引实际上等同于创建了 `(user_id)`、`(user_id, status)` 和 `(user_id, status, order_date)` 三个索引，但只需要维护一个索引结构，从而节省了存储空间和维护成本。

#### 2. 最左前缀原则

最左前缀原则是指在联合索引中，查询条件必须从索引的最左边开始匹配，才能有效地利用索引。

具体来说：

- **全字段匹配**：如果查询条件包含联合索引中的所有字段，并且这些字段的顺序与索引定义的顺序一致，那么索引可以被完全利用。

例如：

```sql
SELECT * FROM orders WHERE user_id = 12345 AND status = 'shipped' AND order_date > '2024-10-09';
```

这个查询可以完全利用 `idx_user_status_date` 索引。

- **部分字段匹配**：如果查询条件包含联合索引中的部分字段，并且这些字段的顺序与索引定义的顺序一致，那么索引仍然可以被部分利用。

例如：

```sql
SELECT * FROM orders WHERE user_id = 12345 AND status = 'shipped';
```

这个查询可以利用 `idx_user_status_date` 索引中的 `(user_id, status)` 部分。

```sql
SELECT * FROM orders WHERE user_id = 12345;
```

这个查询可以利用 `idx_user_status_date` 索引中的 `(user_id)` 部分。

- **中间字段缺失**：如果查询条件跳过了联合索引中的某个字段，那么后续的字段将无法利用索引。

例如：

```sql
SELECT * FROM orders WHERE user_id = 12345 AND order_date > '2023-10-09';
```

这个查询无法利用 `idx_user_status_date` 索引中的 `status` 字段，因为 `status` 字段被跳过了。

- **范围查询**：如果查询条件中包含范围查询（如 `>`、`<`、`BETWEEN`、`LIKE` 等），那么范围查询之后的字段将无法利用索引。

例如：

```sql
SELECT * FROM orders WHERE user_id = 12345 AND status > 'shipped' AND order_date > '2024-10-09';
```

这个查询可以利用 `idx_user_status_date` 索引中的 `(user_id, status)` 部分，但 `order_date` 无法利用索引。

#### 3. 覆盖索引

联合索引的一个重要特性是**覆盖索引**（Covering Index），即索引包含了查询所需的所有字段，数据库无需回表查找数据。这可以显著减少IO操作，提高查询性能。

例如：

```sql
SELECT user_id, status FROM orders WHERE user_id = 12345 AND status = 'shipped';
```

在这个查询中，`user_id` 和 `status` 都包含在 `idx_user_status_date` 索引中，因此数据库可以直接从索引中获取数据，而无需回表查找。

#### 4. 索引的创建和优化

在创建联合索引时，需要考虑以下几个因素：

- 查询频率：将查询频率较高的字段放在联合索引的前面。
- 选择性：将选择性较高的字段放在联合索引的前面，以减少索引的大小和提高查询效率。
- 排序和分组：如果查询中包含 `ORDER BY` 或 `GROUP BY` 操作，将这些字段放在联合索引的前面，以减少排序操作的开销。

#### 5. 示例

假设有一个表 `users`，包含字段 `last_name`、`first_name` 和 `age`，可以创建一个联合索引：

```sql
CREATE INDEX idx_name_age ON users (last_name, first_name, age);
```

以下查询可以有效利用 `idx_name_age` 索引：

```sql
SELECT * FROM users WHERE last_name = 'Smith';
```

```sql
SELECT * FROM users WHERE last_name = 'Smith' AND first_name = 'John';
```

```sql
SELECT * FROM users WHERE last_name = 'Smith' AND first_name = 'John' AND age > 30;
```

以下查询无法有效利用 `idx_name_age` 索引：

```sql
SELECT * FROM users WHERE first_name = 'John';
```

```sql
SELECT * FROM users WHERE age > 30;
```

```sql
SELECT * FROM users WHERE first_name = 'John' AND age > 30;
```

### 7、怎么看一条sql语句的执行计划，具体输出哪些结果？

#### 1. MySQL

##### 方法

在MySQL中，可以通过在SQL语句前加上`EXPLAIN`关键字来查看执行计划。

例如：

```sql
EXPLAIN SELECT * FROM orders WHERE user_id = 12345 AND status = 'shipped';
```

##### 输出结果

`EXPLAIN`命令会返回一个包含多列的表格，每一列代表执行计划的不同方面。

主要的列包括：

- id：表示查询的序列号，同一查询的多个部分会有相同的id。
- select_type：表示查询的类型，如`SIMPLE`（简单查询）、`PRIMARY`（最外层查询）、`DERIVED`（派生表）、`SUBQUERY`（子查询）等。
- table：表示当前行的数据是关于哪张表的。
- partitions：表示当前查询匹配的分区（仅适用于分区表）。
- type：表示连接类型，从最好到最差的连接类型为`system` > `const` > `eq_reg` > `ref` > `range` > `index` > `ALL`。
- possible_keys：表示可能使用的索引。
- key：表示实际使用的索引。
- key_len：表示实际使用的索引长度。
- ref：表示与索引比较的列或常量。
- rows：表示MySQL估计需要检查的行数。
- filtered：表示通过所有条件过滤后剩余的行数比例。
- Extra：包含额外的信息，如`Using filesort`（需要排序）、`Using temporary`（需要临时表）、`Using index`（使用覆盖索引）等。

#### 2. Oracle

##### 方法

在Oracle中，可以通过`EXPLAIN PLAN FOR`命令生成执行计划，然后使用`DBMS_XPLAN.DISPLAY`函数来显示执行计划。

例如：

```sql
EXPLAIN PLAN FOR SELECT * FROM employees WHERE salary > 5000;
SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
```

##### 输出结果

`EXPLAIN PLAN FOR`命令会生成一个执行计划，并将其存储在`PLAN_TABLE`中。`DBMS_XPLAN.DISPLAY`函数会显示这个执行计划。

主要的列包括：

- Id：表示操作的唯一标识。
- Operation：表示当前的操作，如`TABLE ACCESS FULL`（全表扫描）、`INDEX RANGE SCAN`（索引范围扫描）等。
- Name：表示访问的表名或索引名。
- Rows：表示估计的返回行数。
- Bytes：表示估计的返回数据量。
- Cost (%CPU)：表示操作的代价。
- Time：表示操作的预计时间。
- Predicate Information：包含与操作相关的谓词信息，如访问条件和过滤条件。

#### 3. SQL Server

##### 方法

在SQL Server中，可以通过多种方式查看执行计划，包括使用SQL Server Management Studio（SSMS）的图形化界面或T-SQL命令。

例如，使用T-SQL命令：

```sql
SET SHOWPLAN_ALL ON;
GO
SELECT * FROM Orders WHERE User_ID = 12345 AND Status = 'Shipped';
GO
SET SHOWPLAN_ALL OFF;
GO
```

或者使用图形化界面：

1. 在SSMS中打开查询编辑器。
2. 选择要执行的SQL语句。
3. 右键点击并选择“显示估计的执行计划”。

##### 输出结果

`SHOWPLAN_ALL`命令会返回一个包含多列的表格，每一列代表执行计划的不同方面。主要的列包括：

- StmtText：表示每个步骤的具体描述。
- StmtId：表示语句的编号。
- NodeId：表示当前操作步骤的节点号。
- Parent：表示当前操作步骤的父节点。
- PhysicalOp：表示物理操作，如`Nested Loops`（嵌套循环连接）、`Hash Match`（哈希连接）等。
- LogicalOp：表示逻辑操作，如`Inner Join`（内连接）等。
- Argument：表示操作使用的参数。
- DefinedValues：表示定义的变量值。
- EstimateRows：表示估计返回的行数。
- EstimateIO：表示估计的IO成本。
- EstimateCPU：表示估计的CPU成本。
- AvgRowSize：表示平均行大小。
- TotalSubtreeCost：表示子树的总成本。
- OutputList：表示输出的列列表。
- OrderBy：表示排序的列列表。
- Predicate：表示谓词信息。
- ResidualPredicate：表示残余谓词信息。

### 8、innodb用b+树存索引的好处是什么？

1. **高效的查询性能**

B+树索引能够提供高效的查询性能。在B+树中，所有叶子节点都在同一层级，这使得在查询过程中只需进行一次平衡的二分查找，从而保证了较快的查询速度。此外，B+树的节点通常较大，可以减少访问节点的次数，提高查询效率。

2. **范围查询效率高**

B+树索引非常适合范围查询，例如`BETWEEN`和`>=`、`<=`等操作。B+树的所有叶子节点通过指针连接成有序链表，这使得范围查询和顺序访问更加高效。相比之下，B树和二叉树在范围查询时需要进行中序遍历，而Hash索引则不支持范围查询和排序。

3. **减少磁盘I/O次数**

B+树通过使用较大的数据块（通常是页面）来减少磁盘I/O操作。由于B+树的叶子节点存储了全部数据，而内部节点仅存储键值和指针，这样在数据库查询时，可以更有效地利用磁盘块的预读能力，减少磁盘I/O访问次数。相比之下，Hash索引在每次查询时都需要计算哈希值并访问相应的哈希桶，增加了磁盘I/O开销。

4. **磁盘空间利用率高**

B+树索引相对节约磁盘空间。B+树的内部节点仅存储键值和指针，而叶子节点存储了全部数据和指针，这样可以更好地利用磁盘空间。相比之下，Hash索引需要为每个键值存储哈希值和指针，因此可能需要更多的磁盘空间。

5. **支持并发和事务**

B+树索引适用于并发访问和事务处理。B+树的结构特性使得在并发访问时更容易实现锁机制和保持数据的一致性。另外，InnoDB存储引擎使用MVCC（多版本并发控制）来支持高并发事务，B+树结构更适合支持MVCC机制。

6. **高扇出，树的高度小**

B+树的一个显著特点是高扇出，即一个节点可以存放更多的数据。这使得整棵树更加矮胖，树的高度通常在2到4层之间。高度越小，查找的I/O次数越少，查询效率越高。

7. **非叶子节点只存放键值**

B+树的非叶子节点只存放键值，不存放实际数据，这使得每个节点可以存储更多的键值，进一步减少了树的高度。相比之下，B树的非叶子节点也需要存储数据，导致每个节点存储的键值较少，树的高度较高。

8. **叶子节点之间通过指针连接**

B+树的叶子节点之间通过指针连接成有序链表，这使得范围查询和顺序访问更加高效。这种结构使得B+树在处理大量数据时，能够快速定位到目标数据范围，并进行顺序扫描。

9. **适应磁盘读写特性**

B+树能够很好地配合磁盘的读写特性，减少单次查询的磁盘访问次数，降低I/O开销，提升性能。在机械硬盘时代，从磁盘随机读一个数据块需要10ms左右的寻址时间。B+树通过减少树的高度和增加每个节点的扇出，显著减少了磁盘I/O次数。

10. **插入和删除操作效率高**

B+树在插入和删除操作时，能够高效地进行节点分裂和合并。虽然这些操作可能导致节点的重新分配，但B+树的设计使得这些操作的复杂度较低，不会显著影响整体性能。

11. **适合大规模数据存储**

B+树的设计使得它非常适合存储大规模数据。由于每个节点可以存储更多的键值，B+树能够有效地管理大量数据，而不会因为树的高度过高而导致性能下降。

### 9、一致性哈希算法，什么场景，解决什么问题？

#### 基本原理

一致性哈希算法将整个哈希值空间组织成一个虚拟的圆环，即哈希环。在这个环上，每个节点（服务器）和每个数据项都会被哈希到环上的某个位置。当需要存储或查找数据时，通过对数据键值进行哈希计算，找到其在环上的位置，然后顺时针查找第一个节点，该节点即为负责存储或处理该数据的节点。

#### 主要优点

1. 减少数据迁移：当集群中的节点增加或减少时，一致性哈希算法只会导致少量数据的迁移，而不是所有数据的重新分布。这大大减少了因节点变化引起的网络通信压力，提高了系统的稳定性和性能。
2. 负载均衡：通过将数据和节点均匀分布在哈希环上，一致性哈希算法能够实现较为均衡的负载分配，避免了某些节点过载而其他节点空闲的情况。
3. 动态扩展：一致性哈希算法支持动态扩展，即在不影响现有数据分布的情况下，轻松添加或移除节点，这对于分布式系统来说是非常重要的特性。

#### 应用场景

1. 分布式缓存系统：如Memcached、Redis等分布式缓存系统中，一致性哈希算法用于将数据均匀分布到多个缓存节点上，确保即使在节点增加或减少时，也能保持较高的缓存命中率。
2. 负载均衡：在分布式系统中，一致性哈希算法可以用于将客户端请求均匀分配到多个服务器节点上，避免因节点变化导致的大量请求重定向。
3. 分布式数据库：在分布式数据库中，一致性哈希算法用于将数据分片存储到多个节点上，确保数据的均匀分布和高效查询。
4. 内容分发网络（CDN）：在CDN中，一致性哈希算法用于将用户请求分配到最近的边缘节点，提高内容的加载速度和用户体验。
5. P2P网络：在P2P网络中，一致性哈希算法用于将文件分片存储到多个对等节点上，确保文件的高效共享和下载。

#### 解决的问题

1. 节点动态变化时的数据迁移问题：传统哈希算法在节点数量变化时，需要重新计算所有数据的哈希值，导致大量数据迁移。一致性哈希算法通过环形结构，只需迁移受影响的数据，大大减少了数据迁移的成本。
2. 负载不均衡问题：在分布式系统中，如果节点分布不均匀，会导致某些节点过载而其他节点空闲。一致性哈希算法通过将节点和数据均匀分布在哈希环上，实现了较为均衡的负载分配。
3. 单点故障问题：在分布式系统中，单点故障可能导致整个系统的不可用。一致性哈希算法通过多节点冗余和数据迁移，提高了系统的可靠性和可用性。
4. 扩展性问题：传统哈希算法在节点数量变化时，需要重新配置整个系统，扩展性较差。一致性哈希算法支持动态扩展，无需重新配置，提高了系统的灵活性和可扩展性。

### 10、分布式事务的写一致性算法有了解过吗？Paxos协议，Raft协议知道吗？

#### Paxos协议

##### 基本原理

Paxos协议是由Leslie Lamport在1990年提出的一种基于消息传递的分布式一致性算法。Paxos协议的核心目标是在分布式系统中就某个值（决议）达成一致。Paxos协议通过多个轮次的协商过程，确保即使在网络不稳定或节点故障的情况下，系统中的多个节点也能达成一致。

##### 角色

Paxos协议中有三个主要角色：

1. Proposer（提议者）：负责提出提案（Proposal），提案中包含一个全局唯一的编号和一个值。
2. Acceptor（接受者）：负责接收和处理提案，参与投票过程。
3. Learner（学习者）：不参与提案和投票，只被动接收提案结果，用于扩展读性能或跨区域读操作。

##### 流程

Paxos协议的流程分为两个主要阶段：Prepare阶段和Accept阶段。

1. Prepare阶段：
   - Proposer选择一个提案编号 n*n*，并向半数以上的Acceptor广播Prepare(n)请求。
   - Acceptor收到Prepare(n)请求后，如果 n*n* 大于其已经响应过的提案的最大编号，则承诺不再接受编号小于 n*n* 的提案，并返回其已经接受的最大编号的提案内容（如果有）。
2. Accept阶段：
   - Proposer在Prepare阶段收到大多数Acceptor的响应后，选择一个值 v*v*（通常是响应中编号最大的提案的值），并向所有Acceptor广播Accept(n, v)请求。
   - Acceptor收到Accept(n, v)请求后，如果 n*n* 大于或等于其已经响应过的提案的最大编号，则接受该提案，并返回已接受的提案编号和值。
   - Proposer收到大多数Acceptor的接受响应后，认为该提案已被批准。

##### 优点

1. 高容错性：Paxos协议能够在网络不稳定或节点故障的情况下，保证系统的可用性和一致性。
2. 灵活性：Paxos协议可以应用于多种分布式系统场景，如分布式数据库、分布式存储等。

##### 缺点

1. 复杂性：Paxos协议的实现较为复杂，理解和实现难度较高。
2. 性能瓶颈：Paxos协议对网络延迟和故障处理要求较高，容易出现性能瓶颈。

#### Raft协议

##### 基本原理

Raft协议是由Diego Ongaro和John Ousterhout在2013年提出的一种分布式一致性算法。Raft协议的设计目标是提供一种易于理解和实现的解决方案，相比Paxos协议，Raft协议在角色和流程设计上更加清晰，降低了实现的难度。

##### 角色

Raft协议中有三个主要角色：

1. Leader（领导者）：负责处理客户端的请求和日志复制。
2. Follower（跟随者）：接受来自Leader的心跳信号，维持与Leader的通信，响应Leader的指令。
3. Candidate（候选人）：在选举过程中尝试成为Leader。

##### 流程

Raft协议的流程分为三个主要阶段：Leader选举、日志复制和安全性保证。

1. Leader选举：

   - 初始状态下，所有节点都是Follower。
   - 如果Follower在一定时间内没有收到Leader的心跳信号，会转变为Candidate，向其他节点发起投票请求。
   - 如果Candidate获得了超过半数节点的选票，将成为Leader，并开始处理客户端的请求和日志复制。

2. 日志复制：

   - Leader将客户端的操作序列化成日志条目，并将日志条目分发给其他节点。
   - 当大多数节点确认日志条目后，Leader将日志条目应用到状态机中，并返回结果给客户端。

3. 安全性保证：

   Raft协议通过一系列的安全性规则，确保日志的一致性和安全性，例如确保任何一个节点在任意时间点只有一个Leader，同时保证日志的线性顺序。

##### 优点

1. 易于理解和实现：Raft协议的设计更加直观，角色和流程清晰，降低了实现的难度。
2. 高可用性：Raft协议通过Leader选举和日志复制机制，保证了系统的高可用性和一致性。

##### 缺点

1. 性能问题：在极端网络分区或故障的情况下，Raft协议的性能可能会受到影响。
2. 单点故障：尽管Raft协议通过选举机制避免了单点故障，但在Leader节点故障时，系统可能会经历短暂的不可用期。

#### 应用场景

1. 分布式存储系统：如etcd、Consul等分布式键值存储系统，利用Raft协议实现了高可用的分布式存储服务，保证了数据的一致性和可靠性。
2. 分布式数据库：如Google的Chubby锁服务，利用Paxos协议实现了分布式锁服务，保证了系统中各个节点之间的数据一致性和并发控制。
3. 分布式系统中的领导者选举：Raft协议通过Leader选举机制，确保系统中只有一个领导者，避免了多个领导者同时处理请求导致的不一致问题。

### 11、实现直播打赏榜单，用redis哪种数据结构？

在实现直播打赏榜单时，使用Redis的**有序集合（Sorted Set）**数据结构是最佳选择。有序集合是一种特殊的集合，它不仅能够存储不重复的成员，而且每个成员还关联了一个分数（score），Redis会根据这个分数对集合中的成员进行排序。这种特性非常适合用来实现排行榜功能，因为可以方便地根据用户的贡献值（如打赏金额）对用户进行排序。

#### 为什么选择有序集合（Sorted Set）

1. 成员唯一性：有序集合中的成员是唯一的，这意味着每个用户在排行榜中只能出现一次，符合排行榜的需求。
2. 分数排序：每个成员都与一个分数相关联，Redis会根据分数自动对成员进行排序。这使得获取排行榜变得非常简单和高效。
3. 灵活的增删改查操作：有序集合提供了丰富的命令，如`ZADD`、`ZINCRBY`、`ZRANGE`、`ZREVRANGE`等，可以方便地添加、更新、查询和删除成员及其分数。
4. 高效性：由于Redis是内存数据库，操作速度非常快，即使是大规模数据量，也能保证高效的性能。

#### 实现步骤

##### 1. 创建有序集合

首先，需要为每个直播间创建一个有序集合，用于存储该直播间内的用户打赏情况。可以使用直播间的ID作为键名，用户ID作为成员，打赏金额作为分数。

```bash
ZADD live:room:<room_id> <score> <member>
```

例如，对于直播间ID为`123`，用户ID为`user1`，打赏金额为`100`的情况，可以这样添加：

```bash
ZADD live:room:123 100 user1
```

##### 2. 更新用户打赏金额

当用户再次打赏时，可以使用`ZINCRBY`命令来增加用户的打赏金额，从而更新排行榜。

```bash
ZINCRBY live:room:<room_id> <increment> <member>
```

例如，用户`user1`再次打赏`50`元：

```bash
ZINCRBY live:room:123 50 user1
```

##### 3. 获取排行榜

可以使用`ZRANGE`或`ZREVRANGE`命令来获取排行榜。`ZRANGE`按分数从小到大排序，`ZREVRANGE`按分数从大到小排序。

- 获取前10名用户（按打赏金额从高到低）：

```bash
ZREVRANGE live:room:<room_id> 0 9 WITHSCORES
```

- 获取指定范围的用户（例如第11到第20名）：

```
ZREVRANGE live:room:<room_id> 10 19 WITHSCORES
```

##### 4. 删除用户

如果需要删除某个用户，可以使用`ZREM`命令：

```bash
ZREM live:room:<room_id> <member>
```

例如，删除用户`user1`：

```bash
ZREM live:room:123 user1
```

#### 示例代码

以下是一个简单的C++示例，展示如何使用Redis的有序集合实现直播打赏榜单：

```c++
#include <hiredis/hiredis.h>
#include <iostream>
#include <string>

// 连接到Redis
redisContext* connectToRedis(const std::string& host, int port) {
    redisContext* c = redisConnect(host.c_str(), port);
    if (c == nullptr || c->err) {
        if (c) {
            std::cerr << "Error: " << c->errstr << std::endl;
            redisFree(c);
        } else {
            std::cerr << "Cannot allocate redis context" << std::endl;
        }
        return nullptr;
    }
    return c;
}

// 添加用户打赏
bool addDonation(redisContext* c, const std::string& room_id, const std::string& user_id, double amount) {
    redisReply* reply = (redisReply*)redisCommand(c, "ZINCRBY live:room:%s %f %s", room_id.c_str(), amount, user_id.c_str());
    if (reply == nullptr) {
        std::cerr << "Error adding donation: " << c->errstr << std::endl;
        return false;
    }
    freeReplyObject(reply);
    return true;
}

// 获取排行榜
std::vector<std::pair<std::string, double>> getLeaderboard(redisContext* c, const std::string& room_id, int start, int end) {
    redisReply* reply = (redisReply*)redisCommand(c, "ZREVRANGE live:room:%s %d %d WITHSCORES", room_id.c_str(), start, end);
    if (reply == nullptr) {
        std::cerr << "Error getting leaderboard: " << c->errstr << std::endl;
        return {};
    }

    std::vector<std::pair<std::string, double>> leaderboard;
    for (size_t i = 0; i < reply->elements; i += 2) {
        leaderboard.emplace_back(std::make_pair(reply->element[i]->str, std::stod(reply->element[i + 1]->str)));
    }

    freeReplyObject(reply);
    return leaderboard;
}

int main() {
    // 连接到Redis
    redisContext* c = connectToRedis("127.0.0.1", 6379);
    if (c == nullptr) {
        return 1;
    }

    // 添加用户打赏
    addDonation(c, "123", "user1", 100.0);
    addDonation(c, "123", "user2", 150.0);
    addDonation(c, "123", "user1", 50.0);

    // 获取前10名
    auto leaderboard = getLeaderboard(c, "123", 0, 9);
    for (const auto& [user, score] : leaderboard) {
        std::cout << "User: " << user << ", Score: " << score << std::endl;
    }

    // 断开连接
    redisFree(c);

    return 0;
}
```

### 12、挑一个项目，介绍你负责的工作，遇到什么问题，怎么解决的

### 13、能不能接受pdd的工作强度