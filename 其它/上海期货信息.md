> https://www.nowcoder.com/feed/main/detail/48511260866541ffbcb9cdbc4adbd964

## 一面

### 1、C++ 多态是什么？

C++ 中实现多态性的主要方式是通过虚函数（virtual function）。虚函数是在基类中声明为虚函数的函数，在派生类中可以重写（override）它们，从而实现不同的行为。当通过基类指针或引用调用虚函数时，实际调用的是根据对象类型确定的派生类中的函数实现，而不是基类中的函数。

1. 编译时多态（静态多态）：

函数重载：在同一个作用域内，可以定义多个函数具有相同的名称但不同的参数列表。编译器根据调用时提供的参数类型和数量来决定调用哪个函数。这种多态性称为编译时多态，因为在编译时就可以确定调用的函数。

示例：

```
#include <iostream>

void print(int value) {
    std::cout << "Integer: " << value << std::endl;
}

void print(double value) {
    std::cout << "Double: " << value << std::endl;
}

int main() {
    print(5);       // 调用 print(int)
    print(5.5);     // 调用 print(double)
    return 0;
}
```

2. 运行时多态（动态多态）：

虚函数：通过在基类中声明虚函数，并在派生类中进行重写，可以实现运行时多态。在使用基类指针或引用调用虚函数时，实际调用的是根据对象类型确定的派生类中的函数实现，而不是基类中的函数。这种多态性称为运行时多态，因为调用的函数在运行时确定。

示例：

```
#include <iostream>

class Animal {
public:
    virtual void makeSound() {
        std::cout << "Some sound" << std::endl;
    }
};

class Dog : public Animal {
public:
    void makeSound() override {
        std::cout << "Woof!" << std::endl;
    }
};

class Cat : public Animal {
public:
    void makeSound() override {
        std::cout << "Meow!" << std::endl;
    }
};

int main() {
    Animal* animal1 = new Dog();
    Animal* animal2 = new Cat();

    animal1->makeSound(); // 输出 "Woof!"
    animal2->makeSound(); // 输出 "Meow!"

    delete animal1;
    delete animal2;
    return 0;
}
```

### 2、虚函数是什么？纯虚函数是什么？

1. 虚函数

虚函数是 C++ 中用于实现多态性的一种机制。在基类中声明一个函数为虚函数，然后在派生类中重新定义（override）这个函数，可以使得通过基类指针或引用调用这个函数时，实际调用的是根据对象类型确定的派生类中的函数实现。

要将一个成员函数声明为虚函数，只需在其声明前面加上 `virtual` 关键字。例如：

```
class Base {
public:
    virtual void foo() {
        // 基类中的虚函数实现
    }
};
```

上面的例子中，`foo` 被声明为虚函数。然后，可以在派生类中重写这个虚函数：

```
class Derived : public Base {
public:
    void foo() override {
        // 派生类中的虚函数实现
    }
};
```

通过将 `Base*` 类型的指针指向 `Derived` 类型的对象，并调用 `foo` 函数，可以实现动态绑定，确保在运行时调用的是 `Derived` 类中的 `foo` 函数。

2. 纯虚函数

纯虚函数是一个在基类中声明的虚函数，但没有在基类中给出实际的实现，而是通过在函数声明末尾添加 `= 0` 来指示这是一个纯虚函数。纯虚函数的存在使得基类成为一个抽象类，无法被实例化，只能用作其他类的基类。派生类必须实现纯虚函数才能成为可实例化的类。

一个包含纯虚函数的抽象类的示例：

```
class AbstractShape {
public:
    virtual void draw() const = 0; // 纯虚函数
    virtual double area() const = 0; // 另一个纯虚函数
};

class Circle : public AbstractShape {
public:
    void draw() const override {
        // 实现 draw 函数
    }

    double area() const override {
        // 实现 area 函数
    }
};

int main() {
    // AbstractShape* shape = new AbstractShape(); // 错误，抽象类不能实例化
    AbstractShape* circle = new Circle(); // 正确，派生类实例化
    circle->draw(); // 调用 Circle 类中的 draw 函数
    delete circle;
    return 0;
}
```

