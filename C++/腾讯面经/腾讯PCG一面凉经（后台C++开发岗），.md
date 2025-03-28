# 腾讯PCG一面凉经（后台C++开发岗），

> 来源：https://www.nowcoder.com/feed/main/detail/ceb0a5dfaee0415191c09982ef4f2f27

## 1、TCP和UDP区别，HTTP用的谁，为什么？

#### TCP 和 UDP 的区别

##### 1. 连接性

- TCP：
  - 面向连接，建立连接需三次握手（SYN、SYN-ACK、ACK）。
  - 确保通信双方建立可靠通道。
- UDP：
  - 无连接，直接发送数据报，无需握手。
  - 不保证数据到达。

##### 2. 可靠性

- TCP：
  - 提供可靠性，通过序列号、确认（ACK）和重传机制确保数据不丢失、不重复且按序到达。
  - 有错误检测和纠正（如校验和）。
- UDP：
  - 无可靠性，数据可能丢失、乱序或重复。
  - 仅提供基本校验和，无重传。

##### 3. 数据传输方式

- TCP：
  - 字节流传输，无消息边界，数据按流处理。
  - 适合大块连续数据。
- UDP：
  - 数据报传输，保留消息边界，每个包独立。
  - 适合小块独立数据。

##### 4. 流量控制与拥塞控制

- TCP：
  - 有流量控制（滑动窗口）和拥塞控制（慢启动、拥塞避免）。
  - 防止网络过载。
- UDP：
  - 无流量或拥塞控制，发送速率由应用决定。
  - 可能导致网络拥堵。

##### 5. 开销与速度

- TCP：
  - 头部较大（20-60 字节），包含序列号、确认号等。
  - 速度较慢，因需确认和重传。
- UDP：
  - 头部小（8 字节），仅含源端口、目标端口、长度和校验和。
  - 速度快，无额外确认开销。

##### 6. 使用场景

- TCP：HTTP、FTP、SMTP（需要可靠传输）。
- UDP：DNS、DHCP、实时流媒体（如视频、VoIP）。

#### HTTP 使用的协议及其原因

##### 1. HTTP 1.x 和 HTTP 2 使用 TCP

- 版本：HTTP/1.0（1996）、HTTP/1.1（1997）、HTTP/2（2015）。
- 原因：
  1. 可靠性需求：Web 页面需要完整传输（HTML、CSS、图像等），TCP 的重传和顺序保证确保数据无误。
  2. 顺序性：HTTP 请求和响应需按序处理（如 HEAD、BODY），TCP 的字节流特性匹配此需求。
  3. 错误处理：TCP 的错误检测和纠正减少应用层负担。
  4. 历史因素：HTTP 诞生时（1990 年代），TCP 是成熟的可靠协议，UDP 未广泛用于类似场景。
- 流程：
  - 客户端发起 TCP 三次握手。
  - 发送 HTTP 请求（如 GET /index.html）。
  - 服务器响应，TCP 确保数据完整传输。

##### 2. HTTP/3 使用 UDP（基于 QUIC）

- 版本：HTTP/3（2022 年标准化）。
- 原因：
  1. 性能优化：QUIC（Quick UDP Internet Connections）基于 UDP，减少连接建立时间（0-RTT 握手对比 TCP 的 1-RTT）。
  2. 多路复用：QUIC 支持多流传输，避免 TCP 的队头阻塞（Head-of-Line Blocking）。
  3. 拥塞控制：QUIC 在应用层实现拥塞控制，比 TCP 更灵活。
  4. 安全性：QUIC 集成 TLS 1.3，提供加密，与 UDP 无关的可靠性由 QUIC 层处理。
- 流程：
  - 客户端发送 UDP 数据报，QUIC 建立连接。
  - HTTP/3 请求通过 QUIC 流传输。
  - QUIC 确保可靠性，UDP 提供底层速度。

##### 3. 为什么 HTTP 不用纯 UDP？

- 可靠性缺失：UDP 本身无法保证数据到达或顺序，HTTP 需要应用层重写大量逻辑。
- 复杂性增加：纯 UDP 需要手动实现重传、拥塞控制，与 TCP 的成熟性相比不划算。
- QUIC 的折中：HTTP/3 使用 UDP，但依赖 QUIC 提供 TCP 的可靠性特性。

## 2、HTTP状态码？

