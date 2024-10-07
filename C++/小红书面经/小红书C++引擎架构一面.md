# 小红书C++引擎架构一面

> 来源：https://www.nowcoder.com/feed/main/detail/42767a0a3cd241d48266b3eb5d9405d4

### 1、自我介绍

略

### 2、项目

略

### 3、c++ 多态，如何实现的，虚表、虚表指针存储位置？

在 C++ 中，多态（Polymorphism）是通过虚函数（virtual functions）和虚表（vtable，虚拟表）来实现的。多态允许通过基类的指针或引用来调用派生类的方法，从而实现运行时的动态绑定。

#### 1. 虚函数（Virtual Functions）

虚函数是 C++ 中实现多态的关键。在基类中声明虚函数，派生类可以重写（override）这些虚函数。通过基类的指针或引用来调用虚函数时，实际调用的是派生类中重写的方法。

```c++
class Base {
public:
    virtual void show() {
        cout << "Base class show function" << endl;
    }
};

class Derived : public Base {
public:
    void show() override {
        cout << "Derived class show function" << endl;
    }
};
```

#### 2. 虚表（VTable）

虚表是一个函数指针数组，每个类都有一个虚表，虚表中存储了该类中所有虚函数的地址。虚表在编译时生成，运行时通过虚表指针访问。

#### 3. 虚表指针（VTable Pointer）

每个对象在内存中都有一个隐式的虚表指针（vptr），指向该对象所属类的虚表。虚表指针通常存储在对象的最前面，但具体位置取决于编译器的实现。

#### 4. 多态的实现步骤

1. 声明虚函数：
   - 在基类中声明虚函数。
   - 派生类中可以重写这些虚函数。
2. 生成虚表：
   - 编译器为每个包含虚函数的类生成一个虚表。
   - 虚表中存储了该类中所有虚函数的地址。
3. 初始化虚表指针：编译器在构造对象时，自动初始化对象的虚表指针，使其指向该类的虚表。
4. 调用虚函数：通过基类的指针或引用来调用虚函数时，实际上通过虚表指针访问虚表，找到对应的函数地址，然后调用该函数。

#### 5. 内存布局

##### 基类 `Base`

```c++
class Base {
public:
    virtual void show() {
        cout << "Base class show function" << endl;
    }
    int baseData;
};
```

##### 派生类 `Derived`

```c++
class Derived : public Base {
public:
    void show() override {
        cout << "Derived class show function" << endl;
    }
    int derivedData;
};
```

##### 内存布局

- ###### Base 对象的内存布局：

  - vptr（虚表指针）
  - baseData

- ###### Derived 对象的内存布局：

  - vptr（虚表指针）
  - baseData
  - derivedData

#### 6. 虚表和虚表指针的存储位置

- 虚表：
  - 虚表是一个静态的数据结构，通常存储在程序的只读数据段中。
  - 每个类有一个虚表，虚表中存储了该类中所有虚函数的地址。
- 虚表指针：
  - 虚表指针（vptr）是每个对象的一部分，通常存储在对象的最前面。
  - 构造对象时，编译器会自动初始化虚表指针，使其指向该类的虚表。

##### 虚表的生成

编译器生成虚表：编译器在编译时为每个包含虚函数的类生成一个虚表。虚表是一个函数指针数组，每个虚函数对应一个函数指针。

- 虚表的结构：

```c++
struct VTable {
    void (*show)(Base*);
};
```

- 对于`Base`类，虚表如下：

```c++
VTable vtable_Base = { &Base::show };
```

- 对于`Derived`类，虚表如下：

```c++
VTable vtable_Derived = { &Derived::show };
```

##### 虚表指针的初始化

构造函数：编译器在构造对象时，自动初始化对象的虚表指针，使其指向该类的虚表。

```c++
class Base {
public:
    void* vptr;
    int baseData;

    Base() {
        vptr = &vtable_Base;  // 初始化虚表指针
    }

    virtual void show() {
        cout << "Base class show function" << endl;
    }
};

class Derived : public Base {
public:
    int derivedData;

    Derived() {
        vptr = &vtable_Derived;  // 初始化虚表指针
    }

    void show() override {
        cout << "Derived class show function" << endl;
    }
};
```

##### 调用虚函数

通过虚表指针调用虚函数：当通过基类的指针或引用来调用虚函数时，实际上通过虚表指针访问虚表，找到对应的函数地址，然后调用该函数。

```c++
void callShow(Base* ptr) {
    // 通过虚表指针访问虚表，调用虚函数
    static_cast<void (*)(Base*)>(ptr->vptr[0])(ptr);
}

int main() {
    Base b;
    Derived d;

    Base* basePtr = &b;
    Base* derivedPtr = &d;

    callShow(basePtr);         // 输出: Base class show function
    callShow(derivedPtr);      // 输出: Derived class show function

    return 0;
}
```

### 4、explicit 关键字？

C++ 中，`explicit` 关键字用于防止隐式类型转换，通常应用于单参数的构造函数和转换运算符。

#### 1. 单参数构造函数

默认情况下，如果一个类有一个单参数的构造函数，编译器会认为这是一个隐式转换构造函数，允许从该参数类型隐式转换为类的对象。这种隐式转换有时会导致意外的行为。

##### 未使用 `explicit`

```c++
class MyClass {
public:
    MyClass(int value) : m_value(value) {}

private:
    int m_value;
};

void printMyClass(const MyClass& obj) {
    std::cout << "MyClass object" << std::endl;
}

int main() {
    MyClass obj1(10);  // 显式构造
    printMyClass(obj1);  // 正常调用

    int x = 20;
    printMyClass(x);  // 隐式转换，x 被转换为 MyClass 对象
    return 0;
}
```

