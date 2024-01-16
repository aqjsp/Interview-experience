来源：https://www.nowcoder.com/discuss/557314429990207488?sourceSSR=users

## 二面 10.19（50min）

1、自我介绍

2、项目创新点

3、项目遇到的难题

4、项目二

5、压缩算法怎么实现的

6、实习

7、长视频如何切分为短视频

8、论文

9、C++11新特性

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

10、右值？

- 左值（Lvalue）： 指向内存位置的表达式，有明确的标识和持久的状态。左值通常是可以取地址的、有名字的变量或对象。
- 右值（Rvalue）： 临时或即将被销毁的表达式，没有明确的标识和持久的状态。右值通常是临时对象、字面常量、表达式的计算结果等。

C++11引入了右值引用（Rvalue Reference）的概念，主要用于实现移动语义和完美转发。右值引用使用 `&&` 符号表示。

特点：

1. 可以修改的左值和不可修改的左值是左值：

   例如：`int x = 5;` 中，`x` 是一个可以修改的左值。

2. 临时对象、字面常量、表达式的计算结果是右值：

   例如：`int result = 2 + 3;` 中，`2 + 3` 是一个右值。

3. 右值引用可以绑定到右值：

   通过右值引用，可以对右值进行引用，允许进行移动语义和避免不必要的复制。

给个例子：

```C++
#include <iostream>

void processInt(int& lref) {
    std::cout << "Lvalue reference" << std::endl;
}

void processInt(int&& rref) {
    std::cout << "Rvalue reference" << std::endl;
}

int main() {
    int x = 5; // x 是左值
    processInt(x); // 调用 lvalue reference 版本

    processInt(10); // 调用 rvalue reference 版本

    return 0;
}
```

11、虚函数机制？

C++中的虚函数机制是为了实现多态性（Polymorphism）的一种机制。在C++中，通过使用虚函数，可以在运行时确定调用的是哪个版本的函数，从而实现动态绑定。

以下是C++虚函数机制的基本原理和使用方式：

1. 虚函数和虚表： 在含有虚函数的类中，会生成一个虚函数表（Virtual Table，简称 vtable）。虚函数表是一个指针数组，其中存储了该类的虚函数的地址。每个对象的实例都包含一个指向虚函数表的指针，这个指针称为虚表指针（vptr）。
2. 虚函数的声明和定义： 在基类中，使用关键字 `virtual` 声明虚函数。派生类可以选择覆盖（override）基类的虚函数。虚函数的实现会放在虚函数表中，而不是直接在类定义中。

```C++
class Base {
public:
    virtual void print() const {
        std::cout << "Base class" << std::endl;
    }
};

class Derived : public Base {
public:
    void print() const override {
        std::cout << "Derived class" << std::endl;
    }
};
```

动态绑定： 当通过基类指针或引用调用虚函数时，实际调用的是对象的动态类型所对应的虚函数。这种在运行时决定调用哪个函数的机制称为动态绑定。在下面的例子中，通过 `print` 函数调用时，根据对象的实际类型来调用相应的虚函数。

```C++
void printInfo(const Base& obj) {
    obj.print(); // 动态绑定，根据实际类型调用相应的虚函数
}

int main() {
    Base baseObj;
    Derived derivedObj;

    printInfo(baseObj);    // 调用 Base 类的虚函数
    printInfo(derivedObj); // 调用 Derived 类的虚函数

    return 0;
}
```

通过虚函数，可以实现在基类指针或引用的情况下，根据实际对象的类型来调用相应的函数，实现了多态性。

12、单继承和虚继承虚表的结构？

1. 单继承情况下的虚表结构：

当一个类使用单继承时，每个类都有自己的虚表。子类的虚表包含了从基类继承的虚函数以及自身新增的虚函数。在对象的内存布局中，通常会有一个指向该类虚表的虚表指针（vptr）。

```C++
class Base {
public:
    virtual void funcBase() { /* Base class implementation */ }
};

class Derived : public Base {
public:
    virtual void funcDerived() { /* Derived class implementation */ }
};
```

内存布局示意：

