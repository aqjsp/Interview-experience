# 字节C++一面，抖音直播方向



> 来源：https://www.nowcoder.com/feed/main/detail/191643ece75b4c5c95e4047f187e61e4

### 1、Python和C++的区别，Python的程序执行过程是怎样的？C和C++呢？

#### Python和C++的区别？

1. **语言类型**：
   - Python：高级、解释型、动态类型语言。强调简洁和可读性，适合快速开发和原型制作。
   - C++：中级、编译型、静态类型语言。结合了高层语言的抽象性与低层语言的效率，适合系统编程和性能要求高的应用。
2. **语法**：
   - Python：语法简洁，使用缩进表示代码块，不需要使用分号结束语句。
   - C++：语法相对复杂，需要使用分号结束语句，支持多种编程范式（面向对象、泛型等）。
3. **内存管理**：
   - Python：自动内存管理和垃圾回收，开发者无需手动管理内存。
   - C++：需要手动管理内存，使用 `new` 和 `delete` 进行动态内存分配和释放，提供更大的灵活性和控制力。
4. **执行速度**：
   - Python：相对较慢，因为是解释型语言，每次执行时都需要解析源代码。
   - C++：通常执行速度较快，因其为编译型语言，编译后生成机器代码直接执行。
5. **库和生态**：
   - Python：拥有丰富的第三方库和框架，广泛应用于数据科学、机器学习、Web 开发等领域。
   - C++：提供标准模板库（STL），适合系统级编程、游戏开发和高性能计算。

#### Python 的程序执行过程？

1. 源代码编写：开发者使用文本编辑器编写 Python 源代码，通常保存为 `.py` 文件。
2. 解释器读取：Python 解释器逐行读取源代码，执行时会将代码转换为中间字节码。
3. 字节码生成：Python 源代码被编译为字节码（.pyc 文件），字节码是一种中间表示，便于在 Python 虚拟机上执行。
4. 虚拟机执行：字节码由 Python 虚拟机（PVM）执行。解释器将字节码逐条解释并执行，同时处理内存管理、异常处理等。
5. 输出结果：程序运行结束后，输出结果或执行特定操作。Python 提供了交互式解释器，方便调试和测试。

#### C和C++的区别？

##### 1. 编程范式

- C：是一种过程式编程语言，强调函数的使用，通过函数和结构体来组织代码。
- C++：是一种多范式编程语言，支持面向对象编程（OOP），允许使用类和对象来组织代码，提供封装、继承和多态等特性。

##### 2. 数据抽象

- C：没有内置的支持面向对象的特性，所有数据结构都是通过结构体和函数来实现。
- C++：支持类和对象，可以通过封装将数据和操作组合在一起，提供更好的数据抽象。

##### 3. 内存管理

- C：使用 `malloc` 和 `free` 进行动态内存分配和释放，需要手动管理内存。
- C++：提供 `new` 和 `delete` 操作符用于动态内存分配和释放，支持构造函数和析构函数，能够在对象创建和销毁时自动处理资源管理。

##### 4. 标准库

- C：提供标准库（如 `stdlib.h`, `stdio.h`）主要用于基本输入输出和内存管理。
- C++：提供标准模板库（STL），支持容器（如向量、链表、集合）、算法和迭代器，提供了更丰富的数据结构和算法支持。

##### 5. 异常处理

- C：不支持异常处理，错误处理通常通过返回值来实现。
- C++：支持异常处理机制（`try`、`catch` 和 `throw`），提供更优雅的错误处理方式。

##### 6. 函数重载和运算符重载

- C：不支持函数重载，所有函数名称必须唯一。
- C++：支持函数重载和运算符重载，可以定义多个同名函数，允许使用自定义的方式操作对象。

##### 7. 命名空间

- C：没有命名空间，所有标识符在全局范围内唯一。
- C++：支持命名空间（`namespace`），避免命名冲突，增强代码组织性。

##### 8. 模板

- C：不支持模板。
- C++：支持模板，可以编写泛型代码，提高代码的复用性。

### 2、操作系统的内存管理方式？

#### 1. 连续内存分配

##### 1.1 固定分区分配

实现方式：系统将内存划分为固定大小的多个分区（例如，每个分区 1MB），每个分区可以分配给一个进程。

优点：

