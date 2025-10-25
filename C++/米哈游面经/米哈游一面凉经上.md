# 米哈游一面凉经上

> 来源：https://www.nowcoder.com/feed/main/detail/7f28c235a0134b4a9be9fea52efe9551

## 1、模板类 模板匹配失败是不是错误？

模板匹配失败 在 C++ 中是一种编译期错误，但具体情况要看你是在哪里失败的，失败类型不同，表现形式也不同：

#### 情况一：模板匹配失败但可以回退到其它重载

这是合法的，不算错误，只要编译器找到了其它可以匹配的函数模板或重载。

```c++
template <typename T>
void func(T t) { std::cout << "Template version\n"; }

void func(int i) { std::cout << "Non-template version\n"; }

int main() {
    func(10); // 模板匹配 int，但也有非模板重载，选择非模板版本，合法！
}
```

#### 情况二：只有一个模板版本，但模板匹配失败

这是 编译错误，因为编译器无法推导或替换模板参数，导致函数/类无法实例化。

```c++
template <typename T>
void func(T t1, T t2) { }

int main() {
    func(1, 1.0); // T 同时要匹配 int 和 double，推导失败，编译错误
}
```

解决方法：显式指定模板类型或者使用两个模板参数：

```c++
template <typename T1, typename T2>
void func(T1 t1, T2 t2); // OK
```

#### 情况三：模板类中的某个方法匹配失败

C++ 会在你实例化这个方法时才做替换（SFINAE：Substitution Failure Is Not An Error），这也是标准行为的一部分。

```c++
template <typename T>
class Wrapper {
public:
    void doSomething() {
        T::nonexistentMethod(); // 如果 T 没有这个方法，只有在用到这个函数时才报错
    }
};

int main() {
    Wrapper<int> w; // ✅ 编译通过
    // w.doSomething(); // ❌ 如果调用就报错
}
```

## 2、三种智能指针？怎么用？区别？

#### `std::unique_ptr`（唯一所有权指针）

独占资源，不能被复制，只能移动（move）。

生命周期随指针对象结束而自动释放资源。

##### 用法

```c++
#include <memory>

std::unique_ptr<int> ptr1(new int(10));      // 创建方式1
auto ptr2 = std::make_unique<int>(20);       // 推荐方式（C++14 起）

// 转移所有权
std::unique_ptr<int> ptr3 = std::move(ptr1); // ptr1 被清空，ptr3 拥有资源
```

##### 禁止复制

```c++
std::unique_ptr<int> ptr4 = ptr2; // 错误，不能复制
```

#### `std::shared_ptr`（共享所有权指针）

多个 `shared_ptr` 可以共享同一块资源。

内部维护一个 引用计数，最后一个引用释放时自动释放资源。

##### 用法

```c++
#include <memory>

std::shared_ptr<int> sp1 = std::make_shared<int>(100); // 推荐方式
std::shared_ptr<int> sp2 = sp1; // 引用计数 +1

std::cout << sp1.use_count();  // 打印引用计数
```

**不能形成循环引用**，否则内存泄漏！

#### `std::weak_ptr`（弱引用指针）

解决 `shared_ptr` 循环引用问题。

**不拥有资源**，不会增加引用计数。

必须用 `.lock()` 转换为 `shared_ptr` 才能使用资源。

##### 用法

```c++
#include <memory>

std::shared_ptr<int> sp = std::make_shared<int>(42);
std::weak_ptr<int> wp = sp;

if (auto locked = wp.lock()) {
    std::cout << *locked << std::endl;  // 安全访问
} else {
    std::cout << "资源已释放" << std::endl;
}
```

#### 区别与对比

| 特性     | unique_ptr     | shared_ptr       | weak_ptr       |
| -------- | -------------- | ---------------- | -------------- |
| 所有权   | 独占           | 共享             | 无所有权       |
| 引用计数 | 无             | 有               | 无             |
| 复制     | 不可，只能移动 | 可复制           | 可复制但不控制 |
| 销毁时机 | 超出作用域     | 计数为 0         | 不直接销毁     |
| 访问方式 | 直接 * 或 ->   | 直接 * 或 ->     | 通过 lock()    |
| 性能开销 | 最低（无计数） | 中等（计数管理） | 中等（需检查） |
| 主要用途 | 单一拥有者     | 多拥有者         | 打破循环引用   |

## 3、unique_ptr能不能被复制？

