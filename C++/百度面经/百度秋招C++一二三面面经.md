来源：https://www.nowcoder.com/discuss/550679042400317440

## 一面

1、虚函数？

1. 什么是虚函数：

虚函数是在基类中声明的，而在派生类中进行重写（override）的函数。通过使用 `virtual` 关键字声明一个函数为虚函数，它使得在运行时能够动态地确定调用的是哪个版本的函数。

```C++
class Base {
public:
    virtual void show() {
        std::cout << "Base class\n";
    }
};

class Derived : public Base {
public:
    void show() override {
        std::cout << "Derived class\n";
    }
};
```

2. 虚函数的作用：

   - 实现多态性（Polymorphism）： 允许通过基类指针或引用调用派生类对象的函数，根据实际对象的类型选择相应的函数实现。

   - 运行时绑定（Runtime Binding）： 虚函数通过表格（虚函数表）的方式实现，使得在运行时动态地绑定函数调用。

3. 虚函数表（vtable）：

每个含有虚函数的类都有一个虚函数表，其中存储了虚函数的地址。对象的内存布局中包含一个指向虚函数表的指针。派生类的虚函数表包含基类的虚函数表，并在适当的位置添加或替换新的虚函数地址。

4. 纯虚函数：

纯虚函数是一个在基类中声明但没有提供实现的虚函数，它通过在声明中使用 `= 0` 来标识。类含有纯虚函数的类被称为抽象类，不能被实例化。派生类必须实现纯虚函数，否则也会变为抽象类。

```C++
class AbstractBase {
public:
    virtual void pureVirtualFunction() = 0;
};
```

5. 虚析构函数：

如果基类的析构函数是虚函数，当通过基类指针删除派生类对象时，会调用派生类的析构函数。这是为了确保正确的对象销毁。

```C++
class Base {
public:
    virtual ~Base() {
        std::cout << "Base destructor\n";
    }
};

class Derived : public Base {
public:
    ~Derived() override {
        std::cout << "Derived destructor\n";
    }
};
```

2、内存池？

内存池（Memory Pool）是一种用于管理内存分配的机制，它通过预先分配一块固定大小的内存块，然后根据需要从这个内存块中分配小块内存给应用程序使用，以减少内存碎片和提高内存分配的效率。

1. 内存池的目的：

   - 减少内存碎片： 内存池通过预先分配一块连续的内存，避免了频繁的小块内存分配和释放，减少了内存碎片的产生。

   - 提高内存分配效率： 内存池可以通过一次分配多次使用，减少了内存分配的开销，提高了内存分配的效率。

2. 内存池的实现方式：

   - 固定大小的块： 内存池通常将内存划分成大小相等的块，每个块都可以独立地分配给应用程序。

   - 预分配内存： 内存池在初始化时会预分配一定大小的内存块，并将这些块组织成链表或其他数据结构。

   - 分配策略： 内存池可以采用不同的分配策略，例如首次适应、最佳适应、最差适应等。

3. 内存池的优势：

   - 降低内存碎片： 内存池通过将内存分配和释放控制在一个固定大小的块中，有效地减少了内存碎片的产生。

   - 提高性能： 内存池减少了频繁的内存分配和释放操作，降低了内存管理的开销，提高了程序性能。

4. 使用场景：

   - 频繁地小块内存分配： 内存池特别适用于需要频繁地分配和释放小块内存的场景，如网络编程、嵌入式系统等。

   - 实时系统： 在实时系统中，内存分配的延迟是一个重要的考量因素，内存池可以帮助降低内存分配的延迟。

5. 内存池的实现步骤：

   - 预分配内存块： 在初始化时，内存池预分配一块固定大小的内存。

   - 组织内存块： 将预分配的内存划分成大小相等的块，并组织成链表或其他数据结构。

   - 分配内存： 当应用程序需要内存时，从内存池中分配一个块。

   - 释放内存： 当应用程序不再需要内存时，将内存块释放回内存池。

6. 简单实现代码：

```C++
#include <iostream>
#include <vector>

class MemoryPool {
private:
    std::vector<void*> blocks;
    size_t block_size;

public:
    MemoryPool(size_t size) : block_size(size) {
        // 初始化时预分配一定数量的内存块
        allocateBlock();
    }

    ~MemoryPool() {
        // 释放所有内存块
        for (void* block : blocks) {
            delete[] static_cast<char*>(block);
        }
    }

    void* allocate() {
        // 从当前内存块中分配内存
        if (blocks.empty() || block_size - usedSpace() < sizeof(void*)) {
            allocateBlock();
        }

        void* allocated = static_cast<char*>(blocks.back()) + usedSpace();
        incrementUsedSpace(sizeof(void*));
        return allocated;
    }

    void deallocate(void* ptr) {
        // 内存释放回内存池
        // 此处简化实现，不进行内存复用
    }

private:
    void allocateBlock() {
        // 分配一个新的内存块
        void* block = new char[block_size];
        blocks.push_back(block);
        resetUsedSpace();
    }

    size_t usedSpace() const {
        // 计算当前内存块已使用的空间
        if (!blocks.empty()) {
            return reinterpret_cast<size_t>(blocks.back());
        }
        return 0;
    }

    void incrementUsedSpace(size_t size) {
        // 增加已使用空间的偏移量
        if (!blocks.empty()) {
            blocks.back() = static_cast<void*>(static_cast<char*>(blocks.back()) + size);
        }
    }

    void resetUsedSpace() {
        // 重置已使用空间的偏移量
        if (!blocks.empty()) {
            blocks.back() = nullptr;
        }
    }
};
```

