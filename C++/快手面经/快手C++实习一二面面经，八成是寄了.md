# 快手C++实习一二面面经，八成是寄了。。

> 来源：https://www.nowcoder.com/feed/main/detail/bd803b3753f343bf9750b7201414d0c9

# 快手C++实习面经

## 一面（45min）:

### 八股+手撕：

#### 1、URL输入后发生了哪些事情，详细介绍步骤？

1. 用户输入URL： 用户在浏览器的地址栏中输入URL（统一资源定位符），这是他们希望访问的网页的地址。
2. DNS解析： 浏览器首先需要解析URL中的主机名部分，以确定目标服务器的IP地址。这个过程称为DNS解析。如果浏览器有缓存，它可能首先查看本地缓存以获取IP地址。如果不在缓存中，浏览器将向本地DNS服务器发出请求，以获取目标主机的IP地址。本地DNS服务器可以向根DNS服务器、顶级域名服务器和权威DNS服务器逐级查询，最终找到目标主机的IP地址。
3. 建立TCP连接： 一旦浏览器获得了目标服务器的IP地址，它将尝试建立到该服务器的TCP连接。这个过程通常涉及三次握手，即浏览器向服务器发送一个连接请求，服务器确认请求并回复，然后浏览器再次确认。一旦建立了TCP连接，数据可以在浏览器和服务器之间进行可靠的双向通信。
4. 发起HTTP请求： 一旦TCP连接建立，浏览器会构建一个HTTP请求，该请求包括用户想要获取的特定页面的信息。HTTP请求通常包括请求方法（如GET、POST等）、请求头部和请求主体。
5. 服务器处理请求： 服务器接收到浏览器发送的HTTP请求后，会根据请求的内容执行相应的操作。这可能涉及查询数据库、处理业务逻辑、生成动态内容等。
6. 服务器发送HTTP响应： 服务器会生成一个HTTP响应，该响应包含请求的内容（通常是HTML页面）、状态码（表示请求是否成功、重定向等）以及响应头部信息。服务器还将此HTTP响应发送回浏览器。
7. 接收和渲染页面： 一旦浏览器收到HTTP响应，它会解析响应的内容，并根据HTML、CSS和JavaScript等数据渲染页面。这包括呈现文本、图像、链接和其他媒体，以及执行JavaScript代码。
8. 页面展示： 最终，浏览器将渲染后的页面呈现给用户，用户可以与页面进行交互。
9. 页面缓存： 浏览器通常会将页面的一些资源（如图像、样式表、脚本等）缓存，以便在用户再次访问相同页面时能够更快地加载。

#### 2、https相关的TLS连接？

1. 握手阶段：客户端发起连接请求，服务器返回证书。客户端验证证书的有效性，包括验证证书是否由可信的证书颁发机构颁发，证书是否过期等。如果验证通过，客户端生成一个随机数，用服务器的公钥加密后发送给服务器。
2. 密钥协商：服务器收到客户端发来的随机数后，使用自己的私钥解密得到该随机数，并生成一个新的随机数，用客户端的公钥加密后发送给客户端。客户端收到服务器发来的随机数后，使用自己的私钥解密得到该随机数。
3. 加密通信：客户端和服务器分别使用前面协商好的随机数作为密钥，采用对称加密算法（如AES）进行通信数据的加密和解密。
4. 数据传输：客户端和服务器使用协商好的密钥进行加密和解密，保证了数据在传输过程中的保密性和完整性。
5. 断开连接：通信结束后，客户端和服务器可以选择关闭连接。