不能。

原因：

- unique_ptr 的设计目标是确保资源（如动态分配的内存）只有一个明确的所有者，避免多个指针同时管理同一资源导致的双重释放或其他未定义行为。
- 复制会破坏独占性，因此编译器禁止复制操作。

**支持移动而不是复制**

unique_ptr 提供了移动构造函数和移动赋值运算符，允许将所有权从一个 unique_ptr 转移到另一个。

#### 为什么禁止复制？

##### 独占所有权的语义

unique_ptr 的核心理念是“独占”（Unique），如果允许复制：

- 多个 unique_ptr 会指向同一块内存。
- 当这些指针销毁时，会多次调用 delete，导致未定义行为（如双重释放）。

##### 与 shared_ptr 的对比

shared_ptr 支持复制，通过引用计数管理多个拥有者。

unique_ptr 不使用计数，追求轻量和明确性，因此禁用复制。

## 4、我一个类实例化不想被复制 怎么办？

1. 显式删除复制函数（C++11 及以上）：使用 = delete。
2. 将复制函数声明为私有（C++98/03）：传统方法。

#### 方法 1：使用 = delete（推荐）

将复制构造函数和复制赋值运算符标记为 deleted。

```c++
#include <iostream>

class NoCopy {
public:
    NoCopy(int val) : value(val) {}
    void print() const { std::cout << "Value: " << value << "\n"; }

    // 禁用复制构造函数
    NoCopy(const NoCopy&) = delete;
    // 禁用复制赋值运算符
    NoCopy& operator=(const NoCopy&) = delete;

private:
    int value;
};

int main() {
    NoCopy obj1(10);
    obj1.print(); // Value: 10

    // NoCopy obj2 = obj1; // 编译错误：调用已删除的复制构造函数
    // NoCopy obj3(20);
    // obj3 = obj1;        // 编译错误：调用已删除的赋值运算符

    return 0;
}
```

#### 方法 2：声明为私有（C++98/03）

将复制构造函数和赋值运算符声明为 private，不提供实现。

```c++
#include <iostream>

class NoCopy {
public:
    NoCopy(int val) : value(val) {}
    void print() const { std::cout << "Value: " << value << "\n"; }

private:
    int value;
    NoCopy(const NoCopy&);            // 私有，未定义
    NoCopy& operator=(const NoCopy&); // 私有，未定义
};

int main() {
    NoCopy obj1(10);
    obj1.print();

    // NoCopy obj2 = obj1; // 编译错误：NoCopy(const NoCopy&) 不可访问
    // NoCopy obj3(20);
    // obj3 = obj1;        // 编译错误：operator= 不可访问

    return 0;
}
```

## 5、shared_ptr是否线程安全（引用计数为什么线程安全）？

shared_ptr **本身并非完全线程安全的**，但它的引用计数管理在特定条件下是线程安全的。

#### 引用计数线程安全

当多个线程对同一个底层资源（由 shared_ptr 管理）的引用计数进行操作（如复制或销毁 shared_ptr），这些操作是原子化的，不会引发竞争。

这是标准库实现（如 libstdc++、libc++）通过原子操作（如 std::atomic）保证的。

#### 对象本身线程不安全

如果多个线程同时修改同一个 shared_ptr 对象（如赋值 sp = new_sp），会导致数据竞争。

原因是 shared_ptr 的赋值涉及指针更新，不是原子操作。

#### 引用计数为什么线程安全？

shared_ptr 的引用计数：

- 内部维护两个计数器：
  - 强引用计数（use_count() 返回）：表示有多少 shared_ptr 拥有资源。
  - 弱引用计数：表示有多少 weak_ptr 引用资源。
- 这些计数器存储在控制块（Control Block）中，与管理的对象分开。

原子操作：

控制块中中的计数器使用原子操作（如 std::atomic<int> 或底层 CAS，Compare-And-Swap）实现。

常见操作：

- 复制 shared_ptr：原子递增强引用计数。
- 销毁 shared_ptr：原子递减强引用计数，若减至 0，则释放资源。

## 6、class一个类的内存大小由什么判断？

C++ 中，一个类（class）的内存大小（通常通过 sizeof 操作符计算）是由其**数据成员**和**内存对齐规则**共同决定的，而不是由成员函数、虚函数表指针的数量、访问权限（如 public、private）等直接决定。

#### 影响类内存大小的因素

