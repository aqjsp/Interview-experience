# 百度C++ 二面凉经

> 来源：https://www.nowcoder.com/feed/main/detail/8d66c46082aa4145a3437d7f383ac1ec

### 1、手撕单例模式？（饿汉、懒汉）

1. 饿汉式单例模式（线程不安全）：

   - 在类的静态成员变量中直接创建实例，并在类的静态方法中返回该实例。
   - 这种方式在程序启动时就会创建单例对象，无论是否需要使用，可能会导致资源浪费。
   - 不适合在多线程环境下使用，因为没有进行线程安全的处理。

   ```c++
   class Singleton {
   public:
       static Singleton& getInstance() {
           static Singleton instance;
           return instance;
       }
   
       // 防止拷贝构造和赋值操作
       Singleton(const Singleton&) = delete;
       Singleton& operator=(const Singleton&) = delete;
   
   private:
       Singleton() {}  // 私有化构造函数，禁止外部创建实例
   };
   ```

2. 懒汉式单例模式（线程不安全）：

   - 在第一次调用时才创建单例对象，避免了在程序启动时就创建对象的资源浪费。
   - 不适合在多线程环境下使用，因为没有进行线程安全的处理。

   ```c++
   class Singleton {
   public:
       static Singleton& getInstance() {
           static Singleton instance;
           return instance;
       }
   
       // 防止拷贝构造和赋值操作
       Singleton(const Singleton&) = delete;
       Singleton& operator=(const Singleton&) = delete;
   
   private:
       Singleton() {}  // 私有化构造函数，禁止外部创建实例
   };
   ```

3. 懒汉式单例模式（线程安全）：

   - 使用加锁的方式保证在多线程环境下也能正常工作，但会影响性能。
   - 在 `getInstance` 方法中加锁，避免了多个线程同时创建实例的问题。

   ```c++
   #include <mutex>
   
   class Singleton {
   public:
       static Singleton& getInstance() {
           std::lock_guard<std::mutex> lock(mutex);
           static Singleton instance;
           return instance;
       }
   
       // 防止拷贝构造和赋值操作
       Singleton(const Singleton&) = delete;
       Singleton& operator=(const Singleton&) = delete;
   
   private:
       Singleton() {}  // 私有化构造函数，禁止外部创建实例
       static std::mutex mutex;
   };
   
   std::mutex Singleton::mutex;
   ```

4. Meyers' Singleton（线程安全）：（不常用）

   - 利用 C++11 的特性，在静态变量的初始化阶段进行初始化，保证了线程安全性。
   - 使用静态局部变量的特性，在第一次调用 `getInstance` 方法时才进行实例化。

   ```c++
   class Singleton {
   public:
       static Singleton& getInstance() {
           static Singleton instance;
           return instance;
       }
   
       // 防止拷贝构造和赋值操作
       Singleton(const Singleton&) = delete;
       Singleton& operator=(const Singleton&) = delete;
   
   private:
       Singleton() {}  // 私有化构造函数，禁止外部创建实例
       ~Singleton() {}
   };
   ```


### 2、voliate关键字用处？（可见性、有序性）

`volatile` 关键字在 C++ 中用于告知编译器，某个变量的值可能会被外部因素（如硬件、其他线程或中断）修改，因此编译器不应对该变量进行优化。这使得每次访问该变量时，程序都会重新从内存中读取其最新值，而不是使用寄存器或缓存的值。

#### `volatile` 关键字的主要作用

1. 防止编译器优化：

   编译器在优化时，可能会将某个变量的值缓存到寄存器中，避免频繁访问内存。如果这个变量的值可能会在程序控制之外被改变（例如在多线程或硬件设备中），这样的优化可能会导致程序读取的是旧值。`volatile` 告诉编译器，每次访问这个变量都必须从内存读取，而不是使用缓存的值。

2. 用于多线程编程：

   - 在多线程编程中，如果一个变量可能会被其他线程修改，那么使用 `volatile` 可以确保每个线程在读取这个变量时都会获取最新值。
   - 需要注意的是，`volatile` 只能确保读取的是最新的值，但不能确保操作的原子性或内存可见性。因此，`volatile` 并不能代替 `mutex` 或其他同步机制来保证线程安全。

3. 硬件寄存器编程：

   在嵌入式系统或与硬件打交道时，某些变量可能对应硬件寄存器的值。这些值可能会被外部硬件改变，而不是通过程序控制修改。`volatile` 确保每次访问这些变量时，都会重新读取内存中的值，而不是使用编译器优化后的缓存值。