![image-20231220001807342](E:\GitHub\Interview-experience\小米面经\assets\image-20231220001807342.png)

2. 虚继承情况下的虚表结构：

当一个类使用虚继承时，主要是为了解决多重继承可能导致的菱形继承问题。在虚继承中，虚表的结构会更为复杂。基类的虚表包含了从所有虚继承的路径继承的虚函数，而子类的虚表只包含自己新增的虚函数。

```C++
class Base {
public:
    virtual void funcBase() { /* Base class implementation */ }
};

class VirtualBase {
public:
    virtual void funcVirtualBase() { /* VirtualBase class implementation */ }
};

class Derived : public virtual Base, public virtual VirtualBase {
public:
    virtual void funcDerived() { /* Derived class implementation */ }
};
```

内存布局示意：

![image-20231220001817547](E:\GitHub\Interview-experience\小米面经\assets\image-20231220001817547.png)

在虚继承中，基类的虚表中包含了虚基类的虚函数，而子类的虚表只包含自身新增的虚函数。虚基类的数据成员只在最终的派生类中存在一份，解决了菱形继承可能导致的二义性问题。

13、纯虚函数是什么?

在C++中，纯虚函数是一种在基类中声明但没有定义的虚函数。这类函数通过在函数声明的结尾添加 "= 0" 来声明为纯虚函数。纯虚函数在基类中没有实际的实现，而是由派生类负责实现。类含有纯虚函数的类称为抽象类。

纯虚函数的语法示例：

```C++
class AbstractBase {
public:
    // 纯虚函数，没有实现
    virtual void pureVirtualFunction() = 0;

    // 普通虚函数，有默认实现
    virtual void virtualFunctionWithDefault() {
        // 默认实现
    }
    
    // 普通函数
    void regularFunction() {
        // 函数实现
    }
};
```

但是要注意的是，包含纯虚函数的类无法实例化对象，因为它包含了未实现的虚函数。抽象类的主要作用是提供一个接口，而具体的实现则由派生类完成。当一个类包含纯虚函数时，它变成了一个抽象基类，其他类可以从它派生，实现其虚函数以创建具体的对象。

14、extern C？

`extern "C"` 是一个用于告诉 C++ 编译器在链接时如何处理函数名和变量名的指令。当 C++ 程序调用其他语言（通常是 C 语言）编写的代码时，由于 C++ 支持函数重载和命名空间等特性，会导致函数名在编译后发生改变。为了正确链接这些外部函数和变量，需要使用 `extern "C"` 来告诉编译器按照 C 语言的方式处理函数名。

具体来说，`extern "C"` 的作用有两个：

1. 禁止名称修饰（Name Mangling）： C++ 编译器会对函数和变量的名称进行修饰，以包含类型信息、命名空间等，从而支持函数重载等特性。而 C 语言不支持这些特性，因此 C++ 编译器会对 C 语言的函数和变量使用一种不带修饰的方式。使用 `extern "C"` 可以告诉编译器，按照 C 语言的方式处理函数名，使得 C++ 能够正确链接 C 语言编写的代码。
2. 指定函数调用规约： 在某些平台上，C++ 和 C 的函数调用规约可能有所不同，包括参数传递、栈清理等。使用 `extern "C"` 可以确保 C++ 函数按照 C 语言的规约进行编译，以便正确调用 C 语言的函数。

给个例子：

```C++
#ifdef __cplusplus
extern "C" {
#endif

// 声明一个 C 语言风格的函数
void cStyleFunction();

#ifdef __cplusplus
}
#endif
```

15、deque的底层实现？

`deque`（双端队列）的底层实现通常是由一系列块（block）组成的，每个块是一个固定大小的数组，用来存储元素。`deque` 以块为单位分配和管理内存，因此可以高效地在两端执行插入和删除操作。

以下是 `deque` 的一般实现细节：