##### 数据成员

类中声明的非静态数据成员（如 int、double、指针等）直接占用内存。

规则：

- 每个数据成员的大小由其类型决定（如 int 通常 4 字节，double 通常 8 字节）。
- 非静态成员按声明顺序排列在对象内存中。

注意：静态成员（static）不占用对象内存，存储在全局数据段。

##### 内存对齐

编译器为了优化内存访问效率，会对数据成员进行对齐填充（padding），使每个成员的起始地址满足特定对齐要求。

规则：

- 每个数据类型有对齐要求，通常是其大小的倍数（如 int 对齐到 4 字节，double 对齐到 8 字节）。
- 类的总大小是对齐到最大成员的对齐边界。

影响：可能在成员间插入填充字节，导致实际大小大于成员大小之和。

##### 虚函数表指针

如果类有虚函数，编译器会为每个对象插入一个虚函数表指针（vptr），指向类的虚函数表（vtable）。

大小：

- 32 位系统：4 字节。
- 64 位系统：8 字节。

条件：只有声明了虚函数（包括虚析构函数）或继承自有虚函数的基类时，才会有 vptr。

##### 继承

单继承：

- 派生类包含基类的非静态成员和自身的非静态成员。
- 如果基类有 vptr，派生类通常复用它（除非多继承）。

多继承：每个有虚函数的基类可能贡献一个 vptr，增加大小。

对齐：继承后仍需满足整体对齐。

##### 空类

规则：空类（无数据成员、无虚函数）大小为 1 字节。

原因：C++ 标准要求每个对象有唯一地址，编译器插入 1 字节占位符。

#### 内存大小计算规则

成员大小之和：计算所有非静态数据成员的原始大小。

对齐填充：

- 按声明顺序排列成员，每个成员起始地址对齐到其类型的要求。
- 在成员间插入填充字节。

整体对齐：类的总大小对齐到最大成员的对齐边界（或 vptr 大小）。

特殊情况：

- 添加 vptr（若有虚函数）。
- 继承时累加基类大小。

## 7、为什么要有内存对齐？

#### 什么是内存对齐？

内存对齐是指将数据存储在内存中的地址调整为特定边界（如 4 字节、8 字节）的倍数，而不是紧密排列。

表现：

- 每个数据类型有对齐要求（如 int 通常对齐到 4 字节，double 对齐到 8 字节）。
- 结构体或类的总大小对齐到最大成员的对齐边界。

效果：

- 在成员间可能插入填充字节（padding）。
- 示例：`struct { char c; int i; }` 大小不是 5（1+4），而是 8（对齐填充）。

#### 为什么要有内存对齐？

##### 硬件性能优化

CPU 访问效率：

- 现代处理器（如 x86、ARM）设计为按字长（word size，通常 4 或 8 字节）访问内存。
- 对齐的内存地址允许 CPU 一次读取整个数据块。
- 未对齐的访问可能需要多次读取和拼接，增加周期数。

假设 CPU 字长为 4 字节：

- 对齐的 int（4 字节）在地址 0x1000，一次读取。
- 未对齐的 int 在 0x1001，需读取 0x1000-0x1003 和 0x1004-0x1007 两次，性能下降。

##### 硬件限制

某些架构的要求：

- 在某些 CPU 架构（如 RISC，ARM、SPARC）上，未对齐的内存访问不仅慢，还会触发硬件异常（如 Bus Error）。
- x86 架构虽支持未对齐访问，但性能较低。

后果：未对齐可能导致程序崩溃或不可预测行为。

##### 数据一致性

多核处理器和缓存：

- 内存对齐确保数据在缓存行（Cache Line，通常 64 字节）中正确分布。
- 未对齐可能导致数据跨缓存行，增加缓存未命中（Cache Miss）和同步开销。

原子操作：对齐的内存支持高效的原子操作（如 std::atomic），未对齐可能失败。

#### 内存对齐的实现原理

##### 对齐规则

类型对齐：

基本类型对齐到自身大小的倍数：

- char：1 字节。
- int：4 字节。
- double：8 字节。

结构体/类对齐：

- 每个成员按其对齐要求排列。
- 总大小对齐到最大成员的对齐边界。

填充：在成员间插入字节，使下一成员地址满足对齐。

## 8、代码编译过程？

#### 预处理

处理源代码中的预处理器指令（如 #include、#define），生成扩展后的源代码。