- 管理简单，分区大小固定，易于实现。
- 进程的内存分配和回收过程简单明了。

缺点：

- 可能出现内部碎片（如一个 1.5MB 的进程只能放入一个 2MB 的分区，留下 0.5MB 未使用）。
- 分区数量固定，无法动态调整。

##### 1.2 可变分区分配

实现方式：根据进程的需求动态分配内存，空闲内存被划分为可变大小的分区。

优点：减少了内部碎片，因为每个进程只使用所需的内存。

缺点：

- 外部碎片问题，随着内存的分配和释放，可能会出现无法使用的空闲内存块。
- 管理较复杂，需要有效的算法来分配和回收内存。

#### 2. 分页

实现方式：将物理内存划分为固定大小的页面（如 4KB），每个进程也被划分为相同大小的页面。使用页表来管理虚拟页与物理页的映射。

优点：

- 消除了外部碎片，物理内存中的页面可以不连续。
- 方便内存的分配与管理，程序的加载和运行更加灵活。

缺点：

- 页表需要占用内存，如果进程较大，页表可能非常庞大。
- 页表查找可能导致额外的时间开销（如 TLB 缓存未命中）。

#### 3. 分段

实现方式：将程序的逻辑结构（如函数、数组、全局变量）划分为不同大小的段，使用段表来管理。

优点：

- 更符合程序的逻辑结构，方便程序的管理和共享（例如共享库）。
- 提供了段的访问控制，可以设定不同段的权限。

缺点：

- 可能导致外部碎片，因为段的大小不固定，内存利用率不如分页高。
- 段的管理和访问比分页复杂。

#### 4. 虚拟内存

实现方式：系统允许进程使用的地址空间大于物理内存，通过页面置换算法（如 LRU、FIFO）将不常用的页面移至磁盘，必要时再调入内存。

优点：

- 使得程序能够运行在比物理内存大得多的地址空间中，支持更复杂的应用。
- 提高了内存的利用率，允许多个进程在内存中共存。

缺点：

- 页面置换可能导致较高的 I/O 开销，频繁的页面置换会导致性能下降（称为“页面抖动”）。
- 实现复杂，需要设计高效的页替换算法和管理策略。

#### 5. 内存交换（Swapping）

实现方式：当系统内存不足时，操作系统会将整个进程从内存交换到磁盘，释放内存空间，待进程需要时再交换回内存。

优点：允许在物理内存不足的情况下运行更多进程。

缺点：

- 交换操作开销大，频繁交换会影响系统性能。
- 需要处理进程状态的保存和恢复，增加了管理复杂度。

#### 6. 内存保护

实现方式：通过硬件支持（如基址寄存器和限界寄存器）来限制进程访问的内存范围，防止进程间的相互干扰。

优点：提高了系统的稳定性和安全性，防止错误的内存访问导致的崩溃或数据损坏。

缺点：需要硬件支持，增加了设计的复杂性。

### 3、排序算法，按照时间复杂度分类

| 排序算法 | 平均时间复杂度 | 最好情况  | 最坏情况  | 空间复杂度 | 稳定性 |
| -------- | -------------- | --------- | --------- | ---------- | ------ |
| 冒泡排序 | O(n^2)         | O(n)      | O(n^2)    | O(1)       | 稳定   |
| 快速排序 | O(nlogn)       | O(nlogn)  | O(n^2)    | O(logn)    | 不稳定 |
| 选择排序 | O(n^2)         | O(n^2)    | O(n^2)    | O(1)       | 不稳定 |
| 插入排序 | O(n^2)         | O(n)      | O(n^2)    | O(1)       | 稳定   |
| 希尔排序 | O(nlogn)       | O(log^2n) | O(log^2n) | O(1)       | 不稳定 |
| 归并排序 | O(nlogn)       | O(nlogn)  | O(nlogn)  | O(n)       | 稳定   |
| 堆排序   | O(nlogn)       | O(nlogn)  | O(nlogn)  | O(1)       | 不稳定 |
| 计数排序 | O(n+k)         | O(n+k)    | O(n+k)    | O(k)       | 稳定   |
| 桶排序   | O(n+k)         | O(n+k)    | O(n^2)    | O(n+k)     | 稳定   |
| 基数排序 | O(nxk)         | O(nxk)    | O(nxk)    | O(n+k)     | 稳定   |