1. 块结构： `deque` 由多个块组成，每个块是一个固定大小的数组。这些块被链接在一起，形成一个双向链表。每个块通常包含若干个元素，这个数量由实现确定。
2. 索引维护： 为了支持在两端高效地插入和删除操作，`deque` 需要维护指向首尾块的索引。同时，每个块也会记录自己的首尾元素的索引。这样，对于在两端的插入和删除，只需要对相应的索引进行调整，而不需要移动整个块内的元素。
3. 块的动态分配： 当 `deque` 需要更多的存储空间时，会动态地分配新的块。这样，`deque` 可以灵活地扩展，而不需要提前知道存储元素的总数。
4. 迭代器： `deque` 的迭代器需要能够跨越块边界，因此需要对块的指针和元素的索引进行适当的管理。迭代器通常会包含指向当前块和元素索引的指针。

16、什么时候使用static？

1. 静态变量：
   - 在函数内部使用 `static` 关键字声明的变量是静态变量。静态变量的生命周期贯穿整个程序运行期间，而不是只在函数调用时存在。
   - 静态变量的初始化只在第一次进入声明它的函数时进行，之后的函数调用不会重新初始化。
   - 静态变量存储在全局数据区，而不是存储在栈上。

```C++
void exampleFunction() {
    static int staticVar = 0; // 静态变量
    // 其他代码
}
```

2. 静态函数：

   - 在类内声明的函数可以使用 `static` 关键字来定义为静态函数。静态函数不属于类的实例，可以直接通过类名调用，而不需要创建类的对象。

   - 静态成员函数不能访问非静态成员变量和非静态成员函数。

```C++
class MyClass {
public:
    static void staticFunction() {
        // 静态函数的实现
    }
};
```

3. 静态成员变量：

   - 在类中声明的 `static` 成员变量是属于类的，而不是属于类的实例。所有类的实例共享同一个静态成员变量。

   - 静态成员变量需要在类外部进行定义和初始化。

```C++
class MyClass {
public:
    static int staticVar; // 静态成员变量声明
};

// 静态成员变量定义和初始化
int MyClass::staticVar = 0;
```

4. 静态全局变量：

   - 在函数外部使用 `static` 关键字声明的变量是静态全局变量。它的作用域仅限于定义它的文件，对其他文件不可见。

   - 静态全局变量的生命周期贯穿整个程序运行期间。

```C++
// 文件1.cppstatic 
int staticGlobalVar = 0;

// 文件2.cpp（无法访问 staticGlobalVar）
```

17、内联的特点，优缺点？

内联函数是一种在编译时将函数调用处直接展开为函数体的机制，而不是通过函数调用的方式执行。

特点：

1. 代码展开： 编译器会将内联函数的代码直接插入到每个调用该函数的地方，而不是创建一个函数调用的框架。
2. 关键字： 使用 `inline` 关键字声明内联函数。
3. 小而简单： 适合用于小而简单的函数，因为展开的代码会增加。

优点：

1. 减少函数调用开销： 由于内联函数在调用处展开，避免了函数调用的开销，减少了函数入栈、出栈以及参数传递的时间。
2. 提高执行速度： 对于短小的函数，内联可以减少函数调用的开销，从而提高执行速度。
3. 避免函数调用开销： 在某些情况下，函数调用可能会引入一定的开销，内联可以避免这些开销。

缺点：

1. 代码膨胀： 内联会导致代码膨胀，因为每个调用点都会复制一份函数的代码，可能导致可执行文件变得更大。
2. 可能降低缓存效率： 内联增加了代码的大小，可能导致缓存未命中率增加，影响程序执行效率。
3. 适用场景受限： 内联适用于简单的函数，在复杂的函数中使用内联可能导致代码难以维护。

适用场景：

- 内联适合用于简短、频繁调用的函数，例如访问器函数、简单的 getter 和 setter 函数等。
- 不适合用于复杂且体积较大的函数，因为这样可能导致代码膨胀，降低了缓存效率。

18、指针常量和常量指针？

1. 指针常量：

- 定义：指针常量是指一个指针，它指向的内容是常量，即不能通过该指针修改所指向的数据。
- 语法：`const int *ptr;` 或 `int const *ptr;`

示例：

```C++
int value = 10;
const int *ptr = &value;  // ptr是指针常量，不允许通过ptr修改value的值
```