`printMyClass` 函数接受一个 `MyClass` 引用，但传入了一个 `int` 类型的变量 `x`。由于 `MyClass` 有一个单参数的构造函数，编译器会隐式地将 `x` 转换为 `MyClass` 对象，然后传递给 `printMyClass` 函数。

##### 使用 `explicit`

为了防止这种隐式转换，可以使用 `explicit` 关键字修饰单参数构造函数。

```c++
class MyClass {
public:
    explicit MyClass(int value) : m_value(value) {}

private:
    int m_value;
};

void printMyClass(const MyClass& obj) {
    std::cout << "MyClass object" << std::endl;
}

int main() {
    MyClass obj1(10);  // 显式构造
    printMyClass(obj1);  // 正常调用

    int x = 20;
    // printMyClass(x);  // 编译错误，无法隐式转换
    printMyClass(MyClass(x));  // 显式构造
    return 0;
}
```

`MyClass` 的构造函数被标记为 `explicit`，因此不能隐式地将 `int` 类型的 `x` 转换为 `MyClass` 对象。如果尝试这样做，编译器会报错。要传递 `x`，必须显式地构造一个 `MyClass` 对象。

#### 2. 转换运算符

转换运算符允许类对象隐式转换为其他类型。同样，为了防止意外的隐式转换，可以使用 `explicit` 关键字修饰转换运算符。

##### 未使用 `explicit`

```c++
class MyInt {
public:
    MyInt(int value) : m_value(value) {}

    operator int() const {
        return m_value;
    }

private:
    int m_value;
};

void printInt(int value) {
    std::cout << "Value: " << value << std::endl;
}

int main() {
    MyInt myInt(10);
    printInt(myInt);  // 隐式转换，myInt 被转换为 int
    return 0;
}
```

`MyInt` 类有一个转换运算符，允许 `MyInt` 对象隐式转换为 `int` 类型。因此，可以直接将 `MyInt` 对象传递给 `printInt` 函数。

##### 使用 `explicit`

为了防止这种隐式转换，可以使用 `explicit` 关键字修饰转换运算符。

```c++
class MyInt {
public:
    MyInt(int value) : m_value(value) {}

    explicit operator int() const {
        return m_value;
    }

private:
    int m_value;
};

void printInt(int value) {
    std::cout << "Value: " << value << std::endl;
}

int main() {
    MyInt myInt(10);
    // printInt(myInt);  // 编译错误，无法隐式转换
    printInt(static_cast<int>(myInt));  // 显式转换
    return 0;
}
```

`MyInt` 的转换运算符被标记为 `explicit`，因此不能隐式地将 `MyInt` 对象转换为 `int` 类型。如果尝试这样做，编译器会报错。要进行转换，必须使用 `static_cast` 进行显式转换。

### 5、unique_ptr、shared_ptr、weak_ptr的原理，有没有线程安全问题，weak_ptr的解决了什么问题？可以用裸指针吗？会有什么问题？

#### 1. `std::unique_ptr`

##### 原理

`std::unique_ptr` 是一种独占所有权的智能指针，确保在同一时间只有一个 `std::unique_ptr` 指向某个对象。它通过 RAII（Resource Acquisition Is Initialization）原则管理资源，当 `std::unique_ptr` 被销毁时，它所管理的对象也会被自动删除。

- 独占所有权：`std::unique_ptr` 不允许复制，但支持移动语义。这意味着所有权可以通过移动操作从一个 `std::unique_ptr` 转移到另一个 `std::unique_ptr`。
- 自动资源管理：当 `std::unique_ptr` 离开作用域时，它会自动调用删除器（默认是 `std::default_delete`）来释放所管理的对象。

```c++
#include <memory>
#include <iostream>

void example() {
    std::unique_ptr<int> ptr(new int(10));  // 创建 unique_ptr
    std::cout << *ptr << std::endl;         // 使用指针
    // ptr 会在函数结束时自动释放资源
}
```

##### 线程安全

- 对象管理：`std::unique_ptr` 本身不是线程安全的，即多个线程同时访问同一个 `std::unique_ptr` 对象是不安全的。
- 资源访问：如果多个线程需要访问 `std::unique_ptr` 所管理的对象，需要使用互斥锁等同步机制来确保线程安全。

#### 2. `std::shared_ptr`

##### 原理

`std::shared_ptr` 是一种共享所有权的智能指针，允许多个 `std::shared_ptr` 同时指向同一个对象。它通过引用计数来管理对象的生命周期，当最后一个 `std::shared_ptr` 被销毁时，所管理的对象才会被删除。

- 共享所有权：多个 `std::shared_ptr` 可以指向同一个对象，每个 `std::shared_ptr` 都会增加引用计数。
- 自动资源管理：当引用计数降为零时，所管理的对象会被自动删除。

```c++
#include <memory>
#include <iostream>

void example() {
    std::shared_ptr<int> ptr1(new int(10));  // 创建 shared_ptr
    std::shared_ptr<int> ptr2 = ptr1;        // 共享所有权
    std::cout << *ptr1 << std::endl;         // 使用指针
    std::cout << *ptr2 << std::endl;         // 使用指针
    // ptr1 和 ptr2 会在函数结束时自动释放资源
}
```

##### 线程安全

- 对象管理：`std::shared_ptr` 的引用计数是线程安全的，即多个线程可以安全地增加或减少引用计数。
- 资源访问：如果多个线程需要访问 `std::shared_ptr` 所管理的对象，仍然需要使用互斥锁等同步机制来确保线程安全。

#### 3. `std::weak_ptr`

##### 原理

`std::weak_ptr` 是一种不增加引用计数的智能指针，主要用于解决 `std::shared_ptr` 之间的循环引用问题。`std::weak_ptr` 不能直接访问所管理的对象，需要通过 `lock` 方法转换为 `std::shared_ptr`。

