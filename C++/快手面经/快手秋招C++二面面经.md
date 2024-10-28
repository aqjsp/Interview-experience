# 快手秋招C++二面面经

> 来源：https://www.nowcoder.com/share/jump/1730118544638

### 1、C++有哪些新特性？

老生常谈的问题了。。这里就罗列几个C11的特性吧，平时也比较常用。

1. 自动类型推导（Auto）： 允许编译器推导变量的类型，使代码更加简洁。

```c
auto x = 5; // x的类型将被推导为int
```

2. 范围-based for 循环： 简化了对容器元素的遍历。

```c
std::vector<int> numbers = {1, 2, 3, 4, 5};
for (const auto& num : numbers) {
    // 使用num
}
```

3. 智能指针： 引入了`std::shared_ptr`和`std::unique_ptr`等智能指针，用于管理动态分配的内存，帮助防止内存泄漏。

```c
std::shared_ptr<int> sharedPtr = std::make_shared<int>(42);
```

4. Lambda 表达式： 允许在函数内部定义匿名函数，提高代码可读性和灵活性。

```c
auto add = [](int a, int b) { return a + b; };
```

5. nullptr： 引入了空指针常量`nullptr`，用于替代传统的空指针`NULL`。

```c
int* ptr = nullptr;
```

6. 强制类型转换（Type Casting）： 引入了`static_cast`、`dynamic_cast`、`const_cast`、`reinterpret_cast`等更安全和灵活的类型转换操作符。

```c
double x = 3.14;
int y = static_cast<int>(x);
```

7. 右值引用和移动语义： 支持通过右值引用实现移动语义，提高了对临时对象的处理效率。

```c
std::vector<int> getVector() {
    // 返回一个临时vector
    return std::vector<int>{1, 2, 3};
}

std::vector<int> numbers = getVector(); // 使用移动语义
```

8. 新的容器和算法： 引入了新的容器，如`std::unordered_map`、`std::unordered_set`，以及一些新的算法。

```c
std::unordered_map<int, std::string> myMap = {{1, "one"}, {2, "two"}};
```

9. 线程支持（std::thread）： 提供了原生的多线程支持，使得并发编程更加方便。

```c
#include <thread>

void myFunction() {
    // 线程执行的代码
}

int main() {
    std::thread t(myFunction);
    t.join(); // 等待线程结束
    return 0;
}
```

### 2、什么情况下用lambda表达式？

这里给大家稍微举几个简单的例子吧。

#### 1. STL 算法

在使用标准模板库（STL）中的算法时，Lambda 表达式可以作为谓词（Predicate）传递给算法函数，简化代码编写。

例如，使用 `std::sort` 进行自定义排序：

```c
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> numbers = {5, 2, 8, 1, 3};
    // 使用 Lambda 表达式进行排序（升序）
    std::sort(numbers.begin(), numbers.end(), [](int a, int b) { return a < b; });

    // 打印排序后的结果
    for (int num : numbers) {
        std::cout << num << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

例子中，Lambda 表达式被用作 `std::sort` 算法的谓词，实现了对向量中元素的升序排序。

#### 2. 事件处理

在事件处理中，Lambda 表达式可以用于定义事件的处理函数，使代码更简洁和可读。

例如，模拟一个事件触发：

```c
#include <iostream>
#include <functional>

int main() {
    // Lambda 表达式作为事件处理函数
    std::function<void()> eventHandler = [] {
        std::cout << "Event occurred!" << std::endl;
    };

    // 模拟事件触发
    eventHandler();

    return 0;
}
```

例子中，Lambda 表达式被用作事件处理函数，当事件触发时，Lambda 表达式会被调用。

#### 3. 简化匿名函数

在需要定义简单函数对象的地方，Lambda 表达式可以显著简化代码。

例如，计算两个数的和：

```c
#include <iostream>

int main() {
    // 定义 Lambda 表达式并计算两个整数的和
    int a = 10;
    int b = 5;
    int result = [&a, b]() -> int { return a + b; }();

    // 打印结果
    std::cout << "Sum: " << result << std::endl;

    return 0;
}
```

例子中，Lambda 表达式用于计算两个整数的和，并立即调用。

#### 4. 多线程编程

在多线程编程中，Lambda 表达式可以用于定义线程的执行函数，简化线程的创建和管理。

例如，启动一个线程：

```c
#include <iostream>
#include <thread>

int main() {
    int x = 10;
    // 使用 Lambda 表达式定义线程的执行函数
    std::thread t([x] {
        std::cout << "Thread running with x = " << x << std::endl;
    });

    // 等待线程结束
    t.join();

    return 0;
}
```

例子中，Lambda 表达式用于定义线程的执行函数，并捕获外部变量 `x`。

#### 5. 函数式编程

在函数式编程中，Lambda 表达式可以用于定义高阶函数，即接受其他函数作为参数或返回函数的函数。

例如，使用 `std::function` 定义高阶函数：

```c
#include <iostream>
#include <functional>

