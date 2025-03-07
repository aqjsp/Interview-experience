# 腾讯会议一面凉经

> 来源：https://www.nowcoder.com/feed/main/detail/80215048f6f24cc6b24ef1dbd23fad31

## 1、TCP 和 UDP 的区别是什么？

#### 定义

##### TCP

是一种面向连接的、可靠的传输协议。确保数据按顺序、无丢失、无重复地传输。

##### UDP

是一种无连接的、不可靠的传输协议。提供尽力而为的交付，不保证数据可靠性。

####  数据传输方式

##### TCP

- 字节流传输，无消息边界。
- 发送端可能将数据分段，接收端需要重组。
- 比如，发送 "HelloWorld"，接收端可能分两次收到 "Hello" 和 "World"。

##### UDP

- 数据报传输，每个包独立，保留边界。
- 发送端发一个包，接收端收到一个完整包。
- 比如，发送 "Hello"，接收端收到完整 "Hello" 或丢包。

#### 头部结构

##### TCP 头部

（20 字节基础，选项可扩展至 60 字节）：

- 源端口、目标端口（各 2 字节）。
- 序列号（4 字节）、确认号（4 字节）。
- 窗口大小（2 字节）、校验和（2 字节）等。

##### UDP 头部

（固定 8 字节）：

- 源端口、目标端口（各 2 字节）。
- 长度（2 字节）、校验和（2 字节）。

#### 区别

| 特性         | TCP                                      | UDP                          |
| ------------ | ---------------------------------------- | ---------------------------- |
| 连接性       | 面向连接（需三次握手建立连接）           | 无连接（直接发送数据）       |
| 可靠性       | 可靠（保证数据无丢失、无重复、按序到达） | 不可靠（不保证送达或顺序）   |
| 数据传输方式 | 流式（字节流，无边界）                   | 数据报（独立数据包，有边界） |
| 流量控制     | 有（通过滑动窗口机制）                   | 无                           |
| 拥塞控制     | 有（动态调整发送速率）                   | 无                           |
| 头部开销     | 较大（头部 20-60 字节）                  | 较小（头部 8 字节）          |
| 传输速度     | 较慢（因可靠性机制增加开销）             | 较快（无额外控制）           |
| 双工性       | 全双工（双向通信）                       | 单向（发送和接收独立）       |
| 应用场景     | 文件传输、网页浏览、邮件                 | 视频流、语音通话、DNS 查询   |

## 2、如何让 UDP 变得比 TCP 更可靠？

要在UDP上实现可靠性，需要在应用层添加一些机制。

#### 序列号

作用：为每个数据包分配唯一标识，跟踪发送顺序并检测丢失或重复。

实现：在数据包中附加序列号字段，接收端根据序列号重组数据。

#### 确认机制

作用：接收端通知发送端已成功接收某些数据包。

实现：

- 逐包确认：每个包都发送 ACK（类似 TCP）。
- 累积确认：确认连续接收的最大序列号。
- 选择性确认（SACK）：确认非连续的包范围，提升效率。

#### 重传机制

- 作用：检测丢失包并重新发送。
- 实现：
  - 超时重传：发送端设置定时器，未收到 ACK 则重传。
  - 快速重传：接收端报告丢失，触发即时重传。

####  数据包排序

- 作用：确保接收端按顺序处理数据。
- 实现：接收端维护缓冲区，乱序包暂存，待顺序完整再交付。

#### 流量控制

- 作用：避免发送端压垮接收端。
- 实现：接收端通告窗口大小（如 TCP 的滑动窗口）。

#### 方法一

简单可靠 UDP（逐包确认 + 重传）

##### 设计

- 数据包格式：[序列号 | 数据]。
- 发送端：发送数据包并等待 ACK，超时重传。
- 接收端：收到包后发送 ACK，丢包不处理。

##### 参考代码