- 弱引用：`std::weak_ptr` 不增加引用计数，因此不会影响对象的生命周期。
- 解决循环引用：当两个 `std::shared_ptr` 互相引用时，引用计数永远不会降为零，导致内存泄漏。`std::weak_ptr` 可以打破这种循环引用。

```c++
#include <memory>
#include <iostream>

void example() {
    std::shared_ptr<int> shared = std::make_shared<int>(10);
    std::weak_ptr<int> weak = shared;  // 创建 weak_ptr

    if (auto locked = weak.lock()) {  // 转换为 shared_ptr
        std::cout << *locked << std::endl;  // 使用指针
    } else {
        std::cout << "Object has been deleted" << std::endl;
    }
}
```

##### 线程安全

- 对象管理：`std::weak_ptr` 的引用计数是线程安全的，即多个线程可以安全地增加或减少引用计数。
- 资源访问：`std::weak_ptr` 本身不能直接访问所管理的对象，需要通过 `lock` 方法转换为 `std::shared_ptr`。转换后的 `std::shared_ptr` 需要使用互斥锁等同步机制来确保线程安全。

#### 4. 裸指针的问题

虽然 C++ 支持使用裸指针（即普通的指针），但在现代 C++ 编程中，推荐使用智能指针来管理动态内存。

使用裸指针的一些潜在问题：

- 内存泄漏：忘记释放分配的内存会导致内存泄漏。
- 悬挂指针：释放内存后继续使用指针会导致未定义行为。
- 双重删除：多次删除同一个指针会导致未定义行为。
- 资源管理复杂：手动管理资源容易出错，特别是在异常处理和多线程环境中。

```c++
void example() {
    int* ptr = new int(10);  // 分配内存
    std::cout << *ptr << std::endl;  // 使用指针
    delete ptr;  // 手动释放内存
    // 忘记释放内存或多次释放内存都可能导致问题
}
```

### 6、介绍B树和B+树？

#### 1. B树（B-tree）

##### 原理

B树是一种自平衡的多路搜索树，能够保持数据有序。B树的设计目的是为了在处理大量数据时减少磁盘I/O操作，从而提高数据访问效率。

B树的主要特点包括：

- 多路搜索：每个节点可以有多个子节点，通常用阶数（order）表示一个节点最多可以有多少个子节点。
- 平衡性：B树是自平衡的，即所有叶子节点都位于同一层，确保了树的高度较低，从而减少了查找、插入和删除操作的时间复杂度。
- 节点结构：每个节点包含多个关键字和子节点指针。关键字按顺序排列，子节点指针指向关键字范围内的子树。

##### 定义

对于一个M阶的B树，必须满足以下条件：

- 每个节点最多有M个子节点。
- 每个非叶子节点（除了根节点）至少有⌈M/2⌉个子节点。
- 如果根节点不是叶子节点，则根节点至少有两个子节点。
- 具有N个子节点的非叶子节点包含N-1个关键字。
- 所有叶子节点都位于同一层，且不包含任何关键字信息。

##### 优点

- 高效的数据访问：由于树的高度较低，查找、插入和删除操作的时间复杂度为O(logₘn)，其中m是树的阶数，n是节点数。
- 减少I/O操作：B树的节点可以存储多个关键字，减少了磁盘I/O操作，特别适合于磁盘存储系统。

##### 缺点

- 节点结构复杂：每个节点需要存储多个关键字和子节点指针，导致节点结构较为复杂。
- 空间利用率：由于非叶子节点也需要存储关键字，空间利用率相对较低。

#### 2. B+树（B+-tree）

##### 原理

B+树是B树的一种变种，主要优化了顺序访问和范围查询的性能。

B+树的主要特点包括：

- 所有数据存储在叶子节点：非叶子节点仅用于索引，不存储实际数据。所有数据都存储在叶子节点上，这使得B+树在顺序访问和范围查询方面具有更高的效率。
- 叶子节点链表：所有叶子节点通过链表连接在一起，便于顺序访问。
- 平衡性：B+树也是自平衡的，所有叶子节点都位于同一层，确保了树的高度较低。

##### 定义

对于一个M阶的B+树，必须满足以下条件：

- 每个节点最多有M个子节点。
- 非叶子根节点至少有两棵子树，其他每个非叶子节点至少有⌈M/2⌉棵子树。
- 结点的子树个数与关键字个数相等。
- 所有叶结点包含全部关键字及指向相应记录的指针，且按大小顺序排列，并通过链表连接。
- 所有非叶子节点只包含其子节点的最大关键字和指向子节点的指针。

##### 优点

- 高效的数据访问：由于所有数据都存储在叶子节点上，查找、插入和删除操作的时间复杂度同样为O(logₘn)。
- 顺序访问和范围查询：叶子节点通过链表连接，便于顺序访问和范围查询。
- 空间利用率：非叶子节点不存储数据，空间利用率较高，可以容纳更多的关键字信息，提高了查询的并行度。

##### 缺点

- 节点结构复杂：虽然非叶子节点不存储数据，但仍然需要存储多个关键字和子节点指针，节点结构较为复杂。
- 插入和删除操作复杂：由于需要维护叶子节点的链表结构，插入和删除操作相对复杂。

#### 应用场景

- B树：适用于需要高效查找、插入和删除操作的场景，尤其是在磁盘存储系统中，如文件系统和数据库索引。
- B+树：适用于需要高效顺序访问和范围查询的场景，特别适合于数据库系统中的索引结构，如MySQL的InnoDB存储引擎。

### 7、介绍unordered_map、map，区别，应用场景？

#### 1. 基本概念