3、C++11新特性？ 

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

4、什么时候用auto？

1. 类型较长或复杂： 当变量的类型较长或复杂时，使用 `auto` 可以简化代码并提高可读性。

```C++
std::map<std::string, std::vector<int>> myMap;
// 使用auto简化迭代器的声明
for (auto it = myMap.begin(); it != myMap.end(); ++it) {
    // ...
}
```

2. 模板编程： 在模板编程中，使用 `auto` 可以避免重复书写类型，使代码更通用。

```C++
template <typename T, typename U>
auto add(T x, U y) -> decltype(x + y) {
    return x + y;
}
```

3. 迭代器和范围-based for 循环： 在使用迭代器和范围-based for 循环时， `auto` 可以简化类型声明。

```C++
std::vector<int> numbers = {1, 2, 3, 4, 5};
// 使用auto声明迭代器
for (auto it = numbers.begin(); it != numbers.end(); ++it) {
    // ...
}
// 使用范围-based for 循环
for (const auto& num : numbers) {
    // ...
}
```

4. 函数返回类型推导： 在函数返回类型不明确的情况下，使用 `auto` 可以方便地进行类型推导。

```C++
auto multiply(int x, double y) -> decltype(x * y) {
    return x * y;
}
```

5. 方便迭代器类型： 当使用模板或容器类型时， `auto` 可以方便地获取迭代器的类型，而无需显式指定。

```C++
std::vector<int> numbers = {1, 2, 3, 4, 5};
auto it = numbers.begin(); // it的类型是std::vector<int>::iterator
```

5、智能指针?

1. `std::shared_ptr`： 共享指针。多个 `std::shared_ptr` 可以共享同一个对象，并且会跟踪对象的引用计数。当引用计数变为零时，对象会被删除。

```C++
#include <memory>

int main() {
    std::shared_ptr<int> sharedPtr = std::make_shared<int>(42);

    // 可以通过 std::shared_ptr 构造函数或 make_shared 函数创建
    std::shared_ptr<int> anotherSharedPtr(sharedPtr);

    // 引用计数减1
    sharedPtr.reset();

    // 另一个共享指针仍然有效，引用计数为1
    return 0; // 离开作用域，引用计数减1，对象被销毁
}
```

2. `std::unique_ptr`： 独占指针。一个 `std::unique_ptr` 拥有对其指向的对象的唯一所有权。当 `std::unique_ptr` 被销毁时，它指向的对象也会被销毁。

```C++
#include <memory>

int main() {
    std::unique_ptr<int> uniquePtr = std::make_unique<int>(42);

    // 不允许直接赋值
    // std::unique_ptr<int> anotherUniquePtr = uniquePtr; // 错误，不允许直接赋值

    // 通过 std::move 进行所有权转移
    std::unique_ptr<int> anotherUniquePtr = std::move(uniquePtr);

    // uniquePtr 不再拥有对象的所有权，变为 nullptr
    // 在 uniquePtr 销毁时，不会删除对象，因为所有权已经转移
    return 0; // 离开作用域，anotherUniquePtr 被销毁，对象也被销毁
}
```

3. `std::weak_ptr`： 弱引用指针。`std::weak_ptr` 是为了解决 `std::shared_ptr` 的循环引用问题而引入的。`std::weak_ptr` 既不增加引用计数，也不影响对象的生命周期。

```C++
#include <memory>

int main() {
    std::shared_ptr<int> sharedPtr = std::make_shared<int>(42);

    std::weak_ptr<int> weakPtr = sharedPtr;

    // 使用 lock 函数获取 shared_ptr
    if (auto shared = weakPtr.lock()) {
        // 可以安全地使用 shared 指针
    }

    // 当 sharedPtr 被销毁后，weakPtr 不再有效
    return 0;
}
```

4. `std::auto_ptr`： C++11 之前引入的智能指针，被 `std::unique_ptr` 取代。`std::auto_ptr` 存在一些问题，例如对数组的管理不当，会导致不确定的行为，因此不建议使用。

6、std::map和std::unordered_map?

1. `std::map`： 是一个有序的关联容器，它使用红黑树实现。这意味着 `std::map` 中的元素是按键的升序顺序进行排序的。插入、查找和删除操作的时间复杂度是 O(log n)。

```C++
#include <iostream>
#include <map>

int main() {
    std::map<int, std::string> myMap;

    // 插入键值对
    myMap[1] = "One";
    myMap[3] = "Three";
    myMap[2] = "Two";

    // 遍历输出
    for (const auto& pair : myMap) {
        std::cout << pair.first << ": " << pair.second << std::endl;
    }

    return 0;
}
```

2. `std::unordered_map`： 是一个无序的关联容器，它使用哈希表实现。这意味着 `std::unordered_map` 中的元素没有特定的顺序。插入、查找和删除操作的平均时间复杂度是 O(1)。但需要注意，对于某些特殊情况，最坏情况下的时间复杂度可能为 O(n)。

```C++
#include <iostream>
#include <unordered_map>

int main() {
    std::unordered_map<int, std::string> myUnorderedMap;

    // 插入键值对
    myUnorderedMap[1] = "One";
    myUnorderedMap[3] = "Three";
    myUnorderedMap[2] = "Two";

    // 遍历输出（无序）
    for (const auto& pair : myUnorderedMap) {
        std::cout << pair.first << ": " << pair.second << std::endl;
    }

    return 0;
}
```

