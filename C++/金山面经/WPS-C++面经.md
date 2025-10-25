# WPS -C++面经

> 来源：https://www.nowcoder.com/feed/main/detail/14b7ca04419f4274ad239381fb378bab

### 1、tcp三次握手

在建立连接之前，Client处于CLOSED状态，而Server处于LISTEN的状态。

- 第一次握手：客户端主动给服务端发送一个SYN报文，并携带自己的初始化序列号一起发送给服务端。此时客户端处于一个SYN_SEND的状态。
- 第二次握手：服务端收到客户端发来的SYN报文之后，就会以自己的SYN报文作为应答，然后将自己的初始化序列号发送给客户端，并且会将客户端的初始化序列号+1作为自己的ACK值发送给客户端，以表示自己已经收到了客户端的SYN报文。此时服务端处于一个SYN_RECV的状态。
- 第三次握手：客户端收到服务端发来的SYN报文之后，会把服务端的初始化序列号+1作为ACK值发送给服务端，用来表示自己已经收到了服务端发来的SYN报文。此时客户端处于一个ESTABLISHED的状态。

### 2、tcp传输数据过程中会粘包问题，解释一下为什么会发生，解决方案？

TCP传输数据过程中会出现粘包问题的主要原因是TCP是面向流的协议，数据被划分为小的数据块，而底层的网络则是面向数据包的。这就导致了在传输过程中，数据块的边界和数据包的边界之间可能不一致，从而发生粘包问题。具体来说，有以下两个主要原因：

1. TCP缓冲区： 发送方在发送数据时，TCP会将数据划分为合适大小的数据块，然后将这些数据块放入TCP缓冲区。接收方从TCP缓冲区中读取数据，但它并不知道发送方在发送时划分的数据块边界。
2. 网络传输： 数据在网络上传输时，可能会因为网络拥塞、路由器缓冲区满等原因导致数据包的延迟、重新排序或合并。

解决TCP粘包问题的常见方法有：

1. 消息定界： 在数据包中添加消息的长度信息，接收方通过消息长度来判断消息的边界。常见的方式有在消息头部加入消息长度字段。
2. 使用特殊字符： 在消息末尾添加特殊字符作为分隔符，接收方根据特殊字符来切分消息。这种方式需要确保特殊字符不会在消息内容中出现。
3. 固定长度： 规定每个数据包的固定长度，不足部分用空格或其他填充字符补齐。
4. 使用回车换行符： 在文本协议中，可以使用回车换行符（\r\n）来作为消息的结束符，接收方根据这个符号来切分消息。
5. 应用层协议： 在应用层设计自定义的协议，通过在消息头部包含长度字段或其他标识信息来标明消息的边界。

### 3、tcp四次挥手，为什么多一次？

1. 客户端向服务端发送一个报文FIN为1，序号为u然后进入FIN-WAIT1状态。
2. 服务端向客户端发送确认报文序号为v，确认序号为u+1然后进入CLOSE-WAIT状态。
3. 客户端收到服务端发回的确认报文之后进入FIN-WAIT2状态此时客户端连接已经关闭客户端无法向服务端传送数据。
4. 然后服务端被动关闭它向客户端发送一个FIN为1的报文段要求释放服务端到客户端的连接。进入LAST-ACK等待客户端发送最后一个ACK报文。
5. 客户端发送最后一次挥手确认报文然后进行closed，服务端直接CLOSED。
6. 客户端要等待2MSL才CLOSED。

为什么多一次？

当Client端发出FIN报文段时，只可以说明Client端没有数据发送了，但是Client端还可以从Server端接收数据。

当Server端发送ACK报文段时，只可以说明它知道了Client端没有数据发送了，但是Server端还可以发送数据。

当Server端发送FIN报文段时，表示Server端也没有数据发送了。

当Client端接收到后会返回ACK，之后彼此就会愉快的中断这次TCP连接。

第二次和第三次是需要分开发送的。

### 4、leetcode  141 环形链表

思路：

