# 腾讯天美游戏C++一面

> 来源：https://www.nowcoder.com/feed/main/detail/bdc8b50f18aa457d8c432a629f71aa97

## 1、多态（Polymorphism）是什么，如何实现的？

**定义（分两类）**

- 编译期（静态）多态：通过模板、函数/运算符重载在编译期决定哪个函数体被选用（例：`template`、`inline`、函数重载）。优点：无运行时开销，能内联。
- 运行时（动态）多态：通过基类指针或引用在运行时根据对象的实际类型选择具体实现（例：虚函数）。这是面向对象设计中常说的“多态”。

**实现（运行时多态）**

- 由虚函数表（vtable） + 对象内的虚函数指针（vptr）实现：
  - 编译器为每个含虚函数的类生成一个 `vtable`（表中存函数地址、RTTI、偏移信息等）。
  - 每个对象包含一个 `vptr`（在对象内，通常第一个成员位置），指向对应类的 `vtable`。
  - 虚函数调用时，编译器生成代码：读取 `p->vptr`，从 `vtable` 的对应 slot 取出函数指针，间接调用。
- 构造/析构期间：`vptr` 在每一级构造/析构时由编译器设置为当前阶段对应类的 `vtable`，因此构造期间调用虚函数不会转发到更派生类。

示例

```cpp
struct Base { virtual void f(); };
struct Derived : Base { void f() override; };
Base* p = new Derived;
p->f(); // 通过 vptr/vtable 动态分派到 Derived::f
```

## 2、虚函数的实现机制？

**核心机制**

- 每个有虚函数的类通常对应一个静态 vtable（编译器/链接器生成，在只读数据区如 `.rodata`）。
- 每个对象有一个隐藏 vptr（存在对象内，可以在栈、堆或静态区，取决于对象在哪里分配）。
- 虚函数调用实现为：间接函数调用（读取 vptr，再从 vtable 中读取函数地址然后 `call`）。

**多继承/虚继承的复杂性**

- 多继承：对象可能包含多个 vptr（每个基类子对象一个）；vtable slot 可能存放 thunk 或需要 `this` 调整。
- 虚继承：vtable 中会包含 offset-to-top、virtual base offset 等信息，运行时需要额外查找/调整以定位虚基子对象。

**其它实现细节**

- vtable 在某个翻译单元发射（key function 策略），遵守 ABI（如 Itanium ABI 或 MSVC ABI）。
- `dynamic_cast` / `typeid` 使用 vtable 中的 RTTI 信息。
- `pure virtual` 未定义行为：在构造/析构阶段或未正确初始化 vptr 时调用纯虚会报错（运行时“pure virtual call”）。

## 3、内存区域除了堆和栈有哪些？

- 代码段（Text / .text）：存放可执行指令。
- 只读数据段（.rodata）：存放常量字符串、静态 const 等。
- 已初始化数据段（Data / .data）：存放已初始化的全局/静态变量。
- 未初始化数据段（BSS / .bss）：存放未初始化或初始化为 0 的全局/静态变量。
- 堆（Heap）：动态分配（`malloc`/`new`），由 malloc 实现的 arena 管理。
- 栈（Stack）：局部变量、函数调用帧，线程各自拥有栈。
- 内存映射区（mmap 区 / 文件映射）：用于共享库、文件映射（`mmap`）或匿名映射（`MAP_ANONYMOUS`）。
- 线程局部存储（TLS）：线程专有的静态数据（`thread_local`）。
- 共享库映射（动态链接库区域）：运行时会把库映射到进程地址空间。
- 内核空间（用户进程不可直接访问，但在内存图中存在）以及 CPU MMU 留出的 guard page（防止栈溢出）。

## 4、虚函数表、虚函数指针位于哪个区域？

- **vtable（虚函数表）**

  通常由编译器在可执行文件/库的只读数据段（`.rodata` / data segment）生成并放置在程序映像中（即静态存放于进程地址空间的只读或数据区）。

- **vptr（虚函数指针，隐藏成员）**

  存在**对象内**，因此它位于对象所在的内存区域：若对象在栈上则 vptr 在栈帧；若对象 `new` 出现在堆上则 vptr 在堆内；若对象为全局/静态则 vptr 在静态数据区。