7、Go map有序吗？底层

在 Go 语言中，`map` 是无序的，即不保证元素的遍历顺序和插入顺序相同。在 Go 的 `map` 实现中，不提供对元素的排序保证。

底层实现是哈希表。哈希表是一种高效的数据结构，用于存储键值对，并且支持常数时间的插入、删除和查找操作。

具体来说，Go 语言的 `map` 使用了开放寻址法和二次探查来解决哈希冲突。当发生哈希冲突时，系统会线性地探查下一个位置，直到找到一个空槽，或者通过二次探查找到下一个可用的位置。

为了提高性能，Go 的 `map` 在插入新元素时，会动态地调整哈希表的大小，以保证负载因子在一定范围内。这样可以有效减少冲突的概率，并保持哈希表的高效性。

需要注意的是，由于哈希表的动态调整，`map` 的遍历顺序不是固定的，即不保证元素的遍历顺序和插入顺序相同。这是因为哈希表的调整可能导致元素在表中的位置发生变化。

8、一个系统进程大概多大?

Linux系统进程的大小会受到多种因素的影响，包括进程的功能、所加载的库和模块、进程的数据结构、系统架构等。通常情况下，一个典型的用户空间进程的大小可以在几十 KB 到几百 MB 之间。

在 Linux 中，可以使用 `pmap` 命令查看进程的内存映射，包括各个段的大小。

参考命令：

```C++
pmap <pid>
```

可以看到进程的内存布局和各个段的大小。

需要注意的是，这里提到的大小是虚拟内存的大小，而不是物理内存。实际占用的物理内存可能会更少，因为虚拟内存中的一部分可以尚未实际分配物理内存。

9、最大socket数量？

在一个系统中，能够同时打开的文件描述符（包括网络套接字）的数量是有限的。这个限制通常由系统的资源限制、文件描述符表的大小等因素决定。具体的限制因系统而异。

在 Linux 系统中，可以通过以下命令查看文件描述符的限制：

```Bash
ulimit -n
```

这个命令会显示当前用户对于文件描述符数量的限制。如果你需要修改这个限制，可以使用 `ulimit` 命令，但是需要注意的是这只是对当前 shell 会话有效，而不是全局设置。

另外，还可以通过修改系统级别的文件描述符限制，例如修改 `/etc/security/limits.conf` 文件或者 `/etc/sysctl.conf` 文件中的相关配置。但是，这样的修改可能需要重启系统才能生效。

10、LC64最小路径和

问题描述：给定一个包含非负整数的 *`m`*`x`*`n`*网格`grid` ，请找出一条从左上角到右下角的路径，使得路径上的数字总和为最小。

思路：

1. 创建一个与给定网格相同大小的数组，用于存储每个位置到达的最小路径和。
2. 初始化第一行和第一列，因为它们只能通过一种方式到达，所以路径和就是当前位置的数字加上前一个位置的路径和。
3. 对于其他位置，选择从上方或左侧到达的路径和较小的一条，将当前位置的路径和设为该值加上当前位置的数字。
4. 最终，右下角的元素将包含到达终点的最小路径和。

参考代码：

```C++
#include <vector>
#include <algorithm>

int minPathSum(std::vector<std::vector<int>>& grid) {
    int m = grid.size();
    int n = grid[0].size();

    // 创建数组用于存储最小路径和
    std::vector<std::vector<int>> dp(m, std::vector<int>(n, 0));

    // 初始化第一行和第一列
    dp[0][0] = grid[0][0];
    for (int i = 1; i < m; ++i) {
        dp[i][0] = dp[i - 1][0] + grid[i][0];
    }
    for (int j = 1; j < n; ++j) {
        dp[0][j] = dp[0][j - 1] + grid[0][j];
    }

    // 动态规划计算最小路径和
    for (int i = 1; i < m; ++i) {
        for (int j = 1; j < n; ++j) {
            dp[i][j] = grid[i][j] + std::min(dp[i - 1][j], dp[i][j - 1]);
        }
    }

    return dp[m - 1][n - 1];
}

int main() {
    // 示例输入网格
    std::vector<std::vector<int>> grid = {
        {1, 3, 1},
        {1, 5, 1},
        {4, 2, 1}
    };

    // 调用最小路径和函数
    int result = minPathSum(grid);

    // 输出结果
    std::cout << "Minimum Path Sum: " << result << std::endl;

    return 0;
}
```

11、LC1594 矩阵中最大非负积

思路：

1. 创建两个二维数组 `minProduct` 和 `maxProduct`，大小均为 m x n，用于存储到达每个位置的最小和最大非负积。
2. 初始化数组的第一个位置 `minProduct[0][0]` 和 `maxProduct[0][0]` 为 `grid[0][0]`。
3. 初始化第一行和第一列的元素，分别计算到达每个位置的最小和最大非负积。
4. 动态规划更新数组，从左上角开始遍历矩阵，对于每个位置 (i, j)，更新 `minProduct[i][j]` 和 `maxProduct[i][j]`：

```C++
minProduct[i][j] = min(minProduct[i-1][j], minProduct[i][j-1]) * grid[i][j];
maxProduct[i][j] = max(maxProduct[i-1][j], maxProduct[i][j-1]) * grid[i][j];
```

5. 这里用到了状态转移方程，即当前位置的最小非负积等于从上方和左方两个位置中选择较小的非负积乘以当前位置的值，最大非负积同理。

6. 如果右下角的最大非负积为负数，说明没有非负积路径，返回 -1。