### 4、TCP和UDP的区别，以及使用场景？

#### 1. 基本区别

##### 1.1 连接方式

TCP：

- 面向连接协议，数据传输前需要建立连接。
- 使用三次握手（Three-way Handshake）建立连接，以确保双方都准备好通信。

UDP：

- 面向无连接协议，不需要建立连接。
- 数据可以直接发送，不需要握手过程。

##### 1.2 可靠性

TCP：

- 提供可靠的数据传输，确保数据包按顺序到达，并且没有丢失。
- 使用确认应答（ACK）机制和重传机制，如果数据包丢失会自动重传。

UDP：

- 不提供可靠性保障，数据包可能丢失、重复或乱序。
- 不会对数据包进行确认或重传，应用程序需要自行处理丢包和错误。

##### 1.3 传输速度

TCP：由于需要建立连接、确认应答和重传机制，传输速度相对较慢。

UDP：由于没有连接和重传机制，传输速度较快，适合实时应用。

##### 1.4 数据流量控制

TCP：实现流量控制，避免网络拥塞，自动调整数据发送速度。

UDP：不提供流量控制，发送速率完全由应用程序控制。

##### 1.5 数据边界

TCP：视为一个连续的字节流，数据在应用层中不区分边界。

UDP：以数据报的形式传输，应用层可以接收固定大小的数据包。

#### 2. 头部结构

TCP 头部：TCP 头部较长（20-60字节），包含源端口、目标端口、序列号、确认号、标志位、窗口大小、校验和等信息。

UDP 头部：UDP 头部较短（8字节），只包含源端口、目标端口、长度和校验和。

#### 3. 使用场景

##### 3.1 TCP 使用场景

- 文件传输：如 FTP、SFTP 等协议，需要确保文件完整传输。
- 网页浏览：HTTP/HTTPS 协议，确保网页内容的可靠传输。
- 电子邮件：如 SMTP、POP3、IMAP 等，确保邮件的可靠投递。
- 远程登录：如 SSH，确保指令和数据的准确性和可靠性。

##### 3.2 UDP 使用场景

- 实时音视频：如 VoIP、视频会议等，对延迟敏感，不需要完全可靠性。
- 在线游戏：对实时性要求高，丢包可容忍，UDP 可以减少延迟。
- 广播和多播：如 DHCP、DNS 发现等，需要向多个接收者发送相同的数据包。
- 简单的查询响应：如 DNS 查询，快速响应且不需要重传。

### 5、C++中vector的介绍？

#### 1. 基本特性

- 动态大小：`vector` 的大小可以在运行时动态改变。当元素数量增加时，`vector` 会自动分配更多的内存。
- 存储类型：`vector` 可以存储任何类型的数据，包括基本类型、自定义类型、对象等。
- 随机访问：提供对元素的随机访问，支持通过下标直接访问元素，类似于数组。
- 内存管理：自动处理内存分配和释放，减少内存泄漏的风险。

#### 2. 主要操作

##### 2.1 创建和初始化

```c++
#include <vector>

// 默认构造函数
std::vector<int> vec1; // 创建一个空的整数 vector

// 指定大小的构造函数
std::vector<int> vec2(10); // 创建一个大小为 10 的 vector，初始值为 0

// 指定大小和初始值
std::vector<int> vec3(10, 5); // 创建一个大小为 10 的 vector，所有元素初始为 5

// 使用初始化列表
std::vector<int> vec4 = {1, 2, 3, 4, 5}; // 创建并初始化
```

##### 2.2 添加和删除元素

```c++
// 添加元素
vec1.push_back(1); // 在末尾添加元素 1
vec1.push_back(2); // 在末尾添加元素 2

// 删除元素
vec1.pop_back(); // 删除末尾的元素
vec1.erase(vec1.begin()); // 删除第一个元素
```

##### 2.3 访问元素

```c++
int first = vec1[0]; // 使用下标访问第一个元素
int second = vec1.at(1); // 使用 at() 方法访问第二个元素，具有边界检查
```

##### 2.4 其他常用成员函数