示例中，`AbstractShape` 是一个抽象类，它包含两个纯虚函数 `draw` 和 `area`。`Circle` 类是 `AbstractShape` 的派生类，并且实现了 `draw` 和 `area` 函数。在 `main` 函数中，不能直接实例化 `AbstractShape`，但可以通过指向 `Circle` 类的指针实现多态性。

### 3、类模板的特化是什么？

在 C++ 中，类模板是一种通用的类定义，它可以用来创建特定类型的类。类模板可以包含一个或多个类型参数，这些参数可以在模板被实例化时替换为具体的类型。类模板的特化（specialization）是对类模板进行特定类型的实例化，使得模板的某些部分针对特定类型做出定制化的实现。

类模板的特化可以分为两种类型：全特化（full specialization）和部分特化（partial specialization）。

1. 全特化（Full specialization）：

   对整个类模板进行特化，为特定的类型提供完全定制的实现。在全特化中，模板的所有类型参数都被具体的类型替换。

```
// 定义一个类模板
template <typename T>
class MyTemplate {
public:
    void print() {
        std::cout << "Generic template\n";
    }
};

// 对类模板进行全特化
template <>
class MyTemplate<int> {
public:
    void print() {
        std::cout << "Specialized template for int\n";
    }
};

int main() {
    MyTemplate<double> obj1; // 使用通用模板
    obj1.print(); // 输出 "Generic template"

    MyTemplate<int> obj2; // 使用特定于 int 的模板
    obj2.print(); // 输出 "Specialized template for int"

    return 0;
}
```

2. 部分特化（Partial specialization）：

   对类模板的部分参数进行特化，允许在模板被实例化时根据特定条件选择不同的实现。部分特化是在模板定义中使用一个或多个类型参数的特定值来创建新的模板定义。

```
// 定义一个类模板
template <typename T, typename U>
class MyTemplate {
public:
    void print() {
        std::cout << "Generic template\n";
    }
};

// 对类模板进行部分特化
template <typename U>
class MyTemplate<int, U> {
public:
    void print() {
        std::cout << "Partial specialization for int\n";
    }
};

int main() {
    MyTemplate<double, int> obj1; // 使用通用模板
    obj1.print(); // 输出 "Generic template"

    MyTemplate<int, double> obj2; // 使用部分特化的模板
    obj2.print(); // 输出 "Partial specialization for int"

    return 0;
}
```

### 4、Tcmalloc用过吗？它的底层是什么样的？

Tcmalloc 是 Google 开发的一个高效的内存分配器，它专门用于替代系统默认的内存分配器（比如 malloc/free）以提高程序的性能和可扩展性。Tcmalloc 的设计目标是在多线程环境下提供高性能的内存分配和释放，以及降低内存碎片化的程度。

Tcmalloc 的底层实现采用了一些高效的技术和算法：

1. 高效的内存分配：Tcmalloc 使用了多级分配器（multi-level allocator）的设计，以减少锁竞争和提高内存分配的并发性。它将内存按照不同的大小进行分类管理，并为不同大小的内存块使用不同的分配策略，从而提高了内存分配的效率。
2. 优化的内存释放：Tcmalloc 在释放内存时会进行一些优化，例如延迟释放（delayed free）和空闲列表的管理，以减少内存碎片化和提高内存的重用率。
3. 线程安全性：Tcmalloc 被设计成线程安全的，可以在多线程环境下高效地进行内存分配和释放，而不需要显式的锁机制。
4. 性能监控和调优：Tcmalloc 提供了丰富的性能监控和调优接口，可以用于监控内存的使用情况、性能指标和调优参数，并且可以根据应用程序的特点进行定制化的调优。

### 5、Define和inline有什么区别？

1. `#define` 宏定义：