7. 返回右下角的最大非负积对 1e9+7 取余的结果。

参考代码：

```C++
#include <iostream>
#include <vector>
#include <climits>

const int MOD = 1e9 + 7;

int maxProductPath(std::vector<std::vector<int>>& grid) {
    int m = grid.size();
    int n = grid[0].size();

    // 创建数组用于存储最小和最大非负积
    std::vector<std::vector<long long>> minProduct(m, std::vector<long long>(n, 0));
    std::vector<std::vector<long long>> maxProduct(m, std::vector<long long>(n, 0));

    // 初始化第一个位置
    minProduct[0][0] = maxProduct[0][0] = grid[0][0];

    // 初始化第一行和第一列
    for (int i = 1; i < m; ++i) {
        minProduct[i][0] = maxProduct[i][0] = minProduct[i - 1][0] * grid[i][0];
    }
    for (int j = 1; j < n; ++j) {
        minProduct[0][j] = maxProduct[0][j] = minProduct[0][j - 1] * grid[0][j];
    }

    // 动态规划计算最小和最大非负积
    for (int i = 1; i < m; ++i) {
        for (int j = 1; j < n; ++j) {
            // 更新最小和最大非负积
            minProduct[i][j] = std::min(minProduct[i - 1][j], minProduct[i][j - 1]) * grid[i][j];
            maxProduct[i][j] = std::max(maxProduct[i - 1][j], maxProduct[i][j - 1]) * grid[i][j];
        }
    }

    // 如果右下角的最大非负积为负数，则返回-1
    if (maxProduct[m - 1][n - 1] < 0) {
        return -1;
    }

    // 返回右下角的最大非负积对MOD取余的结果
    return maxProduct[m - 1][n - 1] % MOD;
}

int main() {
    // 示例输入网格
    std::vector<std::vector<int>> grid = {
        {1, -2, 1},
        {1, -2, 1},
        {3, -4, 1}
    };

    // 调用最大非负积函数
    int result = maxProductPath(grid);

    // 输出结果
    std::cout << "Max Product: " << result << std::endl;

    return 0;
}
```

## 二面

实习，项目

1、C++中的运算符，单目运算符，三元运算符，所有运算符中哪些不可以重载?

1. 算术运算符：+、-、*、/、%、++、--
2. 关系运算符：==、!=、<、>、<=、>=
3. 逻辑运算符：&&、||、!
4. 位运算符：&、|、^、~、<<、>>
5. 赋值运算符：=、+=、-=、*=、/=、%=、&=、|=、^=、<<=、>>=
6. 条件运算符（三元运算符）：? :
7. 成员访问运算符：.、->
8. 指针运算符：*、&、->*
9. 逗号运算符：,
10. sizeof 运算符
11. typeid 运算符
12. new 和 delete 运算符
13. 强制类型转换运算符：dynamic_cast、static_cast、const_cast、reinterpret_cast

不可以被重载的：

1. 点运算符(.)
2. 成员访问运算符(->)
3. 条件运算符(?:)
4. sizeof 运算符
5. typeid 运算符
6. new 和 delete 运算符

2、i++和++i，重载分别要怎么实现?

1. 后缀递增运算符 `i++`：
   - 返回原始值（未递增之前的值）。
   - 先使用原始值进行计算，然后再递增。
2. 前缀递增运算符 `++i`：
   - 返回递增后的值。
   - 先递增，然后使用递增后的值进行计算。

给个例子：

```C++
#include <iostream>

class Counter {
private:
    int count;

public:
    // 构造函数
    Counter() : count(0) {}

    // 后缀递增运算符重载
    Counter operator++(int) {
        Counter temp(*this);  // 保存原始值
        count++;
        return temp;         // 返回原始值
    }

    // 前缀递增运算符重载
    Counter& operator++() {
        count++;
        return *this;        // 返回递增后的值
    }

    // 获取计数值
    int getCount() const {
        return count;
    }
};

int main() {
    Counter c;

    // 后缀递增运算符
    Counter result1 = c++;
    std::cout << "After c++, Count: " << c.getCount() << std::endl;  // 输出 1
    std::cout << "Original Value: " << result1.getCount() << std::endl;  // 输出 0

    // 前缀递增运算符
    Counter result2 = ++c;
    std::cout << "After ++c, Count: " << c.getCount() << std::endl;  // 输出 2
    std::cout << "Updated Value: " << result2.getCount() << std::endl;  // 输出 2

    return 0;
}
```

3、想让一个可执行程序只能运行一个实例应该怎么办？查找运行进程可行吗？写文件作为锁有问题吗？

查找运行进程：

这种方法通过检查当前是否已经有相同的进程在运行，如果有，则终止当前实例。这种方法的实现方式因操作系统而异，例如在 Linux 下可以通过查找进程列表，而在 Windows 下可以使用互斥体。

Linux下参考实现：

```C++
#include <cstdlib>
#include <cstring>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

bool IsAnotherInstanceRunning() {
    const char* process_name = "your_program_name";
    pid_t pid = getpid();

    char path[1024];
    snprintf(path, sizeof(path), "/proc/%d/cmdline", pid);

    int fd = open(path, O_RDONLY);
    if (fd != -1) {
        char cmdline[1024];
        ssize_t len = read(fd, cmdline, sizeof(cmdline) - 1);
        close(fd);

        if (len > 0) {
            cmdline[len] = '\0';
            return strstr(cmdline, process_name) != nullptr;
        }
    }

    return false;
}

int main() {
    if (IsAnotherInstanceRunning()) {
        // 程序已经有实例在运行
        return 0;
    }

    // 正常程序逻辑...
    return 0;
}
```