#### 1xx - 信息性状态码

- 含义：表示请求已接收，正在处理，属于临时响应。
- 用途：主要用于协议升级或通知客户端状态。
- 常见状态码：
  - 100 Continue：客户端可继续发送请求体（如大数据上传）。
  - 101 Switching Protocols：服务器同意切换协议（如升级到 WebSocket）。

#### 2xx - 成功状态码

- 含义：请求成功处理，服务器已完成操作。
- 用途：通知客户端请求已正确执行。
- 常见状态码：
  - 200 OK：请求成功，响应包含请求的数据。
  - 201 Created：资源创建成功（如 POST 创建新记录）。
  - 204 No Content：请求成功，但无响应体（如 DELETE）。

#### 3xx - 重定向状态码

- 含义：客户端需采取进一步操作，通常涉及重定向。
- 用途：管理资源位置变更或负载均衡。
- 常见状态码：
  - 301 Moved Permanently：资源永久移动，新 URL 在 Location 头中。
  - 302 Found：资源临时移动。
  - 304 Not Modified：资源未变更，客户端可使用缓存（与 If-Modified-Since 配合）。

#### 4xx - 客户端错误状态码

- 含义：请求有误，客户端需修正。
- 用途：提示客户端错误，如语法错误或权限不足。
- 常见状态码：
  - 400 Bad Request：请求语法错误或参数无效。
  - 401 Unauthorized：需要认证（如缺少 token）。
  - 403 Forbidden：服务器拒绝访问（如权限不足）。
  - 404 Not Found：资源未找到。
  - 429 Too Many Requests：请求超限（如 API 限流）。

#### 5xx - 服务器错误状态码

- 含义：服务器处理请求时出错。
- 用途：通知客户端服务器端问题。
- 常见状态码：
  - 500 Internal Server Error：服务器未知错误。
  - 502 Bad Gateway：网关或代理收到上游错误。
  - 503 Service Unavailable：服务器暂时不可用（如过载）。
  - 504 Gateway Timeout：网关超时。

## 3、HTTP是长连接还是短连接，如何用UDP实现？

#### 1. HTTP/1.0 - 短连接

- 默认每次请求建立新的 TCP 连接，三次握手后传输数据，四次挥手关闭。
- 响应头不含 Connection: keep-alive 时，连接关闭。

#### 2. HTTP/1.1 - 长连接

- 默认长连接，通过 Connection: keep-alive 保持 TCP 连接开放。
- 可通过 Connection: close 显式关闭。

#### 3. HTTP/2 - 长连接（多路复用）

- 默认长连接，使用单一 TCP 连接，通过多路复用处理多个请求/响应流。
- 二进制帧替代文本协议。

#### 4. HTTP/3 - UDP+QUIC（长连接特性）

- 使用 UDP，QUIC 在应用层实现长连接特性。
- QUIC 支持多路复用、0-RTT 连接和可靠性。

#### 如何用 UDP 实现 HTTP 长连接

##### UDP 的挑战

- 无连接性：UDP 不维护连接状态，每次数据报独立发送。
- 不可靠性：无重传、顺序或流量控制。
- HTTP 需求：长连接需要复用、可靠性和状态管理，UDP 本身无法直接满足。

##### QUIC 的解决方案

QUIC（Quick UDP Internet Connections）是基于 UDP 的协议，为 HTTP/3 提供长连接特性：

1. 连接建立：
   - QUIC 使用 UDP 数据报，通过首次握手交换连接 ID（Connection ID）标识通信双方。
   - 0-RTT（零往返时间）握手复用先前会话密钥，减少延迟。
2. 多路复用：
   - QUIC 在应用层定义流（Stream），每个流独立传输 HTTP 请求/响应。
   - 避免 TCP 的队头阻塞，丢包仅影响单个流。
3. 可靠性：
   - QUIC 实现序列号、确认（ACK）和重传机制，确保数据到达。
   - 校验和检测错误，丢包时重传。
4. 流量与拥塞控制：
   - QUIC 在应用层实现滑动窗口和拥塞控制（如 TCP 的 Cubic）。
   - 动态调整发送速率，防止网络过载。
5. 安全性：
   - QUIC 集成 TLS 1.3，所有数据加密。
   - 连接 ID 防止中间人攻击。 

#### UDP+QUIC 的长连接实现