int main() {
    // 定义一个高阶函数
    std::function<int(int, int)> add = [](int a, int b) { return a + b; };

    // 使用高阶函数
    int result = add(3, 4);
    std::cout << "Result: " << result << std::endl;

    return 0;
}
```

例子中，Lambda 表达式被用作 `std::function` 的参数，定义了一个高阶函数 `add`。

### 3、char s[10* 1024*1024];有问题吗？

这个声明的作用是分配一个大小为 10 MB 的字符数组 `s`

#### 1. 栈空间限制

在大多数操作系统中，每个线程的栈空间是有限的。通常默认的栈大小为 1MB 到 8MB，具体取决于操作系统和编译器的配置。如果你在一个函数中定义了一个非常大的局部数组，例如 `char s[10 * 1024 * 1024];`，这会占用 10MB 的栈空间，可能会导致栈溢出。

示例：

```c
#include <stdio.h>

void test() {
    char s[10 * 1024 * 1024]; // 10MB 的数组
    printf("Array defined successfully\n");
}

int main() {
    test();
    return 0;
}
```

可能的错误：

```bash
Stack overflow
Segmentation fault (core dumped)
```

#### 2. 动态内存分配

为了避免栈溢出的问题，可以使用动态内存分配函数 `malloc` 或 `calloc` 来分配大数组。这样，内存会在堆上分配，不受栈大小的限制。

示例：

```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    char *s = malloc(10 * 1024 * 1024); // 动态分配 10MB 的内存
    if (s == NULL) {
        fprintf(stderr, "Memory allocation failed\n");
        return 1;
    }

    // 使用数组 s
    // ...

    free(s); // 释放内存
    return 0;
}
```

### 4、栈一般多大？可以修改吗？如何修改？

#### 1. 栈的默认大小

栈的默认大小因操作系统和编译器的不同而有所差异。以下是一些常见的默认栈大小：

- Windows: 默认栈大小通常为 1MB。
- Linux: 默认栈大小通常为 8MB，但可以通过 `ulimit -s` 命令查看和修改。

![64位](https://cdn.jsdelivr.net/gh/aqjsp/Pictures/image-20241028211056189.png)

#### 2. 栈的大小限制

栈的大小是有限的，通常由操作系统和编译器预先设定。栈主要用于存储函数的局部变量、参数和返回地址等。如果栈空间不足，可能会导致栈溢出，进而引发程序崩溃。

#### 3. 修改栈大小的方法

在 Linux 环境下，可以通过以下两种方法修改栈大小：

1. 临时修改栈大小 使用 `ulimit -s` 命令可以临时修改当前 shell 会话的栈大小。

   例如，将栈大小设置为 100MB：

   ```bash
   ulimit -s 102400
   ```

2. 永久修改栈大小

   - 方法一：修改 `/etc/rc.local` 文件在 `/etc/rc.local`文件中添加以下行，以在系统启动时设置栈大小：

     ```bash
     ulimit -s 102400
     ```

   - 方法二：修改 `/etc/security/limits.conf` 文件在`/etc/security/limits.conf`文件中添加以下行，以设置所有用户的栈大小：

     ```bash
     * soft stack 102400
     * hard stack 102400
     ```

### 5、map怎么判断key是否存在？用下标判断可以吗？

#### 1. 使用 `find()` 方法

`find()` 方法返回一个迭代器，如果 `key` 存在，则返回指向该 `key` 的迭代器；如果 `key` 不存在，则返回 `map.end()`。

```c
#include <iostream>
#include <map>

int main() {
    std::map<std::string, int> map;
    map["key1"] = 1;
    map["key2"] = 2;

    if (map.find("key1") != map.end()) {
        std::cout << "key1 存在" << std::endl;
    } else {
        std::cout << "key1 不存在" << std::endl;
    }

    return 0;
}
```

#### 2. 使用 `count()` 方法

`count()` 方法返回 `key` 在 `map` 中出现的次数。由于 `map` 中的 `key` 是唯一的，如果 `key` 存在，`count()` 返回 1；否则返回 0。

```c
#include <iostream>
#include <map>

int main() {
    std::map<std::string, int> map;
    map["key1"] = 1;
    map["key2"] = 2;

    if (map.count("key1") > 0) {
        std::cout << "key1 存在" << std::endl;
    } else {
        std::cout << "key1 不存在" << std::endl;
    }

    return 0;
}
```

#### 3. 使用 `contains()` 方法（C++20）

`contains()` 方法是一个更直观的方法，用于检查 `map` 中是否存在指定的 `key`。

```c
#include <iostream>
#include <map>