- `#define` 是 C/C++ 中的预处理指令，用于定义宏。
- 宏定义会在预处理阶段简单地将宏名替换为相应的文本。
- 宏定义不进行类型检查，只是简单的文本替换，可能会导致一些意料之外的行为。
- 由于宏定义只是简单的文本替换，因此可能会导致代码的可读性下降，并且在展开后的代码中可能会出现重复的代码。

```
#define SQUARE(x) ((x) * (x))

int main() {
    int a = 5;
    int result = SQUARE(a + 1); // 展开后变为 ((a + 1) * (a + 1))
    return 0;
}
```

2. `inline` 函数：

- `inline` 是 C++ 中的关键字，用于声明内联函数。
- 内联函数是一种特殊的函数，编译器会尝试在调用处直接将函数的代码插入，而不是通过函数调用的方式来执行。
- 内联函数通常适用于函数体较小且频繁调用的情况，可以减少函数调用的开销。
- 编译器对于内联函数的内联操作并不是强制的，只是建议性的，具体是否内联由编译器决定。

```
inline int square(int x) {
    return x * x;
}

int main() {
    int a = 5;
    int result = square(a + 1); // 内联函数调用，编译器可能会直接将函数体插入到这里
    return 0;
}
```

### 6、重载是什么？底层是怎么实现的？

在 C++ 中，函数重载（Function Overloading）是指在同一个作用域内，允许定义多个同名函数，但这些函数具有不同的参数列表（参数的类型、个数或顺序不同）。在调用这些同名函数时，编译器会根据传入的参数来确定调用哪个重载函数。

函数重载的特点包括：

1. 函数名称相同。
2. 参数列表不同（类型、个数、顺序）。

函数重载的底层实现主要涉及两个方面：

1. 名字修饰（Name Mangling）：
   - 在 C++ 中，函数的名称在编译后会被修饰（mangle），这个修饰是为了区分不同的函数重载版本。
   - 修饰的过程会将函数名和参数列表信息编码成一个唯一的符号，以便在链接阶段能够正确地识别不同的函数版本。
   - 不同的编译器可能会采用不同的名字修饰方案，因此在 C++ 中使用函数重载时需要注意不同编译器之间的兼容性。
2. 编译器解析：
   - 当调用一个重载函数时，编译器会根据传入的参数类型、个数和顺序来选择合适的重载版本。
   - 如果找到了与传入参数匹配的函数，则调用该函数；如果没有找到匹配的函数，则会产生编译错误。

```
#include <iostream>

// 重载的函数
void print(int value) {
    std::cout << "Integer: " << value << std::endl;
}

void print(double value) {
    std::cout << "Double: " << value << std::endl;
}

int main() {
    print(5);       // 调用 print(int)
    print(5.5);     // 调用 print(double)
    return 0;
}
```

### 7、如果有两个重载函数它们的输入参数是 int char 和 char int 那么如果输入char char会怎么调用？

在 C++ 中，如果存在两个重载函数，一个接受 `int` 和 `char` 参数，另一个接受 `char` 和 `int` 参数，那么当传入 `char` 类型的参数时，编译器会根据函数调用的上下文和可选的隐式类型转换来确定调用哪个重载函数。

在这种情况下，如果传入的参数是两个 `char` 类型的值，编译器会尝试进行类型提升（Type Promotion）来匹配重载函数的参数类型。在 C++ 中，`char` 类型可以被隐式地转换为 `int` 类型，因此编译器会将两个 `char` 类型的参数分别转换为 `int` 类型，然后选择与 `int` 类型和 `int` 类型匹配的重载函数进行调用。

给个例子：

```
#include <iostream>

void foo(int x, char y) {
    std::cout << "foo(int, char) called with " << x << " and " << y << std::endl;
}

void foo(char x, int y) {
    std::cout << "foo(char, int) called with " << x << " and " << y << std::endl;
}

int main() {
    char a = 'a';
    char b = 'b';
    foo(a, b); // 将两个 char 类型的参数分别转换为 int 类型，然后调用 foo(int, char)
    return 0;
}
```

