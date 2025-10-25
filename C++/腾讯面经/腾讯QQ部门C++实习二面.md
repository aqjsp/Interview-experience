# 腾讯QQ部门C++实习二面

## 腾讯QQ部门C++实习二面

### 1、什么时候开始接触计算的？平常怎样学习？

### 2、了解过C++20的特性吗？说了解过C++11

#### C++11

1. 自动类型推导（Auto）： 允许编译器推导变量的类型，使代码更加简洁。

```C++
auto x = 5; // x的类型将被推导为int
```

2. 范围-based for 循环： 简化了对容器元素的遍历。

```C++
std::vector<int> numbers = {1, 2, 3, 4, 5};
for (const auto& num : numbers) {
    // 使用num
}
```

3. 智能指针： 引入了`std::shared_ptr`和`std::unique_ptr`等智能指针，用于管理动态分配的内存，帮助防止内存泄漏。

```C++
std::shared_ptr<int> sharedPtr = std::make_shared<int>(42);
```

4. Lambda 表达式： 允许在函数内部定义匿名函数，提高代码可读性和灵活性。

```C++
auto add = [](int a, int b) { return a + b; };
```

5. nullptr： 引入了空指针常量`nullptr`，用于替代传统的空指针`NULL`。

```C++
int* ptr = nullptr;
```

6. 强制类型转换（Type Casting）： 引入了`static_cast`、`dynamic_cast`、`const_cast`、`reinterpret_cast`等更安全和灵活的类型转换操作符。

```C++
double x = 3.14;
int y = static_cast<int>(x);
```

7. 右值引用和移动语义： 支持通过右值引用实现移动语义，提高了对临时对象的处理效率。

```C++
std::vector<int> getVector() {
    // 返回一个临时vector
    return std::vector<int>{1, 2, 3};
}

std::vector<int> numbers = getVector(); // 使用移动语义
```

8. 新的容器和算法： 引入了新的容器，如`std::unordered_map`、`std::unordered_set`，以及一些新的算法。

```C++
std::unordered_map<int, std::string> myMap = {{1, "one"}, {2, "two"}};
```

9. 线程支持（std::thread）： 提供了原生的多线程支持，使得并发编程更加方便。