#### 使用 `volatile` 的典型场景

1. 硬件寄存器：在与硬件交互时，例如从设备读取状态寄存器的数据，寄存器的值可能随时变化，使用 `volatile` 可以防止编译器优化掉这些读取操作。

   ```c++
   volatile int* statusRegister = (int*)0x40001000; // 假设是硬件寄存器地址
   while (*statusRegister != READY) {
       // 等待状态变化
   }
   ```

2. 共享内存（多线程）：一个线程可能会修改一个变量，而另一个线程则需要不断读取这个变量的变化。在这种情况下，`volatile` 可以防止编译器优化导致读取不到最新值。

   ```c++
   volatile bool stopThread = false;
   
   void threadFunction() {
       while (!stopThread) {
           // 执行任务
       }
   }
   ```

3. 中断服务程序：在中断处理程序中，某些变量可能会被中断处理程序修改，但主程序中也会访问这些变量，使用 `volatile` 可以防止编译器优化。

   ```c++
   volatile bool interruptFlag = false;
   
   void interruptHandler() {
       interruptFlag = true;
   }
   ```

#### `volatile` 的局限性

- 不提供原子性：`volatile` 并不能保证对变量的操作是原子的，特别是在多线程环境中。例如，`volatile` 不能确保对 `volatile int` 的加减操作是线程安全的。
- 不提供同步：`volatile` 仅仅是防止编译器优化，无法提供线程间的同步。如果两个线程同时访问一个 `volatile` 变量，仍然需要使用同步机制（如互斥锁）来保证线程安全。

### 3、手撕sql查询，在一个(学生、课程、分数中)查询所有平均分不及格的学生id和平均分？

假设有一个表结构如下：

- `students`：学生表
  - `student_id`：学生 ID
  - `student_name`：学生姓名
- `courses`：课程表
  - `course_id`：课程 ID
  - `course_name`：课程名称
- `scores`：成绩表
  - `student_id`：学生 ID
  - `course_id`：课程 ID
  - `score`：分数

问题要求查询平均分不及格的学生 `student_id` 和他们的平均分（假设及格线为 60 分）。

```sql
SELECT s.student_id, AVG(sc.score) AS avg_score
FROM students s
JOIN scores sc ON s.student_id = sc.student_id
GROUP BY s.student_id
HAVING AVG(sc.score) < 60;
```

### 4、一个SQL语句执行过程？

#### 1. 客户端发送 SQL 语句

首先，客户端（如应用程序或终端）将 SQL 查询语句发送到数据库服务器。

#### 2. 服务器接收并解析 SQL 语句

##### 2.1 语法解析

- 词法分析：数据库服务器对 SQL 语句进行词法分析，将 SQL 语句拆解为不同的标记（token），例如关键字、表名、字段名等。
- 语法分析：数据库服务器会检查 SQL 语句的语法是否正确。若语法有误，则会返回语法错误，执行过程就会在此停止。

##### 2.2 查询优化器

- 解析树：数据库通过词法、语法分析生成一棵解析树（Parse Tree）。
- 逻辑优化：优化器会根据解析树进行逻辑优化，判断查询是否可以改写为更高效的形式，比如合并 WHERE 条件、简化表达式等。
- 物理优化：优化器会根据表的大小、索引、统计信息等情况选择最优的执行计划。这个过程中，优化器可能会决定：
  - 是否使用索引来加速查询；
  - 是否进行表扫描；
  - 选择哪种连接算法（如嵌套循环、哈希连接等）；
  - 是否进行排序或合并等操作。

优化器的目标是生成一个最优的查询执行计划（Execution Plan），以最小的资源消耗完成查询。

#### 3. 执行计划生成与执行

- 生成执行计划：根据优化器提供的最优执行路径，生成具体的执行计划。
- 执行 SQL 语句：数据库引擎开始根据执行计划执行相应的操作，具体包括：
  - 从磁盘或内存中读取数据；
  - 利用索引查找数据；
  - 进行表连接（如果查询涉及多个表）；
  - 进行排序、过滤、聚合等操作。

在这个阶段，数据库会访问存储引擎（如 MyISAM、InnoDB），存储引擎负责从物理磁盘中检索数据或更新数据。

#### 4. 返回结果集

一旦查询执行完成，结果集会被返回给客户端。具体的步骤如下：