int main() {
    std::map<std::string, int> map;
    map["key1"] = 1;
    map["key2"] = 2;

    if (map.contains("key1")) {
        std::cout << "key1 存在" << std::endl;
    } else {
        std::cout << "key1 不存在" << std::endl;
    }

    return 0;
}
```

**不推荐**使用下标操作符来判断键是否存在，因为 `myMap[key]` 会在 `key` 不存在时创建该键并将值初始化为默认值（例如 `0` 或空字符串）。而且使用下标操作符会**修改 `map`**，并在不必要的情况下占用额外内存。

### 6、为什么map使用下标会初始化？

C++ 中，`std::map` 使用下标（`operator[]`）访问键的设计是为了方便插入和访问元素，但这也意味着如果键不存在时，它会自动插入一个具有默认值的键值对。

一些原因：

1. 方便插入和访问

   - `operator[]`提供了简洁的接口，可以在同一操作中插入和访问元素。例如：

     ```c
     myMap[key] = value;
     ```

   - 如果 `key` 不存在，`std::map` 会在内部创建该键并将值初始化为类型的默认值，再赋予新值。这样可以一行代码完成插入和赋值操作。

2. 返回引用的需求

   - `operator[]` 返回一个对值类型的**引用**（`mapped_type&`），这意味着即使键不存在，`operator[]` 也必须返回一个可操作的值。

   - 为此，`std::map` 会自动插入该键并返回一个新插入元素的引用，以保证引用的有效性。

3. 保证接口一致性
   - C++ 设计中更偏向于**“使用者不用关心元素是否存在”**，可以用下标直接操作。虽然 `std::map` 提供了 `find` 等方法来避免插入，但 `operator[]` 的这种行为在某些情况下能简化代码，尤其是在希望直接访问或更改元素值时。

### 7、tcp可以两次握手吗？会造成什么问题？

1. 防止历史连接的初始化

   如果客户端发送的 SYN 报文在网络中延迟或丢失，而客户端重新启动后再次尝试建立连接，发送新的 SYN 报文。此时，如果服务端收到的是之前的旧 SYN 报文，而客户端并没有收到服务端的响应，服务端将错误地认为连接已经建立，但实际上客户端并不知道这个连接的存在。这会导致服务端浪费资源，维持一个无效的连接。三次握手通过客户端的第三次确认，确保了服务端的 SYN 报文确实被客户端接收，从而避免了这种历史连接的问题。

2. 确认双方的发送和接收能力

   - 确认发送能力：第一次握手时，客户端发送 SYN 报文，服务端收到后确认客户端的发送能力。
   - 确认接收能力：第二次握手时，服务端发送 SYN-ACK 报文，客户端收到后确认服务端的发送能力和自身的接收能力。
   - 确认服务端的接收能力：第三次握手时，客户端发送 ACK 报文，服务端收到后确认客户端的发送能力和自身的接收能力。如果只有两次握手，服务端无法确认客户端的接收能力，可能导致数据传输失败。

3. 资源管理

   如果只有两次握手，服务端在收到客户端的 SYN 报文后会立即分配资源来建立连接。如果客户端没有收到服务端的 SYN-ACK 报文，它可能会重新发送 SYN 报文。服务端每收到一个 SYN 报文都会分配资源，但客户端可能根本没有准备好接收这些资源，导致资源浪费。三次握手通过客户端的第三次确认，确保了服务端分配的资源是有意义的。

### 8、tcp发送数据是有序的吗？

TCP **发送数据包的顺序是有序的**，即发送方会按数据流的顺序依次将数据包推送到网络中。然而，在网络传输的过程中，由于各种原因（如不同路径的延迟、路由器的处理时间等），数据包可能会乱序到达接收方。

1. 发送方数据流顺序

   - TCP 在发送数据时，会根据数据流的顺序生成数据包，每个包会带上一个唯一的**序列号（Sequence Number）**，表示数据包在整个传输流中的位置。

   - 发送方会依次将数据包放入发送缓冲区，按照顺序发往网络。这意味着**发送方在发送时是按顺序发送的**。

2. 网络传输中的乱序

   虽然发送时是有序的，但数据包在网络上传输过程中会受到不同因素的影响，例如不同路径的路由器转发速度不一致，导致到达顺序可能不同。因此，接收方接收到的包可能会乱序。

3. 接收方的重排与重传机制

   - 接收方根据序列号识别数据包顺序，将乱序的包缓存起来，并等待缺失的数据包到达后，按照序列号顺序组装。

   - 如果某些数据包丢失或延迟，接收方会请求发送方重新发送，以确保数据最终按序排列并完整交付到应用层。

### 9、无序怎么保证最后的数据有序？

在TCP协议中，尽管数据包在网络传输过程中可能会因为不同的路由路径、网络拥塞等因素导致乱序到达，但TCP通过一系列机制确保最终数据能够按正确的顺序重组并传递给应用层。

#### 1. 序列号（Sequence Number）

每个TCP数据包的头部都包含一个32位的序列号，该序列号表示该数据包中第一个字节的编号。发送方为每个数据包分配一个唯一的序列号，接收方根据这些序列号对数据包进行排序。

#### 2. 确认应答（ACK）

接收方在收到数据包后，会发送一个确认应答（ACK）给发送方，告知发送方已经成功接收到哪些数据包。ACK中包含了一个确认号（Acknowledgment Number），表示接收方期望接收的下一个数据包的序列号。

#### 3. 乱序检测与处理

当接收方收到乱序的数据包时，它会根据序列号将这些数据包存储在接收缓冲区中，等待缺失的数据包到达。例如，假设接收方收到了序列号为100-199和200-299的数据包，但没有收到序列号为300-399的数据包，它会将100-199和200-299的数据包存储在缓冲区中，并发送一个ACK，表示期望接收序列号为300的数据包。

#### 4. 重传机制

如果发送方在设定的超时时间内没有收到某个数据包的ACK，它会认为该数据包丢失，并重新发送该数据包。这种机制确保了数据包的可靠传输。

#### 5. 数据重组

当所有数据包按顺序到达接收方后，接收方会根据序列号将这些数据包从缓冲区中取出，并按正确的顺序重组。重组后的数据流将按顺序传递给应用层。

#### 6. 选择确认（Selective Acknowledgment, SACK）

SACK是一种可选的TCP扩展，它允许接收方在ACK中通知发送方哪些数据包已经收到，哪些数据包丢失。这有助于发送方更准确地确定需要重传的数据包，从而提高传输效率。例如，如果接收方收到了100-199、200-299和400-499的数据包，但没有收到300-399的数据包，它可以发送一个包含SACK选项的ACK，告知发送方100-199和200-299的数据包已经收到，但300-399的数据包丢失。

#### 7. 缓冲区管理

TCP接收方维护一个接收缓冲区，用于存储乱序到达的数据包。缓冲区的大小可以根据接收方的处理能力动态调整，以防止缓冲区溢出。当缓冲区中的数据包按顺序重组后，这些数据将被传递给应用层。

#### 8. 重复数据包处理

TCP接收方能够识别并丢弃重复的数据包。如果某个数据包在网络中多次传输并到达接收方，接收方会根据序列号检测到重复的数据包，并将其丢弃，确保数据的唯一性和顺序。

### 10、重传是保证可靠性，有序应该用什么保证？

TCP 中，**序列号**机制是确保数据包最终按顺序到达的核心手段。可参考上一题的回答。

### 11、滑动窗口除了保证有序，还有其他作用吗？

#### 1. 流量控制

滑动窗口机制是TCP实现流量控制的关键手段。通过滑动窗口，接收方可以动态调整窗口大小，告知发送方自己当前的接收能力。如果接收方的缓冲区接近满载，它可以减小窗口大小，从而减缓发送方的数据发送速率，防止接收方的缓冲区溢出。相反，如果接收方的缓冲区有足够的空间，它可以增大窗口大小，允许发送方更快地发送数据。

#### 2. 提高传输效率

传统的停止等待协议（Stop-and-Wait）要求发送方每发送一个数据包后必须等待接收方的确认应答才能发送下一个数据包。这种方式效率低下，因为发送方在等待确认应答期间处于空闲状态。滑动窗口机制允许多个数据包在未收到确认应答的情况下连续发送，从而提高了数据传输的效率。

#### 3. 拥塞控制

滑动窗口机制还可以用于拥塞控制。TCP通过动态调整滑动窗口的大小来适应网络的拥塞状况。当网络出现拥塞时，发送方会减小窗口大小，减少数据发送速率，以减轻网络拥塞。当网络状况改善时，发送方可以逐步增大窗口大小，恢复正常的传输速率。

#### 4. 可靠传输

滑动窗口机制与确认应答（ACK）机制相结合，确保了数据传输的可靠性。发送方在发送数据包后，会等待接收方的确认应答。如果在设定的超时时间内没有收到确认应答，发送方会认为数据包丢失，并重新发送该数据包。通过这种方式，滑动窗口机制确保了数据包的可靠传输。

#### 5. 动态调整窗口大小

滑动窗口的大小可以根据网络状况和接收方的处理能力动态调整。发送方和接收方在建立连接时协商初始窗口大小，之后在数据传输过程中根据实际情况动态调整窗口大小。这种灵活性使得TCP能够在不同网络环境中高效工作。

#### 6. 乱序处理

滑动窗口机制还能够处理乱序到达的数据包。接收方可以将乱序到达的数据包暂时存储在缓冲区中，等待缺失的数据包到达后再进行重组。通过这种方式，滑动窗口机制确保了数据的有序传输。

#### 7. 选择确认（Selective Acknowledgment, SACK）

SACK是一种可选的TCP扩展，它允许接收方在ACK中通知发送方哪些数据包已经收到，哪些数据包丢失。这有助于发送方更准确地确定需要重传的数据包，从而提高传输效率。滑动窗口机制与SACK结合使用，进一步增强了TCP的可靠性和效率。

### 12、详细讲下拥塞控制的过程？

![拥塞控制流程图](https://cdn.jsdelivr.net/gh/aqjsp/Pictures/image-20241028225246425.png)

#### 1. 慢启动（Slow Start）

慢启动的目标是逐步探测网络的承载能力，避免过早造成拥塞。

- 初始状态：`cwnd`通常从 1 个最大报文段（MSS）开始。
- 指数增长：每次收到一个 ACK，`cwnd`增加一个 MSS，即每轮 RTT `cwnd`大小翻倍。
- 阈值限制：当`cwnd`达到慢启动阈值 `ssthresh` 时，慢启动结束，转入拥塞避免阶段。

慢启动通过指数增长方式快速增大窗口，利用带宽，但也会迅速接近网络的最大承载量。

#### 2. 拥塞避免（Congestion Avoidance）

在拥塞避免阶段，`cwnd`增长速率变慢，改为线性增长。

- 线性增长：每经过一轮 RTT，`cwnd`增加一个 MSS，而不是像慢启动时那样指数增长。
- 持续增长：拥塞避免阶段的增长过程是缓慢的，这样减少了因窗口过大而导致的网络拥塞风险。

拥塞避免阶段使得 TCP 在接近网络极限时保持稳定传输，减少发生拥塞的概率。

#### 3. 快速重传（Fast Retransmit）

快速重传通过检测**重复 ACK**来快速发现丢包，并立即重传丢失的包，而无需等待超时。

- 触发条件：当发送方连续收到三个重复 ACK 时，认为出现了丢包。
- 立即重传：发送方会立即重传该丢失的数据包，不再等待重传超时，从而降低等待时间。

快速重传提高了丢包时的响应速度，避免了长时间等待超时引起的传输中断。

#### 4. 快速恢复（Fast Recovery）

快速恢复与快速重传一起工作，避免了每次丢包都重新进入慢启动。

- 减小窗口：收到三个重复 ACK 后，将 `ssthresh`设为 `cwnd`的一半，并将`cwnd`缩减为 `ssthresh`的大小。
- 继续发送：不进入慢启动，而是维持较大的窗口，通过线性增长的方式逐渐增大`cwnd`。

快速恢复允许丢包后保持更高的传输速率，不必返回慢启动的最小窗口值。

### 13、recv的返回值是什么？-1和0的时候呢？

#### `recv` 函数的定义

```c
#include <sys/types.h>
#include <sys/socket.h>

