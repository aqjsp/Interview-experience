# 腾讯天美C++面经

>来源：https://www.nowcoder.com/feed/main/detail/2080e796a0a64372b8cee0cbf7cf79e9

## 一面

### 1、Linux命令？

### 2、什么是网络拥塞机制？

拥塞控制是指在网络中发生拥塞时，减少向网络中发送数据的速度，防止造成恶性循环；同时在网络空闲时，提高发送数据的速度，最大限度地利用网络资源。

以 TCP 协议为例，其拥塞控制机制主要包括以下算法：

1. 慢启动： 在连接建立初期，TCP 会将拥塞窗口（cwnd）初始化为一个较小的值，然后每收到一个确认（ACK），拥塞窗口就增加一个 MSS 大小。
2. 拥塞避免： 当拥塞窗口达到阈值后，TCP 不再采用慢启动的增长方式，而是进入拥塞避免阶段。在这个阶段，拥塞窗口会以线性方式增长，以避免快速增长导致的网络拥塞风险。
3. 快速重传： 当发送方收到三个重复的确认时，会立即重传丢失的数据包，而不必等待重传计时器超时。
4. 快速恢复： 与快速重传配合使用，在快速重传后，TCP 不会立即回到慢启动阶段，而是进入快速恢复阶段。

通过这些机制，TCP 能够动态地根据网络状况调整数据发送速率，从而在保证数据传输效率的同时，避免因过快发送数据而导致的网络拥塞问题。

### 3、为什么TIME_WAIT2要等待2RTT？

1. **确保可靠的连接关闭：** 在 TCP 的四次挥手关闭过程中，主动关闭连接的一方在发送最后的 ACK 后进入 `TIME_WAIT` 状态。 如果对方的 FIN 或 ACK 包丢失，主动关闭方需要重传相应的包。 `TIME_WAIT` 状态的持续时间为 2MSL，确保了在此期间，网络中可能存在的重传包能够被丢弃，避免它们被误认为是新连接的数据包，从而导致数据混乱。

2. **防止旧数据包影响新连接：** TCP 连接由四元组（源 IP、源端口、目标 IP、目标端口）唯一标识。 如果在关闭连接后，网络中仍有旧的数据包存在，且这些数据包的四元组与新建立的连接相同，可能导致旧数据被误传递到新连接的应用层。 通过在 `TIME_WAIT` 状态下保持连接，确保网络中的旧数据包在 2MSL 时间内被丢弃，避免对新连接造成影响。

### 4、什么是静态多态，什么是虚函数？

#### 静态多态（编译时多态）

静态多态是在编译期间确定函数调用的绑定方式，主要通过函数重载和模板实现。

- **函数重载：** 在同一作用域内，可以定义多个同名函数，但它们的参数列表（参数个数、类型、顺序）不同。编译器根据函数调用时传入的参数类型和数量来确定调用哪个函数。

  ```c++
    #include <iostream>
  
    class Printer {
    public:
        void print(int i) {
            std::cout << "Printing integer: " << i << std::endl;
        }
  
        void print(double d) {
            std::cout << "Printing double: " << d << std::endl;
        }
    };
  
    int main() {
        Printer p;
        p.print(5);     // 调用 print(int)
        p.print(3.14);  // 调用 print(double)
        return 0;
    }
  ```

- **函数模板：** 通过模板，可以编写泛型函数，使其能够处理多种数据类型。编译器在编译时根据传入的参数类型生成相应的函数实例。

  ```c++
    #include <iostream>
  
    template <typename T>
    void print(T t) {
        std::cout << "Printing: " << t << std::endl;
    }
  
    int main() {
        print(5);     // 打印整数
        print(3.14);  // 打印浮点数
        print("Hello"); // 打印字符串
        return 0;
    }
  ```

#### 动态多态（运行时多态）

动态多态是在运行期间确定函数调用的绑定方式，主要通过虚函数和继承实现。

**虚函数：** 在基类中使用 `virtual` 关键字声明的成员函数称为虚函数。派生类可以重写这些虚函数。当通过基类指针或引用调用虚函数时，程序会在运行时根据对象的实际类型来决定调用哪个派生类的重写函数。

```c++
  #include <iostream>

  class Animal {
  public:
      virtual void makeSound() {
          std::cout << "Animal makes sound" << std::endl;
      }
  };

  class Dog : public Animal {
  public:
      void makeSound() override {
          std::cout << "Dog barks" << std::endl;
      }
  };

  int main() {
      Animal* animal = new Dog();
      animal->makeSound(); // 输出: Dog barks
      delete animal;
      return 0;
  }
```