使用文件作为锁：

这种方法通过创建或锁定一个文件，确保只有一个实例可以拥有该文件的锁。在 Linux 下通常使用文件锁（fcntl 或 flock），在 Windows 下可以使用互斥体。

Linux 下的参考实现：

```C++
#include <fcntl.h>
#include <unistd.h>

bool IsAnotherInstanceRunning() {
    int fd = open("/tmp/your_program.lock", O_RDWR | O_CREAT, 0666);

    if (fd == -1 || flock(fd, LOCK_EX | LOCK_NB) == -1) {
        // 程序已经有实例在运行
        close(fd);
        return true;
    }

    return false;
}

int main() {
    if (IsAnotherInstanceRunning()) {
        // 程序已经有实例在运行
        return 0;
    }

    // 正常程序逻辑...

    flock(fd, LOCK_UN);
    close(fd);
    return 0;
}
```

4、Kill -9，这个信号可以捕获吗？

`kill -9` 发送的是 SIGKILL 信号，它是一个不可被捕获、处理或忽略的信号。该信号会立即终止目标进程，不给进程进行清理或处理的机会。因此，无法在程序中捕获 `kill -9` 发送的 SIGKILL 信号。

对于其他一些信号，可以使用信号处理器（signal handler）来捕获和处理，但是 SIGKILL 不是其中之一。如果需要在程序中处理某些退出信号，可以考虑使用其他信号，如 SIGTERM（15）等，这些信号可以被捕获并在程序中进行处理。

5、动态链接和静态链接？

1. 静态链接（Static Linking）：
   - 在编译时将所有的程序模块和库文件链接成一个完整的可执行文件。
   - 静态链接的结果是生成一个包含所有程序和库函数的可执行文件，这个文件在运行时不再依赖于原始的模块和库文件。
   - 优点是执行速度较快，因为所有的链接和地址解析在编译时完成。
   - 缺点是生成的可执行文件较大，且每次更新程序或库时都需要重新编译和链接。
2. 动态链接（Dynamic Linking）：
   - 在编译时只生成程序模块的目标文件，在运行时再动态加载所需的库文件。
   - 程序在运行时需要一些共享库，这些库在系统中单独存储，程序在运行时通过动态链接器将它们加载到内存中。
   - 优点是可执行文件较小，且共享库的更新不需要重新编译程序。
   - 缺点是每次运行程序都需要进行动态链接，可能稍微降低执行速度。

6、如果要更新一个运行中的动态链接库，可以直接替换吗，mv和cp分别会有问题吗？

更新一个运行中的动态链接库通常是可以的，但需要注意一些细节，以确保平滑的切换。

使用 `mv` 命令：

```Bash
mv new_lib.so old_lib.so
```

这将新库重命名为旧库的名称。这样做的好处是，已经在运行的程序将继续使用旧版本的库，而新启动的程序将使用新版本的库。但是，这种方式也可能导致新旧库混用的问题，特别是在长时间运行的程序中。

使用 `cp` 命令：

```Bash
cp new_lib.so old_lib.so
```

这样会在文件系统中创建一个新的文件，两个文件名都指向相同的物理文件。这种方式对于长时间运行的程序可能更为安全，因为不会在运行过程中切换库文件。

7、进程间通信，共享内存为什么快

1. 管道（Pipes）： 管道是一种半双工通信机制，它允许一个进程写入数据，另一个进程读取这些数据。在父子进程或者兄弟进程之间使用较为方便。Linux/Unix 系统中使用 `pipe` 函数创建。
2. 命名管道（Named Pipes）： 与管道类似，但是命名管道是有名字的，可以在无亲缘关系的进程之间进行通信。在文件系统中有对应的文件名，可以通过文件名在不同的进程之间进行通信。
3. 消息队列（Message Queues）： 进程可以通过消息队列向其他进程发送消息。消息队列允许进程通过消息进行异步通信，发送方将消息放入队列，接收方从队列中读取消息。不同进程可以通过消息队列进行通信。
4. 共享内存（Shared Memory）： 多个进程可以访问同一块共享的内存区域，从而实现高效的数据传输。需要使用同步机制（如信号量）来避免竞态条件。
5. 信号（Signals）： 进程可以通过发送信号来通知其他进程发生了某个事件。例如，用于处理中断、异常等情况。信号的使用需要处理信号的机制。
6. 套接字（Sockets）： 进程可以使用套接字进行网络通信，也可以在本地进行进程间通信。套接字提供了一种通用的通信机制，可以用于不同计算机之间的通信。
7. 文件锁（File Locking）： 进程可以使用文件锁来实现对文件的互斥访问，从而实现进程间的通信。文件锁可以避免多个进程同时对文件进行写入操作。
8. 信号量（Semaphores）： 信号量是一种用于进程同步的机制，也可用于进程间通信。信号量可以用于控制多个进程对共享资源的访问。

共享内存为什么那么快？