2. 常量指针（Constant Pointer）：

- 定义：常量指针是指一个指针，它本身是常量，即不能通过该指针修改指针的指向。
- 语法：`int *const ptr;`

示例：

```C++
int value = 10;
int *const ptr = &value;  // ptr是常量指针，不允许修改ptr指向的地址
```

- 在指针常量的示例中，`ptr` 是一个指向整数常量的指针，不能通过 `ptr` 修改 `value` 的值，但可以通过其他途径修改 `value`。
- 在常量指针的示例中，`ptr` 是一个常量指针，不能修改 `ptr` 指向的地址，但可以通过 `ptr` 修改 `value` 的值。

19、重载和重写？

函数重载（Function Overloading）：

- 定义： 函数重载是指在同一个作用域内，定义了多个具有相同名称但参数列表不同的函数。
- 特点： 重载函数具有相同的名称但不同的参数类型、参数个数或参数顺序。
- 示例：

```C++
// 重载函数
int add(int a, int b) {
    return a + b;
}

double add(double a, double b) {
    return a + b;
}
```

方法重写（Method Overriding）：

- 定义： 方法重写是指在派生类中重新实现（覆盖）其基类中已经定义的方法。
- 特点： 重写的方法在基类和派生类中有相同的名称、相同的参数列表和相同的返回类型。
- 示例：

```C++
// 基类
class Shape {
public:
    virtual void draw() {
        // 基类中的绘制方法
    }
};

// 派生类
class Circle : public Shape {
public:
    void draw() override {
        // 派生类中重写的绘制方法
    }
};
```

总的来说：

- 函数重载关注于同一作用域内多个同名函数的定义，通过参数的不同来区分。
- 方法重写关注于派生类中重新定义基类中已有的虚函数，以实现特定的行为。

20、typedef和define的区别？

1. 作用域不同：

   - `typedef` 创建一个类型别名，而且这个别名是在类型的作用域内的，可以限制在特定的作用域。
   - `#define` 创建的是一个预处理宏，它的作用域是整个程序，没有局部作用域的概念。

2. 类型安全：

   - `typedef` 是类型安全的，它只能用于定义新类型名。
   - `#define` 不是类型安全的，它可以用于定义任何文本替换，包括常量、表达式等，可能导致不同类型的错误。

3. 语法形式：

   - `typedef` 的语法是：`typedef 原类型 新类型名;`
   - `#define` 的语法是：`#define 别名 值`

4. 编译时检查：

   - `typedef` 在编译时进行类型检查，如果尝试给已定义的类型名赋予不同类型的值，编译器会发出错误。
   - `#define` 是简单的文本替换，没有编译时类型检查。

5. 宏的额外功能：

   `#define` 可以定义更复杂的宏，可以包含参数、条件语句等。这使得 `#define` 更加强大，但也更容易出现错误。

6. 对模板的支持：

   - `typedef` 支持模板，可以用于为模板类型创建别名。
   - `#define` 不支持模板，因为它是简单的文本替换。

示例：

```C++
// 使用 typedef 创建类型别名
typedef int Integer;

// 使用 #define 创建常量
#define MAX_VALUE 100

// 使用 #define 创建宏
#define SQUARE(x) ((x) * (x))
```

21、C++数据类型转换？

1. 静态转换（Static Cast）:

   - 使用 `static_cast<目标类型>(表达式)` 进行转换。

   - 主要用于相关类型之间的转换，例如基本数据类型之间的转换，父子类之间的指针或引用转换。静态转换在编译时进行，不提供运行时类型检查。

```C++
double myDouble = 3.14;
int myInt = static_cast<int>(myDouble);
```

2. 动态转换（Dynamic Cast）:

   - 使用 `dynamic_cast<目标类型>(表达式)` 进行转换。

   - 主要用于具有多态性质的类之间的指针或引用转换，提供运行时类型检查。只能用于含有虚函数的类。

```C++
class Base {
    virtual void foo() {}
};

class Derived : public Base {};

Base* basePtr = new Derived;
Derived* derivedPtr = dynamic_cast<Derived*>(basePtr);
```

