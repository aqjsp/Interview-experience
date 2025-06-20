# 腾讯wxg金融微信C++后台开发 一面凉经

> 来源：https://www.nowcoder.com/feed/main/detail/7c4fe897008147eba93ad821ebc60e57

## 手撕算法

### 第一题： IPv4地址字符串转为无符号整数。

#### 思路

IPv4地址由四个0-255之间的十进制数组成，每个数之间用点号（.）分隔。例如：192.168.1.1。将其转换为无符号整数，实际上是将这四个十进制数看作是四个字节，然后将它们组合成一个32位的无符号整数。转换的原理是：

IP地址 = (第一段数字 * 256^3) + (第二段数字 * 256^2) + (第三段数字 * 256^1) + (第四段数字 * 256^0)

或者更直观地理解为：

IP地址 = (第一段数字 << 24) | (第二段数字 << 16) | (第三段数字 << 8) | (第四段数字 << 0)

其中，<< 是位左移运算符，| 是按位或运算符。这种转换方式利用了每个字节在32位整数中的位置权重。例如，第一段数字占据最高8位，第二段数字占据次高8位，以此类推。

实现步骤如下：

1. 解析字符串：将输入的IPv4地址字符串按照点号（.）进行分割，得到四个字符串形式的数字。

2. 字符串转整数：将这四个字符串数字分别转换为对应的十进制整数。

3. 位运算组合：将转换后的四个整数按照上述位运算的规则组合成一个32位无符号整数。

4. 错误处理：在解析和转换过程中，需要考虑各种异常情况，例如：
   - 字符串格式不正确（缺少点号、多余点号、非数字字符等）。
   - 数字超出0-255的范围。
   - 空字符串或只有点号的字符串。

#### 参考代码

```c++
#include <iostream>
#include <string>
#include <vector>
#include <sstream>
#include <stdexcept>

// 函数：将IPv4地址字符串转换为无符号整数
unsigned int ipv4_to_uint(const std::string& ipv4_str) {
    std::vector<std::string> segments;
    std::string segment;
    std::stringstream ss(ipv4_str);
    
    // 按点号分割字符串
    while (std::getline(ss, segment, '.')) {
        segments.push_back(segment);
    }

    // 检查是否恰好有四段
    if (segments.size() != 4) {
        throw std::invalid_argument("Invalid IPv4 address format: incorrect number of segments.");
    }

    unsigned int result = 0;
    for (int i = 0; i < 4; ++i) {
        int num;
        try {
            // 字符串转整数
            num = std::stoi(segments[i]);
        } catch (const std::out_of_range& oor) {
            throw std::out_of_range("IPv4 segment out of range: " + segments[i]);
        } catch (const std::invalid_argument& ia) {
            throw std::invalid_argument("Invalid IPv4 segment: non-numeric characters found in " + segments[i]);
        }

        // 校验数字范围
        if (num < 0 || num > 255) {
            throw std::out_of_range("IPv4 segment out of valid range (0-255): " + std::to_string(num));
        }
        
        // 位运算组合
        result = (result << 8) | num;
    }

    return result;
}

int main() {
    // 测试用例
    std::vector<std::string> test_ips = {
        "192.168.1.1",
        "0.0.0.0",
        "255.255.255.255",
        "10.0.0.1",
        "172.16.254.1",
        "invalid.ip.address", // 非法格式
        "1.2.3",              // 段数不足
        "1.2.3.4.5",          // 段数过多
        "256.0.0.1",          // 数字超出范围
        "1.2.3.abc",          // 包含非数字字符
        "",                   // 空字符串
        ".1.2.3.4",           // 开头有点号
        "1.2.3.4.",           // 结尾有点号
        "1..2.3.4"            // 中间有连续点号
    };

    for (const std::string& ip : test_ips) {
        std::cout << "Converting IP: " << ip << " -> ";
        try {
            unsigned int uint_ip = ipv4_to_uint(ip);
            std::cout << uint_ip << " (0x" << std::hex << uint_ip << std::dec << ")" << std::endl;
        } catch (const std::exception& e) {
            std::cerr << "Error: " << e.what() << std::endl;
        }
    }

    return 0;
}
```

输出：