- 逻辑连接：QUIC 通过 Connection ID 标识“连接”，在 UDP 上模拟持久状态。
- 复用：多个 HTTP 请求通过流复用同一 UDP“连接”。
- 关闭：QUIC 定义关闭帧（如 CONNECTION_CLOSE），优雅终止。

## 4、Linux查看进程指令，ps -aux第二列是什么，和ps -elf区别？

##### 基本用法

- 命令：ps [选项]
- 常用选项：
  - -a：显示所有终端的进程。
  - -u：显示用户相关信息。
  - -x：包括无终端进程。
  - -e：显示所有进程。
  - -l：长格式输出。
  - -f：完整格式输出。

##### ps -aux

- 含义：显示所有用户的所有进程（包括无终端进程）。
- 输出格式：BSD 风格，简洁易读。

##### ps -elf

- 含义：显示所有进程的详细信息。
- 输出格式：System V 风格，列数更多。

#### ps -aux 第二列是什么？

- 第二列：PID（Process ID，进程标识符）。
- 含义：
  - PID 是操作系统分配给每个进程的唯一整数，用于标识进程。
  - 范围：1（init 进程）到系统上限（通常 32768 或更高，受 /proc/sys/kernel/pid_max 限制）。
- 作用：
  - 用于进程管理，如 kill <PID> 终止进程。
  - 在 ps -aux 输出中，PID 是第二列，紧随 USER 之后。

#### ps -aux 与 ps -elf 的区别

##### 1. 输出风格

- ps -aux：BSD 风格，选项不带 -，更简洁，用户友好。
- ps -elf：System V 风格，选项带 -，信息更详细，偏技术。

##### 2. 列数与含义

- ps -aux：（11 列）
  1. USER：进程所属用户。
  2. PID：进程 ID。
  3. %CPU：CPU 使用率。
  4. %MEM：内存使用率。
  5. VSZ：虚拟内存大小（KB）。
  6. RSS：物理内存大小（KB）。
  7. TTY：控制终端。
  8. STAT：进程状态（如 S=睡眠，R=运行）。
  9. START：启动时间。
  10. TIME：累计 CPU 时间。
  11. COMMAND：命令行。
- ps -elf：（15列）
  1. F：进程标志（如 4=超级用户）。
  2. S：状态（如 S=睡眠）。
  3. UID：用户 ID。
  4. PID：进程 ID。
  5. PPID：父进程 ID。
  6. C：CPU 使用百分比。
  7. PRI：优先级。
  8. NI：Nice 值（优先级调整）。
  9. ADDR：内存地址（通常 -）。
  10. SZ：内存大小（页面数）。
  11. WCHAN：等待事件。
  12. STIME：启动时间。
  13. TTY：终端。
  14. TIME：累计 CPU 时间。
  15. CMD：命令。

##### 3. 信息侧重

- ps -aux：
  - 更关注用户视角（如 %CPU、%MEM）。
  - 适合快速查看运行进程。
- ps -elf：
  - 提供技术细节（如 PPID、PRI、NI）。
  - 适合调试或分析进程关系。

## 5、MySQL索引为什么用B+不用B树？如果把内容存在内存上，用B树会不会快一点？

#### B 树与 B+ 树的区别

##### B 树（B-Tree）

- 结构：
  - 每个节点包含键值（key）和数据（value）。
  - 键值按序排列，子节点指针夹在键值之间。
  - 所有节点（包括中间节点和叶子节点）都存数据。
- 性质：
  - 阶数（order）m：每个节点最多 m 个子节点，最多 m-1 个键。
  - 平衡：所有叶子节点在同一层。
- 示例（m=3）：

```tex
     [5, 10]
    /  |  \
[1, 3] [6, 8] [12, 15]
[数据]  [数据]  [数据]
```

##### B+ 树（B+-Tree）

- 结构：
  - 只有叶子节点存数据，中间节点仅存键值和指针。
  - 叶子节点形成链表，支持顺序访问。
  - 键值可能重复出现在中间节点和叶子节点。
- 性质：
  - 阶数 m：非叶子节点最多 m 个子节点，叶子节点存更多键值。
  - 所有数据在叶子层，中间节点仅用于导航。
- 示例（m=3）：

```tex
     [5, 10]
    /  |  \
[1, 3] [5, 8] [10, 15]
[1,3]→[5,8]→[10,15]
[数据] [数据] [数据]
```