![TLS握手流程](https://raw.githubusercontent.com/aqjsp/Pictures/main/202402042306258.jpeg)

#### 3、TCP连接的三次握手 为什么是三次 不是两次 四次挥手 为什么是四次 ？

##### 三次握手

在建立连接之前，Client处于CLOSED状态，而Server处于LISTEN的状态。

1. 第一次握手：客户端主动给服务端发送一个SYN报文，并携带自己的初始化序列号一起发送给服务端。此时客户端处于一个SYN_SEND的状态。
2. 第二次握手：服务端收到客户端发来的SYN报文之后，就会以自己的SYN报文作为应答，然后将自己的初始化序列号发送给客户端，并且会将客户端的初始化序列号+1作为自己的ACK值发送给客户端，以表示自己已经收到了客户端的SYN报文。此时服务端处于一个SYN_RECV的状态。
3. 第三次握手：客户端收到服务端发来的SYN报文之后，会把服务端的初始化序列号+1作为ACK值发送给服务端，用来表示自己已经收到了服务端发来的SYN报文。此时客户端处于一个ESTABLISHED的状态。

![三次握手](https://raw.githubusercontent.com/aqjsp/Pictures/main/202402042308488.png)

##### 为什么是三次不是两次？

###### 三次的原因

![流程图](https://raw.githubusercontent.com/aqjsp/Pictures/main/202402042311957.jpg)

如图所示它主要是为了起始数据字节编号协商1是我发送的报文序号是456,2是我接受到了你下次从457开始发，3是好的我准备接收124。如果没有这个3号报文B端就不知道下次从哪开始发。

###### 为什么不是两次？

![image-20240204231135624](https://raw.githubusercontent.com/aqjsp/Pictures/main/202402042311137.png)

一开始A向B发出建立连接的请求但是这个报文超时了，A又向B发送了一个请求连接报文然后通过两次握手建立连接传送数据最后释放连接。但是这时候，超时的报文迟到发送他们又会建立连接然后服务端等待发送数据，但是这时候客户端已经没有数据可发了。

##### 四次挥手

![](https://raw.githubusercontent.com/aqjsp/Pictures/main/202402042322881.png)

1. 客户端向服务端发送一个报文FIN为1，序号为u然后进入FIN-WAIT1状态。
2. 服务端向客户端发送确认报文序号为v，确认序号为u+1然后进入CLOSE-WAIT状态。
3. 客户端收到服务端发回的确认报文之后进入FIN-WAIT2状态此时客户端连接已经关闭客户端无法向服务端传送数据。
4. 然后服务端被动关闭它向客户端发送一个FIN为1的报文段要求释放服务端到客户端的连接。进入LAST-ACK等待客户端发送最后一个ACK报文。
5. 客户端发送最后一次挥手确认报文然后进行closed，服务端直接CLOSED。
6. 客户端要等待2MSL才CLOSED。

当不按套路出牌时返回RST比如上来没建立连接服务端给客户端来一个ACK或FIN。

##### 为什么是四次？

1. 全双工通信：TCP 是一种全双工通信协议，允许在同一时间里双方都可以发送和接收数据。因此，在关闭连接时，需要分别关闭双方的发送通道和接收通道，这就需要四次挥手来完成。
2. 确认关闭：在四次挥手中，每一次挥手都需要得到对方的确认，这样可以确保双方都知道对方已经关闭了发送通道。如果只有三次挥手，就无法保证对方已经收到关闭请求并进行了相应的处理。
3. 数据传输完整性：在四次挥手中，最后一次挥手是为了确保双方都已经完成了数据的传输和接收，可以安全地关闭连接。如果只有三次挥手，无法保证双方都已经完成了数据的传输，可能会导致数据丢失或不完整。

#### 4、操作系统中的缺页中断是什么？

缺页中断（Page Fault）是操作系统中的一种异常情况，发生在程序访问虚拟内存中的某个页面（Page）时，但该页面并未加载到物理内存中。这种情况通常发生在使用虚拟内存管理的系统中，即将虚拟地址空间映射到物理地址空间的过程中。

当发生缺页中断时，操作系统会执行以下步骤：

1. 中断处理：CPU收到缺页中断信号后，会暂停当前正在执行的指令，切换到内核态。
2. 异常处理：操作系统内核会根据异常的原因进行处理。在缺页中断的情况下，操作系统会根据缺页地址查找页表（Page Table），判断该页面是否已经加载到物理内存中。
3. 页表查找：如果页表中记录了该页面的物理地址，则表示该页面已经在物理内存中，操作系统会更新页表中的相关信息（如访问位、修改位等），并重新执行导致缺页中断的指令，以完成对该页面的访问操作。
4. 页面调入：如果页表中未记录该页面的物理地址，表示该页面尚未加载到物理内存中，需要进行页面调入（Page In）操作。操作系统会选择一个物理页面用于存储该虚拟页面的内容，并将该页面从磁盘读取到物理内存中。在页面调入完成后，操作系统会更新页表中的相关信息，并重新执行导致缺页中断的指令。
5. 继续执行：当页面调入完成后，操作系统会重新执行导致缺页中断的指令，以完成对该页面的访问操作。

#### 5、TCP 和 UDP区别 举例说明具体的应用场景？

1. 连接性
   - TCP是一种连接导向的协议，它要求在通信之前建立一个连接。连接建立后，数据传输是可靠的，有序的，并且可以进行双向通信。
   - UDP是一种无连接协议，不需要建立连接。每个UDP数据包都是独立的，没有与之前或之后的数据包的关联。这使得UDP更加轻量级，但不可靠。
2. 可靠性
   - TCP提供可靠的数据传输，确保数据的完整性和顺序。如果数据包丢失或损坏，TCP会自动重传丢失的数据，直到接收方确认收到。
   - UDP不提供可靠性保证。数据包可能会丢失、重复或乱序，而且UDP不执行自动重传。
3. 流量控制
   - TCP使用滑动窗口和拥塞控制算法来进行流量控制，以避免过载网络和接收方。
   - UDP不执行流量控制，发送方可以以任何速率发送数据，而不受接收方的限制。
4. 头部开销
   - TCP头部相对较大，通常包含20字节的头部信息。这增加了数据传输的开销。
   - UDP头部相对较小，通常包含8字节的头部信息。这使得UDP在某些情况下更高效。
5. 应用场景
   - TCP适用于需要可靠数据传输的应用，如网页浏览、文件传输、电子邮件等。它也适用于对数据的顺序性有要求的应用。
   - UDP适用于实时性要求高、数据丢失不会引起问题的应用，如音频/视频流传输、在线游戏、DNS查询等。
6. 连接状态
   - TCP维护连接状态，包括三次握手建立连接和四次握手终止连接。
   - UDP没有连接状态，每个数据包都是独立的。
7. 性能开销
   - 由于提供可靠性和流控，TCP的性能开销较高，适用于低延迟和高可靠性的场景。
   - UDP的性能开销较低，适用于高吞吐量和实时性要求高的场景。

#### 6、多线程保证线程安全的方式，具体C++中的哪些实现方法？

1. 互斥锁（Mutex）：使用互斥锁来保护共享资源，确保同一时刻只有一个线程可以访问共享资源。在C++中，可以使用 `std::mutex` 来实现互斥锁。

```
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx;

void threadFunction() {
    std::lock_guard<std::mutex> lock(mtx); // 自动加锁
    // 访问共享资源的代码
}

int main() {
    std::thread t1(threadFunction);
    std::thread t2(threadFunction);
    t1.join();
    t2.join();
    return 0;
}
```

2. 读写锁（Read-Write Lock）：对于读多写少的场景，可以使用读写锁来提高并发性能。读写锁允许多个线程同时读取共享资源，但只允许一个线程写入共享资源。在C++中，可以使用 `std::shared_mutex`（C++17引入）来实现读写锁。

3. 原子操作：对于简单的操作（如增加计数器），可以使用原子操作来保证操作的原子性。在C++中，可以使用 `std::atomic` 模板来定义原子操作的变量。

```
#include <iostream>
#include <thread>
#include <atomic>

std::atomic<int> counter(0);

void threadFunction() {
    counter++; // 原子增加操作
}

int main() {
    std::thread t1(threadFunction);
    std::thread t2(threadFunction);
    t1.join();
    t2.join();
    std::cout << "Counter: " << counter << std::endl;
    return 0;
}
```

4. 条件变量（Condition Variable）：用于线程间的通信，允许线程等待某个条件成立后再继续执行。在C++中，可以使用 `std::condition_variable` 来实现条件变量。

```
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
bool ready = false;

void threadFunction() {
    std::unique_lock<std::mutex> lock(mtx);
    while (!ready) {
        cv.wait(lock); // 等待条件变量
    }
    // 条件成立后继续执行
}

int main() {
    std::thread t(threadFunction);
    {
        std::lock_guard<std::mutex> lock(mtx);
        ready = true;
    }
    cv.notify_one(); // 唤醒等待线程
    t.join();
    return 0;
}
```

#### 7、http1.0 和 1.1 区别？

1. 持久连接（Persistent Connection）：HTTP/1.0 默认使用非持久连接，即每次请求都需要建立新的TCP连接，完成请求后立即关闭连接。而HTTP/1.1 默认使用持久连接，允许在单个TCP连接上发送和接收多个HTTP请求和响应，减少了连接建立和关闭的开销，提高了性能。
2. 管道化（Pipeline）：HTTP/1.1 支持管道化，即在一个TCP连接上可以发送多个请求而无需等待响应。这样可以减少网络延迟，提高了并发性能。HTTP/1.0 不支持管道化，每个请求必须等待前一个请求的响应后才能发送下一个请求。
3. 缓存控制（Cache Control）：HTTP/1.1 引入了更强大的缓存控制机制，包括了更多的缓存控制头字段（如`Cache-Control`、`ETag`、`If-Modified-Since`等），能够更精确地控制缓存的行为，提高了缓存效率。
4. Host 头字段：HTTP/1.1 引入了`Host`头字段，用于指定请求的目标主机，使得同一台服务器可以承载多个域名的网站。
5. 错误处理：HTTP/1.1 对错误处理机制进行了改进，引入了更多的状态码（如`206 Partial Content`、`416 Requested Range Not Satisfiable`等），提高了对各种情况的处理能力。
6. 其他改进：HTTP/1.1 还对请求和响应的格式进行了调整，引入了更多的头字段（如`Range`、`Content-Length`等），增加了对压缩（如gzip、deflate）和分块传输编码（Chunked Transfer Encoding）的支持，提高了数据传输的效率和灵活性。

#### 8、手撕：手撕线程池

这里就讲讲正常的一个线程池的实现步骤。

1. 定义任务类：首先需要定义一个任务类，用于封装需要在线程池中执行的任务。任务类至少应该包含一个执行任务的方法，可以是一个函数指针或者是一个函数对象。

   ```
   class Task {
   public:
       virtual void execute() = 0;
   };
   ```

2. 定义线程池类：接下来定义线程池类，其中包含了线程池的管理逻辑，如线程的创建、销毁、任务的添加等。线程池类需要包含一个线程池容器，用于存放线程对象。

   ```
   #include <vector>
   #include <thread>
   #include <queue>
   #include <mutex>
   #include <condition_variable>
   
   class ThreadPool {
   public:
       ThreadPool(size_t numThreads);
       ~ThreadPool();
   
       void addTask(Task* task);
   
   private:
       std::vector<std::thread> workers;  // 线程池中的线程
       std::queue<Task*> tasks;            // 任务队列
       std::mutex queueMutex;              // 保护任务队列的互斥量
       std::condition_variable condition;  // 用于线程间通信的条件变量
       bool stop;                          // 标志线程池是否停止的标志位
   };
   ```

3. 实现线程池类的构造函数和析构函数：在构造函数中创建指定数量的线程，并启动这些线程；在析构函数中停止线程池中的所有线程。

   ```
   ThreadPool::ThreadPool(size_t numThreads) : stop(false) {
       for (size_t i = 0; i < numThreads; ++i) {
           workers.emplace_back([this] {
               while (true) {
                   Task* task = nullptr;
                   {
                       std::unique_lock<std::mutex> lock(queueMutex);
                       condition.wait(lock, [this] { return stop || !tasks.empty(); });
                       if (stop && tasks.empty()) return;
                       task = tasks.front();
                       tasks.pop();
                   }
                   task->execute();
                   delete task;
               }
           });
       }
   }
   
   ThreadPool::~ThreadPool() {
       {
           std::unique_lock<std::mutex> lock(queueMutex);
           stop = true;
       }
       condition.notify_all();
       for (std::thread& worker : workers) {
           worker.join();
       }
   }
   ```

4. 实现添加任务的方法：在线程池类中添加一个方法用于向任务队列中添加任务。

   ```
   void ThreadPool::addTask(Task* task) {
       {
           std::unique_lock<std::mutex> lock(queueMutex);
           tasks.push(task);
       }
       condition.notify_one();
   }
   ```

5. 使用线程池：最后，在主程序中使用定义好的线程池类来执行任务。

   ```
   int main() {
       ThreadPool pool(4);  // 创建一个包含4个线程的线程池
   
       // 添加任务到线程池
       for (int i = 0; i < 8; ++i) {
           pool.addTask(new YourTask());  // YourTask 是需要执行的任务类
       }
   
       // ...
   
       return 0;
   }
   ```

## 二面（70min）

重点是C++的八股文 + 项目

### 1、new malloc的区别 至少说出4点以上，在申请内存的时候都做了哪些工作 申请内存的过程是否需要初始化

### 2、delete 和 delete [] 区别 如何对调使用会发生什么事情

### 3、动态多态的虚函数内部原理， 子类继承父类在动态多态中会调用谁的虚方法...

### 4、多线程在C++中保证线程安全的方式有哪些

### 5、多线程只读操作的时候需要加锁吗？

### 6、多个线程读 一个线程写需要加锁吗？

### 7、读写锁如何实现口述

### 8、8大排序方法的时间复杂度？ 口述归并排序和快排

### 9、map 和multimap unordered_map区别 为什么要有 unordered_map 使用场景是什么，这三者访问元素的时间复杂度 底层实现？


手撕： 

### 1、IP4V地址字符串转化为 32整型数字

### 2、词频统计 保证次数相同基础上优先字母排序打印 ACM模式