```c++
// 获取当前大小
size_t size = vec1.size(); // 返回当前元素数量

// 判断是否为空
bool isEmpty = vec1.empty(); // 返回 true 如果 vector 为空

// 清空元素
vec1.clear(); // 移除所有元素

// 访问首尾元素
int front = vec1.front(); // 返回第一个元素
int back = vec1.back(); // 返回最后一个元素
```

#### 3. 内存管理

- `vector` 会在需要时自动扩展其容量。初始时容量可能小于大小，当元素数量超出容量时，`vector` 会分配新的内存并将现有元素复制到新内存中。
- 由于这种扩展操作可能涉及大量内存复制，因此在预先知道 `vector` 最终大小的情况下，使用 `reserve()` 方法预分配内存可以提高性能。

```c++
vec1.reserve(100); // 预分配内存，避免多次扩展
```

#### 4. 迭代器

- `vector` 支持 STL 迭代器，可以使用迭代器遍历元素。

```c++
for (auto it = vec1.begin(); it != vec1.end(); ++it) {
    std::cout << *it << " ";
}
```

- 使用范围-based for 循环遍历更简洁。

```c++
for (const auto& value : vec1) {
    std::cout << value << " ";
}
```

#### 5. 适用场景

- 当需要一个可动态调整大小的数组时，`vector` 是最优选择。
- 适用于需要频繁添加和删除元素的场景，尤其是在尾部插入时。
- 由于随机访问特性，适用于需要快速访问元素的应用。

### 6、哈希表如何实现？哈希冲突？

哈希表是一种用于存储键值对的数据结构，它通过哈希函数将键映射到特定的索引位置，从而实现快速的查找、插入和删除操作。

#### 1. 哈希表的基本结构

##### 1.1 哈希函数

哈希函数是将输入的键（key）映射到哈希表索引的函数。好的哈希函数应该具有以下特性：

- 均匀性：能够将键均匀分布到哈希表的索引中，减少冲突。
- 计算效率：能够快速计算索引。

常见的哈希函数包括取模法、乘法法等。

##### 1.2 哈希表结构

哈希表通常由一个数组和一些辅助结构（如链表或树）组成。数组用于存储值，而辅助结构用于处理哈希冲突。

```
#include <vector>
#include <string>

template <typename K, typename V>
class HashTable {
public:
    struct Node {
        K key; // 存储键
        V value; // 存储值
        Node* next; // 指向下一个节点
    };
    
    std::vector<Node*> table; // 哈希表的数组
    int size; // 哈希表的大小

    HashTable(int s) : size(s) {
        table.resize(size, nullptr); // 初始化哈希表
    }
    
    ~HashTable() {
        clear(); // 析构时清理内存
    }

    void clear() {
        for (int i = 0; i < size; ++i) {
            Node* current = table[i];
            while (current) {
                Node* temp = current;
                current = current->next;
                delete temp; // 释放内存
            }
            table[i] = nullptr; // 清空链表
        }
    }
};
```

#### 2. 哈希冲突

哈希冲突是指不同的键通过哈希函数映射到同一个索引的位置。这种情况是不可避免的，尤其是在键的数量大于数组的大小时。处理哈希冲突的常见方法有两种：链式法和开放地址法。

##### 2.1 链式法（Separate Chaining）

- 每个数组元素存储一个链表（或其他数据结构），用于存储所有映射到该索引的键值对。
- 当发生冲突时，新的键值对被添加到链表中。

```
// 插入操作示例
void insert(K key, V value) {
    int index = hashFunction(key) % size; // 计算索引
    Node* newNode = new Node{key, value, nullptr}; // 创建新节点

    // 如果链表为空，直接插入
    if (!table[index]) {
        table[index] = newNode;
    } else {
        // 否则，查找链表末尾
        Node* current = table[index];
        while (current->next) {
            current = current->next; // 移动到链表末尾
        }
        current->next = newNode; // 添加到链表末尾
    }
}
```

##### 2.2 开放地址法（Open Addressing）

- 在哈希表中寻找下一个可用的位置来存储键值对，常见的策略有：
  - 线性探测：从发生冲突的位置开始，线性向后查找空位。
  - 二次探测：通过二次方函数来查找空位。
  - 双重哈希：使用第二个哈希函数确定步长。

```
// 插入操作示例（线性探测）
void insert(K key, V value) {
    int index = hashFunction(key) % size;
    while (table[index] != nullptr) { // 找到空位置
        index = (index + 1) % size; // 线性探测
    }
    table[index] = new Node{key, value, nullptr}; // 插入
}
```

