# B站 AI开发一二面面经

## 一面

来源：https://www.nowcoder.com/discuss/559453429614075904?sourceSSR=users

1、自我介绍

2、c++的左值和右值

左值：

1. 定义：左值表示一个具体的内存位置。它是一个标识符，可以放在赋值语句的左边。

2. 特点：左值有名字，有确定的内存地址，可以取地址。通常是变量、对象、数组元素等。

3. 示例：

   ```
   int x = 10;  // x是左值
   int* ptr = &x;  // &x是左值，表示x的地址
   ```

右值：

1. 定义：右值表示临时的、无法取地址的值。它通常是在表达式求值后产生的中间结果。

2. 特点右值没有名字，可能是常量、临时对象、表达式的结果等。

3. 示例：

   ```
   int sum = 5 + 3;  // 5 + 3是右值，表示一个临时的中间结果
   int&& ref = 5;    // 5是右值，可以通过右值引用引用
   ```

C++11引入的右值引用：

右值引用是在C++11中引入的，允许我们显式地操作右值。右值引用使用 `&&` 符号表示。

```
int&& rvalueRef = 42;  // 右值引用绑定到右值
```

右值引用的主要用途包括移动语义和完美转发。

移动语义：

移动语义允许我们将资源（如动态分配的内存）从一个对象“移动”到另一个对象，而不是进行昂贵的拷贝操作。这对于提高性能和避免不必要的内存拷贝很有帮助。

```
std::vector<int> getVector() {
    std::vector<int> temp = {1, 2, 3};
    return temp;  // 返回右值
}

std::vector<int> myVector = getVector();  // 使用移动语义
```

3、网络编程中ET和LT两种触发方式的区别(不会)

LT：

1. 触发条件：当文件描述符就绪时，LT 模式会在整个描述符就绪的时间范围内一直触发通知。
2. 特点：LT 是默认的触发方式，也称为水平触发。一旦文件描述符变得可读或可写，它就会触发通知，即使在通知之间这个文件描述符仍然是可读或可写状态。
3. 适用场景：适用于使用阻塞 I/O 模型的情况，不需要关心文件描述符何时变为非阻塞状态。

ET：

1. 触发条件：当文件描述符从非阻塞状态变为可读或可写时，ET 模式才会触发通知。
2. 特点：ET 是边缘触发。它只在文件描述符状态变化的瞬间触发通知，而不是在整个描述符就绪的时间范围内触发。
3. 适用场景：适用于使用非阻塞 I/O 模型的情况，需要及时响应文件描述符状态的变化。

区别总结：

- LT 模式触发条件宽松：只要文件描述符是可读或可写状态，就会触发通知。
- ET 模式触发条件严格：只有在文件描述符状态发生变化的瞬间才会触发通知。
- LT 模式对应阻塞 I/O，ET 模式对应非阻塞 I/O：ET 模式更适用于非阻塞 I/O 模型，能够更及时地响应文件描述符状态的变化。

4、计算机网络，为什么要有time_wait状态

`TIME_WAIT` 状态是 TCP 协议中的一个状态，出现在连接的一端主动关闭连接后。在这个状态中，连接的一方（通常是客户端）等待一段时间，确保网络中的所有数据包都能够正常结束。

`TIME_WAIT` 状态的存在有几个主要原因：

1. 确保数据完整性：当一方关闭连接后，它可能还有一些未发送的数据包在网络中传输。为了确保这些数据包被对方正确接收，连接的一方会进入 `TIME_WAIT` 状态，等待一段时间，以便对方有足够的时间接收并确认所有数据。
2. 防止旧的数据包干扰新连接：在网络中，数据包的传输可能会受到网络延迟、乱序等影响。如果不等待一段时间，而直接关闭连接并尝试建立新连接，可能会导致旧的数据包干扰新连接的数据流。
3. 处理网络中的延迟和乱序：在 `TIME_WAIT` 状态中，连接的一方会等待两倍的最大报文生存时间（2MSL，Maximum Segment Lifetime）。这个时间段内，网络中的数据包有足够的时间被传输和接收，从而保证连接的完整性。
4. 防止连接的冲突：避免在网络中出现连接的冲突，确保一个连接彻底关闭后，再尝试创建一个相同的连接。

需要注意的是，`TIME_WAIT` 状态并非一定会导致问题，而是一种为了保证连接的正确关闭和数据的完整性而设置的一种状态。在实际网络中，通常通过调整操作系统的 TCP 参数，例如减小 `TIME_WAIT` 的时间，来影响 `TIME_WAIT` 的行为。

5、介绍实习

6、代码题，leetcode 124.二叉树的最大路径和 