- **`map`**: `map` 是 C++ 标准模板库（STL）中的一个有序关联容器，用于存储键值对，其中每个键都是唯一的。`map` 的内部实现基于红黑树，这是一种自平衡二叉搜索树，这意味着 `map` 中的元素总是保持排序状态。
- **`unordered_map`**: `unordered_map` 同样是 C++ STL 中的一个关联容器，但它是一个无序的容器，内部实现基于哈希表。这意味着 `unordered_map` 中的元素没有固定的顺序，而是根据哈希函数的输出来确定存储位置。

#### 2. 特点

- **`map`**
  - 有序性：`map`中的元素按照键值自然排序（或者自定义排序规则），这对于需要按键值顺序处理数据的应用非常有用。
  - 时间复杂度：插入、删除和查找操作的时间复杂度均为 O(log n)，其中 n 是容器中元素的数量。这是因为`map`的内部实现依赖于红黑树，这种数据结构保证了这些操作的对数时间复杂度。
  - 空间使用：由于`map`需要在每个节点中存储额外的信息（如颜色、父节点指针等），因此相比于`unordered_map`，`map`的空间开销更大。
- **`unordered_map`**
  - 无序性：`unordered_map`中的元素没有特定的顺序，这取决于哈希函数和哈希表的状态。
  - 时间复杂度: 在理想情况下，`unordered_map`的查找、插入和删除操作的时间复杂度接近 O(1)，但在哈希冲突严重时，最坏情况下的时间复杂度可能退化为 O(n)。
  - 空间使用: 由于 `unordered_map`使用哈希表实现，它在大多数情况下比`map`更节省空间，特别是在存储大量数据时。

#### 3. 应用场景

- **`map`**
  - 有序数据处理: 当需要保持数据的有序性时，`map`是一个合适的选择。例如，实现一个优先队列或需要按键值顺序迭代的场景。
  - 范围查询：`map`支持高效的范围查询，这在需要获取一定范围内所有键值对的场景中非常有用。例如，统计某个区间内的数据。
  - 排序输出： 如果需要按键值的自然顺序输出数据，`map`提供了便利。
- **`unordered_map`**
  - 快速查找: 当需要快速查找特定键对应的值时，`unordered_map`是一个非常好的选择。例如，查找个人的电话号码或实现一个缓存系统。
  - 元素频率计数：在需要计算元素出现频率的场景下，`unordered_map`非常有用。例如，计算文档中每个单词的出现次数。
  - 大数据处理: 当处理大量数据且不需要保持数据的有序性时，`unordered_map`可以提供更高的性能和更低的空间开销。

#### 4. 示例

##### map

```c++
#include <iostream>
#include <map>

int main() {
    std::map<int, std::string> m;
    m[1] = "one";
    m[2] = "two";
    m[3] = "three";

    // 按键值顺序迭代
    for (const auto& pair : m) {
        std::cout << pair.first << ": " << pair.second << std::endl;
    }

    return 0;
}
```

##### unordered_map

```c++
#include <iostream>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> um;
    um["apple"] = 1;
    um["banana"] = 2;
    um["cherry"] = 3;

    // 快速查找
    if (um.find("banana") != um.end()) {
        std::cout << "Found banana: " << um["banana"] << std::endl;
    } else {
        std::cout << "Banana not found" << std::endl;
    }

    return 0;
}
```

### 8、c++ 11 以来有哪些新特性，标准库增加了什么新功能？

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

#### 标准库新增功能

1. 正则表达式支持

引入了`<regex>`头文件，支持正则表达式的匹配和搜索。

```c++
#include <regex>
std::regex re("(\\d+)-(\\d+)-(\\d+)");
std::smatch match;
std::string s = "2023-09-26";
if (std::regex_search(s, match, re)) {
    std::cout << "Year: " << match[1] << ", Month: " << match[2] << ", Day: " << match[3] << std::endl;
}
```

2. 原子操作

引入了`<atomic>`头文件，支持原子操作，提高了多线程环境下的安全性。

```c++
#include <atomic>
std::atomic<int> counter(0);
counter.fetch_add(1);
```

3. 时间库

引入了`<chrono>`头文件，提供了更精确的时间和日期处理功能。

```c++
#include <chrono>
auto start = std::chrono::high_resolution_clock::now();
// 执行一些操作
auto end = std::chrono::high_resolution_clock::now();
auto duration = std::chrono::duration_cast<std::chrono::microseconds>(end - start).count();
std::cout << "Duration: " << duration << " microseconds" << std::endl;
```

4. 文件系统库

C++17 引入了`<filesystem>`头文件，提供了文件和目录操作的功能。

```c++
#include <filesystem>
namespace fs = std::filesystem;
fs::path p = "/path/to/directory";
if (fs::exists(p) && fs::is_directory(p)) {
    for (const auto& entry : fs::directory_iterator(p)) {
        std::cout << entry.path().filename() << std::endl;
    }
}
```

5. 可选值 (`std::optional`)

C++17 引入了`std::optional`，表示可能有值也可能没有值的类型，类似于其他语言中的`Nullable`类型。

```c++
#include <optional>
std::optional<int> value;
value = 42;
if (value) {
    std::cout << "Value is: " << *value << std::endl;
} else {
    std::cout << "Value is not set" << std::endl;
}
```

6. 字符串视图 (`std::string_view`)

C++17 引入了 `std::string_view`，提供了一种轻量级的字符串表示方式，避免了不必要的拷贝。

```c++
#include <string_view>
std::string_view sv = "Hello, World!";
std::cout << "Length: " << sv.length() << std::endl;
std::cout << "Substring: " << sv.substr(0, 5) << std::endl;
```

7. 并行算法

C++17 引入了并行算法，支持并行执行标准库中的算法，提高了性能。

```c++
#include <algorithm>
#include <execution>
std::vector<int> vec = {1, 2, 3, 4, 5};
std::sort(std::execution::par, vec.begin(), vec.end());
```