ssize_t recv(int sockfd, void *buf, size_t len, int flags);
```

- `sockfd`：套接字描述符。
- `buf`：接收数据的缓冲区。
- `len`：缓冲区的长度。
- `flags`：控制接收操作的标志，通常设置为 `0`。

#### 返回值

1. **返回值 > 0**
   - 表示成功接收到了数据，返回值是实际接收到的字节数。
   - 例如，如果 `recv` 返回 `10`，则表示成功接收了 `10` 个字节的数据。
2. **返回值 = 0**
   - 表示对端已经关闭了连接。
   - 这通常发生在对端调用了 `close` 或 `shutdown` 函数，表示连接的结束。
   - 应用程序应该关闭套接字并清理资源。
3. **返回值 < 0（通常是 -1）**
   - 表示发生错误。
   - 具体错误可以通过检查 `errno` 变量来确定。

#### 常见错误码

- EAGAIN 或 EWOULDBLOCK
  - 套接字已标记为非阻塞，但接收操作会被阻塞（即没有数据可读）。
  - 在这种情况下，应用程序可以继续尝试读取，或者等待 `select` 或 `epoll` 的后续通知。
- EINTR
  - 操作被信号中断。
  - 应用程序可以继续尝试读取，或者等待 `select` 或 `epoll` 的后续通知。
- EBADF：`sockfd` 不是有效的文件描述符。
- ECONNRESET：远程主机强制关闭连接。
- EFAULT：缓冲区地址无效。
- EINVAL：参数无效。
- ENOMEM：内存不足。
- ENOTCONN：与面向连接的套接字尚未连接。
- ENOTSOCK：`sockfd` 不是套接字。

#### 示例代码

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <arpa/inet.h>

#define BUFFER_SIZE 1024

int main() {
    int sockfd;
    struct sockaddr_in server_addr;
    char buffer[BUFFER_SIZE];
    ssize_t bytes_received;

    // 创建套接字
    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd < 0) {
        perror("Socket creation failed");
        exit(EXIT_FAILURE);
    }

    // 设置服务器地址
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(8080);
    inet_pton(AF_INET, "127.0.0.1", &server_addr.sin_addr);

    // 连接到服务器
    if (connect(sockfd, (struct sockaddr*)&server_addr, sizeof(server_addr)) < 0) {
        perror("Connection failed");
        close(sockfd);
        exit(EXIT_FAILURE);
    }

    // 接收数据
    bytes_received = recv(sockfd, buffer, BUFFER_SIZE, 0);
    if (bytes_received > 0) {
        buffer[bytes_received] = '\0';
        printf("Received: %s\n", buffer);
    } else if (bytes_received == 0) {
        printf("Connection closed by peer\n");
    } else {
        perror("Recv failed");
        if (errno == EAGAIN || errno == EWOULDBLOCK || errno == EINTR) {
            printf("Continue receiving...\n");
        }
    }

    // 关闭套接字
    close(sockfd);
    return 0;
}
```