```c++
#include <iostream>
#include <string>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <cstring>

#define PORT 12345
#define TIMEOUT 2 // 超时时间（秒）

struct Packet {
    uint32_t seq; // 序列号
    char data[1024]; // 数据
};

// 发送端
void sendReliableUDP(int sock, struct sockaddr_in& server_addr, const std::string& message) {
    Packet packet;
    packet.seq = 1;
    strncpy(packet.data, message.c_str(), sizeof(packet.data));

    while (true) {
        sendto(sock, &packet, sizeof(packet), 0, (struct sockaddr*)&server_addr, sizeof(server_addr));
        std::cout << "Sent packet with seq: " << packet.seq << std::endl;

        // 设置超时
        struct timeval tv;
        tv.tv_sec = TIMEOUT;
        tv.tv_usec = 0;
        setsockopt(sock, SOL_SOCKET, SO_RCVTIMEO, &tv, sizeof(tv));

        // 等待 ACK
        char ack[10];
        socklen_t addr_len = sizeof(server_addr);
        if (recvfrom(sock, ack, sizeof(ack), 0, (struct sockaddr*)&server_addr, &addr_len) > 0) {
            if (strcmp(ack, "ACK1") == 0) {
                std::cout << "Received ACK for seq: " << packet.seq << std::endl;
                break;
            }
        } else {
            std::cout << "Timeout, retransmitting..." << std::endl;
        }
    }
}

// 接收端
void receiveReliableUDP(int sock, struct sockaddr_in& client_addr) {
    Packet packet;
    socklen_t addr_len = sizeof(client_addr);

    while (true) {
        if (recvfrom(sock, &packet, sizeof(packet), 0, (struct sockaddr*)&client_addr, &addr_len) > 0) {
            std::cout << "Received packet with seq: " << packet.seq << " data: " << packet.data << std::endl;
            // 发送 ACK
            const char* ack = "ACK1";
            sendto(sock, ack, strlen(ack) + 1, 0, (struct sockaddr*)&client_addr, addr_len);
            break;
        }
    }
}

int main() {
    int sock = socket(AF_INET, SOCK_DGRAM, 0);
    struct sockaddr_in server_addr, client_addr;

    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = INADDR_ANY;
    server_addr.sin_port = htons(PORT);

    bind(sock, (struct sockaddr*)&server_addr, sizeof(server_addr));

    // 接收端（另开线程或进程模拟）
    receiveReliableUDP(sock, client_addr);

    // 发送端
    sendReliableUDP(sock, server_addr, "Hello, Reliable UDP!");

    close(sock);
    return 0;
}
```

局限：逐包确认效率低，类似 TCP 的 Stop-and-Wait 协议。

#### 方法二

高级可靠 UDP（滑动窗口 + 选择性确认）

##### 设计

- 数据包格式：[序列号 | 数据 | 校验和]。
- 发送端：维护发送窗口，批量发送，超时或收到 SACK 后重传。
- 接收端：维护接收缓冲区，返回累积 ACK 和 SACK。

##### 伪代码

```c++
struct Packet {
    uint32_t seq;
    char data[1024];
    uint16_t checksum;
};

class ReliableUDP {
    vector<Packet> sendWindow; // 发送窗口
    vector<Packet> recvBuffer; // 接收缓冲区
    uint32_t nextSeq;         // 下一个发送序列号
    uint32_t lastAck;         // 最后确认的序列号

    void sendPacket(Packet& p) {
        // 计算校验和，发送 UDP 数据包
        sendto(sock, &p, sizeof(p), ...);
        sendWindow.push_back(p);
        startTimer(p.seq);
    }

    void onReceive(Packet& p) {
        if (verifyChecksum(p)) {
            recvBuffer[p.seq] = p;
            sendAck(lastAck, getMissingSeqs()); // 发送累积 ACK 和 SACK
        }
    }

    void onAck(uint32_t ack, vector<uint32_t> sacks) {
        // 移除已确认的包
        removeFromWindow(ack);
        // 重传丢失的包
        for (auto seq : sacks) resend(seq);
    }

    void onTimeout(uint32_t seq) {
        resend(seq);
    }
};
```

## 3、TCP 目前使用的拥塞控制算法吗？慢启动不是从头开始的?

TCP的拥塞控制是确保网络稳定和高效的关键机制，旨在防止发送端以过高的速率发送数据，导致网络拥堵或数据丢失。当前 TCP 使用的拥塞控制算法并非单一固定的，而是根据实现（如操作系统内核、TCP 变种）和网络条件动态选择。

慢启动（Slow Start）是其中一个重要阶段，但它并不是每次都“从头开始”。

#### 拥塞控制的基本原理

TCP 拥塞控制通过动态调整拥塞窗口（ cwnd）来控制发送速率。拥塞窗口与接收窗口（ rwnd）一起决定实际发送窗口（min(cwnd, rwnd)）。

##### 核心机制