### 9、写一个右值引用的场景？

主要目的是为了支持移动语义（Move Semantics）和完美转发（Perfect Forwarding）。

这里用移动语义给大家做一个分享。

#### 场景描述

假设有一个类 `MyString`，它管理一个动态分配的字符串缓冲区。在 C++98 中，如果要实现一个高效的拷贝构造函数和赋值运算符，通常需要进行深拷贝，这会导致不必要的内存分配和复制操作。通过引入移动语义，可以避免这些不必要的开销。

#### 代码

定义一个 `MyString` 类，包含一个指向动态分配内存的指针：

```c++
#include <iostream>
#include <cstring>

class MyString {
public:
    // 构造函数
    MyString(const char* str) {
        size_t len = std::strlen(str);
        data = new char[len + 1];
        std::strcpy(data, str);
    }

    // 拷贝构造函数
    MyString(const MyString& other) {
        size_t len = std::strlen(other.data);
        data = new char[len + 1];
        std::strcpy(data, other.data);
    }

    // 移动构造函数
    MyString(MyString&& other) noexcept {
        data = other.data;
        other.data = nullptr; // 将 other 的资源置为空
    }

    // 拷贝赋值运算符
    MyString& operator=(const MyString& other) {
        if (this != &other) {
            delete[] data;
            size_t len = std::strlen(other.data);
            data = new char[len + 1];
            std::strcpy(data, other.data);
        }
        return *this;
    }

    // 移动赋值运算符
    MyString& operator=(MyString&& other) noexcept {
        if (this != &other) {
            delete[] data;
            data = other.data;
            other.data = nullptr; // 将 other 的资源置为空
        }
        return *this;
    }

    // 析构函数
    ~MyString() {
        delete[] data;
    }

    // 打印字符串
    void print() const {
        std::cout << data << std::endl;
    }

private:
    char* data;
};
```

#### 应用场景

假设有一个函数 `createString`，它创建并返回一个 `MyString` 对象：

```c++
MyString createString() {
    return MyString("Hello, World!");
}
```

当调用这个函数时，返回的 `MyString` 对象是一个临时对象，即右值。如果使用移动构造函数，可以显著提高性能：

```c++
int main() {
    MyString str1 = createString(); // 使用移动构造函数
    str1.print();

    MyString str2 = "Another String";
    str2 = createString(); // 使用移动赋值运算符
    str2.print();

    return 0;
}
```

- `str1` 的初始化使用了移动构造函数，直接将临时对象的资源转移给了 `str1`，避免了不必要的内存分配和复制。
- `str2` 的赋值使用了移动赋值运算符，同样避免了不必要的资源复制。

### 10、cpp 变成可执行文件的过程，链接的过程在做什么事，可执行文件里各部分都有什么？

从 `.cpp` 文件到可执行文件的过程，可以细分为四个主要步骤：预处理（Preprocessing）、编译（Compilation）、汇编（Assembly）和链接（Linking）。

#### 预处理

预处理是编译过程的第一个阶段，主要处理源代码中的预处理指令，如 `#include`、`#define` 等。在这个阶段，预处理器会将所有的宏定义替换为其实际内容，将被 `#include` 指令引用的头文件内容插入到源文件中相应的位置，删除注释等。预处理的结果是一个扩展源代码文件，通常以 `.i` 为扩展名 。

#### 编译

编译阶段的主要任务是将预处理后的源代码转换成汇编语言代码。编译器在这个过程中进行词法分析、语法分析、语义分析以及优化等操作，最终生成汇编代码文件，通常以 `.s` 为扩展名。这一阶段是对源代码的高级语言结构进行解析和转换，确保其符合目标平台的指令集架构 。

#### 汇编

汇编阶段将上一阶段产生的汇编代码转换为目标代码（Object Code），这是一个二进制文件，包含了可以直接被机器执行的指令。目标代码文件通常以 `.o` 或 `.obj` 为扩展名。这个阶段的工作相对直接，主要是将汇编语言的每一条指令翻译成对应的机器码 。

#### 链接

链接是将多个目标文件和库文件组合成一个可执行文件的过程。在这个阶段，链接器负责解析和解决符号引用，即将不同目标文件或库文件中定义的函数和变量的引用关联起来。此外，链接器还会处理程序的入口点、添加必要的运行时支持代码等，确保生成的可执行文件可以在目标平台上正确运行。链接过程是生成可执行文件的最后一步，它不仅将程序的不同部分整合在一起，还可能包括优化和调整内存布局等工作 。

#### 可执行文件的组成部分

- 文本段（Text Segment）：这部分包含了程序的机器指令，即编译后的代码。它是只读的，因为运行时不应该修改代码。
- 数据段（Data Segment）：数据段又可以分为初始化数据区和未初始化数据区（BSS段）。初始化数据区存放了已经赋予初值的全局变量和静态变量；未初始化数据区存放了未赋予初值的全局变量和静态变量。
- 堆（Heap）：用于动态分配内存。程序运行时，可以请求操作系统分配或释放堆上的内存。
- 栈（Stack）：用于存储函数调用时的局部变量和函数参数。每次函数调用时，都会在栈上创建一个新的栈帧，当函数返回时，栈帧被销毁。
- 重定位表（Relocation Table）：如果存在的话，这部分提供了有关如何调整地址引用的信息，以便程序可以在不同的内存地址空间中正确运行。
- 符号表（Symbol Table）：包含程序中的函数名、变量名及其地址等信息，主要用于调试目的。

### 11、进程空间，栈会保存什么？

进程空间，是操作系统为每个运行中的进程分配的虚拟内存空间，旨在确保每个进程在运行时都感觉自己独占了整个计算机的内存资源。