#### MySQL 索引为何用 B+ 树

##### 1. 磁盘 I/O 效率

- B+ 树优势：
  - 中间节点不存数据，仅存键值和指针，扇出（fanout）更高（一个页面存更多指针）。
  - 树高更低（高度 h ∝ log_m(n)，m 越大 h 越小）。
  - 每次 I/O 读取一个页面（通常 4KB 或 16KB），B+ 树能加载更多导航信息。
- B 树劣势：
  - 每个节点存键和数据，数据占用空间减少扇出。
  - 树高较高，需更多 I/O。
- 实例：
  - 假设页面大小 4KB，键 8 字节，数据 100 字节：
    - B 树：一个节点约存 40 个键（4KB / (8+100)）。
    - B+ 树：一个节点约存 500 个键（4KB / 8）。
  - B+ 树树高更低，I/O 少。

##### 2. 范围查询性能

- B+ 树优势：
  - 叶子节点链表结构，支持顺序遍历。
  - 范围查询（如 WHERE id BETWEEN 5 AND 10）只需定位起点，沿链表读取。
- B 树劣势：无链表，范围查询需回溯或多次搜索，效率低。
- 实例：
  - B+ 树：找到 5，沿链表读到 10。
  - B 树：需遍历树结构，可能多次 I/O。

##### 3. 数据存储与一致性

- B+ 树优势：
  - 数据集中在叶子节点，便于顺序存储和缓存。
  - 中间节点仅导航，更新时影响小。
- B 树劣势：数据分散在所有节点，插入/删除可能引发更多调整。
- MySQL 实践：InnoDB 主键索引（聚集索引）将数据存于叶子节点，B+ 树天然契合。

##### 4. 空间利用率

- B+ 树：叶子节点存满数据，中间节点高效导航。
- B 树：每个节点存数据，空间浪费于中间节点。

#### 如果内容全在内存，B 树会不会更快？

##### 内存场景分析

- 磁盘 vs 内存：
  - 磁盘：I/O 是瓶颈，B+ 树减少 I/O 次数。
  - 内存：I/O 成本消失，访问时间接近均匀，树高影响变小。
- B 树潜在优势：
  - 结构简单：键和数据同节点，查找时无需跳转到叶子。
  - 随机访问：内存中单次查找更快（少一层指针）。
  - 树高：B 树稍高，但内存访问延迟低，影响小。
- B+ 树优势仍存：
  - 范围查询：链表结构依然高效。
  - 扇出高：树高更低，减少比较次数。
  - 缓存友好：顺序访问利于 CPU 缓存。

##### 性能对比

- 随机查询：
  - B 树：直接找到数据，少一次跳转。
  - B+ 树：需到叶子，稍慢。
  - 差异：微秒级，内存中几乎可忽略。
- 范围查询：
  - B+ 树：顺序遍历，缓存命中率高。
  - B 树：需回溯，效率低。
- 插入/删除：
  - B 树：调整可能更频繁（数据分散）。
  - B+ 树：叶子集中调整，内存中差异小。

##### 定量分析

- 假设 n=10^6（百万条记录），页面大小 4KB，键 8 字节，数据 100 字节：
  - B 树：扇出 ≈ 40，高度 ≈ log_40(10^6) ≈ 4。
  - B+ 树：扇出 ≈ 500，高度 ≈ log_500(10^6) ≈ 3。
- 内存中：
  - B 树：4 次比较+访问。
  - B+ 树：3 次比较+1 次跳转。

**结论**：B 树可能快微小幅度，但 B+ 树范围查询优势显著。

## 6、struct和Class区别？

#### 1. 默认访问权限

- struct：

  - 默认访问：public。
  - 成员（变量、函数）和继承默认公开。

- class：

  - 默认访问：private。
  - 成员和继承默认私有。

- 示例：

  ```c++
  struct S {
      int x;  // 默认 public
  };
  class C {
      int x;  // 默认 private
  };
  S s; s.x = 5;    // OK
  C c; c.x = 5;    // 错误：x 是 private
  ```

#### 2. 功能等价性

共同特性：

- 支持成员变量、成员函数、构造函数、析构函数。
- 支持继承、多态、虚函数。
- 可定义静态成员、模板。

#### 3. 语义约定