作用：

- 展开宏定义。
- 处理头文件（将 #include 的内容嵌入）。
- 条件编译（如 #ifdef）。
- 移除注释。

输入：.cpp 文件（源文件）。

输出：预处理后的临时文件（通常以 .i 为扩展名）。

#### 编译

预处理后的源码会被编译器翻译成汇编代码，每个 `.cpp` 文件单独编译，生成 `.o` 或 `.obj` 文件（目标文件）。

作用：

- 语法检查：验证代码是否符合 C++ 语法。
- 语义分析：检查类型匹配、变量声明等。
- 代码优化：生成高效的中间表示。
- 生成汇编代码：针对特定硬件架构。

输入：.i 文件（预处理输出）。

输出：汇编文件（.s 文件）。

#### 汇编

编译生成的汇编代码被 汇编器 转换为机器码（二进制），生成目标文件（`.o` 或 `.obj`）。

作用：

- 将人类可读的汇编指令（如 movl）翻译为二进制指令。
- 生成目标文件，包含未解析的符号引用。

输入：.s 文件（汇编代码）。

输出：目标文件（.o 或 .obj 文件）。

#### 链接

把多个 `.o` 文件和库文件链接在一起，解决函数、变量引用，生成最终的可执行文件（如 `a.out` 或 `.exe`）。

作用：

- 解析符号引用（如 std::cout 的地址）。
- 合并代码段、数据段。
- 处理静态库（如 libstdc++.a）或动态库（如 libstdc++.so）。

输入：

- .o 文件（目标文件）。
- 库文件（如标准库）。

输出：可执行文件（如 a.out 或指定名称）。

## 9、动态库静态库区别？怎么链接？

#### 静态库

静态库是一组目标文件（.o 或 .obj）的集合，通常以 .a（Linux）或 .lib（Windows）为扩展名，在链接时将代码嵌入到最终的可执行文件中。

特点：

- 链接时直接合并到程序中。
- 可执行文件独立运行，无需外部依赖。

#### 动态库

动态库是独立的可执行模块，通常以 .so（Linux）、.dll（Windows）或 .dylib（macOS）为扩展名，在运行时动态加载。

特点：

- 链接时仅记录引用，运行时加载。
- 可共享，节省内存，但依赖库文件存在。

#### 区别

| 特性           | 静态库 (.a/.lib)       | 动态库 (.so/.dll)       |
| -------------- | ---------------------- | ----------------------- |
| 链接时机       | 编译时（链接阶段）     | 运行时（动态加载）      |
| 文件嵌入       | 嵌入到可执行文件中     | 不嵌入，独立文件        |
| 可执行文件大小 | 较大（包含库代码）     | 较小（仅含引用）        |
| 内存使用       | 每个程序独立占用       | 多个程序共享内存        |
| 更新性         | 需重新编译程序         | 替换库文件即可          |
| 依赖性         | 无运行时依赖           | 需确保库文件存在        |
| 加载速度       | 启动快（已嵌入）       | 启动稍慢（需加载）      |
| 平台扩展名     | .a (Linux), .lib (Win) | .so (Linux), .dll (Win) |

#### 链接方式

##### 静态库

在链接阶段，链接器（如 ld）将静态库中的目标代码提取并嵌入到可执行文件中。

步骤：

1. 编译源文件生成目标文件（.o）。
2. 用 ar 创建静态库。
3. 链接时指定静态库。

##### 动态库

链接时仅记录动态库的符号引用，运行时由操作系统加载器（如 ld.so）解析。

步骤：

1. 编译源文件生成动态库（需 -shared 和 -fPIC）。
2. 链接时指定动态库路径或名称。
3. 运行时确保库文件可找到。

## 10、多线程同步机制？

#### 互斥锁

互斥锁确保同一时刻只有一个线程访问临界区。

类型：

- `std::mutex`：基本互斥锁。
- `std::recursive_mutex`：允许同一线程多次锁定。

工作原理：

- 锁定（lock()）：线程获取锁，若已被占用则阻塞。
- 解锁（unlock()）：释放锁，允许其他线程获取。

工具：

- `std::lock_guard`：RAII 封装，自动解锁。
- `std::unique_lock`：更灵活，支持延迟锁定。

示例：