1. 直接内存访问： 共享内存允许多个进程直接访问同一块物理内存空间，而不需要通过内核进行数据的复制或传递。这使得进程可以直接读写内存，而无需进行复杂的数据拷贝。
2. 无需内核参与： 在共享内存中，内核不需要进行数据传输或者维护通信的状态，因为共享内存区域被映射到各个进程的地址空间中，进程可以直接读写。这避免了进程切换和内核态与用户态的切换，从而提高了通信效率。
3. 低开销： 由于共享内存无需内核进行数据复制，通信开销较低。相对于一些其他通信方式（如管道、消息队列等），它不需要进行额外的数据拷贝，减少了额外的开销。
4. 高吞吐量： 共享内存对于数据的读写是非常高效的，特别是对于需要频繁通信的大量数据。这使得它适用于一些需要高吞吐量的应用场景，如图像处理、科学计算等。

8、网络编程基本流程，socket，客户端/服务端一定要绑定端口吗

1. 创建套接字：使用`socket`函数创建一个套接字。在C/C++中，可以使用`socket`函数，返回的套接字描述符用于后续的操作。

```C
int sockfd = socket(AF_INET, SOCK_STREAM,0);
```

这里的`AF_INET`表示使用IPv4地址族，`SOCK_STREAM`表示使用TCP协议，`0`表示使用默认协议。

2. 绑定地址和端口（服务端）：服务端需要绑定一个具体的地址和端口，以监听客户端的连接请求。

```C
struct sockaddr_in server_addr;
server_addr.sin_family = AF_INET;
server_addr.sin_addr.s_addr = INADDR_ANY;// 监听所有网络接口
server_addr.sin_port = htons(PORT);// 端口号
bind(sockfd, (structsockaddr*)&server_addr,sizeof(server_addr));
```

这里`PORT`是你选择的端口号。

3. 监听连接（服务端）：在服务端调用`listen`函数开始监听客户端连接请求。

```C
listen(sockfd, backlog);
```

`backlog`参数指定连接队列的最大长度。

4. 建立连接（客户端）：客户端使用`connect`函数连接到服务端。

```C
struct sockaddr_in server_addr;
server_addr.sin_family = AF_INET;
server_addr.sin_addr.s_addr = inet_addr("服务器IP地址");
server_addr.sin_port = htons(PORT);
connect(sockfd, (structsockaddr*)&server_addr,sizeof(server_addr));
```

5. 这里的服务器IP地址和端口号应该是服务端绑定的地址和端口。

6. 发送和接收数据：使用`send`和`recv`函数进行数据的发送和接收。在服务端和客户端之间可以通过套接字进行双向通信。

```C
send(sockfd, buffer,sizeof(buffer),0);
recv(sockfd, buffer,sizeof(buffer),0);
```

7. 关闭连接：使用`close`函数关闭套接字。

```C
close(sockfd);
```

客户端和服务端在使用套接字进行通信时，一般需要绑定端口。服务端需要绑定一个固定的端口用于监听客户端的连接请求。客户端在连接服务端时，可以让系统自动分配一个可用的本地端口。

9、端口数范围，为什么是这个范围，TCP和UDP，TCP header，七层网络/五层网络

端口号是一个16位的整数，取值范围是0到65535。端口号按照使用目的分为三个范围：

1. 知名端口：范围是0到1023。这些端口通常被系统和服务占用，例如HTTP服务使用80端口，FTP服务使用21端口等。
2. 注册端口：范围是1024到49151。这些端口是由一些应用程序注册的，用于提供特定的服务。
3. 动态和/或私有端口：范围是49152到65535。这些端口通常被客户端应用程序使用，用于发起临时会话。

这个范围的划分主要是为了方便管理和避免冲突。知名端口是为了让系统和服务易于识别，注册端口是为了让应用程序易于管理，而动态和/或私有端口是为了让客户端应用程序有足够的选择空间。

tcp和udp概念和区别这里不讲，可以去看看我别的文章。

TCP头部还包含了序号、确认号、窗口大小等信息，用于实现可靠的连接。这些头部信息都是在数据传输过程中用于控制和管理的。

1. 七层网络模型（OSI模型）：
   - 物理层（Physical Layer）
   - 数据链路层（Data Link Layer）
   - 网络层（Network Layer）
   - 传输层（Transport Layer）
   - 会话层（Session Layer）
   - 表示层（Presentation Layer）
   - 应用层（Application Layer）
2. 五层网络模型（TCP/IP模型）：
   - 物理层（Physical Layer）
   - 数据链路层（Data Link Layer）
   - 网络层（Network Layer）
   - 传输层（Transport Layer）
   - 应用层（Application Layer）

10、IO多路复用，select, poll, epoll？

1. select：
   - `select` 是最早的实现之一，可用于检测多个文件描述符的状态。
   - 在 Windows、Linux 等平台上都有实现。
   - 它使用了三个位集合，分别表示可读、可写和异常事件，通过传入这三个集合进行监听。
   - 缺点包括效率较低，受到文件描述符数量的限制，复杂度为 O(n)。
2. poll：
   - `poll` 是对 `select` 的改进，也可以用于检测多个文件描述符的状态。
   - 只在 Linux/Unix 等类 Unix 系统上实现。
   - 使用一个结构数组来存储每个文件描述符的状态，并将整个数组传递给 `poll` 函数。
   - 也受到文件描述符数量的限制，复杂度为 O(n)。
3. epoll：
   - `epoll` 是 Linux 特有的实现，为了克服 `select` 和 `poll` 的限制而设计。
   - 使用了事件驱动的方式，通过注册事件，只关心那些真正发生了事件的文件描述符。
   - `epoll` 提供了三个函数：`epoll_create` 创建一个 `epoll` 句柄，`epoll_ctl` 修改事件注册，`epoll_wait` 等待事件发生。
   - 支持 Edge Triggered (ET) 和 Level Triggered (LT) 两种模式，其中 ET 模式效率更高。
   - 复杂度为 O(1)，性能更好。