### 14、用过什么linux命令？



### 15、查询io使用率的命令？

#### 1. `iostat`

`iostat` 命令可以显示 CPU 使用情况和 I/O 统计信息。需要安装 `sysstat` 包来使用它。

```bash
iostat -x 1
```

- `-x`：显示扩展统计信息，包括每个设备的 I/O 性能。
- `1`：每 1 秒更新一次统计信息。

#### 2. `vmstat`

`vmstat` 命令可以显示系统的虚拟内存、进程、CPU 活动和 I/O 信息。

```bash
vmstat 1
```

- `1`：每 1 秒更新一次统计信息。
- 关注 **bi** 和 **bo** 列：`bi` 是块设备的输入，`bo` 是块设备的输出。

#### 3. `dstat`

`dstat` 是一个通用的资源统计工具，可以同时显示 CPU、内存、磁盘、网络等的使用情况。

```bash
dstat -cdngy
```

- `-c`：CPU 使用情况。
- `-d`：磁盘使用情况。
- `-n`：网络统计。
- `-g`：页缓存和交换信息。
- `-y`：系统上下文切换。

#### 4. `iotop`

`iotop` 是一个实时监控 I/O 使用情况的工具，可以显示哪些进程正在执行 I/O 操作。

```bash
sudo iotop
```