3. 常量转换（Const Cast）:

   - 使用 `const_cast<目标类型>(表达式)` 进行转换。

   - 主要用于添加或移除 `const`、`volatile` 属性，但并不会真正改变对象的常量性。常量转换通常用于解决一些特殊情况下的编译错误。

```C++
const int myConst = 42;
int* myPtr = const_cast<int*>(&myConst);
```

4. 重新解释转换（Reinterpret Cast）:

   - 使用 `reinterpret_cast<目标类型>(表达式)` 进行转换。

   - 主要用于不同类型之间的指针或引用转换，通常是比较底层的转换，不进行类型安全检查，慎用。

```C++
int myInt = 42;
double* myDoublePtr = reinterpret_cast<double*>(&myInt);
```

22、拷贝构造可以为虚么？

拷贝构造函数不能被声明为虚函数。拷贝构造函数是用于创建对象的一个新实例，通常是通过将已存在的对象的值复制到新对象来完成的。虚函数的主要特性是能够在派生类中被覆盖，而拷贝构造函数并不具备被派生类覆盖的需求。

虚函数在运行时通过虚表（vtable）来实现动态绑定，这样可以在运行时确定调用的是基类还是派生类的函数。而拷贝构造函数是在对象创建时自动调用的，不需要在运行时选择合适的函数版本，因此不需要声明为虚函数。

虚函数的声明和使用通常涉及到运行时多态性和动态绑定，而拷贝构造函数的目的是复制对象，不需要在运行时选择不同的版本。将拷贝构造函数声明为虚函数会导致编译错误。如果有需要在复制时执行特定的逻辑，可以考虑使用其他方法，例如在基类中定义一个虚函数，然后在派生类中实现该虚函数，并在拷贝构造函数中调用该虚函数。

23、进程间通信方式？

1. 管道（Pipes）： 管道是一种半双工通信方式，主要用于具有亲缘关系的进程之间的通信。管道是一种线性数据流，数据只能单向流动。它有两种类型：
   - 无名管道（Anonymous Pipes）： 通常用于父子进程之间的通信。创建管道使用 `pipe()` 系统调用。
   - 命名管道（Named Pipes）： 用于无关联的进程之间的通信，它们以文件系统中的命名管道文件形式存在。
2. 消息队列（Message Queues）： 消息队列允许进程通过消息进行异步通信。消息队列允许多个进程通过将消息发送到队列，然后其他进程从队列中接收消息来进行通信。消息队列通常有操作系统提供的 API 来管理消息的发送和接收。
3. 共享内存（Shared Memory）： 共享内存是一种高效的通信方式，允许多个进程共享相同的物理内存区域。这使得数据在进程之间的传输非常快速，因为它们可以直接读写相同的内存。然而，共享内存需要进行同步以避免数据竞争。
4. 信号（Signals）： 信号是异步通信的一种方式，用于通知进程某些事件的发生，如错误或异常。每个信号都有一个数字标识符，当事件发生时，进程可以注册信号处理程序来处理信号。
5. 套接字（Sockets）： 套接字是一种用于网络通信的通用通信机制，但也可以在同一台计算机上的不同进程之间使用。套接字提供了面向流和面向数据报的通信方式，允许进程通过网络套接字进行通信。
6. 文件（File）： 进程可以通过读写文件来实现通信。一个进程可以将数据写入文件，而另一个进程则可以读取该文件的内容。这种方式不够高效，但是可以应用在不同进程之间的通信需求较少的情况下。
7. 信号量（Semaphores）： 信号量是一种用于控制多个进程对共享资源的访问的同步机制。信号量可以用于避免竞争条件，确保一次只有一个进程可以访问共享资源。
8. 共享文件映射（Memory-Mapped Files）： 共享文件映射允许进程将文件映射到它们的地址空间中，以便多个进程可以访问相同的文件数据。这在共享大量数据时非常有用。

24、虚拟内存有哪几种实现方式？