11、什么时候多线程，什么时候多进程？

1. 多线程：
   - 共享内存： 如果任务之间需要共享内存，多线程可能是更合适的选择。线程共享相同的地址空间，因此数据传递相对更简单。
   - 轻量级任务： 对于一些轻量级的任务，使用多线程可能更加方便。线程的创建和销毁相对于进程来说更加轻量级。
   - 通信开销小： 如果任务之间需要频繁通信，并且通信开销相对较小，多线程可能更适用。
2. 多进程：
   - 独立性要求高： 如果任务之间需要高度的独立性，以防止彼此之间的干扰，多进程可能更适合。每个进程都有独立的地址空间，减少了数据共享带来的复杂性。
   - 可伸缩性： 多进程在处理多核心系统上有更好的可伸缩性，因为每个进程都能够运行在一个独立的核心上。
   - 容错性： 进程之间的容错性更好，一个进程崩溃不会影响其他进程。

12、客户端和服务端通信要怎么做？

1. Socket通信：
   - 使用套接字（Socket）进行通信是一种底层的、通用的方式。它可以实现多种协议，如TCP和UDP。
   - 通过Socket编程，客户端和服务端可以建立连接，并通过发送和接收数据进行通信。
2. HTTP/HTTPS通信：
   - 使用HTTP或HTTPS协议进行通信是一种常见的应用层协议。
   - 客户端通过发送HTTP请求，服务端通过HTTP响应进行通信。HTTPS在HTTP的基础上加入了加密层，提供了更安全的通信。
3. RPC（远程过程调用）：
   - RPC是一种通过网络调用远程服务的机制，使得客户端可以像调用本地函数一样调用远程服务器上的函数。
   - 常见的RPC框架包括gRPC、Apache Thrift等。
4. WebSocket通信：
   - WebSocket提供了一种在单个TCP连接上进行全双工通信的协议，适用于实时性要求较高的应用。
   - WebSocket通信始于一个HTTP握手，之后客户端和服务端可以直接在一个持久化的连接上进行双向数据传输。
5. 消息队列通信：
   - 使用消息队列（例如RabbitMQ、Kafka、ActiveMQ等）作为中间件，实现客户端和服务端之间的异步通信。
   - 客户端通过向消息队列发送消息，服务端从消息队列中接收并处理消息。

13、客户端服务端在同一台机器要怎么通信？

1. 本地套接字（Local Socket）：

   - 使用本地套接字，也称为Unix域套接字，可以在同一台机器上的进程之间建立通信。
   - 这种方式速度较快，适用于进程之间的本地通信。

2. Loopback地址（127.0.0.1）：

   - 客户端和服务端可以通过使用Loopback地址（通常是127.0.0.1）来进行TCP或UDP通信。
   - 这种方式相当于在同一台机器上模拟网络通信，适用于需要网络协议的场景。

3. 进程间通信（IPC）机制：

   - 在同一台机器上的不同进程之间可以使用进程间通信机制，如共享内存、消息队列、信号等。
   - 这种方式适用于需要高效数据共享或异步通信的场景。

4. 本地HTTP服务：

   - 可以在同一台机器上运行一个HTTP服务器，客户端通过HTTP请求与服务端进行通信。
   - 这适用于Web应用或其他基于HTTP的应用。

5. RPC（本地调用）：

   如果采用RPC框架，可以在同一台机器上通过RPC进行本地调用，实现客户端和服务端之间的函数调用。

6. 共享内存：

   客户端和服务端可以使用共享内存来实现数据的共享，以达到进程间通信的目的。

14、RPC，为什么用RPC，介绍RPC框架，如何设计一个RPC框架，RPC框架如何性能调优

RPC（Remote Procedure Call，远程过程调用）是一种用于实现分布式系统中进程间通信的协议，它允许程序调用另一个地址空间（通常是网络上的另一台机器上）的过程或函数，而就像本地调用一样。RPC的目标是使远程过程调用看起来像本地调用一样，尽可能地屏蔽底层通信细节。

为什么使用RPC？

1. 抽象分布式通信： RPC使分布式系统中的通信更加抽象，使开发者不必关心底层的通信细节，而可以专注于业务逻辑。
2. 代码结构清晰： RPC可以让开发者编写更为清晰和结构化的代码，就像调用本地函数一样调用远程函数。
3. 模块化： RPC支持模块化设计，允许系统通过服务的方式进行组织，各个服务之间通过RPC进行通信。
4. 跨语言支持： RPC框架通常支持跨语言调用，使得不同语言编写的程序可以方便地进行通信。

RPC框架设计要点：

1. 协议定义： 设计通信协议，包括数据序列化和反序列化规则，通常使用类似Protocol Buffers、Thrift或JSON等。
2. 服务接口定义： 定义服务接口，包括服务的方法、参数和返回值等。
3. 通信： 设计底层通信机制，可以选择使用HTTP、TCP、UDP等协议。
4. 服务注册和发现： 提供服务注册和发现机制，使得客户端可以找到可用的服务。
5. 安全性： 考虑通信的安全性，可以采用加密协议等手段保障数据的安全。
6. 错误处理： 设计良好的错误处理机制，使得客户端和服务端能够对各种错误进行适当处理。

RPC框架性能调优：