> 小结：vtable 在 “静态数据区（只读）”，vptr 在对象内（栈/堆/静态），两者分开。

## 5. 堆和栈的区别？

| 方面     | 栈（Stack）                      | 堆（Heap）                                |
| -------- | -------------------------------- | ----------------------------------------- |
| 分配方式 | 编译器自动（函数调用、局部变量） | 程序员或库（`new`/`malloc`）              |
| 生命周期 | 由作用域控制（函数返回即销毁）   | 显式释放：`delete`/`free`，或直到程序结束 |
| 分配速度 | 非常快（指针移动）               | 相对慢（管理 metadata、查找空闲块）       |
| 大小限制 | 较小（受线程栈大小限制）         | 较大（系统和进程限制）                    |
| 访问     | 连续，局部性好                   | 可能碎片化，随机访问                      |
| 线程     | 每线程独立栈                     | 线程共享堆（需同步）                      |
| 用途     | 临时对象、函数帧                 | 动态数据结构、长寿命对象                  |
| 崩溃模式 | 栈溢出（Stack Overflow）         | 内存泄露、碎片、OOM                       |

> 注意：具体细节受 OS/ABI/实现影响（比如栈可向低地址或高地址增长，这由实现决定）。

## 6、C++11 的新特性你平常用哪些？

- `auto`：简化类型声明，避免冗长。
- 范围 `for` (`for (auto &x : container)`)：更简洁、安全。
- `nullptr`：替代 `NULL`，类型安全。
- `std::unique_ptr`, `std::shared_ptr`, `std::weak_ptr`：RAII 智能指针。
- 右值引用与移动语义（`T&&`, `std::move`, `std::forward`）：降低拷贝开销。
- `std::thread`, `<mutex>`, `<condition_variable>`：简化多线程编程（标准线程库）。
- `lambda` 表达式：回调、STL 算法、线程入口简洁化。
- `std::chrono`：精确计时与时间表示。
- `override`, `final`：继承安全与明确覆盖意图。
- `constexpr`（C++11 首版）与 `static_assert`：编译期常量与断言。
- `{}` 统一初始化与 `std::initializer_list`。
- `std::move`/`std::forward` 以及 `decltype` 用于泛型代码。

## 7、Lambda 表达式你有了解吗？

核心点：

- 语法（C++11 基本）：

  ```cpp
  [capture](params) -> ret { body }
  ```

  例如：`auto f = [x](int a){ return a + x; };`

- 捕获方式：

  - `[=]`：按值捕获所有外部变量。
  - `[&]`：按引用捕获。
  - `[this]`：捕获 `this` 指针（C++11）。
  - `[a, &b]`：混合捕获——显式指定每个变量的捕获方式。
  - C++14 起支持初始化捕获（generalized capture）：`[x = std::move(x)]`，C++14 的语法。

- `mutable`：默认按值捕获后 lambda 为 `const`，使用 `mutable` 允许在 body 中修改被捕获的副本。

- 返回类型推断：如果有多条 `return`，可用 `-> type` 指定返回类型，C++14 之后自动推断更强。

- 模板与泛型 lambda（C++14）：`auto` 参数（`[](auto x){}`）或 `template` lambda。

- 捕获 `this` 的风险：若 lambda 在异步场景中使用并延长生命周期，需要保证 `this` 在 lambda 执行期间有效，否则产生悬挂引用。常见做法是在捕获时用 `shared_from_this()`（`enable_shared_from_this`）或捕获 `weak_ptr` 并在执行时 lock。

## 8、介绍一下 `auto` 关键字

作用：类型推导（编译器根据初始化表达式推断变量类型）。常见用途：

- 简化复杂类型：`auto it = container.begin();`
- 结合 `decltype` 和模板：`auto` 在泛型代码里很方便。
- 与 `std::move`、返回类型配合使用。

注意点 / 陷阱

- `auto` 不会保留引用/`const` 除非显式写：`auto x = y;`（剥离引用与 top-level const）。要保留写 `auto &` 或 `const auto&`。
- `auto` 与 braced-init-list：`auto x = {1,2};` 会把 `x` 推断为 `std::initializer_list<int>`（可能不是你想要的）。
- `auto` 在模板返回类型/函数中需小心（返回时类型确定性）。

