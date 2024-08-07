# 金山C++一面

来源：https://www.nowcoder.com/feed/main/detail/8f4ef378972d411cb6961255b58a560e

### 1、静态变量生命周期和普通变量比较？

1. 生命周期：
   - 普通变量（自动变量）： 自动变量的生命周期与其所在的代码块（作用域）相关。它们在离开其定义的作用域时被销毁，通常是栈上分配的。
   - 静态变量（静态存储期变量）： 静态变量的生命周期在整个程序运行期间，它们在程序启动时分配内存，在程序结束时才会被释放。静态变量可以存储在全局存储区或静态存储区。
2. 作用域：
   - 普通变量： 普通变量的作用域通常仅限于定义它们的代码块，它们不能被其他代码块访问。
   - 静态变量： 静态变量可以具有更广泛的作用域，例如，如果它们在全局范围内声明，它们可以被整个程序访问。
3. 初始化：
   - 普通变量： 普通变量在声明时不会自动初始化，除非显式赋初值。
   - 静态变量： 静态变量在定义时如果不显式初始化，会自动初始化为零（对于基本数据类型）或空（对于类对象）。
4. 存储位置：
   - 普通变量： 普通变量通常分配在栈上，每次函数调用时都会创建新的实例。
   - 静态变量： 静态变量可以存储在全局存储区或静态存储区。全局变量在程序启动时初始化，它们的值在程序的整个生命周期内都可用

### 2、什么时候创建虚函数表？

1. 虚函数的定义： 虚函数是通过在基类（父类）中声明的。虚函数的定义包括返回类型、函数名和参数列表。

```C++
class Base {
public:
    virtual void foo() { /* Base class implementation */ }
};
```

2. 派生类中的重写： 派生类（子类）可以重写虚函数，这意味着它可以提供自己的实现版本。在派生类中重新定义虚函数时，需要使用 `virtual` 关键字来保持虚函数性质。

```C++
class Derived : public Base {
public:
    virtual void foo() override { /* Derived class implementation */ }
};
```

3. 虚函数表的创建： 虚函数表在编译时由C++编译器自动生成。对于每个具有虚函数的类，编译器会生成一个虚函数表，其中包含了该类的虚函数指针。这个虚函数表通常位于类的内部，它是静态的，一旦创建就不会更改。

4. 虚函数表的指针： 对于每个包含虚函数的类，编译器会在类的内部添加一个指向虚函数表的指针，通常位于对象的内存布局的最前面。这个指针被称为虚函数表指针（vptr）。

5. 运行时动态绑定： 当你通过基类指针或引用调用虚函数时，实际执行的是派生类中的版本（如果派生类重新定义了虚函数）。这是因为虚函数表指针（vptr）指向了正确的虚函数表，允许在运行时根据对象的类型进行动态绑定。

```C++
Base* ptr = new Derived;
ptr->foo(); // 调用Derived类中的foo()
```

### 3、虚函数指针会不会变，什么时候初始化，在析构里会不会变，析构函数能访问虚函数吗？

1. 初始化时创建： 虚函数表指针在对象创建时就被初始化。当你创建一个类的实例（对象）时，其中包括一个指向该类虚函数表的虚函数表指针。这个指针是在对象构造过程中初始化的。
2. 不会在析构函数中改变： 虚函数表指针通常在对象的整个生命周期内保持不变。这意味着即使在对象的析构函数中，虚函数表指针也不会改变。析构函数是用于对象销毁的，不负责改变虚函数表指针。
3. 虚函数的调用： 虚函数表指针的主要作用是支持运行时多态性。通过这个指针，程序可以在运行时查找并调用正确的虚函数版本。在析构函数中，你可以调用虚函数，但需要注意的是析构函数本身不会改变虚函数表指针。在析构函数中调用虚函数时，通常执行的是对象的类型，而不是基类的类型。
4. 注意虚函数表指针变化的情况： 在一些特殊情况下，虚函数表指针可能会变化。例如，当一个对象通过复制构造函数复制时，复制的对象将有自己的虚函数表指针。这通常发生在基类和派生类之间的复制。另外，如果你使用虚继承，虚函数表指针也可能会更改。

