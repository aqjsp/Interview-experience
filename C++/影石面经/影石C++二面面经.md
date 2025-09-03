# 影石C++二面面经

大家好，我是Q。

![来源](https://cdn.jsdelivr.net/gh/aqjsp/photos/image-20250903170442558.png)

## 1、C++ 的覆盖（override）和重载（overload）有什么区别？

**概念差异**

重载（overload）：同一作用域内函数名相同，但参数列表（参数类型或数量或顺序）不同，是**编译期多态**（函数签名不同，编译器根据参数选择重载的函数）。
 例：

```cpp
void f(int);
void f(double);
void f(int, int);
```

覆盖（override）：派生类对基类的**虚函数**提供新的实现（签名相同或协变返回类型），是**运行时多态**（通过虚表动态分派）。覆盖发生在基类和派生类之间。C++11 引入 `override` 关键字用于显式标注覆盖意图并让编译器检查。

**关键点**

- 重载是同一类（或同一命名空间）内不同签名的多个函数；覆盖是派生类重写基类的虚方法（签名兼容）。
- 如果派生类定义了与基类同名但**不同签名**的函数，会**隐藏（name hiding）**基类的所有同名重载，除非用 `using Base::foo;` 显式引入。
- 覆盖要求（通常）签名一致；但允许**协变返回类型**（基类方法返回 `Base*`，派生类可以返回 `Derived*`）。

**示例（覆盖 vs 重载）**：

```cpp
struct Base {
    virtual void g(int);
    void f(int);
    void f(double);
};

struct Derived : Base {
    // 这是覆盖（override）
    void g(int) override;

    // 这是定义了一个新函数，参数不同 -> 重载（但在派生类会隐藏基类的 f）
    void f(std::string);

    // 如果想保留 Base::f 的两个重载：
    using Base::f; // 把基类的重载带进来
};
```

## 2、父类和子类有同名同参的函数，加 `virtual` 和不加有什么区别？

**场景**：基类 `Base` 有一个同名同参数的函数 `foo()`，派生类 `Derived` 也实现了同签名的 `foo()`。

- **基类函数带 `virtual`（或是虚函数）**：调用发生**动态绑定**（运行时多态）。当通过基类指针或引用指向派生对象时，调用会走派生类的实现。

  ```cpp
  Base* p = new Derived;
  p->foo(); // 调用 Derived::foo()
  ```

  实现机制通常是每个多态类型有一个 vtable（虚表），对象里有 vptr 指向 vtable。

- **基类函数不带 `virtual`**：调用发生**静态绑定**（编译时决定）。通过基类指针/引用调用，**总是**调用基类实现；通过派生对象直接调用则调用派生实现（因为派生对象视角下派生函数存在）。

  ```cpp
  Base* p = new Derived;
  p->foo(); // 调用 Base::foo() —— 静态绑定
  Derived d;
  d.foo();  // 调用 Derived::foo()
  ```

如果你期望通过基类指针删除派生对象，基类**必须有虚析构函数**，否则会发生未定义行为（派生析构不会被调用，资源泄漏/破坏）。

## 3、在派生类里，怎么调用父类那个同名同参的方法？

在派生类中显式调用基类的实现，使用**作用域操作符** `Base::method(...)`。常见用途是派生实现中先调用基类再扩展逻辑。

```cpp
struct Base {
    virtual void process() { /* 基类实现 */ }
};

struct Derived : Base {
    void process() override {
        Base::process(); // 调用父类实现
        // 然后添加派生逻辑
    }
};
```

如果基类函数被隐藏（同名但不同签名），也可以用 `Base::f(...)` 指定要调用的版本。

## 4、派生类里怎么定义一个父类的指针？

在派生类内部（或其它地方）把 `this` 当作基类指针，隐式转换可用：

```cpp
struct Base { };
struct Derived : Base {
    void foo() {
        Base* pb = this;            // 隐式转换，Derived* -> Base*
        Base& rb = *this;           // 引用也可以
        // 或者显示转换：
        Base* pb2 = static_cast<Base*>(this);
    }
};
```

注意：这种转换是安全的（指向同一个对象内的基类子对象）。但用基类指针访问派生特有成员需要显式向下转换（`static_cast`，`dynamic_cast`）且要小心类型和多重继承情况。

## 5、`static` 关键字修饰：局部变量、全局变量、成员函数、成员变量分别有什么用？

### 5.1、局部变量（函数内 `static`）

含义：变量的生命周期延长为程序全局（静态存储期），但作用域仍然局限在函数内（局部可见）。值在调用间保存，仅初始化一次（C++11 起局部 `static` 的初始化是线程安全的）。

用途：保存函数状态、实现单例（简单版）等。

```cpp
int counter() {
    static int cnt = 0; // 只初始化一次
    return ++cnt;
}
```

### 5.2、全局变量（或函数）前的 `static`（在命名空间/文件作用域）

含义（旧风格）：将符号的链接性设为“内部链接”（internal linkage），只能被当前翻译单元（.cpp）看到。等价于 C 风格的 `static` 文件作用域。

C++ 推荐：使用匿名 namespace 替代 `static`（更现代且语义清晰）。

```cpp
// old: static int s_val = 0;
namespace { int s_val = 0; } // 建议用这种
```

### 5.3、类的静态成员变量（`static` 成员变量）

含义：类级别变量，所有实例共享一份；需要在类外定义（除非是 `inline` static 或常量整型有特例）。

```cpp
struct S {
    static int count;
};
int S::count = 0; // 在 .cpp 中定义并初始化
```

### 5.4、静态成员函数（`static` 成员函数）

含义：不依赖 `this`，只能访问静态成员；可以通过 `S::f()` 或对象调用。常用于工具函数或工厂方法。

```cpp
struct S {
    static void help() { /* 无 this */ }
};
S::help();
```

## 6、`volatile` 有什么用？

**目的**：告诉编译器该变量的值随时可能被“外部因素”改变，禁止某些对该变量的优化（比如寄存器缓存、消除重复读取等）。

常见场景：

- 内存映射 I/O（设备寄存器），
- 由中断/信号处理器修改的全局变量（C 风格），
- 和操作系统或硬件交互时。

重要说明（现代 C++）：

- `volatile` **不等价于** 线程同步，也不保证原子性、内存顺序／屏障。**不能**用于实现多线程的同步。应使用 `std::atomic`、互斥量、内存序等标准并发工具。
- 在多线程程序里，`volatile` 几乎没用（除非是特殊平台的内存映射场景）。

示例（设备寄存器）：

```cpp
volatile uint32_t* UART_DR = reinterpret_cast<uint32_t*>(0x4000'0000);
uint32_t read = *UART_DR; // 每次都直接从内存/寄存器读取
```

## 7、`inline` 有什么用？

`inline` 在 C++ 有两个常见含义：

1. **编译器内联建议**：提示编译器“可以把函数体内联展开，消除调用开销”。现代编译器会根据优化策略自行决定是否内联，`inline` 只是一个建议（不过编译器也会把它当成“允许多定义”的标记）。
2. **链接语义（重要）**：`inline` 允许在多个翻译单元中定义同一个函数（或变量，自 C++17 起支持 `inline` 变量），而不违反 ODR（One Definition Rule）。这使得在头文件中定义函数成为安全做法（例如内联的模板辅助函数或小函数直接写在头里）。

补充：

- 在类内定义的成员函数默认就是 `inline`。
- C++17 引入 `inline` 变量，允许在头文件中定义静态常量等。

示例：

```cpp
// header.h
inline int add(int a, int b) { return a + b; } // 可以放在头文件
```

## 8、`inline` 是怎么提高函数运行效率的？

**如何带来性能提升**：

- 消除函数调用开销：内联化把函数体直接插入调用点，省去函数调用/返回的栈操作和跳转。
- 更多优化机会：内联后，编译器能跨越函数边界做常量折叠、死代码消除、寄存器分配优化、循环展开等（例如内联后发现常量参数可以被折叠）。
- 减少抽象开销：对于小的辅助函数（访问器、轻量计算），内联能显著减少开销。

**权衡与局限性**：

- 代码膨胀（binary size 增长）：过度内联会增加代码体积，影响 I-cache，从而反而降低性能。
- 编译器自由度：`inline` 只是提示；现代优化器根据函数大小、调用频率和编译器策略决定最终是否内联。
- 不影响语义：即使函数没有被内联，`inline` 对程序行为无影响（除了链接规则）。

## 9、`new/delete` 和 `malloc/free` 的区别？

| 项目          | `new/delete`                                                 | `malloc/free`                                  |
| ------------- | ------------------------------------------------------------ | ---------------------------------------------- |
| 调用构造/析构 | `new` 调用构造，`delete` 调用析构                            | 不调用构造或析构                               |
| 返回类型      | 返回类型化指针（例如 `T*`）                                  | 返回 `void*`，需要强转                         |
| 失败行为      | 抛出 `std::bad_alloc`（可用 `std::nothrow` 改为返回 `nullptr`） | 返回 `NULL`                                    |
| 对象数量      | `new T` / `new T[n]`（对应 `delete` / `delete[]`）           | `malloc(size)` / `free(ptr)`                   |
| 内存对齐      | 满足对任何对象类型的对齐要求（实现保证）                     | C 标准保证返回能满足任何对象的对齐（现代实现） |
| 可重载        | 可以重载 `operator new` / `operator delete`                  | 一般不可重载（实现相关）                       |
| 使用场景      | C++ 对象动态分配（需要构造/析构）                            | C 风格的原始内存分配（或对性能有特殊需求时）   |

**常见错误**：

用 `malloc` 分配对象但用 `delete` 释放，或 `new` 分配但用 `free` 释放 —— **未定义行为**。必须配套使用。

补充：还有 **placement new**（定位 new）`new (buf) T(args...)` 用于在已有内存上构造对象；对应析构需手动调用显式析构 `obj->~T()`。

## 10、C++ 的强制类型转换和 C 的有什么区别？

C++ 有 4 种显式转换运算符（比 C 风格强、语义更清晰）：

- `static_cast<T>(expr)`：编译时类型转换，安全于相关类型间的非指针重新解释（例如数值类型转换、上行转换 `Derived*`->`Base*`、去掉 `const` 不能用）。
- `dynamic_cast<T>(expr)`：用于多态类型（含虚函数）的安全向下转换，运行时检查，失败返回 `nullptr`（指针）或抛出 `std::bad_cast`（引用）。
- `const_cast<T>(expr)`：去除或添加 `const`/`volatile` 的转换（唯一用于修改 cv-qualifier 的 C++ 转换）。
- `reinterpret_cast<T>(expr)`：位级重解释转换（危险），将指针/整型等在底层重新解释，一般用于底层接口或与硬件交互。

C 风格的 cast (`(T)expr`)：

- 功能等同于尝试上述转换的**组合**（`const_cast`/`static_cast`/`reinterpret_cast` 的某种组合），因此不安全且可读性差。C++ 建议优先使用四种 C++ 明确 cast，使意图清楚、便于编译器/审查器捕捉错误。

示例：

```cpp
Base* b = /* ... */;
Derived* d1 = static_cast<Derived*>(b);   // 编译时转换，危险（无检查）
Derived* d2 = dynamic_cast<Derived*>(b);  // 运行时检查，若不属于 Derived 则返回 nullptr
const int ci = 10;
int& r = const_cast<int&>(ci); // 有 UB（除非原对象非 const）
```

## 11、介绍下 4 种智能指针？

现代 C++（C++11 以后）常见智能指针主要是 **`std::unique_ptr`**, **`std::shared_ptr`**, **`std::weak_ptr`**。

面试/历史上有时还会提到 `std::auto_ptr`（已弃用），所以这里把这四种都讲清楚并说明差别与注意点。

### 11.1、`std::unique_ptr<T>`

- 语义：独占所有权（不可复制，只能移动）。适合表示唯一所有者。

- 特点：轻量、没有引用计数、支持自定义删除器 `unique_ptr<T, Deleter>`。

- 用法：

  ```cpp
  auto p = std::make_unique<MyClass>(args...); // 推荐
  std::unique_ptr<MyClass> p2 = std::move(p);  // 转移所有权
  ```

- 典型场景：表示独占资源、工厂函数返回值。

### 11.2、`std::shared_ptr<T>`

- 语义：共享所有权，采用引用计数（control block）。最后一个 `shared_ptr` 销毁时释放资源或调用自定义删除器。

- 特点：可复制，支持 `make_shared`（更高效，创建 control block 与对象一次分配），引用计数的修改是线程安全的（对计数的修改是原子），但对所管理对象的并发访问仍需同步。

- 用法：

  ```cpp
  auto p1 = std::make_shared<MyClass>(...);
  auto p2 = p1; // 引用计数 +1
  ```

- 缺点：循环引用会导致内存泄漏（`A` 持有 `shared_ptr<B>`，`B` 持有 `shared_ptr<A>`）。

### 11.3、`std::weak_ptr<T>`

- 语义：观察者指针，不参与引用计数（不拥有）。可以从 `weak_ptr` 安全地获取 `shared_ptr`（`lock()`，如果对象已销毁则返回 `nullptr`）。

- 用途：打破 `shared_ptr` 循环引用、缓存/临时检查等。

  ```cpp
  std::weak_ptr<MyClass> wp = p_shared;
  if (auto sp = wp.lock()) { /* safe to use sp */ }
  ```

### 11.4、`std::auto_ptr<T>`（历史，已弃用/移除）

- 语义：C++98 的智能指针，语义上像 `unique_ptr`（转移所有权），但它**复制语义是移动式的**（复制会转移所有权），导致代码直观性差，故被弃用并在 C++11 被移除。
- 建议：不要使用，改用 `unique_ptr` / `shared_ptr`。

补充：Boost 里还有 `intrusive_ptr`（计数存在对象里）等。实际工程中优先用 `unique_ptr`（表达唯一所有权）和 `shared_ptr`（必要时共享），`weak_ptr` 用于解决循环或观察者场景。

## 12、在一个类内部，怎么返回一个指向自己的智能指针？

常用场景是：类方法希望返回一个 `shared_ptr<T>` 指向当前对象（`this`）。

正确方法是继承 `std::enable_shared_from_this<T>`，并用 `shared_from_this()` 返回。

前提：该对象**必须已经由 `shared_ptr` 管理**（例如通过 `std::make_shared` 创建）；否则 `shared_from_this()` 是未定义行为（通常抛出异常或崩溃）。

示例：

```cpp
#include <memory>
#include <iostream>

struct My : std::enable_shared_from_this<My> {
    std::shared_ptr<My> getPtr() {
        return shared_from_this(); // 返回 shared_ptr，引用计数 +1
    }
    void hello() { std::cout << "hello\n"; }
};

int main() {
    auto p = std::make_shared<My>();
    auto p2 = p->getPtr(); // OK，p 和 p2 共享控制块
}
```

关于 `unique_ptr`：

没有直接对应的 `unique_shared_from_this`。如果对象最初是由 `unique_ptr` 管理，**类方法内部不能安全地返回 `unique_ptr<this>`**，因为那会导致双重释放或所有权不明确。若确实要把 `unique_ptr` 转为 `shared_ptr`，在创建时就把它包装到 `shared_ptr`：

```cpp
std::unique_ptr<T> u = std::make_unique<T>();
std::shared_ptr<T> s = std::move(u); // 这在语义层面不可行：不能直接从 unique 转为 shared 除非用 shared_ptr<T>(u.release())
```

更安全的做法是：**通过工厂函数**返回 `unique_ptr` 或 `shared_ptr`，并以契约明确所有权归属。

安全总结：

- 想从对象内部获取 `shared_ptr`：用 `enable_shared_from_this` 并确保对象被 `shared_ptr` 管理。
- 想返回 `unique_ptr`：最好由外部负责创建/返回（工厂函数），不要在对象内部 new 一个指向 `this` 的 `unique_ptr`。

## 13、结构体的内存对齐规则是什么？

基本原则：

1. 每个成员的对齐要求（alignment）由该成员类型决定（通常是类型大小或实现定义的对齐）。
2. 成员在结构体中的偏移必须是该成员对齐要求的整数倍（因此编译器会在成员之间插入填充 `padding`）。
3. 结构体整体的对齐要求是其成员对齐要求的最大值。
4. 结构体的 `sizeof`必须是结构体对齐要求的整数倍，编译器会在结构体尾部加填充使其满足对齐。

示例说明：

```cpp
struct A {
    char c;    // offset 0
    // padding 1..3
    int  i;    // offset 4 (假设 int 对齐为 4)
    short s;   // offset 8 (短整型对齐 2，但上一个成员结束在 8)
};
// sizeof(A) 可能是 12（若最大对齐 4，则整体大小为 12）
```

减小内存浪费的方法：按**从大到小**的类型顺序组织成员，减少填充。例如把 `int` 放在前面，`char` 放在后面。

示例对比：

```cpp
struct Bad {
    char c;
    int i;
    char d;
}; // 可能 sizeof = 12 (padding between c-i and after d)

struct Good {
    int i;
    char c;
    char d;
}; // 可能 sizeof = 8
```

强制打包/改变对齐：

- 可以用 `#pragma pack(push,1)` 或编译器特定指令强制 1 字节对齐，减少大小但可能影响访问性能并在某些平台产生未对齐访问的硬件异常。
- C++11 提供 `alignas` 明确指定对齐。
- 使用强制打包要谨慎，仅在二进制协议或磁盘/网络布局匹配时使用。

**offsetof** 宏可以获取成员偏移量（标准 `<cstddef>`）。

## 14、介绍下 DHCP 协议？

**DHCP（Dynamic Host Configuration Protocol）**是用于在IP网络中动态分配网络参数（尤其是 IP 地址、子网掩码、默认网关、DNS 等）的协议。它是 BOOTP 的继承与扩展。

### 核心思想

- 客户端在加入网络时，从 DHCP 服务器请求网络配置，服务器按策略分配地址（临时租约）。
- 自动化配置减少手工管理静态 IP 的开销，便于移动主机和大量终端管理。

### 主要消息类型（UDP，端口：服务器 67，客户端 68）

- DHCPDISCOVER：客户端广播寻找 DHCP 服务器。
- DHCPOFFER：服务器回应并提供一个 IP 地址（offer）。
- DHCPREQUEST：客户端选择一个 offer 并向所有服务器表明接受；这个消息用于请求/续租。
- DHCPACK：服务器确认并完成分配（ACK）。
- DHCPNAK：服务器拒绝请求。
- DHCPDECLINE：客户端告知服务器提供的地址不可用（例如 ARP 冲突）。
- DHCPRELEASE：客户端释放地址（告知服务器该地址可回收）。
- DHCPINFORM：客户端已有静态 IP，但需要其它参数（例如 DNS）。

这个经典流程常称为 **DORA**（Discover → Offer → Request → Acknowledge）。

### 租约与续租（lease）

- 分配的是**租约**，有过期时间。常见续租策略：
  - T1（renew）：一般为租期的一半（默认 50%），客户端直接向原服务器发送 unicast 请求续租。
  - T2（rebind）：一般为租期的 87.5%（0.875 倍），如果在 T1 时没有成功续租，则广播到所有服务器以重新绑定。
  - 如果过期且未续租，客户端应重新走 DORA 流程获取地址。

### DHCP 中继（Relay）

当客户端和 DHCP 服务器不在同一子网时，路由器/交换机上的 DHCP relay agent（如 `ip helper-address`）会转发 DHCP 广播到服务器（封装客户端信息并添加 relay-agent 信息）。

### DHCP Options（可选字段）

提供子网掩码、默认网关、DNS 服务器、租期（lease time）、域名、NTP、引导文件名、供应商特定选项等信息。协议用选项字段传送这些参数。

### 安全与管理

- 安全问题：DHCP 易被伪造（伪 DHCP 服务器）或进行地址耗尽攻击。常用防护手段：DHCP Snooping（交换机功能，允许仅可信端口回应）、端口安全、认证机制（在企业环境）。
- 地址分配策略：
  - 动态分配：从地址池临时分配，租期到期回收。
  - 静态分配（Reservation）：基于 MAC 做保留，使特定主机总是获得同一 IP（常在 DHCP 服务器上配置）。
  - 自动分配：首次分配后一直保留（某些实现）。

### DHCP 与 IPv6

对应为 **DHCPv6**，与 IPv4 的 DHCP 在实现/消息格式上有差异，并且 IPv6 网络中也存在基于 Router Advertisement（RA）的无状态地址自动配置（SLAAC），两者可以结合使用。