```C++
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

#### C++20

1. 概念（Concepts）：概念是一种约束，用于定义模板参数的要求。概念可以在编译时检查模板参数是否符合要求，提供了更加灵活和明确的模板编程方式。例如：

   ```c++
   template <typename T>
   concept Integral = std::is_integral_v<T>;
   
   template <Integral T>
   void foo(T t) {
       // 只能被整数调用
   }
   ```

2. 协程（Coroutines）：协程是一种轻量级的线程，可以在函数内部实现暂停和恢复执行，提供了一种更加高效和可控的并发编程方式。例如：

   ```c++
   #include <coroutine>
   #include <iostream>
   
   // 定义一个协程结构体
   struct MyCoroutine {
       // 定义协程的 promise_type
       struct promise_type {
           int value;
   
           // 获取协程的返回对象
           auto get_return_object() { return MyCoroutine{std::coroutine_handle<promise_type>::from_promise(*this)}; }
   
           // 初始挂起点，这里是永不挂起
           auto initial_suspend() { return std::suspend_never{}; }
   
           // 最终挂起点，这里是永不挂起
           auto final_suspend() noexcept { return std::suspend_never{}; }
   
           // 返回空值
           void return_void() {}
   
           // 处理未捕获的异常
           void unhandled_exception() {}
       };
   
       std::coroutine_handle<promise_type> coro; // 协程句柄
   
       // 构造函数，传入协程句柄
       MyCoroutine(std::coroutine_handle<promise_type> h) : coro(h) {}
   
       // 析构函数，在协程对象销毁时销毁句柄
       ~MyCoroutine() {
           if (coro) coro.destroy();
       }
   
       // 恢复协程执行
       void resume() {
           coro.resume();
       }
   };
   
   // 定义一个计数器协程函数
   MyCoroutine counter() {
       for (int i = 0; i < 5; ++i) {
           std::cout << "Coroutine: " << i << std::endl;
           co_await std::suspend_always{}; // 挂起协程
       }
   }
   
   int main() {
       MyCoroutine c = counter(); // 创建计数器协程
       for (int i = 0; i < 5; ++i) {
           std::cout << "Main: " << i << std::endl;
           c.resume(); // 恢复协程执行
       }
       return 0;
   }
   ```

3. 初始化改进：C++20引入了一些初始化方面的改进，如默认成员初始化器、聚合类的设计者指定初始化、constexpr动态内存分配等，使得初始化更加灵活和易用。

4. 范围的迭代器和插入器：C++20引入了`std::ranges`命名空间，提供了一组新的范围操作函数和迭代器概念，使得范围操作更加直观和高效。

5. 三向比较运算符：引入了`<=>`运算符，用于进行三向比较，简化了比较操作的实现。

6. 并发改进：C++20引入了一些并发方面的改进，如`std::jthread`、`std::stop_token`、`std::latch`等，提供了更加灵活和安全的并发编程方式。

7. 其他改进：C++20还引入了许多其他改进，如模块化、constexpr函数和算法、容器和算法的改进、类型特征和类型转换的改进等，提高了代码的可读性、性能和安全性。

### 3、介绍一下智能指针是干什么用的？是怎么实现的？

1. std::unique_ptr：独占式智能指针，表示对其指向的对象拥有独占所有权。当 unique_ptr 被销毁时，它所管理的对象也会被销毁。不能进行复制，但可以进行移动。可以通过 std::make_unique 来创建。

   ```c++
   std::unique_ptr<int> ptr = std::make_unique<int>(10);
   ```

2. std::shared_ptr：共享式智能指针，允许多个指针共享同一个对象。它使用引用计数来管理对象的生命周期，当最后一个 shared_ptr 被销毁时，引用计数为0，对象被销毁。可以通过 std::make_shared 来创建。

   ```c++
   std::shared_ptr<int> ptr1 = std::make_shared<int>(20);
   std::shared_ptr<int> ptr2 = ptr1; // 共享所有权
   ```

3. std::weak_ptr：弱引用智能指针，用于解决 shared_ptr 的循环引用问题。weak_ptr 可以观测 shared_ptr 的生命周期，但不会增加引用计数。可以通过 shared_ptr 的成员函数 `weak_ptr::lock` 来获取一个指向同一对象的 shared_ptr。

   ```c++
   std::weak_ptr<int> weakPtr = ptr1;
   if (std::shared_ptr<int> sharedPtr = weakPtr.lock()) {
       // 使用 sharedPtr
   }
   ```

4. std::auto_ptr：已经被废弃，不再建议使用。它是 C++11 之前的标准库中的一种智能指针，与 unique_ptr 类似，但不支持移动语义，存在潜在的安全问题。

智能指针的实现主要依赖于模板类和 RAII（资源获取即初始化）原则。在构造函数中，智能指针会获取所管理对象的内存资源，并在析构函数中释放这些资源。通过这种方式，即使发生异常或提前返回等情况，资源也能得到正确释放，避免内存泄漏。

### 4、介绍一下你的两个项目？

### 5、说一说这个项目为什么比malloc函数快？

### 6、你有没有思考过为什么C语言中要是用malloc而不使用这个项目？这个项目不是更优吗？

### 7、做这两个项目的时候有没有遇见什么难点？最后是怎样解决的？

### 8、这两个项目有没有什么缺陷？使用起来有bug吗？

### 9、手撕算法题：判断链表中是否存在环？

#### 问题描述

给你一个链表的头节点 `head` ，判断链表中是否有环。

如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（索引从 0 开始）。**注意：`pos` 不作为参数进行传递** 。仅仅是为了标识链表的实际情况。

如果链表中存在环 ，则返回 `true` 。 否则，返回 `false` 。

#### 思路

1. 定义两个指针 `slow` 和 `fast`，初始时都指向链表的头节点 `head`。
2. `slow` 每次向后移动一个节点，`fast` 每次向后移动两个节点。这样，如果链表中有环，`fast` 最终会追上 `slow`。
3. 如果 `fast` 为 `nullptr` 或者 `fast->next` 为 `nullptr`，则说明链表中不存在环，返回 `false`。
4. 如果 `fast` 和 `slow` 相遇，则说明链表中存在环，返回 `true`。

#### 参考代码

```c++
#include <iostream>

// 定义链表节点结构
struct ListNode {
    int val;
    ListNode *next;
    ListNode(int x) : val(x), next(nullptr) {}
};

class Solution {
public:
    bool hasCycle(ListNode *head) {
        // 定义快慢指针，并初始化为链表头节点
        ListNode *slow = head;
        ListNode *fast = head;

        // 使用快慢指针判断链表中是否存在环
        while (fast != nullptr && fast->next != nullptr) {
            slow = slow->next;       // 慢指针每次移动一步
            fast = fast->next->next; // 快指针每次移动两步
            if (slow == fast) {      // 如果快慢指针相遇，说明存在环
                return true;
            }
        }

        // 如果快指针到达链表尾部，说明不存在环
        return false;
    }
};