### 4、静态函数可以访问非静态成员变量吗为什么？

静态成员函数（静态方法）可以访问非静态成员变量。

1. 静态函数（静态成员函数）： 静态函数是与类关联，而不是与类的实例（对象）关联的函数。这意味着它不依赖于特定的对象，可以通过类名直接调用。静态函数通常用于执行与类本身相关的操作，而不涉及特定对象的状态。
2. 非静态成员变量： 非静态成员变量是与类的实例（对象）关联的变量。每个类的对象都有其自己的一组非静态成员变量，它们可以存储对象特定的状态信息。

现在来看静态函数访问非静态成员变量的情况：

- 可以访问静态成员变量： 静态函数可以自由地访问同一类的静态成员变量，因为这些成员变量与类相关，而不是与对象相关。在静态函数中，你可以使用类名或`this`指针（在类的范围内）来访问静态成员变量。
- 不能直接访问非静态成员变量： 静态函数不能直接访问特定对象的非静态成员变量。这是因为静态函数没有"this"指针，它不知道它应该关联到哪个对象的非静态成员变量。
- 需要对象实例： 如果静态函数需要访问特定对象的非静态成员变量，它必须通过传递对象实例作为参数或在函数内部创建对象实例，然后使用该对象实例来访问非静态成员变量。

示例：

```C++
class MyClass {
public:
    int nonStaticVar; // 非静态成员变量

    static void StaticFunction(MyClass& obj) {
        obj.nonStaticVar = 42; // 通过对象实例访问非静态成员变量
    }
};
```

### 5、编译实现重载？

函数重载（Function Overloading）是一种允许你定义多个同名函数，但它们具有不同的参数列表的机制。编译器会根据不同的参数列表来决定调用哪个重载函数。函数重载使你能够使用相同的函数名来处理不同类型的数据或不同数量的参数，从而提高了代码的可读性和复用性。

1. 函数名称相同： 在函数重载中，你可以定义多个具有相同名称的函数，但它们的参数列表不同。
2. 参数列表不同： 参数列表包括参数的数量、参数的类型、参数的顺序等。至少需要有一个方面在参数列表中不同，否则编译器将无法区分这些函数。
3. 返回类型不同： 重载函数的返回类型可以不同，但它通常不是编译器用于函数重载决策的关键因素。编译器主要关注参数列表。
4. 与函数调用相关： 编译器会根据函数调用时提供的参数来决定调用哪个重载函数。

示例：

```C++
#include <iostream>

void print(int value) {
    std::cout << "Printing integer: " << value << std::endl;
}

void print(double value) {
    std::cout << "Printing double: " << value << std::endl;
}

int main() {
    int intVar = 42;
    double doubleVar = 3.14159;

    print(intVar);     // 调用第一个重载函数
    print(doubleVar);  // 调用第二个重载函数

    return 0;
}
```

### 6、静态变量新特性保证原子性？

1. 初始化保护： C++11规定了静态局部变量的初始化必须是线程安全的。这意味着在多线程环境中，多个线程第一次进入函数并尝试初始化同一个静态局部变量时，只有一个线程会执行初始化，其他线程会等待。这种机制确保了静态局部变量的初始化是线程安全的。

示例：

```C++
#include <iostream>
#include <thread>

void foo() {
    static int x = 0; // C++11以后的标准保证x的初始化是线程安全的
    std::cout << x++ << std::endl;
}

int main() {
    std::thread t1(foo);
    std::thread t2(foo);
    t1.join();
    t2.join();
    return 0;
}
```

2. 内存模型： C++11引入了内存模型，定义了多线程程序中的内存访问行为。这使得对静态局部变量的访问在多线程环境下更加可控。

C++11引入的内存模型定义了一些术语，如"原子操作"、"memory order"等，允许程序员更精确地控制多线程环境中的内存访问。可以使用`std::atomic`类型来声明原子变量，确保它们的操作是线程安全的。

示例：