#### 内存布局

进程地址空间通常被划分为几个关键部分，每部分承担不同的功能，具体包括：

- 代码段（Text Segment）：这部分内存区域用于存放程序的机器指令。它是只读的，以防止程序意外修改自身代码，从而导致不可预测的行为。
- 初始化数据段（Initialized Data Segment）：存储已初始化的全局变量和静态变量。这些变量在程序启动时已经被赋予了初始值。
- 未初始化数据段（BSS Segment）：存放未初始化的全局变量和静态变量。尽管这些变量没有显式初始化，但在程序启动时会被操作系统自动初始化为零。
- 堆（Heap）：用于动态内存分配。程序员可以通过标准库函数（如`malloc()`、`calloc()`、`realloc()`和`free()`）来请求或释放堆上的内存。堆的管理较为复杂，需要堆管理器（如glibc中的ptmalloc）来维护空闲内存列表，跟踪哪些内存区域是可用的，并在需要时将其分配给程序。
- 栈（Stack）：用于存储函数调用时的**局部变量**、**函数参数**、**返回地址**、**寄存器值**、**临时存储区**等。栈的操作遵循后进先出（LIFO）原则，每当函数被调用时，一个新的栈帧会被压入栈顶；当函数返回时，该栈帧会被从栈顶弹出。

#### 地址空间的虚拟化

进程地址空间本质上是虚拟内存的一部分，而非实际的物理内存。这意味着每个进程都有自己的独立地址空间，即使物理内存有限，也能通过虚拟内存技术让进程认为自己拥有了大量的内存资源。操作系统通过页表等机制将虚拟地址映射到物理地址，从而实现了内存的虚拟化、隔离和保护。

#### 进程间的隔离性

为了保证进程间的独立性和安全性，操作系统确保每个进程只能访问自己的地址空间，而不能直接访问其他进程的内存。这种隔离机制有效地防止了恶意程序或错误代码对系统其他部分造成损害。然而，进程间的信息交换仍然是必要的，操作系统为此提供了多种通信机制，如管道、套接字、共享内存等。

### 12、介绍一下知道的内存管理？

### 13、new 的底层原理是什么，底层操作系统如何将空间分配给用户进程的，new有哪些用法？

#### `new` 的底层原理

1. 内存分配：当使用 `new` 时，首先会调用底层的内存分配函数，通常是 `operator new` 或者 `malloc` 函数。`operator new` 是C++中提供的一个全局函数，用于分配指定大小的内存块。如果分配成功，`operator new` 返回指向这块内存的指针；如果失败，则抛出 `std::bad_alloc` 异常。
2. 构造函数调用：一旦内存分配成功，`new` 操作符会调用相应的构造函数来初始化这块内存上的对象。这意味着，`new` 不仅分配内存，还负责对象的初始化。这对于包含复杂状态的类对象尤为重要，因为它们可能需要执行特定的初始化逻辑。

#### 操作系统如何将空间分配给用户进程

操作系统通过虚拟内存系统来管理内存分配。当进程请求分配内存时（例如，通过 `new` 操作符），操作系统会检查是否有足够的物理内存可用。如果没有足够的空闲物理内存，操作系统可能会将一些当前不活跃的页面交换到磁盘上，腾出空间来满足新的内存请求。这个过程对用户进程来说是透明的。操作系统还会维护一个页表，用于跟踪虚拟地址与物理地址之间的映射关系。

#### `new` 的用法

1. 普通对象的分配：最基本的 `new` 用法是为单个对象分配内存。

```c++
MyClass* obj = new MyClass();
```

首先调用 `operator new` 分配足够的内存来存储一个 `MyClass` 对象，然后调用 `MyClass` 的默认构造函数来初始化这个对象。

2. 数组的分配：`new` 也可以用来分配一个对象数组。

```c++
MyClass* arr = new MyClass[10];
```

`new` 会为10个 `MyClass` 对象分配内存，并调用每个对象的默认构造函数。

注意，对于数组的删除，应该使用 `delete[]` 而不是 `delete`，以确保所有对象的析构函数都被正确调用。

3. 带参数的构造函数：如果类的构造函数需要参数，可以在 `new` 表达式中直接提供这些参数。

```c++
MyClass* obj = new MyClass(param1, param2);
```

为 `MyClass` 对象分配内存，并使用 `param1` 和 `param2` 作为参数调用相应的构造函数。

4. Placement new：这是一种特殊的 `new` 形式，允许程序员指定内存的位置，而不是由 `new` 自动分配。这主要用于需要精细控制内存布局的场景，如内存池管理。

```c++
char buffer[sizeof(MyClass)];
MyClass* obj = new(buffer) MyClass();
```

`new(buffer)` 表示使用 `buffer` 中的内存来创建 `MyClass` 对象。使用 Placement new 创建的对象需要手动调用析构函数来销毁。

### 14、怎么调试-gdb, 介绍你知道的gdb命令？

给个简单的流程作参考

#### 启动GDB

1. 启动调试：可以通过命令行启动GDB，同时指定要调试的程序。

   ```sh
   gdb ./your_program
   ```

   这将启动GDB，并加载名为 `your_program` 的可执行文件。如果需要传递命令行参数给程序，可以在启动GDB后使用 `set args` 命令。

   ```
   set args arg1 arg2
   ```

2. 运行程序：在GDB中启动程序的执行，可以使用 `run` 命令。

   ```
   run
   ```

#### 设置断点

1. 在指定行设置断点：使用 `break` 命令在源代码的特定行设置断点。

   ```
   break filename:line_number
   ```

   比如，要在 `main.c` 文件的第10行设置断点，可以使用 `break main.c:10`。

2. 在函数入口设置断点：同样使用 `break` 命令，但这次指定函数名。

   ```
   break function_name
   ```