- 需要 root 权限才能看到所有进程的 I/O 活动。

#### 5. `pidstat`

`pidstat` 是 `sysstat` 包中的一个工具，可以显示指定进程的 I/O 统计信息。

```bash
pidstat -d 1
```

- `-d`：显示 I/O 统计信息。
- `1`：每 1 秒更新一次。

#### 6. `sar`

`sar` 命令用于收集和报告系统活动，包括 I/O 统计信息。

```bash
sar -d 1
```

- `-d`：显示磁盘活动。
- `1`：每 1 秒更新一次。

### 16、怎么创建子进程？（fork）

#### 使用 `fork()` 创建子进程

##### 1. `fork()` 函数原型

`fork()` 的函数原型如下：

```c
pid_t fork(void);
```

##### 2. 返回值

- 在父进程中：`fork()` 返回新创建子进程的进程 ID（PID）。
- 在子进程中：`fork()` 返回 0。
- 出错时：返回 -1，并设置 `errno` 以指示错误原因。

##### 3. 示例代码

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>

int main() {
    pid_t pid = fork(); // 创建子进程

    if (pid < 0) {
        // 错误处理
        perror("Fork failed");
        exit(EXIT_FAILURE);
    } else if (pid == 0) {
        // 子进程代码
        printf("This is the child process with PID: %d\n", getpid());
        // 可以在这里执行子进程特定的代码
        exit(EXIT_SUCCESS);
    } else {
        // 父进程代码
        printf("This is the parent process with PID: %d and created child PID: %d\n", getpid(), pid);
        // 等待子进程结束
        wait(NULL); // 可选，等待子进程完成
        printf("Child process has finished.\n");
    }

    return 0;
}
```

除了 `fork`，还有其他一些方法可以创建子进程：

- **`vfork`**：`vfork` 类似于 `fork`，但它在子进程调用 `exec` 或退出之前不会复制父进程的地址空间。子进程和父进程共享地址空间，子进程必须尽快调用 `exec` 或退出。
- **`clone`**：`clone` 是一个更底层的系统调用，允许更细粒度的控制子进程的创建。它可以创建一个具有独立内存空间、文件描述符等的子进程。

### 17、怎么区分子进程和父进程？

参考上面给出的示例代码，可根据返回值进行区分。

### 18、进程间的通信方式？

#### 1. 管道（Pipe）

##### 1.1 无名管道（Anonymous Pipe）

- 特点：
  - 半双工通信，数据只能单向流动。
  - 只能用于具有亲缘关系的进程间通信，通常是父子进程或兄弟进程。
  - 可以看作是一种特殊的文件，但不属于任何文件系统，只存在于内存中。
- 使用场景：
  - 父子进程间的简单数据传递。
  - Shell命令中的命令管道（如 `ls | grep`）。

```c
#include <stdio.h>
#include <unistd.h>
#include <string.h>