1. 使用两个指针，分别为快指针 `fast` 和慢指针 `slow`，初始时都指向链表的头节点 `head`。
2. 快指针每次向前移动两步，慢指针每次向前移动一步。
3. 如果链表中存在环，快指针和慢指针最终会相遇；如果不存在环，快指针会先到达链表的末尾，此时可以判断链表无环。
4. 在相遇的时候，可以进一步判断环的起始位置。将其中一个指针重新指向头节点，然后两个指针每次向前移动一步，再次相遇的位置就是环的起始位置。

参考代码：

```C++
#include <iostream>

struct ListNode {
    int val;
    ListNode *next;
    ListNode(int x) : val(x), next(nullptr) {}
};

bool hasCycle(ListNode *head) {
    // 初始时，快指针和慢指针都指向头节点
    ListNode *slow = head;
    ListNode *fast = head;

    // 使用快慢指针判断是否有环
    while (fast != nullptr && fast->next != nullptr) {
        slow = slow->next;         // 慢指针每次向前移动一步
        fast = fast->next->next;   // 快指针每次向前移动两步

        // 如果快指针和慢指针相遇，说明有环
        if (slow == fast) {
            // 重新将一个指针指向头节点
            fast = head;

            // 两个指针每次向前移动一步，再次相遇的位置即为环的起始位置
            while (slow != fast) {
                slow = slow->next;
                fast = fast->next;
            }

            // 返回 true 表示链表有环
            return true;
        }
    }

    // 如果快指针先到达链表末尾，说明无环
    return false;
}

int main() {
    // 创建链表：1 -> 2 -> 3 -> 4 -> 5 -> 2（形成环，尾部连接到索引为 1 的位置）
    ListNode *head = new ListNode(1);
    head->next = new ListNode(2);
    head->next->next = new ListNode(3);
    head->next->next->next = new ListNode(4);
    head->next->next->next->next = new ListNode(5);
    head->next->next->next->next->next = head->next;  // 形成环

    // 判断链表是否有环
    bool result = hasCycle(head);

    // 输出结果
    std::cout << "链表是否有环: " << (result ? "是" : "否") << std::endl;

    return 0;
}
```

### 5、leetcode  236二叉树的最近公共祖先 递归法

思路：

1. 递归遍历： 从根节点开始递归遍历二叉树。
2. 判断条件： 在递归过程中，对于当前节点，分别判断左右子树是否包含目标节点 p 和 q。
3. 递归终止条件： 当当前节点为空（遍历到叶子节点的子节点）或等于目标节点 p 或 q 时，直接返回当前节点。
4. 递归调用： 递归调用左右子树，并将返回值存储在 left 和 right 中。
5. 判断返回值： 根据 left 和 right 的情况判断最近公共祖先的位置。

参考代码：

```C++
#include <iostream>

struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
    // 递归终止条件
    if (root == nullptr || root == p || root == q) {
        return root;
    }

    // 在左子树中寻找 p 和 q 的最近公共祖先
    TreeNode* left = lowestCommonAncestor(root->left, p, q);

    // 在右子树中寻找 p 和 q 的最近公共祖先
    TreeNode* right = lowestCommonAncestor(root->right, p, q);

    // 判断返回值，确定最近公共祖先的位置
    if (left != nullptr && right != nullptr) {
        return root;  // 如果左右子树分别包含 p 和 q，当前节点即为最近公共祖先
    } else if (left != nullptr) {
        return left;  // 如果只在左子树找到 p 或 q，返回左子树的结果
    } else {
        return right; // 如果只在右子树找到 p 或 q，返回右子树的结果
    }
}

int main() {
    // 创建二叉树
    TreeNode* root = new TreeNode(3);
    root->left = new TreeNode(5);
    root->right = new TreeNode(1);
    root->left->left = new TreeNode(6);
    root->left->right = new TreeNode(2);
    root->right->left = new TreeNode(0);
    root->right->right = new TreeNode(8);
    root->left->right->left = new TreeNode(7);
    root->left->right->right = new TreeNode(4);

    // 选择两个节点
    TreeNode* p = root->left;
    TreeNode* q = root->right;

    // 寻找最近公共祖先
    TreeNode* result = lowestCommonAncestor(root, p, q);

    // 输出结果
    if (result != nullptr) {
        std::cout << "最近公共祖先的值为: " << result->val << std::endl;
    } else {
        std::cout << "未找到最近公共祖先" << std::endl;
    }

    return 0;
}
```