3. 条件断点：可以在满足特定条件时才触发的断点。

   ```
   break filename:line_number if condition
   ```

   例如，`break main.c:10 if x > 10` 会在 `x` 大于10时在第10行触发断点。

#### 单步执行

1. 单步执行：使用 `next` 命令单步执行代码，每次执行一行。

   ```
   next
   ```

2. 进入函数内部执行：如果当前行是一次函数调用，使用 `step` 命令可以进入该函数内部继续单步执行。

   ```
   step
   ```

3. 继续执行：当程序暂停在断点处时，使用 `continue` 命令可以让程序继续执行直到下一个断点或程序结束。

   ```
   continue
   ```

#### 查看变量

1. 打印变量值：使用 `print` 命令可以查看变量的当前值。

   ```
   print variable_name
   ```

2. 查看局部变量：使用 `info locals` 命令可以列出当前作用域内的所有局部变量及其值。

   ```
   info locals
   ```

#### 查看和操作内存

1. 查看内存内容：使用 `x` 命令可以查看内存地址的内容。

   ```
   x/4wx 0xaddress
   ```

   这个命令表示从 `0xaddress` 地址开始，以16进制显示4个字的数据。

2. 修改变量值：使用 `set` 命令可以直接修改变量的值。

   ```
   set variable_name = value
   ```

#### 查看调用栈

1. 查看调用栈：使用 `backtrace` 或 `bt` 命令可以查看当前的调用栈。

   ```
   backtrace
   ```

2. 切换调用栈帧：使用 `frame` 命令可以选择不同的调用栈帧。

   ```
   frame frame_number
   ```

#### 结束调试

退出GDB：使用`quit`命令可以退出GDB。

```
quit
```

### 15、介绍一下你知道的linux指令？

### 16、文件的软连接和硬链接？

#### 软连接（Symbolic Link）

##### 定义

软连接，也称为符号链接，是一种特殊的文件，它包含指向另一个文件或目录的路径信息。软连接本身是一个独立的文件，它并不包含实际的数据，而是作为目标文件或目录的引用。

##### 创建方法

使用 `ln -s` 命令创建软连接：

```
ln -s source_file_or_directory target_link
```

`source_file_or_directory` 是你想要链接的目标文件或目录，`target_link` 是创建的软连接文件名。

##### 特点

- 独立性：软连接是一个独立的文件，有自己的inode号。
- 跨文件系统：软连接可以跨越不同的文件系统。
- 目录支持：可以为目录创建软连接。
- 路径依赖：如果目标文件或目录被删除或移动，软连接将失效。
- 大小：软连接的大小为其指向的路径字符串的长度。
- 权限：软连接的权限无关紧要，关键在于目标文件或目录的权限。

#### 硬链接（Hard Link）

##### 定义

硬链接是指向同一个文件数据的多个入口。每个硬链接都指向同一个inode，这意味着所有硬链接共享相同的文件数据。

##### 创建方法

使用 `ln` 命令创建硬链接（不带 `-s` 参数）：

```
ln source_file target_link
```

 `source_file` 是你想要链接的目标文件，`target_link` 是创建的硬链接文件名。

##### 特点

- 共享数据：硬链接共享同一个inode，因此所有硬链接都指向相同的文件数据。
- 不跨文件系统：硬链接必须在同一个文件系统内创建。
- 不支持目录：不能为目录创建硬链接。
- 删除影响：删除源文件不会影响其他硬链接，只有当所有硬链接都被删除时，文件数据才会被真正删除。
- inode计数：每个硬链接都会增加文件的链接计数，直到链接计数为零，文件数据才会被释放。

### 17、介绍一下Go的Goroutine, 和线程的区别？

Go语言中的Goroutine是一种轻量级的线程，由Go语言的运行时（runtime）管理。Goroutine的设计使得Go程序能够高效地处理大量并发任务，同时保持较低的资源消耗。

#### 主要特点

1. 轻量级：与操作系统线程相比，Goroutine的创建和销毁成本更低，内存占用也更少。每个Goroutine初始仅占用约2KB的内存，这使得在一个Go程序中可以轻松创建成千上万个Goroutine。
2. 高效调度：Goroutine的调度是由Go运行时管理的，而不是由操作系统管理。这意味着Goroutine的调度可以更灵活、更快速，减少了上下文切换的成本。
3. 动态栈：Goroutine的栈大小是动态调整的，可以根据需要增长或缩小，这避免了像操作系统线程那样需要预先分配固定大小的栈空间。
4. 高并发：Go运行时可以将大量的Goroutine高效地分配到较少的OS线程上运行，即使是在多核处理器上也能充分利用硬件资源，实现真正的并行计算。
5. 简单的并发模型：通过使用`go`关键字就可以启动一个Goroutine，这种方式简洁明了，降低了并发编程的门槛。

#### 与线程的区别

1. 资源消耗：Goroutine比操作系统线程更轻量级，创建和销毁的成本更低，更适合处理大量并发任务。
2. 调度机制：Goroutine的调度是由Go运行时管理的，而操作系统线程的调度是由操作系统内核管理的。这意味着Goroutine的上下文切换更快，因为它们不需要进入内核态。
3. 栈管理：Goroutine的栈是动态调整的，而大多数操作系统线程的栈大小是固定的。
4. 并发模型：Goroutine的设计是为了支持高并发场景，而操作系统线程虽然也可以支持并发，但在资源消耗和调度效率上不如Goroutine。
5. 适用场景：Goroutine特别适合I/O密集型和网络服务等需要处理大量并发请求的应用，而操作系统线程则更适合计算密集型任务。

### 18、IO多路复用的原理，应用场景？

#### 原理