```C++
#include <iostream>
#include <thread>
#include <atomic>

std::atomic<int> x(0); // 使用std::atomic声明原子变量

void foo() {
    x.fetch_add(1, std::memory_order_relaxed); // 使用原子操作
}

int main() {
    std::thread t1(foo);
    std::thread t2(foo);
    t1.join();
    t2.join();
    std::cout << x.load(std::memory_order_relaxed) << std::endl;
    return 0;
}
```

### 7、静态局部变量的使用？

静态局部变量是在函数内部定义的局部变量，但它与普通局部变量不同，它的生命周期在整个程序运行期间保持不变。这意味着它只会被初始化一次，并且在后续的函数调用中保持其值，直到程序终止。静态局部变量通常用关键字 `static` 来声明。

1. 初始化保护： 静态局部变量的初始化仅在第一次进入包含它的函数时执行，后续的函数调用将跳过初始化过程。这使得静态局部变量适合用于一些需要在多次函数调用中保持状态的情况。

```C++
int myFunction() {
    static int count = 0; // 静态局部变量，只在第一次调用时初始化
    count++;
    return count;
}
```

2. 作用域： 静态局部变量的作用域限于包含它的函数。这意味着它只能在该函数内部访问，不能被其他函数直接访问。

3. 内存分配： 静态局部变量通常存储在程序的全局数据区中，而不是堆栈中。因此，它们在程序启动时分配内存，并在程序终止时释放内存。

4. 线程安全性： 静态局部变量的初始化是线程安全的。C++11以后的标准规定了这一点，确保只有一个线程在初始化时访问该变量。

5. 初始值： 如果不明确初始化静态局部变量，它们将被自动初始化为零值（对于内置类型，如整数）或者空值（对于类类型，如指针或字符串）。

6. 保持状态： 静态局部变量通常用于需要在多次函数调用之间保持状态的情况，如计数器、状态标志或缓存。

示例：

```C++
#include <iostream>

int myFunction() {
    static int count = 0; // 静态局部变量，只在第一次调用时初始化
    count++;
    return count;
}

int main() {
    std::cout << myFunction() << std::endl; // 输出 1
    std::cout << myFunction() << std::endl; // 输出 2
    std::cout << myFunction() << std::endl; // 输出 3
    return 0;
}
```

### 8、四种cast用在什么时候，dynamic什么时候用，不这么用返回值是什么？

1. `static_cast`：

   - 用途：主要用于基本类型之间的转换，如将整数转换为浮点数，或者将指针或引用从一个类型转换为另一个相关类型。

   - 安全性：在编译时执行，较为安全。不进行运行时类型检查。

示例：

```C++
int i = 42;
double d = static_cast<double>(i); // 整数到浮点数的转换
```

2. `dynamic_cast`：

   - 用途：用于在继承关系中进行安全的向下转型（向子类转换）。通常与多态一起使用，可在运行时检查对象的类型。

   - 安全性：在运行时执行，较为安全。如果无法进行安全的向下转型，返回空指针（对于指针）或引发`std::bad_cast`异常（对于引用）。

示例：

```C++
class Base {
    virtual void foo() {}
};
class Derived : public Base {
    void bar() {}
};

Base* basePtr = new Derived;
Derived* derivedPtr = dynamic_cast<Derived*>(basePtr);
if (derivedPtr) {
    // 安全的向下转型
    derivedPtr->bar();
}
```

3. `const_cast`：

   - 用途：用于添加或去除`const`限定符。主要用于让常量变量变为非常量，或非常量变为常量。

   - 安全性：在编译时执行，较为安全。但要注意，滥用`const_cast`可能导致未定义行为。

示例：

```C++
const int i = 42;
int* nonConstPtr = const_cast<int*>(&i); // 去除常量限定符
```

4. `reinterpret_cast`：

   - 用途：进行低级别的类型转换，通常用于指针和整数之间的转换。不执行类型检查，可能导致未定义行为。

   - 安全性：不进行类型检查，非常危险，应慎用。

示例：

```C++
int i = 42;
void* ptr = reinterpret_cast<void*>(&i); // 指针到无关类型的转换
```

注意事项：

- `dynamic_cast`主要用于多态场景，只适用于具有虚函数的类，用于安全的向下转型。
- 滥用强制类型转换可能导致程序错误，建议遵循类型安全和良好的设计原则，尽量减少类型转换的需求。

