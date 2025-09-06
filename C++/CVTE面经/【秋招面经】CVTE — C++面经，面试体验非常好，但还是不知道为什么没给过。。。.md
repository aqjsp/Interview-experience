# 【秋招面经】CVTE — C++面经，面试体验非常好，但还是不知道为什么没给过。。。

> 来源：https://www.nowcoder.com/discuss/793241858179600384

1、4种智能指针有哪些分类？分别有什么作用？

2、lambda有什么作用，C++11-20各有什么什么改进等？捕获的类型有哪几种，有啥作用如果在lambda函数中捕获this指针，会不会有什么风险？

3、enable_shared_from_this()和shared_from_this()的作用和原理是什么？以及这两个函数的使用场景？

4、常用的STL容器有什么？

5、vector的扩容机制是怎么样的？扩容会涉及元素的移动和复制，那么具体是怎么判断移动还是复制的，如果是复制的话是深拷贝还是浅拷贝？

6、如果循环里面用迭代器去遍历map的话，什么情况下可能导致迭代器失效？

7、class和struct有什么区别？

8、虚函数的实现机制？虚函数表和虚表指针的内存空间布局，他们的分别存放在哪里？

9、构造函数里面可以使用this指针吗？

10、进程和线程的区别？进程的通信方式有哪些？介绍一下

11、线程间的同步和互斥怎么解决？

12、死锁？

13、new和malloc的区别是什么？

14、如果我只有一个2G的物理内存，new一个4G的对象，能实现吗？为什么

15、传输层有哪些协议？

16、在Linux系统下，有一个client和server，他们要建立连接通信需要调用哪些函数？

17、编码题目，两个线程交替打印字符串，一个打印"1234",一个打印”abcd"，要求打印输出为"1a2b3c4d"这种类型？

## 1、4 种智能指针有哪些分类？分别有什么作用？