IO多路复用是一种在单个线程中管理多个输入/输出通道的技术，允许一个线程同时监听多个输入流（例如网络套接字、文件描述符等），并在有数据可读或可写时进行相应的处理。这种技术通过将多个IO通道注册到一个事件管理器中，然后通过阻塞方式等待事件的发生。一旦有事件发生（如有数据可读或可写），线程就会被唤醒，然后可以针对具体的事件进行处理。

IO多路复用的核心原理是利用操作系统提供的IO多路复用技术，如`select`、`poll`或`epoll`等，来监控多个文件描述符的状态变化。

#### 具体实现

- Select：最早出现的IO多路复用接口，适用于所有平台。但是其存在一些局限性，比如最大文件描述符数限制（通常是1024），并且每次调用`select`都需要重新传入文件描述符集合。
- Poll：解决了`select`的最大文件描述符数限制问题，但是其性能随着文件描述符数量的增加而下降。
- Epoll：Linux特有的IO多路复用接口，具有更好的性能和更高的扩展性。`epoll`通过维护一个文件描述符表来跟踪关注的事件，只有当事件发生时才会通知应用程序。这使得`epoll`在处理大量并发连接时表现出色。

#### 应用场景

IO多路复用主要应用于网络编程中，特别是在需要处理大量并发连接的场景下。以下是几个典型的应用场景：

1. 高性能Web服务器：在Web服务器中，经常需要同时处理多个客户端的连接请求。使用IO多路复用可以避免为每个客户端连接创建一个线程或进程，从而减少系统资源的占用，提高服务器的并发能力。例如，Nginx就是一个广泛使用的支持IO多路复用的Web服务器。
2. 数据库服务器：数据库服务器需要处理来自多个客户端的查询请求。通过使用IO多路复用，数据库服务器可以用一个线程同时监听所有连接的请求，并在有请求到达时进行处理。这不仅提高了系统的响应速度，还减少了线程切换的开销。
3. 实时通信系统：在实时通信系统中，服务器需要同时处理多个客户端的连接，并及时响应客户端的消息。使用IO多路复用可以确保服务器在高并发情况下仍能保持良好的性能和稳定性。
4. 游戏服务器：在线游戏服务器需要处理来自大量玩家的实时数据传输。使用IO多路复用可以有效地管理这些连接，确保游戏的流畅性和响应性。
5. 物联网（IoT）平台：在物联网平台中，服务器需要处理来自大量设备的数据上报和指令下发。使用IO多路复用可以高效地管理和响应这些设备的连接，提高系统的整体性能。

#### 优势

- 减少线程创建和销毁开销：通过在单个线程中管理多个IO通道，避免了频繁创建和销毁线程带来的资源消耗。
- 降低线程切换开销：减少了线程之间的上下文切换，提高了系统的并发性能。
- 提高资源利用率：通过高效地管理多个IO通道，提高了系统资源的利用率，特别是对于内存和CPU资源。

### 19、在linux c++ 写一个服务器应该怎么写？各个模块应该怎么设计

### 20、手写Trie

#### 思路

1. 节点定义：

   - 每个节点存储一个字符和指向子节点的指针（通常使用数组或哈希表来表示）。

   - 需要一个布尔值来标记该节点是否是某个单词的结束。

2. 插入操作 (`insert`)：

   - 从根节点开始，依次遍历字符串中的每个字符。

   - 如果字符不存在，则创建新的子节点。

   - 在最后一个字符的节点上标记为单词的结束。

3. 查找操作 (`search`)：

   - 从根节点开始，遍历字符串中的每个字符。

   - 如果字符不存在，返回 `false`。

   - 如果遍历完所有字符，检查最后一个节点是否标记为单词结束。

4. 前缀查找操作 (`startsWith`)：与查找操作类似，但不需要检查最后一个节点是否为单词结束。

#### 参考代码

```
#include <iostream>
#include <unordered_map>
using namespace std;

// 定义 Trie 树的节点
class TrieNode {
public:
    unordered_map<char, TrieNode*> children; // 存储子节点
    bool isEnd; // 标记是否为单词的结束

    TrieNode() : isEnd(false) {} // 初始化
};

// Trie 类的定义
class Trie {
private:
    TrieNode* root; // 根节点

public:
    // 构造函数
    Trie() {
        root = new TrieNode(); // 初始化根节点
    }

    // 插入单词
    void insert(string word) {
        TrieNode* node = root; // 从根节点开始
        for (char ch : word) {
            // 如果当前字符不存在，创建新节点
            if (node->children.find(ch) == node->children.end()) {
                node->children[ch] = new TrieNode();
            }
            node = node->children[ch]; // 移动到子节点
        }
        node->isEnd = true; // 标记单词结束
    }

    // 搜索单词
    bool search(string word) {
        TrieNode* node = root; // 从根节点开始
        for (char ch : word) {
            // 如果字符不存在，返回 false
            if (node->children.find(ch) == node->children.end()) {
                return false;
            }
            node = node->children[ch]; // 移动到子节点
        }
        return node->isEnd; // 返回是否为单词结束
    }

    // 前缀查找
    bool startsWith(string prefix) {
        TrieNode* node = root; // 从根节点开始
        for (char ch : prefix) {
            // 如果字符不存在，返回 false
            if (node->children.find(ch) == node->children.end()) {
                return false;
            }
            node = node->children[ch]; // 移动到子节点
        }
        return true; // 前缀存在
    }
};

// 示例主函数
int main() {
    Trie trie; // 创建 Trie 对象
    trie.insert("apple");
    cout << trie.search("apple") << endl;   // 返回 true
    cout << trie.search("app") << endl;     // 返回 false
    cout << trie.startsWith("app") << endl; // 返回 true
    trie.insert("app");
    cout << trie.search("app") << endl;     // 返回 true
    return 0;
}
```

### 21、反问