### 5、什么是volatile变量？

#### 作用

1. **防止编译器优化：** 编译器可能会对代码进行优化，例如将变量缓存到寄存器中，以提高访问速度。 如果一个变量的值可能在程序外部被改变，编译器可能会错误地优化对该变量的访问，导致程序行为不符合预期。 使用 `volatile` 修饰符，编译器会每次直接从内存读取该变量的值，避免了优化带来的问题。 

2. **确保变量的可见性：** 在多线程或中断服务程序中，一个线程或中断可能会修改某个变量的值，而其他线程或主程序需要读取该值。 如果该变量未被声明为 `volatile`，编译器可能会缓存其值，导致其他线程或主程序无法获取到最新的值。 使用 `volatile` 可以确保每次访问该变量时，都直接从内存中读取最新的值。

#### 使用场景

1. **中断服务程序中的变量：** 在嵌入式系统中，中断服务程序可能会修改某些变量的值，而主程序需要读取这些值。 将这些变量声明为 `volatile`，可以确保主程序每次读取时都获取到最新的值。 

2. **多线程共享变量：** 在多线程环境中，多个线程可能会共享某些变量。 如果这些变量的值可能被任何线程修改，应该将其声明为 `volatile`，以确保每个线程都能读取到最新的值。 

3. **硬件寄存器访问：** 在嵌入式编程中，硬件寄存器的值可能会被硬件本身的操作所改变。 使用 `volatile` 修饰硬件寄存器变量，可以确保每次访问都是直接从硬件寄存器中读取，而不是从 CPU 缓存中读取。

### 6、算法

#### 6.1、手撕快排？

基本思想：从待排序的数组中选择一个元素作为基准（pivot），将数组分为两部分，一部分所有元素都小于基准，另一部分所有元素都大于基准，然后递归地对这两部分进行排序。

```c++
#include <iostream>
#include <stack>
#include <vector>

void swap(int& a, int& b) {
    int temp = a;
    a = b;
    b = temp;
}

int partition(std::vector<int>& arr, int low, int high) {
    int pivot = arr[low];
    int left = low;
    int right = high;
    while (left < right) {
        while (arr[left] <= pivot && left < right) {
            left++;
        }
        while (arr[right] > pivot) {
            right--;
        }
        if (left < right) {
            swap(arr[left], arr[right]);
        }
    }
    swap(arr[low], arr[right]);
    return right;
}

void quickSort(std::vector<int>& arr, int low, int high) {
    std::stack<int> s;
    s.push(low);
    s.push(high);

    while (!s.empty()) {
        high = s.top();
        s.pop();
        low = s.top();
        s.pop();

        int pivotIndex = partition(arr, low, high);

        if (pivotIndex - 1 > low) {
            s.push(low);
            s.push(pivotIndex - 1);
        }
        if (pivotIndex + 1 < high) {
            s.push(pivotIndex + 1);
            s.push(high);
        }
    }
}

int main() {
    std::vector<int> arr = {6, 5, 7, 9, 2, 0, 3, 1, 8, 4, 10};
    quickSort(arr, 0, arr.size() - 1);

    for (int num : arr) {
        std::cout << num << " ";
    }
    return 0;
}
```

#### 6.2、LRUCache？

#### 实现步骤：

1. **定义双向链表节点：** 每个节点包含键、值，以及指向前后节点的指针。
2. **设计 `LRUCache` 类：** 包含以下成员：
   - `cap`：缓存的容量。
   - `cache`：双向链表，存储缓存项。
   - `hashmap`：哈希表，存储键到链表节点的映射。
3. **实现核心操作：**
   - `get(int key)`：如果键存在于缓存中，返回其值，并将该节点移动到链表头部；否则，返回 -1。
   - `put(int key, int value)`：如果键已存在，更新其值，并将该节点移动到链表头部；如果键不存在，插入新节点，并在缓存满时删除尾部节点。

#### 代码实现