```c++
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx;
int counter = 0;

void increment(int id) {
    std::lock_guard<std::mutex> lock(mtx); // 自动加锁和解锁
    counter++;
    std::cout << "Thread " << id << " counter: " << counter << "\n";
}

int main() {
    std::thread t1(increment, 1);
    std::thread t2(increment, 2);
    t1.join();
    t2.join();
    std::cout << "Final counter: " << counter << "\n";
    return 0;
}
```

#### 读写锁

允许多个线程同时读，但写操作互斥。

C++ 支持：C++17 引入 `std::shared_mutex`。

工作原理：

- 共享锁（读锁）：多线程可同时获取。
- 独占锁（写锁）：仅一个线程获取。

示例：

```c++
#include <iostream>
#include <thread>
#include <shared_mutex>

std::shared_mutex rw_mtx;
int data = 0;

void reader(int id) {
    std::shared_lock<std::shared_mutex> lock(rw_mtx); // 读锁
    std::cout << "Reader " << id << " reads: " << data << "\n";
}

void writer(int id) {
    std::unique_lock<std::shared_mutex> lock(rw_mtx); // 写锁
    data++;
    std::cout << "Writer " << id << " writes: " << data << "\n";
}

int main() {
    std::thread r1(reader, 1);
    std::thread r2(reader, 2);
    std::thread w1(writer, 1);
    r1.join(); r2.join(); w1.join();
    return 0;
}
```

#### 条件变量

用于线程间同步，等待特定条件成立。

工具：`std::condition_variable`，需配合 `std::mutex`。

工作原理：

- `wait()`：释放锁并等待通知。
- `notify_one()/notify_all()`：唤醒等待线程。

示例：

```c++
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
bool ready = false;

void worker() {
    std::unique_lock<std::mutex> lock(mtx);
    cv.wait(lock, [] { return ready; }); // 等待 ready 为 true
    std::cout << "Worker proceeds\n";
}

void signaler() {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    {
        std::lock_guard<std::mutex> lock(mtx);
        ready = true;
    }
    cv.notify_one(); // 唤醒一个等待线程
}

int main() {
    std::thread t1(worker);
    std::thread t2(signaler);
    t1.join();
    t2.join();
    return 0;
}
```

#### 信号量

计数器，控制多个线程对有限资源的访问。

C++ 支持：C++20 引入 `std::counting_semaphore`。

工作原理：

- `acquire()`：减少计数，若为 0 则阻塞。
- `release()`：增加计数，唤醒等待线程。

示例：

```c++
#include <iostream>
#include <thread>
#include <semaphore>

std::counting_semaphore<2> sem(2); // 最多 2 个线程访问

void task(int id) {
    sem.acquire();
    std::cout << "Thread " << id << " enters\n";
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::cout << "Thread " << id << " exits\n";
    sem.release();
}

int main() {
    std::thread t1(task, 1);
    std::thread t2(task, 2);
    std::thread t3(task, 3);
    t1.join(); t2.join(); t3.join();
    return 0;
}
```

#### 原子操作

无锁操作，直接在硬件层面保证线程安全。

工具：`std::atomic`。

工作原理：使用 CPU 提供的原子指令（如 CAS，Compare-And-Swap）。

示例：

```c++
#include <iostream>
#include <thread>
#include <atomic>

std::atomic<int> counter(0);

void increment(int id) {
    counter.fetch_add(1); // 原子递增
    std::cout << "Thread " << id << " counter: " << counter << "\n";
}

int main() {
    std::thread t1(increment, 1);
    std::thread t2(increment, 2);
    t1.join(); t2.join();
    std::cout << "Final counter: " << counter << "\n";
    return 0;
}
```

#### 对比与选择

| 机制     | 互斥性 | 同步性 | 性能开销 | 适用场景     |
| -------- | ------ | ------ | -------- | ------------ |
| 互斥锁   | 是     | 否     | 中等     | 保护共享资源 |
| 读写锁   | 是     | 否     | 中等     | 读多写少     |
| 条件变量 | 否     | 是     | 中等     | 线程等待条件 |
| 信号量   | 是     | 是     | 中等     | 资源计数     |
| 原子操作 | 是     | 否     | 低       | 简单变量操作 |

## 11、生产者消费者线程同步机制用的什么？

在 C++ 中，生产者-消费者（Producer-Consumer）问题是一个经典的多线程同步场景，要求生产者线程生成数据并放入共享缓冲区，消费者线程从中取出数据处理。为了避免数据竞争和确保线程间的正确协作，通常使用特定的同步机制。

