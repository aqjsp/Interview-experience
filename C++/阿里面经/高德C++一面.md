## 高德C++一面

> 来源：[职言详情页 (maimai.cn)](https://maimai.cn/web/gossip_detail/33193306?egid=3822a98552934b5a90649a92c5fca036&gid=33193306&operation_id=_e7BFY0cVosLusJ3En_kn&share_channel=5&share_euid=oiBH8URbsYQejpaaCiLBNULFLQXjFamXEXtPQLUB-qrFJCzFPRlQHouJgy5MUxGfV4vB4riWyb-fYJ6UGRm9jg)

### 1、结合工作经历讲讲C++

### 2、讲讲多态，多态的原理？

多态的实现依赖于两个：继承和虚函数。

1. 继承：
   - 多态的基础是继承，其中一个类（称为子类或派生类）可以继承另一个类（称为父类或基类）的属性和方法。
   - 子类通过继承从父类获得了一些通用的特性和行为，同时还可以在其自身中添加特定的属性和方法。
2. 虚函数：
   - 多态的核心是虚函数。虚函数是在基类中声明的函数，其行为可以在派生类中重写。
   - C++ 使用关键字 `virtual` 来声明虚函数，而其他面向对象编程语言（如Java和C#）通常默认所有方法都是虚函数，不需要显式声明。
   - 当派生类重写基类中的虚函数时，派生类的实现将被调用而不是基类的实现。
3. 动态绑定：
   - 多态的一个关键特征是动态绑定，也称为运行时绑定。这意味着在运行时，程序会根据对象的实际类型来调用相应的方法，而不是根据变量的声明类型。
   - 这使得程序能够根据具体对象的特性来调用适当的方法，而不需要在编译时知道对象的确切类型。
4. 虚函数表（VTable）：
   - 为了实现动态绑定，大多数编译器在每个类对象的内部维护一个虚函数表（VTable）。这个表是一个指针数组，其中包含了虚函数的地址。
   - 对象的内存布局通常包括一个指向其类的虚函数表的指针。当调用虚函数时，程序会查找虚函数表，找到相应的函数地址，然后调用函数。
   - 派生类可以重写虚函数，并且它们的虚函数表中包含了它们自己的实现。这就实现了多态，因为在运行时，会调用派生类的实际虚函数。

### 3、类的内存排布情况，基类数据成员在哪里？为什么这么放？

在 C++ 中，类的内存排布情况是由编译器决定的，通常遵循一些规则，但并不是所有编译器都完全一样。

1. 类的数据成员按照声明的顺序依次排列，没有特殊指定的话，编译器可能会对数据成员进行内存对齐。
2. 基类的数据成员会位于派生类的数据成员之前，按照声明顺序依次排列。这是因为派生类的对象包含了基类的所有数据成员。
3. 虚函数表指针（vptr）通常会放在对象的起始位置（某些情况下可能会放在末尾），用于实现多态性（动态绑定）。

举个例子，说明基类和派生类的内存排布情况

```
#include <iostream>

class Base {
public:
    int base_data;
};

class Derived : public Base {
public:
    int derived_data;
};

int main() {
    Derived d;
    std::cout << "Size of Derived: " << sizeof(d) << std::endl;
    std::cout << "Offset of base_data in Derived: " << offsetof(Derived, base_data) << std::endl;
    std::cout << "Offset of derived_data in Derived: " << offsetof(Derived, derived_data) << std::endl;

    return 0;
}
```

`Base` 类包含了一个 `int` 类型的数据成员 `base_data`，`Derived` 类继承自 `Base` 类，并包含了一个额外的 `int` 类型的数据成员 `derived_data`。通过 `sizeof` 运算符和 `offsetof` 宏（需要包含 `<cstddef>` 头文件）可以获取类的大小和数据成员的偏移量。

编译器通常会对数据成员进行内存对齐，以提高访问速度和效率。内存对齐是指数据成员在内存中的地址是某个值的倍数，这个值通常是成员类型的大小或者是编译器指定的对齐值。内存对齐可以减少内存访问的次数，提高程序的性能。

### 4、讲讲内存在系统中是如何存储的？

内存在计算机系统中是以二进制形式存储的，它是计算机用来存储程序和数据的重要组成部分。内存通常是由许多存储单元组成，每个存储单元都有一个唯一的地址，可以通过地址访问其中存储的数据。

计算机内存可以分为主存储器（主存）和辅助存储器（辅存）。主存储器通常是指随机访问存储器（RAM），它的特点是可以随机访问任意存储单元，读写速度较快，但是数据不稳定，断电后数据会丢失。辅助存储器通常是指硬盘、固态硬盘（SSD）等，它的特点是数据稳定，断电后数据不会丢失，但读写速度较慢。

内存的存储方式通常包括以下几个方面：

1. 存储单元：内存由许多存储单元组成，每个存储单元可以存储一个字节的数据。每个存储单元都有一个唯一的地址，用于访问该存储单元中的数据。
2. 存储结构：内存通常以字节为单位进行存储，每个存储单元都有一个地址。内存的存储结构可以看作是一个由地址和数据组成的键值对集合。
3. 存储模型：计算机系统中常用的存储模型包括大端序（Big Endian）和小端序（Little Endian）。大端序是指存储数据时高位字节存储在低地址，小端序是指存储数据时低位字节存储在低地址。不同的存储模型会影响数据在内存中的存储方式和访问方式。
4. 内存管理：内存管理是计算机系统中的重要组成部分，包括内存分配、内存释放、内存保护等功能。操作系统负责管理内存的分配和释放，确保程序能够正常运行并且不会相互干扰。

### 5、对哪些数据结构比较熟悉？

这些是我们常见的一些数据结构。

1. 数组（Array）：数组是一种线性数据结构，它由一组连续的存储单元组成，用于存储相同类型的数据。数组的特点是支持随机访问，但插入和删除操作可能比较耗时。
2. 链表（Linked List）：链表是一种线性数据结构，它由一系列节点组成，每个节点包含数据和指向下一个节点的指针。链表的特点是插入和删除操作效率高，但随机访问效率较低。
3. 栈（Stack）：栈是一种后进先出（LIFO）的数据结构，只能在栈顶进行插入和删除操作。栈常用于实现函数调用的追踪、表达式求值等场景。
4. 队列（Queue）：队列是一种先进先出（FIFO）的数据结构，只能在队尾插入元素，在队头删除元素。队列常用于实现任务调度、消息传递等场景。
5. 哈希表（Hash Table）：哈希表是一种根据关键字直接访问数据的数据结构，通过哈希函数将关键字映射到表中的位置。哈希表的特点是查找、插入和删除操作的平均时间复杂度为 O(1)。
6. 二叉树（Binary Tree）：二叉树是一种树形数据结构，每个节点最多有两个子节点。二叉树常用于实现搜索和排序算法，如二叉搜索树、堆等。
7. 图（Graph）：图是一种非线性数据结构，由节点（顶点）和边组成。图常用于表示网络结构、路径搜索等场景，有多种表示方法，如邻接矩阵、邻接表等。
8. 堆（Heap）：堆是一种特殊的树形数据结构，通常用于实现优先队列。堆分为最大堆和最小堆，最大堆的根节点是最大值，最小堆的根节点是最小值。

### 6、vector 使用时需要注意什么？知道它底层实现吗？ reverse 和 resize 的区别？

使用 `std::vector` 时需要注意以下几点：

1. 内存动态扩展：`std::vector` 使用动态数组实现，当元素数量超过当前容量时，会自动进行内存动态扩展。这可能会导致插入操作的时间复杂度为 O(n)，需要注意性能影响。
2. 内存拷贝：`std::vector` 的元素是连续存储的，在进行插入、删除等操作时，可能会涉及到元素的内存拷贝，需要注意对对象的拷贝构造函数和析构函数的影响。
3. 内存分配：`std::vector` 使用动态内存分配，可能会导致内存碎片的问题，特别是在大量插入、删除操作后，需要注意内存的释放和重新分配。
4. 迭代器失效：在对 `std::vector` 进行插入、删除等操作时，可能会导致迭代器失效，需要谨慎使用迭代器。
5. 容量管理：`std::vector` 提供了 `capacity()` 函数用于获取当前容量，可以通过 `reserve()` 函数预留一定的容量，以减少动态扩展的次数。

`std::vector` 的底层实现通常是使用动态数组（dynamic array）实现的，它通过动态分配内存来存储元素，当容量不足时会自动扩展内存。具体实现可能会涉及到内存分配、内存释放、元素的移动和拷贝等操作。

`reserve` 和 `resize` 是 `std::vector` 的区别：

- `reserve(size_type new_cap)`：用于预留容器的存储空间，但不改变容器的大小。如果 `new_cap` 大于当前容器的容量，则分配新的内存空间并将原有元素移动到新空间中，否则不进行任何操作。
- `resize(size_type count, const T& value)`：用于改变容器的大小，如果 `count` 小于当前容器的大小，会删除多余的元素；如果 `count` 大于当前容器的大小，会在末尾添加新元素，并使用 `value` 进行初始化。

### 7、emplace_back 与 push_back 有啥区别？

1. `push_back(const T& value)`：将一个类型为 `T` 的元素 `value` 添加到 `std::vector` 的末尾。如果 `value` 是一个临时对象或者右值引用，会调用移动构造函数将其添加到容器中；如果 `value` 是一个左值引用，会调用拷贝构造函数将其添加到容器中。
2. `emplace_back(Args&&... args)`：直接在 `std::vector` 的末尾构造一个类型为 `T` 的元素，使用 `args` 作为构造函数的参数。与 `push_back` 不同的是，`emplace_back` 不需要创建临时对象或者进行拷贝或移动操作，它直接在容器的末尾构造元素，因此效率更高。

`emplace_back` 相比 `push_back` 更高效，因为它避免了拷贝或移动操作，直接在容器的末尾构造元素。在使用时，可以根据具体情况选择合适的函数来添加新元素，如果需要添加已有对象，可以使用 `push_back`；如果需要直接在容器中构造新对象，可以使用 `emplace_back`。

### 8、sort 的原理知道吗？为什么会使用不同的排序算法？

`std::sort()` 的实现原理通常是采用一种变种的快速排序算法（introsort），它结合了快速排序（Quick Sort）、插入排序（Insertion Sort）和堆排序（Heap Sort）的优点，并在实际应用中根据情况选择合适的排序方法。

1. 快速排序（Quick Sort）：快速排序是一种分治策略的排序算法，它选择一个基准元素（pivot），将序列分为两部分，一部分小于基准，一部分大于基准，然后对两部分分别递归排序。快速排序的关键在于分区（Partition）过程，可以通过双指针法或者三指针法实现。
2. 插入排序（Insertion Sort）：当序列长度较小时，插入排序的性能优于快速排序。因此，当待排序序列长度小于一定阈值时，`std::sort()` 可能会切换到插入排序来提高性能。
3. 堆排序（Heap Sort）：在进行快速排序时，为了避免最坏情况下的性能退化（例如，序列已经有序或者逆序），`std::sort()` 可能会在递归深度达到一定限制时切换到堆排序。

`std::sort()` 的具体实现会根据序列的大小、元素的分布情况和系统的实现选择合适的排序方法，以达到较好的性能。

不同的排序算法适用于不同的数据分布情况和数据规模，具有不同的时间复杂度和空间复杂度。选择合适的排序算法可以提高排序的效率。例如，快速排序对随机分布的数据效果较好，而插入排序对近乎有序的数据效果较好；对于小规模的数据，一些简单的排序算法可能比复杂的排序算法更高效。因此，根据具体的数据特点和排序需求来选择合适的排序算法是很重要的。

### 9、快排的时间复杂度是多少？原理是什么？

快速排序（Quick Sort）的时间复杂度为 O(n log n)，待排数据有序时，时间复杂度为O(n^2)其中 n 为待排序序列的长度。

原理如下：

1. 选择基准元素：从待排序序列中选择一个元素作为基准（pivot）。通常选择第一个元素、最后一个元素或者中间元素作为基准。
2. 分区（Partition）：将序列中的其他元素按照与基准的比较结果分为两部分，一部分小于基准，一部分大于基准。在分区过程中，使用双指针法或者三指针法进行操作，将小于基准的元素放在基准的左边，大于基准的元素放在基准的右边。
3. 递归排序：对基准元素左右两部分分别进行递归排序，直到每个部分只有一个元素或为空。
4. 合并：合并左右两部分已排序的子序列，得到最终的排序结果。

### 10、了解哪些设计模式？使用单例模式时该注意什么？

1. 单例模式（Singleton Pattern）： 用于确保一个类只有一个实例，并提供全局访问点。在项目中，单例模式常用于管理共享资源，如配置信息、日志记录器等。

   例如，一个日志记录器可以使用单例模式来确保在应用程序中只有一个日志记录器实例，以便在多个地方记录日志。

```C
class Logger {
private:
    Logger() {}  // 私有构造函数，防止外部实例化
    static Logger* instance;
public:
    static Logger* getInstance() {
        if (!instance) {
            instance = new Logger();
        }
        return instance;
    }
    void log(const std::string& message) {
        // 实现日志记录逻辑
        std::cout << "Log: " << message << std::endl;
    }
};

// 在应用程序中获取日志记录器实例并使用
int main() {
    Logger* logger = Logger::getInstance();
    logger->log("This is a log message.");
    return 0;
}
```

需要注意的是：

- 线程安全性：如果单例对象在多线程环境下使用，需要考虑其线程安全性。可以通过加锁（悲观锁或乐观锁）、双重检查锁（Double-Checked Locking）等方式来保证线程安全。
- 对象生命周期管理：单例对象的生命周期通常与整个应用程序的生命周期相同，需要注意在合适的时机释放资源，避免内存泄漏。
- 全局状态管理：单例对象是全局唯一的，因此对全局状态的管理需要谨慎，避免出现意外的状态修改导致程序错误。

2. 工厂模式（Factory Pattern）： 用于创建对象的模式，通过将对象的创建逻辑封装在工厂类中，客户端代码可以通过工厂类来创建对象，而不必直接实例化对象。工厂模式在项目中常用于解耦对象的创建和使用。

​	例如，一个图形库可以使用工厂模式来创建不同类型的图形对象。

```C
// 基类
class Shape {
public:
    virtual void draw() = 0;
};

// 具体的图形类
class Circle : public Shape {
public:
    void draw() override {
        std::cout << "Draw a circle." << std::endl;
    }
};

class Rectangle : public Shape {
public:
    void draw() override {
        std::cout << "Draw a rectangle." << std::endl;
    }
};

// 图形工厂
class ShapeFactory {
public:
    static Shape* createShape(const std::string& type) {
        if (type == "Circle") {
            return new Circle();
        }
        else if (type == "Rectangle") {
            return new Rectangle();
        }
        return nullptr;
    }
};

// 在应用程序中使用工厂创建图形对象
int main() {
    Shape* shape1 = ShapeFactory::createShape("Circle");
    shape1->draw();

    Shape* shape2 = ShapeFactory::createShape("Rectangle");
    shape2->draw();

    delete shape1;
    delete shape2;
    return 0;
}
```

3. 观察者模式（Observer Pattern）： 用于实现对象之间的一对多依赖关系，当一个对象的状态发生变化时，所有依赖于它的对象都会得到通知并自动更新。在项目中，观察者模式用于实现事件处理、消息通知等机制。

​	例如，一个新闻发布系统可以使用观察者模式，让多个订阅者（观察者）订阅新闻发布者（主题）的新闻。

```C
#include <iostream>
#include <vector>

// 主题接口
class Subject {
public:
    virtual void addObserver(Observer* observer) = 0;
    virtual void removeObserver(Observer* observer) = 0;
    virtual void notifyObservers() = 0;
};

// 观察者接口
class Observer {
public:
    virtual void update(const std::string& news) = 0;
};

// 具体主题
class NewsPublisher : public Subject {
private:
    std::vector<Observer*> observers;
    std::string latestNews;
public:
    void addObserver(Observer* observer) override {
        observers.push_back(observer);
    }

    void removeObserver(Observer* observer) override {
        // 实现移除观察者的逻辑
    }

    void setNews(const std::string& news) {
        latestNews = news;
        notifyObservers();
    }

    void notifyObservers() override {
        for (Observer* observer : observers) {
            observer->update(latestNews);
        }
    }
};

// 具体观察者
class NewsSubscriber : public Observer {
private:
    std::string name;
public:
    NewsSubscriber(const std::string& n) : name(n) {}

    void update(const std::string& news) override {
        std::cout << name << " received news: " << news << std::endl;
    }
};

int main() {
    NewsPublisher publisher;
    NewsSubscriber subscriber1("Subscriber 1");
    NewsSubscriber subscriber2("Subscriber 2");

    publisher.addObserver(&subscriber1);
    publisher.addObserver(&subscriber2);

    publisher.setNews("Breaking news: Important event!");

    return 0;
}
```

4. 策略模式（Strategy Pattern）： 用于定义一组算法，将它们封装成独立的策略类，并使它们可以互相替换。在项目中，策略模式用于实现动态选择算法或行为的场景，如排序算法、支付方式选择等。

5. 装饰器模式（Decorator Pattern）： 用于动态地给对象添加新的功能，通过创建装饰器类来包装原始对象，从而扩展其功能。在项目中，装饰器模式常用于添加额外的功能或行为，而无需修改现有代码。

6. 适配器模式（Adapter Pattern）： 用于将一个类的接口转换成客户端所期望的接口，以解决接口不兼容的问题。在项目中，适配器模式用于集成第三方库、服务或接口，使其能够与现有系统协作。

7. 命令模式（Command Pattern）： 用于将请求或操作封装成对象，以支持请求的排队、记录请求、撤销请求等功能。在项目中，命令模式用于实现可撤销的操作、日志记录等。

8. 模板方法模式（Template Method Pattern）： 用于定义一个算法的骨架，将一些步骤延迟到子类实现。在项目中，模板方法模式用于确保算法的基本结构不变，但具体实现可以在子类中定制。

9. 状态模式（State Pattern）： 用于根据对象的状态改变其行为，将状态封装成对象，并使对象的行为可以动态地切换。在项目中，状态模式用于管理对象的状态机、工作流程等。

10. 组合模式（Composite Pattern）： 用于将对象组织成树形结构以表示部分-整体的层次结构，使客户端可以一致地处理单个对象和组合对象。在项目中，组合模式用于处理树形数据结构、菜单系统等。

11. 代理模式：代理模式是一种结构型设计模式，它允许一个对象（代理）充当另一个对象的接口，以控制对这个对象的访问。

12. 远程代理（Remote Proxy）： 在分布式系统中，代理对象可以代表远程对象，使客户端能够访问远程对象，就像访问本地对象一样。远程代理用于隐藏网络通信的复杂性。

13. 虚拟代理（Virtual Proxy）： 虚拟代理用于延迟创建昂贵对象的实例，直到需要时才进行实际的创建。例如，在加载大型图片或文档时，可以使用虚拟代理来避免初始加载时间过长。

```C
#include <iostream>
#include <string>

// 图片接口
class Image {
public:
    virtual void display() = 0;
};

// 具体图片类
class RealImage : public Image {
private:
    std::string filename;
public:
    RealImage(const std::string& file) : filename(file) {
        loadFromDisk();
    }

    void display() override {
        std::cout << "Displaying " << filename << std::endl;
    }

    void loadFromDisk() {
        std::cout << "Loading " << filename << " from disk" << std::endl;
    }
};

// 虚拟代理
class ProxyImage : public Image {
private:
    RealImage* realImage;
    std::string filename;
public:
    ProxyImage(const std::string& file) : filename(file), realImage(nullptr) {}

    void display() override {
        if (!realImage) {
            realImage = new RealImage(filename);
        }
        realImage->display();
    }
};

int main() {
    Image* image1 = new ProxyImage("image1.jpg");
    Image* image2 = new ProxyImage("image2.jpg");

    // 图片并未加载，直到调用display方法
    image1->display();
    image2->display();

    // 第二次调用时，图片已经加载，不再从磁盘加载
    image1->display();
    image2->display();

    delete image1;
    delete image2;

    return 0;
}
```

14. 保护代理（Protection Proxy）： 保护代理控制对真实对象的访问权限，允许或拒绝对对象的某些操作。这用于实施访问控制或安全策略。

15. 缓存代理（Cache Proxy）： 缓存代理保存对真实对象的引用，以避免频繁的访问和计算。当下次请求相同数据时，代理可以返回缓存的结果，提高性能。

### 11、排查问题用哪些工具？用过哪些内存分析工具？

在排查问题时，可以使用以下工具：

1. 调试器（Debugger）：如 GDB、LLDB、WinDbg 等，用于在代码中设置断点、查看变量值、跟踪函数调用栈等，帮助定位代码逻辑错误。
2. 性能分析工具：如 Valgrind、Intel VTune、Xperf 等，用于分析程序的性能瓶颈，找出程序的性能优化点。
3. 日志工具：如 Log4j、Logback、Boost.Log 等，用于记录程序运行过程中的日志信息，帮助分析程序的运行状态。
4. 静态代码分析工具：如 Clang Static Analyzer、Coverity、PVS-Studio 等，用于在编译阶段对代码进行静态分析，检测潜在的代码缺陷。
5. 内存分析工具：如 Valgrind、Memcheck、AddressSanitizer、Dr. Memory 等，用于检测内存泄漏、越界访问、野指针等内存相关问题。

对于内存分析工具，我了解的有以下几种：

1. Valgrind：一个强大的开源内存分析工具，包括 Memcheck（检测内存泄漏和越界访问）、Cachegrind（缓存分析）、Massif（堆栈分析）等工具，可以帮助发现内存相关的问题。
2. AddressSanitizer（ASan）：Clang 和 GCC 编译器提供的一种内存错误检测工具，可以在运行时检测内存访问错误，如内存泄漏、越界访问等。
3. Dr. Memory：一个基于 Valgrind 的内存错误检测工具，可以检测内存泄漏、越界访问、不正确的内存使用等问题。
4. Electric Fence：一个简单的内存错误检测工具，通过修改内存分配器的行为，在内存的每一端都加上一个保护页面，当程序访问这些保护页面时，会触发异常。

### 12、编译的过程，动态库与静态库的区别？

编译的过程：

1. 预处理：预处理器根据源代码中的预处理指令（如 `#include`、`#define`、`#ifdef` 等）对源代码进行处理，生成经过预处理的源文件。
2. 编译：编译器将预处理后的源文件翻译成汇编代码，包括词法分析、语法分析、语义分析和中间代码生成等过程。
3. 汇编：汇编器将汇编代码翻译成机器码（或者称为目标代码），生成目标文件（Object File）。
4. 链接：链接器将多个目标文件和库文件链接在一起，生成可执行文件。在链接过程中，会解析符号引用（如函数调用、全局变量），将其关联到实际的地址，同时处理重定位、符号表等工作。

动态库与静态库的区别：

- 静态库：静态库在链接阶段会被完整地复制到最终的可执行文件中，因此可执行文件的体积较大。静态库的优点是使用简单，不依赖于外部环境，但缺点是每个使用了该库的程序都会包含一份完整的库代码，造成资源浪费。
- 动态库：动态库在链接阶段并不会被完整地复制到最终的可执行文件中，而是在程序运行时由操作系统动态加载到内存中。因此，可执行文件的体积较小，且多个程序可以共享同一份动态库代码，节省了资源。但动态库需要在运行时进行加载和链接，可能会引入一些性能损失。

### 13、讲一讲锁，你知道 mutex 的底层实现是什么吗？

1. 自旋锁（Spin Lock）：

   - 自旋锁是一种忙等待的同步机制。当线程尝试获取锁但锁已经被其他线程占用时，它会在一个循环中不断尝试获取锁，而不会被挂起。
     - 自旋锁通常在以下情况下使用：
       - 临界区的锁被占用的时间非常短暂，而线程被挂起和唤醒的开销较大。
       - 自旋锁适用于多核CPU，因为在线程自旋等待时，其他线程可以在不同的核上执行。

   - 自旋锁的实现通常使用原子操作，如Compare-And-Swap（CAS）指令。

2. 互斥锁（Mutex）：
   - 互斥锁是一种阻塞式的同步机制。当线程尝试获取锁但锁已经被其他线程占用时，它会被挂起，等待锁的释放。
   - 互斥锁通常在以下情况下使用：
     - 临界区的锁被占用的时间较长，而线程被挂起和唤醒的开销相对较小。
     - 互斥锁适用于单核或多核CPU。
   - 互斥锁的实现依赖于操作系统提供的原语，通常使用系统调用来挂起和唤醒线程。

互斥锁的底层实现通常依赖于操作系统提供的原子操作和同步原语，具体实现方式可能会有所不同。在大多数操作系统中，互斥锁的实现会使用原子操作（如 CAS 指令）来保证多线程环境下的线程安全性。当一个线程尝试获取互斥锁时，如果锁已经被其他线程获取，则该线程会进入等待状态，直到锁被释放为止。

3. 悲观锁

   - 基本思想： 悲观锁认为在并发环境中，总是假设最坏的情况，即认为会发生冲突，因此在访问共享资源之前先加锁，确保同一时刻只有一个线程或进程可以访问。

   - 实现方式： 典型的实现方式是使用传统的互斥锁，如Mutex或Semaphore。当一个线程获得锁之后，其他线程就必须等待，直到获得锁的线程释放锁。

4. 乐观锁

   - 基本思想： 乐观锁认为在并发环境中，冲突是比较少见的，因此在访问共享资源之前不加锁，而是在更新时检查是否有其他线程进行了修改。如果没有发现冲突，则提交更新；如果发现冲突，则进行回滚或执行一些冲突处理策略。

   - 实现方式： 典型的实现方式是使用版本号（Versioning）或时间戳（Timestamping）。每个数据项都有一个版本号或时间戳，当要更新数据时，先检查版本号或时间戳，如果匹配则进行更新，否则认为发生了冲突。

5. 读写锁

它允许多个线程同时对共享资源进行读操作，但在进行写操作时需要独占锁。

### 14、有 100 万个经纬度数据点，再给你一个点，让你找出离该点最近的5个点

给个参考

要找出离给定点最近的5个点，可以使用空间索引结构来加速搜索，最常用的是 KD 树（K-dimensional Tree）。KD 树是一种二叉树，每个节点代表一个 k 维空间中的超矩形区域，每个节点的左子树和右子树分别代表该节点区域中点的左半部分和右半部分。KD 树的构建过程如下：

1. 选择切分维度：选择一个维度作为切分维度，通常是根据当前节点的深度来循环选择。
2. 选择切分值：选择一个切分值，将当前节点的点集分成两部分，左子节点包含小于等于切分值的点，右子节点包含大于切分值的点。
3. 递归构建子树：对左右子节点分别递归构建 KD 树，直到所有点都被处理。

构建好 KD 树后，可以使用以下方法找出离给定点最近的5个点：

1. 最近邻搜索：从根节点开始，递归地向下搜索 KD 树，根据当前节点的切分维度和切分值，决定向左子树或右子树搜索。在搜索过程中，维护一个优先队列（Priority Queue）来存储距离当前点最近的5个点，每次找到更近的点时更新队列。
2. 剪枝：在搜索过程中，可以通过计算当前点到切分超平面的距离，以及当前最近点集中最远点到切分超平面的距离，来确定是否需要搜索对应的子树。如果当前最近点集中最远点到切分超平面的距离小于当前点到切分超平面的距离，可以剪枝，不需要搜索对应的子树。
3. 遍历完整树：当搜索完整个 KD 树后，优先队列中存储的就是离给定点最近的5个点。

### 15、手写一个快排 ，不用AC

基于分治的思想，通过选取一个基准元素将数组划分为两个子数组，然后对子数组进行递归排序。

参考代码：

递归：

```C++
#include <iostream>
#include <vector>

void quickSort(std::vector<int>& arr, int low, int high) {
    if (low < high) {
        // 划分操作，返回基准元素的位置
        int pivotIndex = partition(arr, low, high);

        // 对基准元素左右两侧进行递归排序
        quickSort(arr, low, pivotIndex - 1);
        quickSort(arr, pivotIndex + 1, high);
    }
}

int partition(std::vector<int>& arr, int low, int high) {
    // 选择最右边的元素作为基准
    int pivot = arr[high];
    // i 指向小于基准的区域的最后一个元素
    int i = low - 1;

    // 遍历数组，将小于基准的元素放到左侧，大于基准的元素放到右侧
    for (int j = low; j < high; ++j) {
        if (arr[j] <= pivot) {
            // 将小于基准的元素交换到左侧区域
            i++;
            std::swap(arr[i], arr[j]);
        }
    }

    // 将基准元素交换到正确的位置
    std::swap(arr[i + 1], arr[high]);

    // 返回基准元素的位置
    return i + 1;
}

int main() {
    std::vector<int> arr = {12, 4, 5, 6, 7, 3, 1, 15};
    int n = arr.size();

    std::cout << "Original array: ";
    for (int num : arr) {
        std::cout << num << " ";
    }
    std::cout << std::endl;

    quickSort(arr, 0, n - 1);

    std::cout << "Sorted array: ";
    for (int num : arr) {
        std::cout << num << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

`quickSort` 函数进行递归排序，而 `partition` 函数负责对数组进行划分。选择最右边的元素作为基准，遍历数组，将小于基准的元素交换到左侧，大于基准的元素交换到右侧。最后，将基准元素放到正确的位置。通过递归调用 `quickSort` 函数，可以完成整个数组的排序。

非递归：

```C++
#include <iostream>
#include <vector>
#include <stack>

void quickSort(std::vector<int>& arr, int low, int high) {
    std::stack<std::pair<int, int>> stack; // 用于模拟递归调用的栈
    stack.push(std::make_pair(low, high));

    while (!stack.empty()) {
        std::pair<int, int> range = stack.top();
        stack.pop();
        int pivotIndex = partition(arr, range.first, range.second);

        // 对基准元素左右两侧进行非递归排序
        if (pivotIndex - 1 > range.first) {
            stack.push(std::make_pair(range.first, pivotIndex - 1));
        }
        if (pivotIndex + 1 < range.second) {
            stack.push(std::make_pair(pivotIndex + 1, range.second));
        }
    }
}

int partition(std::vector<int>& arr, int low, int high) {
    int pivot = arr[high];
    int i = low - 1;

    for (int j = low; j < high; ++j) {
        if (arr[j] <= pivot) {
            i++;
            std::swap(arr[i], arr[j]);
        }
    }

    std::swap(arr[i + 1], arr[high]);
    return i + 1;
}

int main() {
    std::vector<int> arr = {12, 4, 5, 6, 7, 3, 1, 15};
    int n = arr.size();

    std::cout << "Original array: ";
    for (int num : arr) {
        std::cout << num << " ";
    }
    std::cout << std::endl;

    quickSort(arr, 0, n - 1);

    std::cout << "Sorted array: ";
    for (int num : arr) {
        std::cout << num << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

使用一个栈 `std::stack<std::pair<int, int>> stack` 来存储待处理的数组范围。初始时将整个数组的范围入栈，然后通过迭代模拟递归的过程。在每一次迭代中，取出栈顶的数组范围，进行划分操作，并将划分后的两个子数组的范围入栈。这样，通过栈的操作，完成了非递归的快速排序。