- 将执行结果从存储引擎中返回；
- 如果查询有 LIMIT、OFFSET 等限制，会根据这些限制返回结果；
- 最终，数据库服务器将结果集返回给客户端。

### 5、MySQL用长连接有什么好处吗？

#### 1. 减少连接建立的开销

每次创建或断开 MySQL 数据库的连接都需要一定的资源开销，尤其是当数据库连接频繁时，短连接（Short Connection）方式会导致性能下降。长连接可以避免每次执行 SQL 时都重新建立和关闭连接，从而减少这些频繁的操作。

- 连接建立开销：连接到 MySQL 服务器需要进行 TCP 握手、认证、分配资源等，这些操作虽然每次都很快，但在高并发场景下，短连接的频繁建立和断开会增加系统开销，导致性能瓶颈。
- 长连接优势：使用长连接可以在客户端和数据库之间保持连接，只要会话有效，就不必重复创建连接，减少了连接开销。

#### 2. 提高数据库性能

频繁建立和关闭连接会增加数据库的负担，特别是在高并发环境下，频繁的连接和断开可能导致 MySQL 服务器需要频繁分配和释放资源。通过长连接，减少了这种资源的频繁占用和释放，进而提升整体系统性能。

#### 3. 提高连接池效率

在某些应用场景中，使用长连接结合连接池可以有效管理数据库连接的复用。连接池可以维护一定数量的活跃连接，并将这些连接分配给不同的请求。通过长连接，连接池中的连接可以持续保持，并且在多次请求之间复用，从而减少创建新连接的开销。

- 连接复用：长连接使得数据库连接能够在多次数据库操作中复用，从而减少频繁的连接和断开操作。
- 并发支持：对于高并发场景，长连接和连接池结合能够更好地支持大量并发请求，同时保持高性能。

#### 4. 减少网络传输延迟

每次短连接都需要重新进行连接认证、握手等过程，这些步骤在网络通信中会产生延迟。使用长连接可以在应用程序和数据库之间保持一个持续的连接，减少多次握手和网络传输的延迟，从而提高响应速度。

#### 5. 优化大批量操作

长连接适用于需要执行一系列连续 SQL 语句的情况，例如批量插入、更新或删除数据。由于长连接不需要每次都重新连接数据库，因此对于这类操作可以减少不必要的连接开销。

#### 可能的问题

虽然长连接有这些好处，但也需要注意长连接可能带来的一些问题：

1. 连接资源耗尽：长连接如果不及时断开，可能导致连接数过多，消耗大量的 MySQL 服务器资源。如果连接数超过 MySQL 的配置上限，可能会导致新连接无法创建。解决办法是使用连接池，并设置合理的最大连接数。
2. 内存占用：由于每个 MySQL 连接都会占用一些内存和资源，长时间不释放连接会导致服务器内存占用增加。因此在使用长连接时，需要定期释放不再使用的连接，或者使用合理的连接池管理。
3. 空闲连接问题：长连接如果空闲时间过长，可能导致 MySQL 的 wait_timeout 配置超时，服务器自动断开连接。此时，客户端再进行操作时会报错。因此，通常会设置超时检查或空闲连接检测机制。

### 6、ping命令用的什么协议，在哪一层？

`ping` 命令使用的是 **ICMP 协议**（Internet Control Message Protocol，因特网控制消息协议），它属于 **网络层** 协议（即 **OSI 模型的第三层** 或 TCP/IP 模型的 **网络层**）。

具体来说：

- ICMP 协议 是一种用于网络设备（如路由器、主机）之间传递控制消息的协议，通常用于报告网络连接问题或查询网络状态。它不是面向连接的协议，不提供数据传输功能，而是用于发送错误报告、网络诊断等消息。
- 网络层 是负责网络中不同节点间数据包的路由和转发的层。ICMP 协议作为网络层的一部分，与 IP 协议紧密结合，主要用于发送网络问题的反馈或测试网络的连通性。

在 `ping` 命令中，它会发送一个 **ICMP Echo Request**（回显请求）消息到目标主机，并等待接收 **ICMP Echo Reply**（回显应答）消息。如果目标主机收到请求并返回应答，说明该主机是可达的。

### 7、UDP怎么实现可靠传输？

#### 1. 添加确认机制（ACK）