示例中，定义了两个重载函数 `foo`，一个接受 `int` 和 `char` 参数，另一个接受 `char` 和 `int` 参数。在 `main` 函数中，传入两个 `char` 类型的参数给 `foo` 函数，由于 `char` 类型可以隐式转换为 `int` 类型，因此编译器会将这两个 `char` 类型的参数分别转换为 `int` 类型，然后调用与 `int` 和 `int` 类型匹配的重载函数 `foo(int, char)`。

### 8、Vector和deque有什么区别？是线程安全的吗？如何实现的呢？

1. 区别：
   - `std::vector` 是一个动态数组，它在内存中是连续存储的，因此支持快速的随机访问和尾部插入/删除操作。但在中间插入/删除操作时，可能需要移动大量元素，因此效率较低。
   - `std::deque` 是一个双端队列，它在内存中不要求连续存储，因此在中间插入/删除操作时，效率比 `std::vector` 更高。`std::deque` 通常由多个固定大小的缓冲区组成，称为块（chunk），在插入/删除操作时，只需要移动少量的块而不是整个序列。
2. 线程安全性：
   - 标准的 `std::vector` 和 `std::deque` 不是线程安全的，它们不提供任何内置的线程安全保证。如果在多线程环境下使用这些容器，需要自行确保线程安全性，例如使用互斥锁来保护共享的容器访问。
3. 实现：
   - `std::vector` 的实现通常基于动态数组，它会在需要时动态扩展内部数组的大小，并且在元素插入/删除时可能需要移动大量元素以保持内存的连续性。
   - `std::deque` 的实现通常基于双向链表，每个节点都包含一个固定大小的缓冲区（块），在插入/删除操作时，只需要移动少量的块而不是整个序列，因此效率比 `std::vector` 更高。

使用实例：