- 慢启动：探测网络容量，指数增长 cwnd。
- 拥塞避免：线性增长 cwnd，避免过载。
- 快速重传：检测丢包后立即重传。
- 快速恢复：丢包后快速调整 cwnd，避免回到慢启动。

#### 慢启动的基本原理

- 目标：快速探测网络可用带宽。
- 过程：
  - 初始化：cwnd = 1 MSS（最大报文段大小，通常 1460 字节）。
  - 每次收到 ACK，cwnd += 1 MSS（指数增长）。
  - 直到达到慢启动阈值（ssthresh）或检测到拥塞。
- 终止条件：
  - cwnd ≥ ssthresh：进入拥塞避免。
  - 丢包（超时或 3 个重复 ACK）：调整 cwnd。

#### 慢启动是否从头开始？

- 初次连接：从 cwnd = 1 开始，指数增长。

  cwnd = 1 → 2 → 4 → 8 → 16（每个 RTT 翻倍）。

- 丢包后：

  超时（RTO，Retransmission Timeout）：

  - cwnd 置为 1，ssthresh 设为当前 cwnd / 2。
  - 从头开始慢启动。
  - 示例：cwnd = 16，超时后 cwnd = 1，ssthresh = 8。

- 快速重传（3 个重复 ACK）：

  - 不从头开始！
  - cwnd 减半（Cubic 为 0.7 倍），ssthresh 设为新 cwnd，进入快速恢复。
  - 示例：cwnd = 16，丢包后 cwnd = 8，ssthresh = 8。

- 连接复用（如 HTTP Keep-Alive）：不从头开始，沿用之前的 cwnd 和 ssthresh。

## 4、TCP 如何进行重传？如何确定何时重传？要等待多久重传？

#### TCP 如何进行重传

##### 重传机制

序列号和确认（ACK）：

- 每个数据段分配一个序列号。
- 接收端发送 ACK 确认已接收的连续序列号。

检测丢失：

- 超时：发送端未在规定时间内收到 ACK。
- 快速重传：接收端发送 3 个重复 ACK（Duplicate ACK），表明某个段丢失。

重传数据：

- 超时：重传未确认的最早段。
- 快速重传：立即重传丢失的段。

##### 工作流程

- 发送端：发送数据段（如 Seq=1000, Len=500）。
- 接收端：收到后发送 ACK=1500（期待下一个字节）。
- 若丢包（如 Seq=1500 未到达）：
  - 接收端重复发送 ACK=1000。
  - 发送端收到 3 个重复 ACK 或超时后，重传 Seq=1500。

#### 如何确定何时重传

##### 超时重传

- 条件：发送端在一定时间（RTO，Retransmission Timeout）内未收到 ACK。
- 动作：重传未确认的最早数据段，拥塞窗口（cwnd）置为 1，进入慢启动。

##### 快速重传

- 条件：接收端连续发送 3 个重复 ACK。
- 动作：
  - 发送端立即重传丢失段。
  - cwnd 减半，进入快速恢复（Fast Recovery）。

#### 要等待多久重传（RTO 计算）

TCP 动态计算 RTO，基于网络的往返时间（RTT）

RTT 测量：

- 发送端记录数据包发送时间和对应 ACK 接收时间。
- RTT = ACK 接收时间 - 发送时间。

平滑 RTT（SRTT）：

- 使用指数加权移动平均（EWMA）：

  ```
  SRTT = (1 - α) × SRTT + α × RTT
  ```

- α 通常为 0.125（RFC 6298）。

RTT 偏差（RTTVAR）：

- 测量 RTT 的波动：

  ```
  RTTVAR = (1 - β) × RTTVAR + β × |SRTT - RTT|
  ```

- β 通常为 0.25。

RTO 计算：

```
RTO = SRTT + 4 × RTTVAR
```

- 初始值：3 秒（若无 RTT 数据）。
- 上下限：1 秒 ≤ RTO ≤ 60 秒（RFC 6298 建议）。

## 5、你了解 HTTP/2 吗？为什么后端开发要使用 HTTP/2？它的性能优势是什么？

#### HTTP/2 简介

##### 背景

HTTP/1.1（1997 年发布）在当时设计简单，但随着 Web 应用复杂性增加（多资源加载、高并发），其局限性暴露：

- 队头阻塞（Head-of-Line Blocking）：请求按序处理，慢请求阻塞后续请求。
- 多连接开销：浏览器需为每个域名建立多个 TCP 连接。
- 头部冗余：每次请求重复发送大量头部。