1. 序列化和反序列化优化： 选择高效的序列化框架，并进行相应的优化。
2. 连接池： 使用连接池来重用已经建立的连接，减少连接建立和断开的开销。
3. 异步调用： 使用异步调用来提高并发性能，允许同时处理多个请求。
4. 负载均衡： 在多个服务提供者之间进行负载均衡，确保请求被合理地分配。
5. 缓存： 对频繁调用的数据进行缓存，减少对远程服务的请求。
6. 压缩： 对通信的数据进行压缩，减少网络传输的数据量。
7. 连接复用： 复用连接，避免频繁地建立和关闭连接。
8. 超时设置： 合理设置超时时间，防止长时间的等待。
9. 并发控制： 对服务端资源进行并发控制，防止过多的并发请求导致性能下降。

15、HTTP和RPC区别，使用JSON传输和使用protobuf传输区别？

1. 通信模型：

   - HTTP： 是一种无状态协议，每次请求之间不会保存状态信息。通常使用请求-响应模型，客户端发起请求，服务器回送响应。

   - RPC： 通常基于远程过程调用的模型，允许程序调用另一台计算机上的过程或函数。

2. 语义：

   - HTTP： 主要用于传输超文本，支持不同的操作（GET、POST、PUT、DELETE等），但语义相对较为简单。

   - RPC： 专注于调用远程服务或过程，提供更丰富的语义。

3. 性能：

   - HTTP： 通常基于文本传输，相对较为庞大，可能存在较大的传输开销。

   - RPC： 通常采用二进制协议，如Protocol Buffers、Thrift等，可以更有效地利用网络带宽。

4. 序列化：

   - HTTP： 通常使用文本格式（如JSON、XML）进行数据序列化。

   - RPC： 可以使用更为高效的二进制序列化格式，例如Protocol Buffers、MessagePack等。

5. 协议：

   - HTTP： 基于TCP协议，是一种应用层协议。

   - RPC： 可以基于不同的传输层协议，如HTTP、TCP、UDP等。

6. 使用场景：

   - HTTP： 适用于Web应用、浏览器与服务器之间的通信。

   - RPC： 适用于分布式系统、微服务架构中各个服务之间的通信。

7. Json和protobuf使用区别

   - JSON： 文本格式，易读易写，但相对占用带宽和解析开销较大。

   - Protocol Buffers： 二进制格式，更为紧凑，解析效率高，但不易读。

16、负载均衡，nginx为什么吞吐量大/快？

1. 事件驱动架构： Nginx采用事件驱动的异步非阻塞模型，它使用少量的固定的工作进程来处理大量的并发连接。这种事件驱动模型使得 Nginx 能够高效地处理大量并发请求而不会导致线程阻塞，从而提高了系统的并发处理能力。
2. 高效的内存管理： Nginx 的内存分配采用了内存池的方式，减少了内存碎片，提高了内存使用效率。这使得 Nginx 能够在高并发环境下更加稳定地运行。
3. 单机多核支持： Nginx 的多进程和多线程设计使得它能够充分利用多核处理器的性能。每个工作进程都是独立的，可以独立处理请求，提高了整体的并发处理能力。
4. 事件模块： Nginx 提供了丰富的事件模块，包括反向代理、负载均衡、SSL/TLS、HTTP/2 等，这些模块的性能都经过优化，能够高效地处理不同类型的请求。
5. 高度可定制性： Nginx 具有高度可定制性，可以根据具体的需求进行灵活配置。这使得它适用于各种场景，包括反向代理、负载均衡、静态文件服务等。
6. 精简而高效的代码： Nginx 的代码精简而高效，注重性能。通过避免过度的模块化和提供高效的底层数据结构，Nginx 能够更快速地处理请求。

17、算法题：LC48 矩阵顺时针旋转90

思路：

1. 转置矩阵：
   - 遍历矩阵的上三角部分（即主对角线以上的元素）。
   - 将每个元素与其在下三角部分（主对角线以下的元素）对称位置的元素进行交换。
   - 这一步的效果是转置矩阵。
2. 反转每一行：
   - 遍历转置后的矩阵的每一行。
   - 反转每一行的元素。

以下是详细的步骤：

原始矩阵：

```Plaintext
1  2  3
4  5  6
7  8  9
```

转置后的矩阵：

```Plaintext
1  4  7
2  5  8
3  6  9
```

每一行反转后的矩阵：

```Plaintext
7  4  1
8  5  2
9  6  3
```

因此，最终旋转后的矩阵为：

```Plaintext
7  4  1
8  5  2
9  6  3
```

参考代码：

```C++
#include <vector>
#include <algorithm>
#include <iostream>

using namespace std;

void rotate(vector<vector<int>>& matrix) {
    int n = matrix.size();

    // 转置矩阵
    for (int i = 0; i < n; ++i) {
        for (int j = i; j < n; ++j) {
            swap(matrix[i][j], matrix[j][i]);
        }
    }

    // 逆序每一行
    for (int i = 0; i < n; ++i) {
        reverse(matrix[i].begin(), matrix[i].end());
    }
}

int main() {
    vector<vector<int>> matrix = {
        {1, 2, 3},
        {4, 5, 6},
        {7, 8, 9}
    };

    cout << "原始矩阵：" << endl;
    for (const auto& row : matrix) {
        for (int num : row) {
            cout << num << " ";
        }
        cout << endl;
    }

    rotate(matrix);

    cout << "旋转后的矩阵：" << endl;
    for (const auto& row : matrix) {
        for (int num : row) {
            cout << num << " ";
        }
        cout << endl;
    }

    return 0;
}
```

## 三面：

聊天+场景题