```
#include <iostream>
#include <vector>
#include <deque>

int main() {
    // 使用 std::vector
    std::vector<int> vec = {1, 2, 3};
    vec.push_back(4); // 在尾部插入元素
    vec.insert(vec.begin() + 1, 5); // 在指定位置插入元素
    vec.pop_back(); // 删除尾部元素

    for (int i : vec) {
        std::cout << i << " ";
    }
    std::cout << std::endl;

    // 使用 std::deque
    std::deque<int> deq = {1, 2, 3};
    deq.push_back(4); // 在尾部插入元素
    deq.push_front(0); // 在头部插入元素
    deq.pop_back(); // 删除尾部元素
    deq.pop_front(); // 删除头部元素

    for (int i : deq) {
        std::cout << i << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

### 9、Vector如何进行扩容？

`std::vector` 的扩容过程如下：

1. 当需要向 `std::vector` 中添加新元素时，如果当前的内部数组已经满了（即已有的元素个数等于内部数组的容量），`std::vector` 将会申请一块新的内存空间来存储更多的元素。
2. 为了避免频繁的内存分配和释放操作，`std::vector` 通常会申请比当前容量更大的新内存块，例如通常会将容量扩大为当前容量的两倍。
3. 将原有的元素逐个复制到新的内存空间中。
4. 释放原来的内存空间。

这样一来，`std::vector` 就完成了扩容操作，并且可以继续向其中添加新元素了。由于扩容操作可能涉及内存分配和元素复制，因此在频繁插入大量元素时，可能会产生一定的性能开销。

给个例子，演示 `std::vector` 的扩容过程：

```
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec;
    std::cout << "Initial capacity: " << vec.capacity() << std::endl;

    for (int i = 0; i < 10; ++i) {
        vec.push_back(i);
        std::cout << "Size: " << vec.size() << ", Capacity: " << vec.capacity() << std::endl;
    }

    return 0;
}
```

### 10、无锁结构如何实现多线程？

无锁结构（Lock-Free Data Structures）是一种用于多线程环境的数据结构，它们的设计目标是在多线程并发访问时不使用传统的互斥锁机制，而是通过一些特殊的算法和技术来实现线程间的同步和数据访问的安全性。

无锁结构的实现通常基于原子操作（Atomic Operations）和内存屏障（Memory Barriers）等底层的硬件支持或操作系统提供的原语。这些操作能够保证在多线程并发执行时，对共享数据的操作能够以原子的方式进行，从而避免了使用传统的互斥锁带来的性能损失和死锁等问题。

1. 原子操作（Atomic Operations）：
   - 原子操作是一种不可分割的操作，可以在多线程并发执行时保证操作的原子性，即要么操作全部成功，要么全部失败，不会出现中间状态。
   - 在 C++11 及以上版本中，标准库提供了 `std::atomic` 类模板，可以用于实现原子操作。例如，`std::atomic<int>` 可以用于表示一个原子的整数操作。
2. 内存屏障（Memory Barriers）：
   - 内存屏障是一种硬件或者编译器提供的指令，用于控制内存访问的顺序和可见性，以确保多线程并发执行时的正确性。
   - 内存屏障可以保证在某个点之前的内存操作在某个点之后的内存操作之前完成，从而保证了内存操作的顺序性和可见性。
3. 无锁算法（Lock-Free Algorithms）：
   - 无锁算法是一种基于原子操作和内存屏障实现的算法，它们通常通过循环重试（Spin Loop）和 CAS 操作（Compare-And-Swap）等技术来实现数据结构的操作。
   - CAS 操作是一种原子操作，它可以检查某个内存位置的值是否等于预期值，如果相等，则将新值写入该位置；否则不做任何操作。这种操作可以用于实现无锁算法中的原子更新操作。
4. 无锁数据结构的实现：
   - 无锁数据结构的实现通常基于原子操作和 CAS 操作，例如无锁队列、无锁栈、无锁哈希表等。
   - 在实现无锁数据结构时，需要特别注意内存的可见性和操作的原子性，以确保多线程环境下数据结构的正确性和性能。

### 11、Gdb如何查看函数的堆栈情况？

在 GDB 中，你可以使用 `backtrace` 或其缩写 `bt` 命令来查看函数的堆栈情况。这个命令会显示当前函数的调用链，即调用栈（stack trace），显示当前位置及其调用关系。

要使用 `backtrace` 命令，首先需要在 GDB 中加载你的可执行文件，并在程序运行到需要查看堆栈情况的地方暂停。然后可以使用以下步骤来查看函数的堆栈情况：

1. 启动 GDB 并加载可执行文件：

```
gdb your_executable
```

2. 运行程序，直到需要查看堆栈情况的地方：

```
(gdb) run
```

3. 在程序暂停后，使用 `backtrace` 命令查看堆栈情况：

```
(gdb) backtrace
```

这个命令会输出当前函数调用栈的信息，包括当前位置以及调用链上的其他函数信息。如果想要更详细的信息，可以使用 `backtrace full` 命令来显示更多的信息，例如局部变量的值等。

另外，也可以使用 `frame` 命令来切换到不同的堆栈帧（stack frame），然后查看特定帧的信息。例如，使用 `frame 2` 命令可以切换到调用栈的第三层（从 0 开始计数），然后使用 `info locals` 命令可以查看该帧的局部变量信息。

### 12、硬连接和软连接有什么区别？ 

1. 硬链接（Hard Link）：
   - 硬链接是通过文件系统的 inode 来实现的，它将一个文件的 inode 与另一个文件名进行关联。
   - 硬链接创建后，它与原始文件具有相同的 inode 号，因此对于系统来说，无法区分哪一个是原始文件，哪一个是硬链接。
   - 由于硬链接与原始文件共享相同的 inode 和数据块，因此无论是原始文件还是硬链接，对其中一个文件的修改都会影响另一个文件，它们是同一个文件的不同名称。
   - 硬链接只能链接到同一文件系统中的文件，不能跨文件系统链接。
2. 软链接（Symbolic Link）：
   - 软链接是一个特殊的文件，它包含了指向另一个文件的路径，类似于 Windows 中的快捷方式。
   - 软链接与原始文件有不同的 inode 号和数据块，它只是一个指向原始文件的指针，因此修改软链接不会影响原始文件。
   - 软链接可以链接到任何文件系统中的文件，可以跨文件系统链接。

总结：

- 硬链接是文件系统中的一个实体，它与原始文件共享相同的 inode 和数据块，因此修改其中一个会影响另一个。
- 软链接是一个特殊的文件，它包含了指向另一个文件的路径，修改软链接不会影响原始文件。

### 13、Udp如何实现可靠传输？为什么不直接用tcp？

UDP（User Datagram Protocol）是一种无连接的、不可靠的传输协议，它提供了一种简单的数据传输机制，适用于一些对实时性要求较高，但对可靠性要求不高的应用场景，比如音频、视频等实时数据传输。

在 UDP 中实现可靠传输是一种比较复杂的任务，因为 UDP 本身不提供可靠性保证，它没有像 TCP 那样的重传机制、确认机制和流量控制机制。因此，如果需要在 UDP 上实现可靠传输，就需要在应用层自行实现这些机制，这可能会增加开发和维护的复杂性。

下面是一些可能用到的机制来实现在 UDP 上实现可靠传输：

1. 应用层重传机制：
   - 应用层可以通过在发送端设置定时器，在一定时间内未收到确认则进行重传，来实现数据的可靠传输。
   - 接收端收到数据后，可以发送确认消息给发送端，以通知发送端数据已经收到，如果发送端在一定时间内未收到确认，则可以进行重传。
2. 应用层流控制：
   - 应用层可以通过控制发送数据的速率，来避免发送端发送过快导致接收端无法处理。
3. 应用层数据校验：
   - 应用层可以在发送数据时附加校验和，接收端在接收数据时进行校验，以确保数据的完整性。

虽然可以在应用层实现类似于 TCP 的可靠传输机制，但是相比 TCP，UDP 的可靠传输机制更为复杂且需要更多的开发工作。因此，通常情况下，如果应用场景对数据的可靠性有较高的要求，而对实时性要求不是特别高，那么直接使用 TCP 更为简单和可靠。

### 14、Valgrind用了哪些组件？怎么从结果分析问题？

1. Memcheck：
   - Memcheck 是 Valgrind 最常用的组件，用于检测程序中的内存错误，比如使用未初始化的内存、读写越界等。
   - Memcheck 通过在运行时检测程序对内存的访问情况来发现内存错误，它会跟踪程序的每一个字节，以确保程序对内存的访问是合法的。
2. Cachegrind：
   - Cachegrind 是 Valgrind 的一个组件，用于模拟 CPU 缓存的行为，帮助分析程序的缓存使用情况。
   - Cachegrind 会记录程序执行过程中对缓存的读写情况，并统计缓存命中率、缓存行的使用情况等信息，从而帮助优化程序的性能。
3. Callgrind：
   - Callgrind 也是 Valgrind 的一个组件，用于分析程序的函数调用关系和函数执行时间。
   - Callgrind 会记录程序执行过程中的函数调用关系，并统计每个函数的执行时间、调用次数等信息，从而帮助优化程序的性能。
4. Helgrind：
   - Helgrind 是 Valgrind 的一个组件，用于检测多线程程序中的竞态条件和死锁问题。
   - Helgrind 会分析程序中的线程间同步操作，发现可能导致竞态条件和死锁的代码，从而帮助调试多线程程序。
5. Massif：
   - Massif 是 Valgrind 的一个组件，用于分析程序的堆内存使用情况。
   - Massif 会记录程序执行过程中的堆内存分配情况，并生成堆内存的使用情况图，从而帮助分析程序的内存泄漏和内存使用情况。

Valgrind 的使用通常需要结合具体的工具来进行，一般的步骤包括：

1. 运行 Valgrind：

   使用 Valgrind 工具集中的相应组件来运行需要分析的程序。

2. 分析结果：

   根据工具的输出结果，查找具体的问题，比如内存错误、内存泄漏、性能瓶颈等。

3. 修复问题：

   根据分析结果修复程序中的问题，比如修复内存错误、优化性能等。

4. 再次运行：

   修复问题后，再次运行 Valgrind 进行验证，确保问题已经解决。

5. 迭代优化：

   可以通过多次运行 Valgrind，并结合其他性能分析工具，不断优化程序的性能和可靠性。

### 15、如何让一个类在堆上生成？

先给个例子，然后根据例子展开讲讲。

```
#include <iostream>