问题描述：二叉树中的 路径 被定义为一条节点序列，序列中每对相邻节点之间都存在一条边。同一个节点在一条路径序列中 至多出现一次 。该路径至少包含一个节点，且不一定经过根节点。

路径和 是路径中各节点值的总和。

思路及代码：

思路：

1. 深度优先搜索： 从根节点开始递归地遍历每个节点。对于每个节点，计算以该节点为根的子树的最大路径和，分别考虑两种情况：一种是以该节点的左子树为路径的一部分，一种是以该节点的右子树为路径的一部分。同时，更新全局最大路径和。
2. 递归终止条件： 当前节点为null时，返回0，因为空节点对路径和没有影响。
3. 子树最大路径和计算： 递归计算左右子树的最大路径和，如果子树最大路径和为负数，则舍弃子树，否则将子树路径和加到当前节点值上。
4. 更新全局最大路径和：计算包含当前节点的路径和，如果该路径和大于全局最大路径和，则更新全局最大路径和。

参考代码：

```
#include <iostream>
#include <algorithm>

using namespace std;

// 二叉树节点的定义
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class Solution {
public:
    int maxPathSum(TreeNode* root) {
        int maxSum = INT_MIN; // 全局最大路径和初始化为最小整数

        // 调用递归函数
        maxPathSumHelper(root, maxSum);

        return maxSum;
    }

private:
    int maxPathSumHelper(TreeNode* root, int& maxSum) {
        // 递归终止条件：当前节点为null
        if (!root) {
            return 0;
        }

        // 计算左右子树的最大路径和，如果为负数则舍弃
        int leftMax = max(0, maxPathSumHelper(root->left, maxSum));
        int rightMax = max(0, maxPathSumHelper(root->right, maxSum));

        // 计算以当前节点为根的子树的最大路径和
        int currentSum = root->val + leftMax + rightMax;

        // 更新全局最大路径和
        maxSum = max(maxSum, currentSum);

        // 返回以当前节点为根的子树的最大路径和，只能选择左子树或右子树的一条路径
        return root->val + max(leftMax, rightMax);
    }
};

int main() {
    Solution solution;

    // 构建二叉树
    TreeNode* root = new TreeNode(10);
    root->left = new TreeNode(2);
    root->right = new TreeNode(10);
    root->left->left = new TreeNode(20);
    root->left->right = new TreeNode(1);
    root->right->right = new TreeNode(-25);
    root->right->right->left = new TreeNode(3);
    root->right->right->right = new TreeNode(4);

    // 调用函数计算最大路径和
    int result = solution.maxPathSum(root);

    // 输出结果
    cout << "二叉树的最大路径和为: " << result << endl;

    return 0;
}
```

## 二面

1、自我介绍  

2、介绍ky项目

3、介绍C++的三种智能指针

1. `std::shared_ptr`：

   - 原理：`std::shared_ptr`是基于引用计数的智能指针，用于管理动态分配的对象。它维护一个引用计数，当计数为零时，释放对象的内存。

   - 使用场景：适用于多个智能指针需要共享同一块内存的情况。例如，在多个对象之间共享某个资源或数据。

   - ```C++
     std::shared_ptr<int> sharedInt = std::make_shared<int>(42);
     std::shared_ptr<int> anotherSharedInt = sharedInt; // 共享同一块内存
     ```

2. `std::unique_ptr`：

   - 原理：`std::unique_ptr`是独占式智能指针，意味着它独占拥有所管理的对象，当其生命周期结束时，对象会被自动销毁。

   - 使用场景：适用于不需要多个指针共享同一块内存的情况，即单一所有权。通常用于资源管理，例如动态分配的对象或文件句柄。

   - ```C++
     std::unique_ptr<int> uniqueInt = std::make_unique<int>(42);
     // uniqueInt 的所有权是唯一的
     ```

3. `std::weak_ptr`：

   - 原理：`std::weak_ptr`是一种弱引用指针，它不增加引用计数。它通常用于协助`std::shared_ptr`，以避免循环引用问题。

   - 使用场景：适用于协助解决`std::shared_ptr`的循环引用问题，其中多个`shared_ptr`互相引用，导致内存泄漏。

   - ```C++
     std::shared_ptr<int> sharedInt = std::make_shared<int>(42);
     std::weak_ptr<int> weakInt = sharedInt;
     ```

4、weak_ptr如何解决shared_ptr循环引用？

`std::weak_ptr` 用于解决 `std::shared_ptr` 的循环引用问题，避免形成循环引用导致资源无法释放的情况。循环引用通常发生在两个或多个对象之间相互持有 `std::shared_ptr` 的情况下。

给个例子来说明循环引用及如何使用 `std::weak_ptr` 解决问题：