```c++
#include <iostream>
#include <list>
#include <unordered_map>
#include <utility>

class LRUCache {
private:
    int cap;  // 缓存容量
    std::list<std::pair<int, int>> cache;  // 双向链表，存储缓存项
    std::unordered_map<int, std::list<std::pair<int, int>>::iterator> hashmap;  // 哈希表，存储键到链表节点的映射

public:
    // 构造函数，初始化缓存容量
    LRUCache(int capacity) : cap(capacity) {}

    // 获取缓存中的值，如果键不存在，返回 -1
    int get(int key) {
        if (hashmap.find(key) == hashmap.end()) {
            return -1;
        }
        // 将访问的节点移动到链表头部
        auto it = hashmap[key];
        cache.splice(cache.begin(), cache, it);
        return it->second;
    }

    // 向缓存中插入或更新键值对
    void put(int key, int value) {
        if (hashmap.find(key) != hashmap.end()) {
            // 更新已存在的键值对
            auto it = hashmap[key];
            it->second = value;
            // 将该节点移动到链表头部
            cache.splice(cache.begin(), cache, it);
            return;
        }

        if (cache.size() >= cap) {
            // 缓存已满，删除最久未使用的节点
            auto del = cache.back();
            hashmap.erase(del.first);
            cache.pop_back();
        }

        // 插入新节点到链表头部
        cache.emplace_front(key, value);
        hashmap[key] = cache.begin();
    }
};

int main() {
    LRUCache cache(2);
    cache.put(1, 1);
    cache.put(2, 2);
    std::cout << cache.get(1) << std::endl; // 返回 1
    cache.put(3, 3); // 该操作会使得关键字 2 作废
    std::cout << cache.get(2) << std::endl; // 返回 -1 (未找到)
    cache.put(4, 4); // 该操作会使得关键字 1 作废
    std::cout << cache.get(1) << std::endl; // 返回 -1 (未找到)
    std::cout << cache.get(3) << std::endl; // 返回 3
    std::cout << cache.get(4) << std::endl; // 返回 4
    return 0;
}
```

## 二面

### 1、什么是RPC？RPC如何实现？

RPC（Remote Procedure Call，远程过程调用）是一种通信协议，允许程序在不同的地址空间（通常是不同的计算机）上调用另一个程序的子程序或服务，就像调用本地函数一样。 

#### 工作原理：

1. **客户端调用：** 客户端应用程序调用本地的代理（stub），该代理负责将调用请求序列化并发送到服务器。
2. **网络传输：** 请求通过网络传输到远程服务器。
3. **服务器处理：** 服务器接收到请求后，解包参数并调用相应的服务方法。
4. **返回结果：** 服务器执行完操作后，将结果打包并通过网络返回给客户端。
5. **客户端接收：** 客户端接收到结果后，解包并将结果传递给原始调用者。

#### RPC的关键组件：

- **客户端Stub：** 负责将客户端的调用请求序列化并发送到服务器。
- **服务器Skeleton：** 负责接收客户端的请求，解包并调用相应的服务方法。
- **传输协议：** 常用的有HTTP、TCP等。
- **序列化/反序列化：** 用于将对象转换为字节流（序列化）和将字节流转换回对象（反序列化）。

### 2、怎么实现可靠udp？

UDP（用户数据报协议）是一种无连接、面向报文的传输层协议，不提供可靠性保障。为了在UDP上实现可靠传输，需要在应用层或结合其他协议添加额外的机制。

常见的实现方法：

**1. 超时重传机制：**

发送方在发送数据包后设置一个定时器，如果在定时器超时之前没有收到确认包，则重新发送该数据包。这种机制可以确保丢失的数据包被重传，提高传输的可靠性。 

**2. 序列号和确认应答机制：**

每个数据包被分配一个递增的序列号，接收方在接收到数据包后发送一个确认应答（ACK），发送方根据ACK来判断数据包是否成功接收。如果在超时时间内未收到ACK，发送方会重新发送该数据包。 

**3. 滑动窗口流量控制：**

通过滑动窗口协议来控制数据传输速率，确保网络不被过量的数据包塞满。接收方通过反馈窗口大小来控制发送方的发送速率，避免网络拥塞。 

**4. 错误检测和校验和：**

虽然UDP提供校验和来检测数据包在传输过程中是否被篡改或损坏，但这并不保证数据的完整性或可靠性。因此，需要在应用层实现更强大的错误检测和校验机制，以确保数据的完整性。 

**5. 应用层实现TCP特性：**

在应用层模仿TCP的可靠性传输特性，如序列号、确认应答、超时重传等。这种方法可以在UDP的基础上实现类似于TCP的可靠性，但需要开发者自行处理相关机制。 