#### 3. 哈希表的操作

##### 3.1 插入

1. 使用哈希函数计算索引。
2. 检查该索引位置的元素。
   - 如果为空，则直接插入。
   - 如果不为空，则根据选择的冲突处理方法进行处理（链式法或开放地址法）。

##### 3.2 查找

1. 通过哈希函数计算索引。
2. 检查索引位置：
   - 如果为空，返回未找到。
   - 如果不为空，遍历链表或根据开放地址法查找，直到找到匹配的键。

```
V find(K key) {
    int index = hashFunction(key) % size;
    Node* current = table[index];
    while (current) {
        if (current->key == key) {
            return current->value; // 找到对应的值
        }
        current = current->next; // 继续查找
    }
    throw std::runtime_error("Key not found!"); // 未找到抛出异常
}
```

##### 3.3 删除

1. 使用哈希函数查找键的位置。
2. 如果找到，删除该节点。
3. 更新链表或处理开放地址法中的占位符。

```
void remove(K key) {
    int index = hashFunction(key) % size;
    Node* current = table[index];
    Node* prev = nullptr;
    while (current) {
        if (current->key == key) {
            if (prev) {
                prev->next = current->next; // 删除当前节点
            } else {
                table[index] = current->next; // 更新头指针
            }
            delete current; // 释放内存
            return;
        }
        prev = current;
        current = current->next;
    }
    throw std::runtime_error("Key not found!"); // 未找到抛出异常
}
```

#### 4. 性能分析

- 平均时间复杂度：查找、插入和删除操作在理想情况下为 O(1)O(1)O(1)，因为只需通过哈希函数访问数组元素。
- 最坏时间复杂度：
  - 如果所有键通过哈希函数都映射到同一位置（所有冲突），那么时间复杂度会变为 O(n)O(n)O(n)。
  - 通过选择合适的哈希函数和合适的冲突处理方法，可以大大降低发生这种情况的可能性。
- 负载因子（Load Factor）：
  - 负载因子是已存储元素数量与哈希表大小的比率。负载因子过高可能导致性能下降。
  - 通常设置一个阈值（如 0.7），当负载因子超过该阈值时进行扩展。

#### 5. 扩展和收缩

- 扩展：当负载因子超过设定阈值时，创建一个更大的数组（通常是原数组大小的两倍），并重新计算所有元素的索引，插入到新的数组中。

```
void resize() {
    int oldSize = size;
    size *= 2; // 扩展数组大小
    std::vector<Node*> newTable(size, nullptr); // 创建新表

    // 重新哈希所有元素
    for (int i = 0; i < oldSize; ++i) {
        Node* current = table[i];
        while (current) {
            insert(current->key, current->value); // 重新插入
            current = current->next;
        }
    }
    table = std::move(newTable); // 替换旧表
}
```

- 收缩：当负载因子低于设定阈值时，可能会进行收缩，减少数组的大小。

### 7、索引怎么实现？B+树的优势？

索引是数据库和数据结构中用于快速查找数据的机制。常见的索引结构包括哈希索引、B+树索引等。

#### 1. B+树的基本结构

B+树是一种自平衡的树数据结构，通常用于数据库和文件系统中的索引。其主要特性包括：

- 多路搜索树：每个节点可以有多个子节点，通常具有较高的分支因子，能够存储更多的键值。
- 所有键值都在叶子节点：非叶子节点只存储键（用于导航），而叶子节点存储实际数据或指向数据的位置。
- 顺序存储：叶子节点之间通过指针连接，便于范围查询和顺序访问。

#### 2. B+树的结构示意

```
          [30]
         /    \
     [10,20]   [40,50]
       /  |  \     /  \
   [1,2] [15] [35] [45,46]
```

#### 3. B+树的基本操作

##### 3.1 插入操作

1. 从根节点开始，根据键值查找合适的叶子节点。
2. 在叶子节点中插入新键值。
3. 如果叶子节点满了（超过最大容量），则需要分裂该节点，将中间键提升到父节点。
4. 递归处理父节点，直到根节点。

```
void insert(int key) {
    // 查找插入位置并插入
    // 处理节点分裂
}
```