- struct：
  - 习惯用于简单数据聚合，如 C 风格结构体。
  - 常表示 POD（Plain Old Data）类型。
  - 示例：坐标、配置项。
- class：
  - 习惯用于封装数据和行为，强调对象特性。
  - 常用于复杂逻辑和抽象。
  - 示例：类层次（如动物、车辆）。
- 示例：

```c++
struct Point {
    int x, y;  // 公开数据
};
class Animal {
private:
    int age;
public:
    void speak();  // 封装行为
};
```

#### 4. 继承时的访问权限

- 默认继承：
  - struct：public 继承。
  - class：private 继承。
- 示例：

```c++
struct Base { int x; };
struct Derived : Base { };  // public 继承
class DerivedC : Base { };  // private 继承
Derived d; d.x = 5;         // OK
DerivedC dc; dc.x = 5;      // 错误
```

#### 5. 与 C 的对比

- C 中的 struct：
  - 仅数据聚合，无成员函数、无继承。
  - 使用需加 struct 关键字（如 struct Point p;）。
- C++ 中的 struct：
  - 功能扩展，与 class 等价。
  - 无需 struct 关键字。

## 7、简单介绍一下STL有哪些容器？

#### 1. 序列容器

序列容器按元素插入顺序存储，支持线性访问。

- std::vector：
  - 特性：动态数组，连续内存，尾部插入/删除快。
  - 时间复杂度：尾部 O(1)，中间插入/删除 O(n)。
  - 头文件：<vector>
  - 用途：需要快速随机访问和动态大小的场景。
- std::deque（双端队列）：
  - 特性：连续内存块，支持首尾快速插入/删除。
  - 时间复杂度：首尾 O(1)，中间 O(n)。
  - 头文件：<deque>
  - 用途：需要首尾操作的动态序列。
- std::list：
  - 特性：双向链表，非连续内存，支持双向遍历。
  - 时间复杂度：插入/删除 O(1)，访问 O(n)。
  - 头文件：<list>
  - 用途：频繁插入/删除，不需随机访问。
- std::array（C++11）：
  - 特性：固定大小数组，连续内存，栈分配。
  - 时间复杂度：访问 O(1)，无动态操作。
  - 头文件：<array>
  - 用途：固定长度、性能敏感场景。
- std::forward_list（C++11）：
  - 特性：单向链表，非连续内存，仅前向遍历。
  - 时间复杂度：插入/删除 O(1)，访问 O(n)。
  - 头文件：<forward_list>
  - 用途：节省内存的链表场景。

#### 2. 关联容器

关联容器基于键排序存储，支持快速查找，通常用红黑树实现。

- std::set：
  - 特性：有序集合，键唯一，自动排序。
  - 时间复杂度：查找/插入/删除 O(log n)。
  - 头文件：<set>
  - 用途：需要唯一有序元素的场景。
- std::multiset：
  - 特性：有序集合，允许重复键。
  - 时间复杂度：O(log n)。
  - 头文件：<set>
  - 用途：有序可重复元素。
- std::map：
  - 特性：有序键值对，键唯一，自动按键排序。
  - 时间复杂度：O(log n)。
  - 头文件：<map>
  - 用途：键值映射。
- std::multimap：
  - 特性：有序键值对，允许重复键。
  - 时间复杂度：O(log n)。
  - 头文件：<map>
  - 用途：多值映射。

#### 3. 无序关联容器（C++11）

无序容器基于哈希表实现，键无序，查找更快。

- std::unordered_set：
  - 特性：无序集合，键唯一，哈希存储。
  - 时间复杂度：平均 O(1)，最坏 O(n)。
  - 头文件：<unordered_set>
  - 用途：快速查找唯一元素。
- std::unordered_multiset：
  - 特性：无序集合，允许重复键。
  - 时间复杂度：平均 O(1)。
  - 头文件：<unordered_set>
  - 用途：无序可重复元素。
- std::unordered_map：
  - 特性：无序键值对，键唯一。
  - 时间复杂度：平均 O(1)。
  - 头文件：<unordered_map>
  - 用途：快速键值查找。
- std::unordered_multimap：
  - 特性：无序键值对，允许重复键。
  - 时间复杂度：平均 O(1)。
  - 头文件：<unordered_map>
  - 用途：无序多值映射。

#### 4. 容器适配器

容器适配器基于其他容器，提供受限接口。