**6. 使用改进的协议：**

例如，QUIC协议，它基于UDP但引入了类似于TCP的可靠性机制，如序号和确认机制，以及改进的拥塞控制算法。QUIC协议通过引入流量控制、拥塞控制和加密等机制，提供了类似于TCP的可靠性和安全性。 

**7. 使用现有的可靠UDP协议：**

一些现有的协议在UDP的基础上增加了可靠性机制，如RUDP、RTP和UDT等。这些协议通过引入序列号、确认应答、重传机制和流量控制等，提供了可靠的数据传输。 

### 3、了解拥塞控制吗？

### 4、什么是快重传？

#### 工作原理：

在TCP连接中，接收方会对收到的数据包发送确认（ACK）。如果接收方收到一个乱序的数据包（即期望的下一个数据包尚未到达），它会重复发送对上一个已成功接收的数据包的确认。

当发送方连续收到三个相同的重复ACK时，认为对应的数据包丢失，立即重传该数据包，而无需等待超时重传。

#### 示例：

假设发送方发送了数据包1、2、3、4、5，接收方成功接收到1、2、4、5，但未收到3。接收方会连续发送对数据包2的确认。发送方连续收到三个对数据包2的重复ACK后，立即重传数据包3。

### 5、进程调度算法有哪些？

#### 1. 先来先服务（FCFS，First-Come, First-Served）调度算法：

- **原理：** 按照进程到达就绪队列的顺序分配CPU，先到达的进程先被调度执行。
- **优点：** 实现简单，公平地处理进程。
- **缺点：** 可能导致“长进程”占用CPU过长时间，导致“短进程”等待过久，增加平均周转时间。

#### 2. 短作业优先（SJF，Shortest Job First）调度算法：

- **原理：** 优先调度估计运行时间最短的进程。
- **优点：** 能最小化平均周转时间。
- **缺点：** 需要准确预测进程的执行时间，可能导致“长进程”饿死。

#### 3. 优先级调度算法：

- **原理：** 为每个进程分配一个优先级，优先级高的进程先被调度执行。
- **优点：** 适用于需要对关键任务进行及时响应的系统。
- **缺点：** 可能导致低优先级进程长时间得不到执行，发生“饥饿”现象。

#### 4. 时间片轮转（RR，Round Robin）调度算法：

- **原理：** 将CPU时间划分为固定长度的时间片，按顺序循环分配给各个进程。
- **优点：** 公平地分配CPU时间，适用于交互式系统。
- **缺点：** 时间片的大小对系统性能影响较大，过小会导致频繁的上下文切换，过大则可能导致响应时间增加。

#### 5. 多级反馈队列调度算法：

- **原理：** 结合优先级调度和时间片轮转，根据进程的行为动态调整其优先级和时间片大小。
- **优点：** 能兼顾多方面的系统目标，如提高系统吞吐量和缩短平均周转时间。
- **缺点：** 实现复杂，需要合理设置各级队列的优先级和时间片大小。

### 6、算法题：矩形中的最大正方形面积？

#### 解题思路

定义一个二维数组`dp`，其中`dp[i][j]`表示以`matrix[i][j]`为右下角的最大正方形的边长。

状态转移方程为：`dp[i][j] = min(dp[i-1][j-1], dp[i-1][j], dp[i][j-1]) + 1`，

当`matrix[i][j]`为'1'时，`dp[i][j]`的值取决于其上方、左方和左上方三个位置的`dp`值的最小值加1；

当`matrix[i][j]`为'0'时，`dp[i][j]`为0。

遍历整个矩阵，更新`dp`数组，并记录最大边长，最后返回最大边长的平方即为最大正方形的面积。

#### 代码实现

```c++
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int maximalSquare(vector<vector<char>>& matrix) {
        if (matrix.empty() || matrix[0].empty()) return 0;
        int rows = matrix.size();
        int cols = matrix[0].size();
        vector<vector<int>> dp(rows + 1, vector<int>(cols + 1, 0));
        int max_side = 0;
        for (int i = 1; i <= rows; ++i) {
            for (int j = 1; j <= cols; ++j) {
                if (matrix[i - 1][j - 1] == '1') {
                    dp[i][j] = min({dp[i - 1][j - 1], dp[i - 1][j], dp[i][j - 1]}) + 1;
                    max_side = max(max_side, dp[i][j]);
                }
            }
        }
        return max_side * max_side;
    }
};
```