可以看我之前专门总结的：[智能指针全解析](https://mp.weixin.qq.com/s/NIxtfgjND_Y0HJJ_yymkvQ)

### `std::unique_ptr<T>`

- 语义：独占所有权（single ownership）。不可拷贝，可移动。
- 用途：表达唯一资源所有权、用于 RAII、替代裸指针用于对象生命周期管理。轻量、无引用计数开销。
- 示例：

```cpp
auto p = std::make_unique<Foo>(args);
std::unique_ptr<Foo> q = std::move(p); // 转移所有权
```

### `std::shared_ptr<T>`

- 语义：共享所有权（引用计数 control block）。最后一个 `shared_ptr` 销毁时释放资源。
- 用途：多个持有者场景（观察者、缓存、任务系统），需要注意循环引用（用 `weak_ptr` 打破）。
- 示例：

```cpp
auto sp = std::make_shared<Foo>(args);
auto sp2 = sp; // 引用计数 +1
```

### `std::weak_ptr<T>`

- 语义：观察者指针，不增加引用计数，不拥有资源。通过 `lock()` 获取 `shared_ptr`（如果资源还活着）。
- 用途：打破 `shared_ptr` 循环引用、缓存中的弱引用、临时访问。
- 示例：

```cpp
std::weak_ptr<Foo> wp = sp;
if (auto s = wp.lock()) { /* safe to use s */ }
```

### `std::auto_ptr<T>`（已弃用）

- 语义：C++98 的智能指针，拷贝是转移语义（有问题），已在 C++11 标准被废弃并由 `unique_ptr` 取代。**不要在新代码中使用**。

补充

- `boost::intrusive_ptr`：计数存在对象内部，适合某些嵌入式/低内存场景。
- 设计原则：能用 `unique_ptr` 就用 `unique_ptr`，只有确实需要共享所有权时才用 `shared_ptr`。

## 2、Lambda 的作用、C++11→C++20 演进、捕获类型、`this` 捕获风险

### Lambda 的作用（为什么用）

- 本地函数对象：像内联的匿名函数，适合回调、算法短闭包、局部策略等。
- 捕获外部状态：相比普通函数，lambda 能捕获外部变量（值或引用），更方便编写闭包逻辑。
- 类型轻量：编译期生成闭包类型（函数对象），可内联优化，无堆分配（除非你显式 `new`）。

### C++11 / C++14 / C++17 / C++20 的主要改进

- **C++11**：引入 lambda。支持捕获列表（`[=]`, `[&]`, `[a, &b]`）、`mutable`、显式返回类型 `-> R`、能捕获 `this`（捕获 `this` 是捕获指针）。（参考：cppreference 总览）([CPP参考](https://en.cppreference.com/w/cpp/language/lambda.html?utm_source=chatgpt.com))
- **C++14**：两个重要增强：
  - 泛型 lambda：参数可以用 `auto`，使 lambda 成为模板。
  - 初始化捕获：例如 `[x = std::move(vec)]`，可以在捕获时做 move 或计算表达式，从而实现“move capture”。（这点便于把不可拷贝资源移入闭包中）。
- **C++17**：若干细节改进（比如允许用 `[=, *this]` 捕获对象副本 —— 把 `*this` 的值拷贝到闭包里），并增强了 `constexpr` 支持（能把一些 lambda 标记为 `constexpr`，在编译期求值更友好）。（可把 `*this` 按值捕获以避免仅捕指针）
- **C++20**：**模板参数列表的 lambda**（即显式模板参数列表 `[]<typename T>(T x){...}`，增强泛型 lambda 的可读性和能力），并对捕获/默认构造/拷贝行为等做了若干修正以提升一致性。

> 总结：从 C++11 的基础闭包到 C++14 的泛型/初始化捕获，再到 C++20 的模板化语法，lambda 越来越灵活且可以安全地移动/持有资源。

### 捕获的类型（几种常见形式）

- 捕值（by value）：`[x]` 或 `[=]` 把外部值拷贝到闭包成员中。拷贝语义由被捕变量类型决定（若是可移动可用 `move` 初始化）。
- 捕引用（by reference）：`[&x]` 或 `[&]` 捕外部变量的引用（闭包保存引用）。风险是闭包比被捕对象存活时间长会悬垂引用。
- 捕 `this`（捕指针）：`[this]` 捕 `this` 指针（注意：保存的是指针，不是完整对象）。C++17 可写为 `[*this]` 把对象拷贝（按值）到闭包中。
- 初始化捕获（C++14）：`[m = std::move(x)]`。可用于将独占资源移动进闭包。([流利的C++](https://www.fluentcpp.com/2021/12/13/the-evolutions-of-lambdas-in-c14-c17-and-c20/?utm_source=chatgpt.com))

### 捕获 `this` 的风险

- 捕获 `this`（或 `[=]` 默认捕值含 `this`）**只是把 `this` 指针保存进闭包**。若闭包在对象销毁后调用，会出现悬挂指针（UB）。
- 安全做法：
  - 如果闭包执行期可能超出对象生存期，**不要捕 `this`**；改捕 `weak_ptr`（对象由 `shared_ptr` 管理）或用 `[*this]`（C++17，把对象拷贝到闭包中）或使用 `enable_shared_from_this` + `shared_ptr`（见下一题）。
  - 使用 `init-capture` 以 `std::move` 把资源搬入闭包，从而避免引用到外部临时/局部变量。

## 3、`enable_shared_from_this()` 与 `shared_from_this()`：作用、原理、使用场景

### 问题出发点

当一个对象需要在其成员函数内部生成一个指向自身的 `shared_ptr`（比如把 `this` 交给异步任务，使其被延长生存期），如果你直接 `std::shared_ptr<T>(this)`，会产生两个独立的 control block（双重管理），最终会发生双重 delete。为避免这个问题，用 `enable_shared_from_this`。

### `enable_shared_from_this<T>` 做了什么（原理）

- `enable_shared_from_this` 在对象中保存一个弱引用 `weak_ptr<T>`（隐含成员），当对象第一次由 `shared_ptr<T>` 管理时（例如通过 `std::make_shared<T>`），`shared_ptr` 的实现会把自己的 control block 的弱引用指向这个对象填入 `enable_shared_from_this` 的内部弱指针（实现细节由库实现）。
- 之后在对象内部可以调用 `shared_from_this()`，该方法会返回一个指向相同 control block 的 `shared_ptr`（引用计数正确增长），不会产生新的 control block。

### 使用示例

```cpp
struct My : std::enable_shared_from_this<My> {
    std::shared_ptr<My> getPtr() {
        return shared_from_this(); // safe: returns shared_ptr that shares ownership
    }
};

int main() {
    auto p = std::make_shared<My>();
    auto p2 = p->getPtr(); // p and p2 share same control block
}
```

### 注意事项 / 陷阱

- **必须**把对象**最初**用 `shared_ptr` 管理（例如 `make_shared` 或 `shared_ptr<T>(new T)`）。若对象直接在栈上创建或用 `unique_ptr` 管理再调用 `shared_from_this()` 会出错（通常抛 `std::bad_weak_ptr` 或 UB）。
- 如果想在构造函数里获取 `shared_ptr`（把 `shared_from_this()` 放在构造里），通常也会失败，因为 control block 还没建立：**不要在构造过程中调用 `shared_from_this()`**，除非你知道 object 已被 shared_ptr 管理（工厂函数模式可解决）。
- 场景：回调注册、异步操作发出 `shared_ptr` 给队列/线程池以延长对象生命周期、缓存需要弱指针检查时配合 `weak_ptr`。

## 4、常用的 STL 容器（概览）

已全面总结的STL文章：[STL全解析](https://mp.weixin.qq.com/s/mIbS7Rqnd6Tim1at3_iG-w)

- **序列容器（sequence）**
  - `std::vector<T>`：动态数组，随机访问 O(1)。首选容器（缓存友好）。
  - `std::deque<T>`：双端队列，支持两端高效插入。
  - `std::list<T>`：双向链表，节点稳定地址、插入/删除 O(1)，但不缓存友好，迭代成本高，近年来用得少。
- **关联容器（有序）**
  - `std::map<Key,T>` / `std::set<Key>`：基于自平衡树（通常红黑树），有序、支持 range query、O(log n)。适用于需要有序遍历或前驱/后继操作。
- **无序关联容器（哈希）**
  - `std::unordered_map` / `std::unordered_set`：哈希表，平均 O(1) 查找/插入/删除，适合快速精确匹配。注意 rehash/迭代器失效语义。
- **容器适配器**
  - `std::stack`, `std::queue`, `std::priority_queue`：在已有容器上封装后端行为。
- **字符串**
  - `std::string`：按需管理字符数组（像 `vector<char>`），注意短字符串优化和可变/不可变语义（C++17 起 `std::string` 的小调整）。
- **其他**
  - `std::array<T, N>`：固定长度数组，栈分配，轻量。
  - `std::span<T>`（C++20）：视图，不拥有元素，适合函数参数传递。

## 5、`vector` 的扩容机制是怎么样的？移动还是复制？深拷贝还是浅拷贝？

### 扩容策略（总体）

- `std::vector` 用连续内存。当 `size() == capacity()` 时，再插入会触发 **重新分配（reallocate）**：分配更大的内存块，把现有元素搬到新内存，然后释放旧内存。
- **增长因子**是**实现定义**的，但主流实现（libstdc++ / MSVC）通常采用**几何增长**（常见是 *约为倍增*），以保证 `push_back` 的摊销复杂度为常数（amortized O(1)）。你不能依赖精确倍数，应该通过 `reserve()` 预留以避免扩容。([CPP参考](https://en.cppreference.com/w/cpp/container/vector.html?utm_source=chatgpt.com))

### 移动还是复制？（何时用 move，何时用 copy）

- 在 C++11 之后，当 `vector` 重新分配时，库会尝试 **移动构造**已有元素到新内存以减少开销，但 **前提**是移动构造不会抛异常（或满足某些条件）。为了保证强异常安全，标准库在搬迁时通常使用 `std::move_if_noexcept` 的策略：
  - 如果 `T` 的移动构造是 `noexcept`（或 `T` 不可拷贝），则使用移动构造。
  - 否则可能使用拷贝构造以维持异常安全（对某些实现还有细节，但总体思想是“只在安全的情况下使用移动”）。([CPP参考](https://en.cppreference.com/w/cpp/utility/move_if_noexcept.html?utm_source=chatgpt.com))

### 拷贝是深拷贝还是浅拷贝？

- **这取决于类型 `T` 的拷贝/移动构造函数**：对内置类型或 POD（plain-old-data）是按位复制；对用户类型，copy/move 构造由类型实现决定（可能是深拷贝、引用计数的浅复制、或其他语义）。`vector` 只会调用 `T` 的构造/移动/拷贝函数，那么语义由 `T` 本身定义。

### 工程建议

- 对能支持 `noexcept` 移动的类型，**标记移动构造为 `noexcept`**，以帮助容器在扩容时使用 move（提高性能）。
- 对元素生命周期敏感或拷贝代价大的场景，调用 `reserve(n)` 预分配以避免多次 reallocation。

## 6、在循环里面用迭代器遍历 `map` 时，什么时候会导致迭代器失效？

### `std::map` / `std::set`（基于树）

- **插入（insert）**：**不会**失效已有迭代器。
- **删除（erase）**：只**失效被删除的那些迭代器**，而其他迭代器保持有效。
- 因此在 `for (auto it = m.begin(); it != m.end(); ++it)` 中删除当前元素时要小心：**先记录下 `it_next = std::next(it)`，再 `m.erase(it)`, 然后 `it = it_next`**；或者使用 `it = m.erase(it)`（从 C++11 起某些 overload 返回下一个 iterator）。查看标准实现细节便可确认 `erase` 重载返回值。([CPP参考](https://en.cppreference.com/w/cpp/container/map.html?utm_source=chatgpt.com))

### `std::unordered_map`

- **插入**：通常不会失效已有迭代器，**但**当触发 rehash（例如负载因子过高）时，会重新分配桶并**使所有迭代器失效**（因为底层数组变了）。
- **erase**：会失效被删除的迭代器，但其他迭代器通常保持有效（详见标准和实现）。若你在遍历过程中进行可能触发 rehash 的插入，不能假设迭代器有效。([CPP参考](https://en.cppreference.com/w/cpp/container/unordered_map.html?utm_source=chatgpt.com))

### 小结（实践准则）

- 如果在迭代过程中做 `erase`，使用安全模式：`it = container.erase(it);`（若该重载返回 iterator），或 `auto next = std::next(it); container.erase(it); it = next;`。
- 避免在遍历时做会触发 rehash/resize 的插入；若必须，考虑先收集要插入/删除的 key，遍历结束后统一执行变更。

## 7、`class` 和 `struct` 有什么区别？

在 C++ 中 **语法上**的区别非常小：

- **默认访问权限**：
  - `struct`：成员和基类默认 **public**。
  - `class`：成员和基类默认 **private**。
- **其他**：没有其它语义差别（都支持继承、虚函数、成员、模版等）。
  - 习惯上：`struct` 常用于**聚合类型 / POD / 数据包**，`class` 常用于**封装良好、有行为（方法）的对象**。

示例：

```cpp
struct S { int x; };      // x 默认 public
class C { int x; };      // x 默认 private
```

## 8、虚函数的实现机制？vtable 与 vptr 的内存布局在哪里存放？

### 基本实现思路（常见编译器实现）

- **虚函数表（vtable）**：对每个含虚函数的类，编译器生成一个静态的“虚表”——一个函数指针数组（指向该类或覆盖函数的实现）。这个表通常被放在只读数据段（程序静态区）。
- **虚表指针（vptr）**：每个对象通常含有一个指向其类 vtable 的隐藏指针（`vptr`），常放在对象的内存开始处（编译器实现细节）。当创建对象时，构造函数会把 `vptr` 指向对应类的 vtable。
- **调用过程**：通过基类指针调用虚函数时，编译器生成代码读取对象的 `vptr`，然后从 vtable 中取出对应 slot 的函数指针并跳转调用（动态绑定）。

### 多重继承 / 虚继承 情况

多重继承或虚继承时，编译器可能需要 **多个 `vptr`** 或在 vtable 中保存额外偏移信息来修正 `this` 指针。实现更复杂，但基本思想不变：动态查表并调用，对象内可能有多个指针或隐形偏移。

### 存放位置

- **vtable**：编译时生成，放在程序的静态/只读数据段（不在每个对象里重复存放），可被多个对象共享。
- **vptr**：每个对象的一块内存（对象头部或实现选择的位置），占用对象内存（通常 8 字节/指针大小）。

### 小结

虚函数的动态分派靠 vptr→vtable 的查表机制实现；这是运行时多态的常见实现方式。实现细节（位置、是否有多 vptr）取决于编译器与继承模型。

## 9、构造函数里可以使用 `this` 指针吗？

**可以使用 `this`，但要小心**。

主要注意两点：

1. **对象尚未完全构造**：在基类构造期间，虚函数的动态绑定仍然指向基类（派生类的虚表尚未安装）；因此，在构造函数内部调用虚函数**会调用当前类（正在构造的类）的版本**，而不是派生类覆盖版本（这是语言定义，不是 BUG）。
2. **成员的初始化顺序**：使用 `this` 访问成员前必须确保该成员已初始化（成员按声明顺序初始化）。在初始化列表里可以引用前面已初始化的成员，但在成员尚未初始化使用会产生未定义行为。

示例问题点：

```cpp
struct Base {
    Base() { foo(); }          // 在 Base 构造阶段，虚调用会解析到 Base::foo
    virtual void foo() { /* base */ }
};
struct Derived : Base {
    Derived() : Base() {}
    void foo() override { /* derived */ } // 不会被 Base() 内部的 foo() 调用
};
```

**工程建议**：

不要在构造/析构过程中依赖虚函数多态行为。若需要构造期间调用可覆盖行为，采用工厂函数（对象构造后再调用 init 方法）或把可变部分外置。

## 10、进程和线程的区别？进程的通信方式有哪些？

### 进程（process） vs 线程（thread）

- 进程：拥有独立的地址空间（虚拟地址空间）、文件描述符表（可以继承）、资源表等。进程间隔离较好，崩溃影响小。
- 线程：属于同一进程、共享进程地址空间、文件描述符等资源；每线程有自己栈和寄存器上下文。线程切换成本比进程低（通常），通信更容易（共享内存），但需要同步以防竞态。

### 进程间通信（IPC）方式（Linux/Unix 常用）

- 匿名/命名管道（pipe / FIFO）：半双工/全双工（pipe），适合父子或本地进程间通信。
- Unix Domain Socket（本地域套接字）：比 TCP 快、能传递文件描述符。
- TCP/UDP Socket：进程可通过网络协议通信（本机也可通过 loopback）。
- 共享内存：最快的 IPC（零拷贝），但需要同步机制（信号量、futex 等）。
- 消息队列：内核维护的消息队列，进程发送接收消息。
- 信号：异步通知（但信息量小），适用于简单通知/中断。
- RPC / gRPC / DBus 等高级机制：用于分布式或复杂进程交互。

## 11、线程间的同步和互斥怎么解决？

### 基本原语（C++ 标准库）

- `std::mutex` + `std::lock_guard` / `std::unique_lock`：互斥访问临界区。用 RAII 防止死锁。
- `std::recursive_mutex`：允许同一线程多次锁（谨慎使用）。
- `std::shared_mutex`（C++17）：读写锁，`shared_lock` 多读、独占写。
- `std::condition_variable`：线程间等待/通知（生产者-消费者）。
- `std::atomic<T>`：原子操作、无锁计数（对简单标量非常高效），支持 `fetch_add`, `compare_exchange_weak/strong` 等。

### 高级/系统级

- 信号量（POSIX sem_t / C++20 `std::counting_semaphore`）。
- futex（Linux 用户态快速等待机制，用于实现高性能锁）。
- lock-free 数据结构（基于 CAS / 原子操作），并发库（Folks：concurrent hash maps）。

### 实战建议

- 优先使用标准库（`std::mutex`、`std::atomic`）和 RAII（`std::lock_guard`）。
- 对短临界区使用自旋锁或 CAS；对长时间阻塞使用阻塞式互斥。
- 尽量减少锁粒度，避免持有锁做 IO/阻塞操作。
- 对复杂并发使用现成成熟的数据结构或并发库，避免自己实现复杂无锁算法（容易出错：ABA 等问题）。

## 12、死锁（Deadlock）：产生条件、预防与解决

### 四个必要条件（Coffman 条件）

1. 互斥（Mutual Exclusion）：资源一次只被一个线程占用。
2. 持有并等待（Hold and Wait）：线程持有资源并等待其它资源。
3. 不可剥夺（No Preemption）：资源不能被强行剥夺，必须由占有者释放。
4. 循环等待（Circular Wait）：存在一个资源等待的环。

### 预防 / 避免策略

- 资源有序分配：按固定顺序获取锁（全局顺序），破坏循环等待。
- 尝试获取（try_lock）+ 回溯：若无法获得，释放已持有锁并重试。
- 避免持锁做阻塞操作：减少持锁时间。
- 使用锁层次与超时：例如 `std::timed_mutex`，超时后回退。
- 死锁检测：收集锁等待图，后台检测环路并采取回滚策略（复杂，通常用于数据库等系统）。
- 使用事务/乐观并发：通过版本、回滚避免锁的使用。

### 例子（两把锁导致死锁）

```cpp
std::mutex m1, m2;
void t1() {
    std::lock_guard<std::mutex> l1(m1);
    std::this_thread::sleep_for(...);
    std::lock_guard<std::mutex> l2(m2); // 可能等待
}
void t2() {
    std::lock_guard<std::mutex> l2(m2);
    std::this_thread::sleep_for(...);
    std::lock_guard<std::mutex> l1(m1); // 可能等待 -> deadlock
}
```

解决：始终以相同顺序获取锁，或使用 `std::scoped_lock(m1, m2)`（C++17，避免死锁）。

## 13、`new` 和 `malloc` 的区别

| 特性          | `new` / `delete`                                         | `malloc` / `free`                  |
| ------------- | -------------------------------------------------------- | ---------------------------------- |
| 调用构造/析构 | `new` 会调用构造函数，`delete` 调用析构函数              | 不调用构造或析构（裸内存）         |
| 返回类型      | 类型化指针（`T*`）                                       | 返回 `void*`，需强转               |
| 失败行为      | 抛 `std::bad_alloc`（可用 `nothrow` 版本返回 `nullptr`） | 返回 `NULL`                        |
| 重载          | `operator new/delete` 可被重载                           | 通常不重载（实现层）               |
| 对象数组      | `new[]`/`delete[]` 具有元素析构语义                      | `malloc` 分配数组，需手工构造/析构 |

**placement new**：`new (buf) T(args...)` 在已有内存位置构造对象，须显式调用析构 `obj->~T()`。

## 14、只有 2GB 物理内存，`new` 一个 4GB 对象，能实现吗？为什么？

**区别：虚拟内存 vs 物理内存**

在现代操作系统（有虚拟内存）下，**分配（allocation）和实际物理页面分配（物理内存或交换）是不同的事**：

- `new`（或 `malloc`）通常向操作系统请求虚拟地址空间（通过 `mmap`/`brk`），操作系统可以采用 **overcommit**（允许分配超出物理内存的虚拟内存），即直到程序真正访问这些页面时（触发缺页），系统才为页面提供物理内存或 swap。
- 因此 **仅凭物理内存小于请求大小并不一定会导致申请失败**；是否成功依赖于 OS 的 overcommit 策略（Linux `/proc/sys/vm/overcommit_memory`）、可用 swap、以及当前内存压力。

**结论**：

- 如果系统允许 overcommit，`new` 可能返回一个指针（当你实际写入这些页面时，内核再分配物理页或用 swap）；在内存不足或 overcommit 被禁用且无法分配对应物理页时，分配会失败（抛 `std::bad_alloc`）或在触发 OOM 时被内核终结。
- **实际可用性取决于操作系统策略与运行时内存压力**，所以不能保证在只有 2GB 物理内存的机器上安全 `new` 出 4GB 并能成功使用。

## 15、传输层有哪些协议？

常见传输层协议（OSI 模型中的 Transport Layer）：

- TCP：面向连接、可靠、字节流、顺序传输、拥塞控制与重传机制。
- UDP：无连接、不可靠、无顺序保证、低开销，适合实时音视频/简单协议/自实现可靠性。
- SCTP：面向消息、多流、部分可靠选项、四次握手、支持多宿主（multi-homing）。
- DCCP：提供拥塞控制的无连接数据报服务。
- QUIC：虽然底层走 UDP，但提供了类似传输层的特性（多路复用、可靠性、TLS 1.3 集成、连接迁移）。在很多文献里 QUIC 被视为“下一代传输协议”。

## 16、在 Linux 下，client/server 建立连接通信需要调用哪些函数？

以经典 TCP socket 编程为例（IPv4 简化流程）：

### Server（监听端）

```text
socket() -> bind() -> listen() -> accept() -> recv()/send() -> close()
```

示例步骤：

1. `int s = socket(AF_INET, SOCK_STREAM, 0);`
2. `setsockopt(s, SOL_SOCKET, SO_REUSEADDR, ...)`（可选）
3. `bind(s, ...)`：绑定地址/端口
4. `listen(s, backlog)`：进入监听
5. `int c = accept(s, &peer_addr, &peer_len)`：接受连接（阻塞或非阻塞）
6. 使用 `recv/send` 或 `read/write` 与客户端通信
7. `close(c)` 然后最终 `close(s)`

### Client（主动发起端）

```text
socket() -> connect() -> send()/recv() -> close()
```

示例步骤：

1. `int s = socket(AF_INET, SOCK_STREAM, 0);`
2. `connect(s, server_addr, addrlen)`：发起连接（三次握手）
3. `send/recv` 或 `read/write`
4. `close(s)`

**DNS / 地址解析**：现代代码通常使用 `getaddrinfo()` 进行地址解析以支持 IPv4/IPv6。
 **非阻塞/异步 I/O**：`select/epoll/poll` 或异步框架（libuv、asio）用于高并发。

## 17、编码题：两个线程交替打印字符串（线程 A 打印 `"1234"`，线程 B 打印 `"abcd"`），要求输出 `"1a2b3c4d"` 

给一个简单且稳健的 C++11+ 实现，使用 `std::mutex` + `std::condition_variable` 做线程间同步（按字符逐个交替打印）：

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

int main() {
    const std::string nums = "1234";
    const std::string chars = "abcd";
    std::mutex m;
    std::condition_variable cv;
    bool turn_num = true; // true -> number's turn; false -> char's turn
    size_t i = 0, j = 0;
    std::string result;
    result.reserve(nums.size() + chars.size());

    auto tnum = std::thread([&](){
        while (i < nums.size()) {
            std::unique_lock<std::mutex> lk(m);
            cv.wait(lk, [&]{ return turn_num; }); // wait until number's turn
            result.push_back(nums[i++]);
            turn_num = false;
            lk.unlock();
            cv.notify_one();
        }
    });

    auto tchar = std::thread([&](){
        while (j < chars.size()) {
            std::unique_lock<std::mutex> lk(m);
            cv.wait(lk, [&]{ return !turn_num; }); // wait until char's turn
            result.push_back(chars[j++]);
            turn_num = true;
            lk.unlock();
            cv.notify_one();
        }
    });

    tnum.join();
    tchar.join();

    std::cout << result << '\n'; // 输出: 1a2b3c4d
    return 0;
}
```

**解释**：

- 用 `turn_num` 标记当前轮到哪一线程打印。每次打印完该线程修改 `turn_num` 并 `notify_one()` 唤醒对方。
- 该代码假设两个字符串长度相同（若长度不同，需在循环中做边界处理与最后唤醒保证）。
- 这是最直接、可读的方式；也可用 `semaphore`、`atomic` + busy-wait（不推荐）或更高级的协程/管道方式。