##### 3.2 删除操作

1. 查找要删除的键值所在的叶子节点。
2. 从叶子节点中删除键值。
3. 如果叶子节点的键值少于最小容量，需要从兄弟节点借一个键值或合并节点。
4. 更新父节点的键值。

```
void remove(int key) {
    // 查找并删除键值
    // 处理节点合并或借用
}
```

##### 3.3 查找操作

1. 从根节点开始，根据键值逐层向下查找。
2. 到达叶子节点后返回数据或指针。

```
bool search(int key) {
    // 查找并返回键值
}
```

代码示例：

```c++
#include <iostream>
#include <vector>
#include <algorithm>

#define MAX_KEYS 3 // 最大键值数量
#define MIN_KEYS (MAX_KEYS / 2) // 最小键值数量

// B+树节点
struct BPlusTreeNode {
    bool isLeaf; // 是否为叶子节点
    std::vector<int> keys; // 存储的键值
    std::vector<BPlusTreeNode*> children; // 子节点指针

    BPlusTreeNode(bool leaf) : isLeaf(leaf) {}
};

// B+树类
class BPlusTree {
public:
    BPlusTree() : root(new BPlusTreeNode(true)) {}

    void insert(int key);
    void remove(int key);
    bool search(int key);

private:
    BPlusTreeNode* root;

    // 辅助函数
    void insertNonFull(BPlusTreeNode* node, int key);
    void splitChild(BPlusTreeNode* parent, int index, BPlusTreeNode* child);
    BPlusTreeNode* searchNode(BPlusTreeNode* node, int key);
    void removeFromLeaf(BPlusTreeNode* node, int index);
    void removeFromNonLeaf(BPlusTreeNode* node, int index);
    int getPredecessor(BPlusTreeNode* node, int index);
    int getSuccessor(BPlusTreeNode* node, int index);
    void fill(BPlusTreeNode* node, int index);
    void borrowFromPrev(BPlusTreeNode* node, int index);
    void borrowFromNext(BPlusTreeNode* node, int index);
    void merge(BPlusTreeNode* node, int index);
};

void BPlusTree::insert(int key) {
    if (root->keys.size() == MAX_KEYS) {
        BPlusTreeNode* newRoot = new BPlusTreeNode(false);
        newRoot->children.push_back(root);
        splitChild(newRoot, 0, root);
        root = newRoot;
    }
    insertNonFull(root, key);
}

void BPlusTree::insertNonFull(BPlusTreeNode* node, int key) {
    int i = node->keys.size() - 1;
    if (node->isLeaf) {
        node->keys.push_back(0); // 增加一个位置
        while (i >= 0 && key < node->keys[i]) {
            node->keys[i + 1] = node->keys[i];
            i--;
        }
        node->keys[i + 1] = key;
    } else {
        while (i >= 0 && key < node->keys[i]) {
            i--;
        }
        i++;
        if (node->children[i]->keys.size() == MAX_KEYS) {
            splitChild(node, i, node->children[i]);
            if (key > node->keys[i]) {
                i++;
            }
        }
        insertNonFull(node->children[i], key);
    }
}

void BPlusTree::splitChild(BPlusTreeNode* parent, int index, BPlusTreeNode* child) {
    BPlusTreeNode* newChild = new BPlusTreeNode(child->isLeaf);
    for (int j = 0; j < MIN_KEYS; j++) {
        newChild->keys.push_back(child->keys[j + MIN_KEYS + 1]);
    }
    if (!child->isLeaf) {
        for (int j = 0; j < MIN_KEYS + 1; j++) {
            newChild->children.push_back(child->children[j + MIN_KEYS + 1]);
        }
    }
    parent->children.insert(parent->children.begin() + index + 1, newChild);
    parent->keys.insert(parent->keys.begin() + index, child->keys[MIN_KEYS]);
    child->keys.resize(MIN_KEYS);
}

bool BPlusTree::search(int key) {
    BPlusTreeNode* resultNode = searchNode(root, key);
    return resultNode != nullptr;
}

BPlusTreeNode* BPlusTree::searchNode(BPlusTreeNode* node, int key) {
    int i = 0;
    while (i < node->keys.size() && key > node->keys[i]) {
        i++;
    }
    if (i < node->keys.size() && key == node->keys[i]) {
        return node; // 找到节点
    }
    if (node->isLeaf) {
        return nullptr; // 叶子节点未找到
    }
    return searchNode(node->children[i], key); // 递归查找
}

// 删除操作的实现
void BPlusTree::remove(int key) {
    // 删除操作需要更复杂的实现，这里简化处理
    std::cout << "删除操作尚未实现。" << std::endl;
}

int main() {
    BPlusTree bpt;
    bpt.insert(10);
    bpt.insert(20);
    bpt.insert(5);
    bpt.insert(6);
    bpt.insert(12);
    bpt.insert(30);
    bpt.insert(7);
    bpt.insert(17);

    std::cout << "查找 10: " << (bpt.search(10) ? "找到" : "未找到") << std::endl;
    std::cout << "查找 15: " << (bpt.search(15) ? "找到" : "未找到") << std::endl;

    return 0;
}
```