- 在应用层协议中，可以引入与 TCP 类似的确认机制。
- 当接收方收到一个 UDP 数据包时，向发送方发送一个确认消息（ACK）。
- 发送方在规定的时间内未收到确认消息，就重发该数据包，直到收到 ACK 为止。
- 通过此机制可以确保数据包成功到达对方，避免丢包的影响。

#### 2. 重传机制

- 在发送数据包时，设置一个超时时间。如果发送方在该时间内没有收到接收方的确认消息，就假定数据包在传输过程中丢失或损坏，并重新发送。
- 可以设置一定的重发次数上限，避免无限重传。

#### 3. 序列号机制

- 在数据包中添加序列号，帮助接收方识别数据包的顺序，确保接收方可以按顺序处理数据包。
- 如果接收方发现序列号不连续，可以请求发送方重传缺失的数据包。
- 通过序列号，还能避免数据包重复接收的问题（即去重机制）。

#### 4. 超时与重发机制

- 发送方可以为每个数据包设置一个超时时间（Timer），如果在超时之前没有收到 ACK，就认为该数据包丢失，重新发送数据。
- 这个机制类似于 TCP 中的超时重传机制，能够减少网络抖动或暂时性丢包带来的影响。

#### 5. 滑动窗口机制

- 可以借鉴 TCP 的滑动窗口机制，发送方可以同时发送多个数据包，而不是每发送一个数据包都等待 ACK，直到接收方确认一批数据后再继续发送。
- 这种机制可以提高传输效率，特别是在高延迟网络中显得尤为重要。

### 6. **拥塞控制**

- UDP 本身没有拥塞控制机制，但在需要可靠传输时，可以在应用层加入类似 TCP 的拥塞控制逻辑。根据网络反馈（如 ACK 的延迟或丢包率），动态调整数据发送的速率，避免网络过载。

#### 7. 错误检测和校验

- UDP 包含简单的校验和（Checksum）用于检测数据包在传输中的损坏。为了更可靠地传输，可以在应用层引入更高级的错误检测机制，例如 CRC（循环冗余校验）或哈希校验。
- 接收方如果发现数据包的校验不通过，可以丢弃该包并通知发送方重传。

#### 8. 数据分片与重组

- 如果需要发送的数据量较大，可以将其分成多个小的 UDP 数据包进行传输，接收方收到数据后负责将其重组为完整的数据。
- 发送方需要将分片信息和序列号放在每个 UDP 包中，以便接收方按正确顺序重组。

#### 9. 冗余包和前向纠错（FEC）

- 在某些实时性要求较高的应用（如视频流、语音传输）中，重传可能不合适。这时可以通过发送冗余数据或使用前向纠错（FEC）技术来实现可靠传输。
- FEC 可以通过发送额外的校验包，允许接收方通过校验数据来恢复丢失的数据包，减少丢包带来的影响。

### 8、从一堆数中查找最大的10个数，应该怎么找？

#### 1. 排序法

最简单直接的方法是对这堆数进行排序，然后取出最大的10个数。

- 步骤：
  1. 对所有数进行排序（从大到小）。
  2. 取出排序后的前10个数。
- 时间复杂度：排序的时间复杂度通常是 O(nlog⁡n)，其中 n 是数组的长度。
- 适用场景：适合数据量较小，且要求一次性得到结果的场景。

#### 2. 维护一个最小堆

当数据量较大时，排序整个数组显得不够高效。此时可以使用 **最小堆**，只维护前10个最大的数。堆的大小始终为10，当遇到比堆顶元素大的数时，替换堆顶元素并重新调整堆。

- 步骤：
  1. 维护一个大小为10的最小堆。
  2. 遍历数组，将前10个数直接加入堆中，形成最小堆。
  3. 从第11个数开始，和堆顶（最小的数）进行比较，如果当前数比堆顶大，替换堆顶元素并调整堆。
  4. 最终堆中剩下的10个数就是最大的10个数。
- 时间复杂度：插入堆的时间复杂度为 O(log⁡k)，其中 k 是堆的大小，最大为10。遍历整个数组的复杂度为 O(nlog⁡10)，简化为 O(n)，因为 k=10是常数。
- 适用场景：数据量非常大时，这种方法更加高效，因为不需要对所有数据排序。

#### 3. 快速选择（基于快速排序的划分思想）

利用快速排序中的**分区思想**可以在 O(n) 的时间内找到前 k 个最大数（k=10）。通过类似于快速排序的分区操作，将数组分成两部分：大于某个数的一部分和小于某个数的一部分。