### 9、Vector数组迭代器失效时机，如何使用迭代器删除vector，erase返回值？

在C++中，`std::vector`的迭代器失效时机主要涉及到插入和删除操作。迭代器失效是指迭代器不再指向有效的元素或容器末尾，因此应该谨慎操作迭代器。

1. 插入元素：如果在`std::vector`中间插入元素，所有在插入点之后的迭代器都会失效，因为插入操作会导致元素的移动，改变容器的内存布局。
2. 删除元素：删除元素时，被删除元素之后的迭代器都会失效。如果使用`erase`函数删除元素，它会返回指向删除元素之后元素的迭代器。

示例：如何使用`erase`来删除`std::vector`中的元素以及`erase`的返回值

```C++
#include <iostream>
#include <vector>

int main() {
    std::vector<int> numbers = {1, 2, 3, 4, 5};

    // 使用迭代器删除元素
    std::vector<int>::iterator it = numbers.begin() + 2; // 指向元素3
    it = numbers.erase(it); // 删除元素3，it指向元素4
    std::cout << "Element at iterator after erasing: " << *it << std::endl;

    // 使用返回值删除元素
    it = numbers.begin() + 2; // 指向元素4
    std::vector<int>::iterator next_it = numbers.erase(it); // 删除元素4，next_it指向元素5
    std::cout << "Element at iterator after erasing: " << *next_it << std::endl;

    for (int num : numbers) {
        std::cout << num << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

### 10、Map数据结构，struct插入map需要注意什么？

在C++中，`std::map`是一个关联容器，用于存储键值对（key-value pairs），并按键的顺序进行排序。

你要往`std::map`中插入`struct`对象时，需要注意以下几点：

1. 定义比较函数或比较运算符：`std::map`根据键来排序，因此需要确保你的`struct`类型有定义适当的比较函数或比较运算符（`<`），以便`std::map`可以根据键的值进行排序。
2. 保证键是唯一的：`std::map`要求键是唯一的，如果你插入的`struct`对象具有相同的键，后一个会覆盖前一个。
3. 插入操作：使用`std::map`的`insert`函数或`[]`运算符来插入`struct`对象。

下面是一个示例，展示如何将自定义`struct`对象插入`std::map`：

```C++
#include <iostream>
#include <map>

// 自定义结构体
struct Person {
    std::string name;
    int age;

    // 自定义比较函数，根据名称比较
    bool operator<(const Person& other) const {
        return name < other.name;
    }
};

int main() {
    std::map<Person, std::string> personMap;

    // 插入数据
    Person person1 = {"Alice", 25};
    Person person2 = {"Bob", 30};

    personMap[person1] = "Entry 1";
    personMap[person2] = "Entry 2";

    // 遍历输出
    for (const auto& entry : personMap) {
        std::cout << "Name: " << entry.first.name << ", Age: " << entry.first.age
                  << ", Data: " << entry.second << std::endl;
    }

    return 0;
}
```

### 11、进程间通信？

1. 管道（Pipes）：

   - 管道是一种单向通信机制，通常用于父子进程之间或者兄弟进程之间。
   - 在Unix/Linux中，可以使用`pipe`系统调用来创建管道。

2. 命名管道（Named Pipes或FIFO）：

   - 命名管道是一种命名的管道，允许不相关的进程进行通信。
   - 在Unix/Linux中，可以使用`mkfifo`函数来创建命名管道。

3. 消息队列（Message Queues）：

   - 消息队列是一种可以在进程之间传递数据的通信方式。
   - 在Unix/Linux中，可以使用`msgget`、`msgsnd`和`msgrcv`函数来操作消息队列。

4. 共享内存（Shared Memory）：

   - 共享内存允许多个进程共享同一块物理内存，以便高效地交换数据。
   - 在Unix/Linux中，可以使用`shmget`、`shmat`和`shmdt`函数来操作共享内存。

5. 信号（Signals）：

   - 信号是一种轻量级的IPC方式，用于通知进程发生了某种事件。
   - 信号通常用于处理异步事件，如进程终止或错误发生。

6. 套接字（Sockets）：

   - 套接字允许进程在不同的主机上通过网络通信。
   - 常见的套接字包括TCP套接字和UDP套接字。

7. 文件锁（File Locking）：

   - 文件锁允许进程通过文件系统进行协同工作，确保数据的一致性。
   - 文件锁通常用于避免多个进程同时写入相同的文件。

8. 信号量（Semaphores）：

   - 信号量是一种用于同步进程之间操作的IPC方式，通常用于解决竞争条件问题。
   - 在Unix/Linux中，可以使用`semget`、`semop`等函数来操作信号量。

9. RPC（远程过程调用）：

   RPC允许进程在不同的机器上调用远程的函数，使得远程调用看起来像本地函数调用。

### 12、如何实现多次运行一个程序只有一个后台进程？

要确保一个程序只有一个后台进程在运行，可以使用锁文件（Lock File）的方式来实现。锁文件是一个特殊的文件，用于表示某个进程是否已经在运行。

给个例子：

1. 创建一个锁文件：在程序启动时，检查是否存在一个特定的锁文件。如果锁文件不存在，程序可以创建一个锁文件，并继续执行。如果锁文件已经存在，说明另一个实例正在运行，程序应该退出。
2. 运行程序：程序在创建锁文件后，继续执行正常的任务。
3. 删除锁文件：当程序完成任务后，应该删除锁文件，以允许将来的实例运行。

```C++
#include <iostream>
#include <fstream>
#include <cstdlib>