#### B+树的优势

B+树在数据库索引中具有显著的优势，主要体现在以下几个方面：

##### 1. 高效的范围查询

由于所有键值都在叶子节点，且叶子节点通过指针连接，B+树可以非常高效地支持范围查询（如查找某个范围内的所有值），只需遍历叶子节点。

##### 2. 节点存储效率

B+树的非叶子节点只存储键值，而不存储数据指针，因而可以容纳更多的键值。这种设计减少了树的高度，从而提高了查找效率。

##### 3. 自平衡特性

B+树保持自平衡特性，即在插入和删除操作时，能够自动调整结构，以确保树的高度最小。这样可以保持 O(log n) 的查找效率。

##### 4. 适合磁盘存储

B+树的节点通常较大，适合与磁盘块对齐，从而减少磁盘 I/O 操作次数。这使得 B+树在处理大量数据时表现更优，尤其是在数据库应用中。

##### 5. 适用于并发操作

B+树支持并发访问，多个线程可以同时进行插入、删除和查找操作，提高了数据库的性能。

### 8、手撕：Leetcode 91：解码方法

#### 思路

1. 动态规划数组：创建一个动态规划数组 `dp`，其中 `dp[i]` 表示前 `i` 个字符的解码方法总数。需要计算 `dp[n]`，其中 `n` 是字符串 `s` 的长度。

2. 初始化：

   - `dp[0] = 1`：表示空字符串有一种解码方式。

   - `dp[1]`：如果字符串的第一个字符不为 '0'，则 `dp[1] = 1`；否则，`dp[1] = 0`。

3. 状态转移：

   对于每个字符 `s[i-1]`（从 `i=2` 开始），考虑以下两种情况：

   - 单字符解码：如果 `s[i-1]` 不为 '0'，则 `dp[i] += dp[i-1]`。

   - 双字符解码：如果 `s[i-2]` 和 `s[i-1]` 组成的数字在 `10` 到 `26` 的范围内，说明可以解码成一个字母，则 `dp[i] += dp[i-2]`。

4. 返回结果：最后返回 `dp[n]`，即整个字符串的解码方式总数。

#### 参考代码

```
#include <iostream>
#include <vector>
#include <string>

using namespace std;

int numDecodings(const string& s) {
    int n = s.size();
    
    // 边界情况处理
    if (n == 0 || s[0] == '0') {
        return 0; // 不能以 '0' 开头
    }

    // 动态规划数组
    vector<int> dp(n + 1);
    dp[0] = 1; // 空字符串有一种解码方式
    dp[1] = s[0] != '0' ? 1 : 0; // 非空且首字符不为 '0'

    for (int i = 2; i <= n; ++i) {
        // 单字符解码
        if (s[i - 1] != '0') {
            dp[i] += dp[i - 1];
        }
        
        // 双字符解码
        int twoDigit = (s[i - 2] - '0') * 10 + (s[i - 1] - '0'); // 将两个字符转为数字
        if (twoDigit >= 10 && twoDigit <= 26) {
            dp[i] += dp[i - 2];
        }
    }

    return dp[n]; // 返回总的解码方式
}

int main() {
    string s;

    // 输入字符串
    cout << "请输入只含数字的非空字符串：";
    cin >> s;

    // 计算解码方法总数
    int result = numDecodings(s);
    cout << "解码方法总数: " << result << endl;

    return 0;
}
```