![IPV4](https://cdn.jsdelivr.net/gh/aqjsp/photos/image-20250621000851907.png)

### 第二题：二叉树中的最大路径和。

#### 思路

二叉树中的“路径”被定义为从任意节点到任意节点的序列，且该序列中的节点最多只出现一次。路径不一定包含根节点。我们需要找到这样一条路径，使其节点值的和最大。

这道题的核心难点在于路径的定义：它可以从树的任意位置开始，在任意位置结束。这意味着路径可能：

1.只包含一个节点：如果所有路径和都为负，则最大路径和可能是最大的那个负数节点的值。

2.从某个节点向下延伸到其左子树或右子树：例如，从父节点到其左子节点，再到左子节点的某个后代。

3.经过某个节点，并同时连接其左子树和右子树：形成一个“V”字形或“倒U”字形的路径，该节点是路径的“顶点”。

为了解决这个问题，我们可以采用深度优先搜索（DFS）或递归的方法。对于每个节点，我们考虑两种情况：

- 作为路径的“顶点”：此时，最大路径和是 当前节点值 + 左子树提供的最大路径和（如果为正） + 右子树提供的最大路径和（如果为正）。这个和可能是全局最大路径和的候选。
- 作为路径的一部分，向上贡献给父节点：此时，当前节点只能选择其左子树或右子树中能提供最大正值路径和的一条路径，加上自身节点值，然后将这个和返回给父节点。如果左右子树提供的路径和都为负，那么就不选择它们，只返回当前节点值（或者0，如果当前节点值也为负，且父节点不希望包含它）。

因此，我们需要一个全局变量来记录迄今为止找到的最大路径和，并在每次计算“作为路径顶点”的情况时更新它。同时，递归函数需要返回“作为路径一部分，向上贡献”的最大值。

具体来说，对于当前节点 node：

1. 递归计算其左子节点 left_sum 和右子节点 right_sum 的最大路径和贡献。如果 `left_sum` 或 `right_sum` 小于0，则将其视为0，因为负的贡献会减小总和。

2. 计算以 node 为“顶点”的最大路径和：`node.val + max(0, left_sum) + max(0, right_sum)`。用这个值更新全局最大路径和。

3. 返回 node 能向上贡献的最大路径和：`node.val + max(0, max(left_sum, right_sum))`。这个值将用于其父节点的计算。

初始时，全局最大路径和应设置为一个非常小的负数，以确保任何有效的路径和都能更新它。

#### 参考代码

```c++
#include <iostream>
#include <algorithm> // For std::max
#include <limits>    // For std::numeric_limits<int>::min()

// Definition for a binary tree node.
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
    // 全局变量，用于存储最大路径和
    int max_path_sum_global;

    // 递归函数，计算从当前节点向下延伸的最大路径和，并更新全局最大值
    int dfs(TreeNode* node) {
        // 基准情况：如果节点为空，则贡献为0
        if (node == nullptr) {
            return 0;
        }

        // 递归计算左右子树的最大路径和贡献
        // 如果子树的路径和为负，则选择0，表示不选择该子树的路径
        int left_gain = std::max(0, dfs(node->left));
        int right_gain = std::max(0, dfs(node->right));

        // 计算以当前节点为“顶点”的路径和
        // 这条路径可以连接左右子树，是全局最大路径和的候选
        int current_path_sum = node->val + left_gain + right_gain;

        // 更新全局最大路径和
        max_path_sum_global = std::max(max_path_sum_global, current_path_sum);

        // 返回当前节点能向上贡献的最大路径和
        // 只能选择左子树或右子树中贡献更大的那一条，因为路径不能分叉
        return node->val + std::max(left_gain, right_gain);
    }

    // 主函数，计算二叉树的最大路径和
    int maxPathSum(TreeNode* root) {
        // 初始化全局最大路径和为一个非常小的负数
        max_path_sum_global = std::numeric_limits<int>::min();
        
        // 调用递归函数开始计算
        dfs(root);
        
        // 返回最终结果
        return max_path_sum_global;
    }
};

// 辅助函数：用于构建测试用例的二叉树
// 简单的层序遍历构建，方便测试
TreeNode* buildTree(const std::vector<int*>& nodes) {
    if (nodes.empty() || nodes[0] == nullptr) {
        return nullptr;
    }

    TreeNode* root = new TreeNode(*nodes[0]);
    std::vector<TreeNode*> q;
    q.push_back(root);
    int i = 1;
    int head = 0;

    while (head < q.size() && i < nodes.size()) {
        TreeNode* current = q[head++];

        if (nodes[i] != nullptr) {
            current->left = new TreeNode(*nodes[i]);
            q.push_back(current->left);
        }
        i++;

        if (i < nodes.size() && nodes[i] != nullptr) {
            current->right = new TreeNode(*nodes[i]);
            q.push_back(current->right);
        }
        i++;
    }
    return root;
}

// 辅助函数：释放二叉树内存
void deleteTree(TreeNode* root) {
    if (root == nullptr) {
        return;
    }
    deleteTree(root->left);
    deleteTree(root->right);
    delete root;
}

int main() {
    Solution sol;

    // Test Case 1: [1,2,3]
    //      1
    //     / \
    //    2   3
    int n1_1 = 1, n1_2 = 2, n1_3 = 3;
    std::vector<int*> vals1 = {&n1_1, &n1_2, &n1_3};
    TreeNode* root1 = buildTree(vals1);
    std::cout << "Test Case 1: [1,2,3] -> Max Path Sum: " << sol.maxPathSum(root1) << std::endl; // Expected: 6 (2-1-3)
    deleteTree(root1);

    // Test Case 2: [-10,9,20,nullptr,nullptr,15,7]
    //       -10
    //       /  \
    //      9   20
    //         /  \
    //        15   7
    int n2_1 = -10, n2_2 = 9, n2_3 = 20, n2_4 = 15, n2_5 = 7;
    std::vector<int*> vals2 = {&n2_1, &n2_2, &n2_3, nullptr, nullptr, &n2_4, &n2_5};
    TreeNode* root2 = buildTree(vals2);
    std::cout << "Test Case 2: [-10,9,20,null,null,15,7] -> Max Path Sum: " << sol.maxPathSum(root2) << std::endl; // Expected: 42 (15-20-7)
    deleteTree(root2);

    // Test Case 3: [5,4,8,11,nullptr,13,4,7,2,nullptr,nullptr,nullptr,1]
    //         5
    //        / \
    //       4   8
    //      /   / \
    //     11  13  4
    //    / \       \
    //   7   2       1
    int n3_1 = 5, n3_2 = 4, n3_3 = 8, n3_4 = 11, n3_5 = 13, n3_6 = 4, n3_7 = 7, n3_8 = 2, n3_9 = 1;
    std::vector<int*> vals3 = {&n3_1, &n3_2, &n3_3, &n3_4, nullptr, &n3_5, &n3_6, &n3_7, &n3_8, nullptr, nullptr, nullptr, &n3_9};
    TreeNode* root3 = buildTree(vals3);
    std::cout << "Test Case 3: [5,4,8,11,null,13,4,7,2,null,null,null,1] -> Max Path Sum: " << sol.maxPathSum(root3) << std::endl; // Expected: 48 (7-11-4-5-8-13)
    deleteTree(root3);

    // Test Case 4: [-3]
    //      -3
    int n4_1 = -3;
    std::vector<int*> vals4 = {&n4_1};
    TreeNode* root4 = buildTree(vals4);
    std::cout << "Test Case 4: [-3] -> Max Path Sum: " << sol.maxPathSum(root4) << std::endl; // Expected: -3
    deleteTree(root4);

    // Test Case 5: [1, -2, -3]
    //      1
    //     / \
    //    -2  -3
    int n5_1 = 1, n5_2 = -2, n5_3 = -3;
    std::vector<int*> vals5 = {&n5_1, &n5_2, &n5_3};
    TreeNode* root5 = buildTree(vals5);
    std::cout << "Test Case 5: [1,-2,-3] -> Max Path Sum: " << sol.maxPathSum(root5) << std::endl; // Expected: 1
    deleteTree(root5);

    return 0;
}
```

输出：

![二叉树](https://cdn.jsdelivr.net/gh/aqjsp/photos/image-20250621001123598.png)

### 第三题：链表尾部的K组节点为一组翻转。

#### 思路

这道题要求我们将链表尾部的节点每 K 个一组进行翻转。与常见的“每K个节点翻转”不同，这里强调的是“从尾部开始分组”。这意味着我们需要先确定从链表尾部开始的K个节点，然后是再往前K个节点，以此类推，对这些分组进行翻转。如果最前面的分组不足K个节点，则不进行翻转。

例如：链表 `1 -> 2 -> 3 -> 4 -> 5 -> 6 -> 7 -> 8`，K = 3。

- 从尾部开始分组：
  - 第一组：`6 -> 7 -> 8`
  - 第二组：`3 -> 4 -> 5`
  - 第三组：`1 -> 2` (不足K个，不翻转)
- 翻转后的结果：
  - `8 -> 7 -> 6`
  - `5 -> 4 -> 3`
  - `1 -> 2` (保持不变)
- 最终链表：`1 -> 2 -> 5 -> 4 -> 3 -> 8 -> 7 -> 6`

解决这个问题的关键在于如何高效地找到从尾部开始的K个节点，并进行翻转。一种直观的方法是：

1. 计算链表总长度：遍历链表一次，得到链表的总节点数 N。

2. 确定需要翻转的起始位置：由于是从尾部开始分组，每 K 个一组，那么需要翻转的节点总数是 (N / K) * K。例如，N=8, K=3，则 (8/3)*3 = 2*3 = 6 个节点需要翻转。这意味着链表的前 N - (N / K) * K 个节点将保持不变。在我们的例子中，8 - 6 = 2，即 1 -> 2 这两个节点保持不变。

3. 找到翻转的起始节点：从链表头开始，走 N - (N / K) * K 步，找到第一个需要翻转的节点的前一个节点（即翻转部分的“前驱”）。

4. 分段翻转：从这个前驱节点开始，每 K 个节点进行一次翻转。这与“每K个节点翻转”的经典问题类似，但需要注意连接。

这种方法需要两次遍历链表：一次计算长度，一次进行翻转。如果 K 很大，或者链表很长，这可能是效率较高的方法。

另一种更巧妙的方法（使用栈）：

1. 遍历链表并入栈：将链表的所有节点依次压入一个栈中。这样，栈顶就是链表的尾部节点。

2. 从栈中取出并翻转：
   - 从栈顶开始，每 K 个节点取出来，形成一个临时链表，并将其翻转。将翻转后的链表头和尾保存下来。
   - 将翻转后的链表段连接到结果链表的尾部。
   - 重复此过程，直到栈中节点不足 K 个。

3. 处理剩余节点：栈中剩余的节点（不足 K 个的部分）是链表的最前端，它们不进行翻转，直接连接到结果链表的头部。

这种方法虽然也需要遍历链表两次（一次入栈，一次出栈），但逻辑上可能更清晰，且不需要提前计算链表长度。缺点是需要额外的栈空间，空间复杂度为O(N)。

考虑到题目可能更倾向于链表操作的技巧，我们选择第一种方法，因为它只涉及指针操作，空间复杂度为O(1)（不考虑递归栈空间）。

#### 参考代码

```c++
#include <iostream>
#include <vector>
#include <utility> // For std::pair

// Definition for singly-linked list.
struct ListNode {
    int val;
    ListNode *next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode *next) : val(x), next(next) {}
};

// 辅助函数：翻转链表的前 k 个节点
// 返回一个 pair: {翻转后的新头, 翻转后的新尾的下一个节点}
std::pair<ListNode*, ListNode*> reverseKNodes(ListNode* head, int k) {
    if (head == nullptr || k <= 1) {
        return {head, head->next}; // 如果不足 k 个或 k=1，不翻转
    }

    ListNode* current = head;
    // 检查是否有 k 个节点
    for (int i = 0; i < k; ++i) {
        if (current == nullptr) {
            return {head, head->next}; // 不足 k 个，不翻转
        }
        current = current->next;
    }

    // 此时 current 指向第 k+1 个节点 (如果存在)
    // head 仍然指向第 1 个节点

    ListNode* prev = nullptr;
    ListNode* curr = head;
    ListNode* next_node = nullptr;

    // 翻转前 k 个节点
    for (int i = 0; i < k; ++i) {
        next_node = curr->next;
        curr->next = prev;
        prev = curr;
        curr = next_node;
    }
    // 翻转完成后：
    // prev 是翻转后的新头 (原第 k 个节点)
    // head 是翻转后的新尾 (原第 1 个节点)
    // curr 是原第 k+1 个节点

    // 将翻转后的尾部连接到原第 k+1 个节点
    head->next = curr;

    return {prev, head}; // 返回新头和新尾
}

class Solution {
public:
    ListNode* reverseKGroupFromEnd(ListNode* head, int k) {
        if (head == nullptr || k <= 1) {
            return head;
        }

        // 1. 计算链表总长度 N
        int n = 0;
        ListNode* temp = head;
        while (temp != nullptr) {
            n++;
            temp = temp->next;
        }

        // 2. 计算头部不需要翻转的节点数
        int skip_count = n % k;

        // 创建一个哑节点，方便处理头部翻转
        ListNode* dummy_head = new ListNode(0);
        dummy_head->next = head;

        // 找到第一个需要翻转的组的前一个节点
        ListNode* current = dummy_head;
        for (int i = 0; i < skip_count; ++i) {
            current = current->next;
        }

        // current 现在指向不需要翻转部分的最后一个节点
        // 或者如果 skip_count 为 0，则指向 dummy_head

        // 3. 分段翻转并连接
        ListNode* prev_tail = current; // 记录上一段翻转后的尾节点

        while (current->next != nullptr) {
            // current->next 是当前需要翻转的 k 个节点的起始
            std::pair<ListNode*, ListNode*> reversed_segment = reverseKNodes(current->next, k);
            
            // reversed_segment.first 是翻转后的新头
            // reversed_segment.second 是翻转后的新尾 (原当前段的头)

            // 将上一段的尾部连接到当前翻转段的新头
            prev_tail->next = reversed_segment.first;
            
            // 更新 prev_tail 为当前翻转段的尾部
            prev_tail = reversed_segment.second;

            // current 移动到当前翻转段的尾部，准备处理下一段
            current = prev_tail; 
        }

        ListNode* new_head = dummy_head->next;
        delete dummy_head; // 释放哑节点内存
        return new_head;
    }
};

// 辅助函数：打印链表
void printList(ListNode* head) {
    ListNode* current = head;
    while (current != nullptr) {
        std::cout << current->val;
        if (current->next != nullptr) {
            std::cout << " -> ";
        }
        current = current->next;
    }
    std::cout << std::endl;
}

// 辅助函数：创建链表
ListNode* createList(const std::vector<int>& nums) {
    if (nums.empty()) {
        return nullptr;
    }
    ListNode* head = new ListNode(nums[0]);
    ListNode* current = head;
    for (size_t i = 1; i < nums.size(); ++i) {
        current->next = new ListNode(nums[i]);
        current = current->next;
    }
    return head;
}

// 辅助函数：释放链表内存
void deleteList(ListNode* head) {
    ListNode* current = head;
    while (current != nullptr) {
        ListNode* next = current->next;
        delete current;
        current = next;
    }
}

int main() {
    Solution sol;

    // Test Case 1: 1->2->3->4->5->6->7->8, K=3
    // Expected: 1->2->5->4->3->8->7->6
    std::vector<int> nums1 = {1, 2, 3, 4, 5, 6, 7, 8};
    ListNode* head1 = createList(nums1);
    std::cout << "Original List 1: ";
    printList(head1);
    ListNode* result1 = sol.reverseKGroupFromEnd(head1, 3);
    std::cout << "Reversed List 1 (K=3): ";
    printList(result1);
    deleteList(result1);

    // Test Case 2: 1->2->3->4->5, K=2
    // Expected: 1->5->4->3->2
    std::vector<int> nums2 = {1, 2, 3, 4, 5};
    ListNode* head2 = createList(nums2);
    std::cout << "Original List 2: ";
    printList(head2);
    ListNode* result2 = sol.reverseKGroupFromEnd(head2, 2);
    std::cout << "Reversed List 2 (K=2): ";
    printList(result2);
    deleteList(result2);

    // Test Case 3: 1->2->3->4->5, K=5
    // Expected: 5->4->3->2->1
    std::vector<int> nums3 = {1, 2, 3, 4, 5};
    ListNode* head3 = createList(nums3);
    std::cout << "Original List 3: ";
    printList(head3);
    ListNode* result3 = sol.reverseKGroupFromEnd(head3, 5);
    std::cout << "Reversed List 3 (K=5): ";
    printList(result3);
    deleteList(result3);

    // Test Case 4: 1->2->3, K=4 (K > N)
    // Expected: 1->2->3 (不翻转)
    std::vector<int> nums4 = {1, 2, 3};
    ListNode* head4 = createList(nums4);
    std::cout << "Original List 4: ";
    printList(head4);
    ListNode* result4 = sol.reverseKGroupFromEnd(head4, 4);
    std::cout << "Reversed List 4 (K=4): ";
    printList(result4);
    deleteList(result4);

    // Test Case 5: 1->2->3->4->5->6, K=3
    // Expected: 3->2->1->6->5->4
    std::vector<int> nums5 = {1, 2, 3, 4, 5, 6};
    ListNode* head5 = createList(nums5);
    std::cout << "Original List 5: ";
    printList(head5);
    ListNode* result5 = sol.reverseKGroupFromEnd(head5, 3);
    std::cout << "Reversed List 5 (K=3): ";
    printList(result5);
    deleteList(result5);

    // Test Case 6: 空链表, K=3
    // Expected: (空)
    std::vector<int> nums6 = {};
    ListNode* head6 = createList(nums6);
    std::cout << "Original List 6: ";
    printList(head6);
    ListNode* result6 = sol.reverseKGroupFromEnd(head6, 3);
    std::cout << "Reversed List 6 (K=3): ";
    printList(result6);
    deleteList(result6);

    // Test Case 7: 单节点链表, K=3
    // Expected: 1
    std::vector<int> nums7 = {1};
    ListNode* head7 = createList(nums7);
    std::cout << "Original List 7: ";
    printList(head7);
    ListNode* result7 = sol.reverseKGroupFromEnd(head7, 3);
    std::cout << "Reversed List 7 (K=3): ";
    printList(result7);
    deleteList(result7);

    return 0;
}
```

输出：

![链表](https://cdn.jsdelivr.net/gh/aqjsp/photos/image-20250621001521257.png)

### 第四题：带有优先级的括号匹配。 (例如 {[()]} 合法, [{}] 不合法)

#### 思路

这道题是经典括号匹配问题的变种，增加了“优先级”的概念。传统的括号匹配只要求开闭括号类型和顺序正确，例如 ()、[]、{}。而带有优先级的括号匹配则要求在嵌套时，外部括号的优先级必须高于内部括号。题目中给出的例子 {[()]} 合法，[{}] 不合法，这明确了优先级规则：

- {} (大括号) 优先级最高。
- [] (中括号) 优先级次之。
- () (小括号) 优先级最低。

这意味着，如果一个括号内部包含另一个括号，那么外部括号的优先级必须严格高于内部括号。例如：

- {[]}: 合法，因为 {} 优先级高于 []。
- {[()]}: 合法，因为 {} 优先级高于 []，[] 优先级高于 ()。
- [{}]: 不合法，因为 [] 优先级低于 {}，但 [] 包含了 {}。
- ([{}]): 不合法，因为 () 优先级低于 []，[] 优先级低于 {}，但 () 包含了 []，[] 包含了 {}。

解决这类问题，栈（Stack）是最佳的数据结构。我们可以遍历输入的字符串，遇到开括号时将其压入栈中，遇到闭括号时检查栈顶元素。具体步骤如下：

1. 定义优先级：首先，我们需要明确每种括号的优先级。可以定义一个映射（map）来存储每种括号的优先级值，例如：

   - '(': 1

   - '[': 2

   - '{': 3

   - 对应的闭括号也应有相同的优先级。

2. 遍历字符串：

   遇到开括号：

   - 如果栈不为空，并且当前开括号的优先级不高于栈顶开括号的优先级，则说明优先级规则被违反，字符串不合法。
   - 将当前开括号压入栈中。

   遇到闭括号

   - 如果栈为空，则说明没有对应的开括号，字符串不合法。
   - 弹出栈顶元素，检查其是否与当前闭括号类型匹配。如果不匹配，字符串不合法。
   - 如果匹配，则继续处理下一个字符。

3. 遍历结束：
   - 如果遍历结束后栈为空，则所有括号都已正确匹配，字符串合法。
   - 如果栈不为空，则说明有未闭合的开括号，字符串不合法。

#### 参考代码

```c++
#include <iostream>
#include <string>
#include <stack>
#include <map>

// 函数：检查带有优先级的括号匹配是否合法
bool isValidWithPriority(const std::string& s) {
    // 定义括号优先级：{ > [ > (
    std::map<char, int> priority_map = {
        {'(', 1},
        {'[', 2},
        {'{', 3}
    };

    // 定义闭括号对应的开括号
    std::map<char, char> matching_bracket_map = {
        {')', '('},
        {']', '['},
        {'}', '{'}
    };

    std::stack<char> st; // 用于存储开括号

    for (char c : s) {
        // 如果是开括号
        if (priority_map.count(c)) {
            // 优先级检查：如果栈不为空，且当前开括号的优先级不高于栈顶开括号的优先级，则不合法
            // 例如：栈顶是 '{' (优先级3)，当前是 '[' (优先级2)，2不高于3，合法。
            // 但如果栈顶是 '[' (优先级2)，当前是 '{' (优先级3)，3高于2，合法。
            // 题目要求：外部括号优先级必须高于内部括号。
            // 所以，如果栈顶是 '(' (优先级1)，当前是 '[' (优先级2)，合法。
            // 如果栈顶是 '[' (优先级2)，当前是 '(' (优先级1)，不合法。
            // 修正：如果栈不为空，且当前开括号的优先级小于等于栈顶开括号的优先级，则不合法。
            // 例如：栈顶是'['(2)，当前是'('(1)，1 <= 2，不合法。
            // 栈顶是'{'(3)，当前是'['(2)，2 <= 3，不合法。
            // 栈顶是'('(1)，当前是'['(2)，2 > 1，合法。
            if (!st.empty() && priority_map[c] <= priority_map[st.top()]) {
                return false;
            }
            st.push(c);
        }
        // 如果是闭括号
        else if (matching_bracket_map.count(c)) {
            // 栈为空或栈顶不匹配，则不合法
            if (st.empty() || st.top() != matching_bracket_map[c]) {
                return false;
            }
            st.pop(); // 匹配成功，弹出栈顶
        }
        // 如果是其他字符，根据题目要求可以忽略或视为非法
        // 这里假设输入只包含括号字符
    }

    // 遍历结束后，如果栈为空，则所有括号都已正确匹配
    return st.empty();
}

int main() {
    // 测试用例
    std::vector<std::string> test_cases = {
        "{[()]}",    // 合法
        "{}",        // 合法
        "[]",        // 合法
        "()",        // 合法
        "{[()]}",    // 合法
        "[{}]",      // 不合法：[] 优先级低于 {}，但包含了 {}
        "([{}])",    // 不合法：() 优先级低于 []，[] 优先级低于 {}，但 () 包含了 []，[] 包含了 {}
        "{([()])}",  // 合法
        "{[]}",      // 合法
        "{[}",       // 不合法：类型不匹配
        "(",         // 不合法：未闭合
        ")",         // 不合法：多余闭括号
        "",          // 合法：空字符串
        "{{}}",      // 合法
        "[()]",      // 合法
        "([)]",      // 不合法：顺序错误
        "{([)]}"     // 不合法：优先级和顺序错误
    };

    for (const std::string& s : test_cases) {
        std::cout << "String: \"" << s << "\" -> " 
                  << (isValidWithPriority(s) ? "Valid" : "Invalid") << std::endl;
    }

    return 0;
}
```

输出：

![括号](https://cdn.jsdelivr.net/gh/aqjsp/photos/image-20250621002058841.png)

## 项目

略

## 基础

### 1、简单介绍一下面向对象的封装、继承和多态。

#### 封装

封装是将数据（成员变量）和操作数据的方法（成员函数）绑定到一个类中，同时隐藏内部实现细节，只向外部暴露必要的接口。这种设计可以保护数据的完整性和安全性，防止外部不当操作。

- 定义与目的：封装通过抽象化数据和操作，将对象的内部状态隐藏起来，外部只能通过定义好的接口（如方法）访问或修改数据。其主要目标是提高软件的可维护性和可修改性，防止不适当的操作。
- 实现方式：在C++中，类通过访问控制符（public、private、protected）来控制成员的访问权限。例如：
  - private：只能在类内部访问。
  - protected：可以在类内部和派生类中访问。
  - public：可以在任何地方访问。
- 示例：以下是一个简单的BankAccount（银行账户）类的示例：

```c++
class BankAccount {
private:
    int accountNumber;  // 私有数据成员
    double balance;     // 私有数据成员
public:
    // 公共方法：存款
    void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }
    // 公共方法：取款
    void withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
        }
    }
    // 公共方法：获取余额
    double getBalance() const {
        return balance;
    }
};
```

好处：

- 提高了数据的安全性和完整性，例如通过deposit方法验证输入金额是否为正。
- 减少了类之间的耦合，外部代码只需关注接口，不需了解内部实现。

局限性：过度封装可能增加代码复杂性，getter/setter方法过多可能降低性能。

#### 继承

继承允许一个类（称为派生类或子类）从另一个类（称为基类或父类）继承属性和方法，从而实现代码的重用和扩展。继承建立了“is-a”关系，例如，Dog是Animal的一种。

- 定义与目的：通过继承，子类可以直接使用父类的非私有成员（数据和方法），并可以扩展或覆盖这些成员。其目标是提高开发效率，减少代码冗余。
- 实现方式：在C++中，使用:符号来表示继承关系，并指定继承类型（public、protected、private）。例如：

```c++
class Animal {
public:
    void eat() {
        std::cout << "Eating..." << std::endl;
    }
};

class Dog : public Animal {  // 公有继承
public:
    void bark() {
        std::cout << "Woof!" << std::endl;
    }
};

class Cat : public Animal {  // 公有继承
public:
    void meow() {
        std::cout << "Meow!" << std::endl;
    }
};
```

- 基类（Base Class）：被继承的类。
- 派生类（Derived Class）：继承其他类的类。
- 继承类型：
  - 公有继承（public）：子类继承父类的公有和保护成员，子类的访问权限与父类相同。
  - 保护继承（protected）：子类继承父类的公有和保护成员，但它们在子类中变为保护成员。
  - 私有继承（private）：子类继承父类的公有和保护成员，但它们在子类中变为私有成员。

优点：

- 减少代码冗余，提高代码重用性。
- 支持层次化设计，方便扩展功能。

局限性：

- 父类变化可能影响子类，破坏封装性。
- 增加类之间的强耦合，可能降低系统灵活性。
- 建议仅在需要向上转型时使用继承。

#### 多态

多态允许同一方法在不同对象上表现出不同的行为。多态是通过继承和方法重写（overriding）实现的，使得代码更加灵活和可扩展。

- 定义与类型：

  - 编译时多态：通过函数重载（overloading）和操作符重载（operator overloading）实现，静态绑定，依赖函数签名（参数类型、数量）。
  - 运行时多态：通过虚函数（virtual function）和继承实现，动态绑定，依赖对象的实际类型。

- 运行时多态的条件：

  - 存在继承关系。
  - 子类重写了父类的虚函数。
  - 对象发生向上转型（将子类对象赋给父类引用或指针）。

- 实现方式：

  通过虚函数：在基类中声明虚函数，子类通过override关键字重写。

  例如：

```c++
class Animal {
public:
    virtual void makeSound() {  // 虚函数
        std::cout << "Some generic animal sound" << std::endl;
    }
};

class Dog : public Animal {
public:
    void makeSound() override {  // 重写父类的虚函数
        std::cout << "Woof!" << std::endl;
    }
};

class Cat : public Animal {
public:
    void makeSound() override {  // 重写父类的虚函数
        std::cout << "Meow!" << std::endl;
    }
};

void animalSound(Animal& animal) {  // 接受 Animal 类型的引用
    animal.makeSound();  // 调用虚函数，根据实际对象类型执行不同的行为
}
```

通过接口：C++中可以通过纯虚函数（pure virtual function）定义抽象基类，类似于接口。

例如：

```c++
class Shape {
public:
    virtual double calculateArea() = 0;  // 纯虚函数
    virtual ~Shape() {}  // 虚析构函数
};

class Rectangle : public Shape {
public:
    double calculateArea() override {
        return length * width;
    }
private:
    double length, width;
};
```

优点：

- 提高代码灵活性和可扩展性，减少条件判断和重复代码。
- 使代码逻辑更清晰，便于维护。

示例：假设有Animal类和其派生类Dog、Cat，当调用animalSound(dog)时，若dog是Dog类型的对象，输出可能为"Woof!"，因为方法调用基于对象的实际类型。

#### 总结

| 特性                  | 定义                               | 实现方式                           | 优点                     | 局限性                     |
| --------------------- | ---------------------------------- | ---------------------------------- | ------------------------ | -------------------------- |
| 封装（Encapsulation） | 隐藏内部细节，保护数据安全         | 私有属性+公共getter/setter方法     | 降低耦合，增强安全性     | 可能增加复杂性，性能开销   |
| 继承（Inheritance）   | 子类复用父类代码，建立层次关系     | extends关键字，super()调用构造函数 | 代码复用，功能扩展       | 耦合性高，父类变化影响子类 |
| 多态（Polymorphism）  | 同一方法不同对象表现不同，动态绑定 | 继承+方法重写，或接口实现          | 提高灵活性，减少重复代码 | 需满足条件，设计复杂       |

### 2、C++中 class 和 struct 有什么区别？

#### 默认访问权限

**class**：默认情况下，class 的成员变量和成员函数的访问权限是 private，即只能在类内部访问。这支持了封装的原则，隐藏实现细节，保护数据安全。

**struct**：默认情况下，struct 的成员变量和成员函数的访问权限是 public，即可以被任何地方直接访问。这更适合用于简单的数据结构，不需要严格的访问控制。

例如：

```c++
struct MyStruct {
    int a;  // 默认 public
    void func() {}  // 默认 public
};

class MyClass {
    int a;  // 默认 private
    void func() {}  // 默认 private
};
```

在 MyStruct 中，a 和 func() 可以直接从外部访问，例如` MyStruct s; s.a = 10; s.func();`。

在 MyClass 中，a 和 func() 默认是私有的，外部无法直接访问，需要通过公共接口（如 getter/setter）来访问。

#### 使用约定

虽然从技术上讲，class 和 struct 在 C++ 中几乎是等价的（都可以包含成员函数、支持继承等），但开发者通常根据以下约定来选择使用哪一个：

struct：通常用于表示简单的数据结构，主要用于存储数据，而不包含复杂的行为。它们类似于 C 语言中的结构体，常用于 POD（Plain Old Data）类型。

例如：

```c++
struct Point {
    int x;
    int y;
};
```

这里，Point 是一个简单的坐标点结构，成员是公开的，适合用于数据传递或存储。

class：通常用于表示具有行为和数据的对象，强调封装、继承和多态等面向对象的特性。class 更适合于需要隐藏实现细节并提供公共接口的场景。

例如：

```c++
class Person {
private:
    std::string name;
public:
    void setName(const std::string& n) { name = n; }
    std::string getName() const { return name; }
};
```

这里，Person 类通过私有成员 name 和公共方法 `setName/getName`实现了封装，外部无法直接修改 name。

#### 继承差异

当 struct 继承自另一个类时，默认的继承方式是 public。这意味着基类的公有和保护成员在派生类中保持其访问权限。

当 class 继承自另一个类时，默认的继承方式是 private。这意味着基类的公有和保护成员在派生类中变为私有。

例如：

```c++
struct Base {
    int x;
};

struct DerivedStruct : Base {  // 默认 public 继承
    // 可以直接访问 Base::x
};

class DerivedClass : Base {  // 默认 private 继承
    // 不能直接访问 Base::x，除非显式指定 public 继承
};
```

在 `DerivedStruct` 中，`Base::x` 是公共的，可以直接访问。

在 `DerivedClass` 中，`Base::x`是私有的，需要通过显式指定 `public Base` 才能保持其公共性。

### 3、如何解决菱形继承问题？

#### 什么是菱形继承问题？

菱形继承是指当一个类（如 D）同时从两个类（如 B 和 C）继承，而 B 和 C 又都从同一个基类（如 A）继承时，形成的继承结构如菱形（A → B 和 C → D）。

这种结构在 C++ 中是合法的，但会导致以下问题：

**成员访问的二义性**：在 D 中访问 A 的成员时，编译器无法确定是通过 B 还是 C 的路径。例如，如果 A 中有一个成员变量 x，在 D 中直接访问 x 时，编译器会报错，因为它不知道是调用 B 中的 x 还是 C 中的 x。

例如：

```c++
class A {
public:
    int x;
};

class B : public A {};
class C : public A {};

class D : public B, public C {
public:
    void printX() {
        std::cout << x << std::endl;  // 编译错误：二义性
    }
};
```

在上述代码中，x 的访问会失败，因为编译器无法确定是 B 的 x 还是 C 的 x。

**数据冗余**：A 的数据成员在 D 中会被复制两次（一次通过 B，一次通过 C），浪费内存空间。例如，如果 A 中有一个成员变量 x，那么 D 中会同时拥有两个 x（一个来自 B，一个来自 C），这不仅占用额外空间，还可能导致数据不一致。

#### 如何解决？

解决菱形继承问题的有效方法是使用虚继承（Virtual Inheritance）。虚继承通过在 B 和 C 继承 A 时添加 virtual 关键字，确保 A 的实例在 D 中只有一份，从而消除二义性和数据冗余。

**虚继承的实现**：

在 B 和 C 继承 A 时，使用 virtual 关键字，例如：

```c++
class B : virtual public A {};
class C : virtual public A {};
```

这样，B 和 C 共享同一个 A 的实例，而不是各自维护一份。

**工作原理**：

- 虚继承引入了“虚基类表”（Virtual Base Table），编译器通过该表管理虚基类的实例，确保所有派生类共享同一个 A 的实例。
- 虚继承会将 A 的数据成员初始化由最派生类（D）负责，避免重复初始化。

**示例代码**：

```c++
class A {
public:
    int x;
};

class B : virtual public A {};
class C : virtual public A {};

class D : public B, public C {
public:
    void printX() {
        std::cout << x << std::endl;  // 现在可以直接访问 x 了
    }
};
```

在这个例子中，D 中只有一份 A 的实例，访问 x 时不会产生二义性。

**效果**：

- 消除二义性：D 可以直接访问 A 的成员，无需使用作用域解析运算符（如 B::x 或 C::x）。
- 避免数据冗余：根据内存分析，虚继承减少了内存占用，例如从 8 字节减少到 4 字节。

#### 非虚继承和虚继承的差异

| 特性           | 非虚继承                       | 虚继承                         |
| -------------- | ------------------------------ | ------------------------------ |
| 数据成员的实例 | 多个（数据冗余）               | 单个（共享实例）               |
| 成员访问       | 二义性，需使用作用域解析运算符 | 无二义性，可直接访问           |
| 内存占用       | 较高（例如 8 字节）            | 较低（例如 4 字节）            |
| 构造函数调用   | 自动调用，可能重复             | 由最派生类显式调用，单次初始化 |
| 性能开销       | 较低                           | 较高（虚基类表管理）           |

### 4、C++的多态有几种实现方式？

#### 编译时多态

编译时多态也称为静态多态（Static Polymorphism），是通过函数重载（Function Overloading）和运算符重载（Operator Overloading）实现的。这种多态在编译阶段就确定了具体调用哪个函数，因此执行效率较高。

##### 函数重载

函数重载允许在同一个作用域中定义多个同名函数，但这些函数的参数列表（参数类型、数量或顺序）必须不同。编译器根据函数调用时的参数类型来选择调用哪个函数。

```c++
class MathOperations {
public:
    int add(int a, int b) {
        return a + b;
    }
    float add(float a, float b) {
        return a + b;
    }
    float add(int a, float b) {
        return a + b;
    }
};

int main() {
    MathOperations math;
    cout << math.add(1, 2) << endl;        // 调用 int add(int, int)
    cout << math.add(1.5f, 2.5f) << endl;  // 调用 float add(float, float)
    cout << math.add(1, 2.5f) << endl;     // 调用 float add(int, float)
    return 0;
}
```

在这个例子中，add函数根据参数类型被重载。编译器在编译时根据参数类型确定调用哪个版本的add函数。

##### 运算符重载

运算符重载允许重定义C++中的运算符（如+, -, ==等）以适应用户自定义的数据类型。

```c++
class Complex {
public:
    float real, imag;
    Complex(float r, float i) : real(r), imag(i) {}
    Complex operator+(const Complex& other) {
        return Complex(real + other.real, imag + other.imag);
    }
};

int main() {
    Complex c1(1.0, 2.0);
    Complex c2(3.0, 4.0);
    Complex c3 = c1 + c2;  // 调用 operator+
    cout << "Real: " << c3.real << ", Imag: " << c3.imag << endl;
    return 0;
}
```

这里，+运算符被重载，用于支持两个Complex对象的加法。

**特点**：

- 编译时多态在编译阶段就确定函数调用，因此没有运行时开销。
- 适用于不需要动态行为的场景。

#### 运行时多态

运行时多态也称为动态多态（Dynamic Polymorphism），是通过虚函数（Virtual Functions）和函数重写（Function Overriding）实现的。这种多态在程序运行时根据对象的实际类型决定调用哪个函数，因此也称为晚绑定（Late Binding）。

##### 虚函数和函数重写

虚函数是通过在基类中使用virtual关键字声明的函数。派生类可以重写（override）这些函数。当通过基类指针或引用调用虚函数时，实际调用的是派生类的版本。

```c++
class Shape {
public:
    virtual float area() = 0;  // 纯虚函数，Shape为抽象类
};

class Circle : public Shape {
private:
    float radius;
public:
    Circle(float r) : radius(r) {}
    float area() override {
        return 3.14 * radius * radius;
    }
};

class Rectangle : public Shape {
private:
    float length, width;
public:
    Rectangle(float l, float w) : length(l), width(w) {}
    float area() override {
        return length * width;
    }
};

int main() {
    Shape* shape1 = new Circle(5.0);
    Shape* shape2 = new Rectangle(4.0, 5.0);
    cout << shape1->area() << endl;  // 调用 Circle::area()
    cout << shape2->area() << endl;  // 调用 Rectangle::area()
    delete shape1;
    delete shape2;
    return 0;
}
```

在这个例子中，Shape是抽象基类，定义了一个纯虚函数area()。Circle和Rectangle重写了area()函数。

当通过Shape*指针调用area()时，运行时根据实际对象类型（Circle或Rectangle）决定调用哪个函数。

**特点**：

- 运行时多态需要通过基类指针或引用调用虚函数。
- 虚函数的调用需要额外的运行时开销（如虚表指针），但提供了高度的灵活性。

#### 模板的多态

除了以上两种主流的多态实现方式，C++还支持一种称为**模板（Templates）**的泛型编程方式，sometimes被视为一种编译时多态。模板允许编写可以适用于不同数据类型的代码，而无需显式地为每种类型编写单独的函数或类。

```c++
template <typename T>
T max(T a, T b) {
    return (a > b) ? a : b;
}

int main() {
    cout << max(1, 2) << endl;        // 调用 max<int>
    cout << max(1.5f, 2.5f) << endl;  // 调用 max<float>
    return 0;
}
```

max函数是一个模板函数，它可以适用于任何支持>运算符的类型。编译器在编译时根据实际类型生成特定的函数版本。

#### 三种多态实现的对比

| 特性         | 编译时多态（函数重载/运算符重载） | 运行时多态（虚函数）   | 模板多态   |
| ------------ | --------------------------------- | ---------------------- | ---------- |
| 实现方式     | 函数重载、运算符重载              | 虚函数、函数重写       | 模板       |
| 绑定时间     | 编译时                            | 运行时                 | 编译时     |
| 性能开销     | 低                                | 高（虚表开销）         | 低         |
| 灵活性       | 有限                              | 高                     | 高（泛型） |
| 典型使用场景 | 静态行为，参数类型不同            | 动态行为，对象类型不同 | 泛型编程   |

### 5、C++11有哪些新特性？

#### 核心语言特性

C++11在核心语言层面引入了许多重要特性，这些特性极大地增强了语言的表达能力和灵活性。

1. Auto关键字（自动类型推断）
   - 描述：auto关键字允许编译器根据初始化表达式自动推断变量的类型，无需显式声明。
   - 语法与示例：auto x = 5; 编译器会推断x为int类型；对于复杂类型，如`auto it = container.begin()`;，避免了冗长的类型名称。
   - 用途：减少了显式类型声明的需要，简化了代码，尤其在使用迭代器、模板或复杂返回类型时。
   - 意义：提高了代码的可读性和简洁性，减少了人为错误。例如，在使用STL容器时，auto可以避免手动指定迭代器类型。
   - 相关讨论：研究表明，auto的使用显著降低了代码复杂性，但需注意过度使用可能导致类型推断不明确的问题。
2. Lambda表达式（匿名函数）
   - 描述：Lambda表达式允许在代码中定义匿名函数，可以捕获外部变量，支持闭包。
   - 语法与示例：`[]() { return 42; }` 定义一个无捕获、无参数的Lambda；`[&](int x) { return x + y; }` 捕获外部变量y。
   - 用途：常用于短期使用的函数，如在算法中传递回调函数，例如`std::sort(v.begin(), v.end(), [](int a, int b) { return a < b; });`。
   - 意义：增强了函数式编程支持，简化了代码，特别是在STL算法中使用。研究表明，Lambda表达式显著提高了代码的可维护性。
   - 相关讨论：Lambda的捕获方式（如按值或按引用）需要谨慎选择，以避免悬空引用或性能问题。
3. 右值引用和移动语义
   - 描述：右值引用（&&）允许高效地转移资源，而非复制，支持移动构造函数和移动赋值运算符。
   - 语法与示例：`std::string s1 = "hello"; std::string s2 = std::move(s1);` 将s1的内容移动到s2，避免复制。
   - 用途：减少了大对象的复制操作，提高了性能，尤其在处理大容器或复杂对象时。
   - 意义：优化了资源管理，减少了不必要的内存分配和复制。研究表明，移动语义显著提升了性能，但可能增加学习曲线。
   - 相关讨论：部分开发者认为右值引用的引入增加了语言复杂度，存在争议，但总体被认为是对C++的重大改进。
4. 变参模板
   - 描述：允许模板接受任意数量的参数，使用参数包实现。
   - 语法与示例：`template<typename... Args> void print(Args... args);` 可以接受任意数量的参数，并通过展开处理。
   - 用途：增强了泛型编程的能力，使模板更加灵活，例如实现可变参数的函数模板。
   - 意义：简化了模板编程，提高了代码的可重用性。研究表明，变参模板是C++11泛型编程的重要组成部分。
   - 相关讨论：变参模板的展开需要注意递归和性能问题，但其灵活性被广泛认可。
5. 静态断言
   - 描述：在编译时检查条件是否满足，如果不满足，则编译失败。
   - 语法与示例：`static_assert(sizeof(int) == 4, "int is not 4 bytes");` 如果int的大小不为4字节，编译失败。
   - 用途：帮助开发者在编译阶段捕获错误，提高代码的可靠性。
   - 意义：增强了编译时检查，减少了运行时错误。研究表明，静态断言在模板元编程中尤为有用。
   - 相关讨论：静态断言的广泛使用提高了代码的健壮性，但需注意条件表达式的复杂性。
6. Constexpr（常量表达式）
   - 描述：允许函数和变量在编译时求值，支持编译时计算。
   - 语法与示例：`constexpr int square(int x) { return x * x; }` 定义一个编译时可计算的函数。
   - 用途：用于定义常量和优化性能-critical的代码，例如数组大小的计算。
   - 意义：提高了编译时计算能力，优化了性能。研究表明，constexpr在嵌入式系统和性能敏感的应用中尤为重要。
   - 相关讨论：constexpr的引入使得C++更接近函数式编程，但需注意函数的纯度和副作用。
7. 线程局部存储
   - 描述：支持每个线程拥有自己的变量副本，使用thread_local关键字声明。
   - 语法与示例：`thread_local int thread_id; `每个线程有独立的thread_id值。
   - 用途：确保线程安全，避免线程间数据冲突。
   - 意义：简化了多线程编程，特别是在需要线程私有数据的场景。研究表明，这是C++11多线程支持的重要组成部分。
   - 相关讨论：线程局部存储的性能开销需注意，但在多线程应用中不可或缺。
8. 初始化列表
   - 描述：提供统一的初始化语法，支持容器和自定义类型的初始化，使用`std::initializer_list`。
   - 语法与示例：`std::vector<int> v = {1, 2, 3};` 使用初始化列表初始化向量。
   - 用途：简化了对象的初始化，尤其是对容器的初始化。
   - 意义：提高了代码的可读性和一致性。研究表明，初始化列表统一了C++的初始化语法，减少了歧义。
   - 相关讨论：初始化列表的引入解决了C++03中初始化方式不一致的问题，但需注意与旧式初始化（如括号初始化）的兼容性。
9. Noexcept（异常规范）
   - 描述：声明函数不会抛出异常，使用noexcept关键字。
   - 语法与示例：`void func() noexcept;` 声明func不会抛出异常。
   - 用途：帮助编译器优化代码，并明确函数的行为。
   - 意义：增强了代码的安全性和性能。研究表明，noexcept在性能敏感的应用中尤为重要。
   - 相关讨论：noexcept的引入提高了异常处理的明确性，但需注意其与旧式throw()的兼容性。
10. Override和Final
    - 描述：override确保函数正确覆盖了基类的虚函数，final禁止进一步覆盖。
    - 语法与示例：`virtual void func() override;` 确保func覆盖了基类的虚函数。
    - 用途：提高了虚函数的安全性，防止意外创建新的虚函数。
    - 意义：增强了代码的可维护性和安全性。研究表明，这两个关键字显著降低了虚函数相关的错误。
    - 相关讨论：override和final的引入被认为是C++11对面向对象编程的重要改进。

#### 标准库特性

C++11不仅在核心语言上进行了改进，还对标准库进行了扩展，引入了一系列新的容器、算法和工具。

1. std::thread（线程支持）
   - 描述：提供标准化的线程创建和管理，使用std::thread类。
   - 语法与示例：`std::thread t(func);` 创建一个新线程运行func。
   - 用途：简化了多线程编程，支持并发执行。
   - 意义：为C++提供了原生的多线程支持，填补了C++03的空白。研究表明，这是C++11最重要的库扩展之一。
   - 相关讨论：std::thread的引入显著简化了多线程开发，但需注意线程安全和死锁问题。
2. std::chrono（时间库）
   - 描述：提供类型安全的时间和持续时间处理，使用`std::chrono`命名空间。
   - 语法与示例：`std::chrono::seconds duration(10);` 定义10秒的持续时间。
   - 用途：替代了旧的、不安全的时间处理方法，如time_t。
   - 意义：提高了时间处理的准确性和可移植性。研究表明，`std::chrono`在实时系统和游戏开发中尤为有用。
   - 相关讨论：`std::chrono`的引入统一了时间处理接口，但需注意其与旧API的兼容性。
3. std::regex（正则表达式）
   - 描述：将正则表达式引入标准库，使用`std::regex`类。
   - 语法与示例：`std::regex r("pattern");` 定义正则表达式模式。
   - 用途：增强了文本处理能力，支持复杂的字符串匹配。
   - 意义：提供了强大的字符串处理工具。研究表明，`std::regex`在数据解析和验证中非常实用。
   - 相关讨论：`std::regex`的性能在某些场景下可能不如第三方库，但其标准性被广泛认可。
4. 智能指针（Smart Pointers）
   - 描述：包括`std::unique_ptr`、`std::shared_ptr`和`std::weak_ptr`，用于自动管理对象的生命周期。
   - 语法与示例：`std::unique_ptr<int> ptr(new int(5))`; 自动管理int对象的内存。
   - 用途：减少内存泄漏，提高资源管理的安全性。
   - 意义：提供了更安全的内存管理方式，减少了手动delete的错误。研究表明，智能指针是C++11资源管理的重要改进。
   - 相关讨论：智能指针的引入显著降低了内存管理复杂度，但需注意循环引用问题。
5. 无序容器
   - 描述：包括`std::unordered_map`和`std::unordered_set`，基于哈希表实现。
   - 语法与示例：`std::unordered_map<std::string, int> map;` 定义一个哈希映射。
   - 用途：提供快速的查找和插入操作，时间复杂度为O(1)。
   - 意义：提高了容器的性能，尤其在需要快速访问时。研究表明，无序容器在高性能应用中非常重要。
   - 相关讨论：无序容器的性能依赖于哈希函数的质量，需注意哈希冲突问题。
6. std::tuple（元组）
   - 描述：支持固定大小、异构类型的集合，使用`std::tuple`类。
   - 语法与示例：`std::tuple<int, float> t(1, 2.5f);` 定义一个包含int和float的元组。
   - 用途：用于返回多个值或存储不同类型的元素。
   - 意义：提供了更灵活的数据结构，简化了多返回值函数的实现。研究表明，std::tuple在函数式编程中非常实用。
   - 相关讨论：元组的访问需要`std::get`或`std::tie`，可能增加代码复杂性。
7. std::function（函数对象）
   - 描述：类型擦除的函数对象，可以存储任意可调用对象，使用std::function类。
   - 语法与示例：`std::function<void(int)> f = [](int x) { std::cout << x; };` 定义一个函数对象。
   - 用途：增强了函数指针和回调的灵活性，支持Lambda、函数指针和函数对象。
   - 意义：简化了函数式编程，特别是在事件处理和回调场景。研究表明，std::function提高了代码的可扩展性。
   - 相关讨论：std::function的类型擦除可能带来性能开销，需注意使用场景。

#### 其他重要特性

以下是一些其他值得注意的特性，涵盖了语言和库的多个方面：

- 范围-based for循环
  - 描述：简化了容器的迭代，使用`for(auto& elem : container)`语法。
  - 示例：`for(auto& elem : v) { std::cout << elem << " "; }`
  - 意义：提高了代码的可读性，减少了迭代器的使用。
- 原始字符串和Unicode字符串
  - 描述：支持原始字符串和Unicode字符串，使用R"(...)"和u8"..."等。
  - 示例：`const char* s = R"(raw string)";` 定义原始字符串。
  - 意义：简化了字符串处理，尤其是对包含特殊字符的字符串。
- 类型特征（Type Traits）
  - 描述：提供编译时类型信息和操作，使用std::is_same等。
  - 示例：`std::is_same<int, int>::value `为true。
  - 意义：增强了泛型编程的能力，支持模板元编程。
- 原子操作（Atomic Operations）
  - 描述：支持无锁的多线程编程，使用std::atomic类。
  - 示例：`std::atomic<int> atomic_var(0);`
  - 意义：提供了线程安全的编程工具，减少了锁的使用。

### 6、介绍一下分段式内存和页式内存。

#### 分段式内存

##### 定义与工作原理

分段式内存是一种将计算机主内存划分为多个段（Segment）的内存管理技术。每个段是一个逻辑上的独立单元，通常对应程序中的一个功能模块，如代码段、数据段、堆栈段等。每个段都有自己的段号（Segment ID）和段内偏移量（Offset），通过段号和偏移量共同构成一个内存地址。

- 地址结构：一个内存地址由两部分组成：段号和段内偏移量。段号用于标识段，偏移量用于标识段内的具体位置。
- 内存管理：操作系统为每个段维护一张段表（Segment Table），其中记录了每个段的基址（Base Address）和段长（Limit）。当程序访问一个地址时，内存管理单元（MMU）会使用段号从段表中查找基址，然后将偏移量与基址相加，得到物理地址。
- 虚拟内存支持：在分段式内存中，每个段可以被标记为驻留在主内存还是在辅存中。如果段不在主内存中，操作系统会触发页面调入（如果支持分页）或整个段的调入（如果不支持分页）。

##### 优点

- 逻辑组织：段是程序的逻辑单位（如函数、数组、数据结构），使得内存管理更符合程序员的思维方式。
- 灵活性：段的大小可以根据需要调整，适合不同大小的程序模块。
- 内存保护：每个段可以设置独立的权限（读、写、执行），实现了更细粒度的内存保护。
- 共享：多个程序可以共享同一个段（如共享库），减少了内存浪费。

##### 缺点

- 碎片化：在不支持分页的分段式系统中，交换整个段可能导致外部碎片（External Fragmentation），即内存中有足够的总空间但无法找到连续的空闲块来放置一个段。
- 复杂性：实现分段式内存需要更复杂的硬件和软件支持，如段表管理和地址翻译。
- 性能开销：地址翻译过程需要额外的计算和表查找，可能引入性能开销。

##### 应用

分段式内存在早期的操作系统中较为常见，如Multics和某些大型机系统。在现代系统中，分段式内存通常与分页式内存结合使用，以发挥两者的优势。

#### 页式内存

##### 定义与工作原理

页式内存是一种将内存划分为固定大小的页（Page）的内存管理技术。每个页的大小由系统决定（通常是2KB、4KB或更大），并被映射到物理内存中的帧（Frame）。页式内存是实现虚拟内存的主要方式。

- 地址结构：一个内存地址由两部分组成：页号（Page Number）和页内偏移量（Offset）。页号用于标识页，偏移量用于标识页内的具体位置。
- 内存管理：操作系统为每个进程维护一张页表（Page Table），其中记录了每个页的物理帧号（Frame Number）和权限信息。当程序访问一个地址时，MMU使用页号从页表中查找物理帧号，然后将偏移量与帧号相加，得到物理地址。
- 虚拟内存支持：页式内存支持虚拟内存，操作系统可以将不常用的页交换到辅存（如硬盘），从而扩展了可用内存空间。

##### 优点

- 高效的内存利用：固定大小的页简化了内存管理，避免了外部碎片化问题。
- 虚拟内存支持：页式内存是实现虚拟内存的基础，允许进程拥有比物理内存更大的地址空间。
- 简单实现：由于所有页大小相同，页表管理和地址翻译相对简单。
- 无外部碎片：页式内存不会产生外部碎片，只可能存在内部碎片（Internal Fragmentation），即页内未使用的空间。

##### 缺点

- 内部碎片：由于页大小固定，如果一个页中只有部分数据被使用，剩余空间会浪费。
- 逻辑不一致：页是物理单位，而不是逻辑单位，程序员无法直接感知页的边界，这可能导致编程复杂性。
- 共享困难：页式内存不适合共享特定内存区域，因为页是固定的物理单位。

##### 应用

页式内存是现代操作系统中最常用的内存管理技术，几乎所有支持虚拟内存的操作系统（如Windows、Linux）都使用了页式内存。页式内存通常与分段式内存结合使用，以提供更灵活的内存管理。

#### 分段式内存与页式内存的比较

| 特征         | 分段式内存               | 页式内存                 |
| ------------ | ------------------------ | ------------------------ |
| 分区单位     | 变量大小的段（逻辑单位） | 固定大小的页（物理单位） |
| 管理方式     | 由编译器和程序员管理     | 由操作系统管理           |
| 地址结构     | 段号 + 段内偏移量        | 页号 + 页内偏移量        |
| 数据结构     | 段表                     | 页表                     |
| 碎片化类型   | 外部碎片化（无分页时）   | 内部碎片化               |
| 速度         | 较慢（地址翻译更复杂）   | 较快（地址翻译更简单）   |
| 程序员可见性 | 高（段是逻辑单位）       | 低（页是物理单位）       |
| 共享         | 容易（段可以共享）       | 困难（页不易共享）       |
| 保护         | 容易（每个段有独立权限） | 较难（需要额外机制）     |
| 大小限制     | 无固定大小               | 页大小固定               |
| 系统效率     | 较高（逻辑组织更灵活）   | 较低（物理组织更 rigid） |

### 7、当内存满了需要换出时，这个工作由谁来做？

#### 内存交换的定义与必要性

内存交换是一种内存管理技术，当RAM无法容纳所有活跃进程的数据时，操作系统会将不活跃或低优先级的内存页转移到交换空间（Swap Space），通常位于硬盘或SSD上。这种机制是虚拟内存（Virtual Memory）实现的核心部分，允许系统运行超过物理RAM容量的进程。

- 虚拟内存：通过将部分数据存储在二级存储中，操作系统可以让应用程序认为可用内存比实际RAM更多。
- 交换空间：在Linux中称为“交换分区”（Swap Partition）或“交换文件”（Swap File），在Windows中称为“页文件”（Page File），在macOS中也有类似机制。

内存交换在内存不足时至关重要，但由于硬盘读写速度远低于RAM，频繁交换可能导致系统性能下降。

#### 谁负责内存交换

内存交换的工作由**操作系统的内核或内存管理子系统**负责。具体来说：

- 操作系统内核：负责监控系统的内存使用情况，并决定何时进行交换。
- 内存管理子系统：具体负责实现交换逻辑，包括选择哪些数据需要交换、管理交换空间等。

不同操作系统的具体实现如下：

- Linux：内核直接管理交换空间。内核会根据内存使用情况和系统配置（如swappiness参数）决定何时交换数据。Swappiness参数控制了内核多快地将内存页从RAM换出到交换空间。例如，swappiness=0表示尽量不交换，swappiness=100表示非常积极地交换。
- Windows：虚拟内存管理器（Virtual Memory Manager, VMM）负责管理页文件（Page File）和交换文件（Swap File）。当RAM不足时，VMM会将不活跃的数据交换到页文件中。
- macOS：内核的内存管理单元（Memory Management Unit, MMU）处理分页和交换，确保系统在内存不足时仍然能够运行。

研究表明，内存交换是操作系统核心功能的一部分，硬件（如CPU或MMU）不直接参与交换决策，但硬件支持（如地址翻译）是实现交换的基础。

#### 内存交换的工作原理

当RAM满了，操作系统会执行以下步骤进行交换：

1. 监控内存使用：操作系统持续监控RAM的使用情况，通过内存管理子系统跟踪每个内存页的使用频率和活跃度。
2. 选择交换目标：操作系统会选择不活跃的内存页（如长时间未使用的页面）作为交换目标。常见的算法包括“最近最少使用”（LRU, Least Recently Used）、“先进先出”（FIFO, First-In-First-Out）等。
3. 写入交换空间：将选定的内存页写入交换空间（如页文件或交换分区）。这一过程涉及硬盘或SSD的读写操作。
4. 释放RAM：一旦数据被写入交换空间，相应的RAM空间就被释放出来，供其他进程使用。
5. 读取交换数据：当需要访问被交换出的数据时，操作系统会从交换空间读取数据，并可能再次交换出其他数据以腾出空间。

这一过程虽然有效，但由于硬盘或SSD的访问速度远低于RAM，频繁交换会导致系统响应时间显著增加。

#### 内存交换的优缺点

- 优点：
  - 扩展内存：通过虚拟内存，系统可以运行比物理RAM更多的进程，增强了多任务处理能力。
  - 稳定性：防止系统因内存不足而崩溃，确保关键进程的连续运行。
- 缺点：
  - 性能下降：硬盘或SSD的读写速度远低于RAM，频繁交换会导致系统响应变慢，可能引发“交换地狱”（Thrashing）现象。
  - 碎片化：长时间使用交换空间可能导致碎片化，影响效率，尤其是在传统硬盘上。

交换的性能影响是用户优化内存管理的重要考虑因素。

#### 现代操作系统的内存交换优化

现代操作系统在内存交换方面引入了一些优化，以减少性能损失：

- Linux：

  - ZRAM：通过压缩数据减少对交换空间的依赖，提高效率。例如，ZRAM将交换数据压缩后存储在RAM中，减少了硬盘I/O。
  - Swap Files：支持使用文件而非专用分区作为交换空间，提供更大的灵活性，方便动态调整。

- Windows：

  - Swap File：除了传统的页文件（Page File），Windows 10及后续版本引入了Swap File，用于存储Windows Store应用的数据。
  - 自动管理：Windows会自动调整页文件大小，但用户也可以手动设置，推荐设置为RAM的1.5-2倍。

- macOS：

  压缩内存：在内存不足时，macOS会先尝试压缩内存数据，而不是立即交换到硬盘，减少性能损失。

研究表明，这些优化显著提高了交换的效率，但仍需用户根据具体需求调整配置。