##### HTTP/2 的目标

- 提高页面加载速度。
- 减少网络延迟。
- 优化资源利用率。
- 保持与 HTTP/1.1 的语义兼容（方法、状态码、URI 等不变）。

#### HTTP/2 的工作原理

##### (1) 二进制分帧

- 原理：
  - HTTP/2 将请求和响应分解为小的二进制帧（如 HEADERS 帧、DATA 帧）。
  - 每个帧包含流 ID（Stream ID），标识所属的请求/响应。
- 对比 HTTP/1.1：
  - HTTP/1.1 使用文本协议，解析复杂且易出错。
  - 二进制格式更紧凑，解析效率高，减少错误。

##### (2) 多路复用

- 原理：
  - 在一个 TCP 连接上并行发送多个请求和响应。
  - 每个请求/响应是一个“流”（Stream），互不干扰。
- 对比 HTTP/1.1：
  - HTTP/1.1 每个请求需等待前一个完成（队头阻塞）。
  - HTTP/2 允许同时传输多个流，充分利用带宽。

#### (3) 头部压缩

- 原理：
  - 使用 HPACK 算法压缩 HTTP 头部，维护动态表记录常见字段。
  - 首次传输完整头部，后续只发送差异或索引。
- 对比 HTTP/1.1：
  - HTTP/1.1 每次请求重复发送冗余头部（如 User-Agent、Accept）。
  - HTTP/2 压缩后减少带宽占用。

#### (4) 服务器推送

- 原理：
  - 服务器预测客户端需求，主动推送资源（如 CSS、JS）。
  - 使用 PUSH_PROMISE 帧通知客户端。
- 对比 HTTP/1.1：
  - HTTP/1.1 需客户端逐个请求资源。
  - HTTP/2 提前推送，减少往返时间（RTT）。

#### (5) 优先级和流量控制

- 原理：
  - 客户端为每个流设置优先级，服务器按需分配资源。
  - 流级别流量控制，避免接收端过载。
- 对比 HTTP/1.1：HTTP/1.1 无优先级机制，资源加载顺序不可控。

#### 为什么后端开发要使用 HTTP/2？

##### (1) 提升用户体验

- 更快加载：多路复用和服务器推送减少页面加载时间。
- 实时性：低延迟适合实时应用（如聊天、通知）。

##### (2) 优化服务器性能

- 单一连接：减少 TCP 连接数，降低服务器资源占用（如文件描述符）。
- 头部压缩：减少带宽消耗，适合高并发场景。

##### (3) 现代 Web 需求

- 复杂页面：现代网站包含数十甚至数百个资源（图片、脚本、样式），HTTP/1.1 的多连接效率低。
- 移动网络：HTTP/2 在高延迟、不稳定网络中表现更好。

##### (4) 与前端生态兼容

- 浏览器支持：主流浏览器（Chrome、Firefox、Safari）已全面支持 HTTP/2。
- 协议演进：HTTP/2 是 HTTP/3（基于 UDP 的 QUIC）的过渡，保持竞争力。

##### (5) 安全性要求

- 强制 HTTPS：HTTP/2 通常与 TLS 搭配（浏览器要求），提升安全性。
- 后端适配：支持 TLS 的后端框架（如 Nginx、Spring Boot）易于部署 HTTP/2。

#### HTTP/2 的性能优势

##### (1) 消除队头阻塞

- HTTP/1.1：
  - 请求按序处理，若首个请求延迟（如 500ms），后续请求被阻塞。
  - 需多个 TCP 连接（通常 6 个/域名），增加握手和拥塞控制开销。
- HTTP/2：多路复用允许并行传输，慢请求不影响其他请求。

##### (2) 减少连接开销

- HTTP/1.1：
  - 每个连接需三次握手（1 RTT）和 TLS 协商（1-2 RTT）。
  - 浏览器限制域名连接数（6-8 个），资源多时需排队。
- HTTP/2：单连接复用，握手只需一次。

##### (3) 降低带宽占用

- HTTP/1.1：头部冗余，例如每次请求发送 500 字节头部，100 个请求浪费 50KB。
- HTTP/2：HPACK 压缩头部至几十字节，节省带宽。

##### (4) 提前推送资源

- HTTP/1.1：客户端发现 HTML 中的 <script> 后才请求，增加 RTT。
- HTTP/2：服务器推送 /index.js，客户端收到 HTML 时脚本已就绪。