- std::stack：
  - 特性：后进先出（LIFO），默认基于 deque。
  - 头文件：<stack>
  - 用途：栈操作。
- std::queue：
  - 特性：先进先出（FIFO），默认基于 deque。
  - 头文件：<queue>
  - 用途：队列操作。
- std::priority_queue：
  - 特性：优先级队列，默认基于 vector，大顶堆。
  - 头文件：<queue>
  - 用途：优先级排序。

#### 数据与对比

| 类别         | 容器                   | 底层实现 | 时间复杂度（查找/插入） | 用途              |
| ------------ | ---------------------- | -------- | ----------------------- | ----------------- |
| 序列容器     | vector                 | 动态数组 | O(1)/O(1)尾部           | 随机访问          |
|              | deque                  | 双端数组 | O(1)/O(1)首尾           | 首尾操作          |
|              | list                   | 双向链表 | O(n)/O(1)               | 频繁插入          |
|              | array                  | 固定数组 | O(1)/不可变             | 固定大小          |
|              | forward_list           | 单向链表 | O(n)/O(1)               | 节省内存          |
| 关联容器     | set/multiset           | 红黑树   | O(log n)                | 有序唯一/重复元素 |
|              | map/multimap           | 红黑树   | O(log n)                | 有序键值映射      |
| 无序关联容器 | unordered_set/multiset | 哈希表   | O(1)平均                | 无序快速查找      |
|              | unordered_map/multimap | 哈希表   | O(1)平均                | 无序键值映射      |
| 容器适配器   | stack                  | deque    | O(1)                    | 栈                |
|              | queue                  | deque    | O(1)                    | 队列              |
|              | priority_queue         | vector   | O(log n)                | 优先级队列        |

## 8、map是线程安全的么？

在 C++ 标准库中，`std::map` 并不是线程安全的。

#### 1. 读与写的区分

- 只读操作：
  如果多个线程仅进行只读操作（即没有线程在修改容器），那么并发访问 `std::map` 是相对安全的，因为只读操作不会修改内部数据结构。但需要保证容器在这些操作期间不会被其他线程修改，否则就会出现数据竞争。
- 读写混合操作：
  如果有线程在修改 `std::map`（比如插入、删除、更新操作），而其他线程同时进行读取或写入操作，就可能会产生竞争条件，导致未定义行为。C++ 标准库的容器在设计时并没有内置同步机制，因此需要外部加锁来确保线程安全。

#### 2. 为什么 `std::map` 不是线程安全的

- 内部数据结构：
  `std::map` 通常使用红黑树等平衡二叉搜索树作为底层数据结构，其插入和删除操作可能会引起树的重新平衡和节点指针的修改。如果在操作过程中没有适当的同步机制，不同线程可能同时修改这些结构，导致数据结构损坏或程序崩溃。
- 迭代器失效：
  当 `std::map` 发生修改（例如插入或删除操作）时，相关的迭代器可能会失效。如果多个线程同时访问这些迭代器而没有同步，可能会导致无法预知的行为。

#### 3. 如何确保线程安全

- 使用互斥锁（Mutex）：
  在多线程环境中访问 `std::map` 时，可以使用互斥锁（如 `std::mutex`）来保护所有对 `std::map` 的访问（读和写）。

```c++
#include <map>
#include <mutex>

std::map<int, int> myMap;
std::mutex mapMutex;

// 写操作示例
void insertValue(int key, int value) {
    std::lock_guard<std::mutex> lock(mapMutex);
    myMap[key] = value;
}

// 读操作示例
int getValue(int key) {
    std::lock_guard<std::mutex> lock(mapMutex);
    auto it = myMap.find(key);
    if (it != myMap.end()) {
        return it->second;
    }
    return -1; // 或其他默认值
}
```

- 读写锁：

  如果读操作远多于写操作，可以考虑使用读写锁（如 `std::shared_mutex` 或平台特定的实现）来允许多个线程并发读，但写操作时仍需要独占锁。

```c++
#include <map>
#include <shared_mutex>

std::map<int, int> myMap;
std::shared_mutex rwMutex;

// 写操作
void insertValue(int key, int value) {
    std::unique_lock<std::shared_mutex> lock(rwMutex);
    myMap[key] = value;
}

// 读操作
int getValue(int key) {
    std::shared_lock<std::shared_mutex> lock(rwMutex);
    auto it = myMap.find(key);
    if (it != myMap.end()) {
        return it->second;
    }
    return -1;
}
```

