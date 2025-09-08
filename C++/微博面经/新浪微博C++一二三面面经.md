# 新浪微博C++一二三面面经

## 一面

### 1、写出字符数组大小

```c++
char buf[10] = "123";
char buf1[] = "123";
char *p = "123";

sizeof(buf) =
sizeof(buf1) =
strlen(p) =
sizeof(p) = 
```

直接编译运行一下：

![运行结果](https://cdn.jsdelivr.net/gh/aqjsp/photos/image-20250907181843566.png)

### 2、请输出下面的内容

```c++
int a = 10;
int &b = a;
b = 11;

printf("%d \n", a);
```

运行结果：

![运行结果](https://cdn.jsdelivr.net/gh/aqjsp/photos/image-20250907182414420.png)

### 3、下面程序输出什么？

```c++
class AAA {
public:
    void test1() {
        printf("This is AAA:test1\n");
        test2();
    }

    void test2() { 
        printf("This is AAA:test2\n"); 
    }
};

class BBB : public AAA {
public:
    void test2() { 
        printf("This is BBB:test2\n"); 
    }
};

int main() {
    BBB *p = new BBB;
    p->test1();
    return 0;
}
```

运行结果：

![运行结果](https://cdn.jsdelivr.net/gh/aqjsp/photos/image-20250907182748758.png)

### 4、函数和变量前面用static修饰的作用？

#### 4.1、修饰全局变量或函数（文件作用域）

 作用：限制符号的链接属性为“内部链接”（internal linkage）

> 即：**只在当前源文件（.cpp）内可见，其他文件无法访问**

示例：

```c++
// file1.cpp
static int globalVar = 10;       // 只在 file1.cpp 内可见
static void helper() {           // 只在 file1.cpp 内可调用
    cout << "Helper called\n";
}

void publicFunc() {
    helper(); //  OK
}
```

```c++
// file2.cpp
extern int globalVar;  // ❌ 链接错误！找不到符号
extern void helper();  // ❌ 链接错误！

void foo() {
    helper(); // ❌ 编译/链接失败
}
```

用途：

- 封装“仅本文件使用”的工具函数或变量
- 避免命名冲突
- 模拟“私有”全局函数/变量（C++ 中没有“private 全局”的语法）

> 类似于 Java 的 `private static`，但作用在文件级别。

#### 4.2、修饰局部变量（函数内部）

作用：延长生命周期到整个程序运行期，但作用域仍限于函数内

- 只初始化一次（首次进入函数时）
- 函数退出后，变量不销毁，值保留
- 下次再进入函数，变量保持上次的值

示例：

```c++
void counter() {
    static int count = 0;  // 只初始化一次
    count++;
    cout << "Called " << count << " times.\n";
}

int main() {
    counter(); // Called 1 times.
    counter(); // Called 2 times.
    counter(); // Called 3 times.
}
```

用途：

- 实现“函数内状态记忆”
- 单例模式中的局部静态变量（线程安全！C++11 起）
- 避免使用全局变量

> 注意：不是“线程局部”，而是“所有线程共享”。C++11 起初始化是线程安全的。

#### 4.3、修饰类的成员变量

作用：属于类本身，不属于任何对象 —— 所有对象共享一份

- 必须在类外定义（除非是 `const static` 整型可在类内初始化）
- 不占用对象的 `sizeof` 空间
- 可通过类名或对象访问

示例：

```c++
class MyClass {
public:
    static int count;  // 声明
    void increment() { count++; }
};

int MyClass::count = 0;  // 必须在类外定义！

int main() {
    MyClass obj1, obj2;
    obj1.increment();
    obj2.increment();
    cout << MyClass::count; // 输出 2
}
```

用途：

- 记录对象数量
- 共享配置、缓存、资源池等
- 实现单例模式（配合私有构造函数）

#### 4.4、修饰类的成员函数

作用：

1. 不依赖对象实例 —— 没有 `this` 指针
2. 只能访问静态成员变量和其他静态成员函数
3. 可通过类名直接调用

示例：

```c++
class MathUtils {
public:
    static int add(int a, int b) {
        return a + b;
    }

    static void printCount() {
        cout << "Count: " << count << endl; // 可访问静态成员
        // cout << x; // 不能访问非静态成员
    }

private:
    static int count;
    int x; // 非静态成员
};

int MathUtils::count = 0;

int main() {
    int result = MathUtils::add(3, 4); // 不需要对象
    MathUtils::printCount();
}
```

### 5、delete和free有什么区别？基类的析构函数不是虚函数会带来什么影响？

#### delete与free

`delete` / `delete[]`（C++）：

1. 会先调用对象的析构函数（`~T()`），用于清理对象所管理的资源（释放内存外的资源：文件、socket、内存等）。
2. 然后调用相应的内存释放函数（`operator delete` / `operator delete[]`），把内存返还给运行时/实现（可能调用 `free`，也可能调用自定义实现）。
3. 适用于 由 `new` / `new[]` 分配 的内存/对象。

`free`（C）：

1. 只是把内存块返还给 C 运行时/操作系统（底层 allocator），不会调用任何析构函数。
2. 适用于 由 `malloc` / `calloc` / `realloc` 分配 的内存。

> 结论：不要混用。`new`/`delete` 成对使用，`new[]`/`delete[]` 成对使用，`malloc`/`free` 成对使用。混用（如 `malloc` + `delete` 或 `new` + `free`）会导致未定义行为（UB）。

##### 细节与常见误区

###### 1. 析构函数调用

- `delete p;` 会执行 `p->~T()`（调用析构），这是 `delete` 区别于 `free` 的根本点。
- `free(p);` 不会调用析构函数，因此如果 `T` 管理资源（如 `std::string`、文件句柄、内存指针等），直接 `free` 会导致资源泄露或未定义行为。

###### 2. 内存分配器与实现

- 虽然在很多实现上 `operator new` 最终会调用 `malloc`，而 `operator delete` 最终会调用 `free`，但这不是语言保证（`operator new/delete` 可被重载/替换）。因此用 `free` 释放 `new` 分配的内存是 UB —— 即使在某实现上临时“看起来可行”，也不可依赖。
- 同样 `malloc` 分配内存再用 `delete` 释放是 UB（`delete` 会调用析构并调用 `operator delete`，该函数可能期待 `operator new` 分配的布局）。

###### 3. `new[]` 与 `delete[]`

- `new[]` 分配数组，会为每个元素调用构造函数（且通常实现会在分配块里记录元素个数）；`delete[]` 会调用每个元素的析构函数，然后释放内存。
- 使用 `delete` 去释放 `new[]` 分配的内存也是 UB（因为 `delete` 不会按数组元素数量调用析构，也可能调用错误的 `operator delete`）。

###### 4. `nullptr` 的行为

- `delete nullptr;` 是安全的 —— 什么也不做。
- `free(nullptr);` 也是安全的 —— 什么也不做。

###### 5. 自定义 `operator new` / `operator delete`

C++ 允许类或全局范围重载 `operator new`/`operator delete`。`delete` 会调用相对应的 `operator delete` 来回收内存。如果你混用 `free`，则绕过了类/全局定制的内存逻辑，可能引发错误。

###### 6. Placement new（定位 new）

`new (buf) T(args...)`：在已给定内存上构造对象；此时不应对该指针使用 `delete` 或 `free` 直接释放。正确做法是显式调用析构 `p->~T()`，然后依申请者的方式释放底层内存（若底层用 malloc 分配则 free，若栈则不释放等）。

##### 典型示例代码

###### 正确：`new` / `delete`

```c++
struct Foo {
    Foo() { puts("ctor"); }
    ~Foo() { puts("dtor"); }
};

Foo* p = new Foo();
delete p; // 先调用 ~Foo，再释放内存
```

###### 错误：`new` + `free`（UB）

```c++
Foo* p = new Foo();
free(p); // UB：不会调用 ~Foo，且释放方式与 operator delete 可能不匹配
```

###### 错误：`malloc` + `delete`（UB）

```c++
Foo* p = (Foo*)malloc(sizeof(Foo));         // 未构造对象
new (p) Foo();                              // placement new，构造对象
delete p; // UB：delete 假设 p 来自 new，而这里是 malloc/placement new
// 正确做法：p->~Foo(); free(p);
```

###### 数组分配注意：`new[]` / `delete[]`

```c++
Foo* a = new Foo[3];    // 调用 3 次构造
delete [] a;            // 调用 3 次析构并释放
// 不能写 delete a; 否则 UB
```

#### 基类析构函数不是 `virtual` 会带来什么影响？

##### 背景与结论

- 如果一个类被多态地使用（即通过基类指针/引用指向派生类对象），且你可能通过基类指针删除对象（`delete base_ptr`），那么基类必须声明虚析构函数（`virtual ~Base()`）。否则，`delete` 这个基类指针将导致 未定义行为（UB）。
- 典型场景：`Base* p = new Derived; delete p;` —— 若 `Base::~Base()` 非 `virtual`，则这是 UB：派生类析构函数可能不会被调用（或更糟，发生内存破坏）。

> 强烈建议：**所有打算作为多态基类（有虚函数或者将通过基类指针/引用操作派生对象）的类，都应声明虚析构函数**。

##### 为什么需要虚析构？

1. 动态类型与析构顺序

   - C++ 要保证“从最派生类到基类”的析构顺序（先执行 `Derived::~Derived()`，再 `Base::~Base()`），以保证派生对象拥有的资源可以在派生析构中正确释放。
   - 这个顺序需要动态（运行时）确定对象的真实类型；实现上通过虚表（vtable）机制来支持虚函数的动态绑定。`delete` 表达式在析构时会查找 vtable 并调用最派生类的析构，从而触发完整的析构链。

2. 非虚析构导致的情况

   如果 `Base` 的析构非虚，`delete base_ptr;` 会按静态类型去调用 `Base::~Base()`（或者更确切地说，标准把在这种场景定义为 UB）。结果：

   - `Derived` 中的析构函数不会被（可靠地）调用 → 派生类持有的资源得不到释放（内存泄漏、文件句柄泄漏等）。
   - 复杂继承（尤其多继承）还可能引发指针调整/内存释放的错误，从而造成更严重的内存破坏。

3. C++ 标准

   标准明确指出：通过基类指针删除派生类对象而基类析构不是虚函数，会产生未定义行为。也就是说，编译器/运行时行为不可预测。

##### 示例

###### 正确写法（基类析构虚）

```c++
#include <iostream>

struct Base {
    virtual ~Base() { std::cout << "Base dtor\n"; }
};

struct Derived : Base {
    ~Derived() { std::cout << "Derived dtor\n"; }
};

int main() {
    Base* p = new Derived();
    delete p; // 输出：Derived dtor\n Base dtor\n （先派生后基类）
}
```

###### 错误写法（基类析构非虚，UB）

```c++
#include <iostream>

struct Base {
    ~Base() { std::cout << "Base dtor\n"; }
};

struct Derived : Base {
    ~Derived() { std::cout << "Derived dtor\n"; }
};

int main() {
    Base* p = new Derived();
    delete p; // UB：可能只调用 Base::~Base()、可能崩溃或别的行为
}
```

实际运行可能只打印 `Base dtor`，此时 `Derived` 的析构没有执行，派生类资源不会被释放 —— 但你不能依赖该“表现”，因为标准定义为 UB。

##### 什么时候需要虚析构？什么时候可以不需要？

- 需要虚析构：基类用于多态（有至少一个虚函数，或会通过基类指针/引用指向派生对象并删除/管理时）。
- 不需要虚析构：基类仅作为聚合数据结构或纯值类型使用，不会被用作多态基类（即不会通过基类指针删除派生对象）。例如 `struct Point { ... };`。

> 经验法则：如果类包含其他虚函数（即已经是多态类型），**同时**你预计有可能用 `Base*` 指向 `Derived`，就把析构声明为 `virtual`。这是一条简单且常用的原则。

##### 更多注意点与变种

###### 1. `virtual` 纯虚析构（pure virtual destructor）

- 可以把析构设为纯虚：`virtual ~Base() = 0;`

- ###### 但必须提供定义（即使是纯虚），因为派生类析构会间接调用基类析构的实现：

  ```c++
  struct Base {
      virtual ~Base() = 0;
  };
  Base::~Base() { /* 可以留空或清理公共资源 */ }
  ```

- 用途：把基类声明为抽象类（不能直接实例化），仍然保证析构链完整。

###### 2. 访问控制：把析构设为 `protected` / `private`

- 有时把基类析构设为 `protected` 来禁止外部通过基类指针删除对象，迫使使用者通过工厂/智能指针等方式销毁对象（常用于实现受控销毁）。
  - 例如：`class NonDeleteable { protected: ~NonDeleteable() {} };`
- 结合 `friend` 或智能指针可以控制谁可以删除。

###### 3. 智能指针与虚析构

- `std::shared_ptr<Base> sp(new Derived);` 的默认 deleter 会在 `sp` 的最后一个副本销毁时调用 `delete`（以 `Base*`）。因此 **`Base` 必须有虚析构**，否则仍然是 UB（与直接 `delete` 一样）。
- 可选方案：用自定义 deleter：`std::shared_ptr<Base> sp(static_cast<Base*>(new Derived), [](Base* p){ delete static_cast<Derived*>(p); });` —— 但这不是常规做法，最佳做法仍然是使 `Base` 的析构虚化。

###### 4. 多重继承 / 指针调整问题

在多重继承场景下，基类子对象的地址可能与最派生对象地址不同；虚析构确保在析构时使用**正确的最派生对象地址**来调用析构并正确释放（对齐 operator delete 的指针）。非虚析构可能导致错误的 `operator delete` 参数或错误的销毁顺序。

### 6、int const* 和int* const的区别？

#### 6.1、`int const*` —— 指向常量的指针

写法等价于：

```c++
const int* p;
```

##### 含义

- `p` 是一个 指针，它 指向一个常量的 int。
- 你 不能修改 p 所指向的值（内容只读）。
- 但你 可以修改 p 本身的指向（让它指向别的 int）。

##### 示例

```c++
int a = 10;
int b = 20;

const int* p = &a;  // p指向a
//*p = 30;          // 错误：不能通过p修改a的值
p = &b;             // 正确：可以修改p指向
```

这里 `*p` 是常量，`p` 本身是变量。
 即：

- `*p` **不可改**
- `p` **可改**

#### 6.2、`int* const` —— 常量指针

##### 含义

- `p` 是一个 常量指针，它 始终指向同一个 int。
- 你 可以修改所指向的值。
- 但你 不能修改指针本身的指向。

##### 示例

```c++
int a = 10;
int b = 20;

int* const p = &a;  // p必须立刻初始化
*p = 30;            // 正确：可以修改p指向的值
//p = &b;           // 错误：p是常量指针，不能改变指向
```

这里 `*p` 是变量，`p` 本身是常量。
 即：

- `*p` 可改
- `p` 不可改

#### 6.3、对比总结

| 写法               | 含义                   | 能否改值 | 能否改指针指向 |
| ------------------ | ---------------------- | -------- | -------------- |
| `int const*`       | 指向 常量 int 的指针   | ❌ 不可改 | ✅ 可改         |
| `int* const`       | 常量指针，指向 int     | ✅ 可改   | ❌ 不可改       |
| `const int* const` | 常量指针，指向常量 int | ❌ 不可改 | ❌ 不可改       |

#### 6.4、应用场景

- `int const\*`（指向常量的指针）
   常用于函数参数，保证函数内部不会修改传入的对象：

  ```c++
  void print(const int* p) {
      std::cout << *p << std::endl;
  }
  ```

  相当于 只读访问。

- `int\* const`（常量指针）
   常用于实现“固定绑定”，比如类成员内部保存某个对象的引用地址：

  ```c++
  class Wrapper {
      int* const ptr;  // 必须在构造函数初始化，之后不能换对象
  public:
      Wrapper(int* p) : ptr(p) {}
      void setValue(int v) { *ptr = v; }
  };
  ```

### 7、以下说法是否正确？shared_ptr的行为最接近原始指针，所以可以在任何地方替换原始指针，消灭内存泄漏。

**这个说法不正确**。`std::shared_ptr` 能解决很多由“忘记 `delete`”引起的泄漏，但它并**不是**原始指针的完全替代，也不能在任何地方替换原始指针来“消灭内存泄漏”。

#### `shared_ptr` 做了什么?

1. 管理共享所有权（reference-counted ownership）。最后一个 `shared_ptr` 销毁时会调用删除器释放资源。
2. 弱化手动 `delete` 的职责，避免大多数因忘记释放导致的内存泄漏（当且仅当使用方式正确时）。
3. 可与自定义删除器一起管理非内存资源（文件描述符、句柄等）。
4. `std::make_shared` 可以合并对象与 control block 的分配，减少内存碎片与额外分配开销。

#### 为什么不能**随处**替换原始指针

##### 1) 循环引用（reference cycle）会导致内存泄漏

`shared_ptr` 通过引用计数释放对象。若对象之间互相 `shared_ptr` 持有，会造成引用计数永远不为 0，从而泄漏：

```c++
struct B;
struct A { std::shared_ptr<B> b; };
struct B { std::shared_ptr<A> a; };

auto a = std::make_shared<A>();
auto b = std::make_shared<B>();
a->b = b;
b->a = a; // 循环：A<->B，引用计数永远 >0 -> 泄漏
```

解决：把其中一侧改为 `std::weak_ptr`（观察者，不增加引用计数）。

##### 2) 错误地从同一原始指针构造多个 `shared_ptr` 会导致双重删除（UB）

```c++
int* p = new int(42);
std::shared_ptr<int> s1(p);
std::shared_ptr<int> s2(p); // 错误：s1 和 s2 有各自 control block -> 双重 delete -> UB
```

正确做法：要么用一个 `shared_ptr` 构造，再复制它；要么使用 `make_shared`。

##### 3) `shared_ptr` 并非“零成本” —— 性能/内存开销

- 控制块（control block）通常包含 `use_count`、`weak_count`、deleter 等；若用 `make_shared`，对象与 control-block 合并为一次分配，否则多一次分配。
- 每次拷贝/销毁 `shared_ptr` 都需要原子增加/减少引用计数（`fetch_add`/`fetch_sub`），在高并发/热路径上开销不可忽视。
- 相比原始指针，`shared_ptr` 的语义更重、内存更大（一般至少两个指针大小 + control block），并影响缓存局部性。

##### 4) 生命周期与析构时机不同（延迟释放问题）

- `shared_ptr` 的资源释放依赖最后一个引用离开作用域，析构时机不再由创建者直接控制。这在有些场景（需要确定顺序或立即释放资源，比如释放大量内存、关闭句柄）不是期望行为。
- 若你想立即释放资源，`unique_ptr` 或显式 `reset()` 更合适。

##### 5) 并发语义容易被误解

- 针对不同 `shared_ptr` 实例（指向同一对象）上的引用计数操作是线程安全的（标准保证：修改 control block 的原子操作）。
- 但对同一个 `shared_ptr` 对象的并发读写（例如一个线程在复制另一个线程在 `reset()`）需要外部同步。
- `shared_ptr` 线程安全语义经常被误用，导致 race。

##### 6) 不能替代“非所有权指针/引用”语义

- 许多接口/函数期望“非所有者”访问（例如观察者、callback 的临时访问），这时传入 `shared_ptr` 会误传“拥有权”的信号，且拷贝 `shared_ptr` 会增加计数。
- 正确做法：对于非拥有关系，用裸指 `T*`、`T&` 或 `std::weak_ptr`（如果要检测被管理对象是否还存在）更合适。

##### 7) `enable_shared_from_this` 的陷阱

若对象不是通过 `shared_ptr` 创建（例如用裸 `new`），而对象内部调用 `shared_from_this()` 会产生异常或 UB。必须保证对象最初由 `shared_ptr` 管理（通常用 `make_shared`）。

#### 什么时候该用 `shared_ptr`？

- 优先用 `unique_ptr`：默认写代码时先想是否能用 `unique_ptr`（唯一所有者，低开销，明确语义）。
- 使用 `shared_ptr` 的场景：确实需要多个相互独立的所有者（比如对象可能存在于多个数据结构中，且谁都可能先释放）；同时对性能和延迟可接受。
- 避免把 `shared_ptr` 用作“全场景万能指针”：不要用它来替代所有裸指/引用，用在真正需要共享所有权的点。
- 非拥有访问：函数参数建议接受裸指或引用（或 `const T*` / `T&`），只有当函数内部需要延长对象寿命时才接受 `shared_ptr`（或 `std::shared_ptr<T> p` 以表所有权意图）。
- 若有跨对象观察关系：使用 `weak_ptr` 打破循环引用或作观察者。

### 8、C++是否支持lambda表达式？如何使用？

**C++ 支持 lambda 表达式**，从 C++11 引入基础语法，之后在 C++14/C++17/C++20 上逐步扩展为更强大、更安全、性能更好的特性。

#### 8.1. 概念

Lambda（闭包）是一个匿名的可调用对象（编译器生成一个“闭包类型” — 一个未命名的类），它可以捕获外层作用域的变量并在其 `operator()` 中执行函数体。Lambda 在 C++11 引入，并在之后的标准中增强。

#### 8.2. 语法总览（最常见形式）

```c++
[capture-clause] (parameters) cv-qualifier exception-spec -> return-type {
    body
}
```

常见可选项：

- `capture-clause`（捕获）：`[]`, `[=]`, `[&]`, `[x, &y]`, `[this]`, `[v = std::move(x)]`（C++14 init-capture）等。
- `parameters`：和普通函数参数一样，C++14 支持用 `auto` 做泛型参数（generic lambda）。
- `mutable`：使 `operator()` 非 `const`，允许修改按值捕获的成员。
- `-> return-type`：尾置返回类型（通常省略，编译器会推导）。
- `constexpr` / `noexcept`：可用于修饰（部分版本支持 `constexpr` lambda）。

#### 8.3、基本示例（C++11 起）

```c++
auto f = [](int x){ return x + 1; };
std::cout << f(10); // 11
```

##### 没有捕获的 lambda

等价于一个无状态的函数对象，可以转换为函数指针：

```c++
int (*fp)(int) = [](int x){ return x+1; }; // 可行（无捕获）
```

#### 8.4、捕获（capture）详解 — 最重要的部分

##### 捕获的分类与语义

- 捕值（by value）：`[x]` 或 默认 `[=]`。闭包内部保存所捕变量的副本（拷贝或按 init-capture 初始化）；对副本的修改不会影响外层变量（除非用 `mutable`）。

- 捕引用（by reference）：`[&x]` 或 默认 `[&]`。保存引用（内部实际上是引用/指针），通过闭包修改会影响外层变量。注意：引用会悬空如果被捕的变量在 lambda 被调用前就已经销毁。

- 捕 `this`：`[this]`（C++11）会把 `this` 指针捕进闭包（保存为指针）——如果对象在 lambda 执行前被销毁，访问会悬空。C++17 引入 `[*this]` 可以把 `*this` 的拷贝捕入闭包（按值捕拷贝整个对象），便于安全地在对象被销毁后仍使用闭包中的副本。[Stack Overflow](https://stackoverflow.com/questions/41637451/c17-lambda-capture-this?utm_source=chatgpt.com)

- 初始化捕获（init-capture，C++14）：`[name = expr]`，可以把表达式的结果初始化到闭包的成员，常用于 **move capture**（把移动-only 类型移入闭包），例如：

  ```c++
  auto up = std::make_unique<int>(42);
  auto f = [p = std::move(up)](){ return *p; }; // p 是闭包内的成员，持有 unique_ptr
  ```

  初始化捕获和泛型 lambda（见下一节）是 C++14 的重要扩展。

##### 默认捕获的混合用法

- `[=, &x]`：默认按值捕，但对 `x` 按引用捕。
- `[&, x]`：默认按引用捕，但对 `x` 按值捕。
   **建议**：尽量**显式**列出需要的捕获，避免滥用 `[=]` 或 `[&]`（可读性差、易引入悬垂引用/不必要的持有）。

#### 8.5、`mutable`、`const` 与 `operator()` 的关系

默认情况下，闭包的 `operator()` 是 `const` 成员函数（因此不能修改按值捕获的成员）。`mutable` 关键字会让 `operator()` **非 const**，从而允许修改按值捕获的成员（只是修改副本，不影响外层变量）。

示例：

```c++
int x = 0;
auto g = [x]() mutable { x += 1; return x; };
std::cout << g(); // 1
std::cout << g(); // 2
std::cout << x;   // 0  （外部 x 未变）
```

#### 8.6、返回类型推导（C++11/14 的差异）

- C++11：只有当 lambda 体是单个 `return` 语句时，编译器可推导返回类型；更复杂的情形需要显式尾置返回类型 `-> T`。
- C++14：通用的返回类型推导（像普通函数 `auto` 一样），可以给出多个 `return` 语句但类型需可推导一致。

示例（若推导失败，可显式写）：

```c++
auto f = [](int x)->double { if (x>0) return 1.0; else return 0.0; }; // 显式尾置
```

#### 8.7、泛型 lambda（generic lambda）与 C++20 的模板化 lambda

- C++14：支持在参数列表中使用 `auto`，生成模板化的 `operator()`（称为“generic lambda”）。例如：

  ```c++
  auto add = [](auto a, auto b){ return a + b; };
  add(1, 2); // int
  add(1.0, 2.5); // double
  ```

- C++20：允许在 lambda 直接写模板参数列表（`[]<typename T>(T x){}`），增强模板化能力（可以指定非参数模板参数等）。

#### 8.8、捕获移动-only 对象（实用技巧）

要把 `unique_ptr` 等 move-only 对象放进闭包，**必须**用 init-capture（C++14）：

```c++
auto up = std::make_unique<Foo>();
auto f = [p = std::move(up)](){ p->do_something(); }; // OK
```

这样闭包拥有 `unique_ptr` 的所有权；注意 `up` 之后为空。

#### 8.9、捕获 this 的安全性（常见坑与安全做法）

- ` [this]`：捕获 `this` 指针（**只是指针**）。如果 lambda 在对象销毁后才执行，访问会产生悬挂指针（UB）。

- 常见解决：

  - 把对象用 `shared_ptr` 管理，捕获 `weak_ptr`，在闭包运行时 `lock()` 检查并转为 `shared_ptr`（避免延长生命周期又防止悬挂）：

    ```c++
    auto wp = std::weak_ptr<My>(shared_ptr_this);
    auto f = [wp]{ if(auto sp = wp.lock()) sp->do_work(); };
    ```

  - C++17 的 `[*this]`：把 `*this` 的副本按值捕获（拷贝整个对象）——适用于对象能安全拷贝且希望闭包持有对象副本的场景。

#### 8.10、闭包类型（底层）与性能

- 每个 lambda 对应一个未命名的闭包类型，捕获列表成为该类型的非静态数据成员；`operator()` 被生成为成员函数（默认是 `const`）。闭包对象通常很小（只包含捕获的数据），编译器能很好地内联优化。
- 但注意：把捕获 lambda 放入 `std::function` 会发生类型擦除，有可能引发额外分配和运行时开销；如果在性能敏感的地方（热路径）尽量避免 `std::function` 的泛化开销，直接用模板或 auto 保存闭包更好。

#### 8.11、转换到函数指针的限制

- 没有捕获的 lambda（闭包为空）可以被隐式转换为普通函数指针。
- 有捕获的 lambda 不可转换为函数指针（因为它有状态）。
   示例：

```c++
auto no_cap = [](int x){ return x+1; };
int (*fp)(int) = no_cap; // OK

int y = 5;
auto cap = [y](int x){ return x + y; };
// int (*fp2)(int) = cap; // 错误：有捕获，不能转换
```

#### 8.12、与线程、异步的交互（常见错误）

- 如果用 lambda 启动 `std::thread`，确保捕获方式安全（不要捕引用到局部变量；线程可能在局部变量离开作用域后才运行）：

  ```c++
  int x = 42;
  std::thread t([x]{ /* safe: captured by value */ });
  // vs
  std::thread t2([&]{ /* danger if x goes out of scope */ });
  ```

- 把 move-only 资源传给线程：`std::thread` 构造会复制/移动可调用对象，所以 init-capture 移动到闭包通常可行；也可用 `std::move` 把 `std::packaged_task` 等移动进线程。

#### 8.13、常见面试题/陷阱（务必记住）

- 捕获 `i` 的 for-loop 问题（那种你在循环里创建多个 lambda 用于 later 调用）：要按值捕 `i`，否则闭包捕到的是同一个 `i`（循环结束后通常为最终值）。

  ```c++
  std::vector<std::function<int()>> v;
  for(int i=0;i<3;++i){
      v.push_back([i]{ return i; }); // correct: capture i by value
      // v.push_back([&]{ return i; }); // bug: all lambdas return 3
  }
  ```

- 捕获 everything (`[=]`/`[&]`) 的可读性与安全性问题：可导致无意中持住某些变量或产生悬挂引用。

- mutable：不要用来“破坏”外部变量的直觉——按值捕获 + mutable 只是修改闭包内部的副本，不影响外部变量。

#### 8.14、各标准的关键扩展（快速速览）

- C++11：引入 lambda 基本语法（捕获、尾置返回类型、`mutable` 等）。
- C++14：引入 generic lambda（`auto` 参数）与 init-capture（`[x = expr]`，方便 move-capture）。
- C++17：允许 `[*this]`（按值捕获 `*this` 的副本）；并允许在常量表达式中使用 lambda（`constexpr` lambda 的支持已增强）。（C++17 改善了对 lambda 在常量表达式中使用的支持。）
- C++20：允许为 lambda 写模板参数列表（`[]<typename T>(T x){}`），模板化能力更强。

#### 8.15、实战建议

1. 先想所有权语义：捕值还是捕引用？是否会把 lambda 存活到原变量失效后？若会，按值捕获或使用 `weak_ptr`/`shared_ptr`。
2. 显式捕获，避免盲用 `[=]` 或 `[&]`。
3. 移动资源进闭包：用 init-capture `[p = std::move(ptr)]`。
4. 不要在构造/析构期间依赖虚拟调度与 this 捕获（构造中捕 `this` 并不安全）。
5. 在性能敏感场景，避免把捕获 lambda 放到 `std::function`；使用 `auto` 或模板参数保留闭包原类型。
6. 对并发场景，用值捕获或 `weak_ptr` + `lock()` 做安全检查；避免引用捕获局部变量给线程带来的悬挂风险。
7. 如果需要把 lambda 转成回调（C API），保证**无捕获**或自己管理 C 风格上下文指针。

#### 8.16、精选示例（不同场景总结）

##### 简单捕值/捕引用

```c++
int a = 1;
auto v = [a]() { return a + 1; }; // 捕值
auto r = [&a]() { a += 2; };      // 捕引用
```

##### move-only capture（C++14）

```c++
auto up = std::make_unique<int>(5);
auto f = [p = std::move(up)] { return *p + 1; };
// up 已被移动，f 拥有该 unique_ptr
```

##### generic lambda（C++14）

```c++
auto add = [](auto x, auto y) { return x + y; };
add(1, 2); add(1.5, 2.5);
```

##### C++20 模板 lambda

```c++
auto map = []<typename F, typename T>(F f, T x){ return f(x); };
map.template operator()<std::function<int(int)>, int>([](int y){return y+1;}, 3);
```

------

#### 8.17、参考资料

- cppreference — Lambda expressions（标准详尽说明）。[Cppreference](https://en.cppreference.com/w/cpp/language/lambda.html?utm_source=chatgpt.com)
- C++14 / C++17 / C++20 特性摘要（init-capture、generic lambda、template-parameter-list 等）。

### 9、auto和decltype在使用上有什么不同？

#### 9.1、`auto`

- 用途：根据 初始化表达式 自动推导变量的类型。
- 限制：必须有初始化表达式。
- 规则：和模板参数推导类似，会忽略顶层 `const`，引用会被剥掉。

##### 示例

```c++
int x = 10;
const int cx = x;
int& rx = x;

auto a = x;   // int
auto b = cx;  // int (顶层 const 被忽略)
auto c = rx;  // int (引用被忽略)

auto& d = cx; // const int& (显式加引用，才保留 const)
```

👉 `auto` 更像是“声明变量时偷懒用的类型占位符”，核心是取初始化表达式的值类型。

#### 9.2、`decltype`

- 用途：提取一个 表达式的类型（而不是值）。
- 规则：
  - 如果表达式是一个变量，结果就是该变量的类型（保留 `const`、引用）。
  - 如果是复杂表达式，规则比较细致（是否是左值、是否加括号都会影响）。

##### 示例

```c++
int x = 10;
const int cx = x;
int& rx = x;

decltype(x) a = 0;   // int
decltype(cx) b = 0;  // const int
decltype(rx) c = x;  // int&

decltype((x)) d = x; // int&   注意括号！(x) 是左值
```

`decltype` 的精度比 `auto` 高，能准确反映表达式的完整类型，包括 `const` 和引用。

#### 9.3、对比总结

| 特性           | `auto`                 | `decltype`                            |
| -------------- | ---------------------- | ------------------------------------- |
| 是否必须初始化 | ✅ 必须有初始化         | ❌ 不需要初始化                        |
| 是否保留 const | ❌ 忽略顶层 const       | ✅ 保留                                |
| 是否保留引用   | ❌ 忽略                 | ✅ 保留                                |
| 用途           | 声明变量类型，简化代码 | 获取任意表达式的精确类型              |
| 典型场景       | `auto it = v.begin();` | `decltype(v.begin()) it = v.begin();` |

### 10、tcp的三次握手、四次挥手的时序图。请标明状态。

![TCP连接建立与断开时序图](https://cdn.jsdelivr.net/gh/aqjsp/photos/image-20250908005737491.png)

### 11、你了解哪些设计模式？

之前专门整理过，C++设计模式全解析：[点击阅读](https://mp.weixin.qq.com/s/JGUg289XfJQS8IvTP6O3iA)

### 12、你是否了解SQL的事务隔离？隔离级别分为几级？每级代表什么含义？

#### 12.1、SQL 事务隔离级别

SQL 标准定义了 4 种隔离级别（由低到高），不同级别对并发下的问题（脏读、不可重复读、幻读）的控制能力不同。

| 隔离级别                     | 允许的并发问题         | 特点                                                         |
| ---------------------------- | ---------------------- | ------------------------------------------------------------ |
| Read Uncommitted（读未提交） | 脏读、不可重复读、幻读 | 几乎没有隔离，性能最高，但数据不安全                         |
| Read Committed（读已提交）   | 不可重复读、幻读       | 避免脏读，大多数数据库（如 Oracle）默认级别                  |
| Repeatable Read（可重复读）  | 幻读                   | 保证多次读取一致，MySQL InnoDB 默认级别（实际通过 MVCC 避免幻读） |
| Serializable（可串行化）     | 无                     | 最严格，完全串行执行，代价最大，性能最差                     |

#### 12.2、各隔离级别详解

##### 1. Read Uncommitted（读未提交）

- 含义：一个事务可以读取到另一个事务尚未提交的数据。
- 问题：可能出现 脏读。

示例：

```sql
-- 事务B 修改 salary 为 5000，但尚未提交
UPDATE employees SET salary = 5000 WHERE id = 1;

-- 事务A 读取到了 5000（脏数据）
SELECT salary FROM employees WHERE id = 1; -- 读到 5000

-- 事务B 回滚
ROLLBACK;

-- 此时事务A 读到的数据是无效的！
```

##### 2. Read Committed（读已提交）

- 含义：一个事务只能读取到别的事务已经提交的数据。
- 解决：避免了脏读。
- 问题：仍可能出现 不可重复读。

示例：

```sql
-- 事务A 第一次读
SELECT salary FROM employees WHERE id = 1; -- 得到 3000

-- 事务B 修改并提交
UPDATE employees SET salary = 4000 WHERE id = 1;
COMMIT;

-- 事务A 第二次读
SELECT salary FROM employees WHERE id = 1; -- 得到 4000 ← 不可重复读！
```

Oracle 默认隔离级别就是 Read Committed。

##### 3. Repeatable Read（可重复读）

- 含义：同一事务中多次读取同一行，结果保持一致（即使别的事务修改并提交了数据）。
- 解决：避免了脏读、不可重复读。
- 问题：仍可能出现 幻读。

示例：

```sql
-- 事务A 第一次查询
SELECT * FROM employees WHERE salary > 3000; -- 返回 2 行

-- 事务B 插入并提交
INSERT INTO employees (name, salary) VALUES ('Alice', 5000);
COMMIT;

-- 事务A 第二次查询
SELECT * FROM employees WHERE salary > 3000; -- 返回 3 行 ← 幻读！
```

MySQL InnoDB 默认隔离级别是 Repeatable Read，而且通过 MVCC + Next-Key Lock 实际上也避免了幻读。

##### 4. Serializable（可串行化）

- 含义：最高隔离级别，事务完全串行执行。
- 解决：避免所有并发问题（脏读、不可重复读、幻读）。
- 代价：性能最差，需要加大量锁，事务之间几乎是排队执行。
- 场景：只有在金融等对数据一致性极度敏感的场景才会使用。

### 13、数据库中，索引的作用是什么？查询聚簇索引和普通索引的区别是什么？

#### 索引的作用

1. 加速查询检索：通过（通常是）B+树或哈希结构，将查找从全表扫描降为对数级（如 O(logN)）的树查找，大幅减少磁盘/页读取次数。
2. 支持有序访问：B+树索引的叶子节点天然有序，便于范围查询、顺序扫描、ORDER BY、GROUP BY 的优化（能“按序取数”，避免额外排序）。
3. 保证或辅助约束：唯一索引/主键索引用于保证数据唯一性，避免重复写入。
4. 加速连接：连接条件上的索引可以显著降低 Join 的代价。
5. 覆盖索引优化：当查询的列全部包含在某个索引里时，可直接从索引返回数据，避免回表，提高性能。
6. 代价与权衡：索引需要额外存储空间，写入/更新需要维护索引结构，可能产生页分裂与碎片，写多读少的场景不宜建过多索引。

#### 聚簇索引 vs 普通（非聚簇/二级）索引

**以 MySQL InnoDB 为例**

1. ##### 物理组织与存储

- 聚簇索引：
  - 表数据按聚簇键的顺序组织存储；聚簇索引的 B+ 树叶子节点直接存放整行数据。
  - 在 InnoDB 中，主键即聚簇索引；若无显式主键，会选择第一个非空唯一索引作为聚簇键；再没有则生成隐藏的 rowid 作为聚簇键。
- 普通索引（二级索引）：
  - B+ 树叶子节点存放的是“索引列的值 + 聚簇键（主键）”，不存整行数据。
  - 通过普通索引命中后，通常还需用聚簇键再去聚簇索引树上取整行数据（回表）。

2. ##### 数量与唯一性

- 聚簇索引：每个表只能有一个（因为数据只能按一种顺序物理组织）。
- 普通索引：可以有多个（包含唯一索引、复合索引等）。
3. ##### 查询路径与回表
- 使用聚簇索引查询：命中后已是整行数据，直接返回，不需要回表。
- 使用普通索引查询：命中二级索引后拿到主键，再去聚簇索引按主键取整行（回表）；如果查询能“覆盖索引”（select 的列都在该索引中），则可避免回表。
4. ##### 有序访问与范围扫描
- 聚簇索引：因为数据物理上按聚簇键排序，对聚簇键的范围查询/排序/前缀扫描非常高效，顺序读更友好。
- 普通索引：对自身索引列有序（索引树叶子有序），能用于范围/排序优化，但返回整行数据时仍可能回表。
5. ##### 写入、更新与碎片
- 聚簇索引：
  - 插入非递增主键可能导致频繁页分裂、产生碎片，写放大较明显；主键更新代价很高（相当于移动整行）。
  - 实践建议：选择“短、单调递增”的主键（如 `BIGINT AUTO_INCREMENT`）以降低页分裂。
- 普通索引：
  
  维护成本主要体现在索引树的插入/删除/更新及其记录的主键更新；行移动对二级索引影响较小（InnoDB 二级索引叶子存主键，不存物理地址）。
6. ##### 空间占用
- 聚簇索引：叶子存整行，索引本身较大，但不需要再为“表数据”单独存一份结构（InnoDB 的表数据即聚簇索引叶子）。
- 普通索引：每建一个就新增一棵索引树，占用额外空间；但叶子记录比整行小（只含索引列+主键）。
7. ##### 典型查询示例
- 精确查找（`where user_id=...`）：
  - 如果`user_id`是二级索引：先在`user_id`索引树命中，得到主键，再回表到聚簇索引取整行。
  - 如果查询只取 `user_id`、`status`，而这两列在同一个联合二级索引(`user_id`, `status`)里，则可覆盖索引直返，避免回表。
- 范围+排序（`where created_at between ... and ... order by created_at limit N`）：
  - 若`created_at`为聚簇键：自然顺序扫描，代价低。
  - 若`created_at`为二级索引：可以在二级索引上顺序取主键，但取整行需回表（多次随机访问）；若能覆盖索引可大幅优化。

### 14、翻转单链表？

#### 思路

1. 使用三个指针：
   - `pre`：指向前一个节点，初始为 nullptr
   - `cur`：指向当前节点，初始为头节点
   - `tmp`：临时保存下一个节点，防止断链后丢失
2. 遍历链表，逐个改变节点的 [next](javascript:void(0)) 指向：
   - 用 `tmp` 保存当前节点的下一个节点
   - 将当前节点的 [next](javascript:void(0)) 指向 `pre`
   - `pre` 和 `cur` 向后移动一位
3. 当遍历结束后，`pre` 指向原链表的最后一个节点，也就是新链表的头节点

#### 参考代码（C++）

```c++
// 定义链表节点结构
struct ListNode {
    int val;
    ListNode* next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode* next) : val(x), next(next) {}
};

// 打印链表
void printList(ListNode* head) {
    ListNode* cur = head;
    while (cur != nullptr) {
        std::cout << cur->val << " -> ";
        cur = cur->next;
    }
    std::cout << "NULL" << std::endl;
}

// 反转链表 - 迭代方法
ListNode* reverseList(ListNode* head) {
    // 如果链表为空或只有一个节点，直接返回
    if (head == nullptr || head->next == nullptr) {
        return head;
    }

    ListNode* pre = nullptr;  // 前一个节点
    ListNode* cur = head;     // 当前节点

    // 遍历链表，逐个反转节点指向
    while (cur != nullptr) {
        ListNode* tmp = cur->next;  // 临时保存下一个节点
        cur->next = pre;            // 当前节点指向前一个节点
        pre = cur;                  // 前一个节点后移
        cur = tmp;                  // 当前节点后移
    }
    
    // 最终pre指向原链表的最后一个节点，即新链表的头节点
    return pre;
}

// 反转链表 - 递归方法
ListNode* reverseListRecursive(ListNode* head) {
    // 递归终止条件：空节点或只有一个节点
    if (head == nullptr || head->next == nullptr) {
        return head;
    }
    
    // 递归反转后面的节点
    ListNode* newHead = reverseListRecursive(head->next);
    
    // 反转当前节点和下一个节点的连接关系
    head->next->next = head;
    head->next = nullptr;
    
    return newHead;
}

int main() {
    // 创建测试链表: 1 -> 2 -> 3 -> 4 -> 5 -> NULL
    ListNode* head = new ListNode(1);
    head->next = new ListNode(2);
    head->next->next = new ListNode(3);
    head->next->next->next = new ListNode(4);
    head->next->next->next->next = new ListNode(5);
    
    std::cout << "原始链表: ";
    printList(head);
    
    // 使用迭代方法反转链表
    ListNode* reversedHead = reverseList(head);
    
    std::cout << "反转后链表: ";
    printList(reversedHead);
    
    // 释放内存
    ListNode* cur = reversedHead;
    while (cur != nullptr) {
        ListNode* tmp = cur->next;
        delete cur;
        cur = tmp;
    }
    
    return 0;
}
```

运行结果：

![运行结果](https://cdn.jsdelivr.net/gh/aqjsp/photos/image-20250908011704926.png)

### 15、判断一颗树是不是二叉搜索树？

#### 二叉搜索树定义

对于树中每一个节点：

- 它的左子树中所有节点的值 **<** 该节点的值
- 它的右子树中所有节点的值 **>** 该节点的值
- 左右子树也必须是 BST

> 注意：是“左子树所有节点”，不是“左孩子”！很多错误解法只比较了直接孩子。

#### 思路

##### 上下界递归法

递归 + 维护当前节点允许的取值范围 [minVal, maxVal]

- 根节点没有限制 → 范围是 `(−∞, +∞)`
- 左孩子必须 `< root->val` → 新范围 `(−∞, root->val)`
- 右孩子必须 `> root->val` → 新范围 `(root->val, +∞)`

递归过程中不断缩小区间，一旦节点值越界 → 不是 BST。

#### 参考代码（C++）

```c++
#include <iostream>
#include <climits>
using namespace std;

// 二叉树节点定义
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
    bool isValidBST(TreeNode* root) {
        return isValidBSTHelper(root, LONG_MIN, LONG_MAX);
    }

private:
    bool isValidBSTHelper(TreeNode* node, long minVal, long maxVal) {
        if (node == nullptr) return true;  // 空树是 BST

        // 当前节点值必须在 (minVal, maxVal) 范围内
        if (node->val <= minVal || node->val >= maxVal) {
            return false;
        }

        // 递归检查左右子树
        // 左子树：上限变为当前节点值
        // 右子树：下限变为当前节点值
        return isValidBSTHelper(node->left, minVal, node->val) &&
               isValidBSTHelper(node->right, node->val, maxVal);
    }
};

int main() {
    /*
        构建 BST:
            5
           / \
          3   8
         / \ / \
        2  4 7  9
    */
    TreeNode* root = new TreeNode(5);
    root->left = new TreeNode(3);
    root->right = new TreeNode(8);
    root->left->left = new TreeNode(2);
    root->left->right = new TreeNode(4);
    root->right->left = new TreeNode(7);
    root->right->right = new TreeNode(9);

    Solution sol;
    cout << (sol.isValidBST(root) ? "✅ 是 BST" : "❌ 不是 BST") << endl;

    // 修改一个节点制造非法 BST
    root->left->right->val = 6; // 现在左子树中有 6 > 5（根）

    cout << (sol.isValidBST(root) ? "✅ 是 BST" : "❌ 不是 BST") << endl;

    return 0;
}
```

运行结果：

![运行结果](https://cdn.jsdelivr.net/gh/aqjsp/photos/image-20250908013507448.png)



## 二面

### 1、C++虚函数机制？

##### 1) 为什么需要“虚函数”？

虚函数（`virtual`）提供运行时多态（dynamic dispatch），允许通过基类指针/引用在运行时根据对象的真实类型调用对应的函数实现。
用途举例：接口/抽象基类、策略模式、插件系统、工厂返回基类指针而隐蔽具体类型实现等。

```cpp
struct Base {
    virtual void foo() { std::cout << "Base\n"; }
    virtual ~Base() = default;
};

struct Derived : Base {
    void foo() override { std::cout << "Derived\n"; }
};

Base* b = new Derived;
b->foo(); // calls Derived::foo()  —— 运行时决定
```

##### 2) 语言语义重点（标准层面）

- 若类中有至少一个 `virtual` 成员函数，则该类是 polymorphic（多态类）。
- 通过基类指针/引用调用虚函数时，调用目标在运行时确定（动态绑定）。
- 在构造函数/析构函数中，虚函数调用按当前正在构造/析构的类解析（不会调用派生类覆盖实现）。
- `virtual` 与 `override` / `final` 可配合使用以显式约束覆盖/禁止覆盖。
- `=0` 定义纯虚函数（abstract），但纯虚函数仍可提供定义（有时用于基类实现可被派生显示调用：`Base::f()`）。

##### 3) 常见实现：`vptr` + `vtable`（虚表）

编译器常见实现（Itanium/MSVC 等 ABI 变体细节略有不同）：

- 每个多态类（polymorphic class）编译时会生成一个虚表（vtable）：一个指向实现函数（函数指针）的数组/表，通常还包含 RTTI/typeinfo 指针、offset 信息等。vtable 通常存放在只读静态数据区。
- 每个对象在其实例内含有一个或多个隐藏指针 `vptr`（vtable pointer），指向该对象的类所对应的 vtable。通常 `vptr` 存放在对象的起始位置（但这不是语言强制的，仅为常见实现）。
- 虚函数调用编译成：读取对象的 `vptr` → 到 vtable 中取对应 slot 的函数指针 → 间接跳转调用（one indirection）。

ASCII 示意（单继承典型布局）：

```
vtable_for_Derived: [ &Derived::foo, &Derived::bar, &typeinfo, ... ]   // static in .rodata
object (Derived):
  [ vptr ] -> points to vtable_for_Derived
  [ members... ]
```

调用流程（伪汇编）：

```
load vptr from object
load func_ptr = vptr[offset_for_foo]
call func_ptr
```

##### 4) 构造/析构期间的虚调用行为

- 在构造期间，对象被视为处于基类子对象的动态类型；基类构造体里对虚函数的调用不会分派到派生类实现，而是解析到基类或当前级别的实现。原因：派生类尚未构造完成，其状态可能未初始化。
- 同理，在析构期间，派生析构先运行，虚表/动态类型退回到基类层次，虚函数调用不会路由到已析构的派生实现。

这是语言规定（保证安全），实现上通常在每个构造/析构阶段把 `vptr` 临时指向当前构造中的类的 vtable。

示例：

```cpp
struct Base {
    Base() { foo(); } // calls Base::foo()
    virtual void foo() { std::cout << "Base\n"; }
    virtual ~Base() { foo(); } // calls Base::foo()
};
struct Derived : Base {
    void foo() override { std::cout << "Derived\n"; }
};
new Derived; // prints Base (in Base ctor), Derived (in Derived ctor if called), ...
```

##### 5) 单继承 vs 多重继承 vs 虚继承：vptr 布局的差异

###### 单继承

- 每个对象通常只有 1 个 vptr（放在对象开始）指向其类 vtable。

###### 多重继承

- 如果类从多个基类继承且这些基类都是 polymorphic，编译器常常需要为每个基类子对象维护一个 vptr，因为每个基类子对象可能有不同的 vtable layout/offset。
- 结果：派生对象可能包含多个 vptr（每个子对象一份）。

示意（Derived : Base1, Base2）：

```c++
object layout:
 [ vptr_for_Base1 ]  // Base1 subobject
 [ Base1 members ]
 [ vptr_for_Base2 ]  // Base2 subobject
 [ Base2 members ]
 [ Derived members ]
```

###### 虚继承

虚继承引入“虚基类子对象”共享，需要存储额外的 **offset/ptables/vbptr**（不同 ABI 名称不同）以在运行时找到虚基类位置；vtable 通常含有用于定位虚基子对象的偏移信息。实现复杂，编译器会在 vtable 中存储额外数据或在对象中添加 `vbptr` 等辅助结构。

> **重点**：确切布局与产生多少 vptr / vbptr 取决于编译器/ABI（Itanium C++ ABI、MSVC ABI 各不相同）。不过概念一致：多继承与虚继承需要额外的指针/偏移信息来支持正确的动态分派与 `this` 调整。

------

##### 6) “this” 指针调整与 thunk（调整函数）

在多重继承或虚继承中，从基类指针调用派生的覆盖函数时，函数实现里 `this` 期望指向该类的起始地址。如果基类子对象相对于最派生对象有非零偏移，调用时需要调整 this 指针到派生对象的实际起始地址或相应子对象地址。实现方式：

- 编译器可以在 vtable 的相应 slot 放入一个 thunk（小的 wrapper），thunk 做 `this` 的偏移调整后跳转到真实函数实现。
- 也可以在 vtable entry 存储`function pointer + this_delta`（复杂的PMF/ABI 表示），运行时调用时先加偏移再调用。

示例（多继承导致 this_adjustment）：

```c++
Base1* pb = static_cast<Base1*>(new Derived());
pb->foo(); // vtable entry for foo may be a thunk that adjusts 'this' and calls Derived::foo
```

##### 7) 指向成员函数的指针（pointer-to-member-function）与虚函数

- 指向成员函数的指针（PMF）在虚函数时表现复杂：对非虚函数是直接函数指针；对虚函数PMF，可能不是直接函数地址，而是一个能够间接解析 vtable slot 的结构（在 Itanium ABI 中 PMF 有复杂编码）。
- 调用 PMF 需要知道是否为虚函数并做相应取 vtable 操作或调用 thunk。

（细节依 ABI，面试时以“实现复杂，需要借助 ABI 文档”作答即可。）

##### 8) 纯虚函数与抽象类

- 声明 `virtual void f() = 0;` 将该类变为抽象类（不能实例化）。
- 可以为纯虚函数提供实现（例如 `void Base::f(){...}`），但仍然是抽象（类不能实例化），派生类可以显式调用 `Base::f()`。这常用于提供公共实现供派生调用。

##### 9) RTTI、`type_info`、`dynamic_cast`

- 多态类的 vtable 通常也包含指向 `type_info`（RTTI）结构的指针。`dynamic_cast` 和 `typeid` 等运行时类型检查利用这部分信息。
- `dynamic_cast` 在向下转换（`Base*` → `Derived*`）时需要访问对象的类型信息并做子对象布局计算（多继承/虚继承时更复杂）。

##### 10) 调用开销与编译器优化（devirtualization）

- 典型虚函数调用成本：一次内存加载（读取 vptr）+ 一次访问（读取函数指针）+ 一次间接跳转。这阻碍内联与某些编译器优化。
- 优化手段：
  - devirtualization（去虚化）：编译器在编译期能确定对象的动态类型（例如对象是 `final`、在函数内是局部变量、新建后立即用）时，会直接将虚调用转为静态直接调用，从而允许内联与更优化代码。
  - LTO/whole-program optimization：链接时优化可帮助去虚化更多调用。
  - Profile-guided optimization（PGO）：利用运行时热点信息做投机内联/inline cache。
  - `final` / `sealed`：将类或方法标记为不可派生/不可覆盖，允许编译器安全地去虚化。

##### 11) 代码示例（单继承、多继承、thunk 示例）

单继承示例（虚调用）：

```cpp
#include <iostream>
struct Base {
    virtual void f() { std::cout << "Base::f\n"; }
    virtual ~Base() = default;
};
struct Derived : Base {
    void f() override { std::cout << "Derived::f\n"; }
};
int main() {
    Base* b = new Derived;
    b->f(); // Derived::f
    delete b;
}
```

多继承 + this 调整示例（概念性）：

```cpp
#include <iostream>
struct A { virtual void f() { std::cout << "A\n"; } };
struct B { virtual void f() { std::cout << "B\n"; } };
struct C : A, B {
    void f() override { std::cout << "C\n"; }
};
int main() {
    A* pa = new C;
    pa->f(); // vtable slot may be a thunk adjusting this for C::f
    delete pa; // UB unless destructor virtual in all relevant classes
}
```

上例中，调用 `pa->f()` 时 `this` 需要被调整以让 `C::f` 内部按预期访问成员；编译器通常为 vtable 提供相应的 thunk。

### 2、const与#define区别？

 `const`（或 `constexpr` / `enum` / `inline`）是类型安全的常量（由编译器处理），而 `#define` 是预处理器的文本替换（没有类型信息、没有作用域、容易出错）。

##### 2.1、根本区别（原理层面）

- `#define`
  - 由预处理器处理，是纯文本替换（token/textual substitution）。
  - 在编译的“真正编译阶段”之前就被替换掉，编译器看不到原始宏名。
  - 没有类型信息、没有作用域（全局生效直到 `#undef`），也无法被编译器类型检查。
  - 能做预处理器能力（条件编译 `#if`/`#ifdef`、字符串化 `#`、拼接 `##` 等）。
- `const`（和 `constexpr`、`enum`、`inline`）
  - 是语言层面的常量，有类型、遵守作用域和访问控制（namespace / class / block）。
  - 编译器能做类型检查、符号表记录（便于调试）、优化（比如折叠为常量或内联）。
  - `constexpr` 明确表示常量表达式（编译期求值），C++11+ 更推荐用 `constexpr`。

##### 2.2、主要差别

###### 2.2.1、类型安全

```cpp
#define PI_MACRO 3.14159
const double PI_CONST = 3.14159;
```

- `PI_MACRO` 没有类型，编译器不会对其进行类型检查。
- `PI_CONST` 有 `double` 类型，编译器能检查类型转换、传参匹配等。

###### 2.2.2、作用域与命名空间

```cpp
#define FLAG 1

namespace A {
    const int FLAG = 2;
}
```

- 宏 `FLAG` 在整个预处理范围有效，会污染全局命名；无法放进 `namespace` 中。
- `const` 可以放在 `namespace` 或 `class` 中，有明确作用域，不会导致命名冲突。

###### 2.2.3、调试与符号

- 宏在预处理后就消失，调试时看不到宏符号（调试器不能直接观察宏）。
- `const` 变量通常在符号表里（或被优化掉但能在源码层看到），调试器可显示其地址和数值（更友好）。

###### 2.2.4、文本替换的危险：优先级 / 括号问题 / 多次求值

错误示例（常见面试题）：

```cpp
#define SQR(x) x * x

int a = SQR(1 + 2); // 展开为: 1 + 2 * 1 + 2  => 优先级错误，结果不是 9
```

正确写法要用括号，但仍有多次求值问题：

```cpp
#define SQR2(x) ((x) * (x))
// 但 SQR2(i++) 会把 i++ 执行两次
```

替代（类型安全、无副作用）：

```cpp
template<typename T>
inline T sqr(T x) { return x * x; }
```

###### 2.2.5、多次评估（副作用）

```cpp
#define INC_AND_SQR(x) ((x) * (x))
int i = 1;
int y = INC_AND_SQR(i++); // i 被递增两次 —— 未预期副作用！
```

`const` / `inline` / `template` 版本不会有这种问题（参数只评估一次）。

###### 2.2.6、编译期使用 / 预处理器条件

- `#if` / `#ifdef`、`#error` 等只能看宏。你不能在预处理阶段用 `const` 值做 `#if` 判定（预处理器早于编译器运行）。

  ```c
  #define MAXN 100
  #if MAXN > 10
  // 成功
  #endif
  ```

- 如果你需要 **预处理器级别的开关或条件编译**，必须用宏或编译器 `-D` 参数。

###### 2.2.7、链接性 / 存储（C 与 C++ 的差异）

- 在 C++ 中，`const` 在 namespace/file scope 默认是内部链接（类似 `static`），即每个翻译单元会有自己的拷贝，除非写 `extern const`。

  ```cpp
  // A.h
  const int N = 10; // 每个 .cpp 有自己的 N，通常没符号导出
  ```

- 若你需要单一定义（跨 TU 共享），在 C++ 中可以写 `extern const int N;` 并在一个 .cpp 中定义 `const int N = 10;`。或者用 `inline constexpr`（C++17）在头文件中定义单一符号。

- 宏没有链接概念 —— 宏在预处理时直接替换。

> 在 **C**（不是 C++）中，`const` 不一定是编译期常量（语义上不等于 `#define` 的常量），常用于读取不可变内容，但不能用在需要编译期常量的上下文（尤其在 C89）。在 C 中，`enum { N = 10 };` 常被用作整型常量以替代 `#define`。

##### 2.3、宏能做但 `const` 不能做的事（宏的优势 / 合法用途）

- 条件编译（`#ifdef` / `#if`）：编译期包含/排除代码。

- 字符串化 / token 拼接：`#` 和 `##` 能把参数转为字符串或拼接标识符（常用于自动生成代码 / 记录 file/line）。

  ```c
  #define STR(x) #x
  #define CONCAT(a,b) a##b
  ```

- 实现 include guards / header guards：`#ifndef HEADER_H`。

- 在预处理阶段创建不同的代码路径（比如平台切换、启用调试宏等）。

所以宏并非完全不能用。

##### 2.4、`const` / `constexpr` / `enum` / `inline` 的替代与最佳实践

现代 C++ 风格建议优先使用语言特性替代宏。

- 整型常量：

  - C++：`constexpr int N = 42;`（C++11 起） 或 `enum { N = 42 };`（传统）
  - 这样可用于数组维度、模板参数（`constexpr` 更保险）

- 浮点与其它常量：`constexpr double PI = 3.1415926;`

- 函数宏 → inline/template 函数：

  ```cpp
  // 避免
  #define MAX(a,b) ((a) > (b) ? (a) : (b))
  // 使用
  template<typename T> constexpr const T& max(const T& a, const T& b) { return a > b ? a : b; }
  ```

- 字符串常量：`constexpr const char* name = "abc";` 或 `static constexpr std::string_view NAME = "abc";`（C++17）

##### 2.5、常见坑 & 面试/代码审查关注点

###### 坑 1 — 括号问题

```cpp
#define DOUBLE(x) x + x
int v = DOUBLE(1) * 3; // 1 + 1 * 3 => 4 (非预期 6)
```

解决：用 `inline`/`template` 或至少给宏加括号： `#define DOUBLE(x) ((x) + (x))`（仍有多次求值问题）。

###### 坑 2 — 多次求值/副作用

```cpp
#define INC_AND_SQR(x) ((x) * (x))
int i = 1;
int r = INC_AND_SQR(i++); // i incremented twice
```

替代：`inline`/template，或保证传入无副作用表达式。

###### 坑 3 — 宏污染命名空间

大写宏名易与库、第三方冲突。尽量把宏限制在必要范围并立即 `#undef`（如果合适）。

###### 坑 4 — 编译期常量差异（C vs C++）

在 C 中（尤其旧标准），`const int` 不总是编译期常量，不能用于 `switch` case 或文件作用域数组大小，推荐 `enum` 或 `#define`。

在 C++ 中，`const int`（有初始化）通常可以用作常量表达式，但更现代的做法是 `constexpr`。

##### 2.6、链接性 / 内存与优化细节

- `const` 变量可能占用存储（有地址），也可能被优化为直接内联到使用代码（无实际内存）。宏没有地址永远不会占存。
- `constexpr` 明确为编译期常量，有时不会产生存储；`inline constexpr`（C++17）允许在头文件中定义单一符号，避免 ODR 问题。

##### 2.7、示例：把宏改为现代 C++ 风格

```cpp
// 不推荐
#define BUFFER_SIZE 1024
#define SQR(x) ((x) * (x))

// 推荐
inline constexpr int BUFFER_SIZE = 1024;

template<typename T>
constexpr T sqr(T x) { return x * x; }
```

特殊场景（必须用宏）：

```cpp
// 包含保护 / 条件编译
#ifndef MYLIB_H
#define MYLIB_H
// ...
#endif

// 用于调试宏的标记粘贴/字符串化
#define STR(x) #x
#define LOG_VAR(x) std::cerr << #x " = " << (x) << std::endl
```

### 3、new的内存能用free释放吗？

**不能**——在标准 C++ 语义下，**用 `free` 去释放 `new` 分配的内存是未定义行为（UB）**。同理，用 `delete` 去释放由 `malloc` 分配的内存也是 UB。

##### 为什么不能混用？

1. 构造/析构的调用语义不同

   - `new T(...)` 做两件事：分配原始内存（调用 `operator new`）并构造对象（调用 `T` 的构造函数）。
   - `delete p` 做两件事：先调用 `p->~T()`（析构），再把内存交给运行时（调用 `operator delete`）。
   - `free` 只做一件事：直接释放内存，不会调用析构函数 → 如果对象有资源（RAII），会产生资源泄漏或未定义行为。

2. 分配/释放器不匹配

   - C++ 的 `operator new`/`operator delete` 可以被重载或实现为与 `malloc`/`free` 不同的 allocator（不同的元数据布局、内部链表、对齐策略等）。即便实现上 `operator new` 最后使用 `malloc`，也不能假设这是标准保证。
   - 因此 `free` 可能不知道如何正确处理 `operator new` 分配的内存（例如前置元数据、对齐、arena、TLS 分配器等），会破坏堆元数据导致崩溃或内存损坏。

3. 数组 `new[]` 的额外元信息

   `new T[n]` 可能会在分配块中保留元素个数信息（用于 `delete[]` 正确调用每个元素析构）。用 `free` 或 `delete`（单个）都不会正确调用所有析构或按正确方式释放元数据 → UB。

4. 对齐（alignment）问题

   C++17 引入对齐分配。`operator new` 可能返回对齐不同于 `malloc` 的地址或使用特殊路径；`free` 不一定以相同方式释放。混用可能无法满足对齐或释放语义。

5. 标准层面

   C++ 标准定义 `delete` 要配对 `new` 使用；`free` 要配对 `malloc` 使用。混合是未定义行为，不能依赖实现行为。

##### 3.1、典型错误示例与正确写法

错误示例（UB）

```cpp
int* p = new int(42);
free(p); // 未定义行为：不会调用析构（对 int 影响小），allocator 也可能不匹配
```

正确（C++ 对象由 new/delete 管理）

```cpp
int* p = new int(42);
delete p; // 调用析构（如果有），再释放内存
```

错误：malloc 分配但用 delete 释放（UB）

```cpp
int* q = (int*)malloc(sizeof(int));
*q = 10;
delete q; // UB：delete 假设内存来自 operator new、且应先调用析构
```

正确：malloc/free（仅裸内存/非对象或 POD）

```cpp
int* r = (int*)malloc(sizeof(int));
if (r) {
    *r = 10;
    free(r);
}
```

##### 3.2、special case：`malloc` + placement new + destructor + `free`（合法用法）

如果你先用 malloc 分配原始内存，然后 在该内存上用 placement new 构造对象，则销毁流程必须是：显式调用析构函数，然后 free 内存。

这是合法且常用的做法（例如自定义内存池）：

```cpp
#include <cstdlib>
#include <new>

void example() {
    void* raw = std::malloc(sizeof(MyClass));
    if (!raw) throw std::bad_alloc();
    MyClass* obj = new (raw) MyClass(args...); // placement new, 构造

    // 使用 obj...

    obj->~MyClass();      // 显式析构
    std::free(raw);       // 释放底层内存
}
```

注意：这里 `free` 是配对 `malloc` 的。你**不能**在 placement new 后用 `delete obj;`，因为 `delete` 会再次尝试调用 `operator delete`（非配对）。

##### 3.3、数组 `new[]` 与 `delete[]` 的注意点

用 `new T[n]` 必须用 `delete[] p`；不能用 `delete p` 也不能用 `free(p)`。否则可能只调用第一个元素析构或不调用析构并破坏内存管理：

```cpp
T* a = new T[10];
delete[] a; // √
delete a;   // × UB
free(a);    // × UB
```

##### 3.4、自定义 `operator new` / `operator delete` 的影响

类或全局可以重载 `operator new` 和 `operator delete`（或 `operator new[]` / `operator delete[]`）。例如：

```cpp
void* operator new(std::size_t sz) {
    // custom allocator logic (maybe use malloc or memory pool)
}
void operator delete(void* p) noexcept {
    // custom free logic
}
```

- 若 `operator new` 使用自定义 allocator（比如内存池），把 `free` 用于 `new` 分配的内存就很可能破坏该 allocator 的内部结构。
- 即使实现上 `operator new` 目前调用 `malloc`，也不能依赖这一点：标准允许实现以任意方式实现 `operator new`。

##### 3.5、对象是否有析构函数决定问题严重性

- 对于内置类型 `int`，`delete` 与 `free` 的差异主要体现在 allocator 匹配上（没有析构逻辑），因此有时混用在某实现下看起来“工作”，但仍是 UB。
- 对于用户类型（含析构逻辑），用 `free` 会跳过析构，导致资源泄露（文件/锁/内存）或不正确的状态清理。

### 4、stuct和class区别， class可以继承struct吗？

在 C++ 中 `struct` 与 `class` 表面上几乎一模一样——语法上它们都是类类型（class type）——唯一区别是默认的访问控制与默认的继承访问权限；其余特性（成员、方法、继承、多态、模板、嵌套、重载 new/delete 等）都通用。

##### 4.1、最核心的两个差别

1. 成员/基类的默认访问权限

   - `struct`：成员默认 `public`，基类默认 `public` 继承（如果不写 `:` 后面的访问说明）。

     ```cpp
     struct S {
         int x; // 等价于 public: int x;
     };
     ```

   - `class`：成员默认`private`，基类默认`private` 继承。

     ```cpp
     class C {
         int x; // 等价于 private: int x;
     };
     ```

2. 其他都相同：除上述默认可见性外，`class`/`struct` 在功能上没有差别 —— 都能有构造/析构、虚函数、继承、模板特化、静态成员、友元、嵌套类型、运算符重载等。

##### 4.2、代码示例对比（显示默认行为）

```cpp
struct S {
    int a;            // public
    void f() {}       // public
};

class K {
    int a;            // private
    void f() {}       // private
};
```

访问差异：

```cpp
S s;
s.a = 1;   // OK: public

K k;
// k.a = 1; // error: a 是 private
```

继承差异（默认继承权限）：

```cpp
struct Base { public: int v; };

struct DerivedStruct : Base { // 等同于 public Base
    // DerivedStruct 继承 Base 为 public
};

class DerivedClass : Base {  // 等同于 private Base
    // DerivedClass 继承 Base 为 private
};
```

因此：

```cpp
DerivedStruct ds;
ds.v = 1;          // OK: Base::v 在 DerivedStruct 对外仍为 public

DerivedClass dc;
// dc.v = 1;       // error: 因为继承是 private，Base::v 对外变为 private
```

##### 4.3、`class` 可以继承 `struct` 吗？`struct` 可以继承 `class` 吗？

**完全可以。**
 `struct` 和 `class` 都是类类型（class type），所以继承关系和语义与它们是哪一个关键字无关。关键点只在于你是否显式写了继承访问控制。

示例：

```cpp
struct S { public: int x; };
class C : public S {   // 注意这里显式写了 public
    // ok: C 继承 S，并且外部可以访问 x
};

class C2 : S {         // 默认 private 继承！（等同于 class C2 : private S）
    // S::x 对外为 private
};
```

以及：

```cpp
class B { protected: int n; };
struct D : B { // struct 默认 public 继承
    void foo() { n = 1; } // OK: protected 权限
};
```

##### 4.4、访问控制的“传播”规则

- 基类成员的原始访问修饰（public/protected/private）不会被改写；但继承的访问说明符（public/private/protected 继承）会影响基类成员对派生类外部的可见性。
   总结：

  - 若 `Derived : public Base`：`Base` 的 `public`→`public`，`protected`→`protected`（对外行为保留）。
  - 若 `Derived : private Base`（或默认的 `class` 继承）：`Base` 的 `public`/`protected` 成员都对 `Derived` 的外部视为 `private`（即不可访问）。

- 示例：

  ```cpp
  struct Base { public: int a; protected: int b; private: int c; };
  class Priv : Base { }; // private 继承
  // Priv p;
  // p.a; // error: a 在 Priv 中为 private（不可从外部访问）
  ```

##### 4.5、struct/class 在类型系统、布局、ABI 上一样吗？

- 对象内存布局（一般情况）相同：`struct`/`class` 都是类类型，内存布局由数据成员、对齐规则、虚表指针（若有虚函数）决定，与用 `struct` 还是 `class` 关键字无关。
- 例外差别不是由 `struct/class` 决定，而是由成员的访问、继承方式与虚函数/虚继承等影响（例如 private 继承并不会改变内存布局，但虚继承会增加 vbptr 等）。
- Empty Base Optimization (EBO) 同样适用于 struct/class。

##### 4.6、`struct` 能否有方法、构造函数、友元、静态成员等？

**当然能。** 在 C++ 中 `struct` 可做任何 `class` 能做的事（只是默认成员是 public）。

示例（完整类）：

```cpp
struct Point {
    int x, y;
    Point() : x(0), y(0) {}
    Point(int X, int Y) : x(X), y(Y) {}
    void move(int dx, int dy) { x += dx; y += dy; }
    static int count;
    ~Point() {}
};
```

### 5、空的class多大？

在 C++ 中，即使一个类里什么都没有，它的对象大小 **也不会是 0**。

例如：

```cpp
class Empty {};

int main() {
    std::cout << sizeof(Empty) << std::endl;
}
```

大多数编译器输出结果是：

```
1
```

##### 为什么不是 0？

C++ 标准规定：
**任何两个不同对象必须拥有不同的地址**。

如果 `sizeof(Empty)` 为 0，那么多个 `Empty` 对象就会占用相同的内存地址，无法区分：

```cpp
Empty a, b;
std::cout << &a << " " << &b << std::endl; 
// 如果大小为0，则 &a == &b，这会违反对象唯一性
```

因此，编译器会给 空类对象分配至少 1 个字节，保证每个对象在内存中有唯一地址。

##### 情况变化（继承、多态）

1. 空类的大小至少是 1 字节

   ```cpp
   class Empty {};
   sizeof(Empty) == 1;
   ```

2. 继承时的优化

   ```cpp
   class Base {};
   class Derived : public Base {};
   std::cout << sizeof(Derived); // 输出 1，而不是 2
   ```

   编译器允许消除基类的空对象开销。

3. 虚函数表指针（vptr）

   ```cpp
   class Empty {
       virtual void foo();
   };
   ```

   对象里会有一个 虚表指针（通常 8 字节 on 64-bit, 4 字节 on 32-bit），所以：

   - `sizeof(Empty)` = 8 (64 位系统)
   - `sizeof(Empty)` = 4 (32 位系统)

### 6、线程与进程区别概念？

##### 6.1、基本概念

- 进程（Process）
  - 是操作系统进行资源分配和调度的基本单位。
  - 一个进程拥有自己独立的地址空间（代码段、数据段、堆、栈等）。
  - 可以把进程理解为一个正在运行的程序。
  - 操作系统保证进程之间的内存隔离，互不干扰。
- 线程（Thread）
  - 是操作系统进行CPU 调度的基本单位。
  - 一个进程可以包含多个线程（多线程）。
  - 同一进程内的所有线程共享进程的地址空间和资源（堆、全局变量等），但每个线程有自己独立的栈和寄存器。
  - 线程是比进程更轻量级的执行单元。

##### 6.2、内存与资源分配

- 进程：
  - 拥有独立的地址空间。
  - 拥有独立的代码段、数据段、堆和栈。
  - 进程间的内存互相隔离，默认情况下一个进程不能直接访问另一个进程的内存。
- 线程：
  - 同一进程内的线程共享代码段、数据段和堆。
  - 每个线程有独立的栈空间（保存函数调用、局部变量）和寄存器（运行状态）。
  - 因为共享内存，线程间通信效率高，但也容易出现竞争条件、死锁问题。

##### 6.3、调度与切换

- 进程切换
  - 开销大（需要切换地址空间、寄存器、文件句柄等上下文）。
  - CPU 需要保存/恢复大量上下文。
- 线程切换
  - 开销小（只需切换栈和寄存器上下文，地址空间不变）。
  - 多线程切换比多进程切换快。

##### 6.4、通信方式

- 进程间通信（IPC）：
  - 管道（pipe）、消息队列、共享内存、信号量、socket 等。
  - 因为地址空间独立，通信需要操作系统内核参与，开销相对大。
- 线程间通信：
  - 因为线程共享进程的全局变量和堆，通信方式非常直接。
  - 常用同步手段：互斥锁（mutex）、读写锁（rwlock）、条件变量（condition_variable）、信号量（semaphore）、原子操作（atomic）等。
  - 更容易产生竞态条件，需要同步机制保护。

##### 6.5、性能与适用场景

- 进程：
  - 更稳定安全，隔离性强，一个进程崩溃不会直接影响另一个进程。
  - 适合多进程服务器（如 Nginx、Apache prefork 模型）。
  - 适合需要强隔离的场景（浏览器不同 tab、数据库实例）。
- 线程：
  - 创建/销毁/切换开销小，资源利用率高。
  - 更适合需要共享大量数据的任务（如并行计算、游戏引擎、多线程爬虫）。
  - 但一个线程崩溃可能导致整个进程挂掉（风险更大）。

##### 总结对比表

| 特性     | 进程 (Process)                          | 线程 (Thread)                            |
| -------- | --------------------------------------- | ---------------------------------------- |
| 地址空间 | 独立，互不干扰                          | 共享进程的地址空间                       |
| 栈空间   | 每个进程独立                            | 每个线程独立                             |
| 资源分配 | OS 分配的基本单位                       | CPU 调度的基本单位                       |
| 通信方式 | IPC（管道、共享内存、消息队列、socket） | 直接访问共享内存，需同步机制             |
| 切换开销 | 大（切换页表、上下文）                  | 小（仅切换寄存器和栈）                   |
| 稳定性   | 崩溃不影响其他进程                      | 崩溃可能导致整个进程挂掉                 |
| 适用场景 | 高安全隔离（浏览器、多进程服务器）      | 高性能共享任务（计算密集、多线程服务器） |

##### 一句话记忆：

- 进程是“工厂”，线程是“工人”。
- 工厂之间互相独立，工人同属一个工厂，共享机器（内存），但要避免互相抢机器。

### 7、线程间通信，进程间通信，用过的话详细说说？

##### 线程间通信（ITC）

###### 基本原理

线程属于同一进程，共享进程的地址空间（代码段、全局数据、堆等），所以通信的基础就是读写共享内存。但由于可能出现数据竞争，需要同步机制来保证正确性。

###### 常用方式

1. 互斥锁（mutex）

   - 保证同一时刻只有一个线程访问临界区。
   - 用于保护共享资源，防止数据竞争。
   - 缺点：可能导致死锁。

   ```cpp
   std::mutex mtx;
   int counter = 0;
   
   void work() {
       std::lock_guard<std::mutex> lock(mtx);
       counter++;
   }
   ```

2. 读写锁（shared_mutex / pthread_rwlock）

   - 多个线程可以同时读，写操作独占。
   - 适合读多写少场景。

3. 条件变量（condition_variable）

   - 用于线程之间的事件通知。
   - 常见场景：生产者-消费者模型。

   ```cpp
   std::mutex mtx;
   std::condition_variable cv;
   bool ready = false;
   
   void worker() {
       std::unique_lock<std::mutex> lock(mtx);
       cv.wait(lock, []{ return ready; });
       // 开始工作
   }
   ```

4. 原子操作（std::atomic）

   - 无锁方式保证操作的原子性。
   - 适合高频计数、标志位操作。
   - 比锁性能更好，但功能有限。

   ```cpp
   std::atomic<int> counter{0};
   counter.fetch_add(1);
   ```

5. 信号量（semaphore）

   - 控制共享资源的访问数量。
   - C++20 提供 `std::counting_semaphore`。

##### 进程间通信（IPC）

###### 基本原理

进程有独立的地址空间，不能直接访问彼此的数据。通信需要借助操作系统提供的 IPC 机制。

###### 常用方式

1. 管道（pipe / named pipe FIFO）

   - 单向、半双工通信。
   - 适合父子进程间简单数据传递。
   - 缺点：只能传字节流，不适合复杂数据。

   ```bash
   // Linux shell
   ps aux | grep nginx
   ```

2. 消息队列（System V / POSIX message queue）

   - 支持消息分块和优先级。
   - 比管道更灵活，但内核维护开销大。

3. 共享内存（shmget/shmat, mmap）

   - 最快的进程通信方式。
   - 多个进程映射到同一块内存区域。
   - 必须配合信号量/互斥锁保证同步。

   ```cpp
   // mmap 实现父子进程共享内存
   int *ptr = (int*)mmap(NULL, sizeof(int), PROT_READ|PROT_WRITE,
                         MAP_SHARED|MAP_ANONYMOUS, -1, 0);
   ```

4. 信号量（semaphore）

   - 用于进程间同步。
   - 常配合共享内存使用，保证互斥。

5. 信号（signal）

   - 一种“事件通知”机制。
   - 常用于进程控制（如 `SIGINT`, `SIGTERM`）。
   - 缺点：只能传简单信息，不适合复杂通信。

6. Socket（套接字）

   - 最通用的 IPC 方式。
   - 支持本机进程通信和跨机器通信（TCP/UDP）。
   - Web 服务器、分布式系统常用。

   ```cpp
   // 服务端: socket + bind + listen + accept
   // 客户端: socket + connect
   ```

##### 对比表

| 特性     | 线程通信（ITC）              | 进程通信（IPC）                     |
| -------- | ---------------------------- | ----------------------------------- |
| 地址空间 | 共享地址空间，通信直接读写   | 独立地址空间，必须通过内核机制      |
| 通信速度 | 快                           | 慢（通常需要内核态 ↔ 用户态切换）   |
| 常用方式 | 锁、条件变量、原子、信号量   | 管道、消息队列、共享内存、socket 等 |
| 隔离性   | 弱（共享内存，容易互相影响） | 强（独立内存，安全性更好）          |
| 应用场景 | 并发计算、任务调度           | 服务进程通信、跨机通信、模块解耦    |

### 8、C++编译过程说说，越详细越好？

##### 一、预处理

处理源代码中的预处理指令（以 `#` 开头），生成预处理后的源代码文件。

###### 主要任务：

1. 头文件包含（`#include`）  
   - 将 `#include <xxx>` 或 `#include "xxx"` 指令替换为对应头文件的内容。
   - 示例：
     ```c++
     #include <iostream>
     ```
     会被替换为 `<iostream>` 头文件的完整内容（如 `std::cout` 的声明）。

2. 宏定义与替换（`#define`）  
   - 将宏名替换为对应的值或代码片段。
   - 示例：
     ```c++
     #define PI 3.14159
     ```
     预处理后，所有 `PI` 会被替换为 `3.14159`。

3. 条件编译（`#ifdef`, `#ifndef`, `#if`, `#else`, `#endif`）  
   - 根据条件决定是否保留或删除代码块。
   - 示例：
     ```cpp
     #ifdef DEBUG
     std::cout << "Debug mode" << std::endl;
     #endif
     ```
     如果定义了 `DEBUG` 宏，则保留该代码块，否则删除。

4. 注释移除  
   
   删除源代码中的 `//` 或 `/* */` 注释。
   
5. 行号标记  
   
   插入行号信息，便于调试器定位源代码位置。

###### 实际命令：
```bash
g++ -E hello.cpp -o hello.i
```
`-E`：只执行预处理，生成 `.i` 文件（预处理后的源代码）。

##### 二、编译

将预处理后的源代码转换为汇编语言（与平台无关的中间代码），并进行优化。

###### 主要任务：

1. 词法分析
   
   将字符序列分解为 词法单元（Token），如关键字（`int`）、标识符（`main`）、运算符（`+`）等。
   
2. 语法分析
   
   - 构建 抽象语法树（AST），验证语法是否符合 C++ 规范。
   - 示例：解析 `int main() { ... }` 的结构。
   
3. 语义分析  
   - 检查类型匹配、变量作用域、函数签名等语义规则。
   - 示例：确保 `std::cout` 的参数类型正确。

4. 中间代码生成  
   
   生成与平台无关的中间表示（如 LLVM IR 或 GCC 的 GIMPLE）。
   
5. 代码优化  
   
   根据优化级别（如 `-O1`, `-O2`, `-O3`）对代码进行优化，如常量折叠、死代码消除、循环展开等。
   
6. 生成汇编代码  
   
   将优化后的中间代码转换为目标平台的汇编语言（`.s` 文件）。

###### 实际命令：
```bash
g++ -S hello.i -o hello.s
```
`-S`：生成汇编文件（`.s` 文件）。

##### 三、汇编

将汇编代码转换为机器码（目标文件），生成 **可重定位目标文件（Object File）**。

###### 主要任务：

1. 汇编翻译  
   
   将汇编指令（如 `mov`, `call`）转换为二进制机器码。
   
2. 符号表生成  
   
   记录全局变量、函数的地址（标记为 `UND` 表示未解析）。
   
3. 段划分  
   
   将代码划分为多个段（`.text`：代码段，`.data`：已初始化数据，`.bss`：未初始化数据）。

###### 实际命令：
```bash
g++ -c hello.s -o hello.o
```
`-c`：生成目标文件（`.o` 或 `.obj` 文件）。

##### 四、链接

将目标文件与库文件（如标准库、静态库、动态库）合并，生成最终的可执行文件。

###### 主要任务：

1. 符号解析  
   
   解决目标文件中引用的外部符号（如 `std::cout` 的实现）。
   
2. 地址重定位
   
   将目标文件中的相对地址转换为最终的绝对地址。
   
3. 库链接  
   
   链接静态库（`.a` 或 `.lib`）或动态库（`.so` 或 `.dll`）。
   
4. 生成可执行文件  
   
   输出可执行文件（如 `a.out` 或 `hello.exe`）。

###### 实际命令：
```bash
g++ hello.o -o hello
```
默认链接 C++ 标准库（如 `libstdc++`）。

##### 五、完整流程示例

###### 示例代码：`hello.cpp`
```cpp
#include <iostream>
using namespace std;

int main() {
    cout << "Hello, World!" << endl;
    return 0;
}
```

###### 手动执行编译流程：
```bash
# 1. 预处理
g++ -E hello.cpp -o hello.i

# 2. 编译
g++ -S hello.i -o hello.s

# 3. 汇编
g++ -c hello.s -o hello.o

# 4. 链接
g++ hello.o -o hello
```

###### 最终运行：
```bash
./hello
```

### 9、TCP三次握手与四次挥手，详细说说？

![TCP连接建立与断开](https://cdn.jsdelivr.net/gh/aqjsp/photos/image-20250908005737491.png)

##### 三次握手

1. Client → Server: `SYN`, `seq = x`
   - 客户端选择一个初始序列号 ISN = x，发送带 SYN 标志的段。客户端进入 `SYN_SENT` 状态。
   - SYN 包里通常携带 TCP 选项，如 MSS、Window Scale、SACK Permitted、Timestamps 等。
2. Server → Client: `SYN+ACK`, `seq = y`, `ack = x+1`
   - 服务器收到 SYN，记录半连接（TCB），选择自己的 ISN = y，发送 SYN+ACK，并进入 `SYN_RECEIVED` 状态。
   - 这里服务器已为该连接分配资源（TCB，socket 结构等）。
3. Client → Server: `ACK`, `seq = x+1`, `ack = y+1`
   - 客户端收到 SYN+ACK，发送 ACK，确认服务器 ISN。双方进入 `ESTABLISHED`。
   - 此时连接建立完成，应用层可以读写数据。

> 注意：ACK 的 `ack = y+1` 表示客户端确认已收到服务器的 SYN（服务器的 ISN 占用了一个序号）。TCP 的序号空间是按字节计数，SYN/FIN 各占用一个序号位。

###### 为什么需要三次而不是两次？

- 目的是 同步双方的初始序号（ISN）并确认对方真的存在，同时避免旧的延迟分组造成连接误判。
- 若仅两次（Client SYN → Server SYN-ACK；并把连接视为已建立就结束）会有问题：若客户端的 ACK 丢失，服务器可能在 `SYN_RECEIVED` 超时后删除半连接（造成半开），或者旧的延迟 SYN 或数据包可能被误解释为新连接的数据。三次握手确保双方都知道对方的 ISN 并且都收到了对方的“可达性确认”。

更直观的理由（防止“旧数据”被误用）：

如果没有 ACK，服务器可能把一个旧的（延迟到来的）数据包当作新连接的一部分，从而把旧连接的数据误当作当前连接的数据。三步确认后，双方都明确了序号空间区间。

###### 初始序列号（ISN）与安全

- ISN 应该看似随机以防止盲注入和劫持（历史上过于可预测的 ISN 导致 TCP 注入攻击）。
- SYN 包里还协商 TCP 选项（MSS、窗口缩放、SACK、时间戳等），这些选项在后续连接中生效。某些选项（如窗口缩放）只有在 SYN/SYN-ACK 中协商成功后方能使用。

##### 四次挥手

1. Client → Server: `FIN`, `seq = u`
   - 客户端表示自己已经没有更多数据要发送（半关闭自己的发送方向）。客户端进入 `FIN_WAIT_1`。
2. Server → Client: `ACK`, `ack = u+1`
   - 服务器确认收到了 FIN。此时服务器可能还有数据要发送。客户端进入 `FIN_WAIT_2`（等待服务器也发送 FIN）。
3. Server → Client: `FIN`, `seq = v`  （在服务器发送完其剩余数据后）
   - 服务器发送 FIN 表示它也不再发送数据（服务器进入 `CLOSE_WAIT` -> 当应用调用 close -> 发 FIN -> LAST_ACK）。如果服务器先发 FIN，则会先进入 `LAST_ACK` 等待 ACK。
4. Client → Server: `ACK`, `ack = v+1`
   - 客户端确认服务器的 FIN，然后客户端进入 `TIME_WAIT` 状态（为了可靠地处理最后的 ACK 丢失或延迟报文）。在 `TIME_WAIT` 期间，客户端仍能重传 ACK（如果服务器重传 FIN）。经过 `2*MSL`（Maximum Segment Lifetime）后，客户端转为 `CLOSED`。

> 注意：如果一端在收到 FIN 后立即调用 `close()` 并发送 FIN，另一端可能会很快进入 `LAST_ACK`、等待对方 ACK，产生不同的状态序列；此外也存在“同时关闭”的情形（双方同时发送 FIN），那时会有两次 FIN 和两次 ACK，但过程也符合四次挥手的语义。

###### 为什么是四次？

TCP 的连接在数据传输上是全双工的，客户端可以向服务器发送数据，服务器也可以向客户端发送数据。

关闭是对单方向的：一端发送 FIN 表示该方向的数据流结束，但对方仍可继续发送数据直到它也发送 FIN。

###### 为什么不能只用两次？

因为关闭是双向独立的：发送方发 FIN 后，接收方需要确认（ACK）；但接收方可能还有未发数据需要发完，然后再发 FIN。每个方向的 CLOSE 都需要自己的一次 FIN 和被 ACK。理论上如果两端都同时关闭并且 ACK/FIN 恰好合并，可能少于四次；但常规情况下是四次。

###### TIME_WAIT 的作用

为什么客户端在发送最后的 ACK 后进入 TIME_WAIT 并等待 2\*MSL？

1. 确保最后的 ACK 能到达对端。 如果服务器没有收到 ACK，会重传 FIN，客户端必须能够重传 ACK；如果客户端直接关闭并释放端口，若服务器重传 FIN，服务器可能收到 RST 或认为连接丢失，导致不一致。TIME_WAIT 允许客户端重传 ACK 应答。
2. 避免旧分组对新连接的影响。 2*MSL（最大报文寿命）保证网络中的老报文在完全消失之前不会被错误地关联到新建立的同一 <四元组>（src IP/port, dst IP/port）连接上。

谁进入 TIME_WAIT？ 

常见实现中是主动关闭方进入 TIME_WAIT（即发送最后 ACK 的那端）。这就是常见“一端等待 2*MSL”的原因。

### 10、在 O(n log n) 时间复杂度和常数级空间复杂度下，对链表进行排序。

##### 思路

使用自底向上的归并排序实现

1. 首先计算链表的长度
2. 从子链表长度为1开始，逐步合并长度为1,2,4,8...的子链表
3. 每次将两个相邻的有序子链表合并成一个更长的有序链表
4. 重复此过程直到整个链表有序

时间复杂度：O(n log n)

空间复杂度：O(1)

##### 参考代码（C++）

```c++
#include <iostream>

// 链表节点定义
struct ListNode {
    int val;
    ListNode *next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode *next) : val(x), next(next) {}
};

// 合并两个有序链表
ListNode* merge(ListNode* list1, ListNode* list2) {
    ListNode dummy(0);
    ListNode* cur = &dummy;
    
    // 按顺序合并两个有序链表
    while (list1 && list2) {
        if (list1->val <= list2->val) {
            cur->next = list1;
            list1 = list1->next;
        } else {
            cur->next = list2;
            list2 = list2->next;
        }
        cur = cur->next;
    }
    
    // 连接剩余节点
    cur->next = list1 ? list1 : list2;
    
    return dummy.next;
}

// 自底向上的归并排序
ListNode* sortList(ListNode* head) {
    // 空链表或只有一个节点，直接返回
    if (!head || !head->next) return head;
    
    // 计算链表长度
    int length = 0;
    ListNode* cur = head;
    while (cur) {
        length++;
        cur = cur->next;
    }
    
    // 创建虚拟头节点
    ListNode dummy(0);
    dummy.next = head;
    
    // 从子链表长度为1开始，逐步加倍
    for (int subLen = 1; subLen < length; subLen <<= 1) {
        ListNode* prev = &dummy;
        cur = dummy.next;
        
        // 每次处理两个长度为subLen的子链表
        while (cur) {
            // 第一个子链表的头节点
            ListNode* head1 = cur;
            
            // 获取第二个子链表的头节点
            ListNode* head2 = nullptr;
            // 将cur移动到第一个子链表的末尾
            for (int i = 1; i < subLen && cur->next; i++) {
                cur = cur->next;
            }
            
            // 如果cur->next为空，说明没有第二个子链表了
            if (cur->next) {
                head2 = cur->next;
                cur->next = nullptr; // 断开第一个子链表
                cur = head2;
                
                // 将cur移动到第二个子链表的末尾
                for (int i = 1; i < subLen && cur->next; i++) {
                    cur = cur->next;
                }
                
                // 断开第二个子链表
                ListNode* next = cur->next;
                cur->next = nullptr;
                cur = next;
            } else {
                cur = nullptr;
            }
            
            // 合并两个子链表
            ListNode* merged = merge(head1, head2);
            
            // 将合并后的链表连接到结果链表中
            prev->next = merged;
            
            // 更新prev到合并后链表的末尾
            while (prev->next) {
                prev = prev->next;
            }
        }
    }
    
    return dummy.next;
}

// 辅助函数：打印链表
void printList(ListNode* head) {
    while (head) {
        std::cout << head->val;
        if (head->next) std::cout << " -> ";
        head = head->next;
    }
    std::cout << std::endl;
}

// 测试函数
int main() {
    // 创建测试链表: 4 -> 2 -> 1 -> 3
    ListNode* head = new ListNode(4);
    head->next = new ListNode(2);
    head->next->next = new ListNode(1);
    head->next->next->next = new ListNode(3);
    
    std::cout << "排序前: ";
    printList(head);
    
    head = sortList(head);
    
    std::cout << "排序后: ";
    printList(head);
    
    // 创建测试链表: -1 -> 5 -> 3 -> 4 -> 0
    ListNode* head2 = new ListNode(-1);
    head2->next = new ListNode(5);
    head2->next->next = new ListNode(3);
    head2->next->next->next = new ListNode(4);
    head2->next->next->next->next = new ListNode(0);
    
    std::cout << "排序前: ";
    printList(head2);
    
    head2 = sortList(head2);
    
    std::cout << "排序后: ";
    printList(head2);
    
    return 0;
}
```

运行结果：

![运行结果](https://cdn.jsdelivr.net/gh/aqjsp/photos/image-20250908232028072.png)

### 11、找到两个单链表相交的起始节点。

##### 解法一：双指针法（推荐）

###### 思路：

1. 使用两个指针分别指向两个链表的头节点
2. 当指针到达链表末尾时，将其重定向到另一个链表的头部
3. 如果两个链表相交，两个指针会在相交节点相遇
4. 如果两个链表不相交，两个指针会在nullptr相遇

##### 解法二：哈希集合法

###### 思路：

1. 遍历第一个链表，将所有节点存入哈希集合
2. 遍历第二个链表，检查节点是否在哈希集合中
3. 第一个在哈希集合中的节点即为相交节点

##### 解法三：长度差法

###### 思路：

1. 计算两个链表的长度
2. 让较长链表的指针先走长度差步
3. 两个指针同时移动，直到相遇

时间复杂度：O(m+n)

空间复杂度：双指针法O(1)，哈希集合法O(m)

##### 参考代码（C++）

```c++
#include <iostream>
#include <unordered_set>

// 链表节点定义
struct ListNode {
    int val;
    ListNode *next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode *next) : val(x), next(next) {}
};

// 解法一：双指针法（最优解）
ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
    if (!headA || !headB) return nullptr;
    
    ListNode *pA = headA, *pB = headB;
    
    // 当两个指针不相等时继续循环
    while (pA != pB) {
        // 当pA到达链表末尾时，重定向到headB
        pA = pA ? pA->next : headB;
        // 当pB到达链表末尾时，重定向到headA
        pB = pB ? pB->next : headA;
    }
    
    // 如果相交，返回相交节点；如果不相交，返回nullptr
    return pA;
}

// 解法二：哈希集合法
ListNode *getIntersectionNodeHash(ListNode *headA, ListNode *headB) {
    if (!headA || !headB) return nullptr;
    
    std::unordered_set<ListNode*> nodeSet;
    
    // 将链表A的所有节点存入哈希集合
    ListNode* cur = headA;
    while (cur) {
        nodeSet.insert(cur);
        cur = cur->next;
    }
    
    // 遍历链表B，查找第一个在哈希集合中的节点
    cur = headB;
    while (cur) {
        if (nodeSet.find(cur) != nodeSet.end()) {
            return cur;  // 找到相交节点
        }
        cur = cur->next;
    }
    
    return nullptr;  // 没有相交节点
}

// 解法三：长度差法
ListNode *getIntersectionNodeLength(ListNode *headA, ListNode *headB) {
    if (!headA || !headB) return nullptr;
    
    // 计算两个链表的长度
    int lenA = 0, lenB = 0;
    ListNode* curA = headA;
    ListNode* curB = headB;
    
    while (curA) {
        lenA++;
        curA = curA->next;
    }
    
    while (curB) {
        lenB++;
        curB = curB->next;
    }
    
    // 重新指向头节点
    curA = headA;
    curB = headB;
    
    // 让较长链表的指针先走长度差步
    if (lenA > lenB) {
        for (int i = 0; i < lenA - lenB; i++) {
            curA = curA->next;
        }
    } else {
        for (int i = 0; i < lenB - lenA; i++) {
            curB = curB->next;
        }
    }
    
    // 两个指针同时移动，直到相遇
    while (curA && curB) {
        if (curA == curB) {
            return curA;  // 找到相交节点
        }
        curA = curA->next;
        curB = curB->next;
    }
    
    return nullptr;  // 没有相交节点
}

// 辅助函数：打印链表
void printList(ListNode* head) {
    while (head) {
        std::cout << head->val;
        if (head->next) std::cout << " -> ";
        head = head->next;
    }
    std::cout << std::endl;
}

// 测试函数
int main() {
    // 创建相交链表:
    // ListA: 4 -> 1 -> 8 -> 4 -> 5
    // ListB: 5 -> 0 -> 1 -> 8 -> 4 -> 5
    // 相交节点为8
    ListNode* intersectNode = new ListNode(8);
    intersectNode->next = new ListNode(4);
    intersectNode->next->next = new ListNode(5);
    
    ListNode* headA = new ListNode(4);
    headA->next = new ListNode(1);
    headA->next->next = intersectNode;
    
    ListNode* headB = new ListNode(5);
    headB->next = new ListNode(0);
    headB->next->next = new ListNode(1);
    headB->next->next->next = intersectNode;
    
    std::cout << "链表A: ";
    printList(headA);
    std::cout << "链表B: ";
    printList(headB);
    
    ListNode* result = getIntersectionNode(headA, headB);
    if (result) {
        std::cout << "相交节点值: " << result->val << std::endl;
    } else {
        std::cout << "无相交节点" << std::endl;
    }
    
    // 测试不相交的链表
    ListNode* headC = new ListNode(1);
    headC->next = new ListNode(2);
    headC->next->next = new ListNode(3);
    
    ListNode* headD = new ListNode(4);
    headD->next = new ListNode(5);
    headD->next->next = new ListNode(6);
    
    std::cout << "\n链表C: ";
    printList(headC);
    std::cout << "链表D: ";
    printList(headD);
    
    result = getIntersectionNode(headC, headD);
    if (result) {
        std::cout << "相交节点值: " << result->val << std::endl;
    } else {
        std::cout << "无相交节点" << std::endl;
    }
    
    return 0;
}
```

运行结果：

![运行结果](https://cdn.jsdelivr.net/gh/aqjsp/photos/image-20250908232823693.png)

### 12、给定一个字符串 s，将 s 分割成一些子串，使每个子串都是回文串。返回 s 所有可能的分割方案。

##### 思路

使用回溯算法（Backtracking）+ 动态规划预处理

###### 方法一

回溯算法 + 中心扩展预处理

1. 预处理：使用中心扩展法找出所有可能的回文子串

2. 回溯：从字符串开始位置进行深度优先搜索，尝试所有可能的分割点

3. 剪枝：只在当前子串是回文串时才继续递归

###### 方法二

回溯算法 + 动态规划预处理
1. 预处理：使用动态规划找出所有可能的回文子串
2. 回溯：从字符串开始位置进行深度优先搜索

时间复杂度：O(N * 2^N)，其中N为字符串长度

空间复杂度：O(N^2)，用于存储回文信息

##### 参考代码（C++）

```c++
#include <iostream>
#include <vector>
#include <string>

// 方法一：回溯算法 + 中心扩展预处理
class Solution {
public:
    std::vector<std::vector<std::string>> partition(std::string s) {
        int n = s.length();
        // 预处理所有回文子串
        std::vector<std::vector<bool>> isPalindrome(n, std::vector<bool>(n, false));
        
        // 中心扩展法预处理回文串
        for (int i = 0; i < n; i++) {
            // 奇数长度回文串（以i为中心）
            expandAroundCenter(s, i, i, isPalindrome);
            // 偶数长度回文串（以i和i+1为中心）
            expandAroundCenter(s, i, i + 1, isPalindrome);
        }
        
        std::vector<std::vector<std::string>> result;
        std::vector<std::string> currentPartition;
        backtrack(s, 0, isPalindrome, currentPartition, result);
        return result;
    }
    
private:
    // 中心扩展法标记回文串
    void expandAroundCenter(const std::string& s, int left, int right, 
                           std::vector<std::vector<bool>>& isPalindrome) {
        while (left >= 0 && right < s.length() && s[left] == s[right]) {
            isPalindrome[left][right] = true;
            left--;
            right++;
        }
    }
    
    // 回溯算法
    void backtrack(const std::string& s, int start,
                   const std::vector<std::vector<bool>>& isPalindrome,
                   std::vector<std::string>& currentPartition,
                   std::vector<std::vector<std::string>>& result) {
        // 递归终止条件：已处理完整个字符串
        if (start == s.length()) {
            result.push_back(currentPartition);
            return;
        }
        
        // 尝试所有可能的分割点
        for (int end = start; end < s.length(); end++) {
            // 只有当前子串是回文串时才继续递归
            if (isPalindrome[start][end]) {
                // 选择：将当前子串加入分割方案
                currentPartition.push_back(s.substr(start, end - start + 1));
                // 递归：处理剩余部分
                backtrack(s, end + 1, isPalindrome, currentPartition, result);
                // 撤销选择：回溯
                currentPartition.pop_back();
            }
        }
    }
};

// 方法二：回溯算法 + 动态规划预处理
class Solution2 {
public:
    std::vector<std::vector<std::string>> partition(std::string s) {
        int n = s.length();
        // 动态规划预处理回文串
        // dp[i][j] 表示 s[i...j] 是否为回文串
        std::vector<std::vector<bool>> dp(n, std::vector<bool>(n, false));
        
        // 初始化：单个字符都是回文串
        for (int i = 0; i < n; i++) {
            dp[i][i] = true;
        }
        
        // 填充dp表
        for (int len = 2; len <= n; len++) {  // 子串长度从2开始
            for (int i = 0; i <= n - len; i++) {  // 起始位置
                int j = i + len - 1;  // 结束位置
                if (s[i] == s[j]) {
                    if (len == 2 || dp[i + 1][j - 1]) {
                        dp[i][j] = true;
                    }
                }
            }
        }
        
        std::vector<std::vector<std::string>> result;
        std::vector<std::string> currentPartition;
        backtrack(s, 0, dp, currentPartition, result);
        return result;
    }
    
private:
    // 回溯算法
    void backtrack(const std::string& s, int start,
                   const std::vector<std::vector<bool>>& dp,
                   std::vector<std::string>& currentPartition,
                   std::vector<std::vector<std::string>>& result) {
        // 递归终止条件：已处理完整个字符串
        if (start == s.length()) {
            result.push_back(currentPartition);
            return;
        }
        
        // 尝试所有可能的分割点
        for (int end = start; end < s.length(); end++) {
            // 只有当前子串是回文串时才继续递归
            if (dp[start][end]) {
                // 选择：将当前子串加入分割方案
                currentPartition.push_back(s.substr(start, end - start + 1));
                // 递归：处理剩余部分
                backtrack(s, end + 1, dp, currentPartition, result);
                // 撤销选择：回溯
                currentPartition.pop_back();
            }
        }
    }
};

// 辅助函数：打印所有分割方案
void printPartitions(const std::vector<std::vector<std::string>>& partitions) {
    std::cout << "[";
    for (size_t i = 0; i < partitions.size(); i++) {
        std::cout << "[";
        for (size_t j = 0; j < partitions[i].size(); j++) {
            std::cout << "\"" << partitions[i][j] << "\"";
            if (j < partitions[i].size() - 1) std::cout << ",";
        }
        std::cout << "]";
        if (i < partitions.size() - 1) std::cout << ",";
    }
    std::cout << "]" << std::endl;
}

// 测试函数
int main() {
    Solution solution;
    
    // 测试用例1: "aab"
    std::string s1 = "aab";
    std::cout << "字符串 \"" << s1 << "\" 的所有回文分割方案:" << std::endl;
    std::vector<std::vector<std::string>> result1 = solution.partition(s1);
    printPartitions(result1);
    
    // 测试用例2: "a"
    std::string s2 = "a";
    std::cout << "\n字符串 \"" << s2 << "\" 的所有回文分割方案:" << std::endl;
    std::vector<std::vector<std::string>> result2 = solution.partition(s2);
    printPartitions(result2);
    
    // 测试用例3: "abcba"
    std::string s3 = "abcba";
    std::cout << "\n字符串 \"" << s3 << "\" 的所有回文分割方案:" << std::endl;
    std::vector<std::vector<std::string>> result3 = solution.partition(s3);
    printPartitions(result3);
    
    return 0;
}
```

运行结果：

![运行结果](https://cdn.jsdelivr.net/gh/aqjsp/photos/image-20250908233534358.png)