- 步骤：
  1. 使用快速选择算法递归或迭代地对数组进行划分。
  2. 在每次划分时，确定枢轴的位置，如果枢轴左边的元素数量小于等于10，则继续在右边部分划分，直到找到前10个最大的数。
- 时间复杂度：平均情况下，时间复杂度为 O(n)，最坏情况可能是 O(n2)，但在大多数情况下可以通过随机化枢轴来避免最坏情况。
- 适用场景：适合需要快速找到前10个最大数且不关心顺序的场景。

#### 4. 部分排序

有时候只需要找到前10个最大的数而不需要对整个数组排序，可以通过 STL 中的 `nth_element` 函数或类似的方法进行部分排序。

- 步骤：
  1. 使用 `nth_element` 将数组中的前10个元素放到数组的前10个位置，但这10个元素不一定是有序的。
  2. 再对这10个数进行一次排序以得到正确的顺序。
- 时间复杂度：O(n) 的复杂度用于将前10个数移动到数组的前面，接着用 O(klog⁡k) 的时间对这10个数排序。
- 适用场景：适用于只关心前10个最大数而不需要排序所有数据的场景。

### 9、linux看端口占用情况，都有什么命令？

#### 1. `netstat` 命令

`netstat` 是一个老牌工具，用于显示网络连接、路由表、接口状态、端口使用情况等。

- 查看端口占用情况：

  ```
  netstat -tuln
  ```

  - `-t`：显示 TCP 端口。
  - `-u`：显示 UDP 端口。
  - `-l`：显示监听的端口。
  - `-n`：以数字形式显示端口号和 IP 地址。

- 查看占用指定端口的进程：

  ```
  netstat -tulnp | grep <port_number>
  ```

  - `-p`：显示占用端口的进程及 PID。

#### 2. `ss` 命令

`ss` 是一个更现代化的工具，速度比 `netstat` 更快，功能类似。

- 查看端口占用情况：

  ```
  ss -tuln
  ```

  - `-t`：显示 TCP 端口。
  - `-u`：显示 UDP 端口。
  - `-l`：显示监听的端口。
  - `-n`：以数字形式显示端口号和 IP 地址。

- 查看占用指定端口的进程：

  ```
  ss -tulnp | grep <port_number>
  ```

  - `-p`：显示占用端口的进程及 PID。

#### 3. `lsof` 命令

`lsof`（list open files）用于列出当前系统中打开的文件和网络连接情况，端口实际上也属于文件。

- 查看某个端口的占用情况：

  ```
  lsof -i :<port_number>
  ```

  这会显示哪个进程在占用指定的端口。

- 查看所有端口的占用情况：

  ```
  lsof -i -P -n
  ```

  - `-P`：显示端口号而不是服务名称。
  - `-n`：不进行 DNS 解析，加快输出速度。

#### 4. `fuser` 命令

`fuser` 可以显示哪些进程在使用某个文件或套接字。

查看某个端口的占用情况：

```
fuser -n tcp <port_number>
```

这会显示占用指定 TCP 端口的进程 ID。

#### 5. `nmap` 命令

`nmap` 是一个网络扫描工具，可以扫描主机的开放端口。

- 扫描某个 IP 的端口占用情况：

  ```
  nmap <ip_address>
  ```

  - 这会显示指定 IP 地址的开放端口情况。

- 扫描本地主机的端口：

  ```
  nmap localhost
  ```

#### 6. `pidstat` 命令

如果你只关心具体进程的端口占用情况，`pidstat` 可以通过 PID 显示进程的网络使用情况。

使用 `pidstat` 查看网络占用情况：

```
pidstat -n -p <PID>
```

显示该进程网络接口的详细信息。

#### 7. `iptables` 命令

如果你想查看防火墙规则中是否有端口占用，可以通过 `iptables` 规则查询端口。

查看端口是否被防火墙规则拦截或允许：

```
sudo iptables -L -n -v | grep <port_number>
```

### 10、如何查看一个进程的使用情况？

#### 1. `top` 命令

`top` 是一个常用的实时系统监控工具，显示系统中运行的所有进程以及其资源使用情况。

- 查看某个进程的详细情况：

  ```
  top -p <PID>
  ```

  `PID` 是你要查看的进程 ID。`top` 会显示该进程的 CPU、内存使用、运行状态等信息。

- 交互操作：

  - 按 `M`：按内存使用排序。
  - 按 `P`：按 CPU 使用排序。
  - 按 `k`：杀死某个进程。