1. 分页（Paging）： 将物理内存和虚拟内存划分成固定大小的页（通常为4KB）。操作系统将进程的虚拟地址空间分成相同大小的页，当进程访问某个虚拟地址时，通过页表将虚拟地址映射到物理内存中的页框。如果页面不在物理内存中，就会发生页面错误，操作系统负责将相应的页面从磁盘加载到物理内存。
2. 分段（Segmentation）： 将进程的地址空间划分成若干个段，每个段代表一类数据或代码，具有不同的属性。段的大小可以动态变化。分段提供了更灵活的地址空间管理，但也增加了管理的复杂性。
3. 页表（Page Tables）： 使用页表将虚拟地址映射到物理地址。在分页系统中，页表的作用是存储虚拟页和物理页的映射关系。在分段系统中，页表用于存储段的信息，包括段的起始地址和长度等。
4. 段页式（Segmentation with Paging）： 结合了分页和分段两种方式，可以同时享受它们的优点。地址空间先按段划分，每个段内再按页进行管理。

25、研究生课程

26、拥塞控制和流量控制？

1. 拥塞控制：
   - 定义：拥塞控制是一种网络管理机制，用于防止网络中的拥塞，确保网络性能的稳定和高效。
   - 目标：避免网络过载，防止网络中的路由器和链路被过度利用，从而导致网络性能下降和数据包丢失。
   - 方法：拥塞控制使用一系列算法和策略，包括减小发送速率、丢弃或延迟数据包、动态选择路由等，以适应网络流量的变化。
2. 流量控制：
   - 定义：流量控制是为了协调发送者和接收者之间的数据流，防止接收者处理不过来而导致数据丢失或网络拥塞。
   - 目标：确保接收者能够有效处理接收到的数据，防止发送者发送过多数据导致接收者缓冲区溢出。
   - 方法：通常通过反馈机制实现，接收者向发送者发送反馈信息，告知其当前的处理能力，发送者根据这些信息动态调整发送速率。

关键区别：

- 拥塞控制主要关注整个网络，防止网络过载和性能下降，是一种全局性的机制。
- 流量控制关注的是发送者和接收者之间的通信，确保发送者不会发送太多数据，以适应接收者的处理能力，是一种点对点的机制。

27、线程的私有和共享资源

1. 私有资源：
   - 每个线程都有自己的私有资源，这些资源在每个线程中都是独立的，不会被其他线程访问。
   - 私有资源包括线程的栈空间、寄存器值、程序计数器（Program Counter）等。这些资源对于每个线程都是独立的，不同线程之间互不影响。
2. 共享资源：
   - 共享资源是多个线程共同访问和操作的资源，可能包括全局变量、堆内存、文件、网络连接等。
   - 对于共享资源的访问需要进行同步控制，以防止多个线程同时修改同一资源导致数据不一致或其他问题。

需要注意的是：

- 私有资源的特点：每个线程拥有自己的一份，不同线程之间互不干扰。
- 共享资源的问题：多个线程同时访问共享资源可能引发竞争条件（Race Condition）、死锁（Deadlock）等并发编程中常见的问题。
- 同步控制：为了保证对共享资源的安全访问，需要使用同步机制，如互斥锁、信号量、条件变量等。

示例：

```C++
#include <iostream>
#include <thread>

// 共享资源
int sharedVariable = 0;

// 线程私有资源
thread_local int threadLocalVariable = 0;

// 线程函数，访问共享资源和线程私有资源
void threadFunction() {
    // 访问共享资源
    sharedVariable++;
    
    // 访问线程私有资源
    threadLocalVariable++;

    // 打印结果
    std::cout << "Shared Variable: " << sharedVariable << ", Thread Local Variable: " << threadLocalVariable << std::endl;
}

int main() {
    // 创建两个线程
    std::thread t1(threadFunction);
    std::thread t2(threadFunction);

    // 等待线程结束
    t1.join();
    t2.join();

    return 0;
}
```

28、野指针是什么、怎么避免？

野指针是指指向已经释放的内存或者未初始化的内存地址的指针。在访问野指针时，由于它指向的内存区域可能已经被其他程序或者当前程序的其他部分使用，可能导致程序崩溃、数据错误等不可预测的行为。

一些避免野指针的方法：