bool isAnotherInstanceRunning() {
    // 尝试打开锁文件
    std::ifstream lockFile("myapp.lock");
    
    if (lockFile.is_open()) {
        // 锁文件存在，另一个实例正在运行
        lockFile.close();
        return true;
    }
    
    // 锁文件不存在，当前实例可以运行
    std::ofstream newLockFile("myapp.lock");
    return false;
}

void removeLockFile() {
    std::remove("myapp.lock");
}

int main() {
    if (isAnotherInstanceRunning()) {
        std::cout << "Another instance is already running. Exiting." << std::endl;
        return 1;
    }

    // 正常的应用逻辑
    std::cout << "Running the application..." << std::endl;

    // 模拟应用程序的工作
    std::this_thread::sleep_for(std::chrono::seconds(5));

    // 删除锁文件，允许其他实例运行
    removeLockFile();

    return 0;
}
```

程序首先检查是否存在名为`myapp.lock`的锁文件。如果锁文件存在，程序会发出警告并退出。如果锁文件不存在，程序创建锁文件并执行其正常任务。在任务完成后，程序会删除锁文件，以允许将来的实例运行。

### 13、Tcp三次握手为什么不是2次或者4次？

TCP（传输控制协议）使用三次握手建立连接的原因是为了确保可靠性和防止旧的连接请求被误认为是新的连接请求。

这三次握手的目的是：

1. 同步双方的序列号（Sequence Number）：在TCP连接建立期间，双方需要交换初始的序列号，以确保数据包按正确的顺序传递。三次握手允许双方同步他们的初始序列号。
2. 确保可靠连接：第三次握手是客户端向服务器发送一个确认，表示服务器已经知道客户端的初始序列号，这样确保了双方都知道对方已经准备好建立连接。
3. 防止旧连接的重新连接：假设连接的第三次握手被延迟，客户端可能认为连接失败并尝试重新连接。如果连接只采用两次握手，这个延迟的第三次握手可能被错误地解释为新连接的请求，从而导致连接混乱。

下面是三次握手的详细步骤：

1. 客户端向服务器发送连接请求：客户端发送一个TCP数据包，其中包含SYN标志位，表示客户端希望建立连接。同时，客户端会选择一个随机的初始序列号。
2. 服务器接受连接请求并回应：服务器接受客户端的连接请求，并发送回一个TCP数据包，其中包含SYN和ACK标志位。服务器也会选择一个随机的初始序列号。
3. 客户端确认连接：客户端接受服务器的回应，发送一个带有ACK标志位的数据包，表示连接已建立。此时，客户端和服务器都知道了彼此的初始序列号，连接已经准备好使用。

四次握手不是必需的，因为第四次握手在连接建立后通常没有必要。连接的终止通常需要四次握手（四次挥手），因为双方都需要确认数据传输已经完成，然后才能安全地关闭连接。但在连接建立过程中，三次握手足以确保连接的可靠性和唯一性。

### 14、二叉树最大深度？

给定一个二叉树 `root` ，返回其最大深度。

二叉树的 最大深度 是指从根节点到最远叶子节点的最长路径上的节点数。

思路：

使用两个栈，一个用于存储节点，另一个用于存储节点的深度。我们从根节点开始，逐层遍历树的节点，同时记录每个节点的深度。最终，返回最大深度作为结果。这种方法避免了递归，减小了函数调用栈的开销。

示例代码：

```C++
#include <iostream>
#include <stack>