class MyClass {
public:
    MyClass() {
        std::cout << "MyClass Constructor" << std::endl;
    }

    ~MyClass() {
        std::cout << "MyClass Destructor" << std::endl;
    }

    void doSomething() {
        std::cout << "Doing something..." << std::endl;
    }
};

int main() {
    // 在堆上动态分配一个 MyClass 对象
    MyClass* ptr = new MyClass();

    // 使用对象的成员函数
    ptr->doSomething();

    // 释放堆上的对象
    delete ptr;

    return 0;
}
```

示例中，首先定义了一个名为 `MyClass` 的类，其中包含了一个构造函数、一个析构函数和一个成员函数 `doSomething`。在 `main` 函数中，使用 `new` 运算符在堆上动态分配了一个 `MyClass` 对象，并将其地址赋值给指针 `ptr`。然后通过指针 `ptr` 调用了对象的成员函数 `doSomething`。最后，使用 `delete` 运算符释放了在堆上分配的对象。

在使用 `new` 运算符动态分配对象时，需要注意以下几点：

1. `new` 运算符返回的是指向动态分配对象的指针。
2. 动态分配的对象不会在作用域结束时自动销毁，需要手动使用 `delete` 运算符释放内存，否则会造成内存泄漏。
3. 使用 `delete` 运算符释放动态分配的对象后，应将指针置为 `nullptr`，以避免悬空指针的问题。

### 16、C++11 了解过多少？

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

### 17、Unique_ptr如何实现？

`std::unique_ptr` 是 C++11 引入的智能指针，用于管理动态分配的内存资源。它的特点是独占所有权，即同一时刻只能有一个 `unique_ptr` 拥有指向的对象，当 `unique_ptr` 被销毁时，它所指向的对象也会被自动释放。

`std::unique_ptr` 的实现主要依赖于移动语义（Move Semantics）和模板特化（Template Specialization）等特性。

unique_ptr实现：

```
template <typename T>
class unique_ptr {
public:
    // 构造函数
    explicit unique_ptr(T* ptr = nullptr) : ptr_(ptr) {}