## 9、右值引用和左值引用有什么区别？

左值引用：`T&`，只能绑定到具名变量（或左值），用于修改可寻址对象。

右值引用：`T&&`（C++11 引入），能绑定到右值（临时对象、即将失去使用权的对象），是实现移动语义的基础。

用途与语义

- 右值引用用于“窃取资源”（move）：转移内部指针而不是拷贝内容，从而避免昂贵拷贝。
- 转发引用 / 完美转发：模板参数 `T&&` 在类型推导中表现为 转发引用，可绑定左值或右值，配合 `std::forward<T>(x)` 完美转发参数。

引用折叠规则（模板中）

- `T& &` -> `T&`
- `T& &&` -> `T&`
- `T&& &` -> `T&`
- `T&& &&` -> `T&&`

**示例**

```cpp
void f(int&);   // 接受左值
void f(int&&);  // 接受右值

int x=0;
f(x);           // 调用 f(int&)
f(1);           // 调用 f(int&&)
```

常用函数：`std::move`（把左值变成右值以触发移动构造），`std::forward`（完美转发）。

## 10、拷贝构造函数和移动构造函数？

拷贝构造函数

- 签名通常 `T(const T& other)`：按值语义拷贝对象（深拷贝或按类定义）。
- 在使用拷贝语义时调用：按值传参、返回值复制（拷贝），显式 `T b = a;`。

移动构造函数

- 签名通常 `T(T&& other) noexcept`（推荐 `noexcept`），转移资源所有权， leaving `other` in a valid but unspecified state。
- 在 `T(std::move(x))` 或临时对象初始化时调用，避免昂贵拷贝。
- **重要**：若移动构造可能抛异常，`std::vector` 在扩容时可能退回到拷贝，以保证异常安全；因此建议 `noexcept`。

规则：

若用户未定义拷贝构造、拷贝赋值、析构函数，编译器会生成默认的拷贝构造；若定义了移动构造，拷贝可能被删除或不再自动生成（详见 Rule of Five / Rule of Zero）。

示例

```cpp
struct S {
    std::string s;
    S(const S& o) : s(o.s) { }           // copy
    S(S&& o) noexcept : s(std::move(o.s)) { } // move
};
```

## 11、介绍一下智能指针？

C++11 标准库智能指针主要有三类（常用）：

`std::unique_ptr<T>`

- 表示对象独占所有权，不能拷贝，只能移动（移动语义）。
- 零开销封装，常用于工厂返回值（`make_unique<T>(...)`）。
- 支持自定义删除器：`std::unique_ptr<T, Deleter>`。

`std::shared_ptr<T>`

- 共享所有权，基于控制块维护强引用计数（strong_count）和弱引用计数（weak_count）。
- 当 `strong_count` 变为 0 时对象被析构；当 `weak_count` 也为 0 时控制块释放。
- `std::make_shared` 将控制块与对象分配在一次内存分配中（更高效）。

`std::weak_ptr<T>`

- 不拥有对象，不影响强引用计数。用于观察 `shared_ptr` 管理的对象并安全地获取临时 `shared_ptr`（`lock()`），常用于解决循环引用的问题。

其它

- `std::scoped_ptr`（Boost）或 `std::auto_ptr`（已弃用）历史存在，现代使用 `unique_ptr`。
- `enable_shared_from_this<T>`：类继承后可以从 `this` 内部创建 `shared_ptr`（要求该对象已被 `shared_ptr` 管理）。

## 12、`shared_ptr` 的实现机制？循环引用该怎么处理？

shared_ptr 的实现机制（简化）

- 核心：控制块，包含
  - 对象指针（或在 `make_shared` 场景中对象与控制块存在同一内存）
  - `strong_count`（强引用计数）
  - `weak_count`（弱引用计数）
  - deleter / allocator info 等
- `shared_ptr`（拷贝/赋值）会对 `strong_count` 做原子增加/减少（线程安全的引用计数）。
- 当 `strong_count` 变为 0，控制块调用析构对象，然后当 `weak_count` 也为 0 时释放控制块本身。

循环引用问题