##### (5) 更好的网络适应性

- HTTP/1.1：在高丢包网络中，多连接加剧拥塞。
- HTTP/2：单连接更易优化拥塞控制（如 BBR），丢包影响小。

## 6、HTTP/2 的多路复用一条 TCP 和 HTTP/1.1 的使用多个 TCP 连接方式，哪个性能更好？

#### HTTP/1.1 使用多个 TCP 连接

##### 工作原理

- HTTP/1.1 默认每个请求需要等待前一个请求完成（队头阻塞，Head-of-Line Blocking）。
- 为提高并发性，浏览器为每个域名建立多个 TCP 连接（通常 6-8 个，视浏览器实现）。
- 每个连接独立处理请求和响应。

#### HTTP/2 的多路复用（单一 TCP 连接）

##### 工作原理

- HTTP/2 将请求和响应分解为二进制帧（Frame），每个帧标记流 ID（Stream ID）。
- 在一条 TCP 连接上并行发送多个流，流之间互不干扰。
- 服务器和客户端按需交错处理帧。

#### 性能对比

以下从多个维度对比两者的性能：

##### (1) 延迟（Latency）

- HTTP/1.1：
  - 多连接需多次握手和 TLS 协商，初始延迟高。
  - 队头阻塞增加单个请求的等待时间。
  - 示例：10 个资源，6 个连接，延迟 ≈ 300-500ms。
- HTTP/2：
  - 单连接一次握手，延迟低。
  - 多路复用并行传输，无队头阻塞。
  - 示例：延迟 ≈ 100-200ms。
- 结论：HTTP/2 在延迟上更优，尤其在高 RTT 网络（如移动网络）。

##### (2) 吞吐量（Throughput）

- HTTP/1.1：
  - 多连接竞争带宽，可能导致拥塞控制不协调。
  - 每个连接慢启动独立进行，初期吞吐量低。
- HTTP/2：
  - 单连接集中优化拥塞控制，快速达到带宽上限。
  - 示例：带宽 100Mbps，HTTP/2 可在 2-3 RTT 内接近峰值，HTTP/1.1 需更多 RTT。
- 结论：HTTP/2 吞吐量更高，尤其在高带宽场景。

##### (3) 资源利用率

- HTTP/1.1：
  - 多个 TCP 连接占用更多服务器资源（如端口、内存）。
  - 示例：100 个客户端 × 6 连接 = 600 个连接。
- HTTP/2：
  - 单连接复用，资源占用低。
  - 示例：100 个客户端 × 1 连接 = 100 个连接。
- 结论：HTTP/2 资源效率更高，适合高并发。

##### (4) 网络适应性

- HTTP/1.1：
  - 高丢包网络中，多连接加剧拥塞，丢包影响更大。
  - 示例：5% 丢包率，6 个连接均需重传。
- HTTP/2：
  - 单连接受丢包影响，但多路复用可优先处理未丢流。
  - 示例：丢包只影响部分流，其他流继续传输。
- 结论：HTTP/2 在不稳定网络中更具优势。

##### (5) 服务器推送（HTTP/2 独有）

- HTTP/2 可主动推送资源，进一步减少 RTT。
- HTTP/1.1 无此功能，需客户端逐个请求。

一般情况下

- HTTP/2 性能更好：
  - 低延迟：单连接减少握手时间。
  - 高效率：多路复用充分利用带宽。
  - 资源优化：适合高并发和复杂页面。
- 数据支持：加载 100 个资源，HTTP/2 比 HTTP/1.1 快 30%-50%。

## 7、在后台开发中，什么时候使用多进程？什么时候使用多线程？

##### 多进程

- 定义：运行多个独立的进程，每个进程有自己的内存空间、地址空间和系统资源。
- 通信：通过进程间通信（IPC）机制，如管道（Pipe）、消息队列、共享内存、Socket。
- 隔离性：进程之间完全隔离，一个进程崩溃不影响其他进程。

##### 多线程

- 定义：在一个进程内运行多个线程，共享同一进程的内存和资源。
- 通信：通过共享内存直接访问数据（如全局变量），但需同步机制（如锁、信号量）。
- 隔离性：线程间耦合紧密，一个线程崩溃可能导致整个进程崩溃。

#### 什么时候使用多进程？

##### CPU 密集型任务

多进程可充分利用多核 CPU，每个进程运行在独立核心上。

##### 高隔离性需求