#### 互斥锁 + 条件变量

工具：

- `std::mutex`：保护缓冲区。
- `std::condition_variable`：通知缓冲区状态（空/满）。

工作原理：

- 互斥锁确保缓冲区操作的线程安全性。
- 条件变量让线程等待特定条件（如缓冲区非空或不满）。

优点：

- C++11 原生支持，灵活通用。
- 可处理复杂条件。

缺点：锁和条件检查有一定开销。

示例：

```c++
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <queue>

std::mutex mtx;
std::condition_variable cv;
std::queue<int> buffer;
const int BUFFER_SIZE = 5;
bool finished = false;

void producer(int id) {
    for (int i = 0; i < 10; ++i) {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [] { return buffer.size() < BUFFER_SIZE; }); // 等待不满
        buffer.push(i);
        std::cout << "Producer " << id << " produced: " << i << "\n";
        lock.unlock();
        cv.notify_one(); // 通知消费者
        std::this_thread::sleep_for(std::chrono::milliseconds(100)); // 模拟生产
    }
    std::lock_guard<std::mutex> lock(mtx);
    if (id == 1) finished = true; // 假设两个生产者，id=1 结束
}

void consumer(int id) {
    while (true) {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [] { return !buffer.empty() || finished; }); // 等待非空
        if (buffer.empty() && finished) break;
        if (!buffer.empty()) {
            int data = buffer.front();
            buffer.pop();
            std::cout << "Consumer " << id << " consumed: " << data << "\n";
        }
        lock.unlock();
        cv.notify_one(); // 通知生产者
        std::this_thread::sleep_for(std::chrono::milliseconds(150)); // 模拟消费
    }
}

int main() {
    std::thread p1(producer, 1);
    std::thread p2(producer, 2);
    std::thread c1(consumer, 1);
    std::thread c2(consumer, 2);

    p1.join(); p2.join();
    c1.join(); c2.join();
    return 0;
}
```

#### 信号量

工具：C++20 的 std::counting_semaphore。

工作原理：

- 一个信号量表示空闲槽位（生产者用）。
- 另一个信号量表示已有数据（消费者用）。

优点：

- 直观，计数器天然适合缓冲区管理。
- 无需显式条件检查。

缺点：C++20 前需自己实现或用第三方库。

示例：

```c++
#include <iostream>
#include <thread>
#include <semaphore>
#include <queue>

std::queue<int> buffer;
std::counting_semaphore<5> sem_empty(5); // 空闲槽位
std::counting_semaphore<5> sem_full(0);  // 数据项
std::mutex mtx;

void producer(int id) {
    for (int i = 0; i < 10; ++i) {
        sem_empty.acquire(); // 等待空位
        {
            std::lock_guard<std::mutex> lock(mtx);
            buffer.push(i);
            std::cout << "Producer " << id << " produced: " << i << "\n";
        }
        sem_full.release(); // 通知有数据
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
    }
}

void consumer(int id) {
    for (int i = 0; i < 10; ++i) {
        sem_full.acquire(); // 等待数据
        int data;
        {
            std::lock_guard<std::mutex> lock(mtx);
            data = buffer.front();
            buffer.pop();
            std::cout << "Consumer " << id << " consumed: " << data << "\n";
        }
        sem_empty.release(); // 释放空位
        std::this_thread::sleep_for(std::chrono::milliseconds(150));
    }
}

int main() {
    std::thread p(producer, 1);
    std::thread c(consumer, 1);
    p.join();
    c.join();
    return 0;
}
```

#### 原子操作 + 无锁队列

工具：std::atomic + 自定义无锁队列。

工作原理：

- 使用原子变量（如头尾指针）实现无锁读写。
- 避免锁的开销。

优点：高性能，适合高并发。

缺点：实现复杂，易出错。

示例：无锁队列实现较复杂，这里仅示意

```c++
#include <iostream>
#include <thread>
#include <atomic>
#include <vector>

std::atomic<int> counter(0);

void producer() {
    for (int i = 0; i < 10; ++i) {
        counter.fetch_add(1); // 原子递增
        std::cout << "Produced: " << i << "\n";
    }
}

void consumer() {
    while (counter.load() < 10) {
        int val = counter.load();
        std::cout << "Consumed: " << val << "\n";
    }
}

int main() {
    std::thread p(producer);
    std::thread c(consumer);
    p.join();
    c.join();
    return 0;
}
```