非递归

思路：

1. 使用迭代方式遍历二叉树，不断更新当前节点的父节点关系。
2. 维护一个存储每个节点父节点的映射表。
3. 从根节点开始，一直遍历到找到目标节点 p 和 q。
4. 使用集合或数组记录访问过的节点，当找到目标节点时，回溯得到目标节点的所有祖先节点。
5. 利用集合或数组找到最近公共祖先。

参考代码：

```C++
#include <iostream>
#include <unordered_map>
#include <unordered_set>
#include <stack>

struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
    std::unordered_map<TreeNode*, TreeNode*> parent;  // 存储节点的父节点映射表
    std::unordered_set<TreeNode*> visited;  // 存储访问过的节点
    std::stack<TreeNode*> stack;  // 辅助栈，用于迭代遍历

    // 从根节点开始迭代遍历二叉树
    stack.push(root);
    while (!parent.count(p) || !parent.count(q)) {
        TreeNode* node = stack.top();
        stack.pop();

        if (node->left) {
            parent[node->left] = node;
            stack.push(node->left);
        }

        if (node->right) {
            parent[node->right] = node;
            stack.push(node->right);
        }
    }

    // 回溯找到目标节点 p 的所有祖先节点
    while (p) {
        visited.insert(p);
        p = parent[p];
    }

    // 找到第一个在访问集合中的祖先节点 q 即为最近公共祖先
    while (!visited.count(q)) {
        q = parent[q];
    }

    return q;
}

int main() {
    // 创建二叉树
    TreeNode* root = new TreeNode(3);
    root->left = new TreeNode(5);
    root->right = new TreeNode(1);
    root->left->left = new TreeNode(6);
    root->left->right = new TreeNode(2);
    root->right->left = new TreeNode(0);
    root->right->right = new TreeNode(8);
    root->left->right->left = new TreeNode(7);
    root->left->right->right = new TreeNode(4);

    // 选择两个节点
    TreeNode* p = root->left;
    TreeNode* q = root->right;

    // 寻找最近公共祖先
    TreeNode* result = lowestCommonAncestor(root, p, q);

    // 输出结果
    if (result != nullptr) {
        std::cout << "最近公共祖先的值为: " << result->val << std::endl;
    } else {
        std::cout << "未找到最近公共祖先" << std::endl;
    }

    return 0;
}
```

### 6、hashmap出现哈希冲突，怎么解决？

1. 开放地址法（Open Addressing）：

在开放地址法中，当一个位置发生冲突时，就去寻找下一个空的位置，直到找到一个空位置或者遍历了整个哈希表。开放地址法有几种常见的处理冲突的方法：

- 线性探测（Linear Probing）：发生冲突时，顺序地查找下一个空槽。
- 二次探测（Quadratic Probing）：发生冲突时，使用二次方程来查找下一个空槽。
- 双重哈希（Double Hashing）：使用第二个哈希函数来计算下一个槽的位置。

2. 链地址法（Chaining）：

在链地址法中，哈希表的每个槽对应一个链表，当发生冲突时，将冲突的元素插入到对应位置的链表中。链地址法的解决冲突的方式主要有两种：

- 头插法：新元素插入到链表的头部。
- 尾插法：新元素插入到链表的尾部。

3. 其他方法：

- 再哈希（Rehashing）：当哈希表中元素达到一定阈值时，可以重新调整哈希表的大小，选择一个新的哈希函数，将所有元素重新插入新的哈希表中，以降低冲突的概率。