int main() {
    // 创建一个有环的链表
    ListNode *head = new ListNode(3);
    head->next = new ListNode(2);
    head->next->next = new ListNode(0);
    head->next->next->next = new ListNode(-4);
    head->next->next->next->next = head->next; // 尾节点连接到第二个节点，形成环

    Solution sol;
    std::cout << std::boolalpha << sol.hasCycle(head) << std::endl; // 输出 true，表示链表中存在环

    // 释放链表内存
    ListNode *curr = head;
    while (curr != nullptr) {
        ListNode *temp = curr;
        curr = curr->next;
        delete temp;
    }

    return 0;
}
```

### 10、手撕算法题：找出两个数组中最长的相同的子数组？

#### 问题描述

给两个整数数组 `nums1` 和 `nums2` ，返回 两个数组中 公共的 、长度最长的子数组的长度 。

#### 思路

1. 创建一个二维数组 `dp`，其中 `dp[i][j]` 表示 `nums1` 中以 `nums1[i-1]` 结尾和 `nums2` 中以 `nums2[j-1]` 结尾的公共子数组的长度。
2. 初始化 `dp[i][0]` 和 `dp[0][j]` 为 0，因为任何一个数组以空数组结尾的公共子数组长度都为 0。
3. 遍历 `nums1` 和 `nums2`，如果当前元素相等，则 `dp[i][j] = dp[i-1][j-1] + 1`，否则为 0。
4. 在遍历过程中，记录最大的公共子数组长度。
5. 返回最大公共子数组长度。

#### 参考代码

```c++
#include <vector>
#include <iostream>

int findLength(std::vector<int>& nums1, std::vector<int>& nums2) {
    int m = nums1.size();
    int n = nums2.size();
    // 创建一个二维数组 dp，初始化为 0，dp[i][j] 表示以 nums1[i-1] 和 nums2[j-1] 结尾的最长公共子数组的长度
    std::vector<std::vector<int>> dp(m + 1, std::vector<int>(n + 1, 0));
    int maxLen = 0; // 记录最大的公共子数组长度

    // 动态规划，遍历 nums1 和 nums2
    for (int i = 1; i <= m; ++i) {
        for (int j = 1; j <= n; ++j) {
            if (nums1[i - 1] == nums2[j - 1]) {
                // 如果当前元素相等，则最长公共子数组的长度加一
                dp[i][j] = dp[i - 1][j - 1] + 1;
                // 更新最大的公共子数组长度
                maxLen = std::max(maxLen, dp[i][j]);
            }
        }
    }

    return maxLen;
}

int main() {
    std::vector<int> nums1 = {1, 2, 3, 2, 1};
    std::vector<int> nums2 = {3, 2, 1, 4, 7};
    // 输出最长公共子数组的长度
    std::cout << "最长公共子数组的长度为：" << findLength(nums1, nums2) << std::endl;
    return 0;
}
```

### 11、手撕算法题：全排列？

#### 问题描述

给定一个不含重复数字的数组 `nums` ，返回其 所有可能的全排列 。你可以 按任意顺序返回答案。

#### 思路

1. 定义一个递归函数 `permuteHelper`，参数包括当前排列 `current`、已经使用过的数字的集合 `used`、原始数组 `nums`。
2. 当当前排列的长度等于原始数组的长度时，表示找到了一个全排列，将其加入结果集合中。
3. 遍历原始数组，如果当前数字没有被使用过，则将其加入当前排列中，将其标记为已使用，然后递归调用 `permuteHelper`。
4. 在递归调用之后，要回溯到上一步的状态，即将当前数字从当前排列中移除，将其标记为未使用，继续遍历其他数字。

#### 参考代码

```c++
#include <vector>
#include <iostream>

class Solution {
public:
    // 返回数组 nums 的全排列
    std::vector<std::vector<int>> permute(std::vector<int>& nums) {
        std::vector<std::vector<int>> result; // 存储所有排列结果
        std::vector<int> current; // 当前排列
        std::vector<bool> used(nums.size(), false); // 记录数字是否被使用过
        permuteHelper(nums, used, current, result); // 调用辅助函数生成全排列
        return result;
    }

    // 辅助函数，生成数组 nums 的全排列
    void permuteHelper(const std::vector<int>& nums, std::vector<bool>& used, std::vector<int>& current, std::vector<std::vector<int>>& result) {
        if (current.size() == nums.size()) { // 如果当前排列长度等于数组长度，表示找到一个全排列
            result.push_back(current); // 将当前排列加入结果集
            return;
        }

        for (int i = 0; i < nums.size(); ++i) {
            if (!used[i]) { // 如果数字未被使用过，则加入当前排列中
                current.push_back(nums[i]); // 加入当前数字
                used[i] = true; // 标记为已使用
                permuteHelper(nums, used, current, result); // 递归生成剩余数字的排列
                current.pop_back(); // 回溯，移除当前数字
                used[i] = false; // 恢复状态
            }
        }
    }
};

int main() {
    std::vector<int> nums = {1, 2, 3};
    Solution solution;
    std::vector<std::vector<int>> result = solution.permute(nums);
    std::cout << "所有可能的全排列：" << std::endl;
    for (const auto& perm : result) {
        for (int num : perm) {
            std::cout << num << " ";
        }
        std::cout << std::endl;
    }
    return 0;
}
```

面试官说他很讨厌八股文，所有问的问题全是和项目相关的，根本水不了。