#### 2. `htop` 命令

`htop` 是 `top` 的一个更友好的版本，提供了彩色和图形化的进程显示，支持鼠标操作。

启动 `htop`：

```
htop
```

在 `htop` 界面中，你可以上下滚动，选择某个进程，并查看其资源使用情况。

#### 3. `ps` 命令

`ps` 用于显示系统当前的进程快照，它不会实时更新，需要重新执行命令来获取最新状态。

- 查看进程详细信息：

  ```
  ps aux | grep <PID>
  ```

  这会列出进程的 CPU 和内存使用情况，以及启动命令、运行时间等。

- 显示所有进程的详细信息：

  ```
  ps aux
  ```

  常见字段包括 `CPU%`（CPU 使用率）、`MEM%`（内存使用率）、`TIME`（运行时间）等。

#### 4. `pidstat` 命令

`pidstat` 是一个专门用来查看进程资源使用情况的命令。

- 查看某个进程的 CPU 使用情况：

  ```
  pidstat -p <PID> 1
  ```

  这会每秒显示一次指定进程的 CPU 使用情况。

- 查看某个进程的内存使用情况：

  ```
  pidstat -r -p <PID> 1
  ```

  显示进程的内存使用情况，包括虚拟内存和物理内存。

- 查看某个进程的 I/O 使用情况：

  ```
  pidstat -d -p <PID> 1
  ```

  显示进程的磁盘 I/O 使用情况。

#### 5. `pmap` 命令

`pmap` 可以用来查看进程的内存映射，包括详细的内存使用情况。

- 查看进程的内存使用情况：

  ```
  pmap <PID>
  ```

  `pmap` 会显示进程的内存区域分布，包括栈、堆、共享库等。

- 查看进程的内存汇总：

  ```
  pmap -x <PID>
  ```

  `-x` 选项会显示内存的详细信息，包括已使用、已分配等信息。

#### 6. `lsof` 命令

`lsof` 可以列出进程打开的文件（包括网络连接、设备等）。

查看进程打开的文件：

```
lsof -p <PID>
```

显示进程打开的所有文件，包括网络连接、文件句柄等。

#### 7. `strace` 命令

`strace` 是一个跟踪进程系统调用的工具，适合用于调试。

跟踪进程的系统调用：

```
strace -p <PID>
```

可以看到该进程执行的每一个系统调用以及其参数和返回值。

#### 8. `iotop` 命令

`iotop` 类似于 `top`，但专门用于显示进程的磁盘 I/O 情况。

查看进程的 I/O 使用情况：

```
iotop -p <PID>
```

该命令会显示进程的磁盘读写速率。

#### 9. `vmstat` 命令

`vmstat` 可以显示进程的内存、I/O、CPU等资源的使用情况。

查看进程的系统资源使用情况：

```
vmstat 1
```

这个命令每隔1秒刷新一次，显示整个系统的资源使用概况，可以通过综合信息推断某个进程的影响。

#### 10. `sar` 命令

`sar` 是系统活动报告工具，可以跟踪某一时间段的 CPU、内存、网络等资源的使用情况。

查看进程的 CPU 使用情况：

```
sar -p <PID> 1
```

显示进程的 CPU 使用情况，适合长时间监控。

### 11、手撕：二叉树的最大路径和

#### 递归

##### 思路

1. 定义路径和：二叉树中的路径可以是从任意节点到任意节点的路径。路径和是路径中各节点值的总和。
2. 递归遍历：
   - 可以使用递归遍历树的每一个节点来计算路径和。
   - 对于每个节点，需要计算以下几种路径和：
     - 以当前节点为根节点，且路径仅向左或向右延伸的最大路径和。
     - 以当前节点为根节点，且路径经过当前节点的最大路径和（即左右子树的最大路径和加上当前节点的值）。
3. 递归函数：
   - 定义一个递归函数 `maxGain`，它接受当前节点作为参数，并返回以当前节点为根节点的最大路径和。
   - 在递归函数中，需要考虑以下几个情况：
     - 当前节点的左子树的最大路径和。
     - 当前节点的右子树的最大路径和。
     - 以当前节点为根节点的路径和，即当前节点值 + 左子树的最大路径和 + 右子树的最大路径和。
   - 还需要维护一个全局变量 `maxSum` 来记录在遍历过程中遇到的最大路径和。