    // 禁用拷贝构造和赋值运算符
    unique_ptr(const unique_ptr&) = delete;
    unique_ptr& operator=(const unique_ptr&) = delete;

    // 移动构造和移动赋值运算符
    unique_ptr(unique_ptr&& other) noexcept : ptr_(other.ptr_) {
        other.ptr_ = nullptr;
    }
    unique_ptr& operator=(unique_ptr&& other) noexcept {
        if (this != &other) {
            delete ptr_;
            ptr_ = other.ptr_;
            other.ptr_ = nullptr;
        }
        return *this;
    }

    // 析构函数
    ~unique_ptr() {
        delete ptr_;
    }

    // 获取指针
    T* get() const {
        return ptr_;
    }

    // 重载解引用操作符
    T& operator*() const {
        return *ptr_;
    }

    // 重载箭头操作符
    T* operator->() const {
        return ptr_;
    }

    // 释放指针
    T* release() {
        T* temp = ptr_;
        ptr_ = nullptr;
        return temp;
    }

    // 重置指针
    void reset(T* ptr = nullptr) {
        if (ptr != ptr_) {
            delete ptr_;
            ptr_ = ptr;
        }
    }

private:
    T* ptr_;  // 指向动态分配的内存资源的指针
};
```

### 18、移动语义是什么？举例说明？

移动语义（Move Semantics）是 C++11 引入的一种语义，用于优化对象的拷贝和赋值操作。传统的拷贝操作会对对象进行深拷贝，即复制对象的所有成员变量，这可能会导致资源的重复分配和复制，效率较低。而移动语义则通过转移对象的资源所有权来避免这种复制，提高了对象的性能和效率。

移动语义的实现依赖于右值引用（Rvalue Reference）和移动构造函数（Move Constructor）以及移动赋值运算符（Move Assignment Operator）。右值引用是一种新的引用类型，使用 `&&` 表示，它可以绑定到临时对象和右值表达式，表示这些对象在语义上是“可以被移动”的。

举个例子：

```
#include <iostream>
#include <vector>