- 如果 A 持有 `shared_ptr<B>`，B 又持有 `shared_ptr<A>`，两者的 `strong_count` 永远 > 0，导致内存泄漏（即便外部没有其他引用）。
- 解决办法：把其中一端改为 `weak_ptr`（常见做法：父拥有子用 `shared_ptr`，子指向父用 `weak_ptr`）。在访问时 `weak_ptr::lock()` 临时获取 `shared_ptr`，如果返回空表示对象已被销毁。

示例

```cpp
struct B;
struct A {
    std::shared_ptr<B> b;
};
struct B {
    std::weak_ptr<A> a; // break cycle
};
```

## 13、对象池有了解过吗？

目的：避免频繁 `new`/`delete` 的开销（时间 + 内存碎片），通过复用对象实例减少分配成本。

常见设计

- 预分配一批对象（数组或内存块），维护一个空闲链表（free list）或 bitmap。
- `acquire()`：从 free list 弹出一个对象（若空可扩容或阻塞/返回 null），用 placement new 构造或复用并 `reset()`。
- `release()`：调用析构或重置后把对象放回 free list。
- 线程安全：为多线程场景加锁或使用无锁算法（如 lock-free stack）。
- 注意点：
  - 必须保证对象在 release 后能被正确重置（避免残留状态）。
  - 管理好生命周期（避免返回给调用者后被其他线程重用导致悬挂引用）。
  - 与 `unique_ptr`/自定义删除器结合自动回收更安全（`unique_ptr<T, PoolDeleter>`）。

简化伪实现

```cpp
template<typename T>
class ObjectPool {
    std::vector<T*> pool_;
    std::mutex mtx_;
public:
    T* acquire() {
        std::lock_guard<std::mutex> lk(mtx_);
        if (pool_.empty()) return new T();
        T* p = pool_.back(); pool_.pop_back();
        return p;
    }
    void release(T* p) {
        std::lock_guard<std::mutex> lk(mtx_);
        pool_.push_back(p);
    }
    ~ObjectPool(){ for(auto p:pool_) delete p; }
};
```

## 14、介绍一下 `vector`？`vector` 插入元素的过程是怎么样的？数组需要扩容怎么办？

**`std::vector` 基本特性**

- 动态数组：随机访问 `O(1)`；尾部插入 `push_back` 均摊 `O(1)`；在中间插入 `O(n)`。
- 内部维护三个重要概念：`begin`（元素开始）、`size`（当前元素数）、`capacity`（已分配空间能容纳的元素数）。
- 内存通过 `allocator` 分配（默认是 `std::allocator<T>`，封装 `operator new`/`delete`）。

插入元素（以 `push_back` 为例）

1. 检查 `size < capacity`：
   - 若是，直接在 `begin + size` 处构造元素（placement new），`size++`。
   - 若否（capacity 不足），触发扩容流程：
     1. 计算新容量（实现依赖；常见策略：`new_capacity = old_capacity * 2` 或 `1.5` 倍，GCC 通常增长因子约 1.5~2）。
     2. 分配新内存（`allocate(new_capacity)`）。
     3. 将旧元素搬移到新内存：若类型提供 `noexcept` 移动构造则使用 `move`（更快）；否则使用拷贝构造。
     4. 销毁旧元素并 `deallocate` 旧内存。
     5. 在新内存上构造新元素。
2. `push_back` 的摊销复杂度为 `O(1)`，但扩容时是 `O(n)`。

浅拷贝还是深拷贝？

- `std::vector` 元素的移动/拷贝语义取决于元素类型的构造函数：
  - 内置/TriviallyCopyable 类型：往往可以 `memcpy`（非常快）。
  - 非平凡类型：调用 `T` 的移动构造或拷贝构造，这不是浅拷贝或“按位复制”除非类型确实是 POD（plain old data）。

如何避免频繁扩容

- 使用 `v.reserve(n)` 预分配容量，避免重复 reallocation。
- 对于已知大小的批量插入，使用 `reserve` 然后 `push_back` 或直接使用 `insert(begin, first, last)`。

插入到中间

- 插入中间位置时，元素后半部分需要整体向后移动一位（使用移动构造或拷贝），复杂度 `O(n)`。

## 15、C++ 的垃圾回收有没有了解？