```
#include <memory>
#include <iostream>

class B;  // 提前声明

class A {
public:
    std::shared_ptr<B> b_ptr;

    A() {
        std::cout << "A Constructor\n";
    }

    ~A() {
        std::cout << "A Destructor\n";
    }
};

class B {
public:
    std::shared_ptr<A> a_ptr;

    B() {
        std::cout << "B Constructor\n";
    }

    ~B() {
        std::cout << "B Destructor\n";
    }
};

int main() {
    std::shared_ptr<A> a = std::make_shared<A>();
    std::shared_ptr<B> b = std::make_shared<B>();

    // 形成循环引用
    a->b_ptr = b;
    b->a_ptr = a;

    // 循环引用会导致内存泄漏，对象无法被正确释放

    return 0;
}
```

在上述例子中，`A` 类和 `B` 类相互持有对方的 `std::shared_ptr`，形成了循环引用，导致对象无法正确释放。

下面是如何使用 `std::weak_ptr` 解决这个问题：

```
#include <memory>
#include <iostream>

class B;

class A {
public:
    std::shared_ptr<B> b_ptr;

    A() {
        std::cout << "A Constructor\n";
    }

    ~A() {
        std::cout << "A Destructor\n";
    }
};

class B {
public:
    std::weak_ptr<A> a_ptr;

    B() {
        std::cout << "B Constructor\n";
    }

    ~B() {
        std::cout << "B Destructor\n";
    }
};

int main() {
    std::shared_ptr<A> a = std::make_shared<A>();
    std::shared_ptr<B> b = std::make_shared<B>();

    // 使用 weak_ptr 解决循环引用
    a->b_ptr = b;
    b->a_ptr = a;

    // ...

    return 0;
}
```

通过使用 `std::weak_ptr`，`B` 类现在使用弱引用来持有 `A` 类的 `std::shared_ptr`。这样就避免了形成循环引用，当 `A` 对象或 `B` 对象被释放时，引用计数会正确减少，从而确保对象的正确释放。

5、数组和链表的区别

1. 存储方式：

- 数组：
  - 连续存储：数组在内存中以连续的块存储元素。每个元素占用相同大小的内存。
  - 固定大小：数组的大小在创建时就固定了，不能动态改变。
- 链表：
  - 分散存储：链表中的元素（节点）在内存中分散存储，每个节点包含一个数据元素和一个指向下一个节点的指针。
  - 动态大小：链表的大小可以动态改变，可以在运行时灵活地添加或删除节点。

2. 访问元素：

- 数组：
  - 随机访问：数组支持通过索引直接访问任何元素，具有 O(1) 的随机访问时间。
  - 顺序访问：遍历数组的时间复杂度为 O(n)。
- 链表：
  - 顺序访问：链表只能通过从头节点开始，依次遍历到目标位置，时间复杂度为 O(n)。
  - 非随机访问：链表不支持直接通过索引进行随机访问。

3. 插入和删除操作：

- 数组：
  - 插入和删除较慢：在数组中插入或删除元素可能需要移动大量元素，时间复杂度为 O(n)。
  - 位置固定：数组中元素的位置是固定的，删除一个元素后，需要手动调整数组。
- 链表：
  - 插入和删除较快：在链表中插入或删除元素只需修改节点的指针，时间复杂度为 O(1)。
  - 位置灵活：链表中元素的位置可以动态调整。

4. 空间复杂度：

- 数组：
  - 较小的额外开销：除了存储元素本身外，数组通常不需要额外的指针或节点，空间利用率较高。
- 链表：
  - 额外的指针开销：每个节点需要额外的指针来存储下一个节点的地址，相对于数组有较大的额外开销。

5. 适用场景：

- 数组：
  - 适用于需要随机访问、知道元素索引的情况。
  - 需要固定大小的存储结构。
- 链表：
  - 适用于需要频繁插入和删除操作的情况。
  - 需要动态大小的存储结构。

6、链表的应用场景

- 动态内存分配：链表允许动态地分配内存，可以在运行时灵活地创建和释放节点，适用于那些无法预知大小的情况。
- 插入和删除操作频繁：由于链表在插入和删除操作上具有 O(1) 的时间复杂度，因此在这些操作频繁发生的场景中，链表比数组更为适用。
- 实现队列和栈：链表可以用于实现队列和栈，其中队列和栈的元素在一端插入，另一端删除，链表提供了一种方便的实现方式。
- LRU Cache：链表可以用于实现LRU缓存算法，通过在链表中维护元素的访问顺序，实现对最近使用的元素的快速访问。
- 图的邻接表：在图的表示中，链表可以用于实现邻接表，每个顶点的邻接点列表就是一个链表，这种表示方式对于稀疏图非常高效。
- 大整数运算：对于超过机器数位数的大整数运算，链表可以用于存储每一位的数值，便于高精度计算。
- 嵌套数据结构：在某些数据结构的设计中，链表可以被嵌套使用，例如树结构的实现中，每个节点可以包含指向子节点的链表。
- 模拟文件系统：文件系统中的目录结构可以使用链表来表示，每个目录节点包含指向子目录或文件的链表。