如何选择解决冲突的方法：

- 性能考虑：不同的解决冲突方法在不同的场景下性能表现可能有差异，需要综合考虑插入、删除、查找等操作的性能。
- 简单性：有些方法实现简单，容易理解和维护，这在一些应用场景中可能更为合适。

### 7、开链法出现聚集，最坏时间复杂度，怎么解决

开链法是一种解决哈希冲突的方法，其中每个哈希桶都包含一个链表，当发生冲突时，新的元素会被添加到相应的链表中。然而，如果哈希函数不够均匀，可能会导致某些链表变得过长，从而降低查找效率。 聚集（Cluster）是指在哈希表中出现了较长的链表，这可能导致查找效率的下降。为了解决这个问题，可以采取以下措施：

1. 重新哈希（Rehashing）：当聚集过多时，可以考虑增大哈希表的大小，然后重新哈希已有的元素。这样可以使每个桶中的元素分布更加均匀。
2. 动态调整哈希表大小：通过动态调整哈希表的大小，可以在运行时根据当前负载因子（已使用桶数与总桶数的比率）来调整表的大小。这有助于避免聚集的出现。
3. 链表优化：可以考虑使用更高效的链表结构，如跳表或平衡树，而不是传统的单链表。这样可以提高在链表中查找元素的效率。
4. 选择更好的哈希函数：一个好的哈希函数可以帮助均匀地分布元素，减少冲突的发生。

### 8、百万数据，找出TOP10？

要找出一个数据集中的 TOP 10 元素，可以使用堆（Heap）数据结构。

具体步骤：

1. 建立最小堆（Min Heap）：遍历数据集的前 10 个元素，将它们构建成一个最小堆。
2. 遍历剩余元素：对于数据集中的其他元素，如果该元素比最小堆的堆顶元素大，就将堆顶元素替换为当前元素，然后调整堆，确保最小堆的性质仍然保持。
3. 最终堆中的元素即为 TOP 10：遍历完整个数据集后，最小堆中的元素即为 TOP 10。

这种方法的时间复杂度为 O(n * log(k))，其中 n 是数据集的大小，k 是 TOP K 的大小。在这里，k 是 10，因此时间复杂度可以看作是 O(n)。

给个小demo：