1. 初始化指针： 在定义指针时，尽量立即将其初始化为 `nullptr` 或者 `NULL`，避免使用未初始化的指针。

```C++
int* ptr = nullptr;  // 或 int* ptr = NULL;
```

2. 在释放指针后将其置为 `nullptr`： 在使用 `delete` 或 `free` 释放内存后，将指针置为 `nullptr`，以避免成为野指针。

```C++
delete ptr;
ptr = nullptr;
```

3. 避免悬挂指针： 在函数结束时，局部变量的指针会被销毁，如果其指向堆内存，需要注意不要返回该指针，否则可能导致悬挂指针问题。

```C++
int* createAndReturnPointer() {
    int* ptr = new int;
    // 不要返回 ptr，因为在函数结束后，ptr 将成为悬挂指针
    return ptr;
}
```

4. 使用智能指针： C++11 引入的智能指针（如 `std::unique_ptr`、`std::shared_ptr`）可以帮助自动管理内存，降低野指针的风险。

```C++
#include <memory>

std::unique_ptr<int> ptr = std::make_unique<int>(42);
```

5. 谨慎使用裸指针： 尽量使用智能指针而非裸指针，特别是在管理动态分配的内存时。智能指针会在适当的时候自动释放资源，降低了手动管理内存的错误可能性。

29、做题，三数之和？

思路：

可以利用双指针法来查找三元组。首先，我们对数组 `S` 进行排序。然后，我们可以使用两层循环遍历数组，以第一个循环中的元素作为固定值 `a`，并使用双指针来查找满足 `b + c = -a` 的 `b` 和 `c`。

具体步骤：

1. 首先对数组 `S` 进行排序，以便在查找过程中使用双指针。
2. 固定一个元素 `a`，从左到右遍历数组。
3. 使用双指针法在当前 `a` 的右侧查找两个元素 `b` 和 `c`，使得 `b + c = -a`。这需要将左指针指向 `b` 的位置，右指针指向 `c` 的位置。
4. 在双指针的过程中，需要注意去重处理，避免出现重复的三元组。
5. 当找到一个满足条件的三元组时，将它添加到结果中。
6. 继续遍历数组，继续寻找下一个 `a` 值的三元组。

参考代码：

```C++
#include <iostream>
#include <vector>
#include <algorithm>

std::vector<std::vector<int>> threeSum(std::vector<int>& nums) {
    std::vector<std::vector<int>> result;
    int n = nums.size();
    if (n < 3) {
        return result; // 如果数组长度小于 3，无法组成三元组
    }
    
    std::sort(nums.begin(), nums.end()); // 先对数组排序
    
    for (int i = 0; i < n - 2; i++) {
        if (i > 0 && nums[i] == nums[i - 1]) {
            continue; // 避免重复固定值
        }
        
        int a = nums[i]; // 固定第一个值 a
        
        int left = i + 1; // 左指针，指向第二个值
        int right = n - 1; // 右指针，指向最后一个值
        
        while (left < right) {
            int b = nums[left]; // 第二个值 b
            int c = nums[right]; // 第三个值 c
            int sum = a + b + c;
            
            if (sum == 0) {
                result.push_back({a, b, c}); // 找到一个满足条件的三元组
                while (left < right && nums[left] == b) {
                    left++; // 避免重复的第二个值
                }
                while (left < right && nums[right] == c) {
                    right--; // 避免重复的第三个值
                }
            } else if (sum < 0) {
                left++; // 总和小于 0，移动左指针
            } else {
                right--; // 总和大于 0，移动右指针
            }
        }
    }
    
    return result;
}

int main() {
    std::vector<int> nums = {-1, 0, 1, 2, -1, -4};
    std::vector<std::vector<int>> result = threeSum(nums);
    
    std::cout << "满足条件的三元组为：" << std::endl;
    for (const auto& triple : result) {
        std::cout << "[";
        for (int i = 0; i < triple.size(); i++) {
            std::cout << triple[i];
            if (i < 2) {
                std::cout << ", ";
            }
        }
        std::cout << "]" << std::endl;
    }
    
    return 0;
}
```