7、调用vector的push_back方法会发生哪些事情？

1. 元素添加：`push_back`用于在vector的末尾添加一个新元素。新元素会被拷贝或移动至vector的尾部。
2. 内存分配：如果vector的内部存储空间不足以容纳新元素，`push_back`可能会触发内存重新分配。这通常会导致分配更大的内存块，然后将原有的元素拷贝（或移动）到新的内存空间中。
3. 元素拷贝或移动：如果新元素的类型是可拷贝的（copyable），那么新元素会被拷贝到vector的尾部。如果新元素是可移动的（movable），则可能会发生移动语义，避免了不必要的拷贝操作。
4. 增加size：`push_back`会增加vector的大小（size）。

给个简单的示例：

```
#include <vector>

int main() {
    std::vector<int> myVector;

    // 调用push_back添加元素
    myVector.push_back(42);

    return 0;
}
```

8、vector的resize和reserve的作用?

1. `resize` 函数：

`resize` 用于改变 `std::vector` 的大小，即容器中元素的数量。如果新的大小比当前大小大，将在尾部插入默认构造的元素；如果新的大小比当前大小小，将截断 vector。

```
#include <vector>

int main() {
    std::vector<int> myVector;

    // 改变vector的大小为5，多出的部分用默认构造的元素填充
    myVector.resize(5);

    return 0;
}
```

2. `reserve` 函数：

`reserve` 用于预留容器的存储空间，但并不改变容器的大小。这样可以在事先知道容器可能需要的存储空间的情况下，避免不必要的内存重新分配，提高效率。

```
#include <vector>

int main() {
    std::vector<int> myVector;

    // 预留容器的存储空间，但容器的大小仍为0
    myVector.reserve(100);

    return 0;
}
```

总结一下：

- `resize` 用于改变容器的大小，并且可以在改变大小的同时指定要插入的元素。
- `reserve` 用于预留容器的存储空间，但并不改变容器的大小。它可以用于避免频繁的内存重新分配，提高性能。

9、代码题：求一个整数的开方根

这道题是一个二分查找的应用实例。

参考代码：

```
#include <iostream>

// 函数定义：计算整数的开方根
int sqrtBinarySearch(int x) {
    // 处理特殊情况：当 x 小于等于 1 时，其开方根即为 x 本身
    if (x <= 1) {
        return x;
    }

    // 定义二分查找的上下界
    int left = 1;
    int right = x;
    int result = 0;

    // 二分查找
    while (left <= right) {
        int mid = left + (right - left) / 2;

        // 判断 mid 的平方与 x 的大小关系
        if (mid <= x / mid) {
            // 如果 mid 的平方小于等于 x，说明答案可能在 mid 右侧
            result = mid;   // 更新结果
            left = mid + 1;  // 调整左边界
        } else {
            // 如果 mid 的平方大于 x，说明答案可能在 mid 左侧
            right = mid - 1; // 调整右边界
        }
    }

    return result;
}

int main() {
    // 测试例子
    int number = 16;

    // 调用函数计算整数的开方根
    int result = sqrtBinarySearch(number);

    // 输出结果
    std::cout << "Square root of " << number << " is: " << result << std::endl;

    return 0;
}
```

10、代码题：给出如www.bilibili.com这样的字符串，将其转换为com.bilibili.www(一开始用栈写的，后面要求优化成O(1)空间复杂度)

思路：

1. 将字符串分成两部分，即左边和右边。
2. 对左右两部分分别进行反转。
3. 最后对整个字符串进行反转

参考代码：

```
#include <iostream>
#include <algorithm>

// 函数声明：旋转字符串
void rotateString(std::string &s, int k);

int main() {
    // 测试例子
    std::string str = "www.bilibili.com";

    // 调用函数旋转字符串
    rotateString(str, 3);

    // 输出结果
    std::cout << "Rotated string: " << str << std::endl;

    return 0;
}

// 函数定义：旋转字符串
void rotateString(std::string &s, int k) {
    int n = s.length();

    // 防止超出字符串长度
    k = k % n;

    // 分为左右两部分，分别进行反转
    std::reverse(s.begin(), s.begin() + n - k);
    std::reverse(s.begin() + n - k, s.end());

    // 对整个字符串进行反转
    std::reverse(s.begin(), s.end());
}
```

11、反问，面试官介绍业务主要和大模型相关的