```C++
#include <iostream>
#include <vector>
#include <algorithm>
#include <queue>

std::vector<int> findTopK(const std::vector<int>& data, int k) {
    // 使用最小堆
    std::priority_queue<int, std::vector<int>, std::greater<int>> minHeap;

    // 将前k个元素插入最小堆
    for (int i = 0; i < k; ++i) {
        minHeap.push(data[i]);
    }

    // 遍历剩余元素，维护最小堆
    for (int i = k; i < data.size(); ++i) {
        if (data[i] > minHeap.top()) {
            minHeap.pop();
            minHeap.push(data[i]);
        }
    }

    // 将最小堆中的元素取出
    std::vector<int> result;
    while (!minHeap.empty()) {
        result.push_back(minHeap.top());
        minHeap.pop();
    }

    // 由于是最小堆，需要翻转结果
    std::reverse(result.begin(), result.end());

    return result;
}

int main() {
    std::vector<int> data = {3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5};
    int k = 10;

    std::vector<int> topK = findTopK(data, k);

    std::cout << "Top " << k << " elements: ";
    for (int num : topK) {
        std::cout << num << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

### 9、快排时间复杂度 给个表供大家参考：

https://vnkshn64w3.feishu.cn/sync/EFxHdIlH9sd54ybyXnFcCUU9nMf

### 10、怎么评判算法的好坏？

给出下边几点供大家参考：

1. 时间复杂度： 算法的时间复杂度是指执行算法所需的计算工作量，通常用大 O 表示。时间复杂度越低越好，表示算法在处理数据量增大时所需的时间增长越慢。
2. 空间复杂度： 算法的空间复杂度是指执行算法所需的额外空间，通常也用大 O 表示。空间复杂度越低越好，表示算法在使用内存方面更加高效。
3. 稳定性： 对于排序算法而言，如果两个相等的元素在排序后的序列中相对位置不变，这样的排序算法就是稳定的。有些应用场景对稳定性要求较高。
4. 可读性： 算法的可读性是指算法的代码是否容易理解，是否符合编码规范，以及是否易于维护。清晰、简洁的代码有助于团队协作和后期维护。
5. 实现难度： 算法的实现难度影响了开发者的开发效率。有时候，一个实现较为简单但性能较差的算法可能比一个性能优越但实现复杂的算法更为合适。
6. 适用场景： 不同的算法可能在不同的应用场景下表现优异。选择算法时需要考虑问题的规模、特性以及对算法性能的具体要求。
7. 稳定性和收敛性： 对于一些迭代算法，需要考虑算法是否能够在有限的步骤内收敛到期望的结果，以及在输入变化时是否能够保持稳定性。

### 11、vector怎么扩容？

在 C++ 的标准库中，`std::vector` 是一个动态数组，其扩容是通过重新分配内存来实现的。当 `std::vector` 的元素数量达到当前分配的内存大小时，系统会为 `std::vector` 分配一块更大的内存，并将原来的元素复制到新的内存中。这个过程中原来的内存会被释放。

以下是 `std::vector` 扩容的基本步骤：

1. 分配新的内存： 当 `std::vector` 中的元素个数达到当前分配的内存大小时，需要分配一块新的内存。新的内存大小通常是当前内存大小的两倍，这样做是为了保证 `std::vector` 的操作复杂度为平摊 O(1)。
2. 将元素复制到新内存： 将原来的元素逐个复制到新分配的内存中。
3. 释放旧内存： 释放原来的内存空间。

### 12、介绍一下reactor模式？

Reactor 模式是一种用于处理事件驱动应用程序中并发请求的设计模式。它主要涉及到两个组件：事件源和事件处理器。

1. 事件源(Event Source):事件源是产生事件的对象，可以是网络套接字、文件描述符、定时器等。这些事件源通常被注册到事件处理器中，当事件源上发生事件时，事件处理器会得到通知。
2. 事件处理器(Event Handler):事件处理器是负责响应和处理事件的组件。它包含有关如何处理事件的逻辑。在 Reactor模式中，事件处理器需要注册到一个中央调度器（也称为 Reactor），以便随时接收和处理事件。

Reactor模式的基本流程：

- 多个事件源注册到 Reactor 中。
- Reactor 不断地检查这些事件源，看它们是否有事件发生。
- 一旦事件发生，Reactor 负责分发事件给对应的事件处理器进行处理。

Reactor模式有两种实现方式：

1. 同步（同步事件处理模型）：Reactor 在单个线程中运行，并负责事件的检测和事件分发。事件处理器在 Reactor 中同步执行。这种方式适用于事件处理逻辑简单的场景。
2. 异步（多线程/多进程事件处理模型）：Reactor 可以在多个线程或多个进程中运行，事件源和事件处理器可能在不同的线程或进程中。这种方式适用于需要处理复杂逻辑或处理大量并发请求的场景。

Reactor 模式的优点包括：

- 可扩展性：可以通过添加新的事件源和事件处理器来轻松扩展系统。
- 并发处理：适用于处理大量并发请求的场景，提供了一种高效的事件驱动的编程模型。

### 13、muduo网络库内存拷贝？

Muduo 是一个基于 Reactor 模式的 C++ 网络库，设计简洁高效，提供了事件驱动的编程模型，用于构建高性能的网络服务器。

1. 零拷贝（Zero-Copy）： Muduo利用操作系统提供的零拷贝技术，避免了在用户态和内核态之间的多次数据拷贝。在网络传输过程中，数据可以从应用程序的缓冲区直接传递给网络协议栈，而无需中间的用户态缓冲区。这减少了不必要的内存拷贝，提高了性能。
2. 缓冲区管理： Muduo中使用了一个高效的缓冲区管理机制，通过预分配一定数量的固定大小的缓冲区（Buffer），在需要时进行缓冲区的借用和归还。这减少了频繁的内存分配和释放操作，提高了内存使用效率。
3. 文件描述符传递： 在 Muduo 中，使用了文件描述符传递的方式，可以将一个已打开的文件描述符从一个进程传递给另一个进程，避免了通过套接字传递数据时的数据拷贝。

### 14、深拷贝浅拷贝？

1. 浅拷贝：
   - 浅拷贝是指将一个对象的值复制到另一个对象，但是只复制对象的基本数据类型的成员变量，而不复制引用类型的成员变量。
   - 对于引用类型的成员变量，浅拷贝后，原对象和拷贝对象仍然指向相同的引用，因此它们共享相同的引用对象。如果修改了其中一个对象的引用对象，另一个对象也会受到影响。
   - 浅拷贝通常通过拷贝构造函数或赋值运算符实现。
2. 深拷贝：
   - 深拷贝是指将一个对象的值复制到另一个对象，并且递归复制对象的所有引用类型的成员变量，而不是简单地复制引用。
   - 深拷贝后，原对象和拷贝对象拥有相同的值，但是它们所引用的对象是独立的，修改其中一个对象的引用对象不会影响另一个对象。
   - 深拷贝通常需要自定义实现，可以通过递归复制对象的所有成员变量来实现。

### 15、消息队列的使用？

消息队列是一种进程间通信的方式，它可以在不同的进程之间传递消息，实现进程之间的解耦。消息队列的基本使用包括以下几个方面：

1. 创建消息队列：

   在消息队列的使用之前，需要先创建消息队列。不同的操作系统和编程语言提供了不同的消息队列接口，通常有相应的创建函数或工具。

2. 发送消息：

   通过消息队列将消息发送到队列中。消息可以是结构化的数据，也可以是简单的文本。发送消息的进程称为生产者。

3. 接收消息：

   从消息队列中接收消息。接收消息的进程称为消费者。通常，多个进程可以同时从同一个消息队列中接收消息，实现消息的广播。

4. 消息队列的特性：

   消息队列通常具有一些特性，如容量限制、优先级等。生产者和消费者需要遵守相应的规则，以确保消息的正确发送和接收。

5. 错误处理：

   在使用消息队列时，需要考虑错误处理机制，例如处理发送消息失败、接收消息失败等情况。

给个demo：

```C++
#include <iostream>
#include <queue>
#include <mutex>
#include <condition_variable>
#include <thread>