int main() {
    int pipefd[2];
    pid_t pid;
    char buffer[128];

    if (pipe(pipefd) == -1) {
        perror("pipe");
        return 1;
    }

    pid = fork();
    if (pid < 0) {
        perror("fork");
        return 1;
    } else if (pid == 0) {
        // 子进程
        close(pipefd[0]); // 关闭读端
        const char *message = "Hello from child";
        write(pipefd[1], message, strlen(message) + 1);
        close(pipefd[1]);
    } else {
        // 父进程
        close(pipefd[1]); // 关闭写端
        read(pipefd[0], buffer, sizeof(buffer));
        printf("Received message: %s\n", buffer);
        close(pipefd[0]);
    }

    return 0;
}
```

##### 1.2 命名管道（Named Pipe）

- 特点：
  - 半双工通信，数据只能单向流动。
  - 允许无亲缘关系的进程间通信。
  - 提供一个路径名与之关联，以文件形式存储在文件系统中。
- 使用场景：
  - 任意进程间的通信。
  - 跨进程的简单数据传递。

```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>

int main() {
    const char *fifo = "/tmp/myfifo";
    char buffer[128];

    if (mkfifo(fifo, 0666) == -1) {
        perror("mkfifo");
        return 1;
    }

    pid_t pid = fork();
    if (pid < 0) {
        perror("fork");
        return 1;
    } else if (pid == 0) {
        // 子进程
        int fd = open(fifo, O_WRONLY);
        const char *message = "Hello from FIFO";
        write(fd, message, strlen(message) + 1);
        close(fd);
    } else {
        // 父进程
        int fd = open(fifo, O_RDONLY);
        read(fd, buffer, sizeof(buffer));
        printf("Received message: %s\n", buffer);
        close(fd);
    }

    unlink(fifo);
    return 0;
}
```

#### 2. 消息队列（Message Queue）

- 特点：
  - 消息队列是由消息的链表组成，存放在内核中并由消息队列标识符标识。
  - 允许一个或多个进程向它写入或读取消息。
  - 消息可以按类型读取，不一定是先进先出。
- 使用场景：
  - 多进程间的数据交换。
  - 需要按类型读取消息的场景。

```c
#include <stdio.h>
#include <sys/msg.h>
#include <string.h>

struct msgbuf {
    long mtype;
    char mtext[128];
};

int main() {
    key_t key = ftok("msgqfile", 'a');
    int msgid = msgget(key, 0666 | IPC_CREAT);
    if (msgid == -1) {
        perror("msgget");
        return 1;
    }

    struct msgbuf msg;
    strcpy(msg.mtext, "Hello from message queue");
    msg.mtype = 1;

    if (msgsnd(msgid, &msg, sizeof(msg.mtext), 0) == -1) {
        perror("msgsnd");
        return 1;
    }

    if (msgrcv(msgid, &msg, sizeof(msg.mtext), 1, 0) == -1) {
        perror("msgrcv");
        return 1;
    }

    printf("Received message: %s\n", msg.mtext);

    if (msgctl(msgid, IPC_RMID, NULL) == -1) {
        perror("msgctl");
        return 1;
    }

    return 0;
}
```

#### 3. 共享内存（Shared Memory）

- 特点：
  - 允许两个或多个进程共享同一段内存区域。
  - 是最快的 IPC 方式，适合大数据量传输。
  - 通常与其他同步机制（如信号量）配合使用。
- 使用场景：
  - 大数据量的快速传输。
  - 高性能的多进程协作。

```c
#include <stdio.h>
#include <sys/shm.h>
#include <string.h>

int main() {
    key_t key = ftok("shmfile", 'a');
    int shmid = shmget(key, 1024, 0666 | IPC_CREAT);
    if (shmid == -1) {
        perror("shmget");
        return 1;
    }

    char *data = (char *)shmat(shmid, (void *)0, 0);
    if (data == (char *)(-1)) {
        perror("shmat");
        return 1;
    }

    strcpy(data, "Hello from shared memory");

    printf("Data in shared memory: %s\n", data);

    if (shmdt(data) == -1) {
        perror("shmdt");
        return 1;
    }

    if (shmctl(shmid, IPC_RMID, NULL) == -1) {
        perror("shmctl");
        return 1;
    }

    return 0;
}
```

#### 4. 信号量（Semaphore）

- 特点：
  - 用于进程间的同步，控制多个进程对共享资源的访问。
  - 是一个计数器，可以用来实现互斥锁和信号量集。
- 使用场景：
  - 控制对共享资源的访问。
  - 实现进程间的同步。

```c
#include <stdio.h>
#include <sys/sem.h>
#include <sys/ipc.h>