class MyObject {
public:
    MyObject() {
        std::cout << "Constructor" << std::endl;
    }

    ~MyObject() {
        std::cout << "Destructor" << std::endl;
    }

    // 移动构造函数
    MyObject(MyObject&& other) noexcept {
        std::cout << "Move Constructor" << std::endl;
        // 转移资源所有权
    }

    // 移动赋值运算符
    MyObject& operator=(MyObject&& other) noexcept {
        std::cout << "Move Assignment Operator" << std::endl;
        if (this != &other) {
            // 释放自身资源
            // 转移资源所有权
        }
        return *this;
    }
};

int main() {
    // 创建一个临时对象
    MyObject obj;

    // 创建一个 vector，并将 obj 放入其中
    std::vector<MyObject> vec;
    vec.push_back(std::move(obj)); // 转移 obj 的资源所有权

    return 0;
}
```

示例中，`MyObject` 类定义了移动构造函数和移动赋值运算符，它们负责将对象的资源所有权从一个对象转移到另一个对象。在 `main` 函数中，创建了一个临时对象 `obj`，然后将它通过 `std::move` 转换为右值，从而可以调用 `std::vector` 的 `push_back` 函数进行移动插入操作，避免了不必要的拷贝。

### 19、说几个常用的设计模式？设计模式的优点与缺点

设计模式今天就不讲了，前几篇有几个讲的非常详细了，可以看看。

### 20、为什么有人说单例模式不应该算作一种设计模式？

可能的原因是单例模式在一些情况下可能会导致代码耦合性增加、单元测试困难、全局状态难以追踪等问题，同时过度使用单例模式可能会导致代码可维护性和可测试性降低。以下是一些人们对单例模式不应算作设计模式的主要观点：

1. 隐藏了依赖关系：

   单例模式隐藏了类的依赖关系，使得代码的依赖关系难以追踪和理解。因为单例模式通过静态方法获取实例，而不是通过构造函数传递依赖。

2. 全局状态：

   单例模式会引入全局状态，可能会导致程序的可测试性下降。全局状态使得单元测试变得困难，因为测试用例可能会相互影响，导致测试结果不确定。

3. 硬编码：

   单例模式通常是通过静态方法获取实例，这种方式在代码中硬编码了对单例类的依赖，使得代码的灵活性降低。

4. 破坏了封装性：

   单例模式可能会破坏类的封装性，因为它通过静态方法提供了对实例的全局访问权限，而不是通过类的接口来控制访问。

5. 过度使用：

   有些人认为，单例模式在一些情况下被过度使用，导致了代码的可维护性和可测试性下降。他们认为应该更多地考虑其他设计模式或者设计原则来解决问题，而不是过度依赖单例模式。

虽然有人对单例模式的使用持有上述观点，但是单例模式在一些特定的场景下仍然是有用的，比如需要确保只有一个实例存在的情况下。

### 21、情景题：略