## 12、系统调用是什么？new是系统调用还是用户调用？

#### 什么是系统调用？

系统调用 是用户程序（运行在用户态）请求操作系统内核（运行在内核态）提供服务的接口。

作用：

- 用户程序无法直接访问硬件或内核资源（如文件、内存、网络），需通过系统调用请求服务。
- 系统调用将控制权从用户态切换到内核态，执行后返回用户态。

#### 工作原理

用户态 vs 内核态：

- 用户态：应用程序运行的环境，权限受限。
- 内核态：操作系统内核运行的环境，可访问硬件。

调用过程：

1. 用户程序调用封装函数（如 C 库的 read）。
2. 触发系统调用指令（如 syscall 或 int 0x80）。
3. CPU 切换到内核态，执行内核服务。
4. 返回结果，切换回用户态。

#### new 是系统调用还是用户调用？

new **不是系统调用**，而是**用户态调用**。

在 C++ 中，new 是一个操作符，用于动态分配内存并构造对象。

new 的工作：

1. 内存分配：调用内存分配函数（如 malloc 或底层分配器）。
2. 对象构造：调用构造函数初始化对象。

用户态部分：

- new 本身是 C++ 标准库的一部分，运行在用户态。
- 它通常通过调用 C 标准库的 malloc 来分配内存。

内核态部分：malloc 最终可能触发系统调用（如 Linux 的 brk 或 mmap）向操作系统请求内存。

## 13、系统调用流程？

系统调用是用户程序请求操作系统内核提供服务的关键机制。系统调用的流程涉及用户态到内核态的切换、内核服务的执行以及结果的返回。

#### 用户态准备

用户程序调用高级接口（如 C 库函数），设置系统调用所需的参数。

作用：

- 将请求转化为标准格式（如系统调用号和参数）。
- 通过库函数封装，避免直接操作底层。

实现：

- C 标准库（如 libc）提供封装函数。
- 示例：read、write、open。

细节：

- 参数通常存放在寄存器或栈中。
- 系统调用号（唯一标识服务的整数）由库函数设置。

#### 触发系统调用

用户程序通过特定指令触发系统调用，切换到内核态。

作用：将控制权从用户态交给内核。

实现：

- Linux：

  - 老方法：软件中断 int 0x80。

  - 新方法：syscall 指令（x86_64，更高效）。

- Windows：使用 int 0x2e 或 sysenter。

细节：

- 系统调用号和参数通过寄存器传递：x86_64：rax（调用号）、rdi、rsi、rdx 等（参数）。
- CPU 执行指令，切换到内核态。

#### 内核态执行

操作系统内核接收请求，执行对应的服务。

作用：

- 处理用户请求（如读文件、分配内存）。
- 访问硬件或其他内核资源。

实现：

- 内核维护系统调用表（System Call Table），根据调用号分派服务。
- 示例（Linux）：read → sys_read（内核函数）。

步骤：

1. 参数校验：检查参数合法性（如文件描述符有效性）。
2. 服务执行：调用底层驱动或内核逻辑。
3. 结果准备：将返回值写入寄存器或缓冲区。

#### 返回用户态

内核完成服务后，将结果返回用户程序，切换回用户态。

作用：将执行结果（如返回值、错误码）传递给用户。

实现：

- 使用返回指令（如 sysret 或 iret）。
- 返回值通常存放在寄存器（如 rax）。

细节：

- Linux：负值表示错误（如 -errno）。
- 用户态库将结果转换为标准形式（如 errno）。

#### 流程示例

```c++
#include <unistd.h>
#include <iostream>

int main() {
    char buf[10];
    ssize_t n = read(0, buf, 10); // 从标准输入读取
    if (n >= 0) {
        write(1, buf, n); // 输出到标准输出
    }
    return 0;
}
```

（1）用户态准备：

- 调用 read(0, buf, 10)。

- libc设置：
  - rax = 0（read 的系统调用号）。
  - rdi = 0（stdin）。
  - rsi = buf 地址。
  - rdx = 10。

（2）触发系统调用：

执行 syscall，切换到内核态。

（3）内核态执行：

- 内核查找 sys_read，从标准输入读取数据到 buf。
- 返回字节数（如 5）到 rax。

（4）返回用户态：

- sysret 切换回用户态，n = 5。
- write 类似流程输出数据。