进程间崩溃不相互影响，适合需要稳定性的任务。

#### 什么时候使用多线程？

##### I/O 密集型任务

线程在等待 I/O（如网络、磁盘）时可切换，充分利用 CPU。

##### 低延迟共享数据

线程共享内存，数据访问无需 IPC，适合频繁通信的任务。

#### 性能与资源对比

| 特性          | 多进程                        | 多线程                          |
| ------------- | ----------------------------- | ------------------------------- |
| 内存使用      | 高（每个进程独立内存）        | 低（线程共享进程内存）          |
| 创建/销毁开销 | 高（fork 或 spawn 成本大）    | 低（线程创建更快）              |
| 通信开销      | 高（需 IPC）                  | 低（直接共享内存）              |
| 隔离性        | 强（进程独立）                | 弱（线程共享进程）              |
| 并行性        | 可充分利用多核（无 GIL 限制） | 受限于 GIL（某些语言如 Python） |
| 调试难度      | 中等（独立运行）              | 高（竞争条件、死锁）            |

## 8、如何处理 Redis 热 Key 问题？

 Redis 中，热 Key 问题是指某些键（Key）被高频访问，导致访问流量集中在少数 Redis 实例或线程上，从而引发性能瓶颈、负载不均甚至服务不可用。这种情况在高并发场景（如秒杀、热点事件、缓存穿透后重建）中尤为常见。

#### 处理热 Key 

##### 分片（Key 分散）

原理：将单一热 Key 拆分为多个子 Key，分散访问压力。

实现：

- 在 Key 中添加随机后缀或分片标识。
- 示例：
  - 原 Key：stock:product123。
  - 分片 Key：stock:product123:shard1、stock:product123:shard2。
- 客户端随机访问某个分片 Key。

##### 本地缓存（客户端缓存）

原理：在客户端或应用服务器内存中缓存热 Key，减少对 Redis 的直接访问。

实现：

- 使用本地缓存（如 Guava Cache、Caffeine）存储热 Key 数据。
- 设置短 TTL（如 1-5 秒），定期从 Redis 更新。

##### 复制（多副本）

- 原理：将热 Key 复制到多个 Redis 实例，客户端随机访问。
- 实现：
  - 主从复制：主节点写入，从节点读取热 Key。
  - 客户端负载均衡：随机选择从节点。

##### 限流（流量控制）

- 原理：限制热 Key 的访问频率，保护 Redis。
- 实现：使用 Redis 的计数器（如 INCR）或令牌桶算法。

##### 热点隔离

- 原理：将热 Key 单独存储到专用 Redis 实例，避免影响其他 Key。
- 实现：
  - 部署独立 Redis 实例（如 hot-redis）。
  - 客户端识别热 Key，定向访问。

## 9、如何统计哪些 Key 是热 Key？

#### 方法 1：使用 Redis 的 MONITOR 命令

原理：

- MONITOR 实时输出 Redis 接收到的所有命令。
- 通过分析命令流，统计 Key 的访问频率。

实现步骤：

1. 在 Redis 客户端执行 MONITOR。
2. 捕获一段时间内的命令。
3. 解析命令，统计 Key 频率。

#### 方法 2：使用 Redis 的 --hotkeys 选项

原理：

- Redis 4.0+ 的 redis-cli 提供 --hotkeys 参数，扫描内存中的 Key，分析访问频率。
- 基于内部统计（需启用相关配置）。

实现步骤：

1. 确保 Redis 实例运行。

2. 执行命令：

   ```
   redis-cli --hotkeys
   ```

#### 方法 3：客户端统计

原理：

- 在应用代码中记录每个 Key 的访问次数，定期汇总。
- 通过客户端拦截 Redis 操作，统计频率。

实现步骤：

1. 在 Redis 客户端封装一层代理。
2. 记录 Key 访问次数。
3. 定时输出或存储统计结果。

#### 方法 4：Redis 插件或扩展

原理：使用 Redis 模块（如 RedisBloom、RedisTimeSeries）或自定义模块统计 Key 访问。

实现步骤：

1. 编写模块（如 C 语言），记录 Key 访问次数。
2. 加载模块到 Redis（loadmodule 命令）。
3. 通过命令查询热 Key。

#### 方法 5：外部监控工具

原理：使用监控系统（如 Prometheus + Redis Exporter）收集 Redis 命令统计，分析热 Key。

实现步骤：