class MessageQueue {
public:
    // 向队列中发送消息
    void sendMessage(const std::string& message) {
        std::unique_lock<std::mutex> lock(mutex_);
        messages_.push(message);
        condition_.notify_one();
    }

    // 从队列中接收消息
    std::string receiveMessage() {
        std::unique_lock<std::mutex> lock(mutex_);
        condition_.wait(lock, [this] { return !messages_.empty(); });

        std::string message = messages_.front();
        messages_.pop();
        return message;
    }

private:
    std::queue<std::string> messages_;
    std::mutex mutex_;
    std::condition_variable condition_;
};

// 生产者线程函数
void producer(MessageQueue& mq) {
    for (int i = 0; i < 5; ++i) {
        std::string message = "Message " + std::to_string(i);
        mq.sendMessage(message);
        std::this_thread::sleep_for(std::chrono::seconds(1));
    }
}

// 消费者线程函数
void consumer(MessageQueue& mq) {
    for (int i = 0; i < 5; ++i) {
        std::string message = mq.receiveMessage();
        std::cout << "Received: " << message << std::endl;
    }
}

int main() {
    MessageQueue messageQueue;

    // 创建生产者和消费者线程
    std::thread producerThread(producer, std::ref(messageQueue));
    std::thread consumerThread(consumer, std::ref(messageQueue));

    // 等待线程执行完成
    producerThread.join();
    consumerThread.join();

    return 0;
}
```