int main() {
    key_t key = ftok("semfile", 'a');
    int semid = semget(key, 1, 0666 | IPC_CREAT);
    if (semid == -1) {
        perror("semget");
        return 1;
    }

    union semun {
        int val;
        struct semid_ds *buf;
        unsigned short *array;
    } arg;

    arg.val = 1;
    if (semctl(semid, 0, SETVAL, arg) == -1) {
        perror("semctl");
        return 1;
    }

    struct sembuf sem_op;
    sem_op.sem_num = 0;
    sem_op.sem_op = -1; // P operation
    sem_op.sem_flg = 0;

    if (semop(semid, &sem_op, 1) == -1) {
        perror("semop");
        return 1;
    }

    // Critical section
    printf("Critical section entered\n");

    sem_op.sem_op = 1; // V operation
    if (semop(semid, &sem_op, 1) == -1) {
        perror("semop");
        return 1;
    }

    if (semctl(semid, 0, IPC_RMID, NULL) == -1) {
        perror("semctl");
        return 1;
    }

    return 0;
}
```

#### 5. 信号（Signal）

- 特点：
  - 用于通知接收进程某个事件已经发生。
  - 信号是一种异步通信方式，处理信号的函数称为信号处理器。
- 使用场景：
  - 处理外部事件（如键盘中断、定时器到期）。
  - 实现进程间的简单通知。

```c
#include <stdio.h>
#include <signal.h>
#include <unistd.h>

void signal_handler(int signum) {
    printf("Caught signal %d\n", signum);
}

int main() {
    signal(SIGINT, signal_handler);

    while (1) {
        printf("Waiting for signal...\n");
        sleep(1);
    }

    return 0;
}
```

#### 6. 套接字（Socket）

- 特点：
  - 支持不同主机上的进程间通信。
  - 可以用于本地主机进程间通信。
  - 支持多种通信协议（如TCP、UDP）。
- 使用场景：
  - 跨网络的进程间通信。
  - 本地主机进程间通信。

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>

#define PORT 8080
#define BUFFER_SIZE 1024

int main() {
    int server_fd, new_socket, valread;
    struct sockaddr_in address;
    int opt = 1;
    int addrlen = sizeof(address);
    char buffer[BUFFER_SIZE] = {0};
    const char *hello = "Hello from server";

    if ((server_fd = socket(AF_INET, SOCK_STREAM, 0)) == 0) {
        perror("socket failed");
        exit(EXIT_FAILURE);
    }

    if (setsockopt(server_fd, SOL_SOCKET, SO_REUSEADDR | SO_REUSEPORT, &opt, sizeof(opt))) {
        perror("setsockopt");
        exit(EXIT_FAILURE);
    }

    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);

    if (bind(server_fd, (struct sockaddr *)&address, sizeof(address)) < 0) {
        perror("bind failed");
        exit(EXIT_FAILURE);
    }

    if (listen(server_fd, 3) < 0) {
        perror("listen");
        exit(EXIT_FAILURE);
    }

    if ((new_socket = accept(server_fd, (struct sockaddr *)&address, (socklen_t *)&addrlen)) < 0) {
        perror("accept");
        exit(EXIT_FAILURE);
    }

    valread = read(new_socket, buffer, BUFFER_SIZE);
    printf("Message from client: %s\n", buffer);
    send(new_socket, hello, strlen(hello), 0);
    printf("Hello message sent\n");

    close(new_socket);
    close(server_fd);
    return 0;
}
```

### 19、手撕：最长无重复字符子串

这道题可以用经典的滑动窗口法，它的基本思想是使用两个指针，分别代表窗口的左右边界，这两个指针都从字符串的起始位置出发。随着右指针的移动，窗口逐渐扩大，直到窗口内出现重复字符为止。此时，需要移动左指针以缩小窗口，直到窗口内的字符再次变得唯一。在此过程中，记录窗口的最大长度，即为所求的最长无重复字符子串的长度。

#### 思路

1. 初始化两个指针 `left` 和 `right`，都指向字符串的起始位置。
2. 使用一个哈希表（`unordered_map`）来记录字符最近出现的位置。
3. 移动右指针`right`，检查当前字符是否已经在哈希表中存在。
   - 如果存在，说明窗口内出现了重复字符，此时需要更新左指针 `left` 的位置，使其指向重复字符的下一个位置。
   - 更新哈希表中当前字符的位置。
4. 计算当前窗口的长度，并更新最长子串的长度。
5. 当右指针到达字符串末尾时，停止循环，返回最长子串的长度。

#### 参考代码

```c
#include <iostream>
#include <unordered_map>
#include <algorithm>

class Solution {
public:
    int lengthOfLongestSubstring(std::string s) {
        std::unordered_map<char, int> charIndexMap; // 用于记录字符最近出现的位置
        int maxLength = 0; // 记录最长子串的长度
        int left = 0; // 窗口的左边界

        for (int right = 0; right < s.size(); ++right) {
            // 如果当前字符已经存在于窗口中
            if (charIndexMap.find(s[right]) != charIndexMap.end() && charIndexMap[s[right]] >= left) {
                // 更新左边界
                left = charIndexMap[s[right]] + 1;
            }
            // 更新当前字符的最新位置
            charIndexMap[s[right]] = right;
            // 计算当前窗口的长度，并更新最大长度
            maxLength = std::max(maxLength, right - left + 1);
        }

        return maxLength;
    }
};

int main() {
    Solution sol;
    std::string input = "abcabcbb";
    int result = sol.lengthOfLongestSubstring(input);
    std::cout << "最长无重复字符子串的长度为: " << result << std::endl;
    return 0;
}
```