1. 部署 Redis Exporter，暴露 Redis 指标。
2. 配置 Prometheus 抓取数据。
3. 用 Grafana 可视化 Key 访问频率。

## 10、如何保证本地缓存的一致性？

#### 本地缓存一致性问题的成因

##### (1) 数据更新不同步

场景：

- 数据库或 Redis 更新数据（如库存减少），本地缓存未及时刷新。
- 多实例部署（如多台服务器），某实例更新数据，其他实例未感知。

结果：本地缓存返回旧数据。

##### (2) 并发修改

场景：多线程或多客户端同时更新同一 Key。

结果：缓存与源数据可能出现短暂不一致（如更新顺序混乱）。

##### (3) 缓存失效

场景：本地缓存过期后，未及时从源数据刷新。

结果：返回过期数据或触发缓存击穿。

#### 保证一致性的核心策略

一致性分为强一致性（实时同步）和最终一致性（允许短暂不一致）。

##### (1) 更新策略

- 写后失效：更新源数据后，主动失效本地缓存。
- 写后更新：更新源数据同时更新本地缓存。
- 写回：先更新本地缓存，异步同步到源数据。

##### (2) 同步机制

- 定时刷新：定期从源数据更新本地缓存。
- 事件通知：监听源数据变更，实时同步。
- 分布式锁：控制并发更新顺序。

##### (3) 失效控制

- TTL：设置过期时间，强制刷新。
- LRU/LFU：淘汰不活跃 Key，保证新鲜度。

## 11、如何防止缓存击穿？热 key 缓存到本地，是有时间差的，这时的击穿风险如何解决？

缓存击穿是指在高并发场景下，某个热 Key（热点数据）因缓存失效（如过期或被主动移除），导致大量请求同时穿透到后端数据源（如数据库），造成数据库压力激增甚至宕机。热 Key 缓存到本地虽然能减轻 Redis 的压力，但由于本地缓存与源数据（如 Redis 或数据库）之间存在同步时间差，击穿风险依然存在。

#### 解决方法

##### 方法 1：设置永不过期

原理：热 Key 不设置 TTL，保持缓存常驻。

实现：

- Redis：SET key value（不加 EXPIRE）。
- 本地缓存：Caffeine 设置无过期时间。

##### 方法 2：异步刷新（后台更新）

原理：缓存快过期时，异步线程提前刷新，避免失效瞬间击穿。

实现：设置 TTL，监控剩余时间，提前刷新。

##### 方法 3：双重检查锁

原理：高并发下，只允许一个线程加载源数据，其他线程等待。

实现：缓存未命中时，加锁检查并更新。

##### 方法 4：分布式锁

原理：使用 Redis 分布式锁，控制多实例并发加载。

## 12、最小路径和

#### 解题思路

##### 定义状态

- 定义`dp[i][j]` 表示从 (0,0) 到 (i,j) 的最小路径和。

  最终目标是求 `dp[m-1][n-1]`。

##### 状态转移方程

对于位置` (i,j)`：

- 若 `i > 0` 且 `j > 0`：`dp[i][j] = min(dp[i-1][j], dp[i][j-1]) + grid[i][j]`。
- 若 `i = 0`：只能从左边来，`dp[0][j] = dp[0][j-1] + grid[0][j]`。
- 若 `j = 0`：只能从上面来，`dp[i][0] = dp[i-1][0] + grid[i][0]`。

##### 初始条件

起点：`dp[0][0] = grid[0][0]`。

##### 遍历顺序

从左上角到右下角，按行（i）或列（j）递增填充 DP 表。

#### 参考代码（C++）

```c++
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int minPathSum(vector<vector<int>>& grid) {
        if (grid.empty() || grid[0].empty()) return 0;

        int m = grid.size();    // 行数
        int n = grid[0].size(); // 列数
        vector<vector<int>> dp(m, vector<int>(n, 0)); // DP 数组

        // 初始化起点
        dp[0][0] = grid[0][0];

        // 初始化第一行
        for (int j = 1; j < n; j++) {
            dp[0][j] = dp[0][j-1] + grid[0][j];
        }

        // 初始化第一列
        for (int i = 1; i < m; i++) {
            dp[i][0] = dp[i-1][0] + grid[i][0];
        }

        // 填充 DP 表
        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                dp[i][j] = min(dp[i-1][j], dp[i][j-1]) + grid[i][j];
            }
        }

        return dp[m-1][n-1]; // 返回终点的最小路径和
    }
};
```