// 定义二叉树节点
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class Solution {
public:
    int maxDepth(TreeNode* root) {
        if (root == nullptr) {
            return 0;  // 如果根节点为空，返回深度0
        }

        // 使用DFS进行深度搜索
        std::stack<TreeNode*> nodes;
        std::stack<int> depths;
        int max = 0;

        nodes.push(root);
        depths.push(1);  // 根节点的深度为1

        while (!nodes.empty()) {
            TreeNode* node = nodes.top();
            nodes.pop();
            int depth = depths.top();
            depths.pop();
            max = std::max(max, depth);  // 更新最大深度

            // 遍历左子树
            if (node->left) {
                nodes.push(node->left);
                depths.push(depth + 1);  // 左子树深度加1
            }
            // 遍历右子树
            if (node->right) {
                nodes.push(node->right);
                depths.push(depth + 1);  // 右子树深度加1
            }
        }

        return max;  // 返回最大深度
    }
};

int main() {
    // 创建一个二叉树
    TreeNode* root = new TreeNode(1);
    root->left = new TreeNode(2);
    root->right = new TreeNode(3);
    root->left->left = new TreeNode(4);
    root->left->right = new TreeNode(5);

    // 创建Solution对象
    Solution solution;

    // 计算二叉树的最大深度
    int depth = solution.maxDepth(root);

    std::cout << "二叉树的最大深度为: " << depth << std::endl;

    return 0;
}
```

15、反转链表？

直接给上代码吧。思路什么的你们可能

思路：

1. 创建两个指针，`prev`（前一个节点）和`current`（当前节点），并初始化为`nullptr`和链表的头节点，即`head`。
2. 使用一个`while`循环来迭代整个链表，循环条件是`current`不为`nullptr`。在每一次迭代中：
   1. 保存`current`的下一个节点为`next`，以免在反转指针后丢失对后续节点的引用。
   2. 将`current`的`next`指针指向`prev`，从而反转当前节点。
   3. 更新`prev`为`current`，将`current`前进到下一个节点，即`next`。
3. 当循环结束时，`prev`将指向反转后链表的头节点，而`current`将为`nullptr`。
4. 返回`prev`，作为新链表的头节点。

示例代码：

```C++
#include <iostream>

// 定义链表节点结构
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};

class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        ListNode* prev = nullptr; // 前一个节点初始化为nullptr
        ListNode* current = head; // 当前节点从链表头开始

        while (current) {
            ListNode* next = current->next; // 保存下一个节点
            current->next = prev; // 当前节点指向前一个节点完成反转

            prev = current; // 更新前一个节点为当前节点
            current = next; // 当前节点指向下一个节点，继续迭代
        }

        return prev; // prev最终指向新链表的头节点
    }
};

int main() {
    // 创建一个链表：1 -> 2 -> 3 -> 4 -> 5
    ListNode* head = new ListNode(1);
    head->next = new ListNode(2);
    head->next->next = new ListNode(3);
    head->next->next->next = new ListNode(4);
    head->next->next->next->next = new ListNode(5);

    // 创建Solution对象
    Solution solution;

    // 反转链表
    ListNode* reversedHead = solution.reverseList(head);

    // 打印反转后的链表
    ListNode* current = reversedHead;
    while (current) {
        std::cout << current->val << " -> ";
        current = current->next;
    }
    std::cout << "nullptr" << std::endl;

    return 0;
}
```