##### 参考代码

```c++
#include <iostream>
#include <algorithm>
#include <climits>

using namespace std;

// 定义二叉树节点结构
struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

// 全局变量，记录最大路径和
int maxSum = INT_MIN;

// 递归函数，计算以当前节点为根的最大路径和
int maxGain(TreeNode* node) {
    if (node == nullptr) {
        return 0;
    }

    // 递归计算左子树和右子树的最大路径和
    int leftGain = max(maxGain(node->left), 0); // 如果子树路径和小于 0，则取 0
    int rightGain = max(maxGain(node->right), 0); // 如果子树路径和小于 0，则取 0

    // 计算当前节点为根的最大路径和
    int currentMaxPathSum = node->val + leftGain + rightGain;

    // 更新全局最大路径和
    maxSum = max(maxSum, currentMaxPathSum);

    // 返回以当前节点为根的最大路径和
    return node->val + max(leftGain, rightGain);
}

// 计算二叉树的最大路径和
int maxPathSum(TreeNode* root) {
    maxSum = INT_MIN; // 初始化最大路径和为最小值
    maxGain(root); // 调用递归函数计算最大路径和
    return maxSum;
}

int main() {
    TreeNode* root = new TreeNode(-10);
    root->left = new TreeNode(9);
    root->right = new TreeNode(20);
    root->right->left = new TreeNode(15);
    root->right->right = new TreeNode(7);

    // 计算最大路径和
    cout << "最大路径和是: " << maxPathSum(root) << endl;

    return 0;
}
```

#### 非递归

##### 思路

1. 使用栈来模拟 DFS：

   - 使用栈来模拟递归的遍历过程。

   - 对于每个节点，计算其左子树和右子树的最大路径和，并更新当前节点的最大路径和。

   - 同时维护一个全局变量来记录遇到的最大路径和。

2. 栈的元素：栈中的元素包括当前节点及其相关的信息（如子树最大路径和）。

3. 遍历过程：

   - 先将根节点压入栈。

   - 然后从栈中弹出节点，处理其子节点并更新路径和。

##### 参考代码

```c++
#include <iostream>
#include <stack>
#include <algorithm>
#include <climits>

using namespace std;

// 二叉树节点定义
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
    int maxPathSum(TreeNode* root) {
        if (root == nullptr) return 0;

        int maxSum = INT_MIN;  // 初始化最大路径和为最小值

        // 使用栈来模拟递归
        stack<TreeNode*> nodeStack;
        TreeNode* curr = root;
        TreeNode* prev = nullptr;  // 记录前一个访问过的节点

        // 模拟递归的中序遍历
        while (curr != nullptr || !nodeStack.empty()) {
            // 向左遍历
            while (curr != nullptr) {
                nodeStack.push(curr);
                curr = curr->left;
            }

            // 访问栈顶元素
            curr = nodeStack.top();
            if (curr->right == nullptr || curr->right == prev) {
                // 处理当前节点
                nodeStack.pop();
                int leftGain = (curr->left == nullptr) ? 0 : max(0, maxPathSumHelper(curr->left));
                int rightGain = (curr->right == nullptr) ? 0 : max(0, maxPathSumHelper(curr->right));
                
                // 更新当前节点路径和
                int currentMaxPathSum = curr->val + leftGain + rightGain;
                maxSum = max(maxSum, currentMaxPathSum);
                
                // 更新全局最大路径和
                int returnGain = curr->val + max(leftGain, rightGain);
                maxSum = max(maxSum, returnGain);

                prev = curr;  // 更新前一个访问的节点
                curr = nullptr;  // 避免再次处理
            } else {
                // 右子树存在且未访问，转到右子树
                curr = curr->right;
            }
        }

        return maxSum;
    }

private:
    int maxPathSumHelper(TreeNode* root) {
        if (root == nullptr) return 0;
        int leftGain = max(0, maxPathSumHelper(root->left));
        int rightGain = max(0, maxPathSumHelper(root->right));
        return root->val + max(leftGain, rightGain);
    }
};

int main() {
    TreeNode* root = new TreeNode(-10);
    root->left = new TreeNode(9);
    root->right = new TreeNode(20);
    root->right->left = new TreeNode(15);
    root->right->right = new TreeNode(7);

    // 创建 Solution 对象并计算最大路径和
    Solution solution;
    cout << "最大路径和是: " << solution.maxPathSum(root) << endl;

    return 0;
}
```