C++ 标准没有内建垃圾回收（GC）机制；C++ 采用确定性析构（RAII）和智能指针管理资源。常见思路：

- RAII：对象生命周期结束时自动调用析构释放资源。
- 智能指针：`unique_ptr`（独占）、`shared_ptr`（引用计数）管理内存。
- 引用计数回收：`shared_ptr`（但不能自动解决循环引用）。
- 第三方 GC：可选集成 Boehm GC（conservative GC）或平台/语言特定 GC，但在高性能或实时场景中不常用。
- 内存回收策略：
  - 线程局部分配器、内存池（对象池）、arena 分配器用于减少内存分配 overhead 与碎片。
  - 在需要时用垃圾收集（例如某些脚本嵌入或托管堆的场景），但这会牵涉暂停时间与吞吐折中。

## 16、STL 是怎么做内存管理的？

**核心是 allocator（分配器）抽象**：

- `std::allocator<T>` 提供 `allocate` / `deallocate` / `construct` / `destroy`（自 C++17 起 `construct`/`destroy` 等通过 `allocator_traits` 实现）。默认由 `operator new`/`delete` 实现分配与释放。
- 容器模板（如 `vector`, `map`）接收一个 allocator 模板参数，用者可以传定制 allocator（例如 arena allocator、pool allocator、jemalloc wrapper）。
- 容器在内部做两步：
  1. allocate 一块原始内存（未构造对象）。
  2. 在该内存上通过 placement new 构造对象（`std::allocator_traits<A>::construct`）。
- 这种分离允许容器高效管理内存（例如 `vector` 先 `allocate` 大块，再按需 `construct`）。
- C++17 后：`std::pmr`（polymorphic memory resource）引入可插拔内存资源（更方便替换 allocator）。

## 17、反转链表（Reverse a singly linked list）

题目：给定单链表，原地反转并返回新头节点。

迭代解法（推荐）

```cpp
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int v=0): val(v), next(nullptr) {}
};

// 迭代反转
ListNode* reverseList(ListNode* head) {
    ListNode* prev = nullptr;
    ListNode* cur = head;
    while (cur) {
        ListNode* nxt = cur->next;
        cur->next = prev;
        prev = cur;
        cur = nxt;
    }
    return prev;
}
```

- 时间复杂度：`O(n)`，空间复杂度：`O(1)`（原地）。

递归解法

```cpp
ListNode* reverseListRec(ListNode* head) {
    if (!head || !head->next) return head;
    ListNode* newHead = reverseListRec(head->next);
    head->next->next = head;
    head->next = nullptr;
    return newHead;
}
```

- 优点：写起来简洁；缺点：递归深度 `O(n)` 可能导致栈溢出（长链表）。

## 18、最长数组子序列（通常指最长递增子序列 LIS）

问题：给定数组 `a[]`，找到最长严格递增子序列的长度（或子序列本身）。

O(n log n) 算法：

- 维护一个数组 `d[]`，其中 `d[len]` 表示当前长度为 `len` 的递增子序列的最小结尾值。
- 遍历 `a[i]`：使用二分查找在 `d` 中找到第一个 `>= a[i]` 的位置 `pos`，替换 `d[pos] = a[i]`；若 `a[i]` 大于所有 `d`，则在末尾扩展。
- 最终 `len(d)` 即为 LIS 长度。

代码

```cpp
#include <vector>
#include <algorithm>

std::vector<int> lis_sequence(const std::vector<int>& a) {
    int n = (int)a.size();
    std::vector<int> d;
    std::vector<int> idx;
    std::vector<int> prev(n, -1);

    for (int i = 0; i < n; ++i) {
        auto it = std::lower_bound(d.begin(), d.end(), a[i]);
        int pos = int(it - d.begin());
        if (it == d.end()) {
            d.push_back(a[i]);
            idx.push_back(i);
        } else {
            *it = a[i];
            idx[pos] = i;
        }
        if (pos > 0) prev[i] = idx[pos - 1];
    }

    std::vector<int> seq;
    if (!idx.empty()) {
        int cur = idx.back();
        while (cur != -1) {
            seq.push_back(a[cur]);
            cur = prev[cur];
        }
        std::reverse(seq.begin(), seq.end());
    }
    return seq;
}
```