## 9、队列和栈有什么区别？

#### 1. 存取顺序

- 栈（Stack）：栈遵循“后进先出”（LIFO, Last In First Out）的原则。
  - 操作特点：
    - 入栈（push）： 数据从栈顶插入。
    - 出栈（pop）： 数据也从栈顶移除，最后加入的元素最先被取出。
  - 应用场景：
    - 函数调用栈
    - 表达式求值与语法解析
    - 回溯算法（如深度优先搜索中的路径记录）
- 队列（Queue）：队列遵循“先进先出”（FIFO, First In First Out）的原则。
  - 操作特点：
    - 入队（enqueue）： 数据从队尾插入。
    - 出队（dequeue）： 数据从队首移除，最先加入的元素最先被取出。
  - 应用场景：
    - 任务调度和进程管理
    - 广度优先搜索（BFS）
    - 消息队列和事件驱动系统

#### 2. 内部结构和实现

- 栈：
  - 通常使用数组或链表实现。
  - 实现简单，仅需要维护一个指针（或索引）指向栈顶即可完成所有操作。
- 队列：
  - 可以使用数组、链表或者循环数组（环形缓冲区）实现。
  - 循环队列可以更有效地利用存储空间，避免因不断出队入队导致的数组移动。

#### 3. 对比表

| 操作     | 栈（Stack）                    | 队列（Queue）                        |
| -------- | ------------------------------ | ------------------------------------ |
| 插入数据 | 入栈（push）：在栈顶插入       | 入队（enqueue）：在队尾插入          |
| 删除数据 | 出栈（pop）：从栈顶删除        | 出队（dequeue）：从队首删除          |
| 访问顺序 | 后进先出（LIFO）               | 先进先出（FIFO）                     |
| 应用示例 | 函数调用、表达式求值、回溯算法 | 任务调度、广度优先搜索、消息队列系统 |

#### 4. 使用场景和优劣

- 栈的优点与使用场景：
  - 优势： 实现简单，适用于只需要临时保存数据的场景，如递归调用、深度优先搜索等。
  - 局限： 只允许在一端插入和删除，无法直接访问中间的元素。
- 队列的优点与使用场景：
  - 优势： 可以按顺序处理数据，适用于需要按先后顺序处理任务的场景，如任务调度和事件处理。
  - 局限： 只能在两端进行插入和删除操作，不适合随机访问。

## 10、手撕算法，大数求和（字符串中等难度）

#### 思路

将两个表示大数的字符串从尾部（即最低位）开始逐位相加，模拟传统的手工加法。

- 每一位相加时，将两个数字和之前的进位相加。
- 将结果除以 10 得到新的进位，用取余得到当前位的数字。
- 因为从尾部开始，所以最终得到的结果是逆序的，最后需要反转字符串。

#### 参考代码（C++）

```c++
#include <iostream>
#include <string>
#include <algorithm>
using namespace std;

// 实现大数求和的函数
string addBigNumbers(const string &num1, const string &num2) {
    int i = num1.size() - 1;  // 从num1的末尾开始
    int j = num2.size() - 1;  // 从num2的末尾开始
    int carry = 0;            // 进位
    string result;            // 存储计算结果

    // 当两个字符串还没遍历完或者还有进位时，继续循环
    while (i >= 0 || j >= 0 || carry) {
        int digit1 = (i >= 0) ? num1[i] - '0' : 0;  // 获取num1的当前位数字
        int digit2 = (j >= 0) ? num2[j] - '0' : 0;  // 获取num2的当前位数字

        int sum = digit1 + digit2 + carry;  // 当前位求和，加上上次的进位
        carry = sum / 10;                   // 更新进位
        int currentDigit = sum % 10;        // 当前位的结果

        result.push_back(currentDigit + '0'); // 将当前位数字转换为字符并加入结果
        i--;
        j--;
    }

    // 由于是从最低位开始加入结果，所以结果字符串是逆序的
    reverse(result.begin(), result.end());
    return result;
}

int main() {
    string a, b;
    cout << "请输入第一个大数：";
    cin >> a;
    cout << "请输入第二个大数：";
    cin >> b;
    
    string sum = addBigNumbers(a, b);
    cout << "两数之和为：" << sum << endl;
    return 0;
}
```

