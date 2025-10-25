# 百度搜索内容技术部c++一面面经

> 来源：https://www.nowcoder.com/discuss/624638390083870720

### 1、cpp编译链接过程？

#### 1. 预处理

预处理器负责处理以 `#` 开头的指令，如宏定义、文件包含等。预处理的结果是一个纯文本文件，其中包含了所有的头文件和替换了所有的宏。

- **宏替换**：所有的宏定义（用 `#define` 定义的）会被替换成对应的值。
- **文件包含**：所有的 `#include` 语句会被替换成包含文件的内容。
- **条件编译**：处理所有的条件编译指令（如 `#ifdef`、`#ifndef`、`#if`、`#else` 和 `#endif`）。
- **注释移除**：移除所有的注释。

预处理器生成的文件通常以 `.i` 为扩展名。

#### 2. 编译

编译器将预处理后的源代码转换为汇编代码。这个阶段，编译器会进行语法和语义检查，并将高层次的 C++ 代码转换为对应的低层次汇编代码。

- **语法检查**：检查代码的语法是否正确。
- **语义分析**：检查代码的逻辑是否正确，例如类型检查、变量定义和作用域检查等。
- **优化**：编译器可能会对代码进行优化，以提高生成代码的性能。
- **生成汇编代码**：将 C++ 代码转换为汇编代码。

编译器生成的汇编文件通常以 `.s` 为扩展名。

#### 3. 汇编

汇编器将汇编代码转换为机器代码（目标代码）。目标代码是特定于目标处理器的二进制代码，但还不是可执行文件。

- **生成机器代码**：将汇编指令转换为机器指令。
- **生成目标文件**：目标文件包含了机器代码和一些元数据（如符号表和重定位信息）。

汇编器生成的目标文件通常以 `.o`（Unix 系统）或 `.obj`（Windows 系统）为扩展名。

#### 4. 链接

链接器负责将一个或多个目标文件和库文件链接在一起，生成最终的可执行文件。

- **符号解析**：解析所有的符号，确保所有的函数和变量都能找到定义。
- **地址分配**：为所有的符号分配内存地址。
- **库链接**：将所需的库文件（静态库或动态库）链接到目标文件中。
- **生成可执行文件**：将所有的目标代码合并，生成最终的可执行文件。

链接器生成的可执行文件通常在 Unix 系统中没有扩展名，在 Windows 系统中以 `.exe` 为扩展名。

### 2、cpp11新特性了解多少？

#### 1. 自动类型推导 (auto)

`auto` 关键字允许编译器根据初始化表达式推导变量的类型，从而简化代码书写，特别是在需要声明复杂类型时。

```c++
auto x = 42;     // int
auto y = 3.14;   // double
auto z = "Hello"; // const char*
```

#### 2. 统一初始化

C++11 引入了一种新的初始化语法，可以统一地对数组、结构体、类等进行初始化。

```c++
int arr[] = {1, 2, 3, 4};
std::vector<int> vec = {1, 2, 3, 4};
struct Point { int x, y; };
Point p = {1, 2};
```

#### 3. Lambda 表达式

Lambda 表达式提供了一种简洁的方式来定义匿名函数，可以在需要函数对象的地方直接使用。

```c++
auto add = [](int a, int b) { return a + b; };
int result = add(2, 3); // result = 5
```

#### 4. 右值引用和移动语义

右值引用 (`&&`) 和移动语义允许更高效地管理资源，减少不必要的拷贝操作。

```c++
class MyClass {
public:
    MyClass(const MyClass& other); // 拷贝构造函数
    MyClass(MyClass&& other);      // 移动构造函数
};

std::vector<int> v1 = {1, 2, 3, 4};
std::vector<int> v2 = std::move(v1); // v1 的资源被移动到 v2
```

#### 5. 智能指针

C++11 引入了智能指针 (`std::unique_ptr` 和 `std::shared_ptr`)，用于自动管理动态内存，避免内存泄漏。

```c++
std::unique_ptr<int> p1(new int(10));
std::shared_ptr<int> p2 = std::make_shared<int>(20);
```

#### 6. `nullptr` 关键字

`nullptr` 引入了一个明确表示空指针的值，代替传统的 `NULL`。

```c++
int* p = nullptr;
```

#### 7. 常量表达式 (constexpr)

`constexpr` 关键字用于声明在编译时求值的常量表达式，可以提高程序的运行效率。

```c++
constexpr int square(int x) {
    return x * x;
}
constexpr int result = square(5); // result 在编译时计算
```

#### 8. 范围循环 

范围循环提供了一种遍历容器元素的简洁语法。

```c++
std::vector<int> vec = {1, 2, 3, 4};
for (int x : vec) {
    std::cout << x << " ";
}
```

#### 9. 类成员初始化

可以直接在类定义中对成员变量进行初始化。

```c++
class MyClass {
    int x = 10;
    int y = 20;
};
```

#### 10. 标准库增强

C++11 对标准库进行了大量增强，包括新容器（如 `std::array`）、多线程支持（`<thread>`、`<mutex>`）、正则表达式（`<regex>`）、随机数库（`<random>`）等。

```c++
std::array<int, 4> arr = {1, 2, 3, 4};
std::thread t([]{ std::cout << "Hello from thread!"; });
t.join();
```

#### 11. 新的函数声明

包括 `noexcept` 指定函数不会抛出异常，以及 `override` 和 `final` 用于更好地控制虚函数的行为。

```c++
void func() noexcept {
    // 不会抛出异常的函数
}

class Base {
    virtual void foo() const;
};

class Derived : public Base {
    void foo() const override; // 确保是重写基类的虚函数
};
```

#### 12. 模板增强

包括变长模板参数（Variadic Templates）和别名模板（Alias Templates）。

```c++
template<typename... Args>
void print(Args... args) {
    (std::cout << ... << args) << std::endl; // C++17 折叠表达式
}

template<typename T>
using Vec = std::vector<T>;
```

#### 13. 强类型枚举 

`enum class` 提供了强类型的枚举，避免与其他枚举类型或整数类型混淆。

```c++
enum class Color { Red, Green, Blue };
Color c = Color::Red;
```

#### 14. `decltype` 关键字

`decltype` 关键字用于查询表达式的类型，可以用于声明与表达式类型一致的变量。

```c++
int x = 10;
decltype(x) y = 20; // y 的类型是 int
```

#### 15. 静态断言

`static_assert` 在编译时进行条件检查，提供更好的调试和错误提示。

```c++
static_assert(sizeof(int) == 4, "Unexpected int size");
```

这些特性使 C++11 成为一个更加现代化、高效和易用的编程语言，同时保持了 C++ 的强大和灵活性。

### 3、介绍右值引用，智能指针？

#### 右值引用

右值引用是 C++11 引入的一种引用类型，用于绑定到右值（临时对象），从而支持移动语义和完美转发。右值引用的语法是使用两个 `&` 符号（`&&`）。

##### 1. 什么是右值和左值？

- **左值（Lvalue）**：具有持久存储的对象，可以取地址。例如，变量、数组元素、类对象的成员等。
- **右值（Rvalue）**：临时对象，不具有持久存储。例如，常量、临时对象、函数返回的非引用值等。

##### 2. 右值引用的使用

右值引用主要用于实现移动语义（Move Semantics），可以避免不必要的拷贝，提高程序性能。

###### 移动构造函数和移动赋值运算符

通过移动构造函数和移动赋值运算符，可以将资源从一个对象转移到另一个对象，而不需要进行昂贵的深拷贝操作。

```c++
#include <iostream>
#include <vector>

class MyClass {
public:
    std::vector<int> data;

    // 普通构造函数
    MyClass(size_t size) : data(size) {
        std::cout << "Constructed\n";
    }

    // 移动构造函数
    MyClass(MyClass&& other) noexcept : data(std::move(other.data)) {
        std::cout << "Move Constructed\n";
    }

    // 移动赋值运算符
    MyClass& operator=(MyClass&& other) noexcept {
        if (this != &other) {
            data = std::move(other.data);
        }
        std::cout << "Move Assigned\n";
        return *this;
    }
};

int main() {
    MyClass a(100);
    MyClass b(std::move(a));  // 调用移动构造函数
    MyClass c(200);
    c = std::move(b);         // 调用移动赋值运算符
    return 0;
}
```

##### 3. 完美转发

完美转发允许函数模板将其参数完全地传递给另一个函数，无论是左值还是右值。

```c++
#include <utility>
#include <iostream>

void process(int& x) {
    std::cout << "Lvalue\n";
}

void process(int&& x) {
    std::cout << "Rvalue\n";
}

template <typename T>
void forwarder(T&& arg) {
    process(std::forward<T>(arg));
}

int main() {
    int x = 10;
    forwarder(x);         // 调用 process(int&)
    forwarder(20);        // 调用 process(int&&)
    return 0;
}
```

#### 智能指针（Smart Pointers）

智能指针是 C++11 引入的，用于自动管理动态内存，避免内存泄漏和悬空指针问题。C++ 标准库中提供了几种智能指针：`std::unique_ptr`、`std::shared_ptr` 和 `std::weak_ptr`。

##### 1. `std::unique_ptr`

`std::unique_ptr` 是独占所有权的智能指针，一个 `unique_ptr` 指针不能被拷贝，只能移动。这保证了同一时间只有一个指针拥有该对象的所有权。

```c++
#include <iostream>
#include <memory>

class MyClass {
public:
    MyClass() { std::cout << "MyClass Constructed\n"; }
    ~MyClass() { std::cout << "MyClass Destroyed\n"; }
};

int main() {
    std::unique_ptr<MyClass> ptr1 = std::make_unique<MyClass>();
    // std::unique_ptr<MyClass> ptr2 = ptr1; // 错误：不能拷贝
    std::unique_ptr<MyClass> ptr2 = std::move(ptr1); // 可以移动
    return 0;
}
```

##### 2. `std::shared_ptr`

`std::shared_ptr` 是共享所有权的智能指针，多个 `shared_ptr` 可以指向同一个对象，通过引用计数来管理对象的生命周期。当最后一个 `shared_ptr` 被销毁时，对象才会被释放。

```c++
#include <iostream>
#include <memory>

class MyClass {
public:
    MyClass() { std::cout << "MyClass Constructed\n"; }
    ~MyClass() { std::cout << "MyClass Destroyed\n"; }
};

int main() {
    std::shared_ptr<MyClass> ptr1 = std::make_shared<MyClass>();
    std::shared_ptr<MyClass> ptr2 = ptr1; // 共享所有权
    std::cout << "Use count: " << ptr1.use_count() << "\n"; // 输出引用计数
    return 0;
}
```

##### 3. `std::weak_ptr`

`std::weak_ptr` 是一种不拥有对象所有权的智能指针，它与 `shared_ptr` 搭配使用，避免循环引用问题。`weak_ptr` 可以观察对象，但不会影响其生命周期。

```c++
#include <iostream>
#include <memory>

class MyClass {
public:
    std::shared_ptr<MyClass> other;

    MyClass() { std::cout << "MyClass Constructed\n"; }
    ~MyClass() { std::cout << "MyClass Destroyed\n"; }
};

int main() {
    std::shared_ptr<MyClass> ptr1 = std::make_shared<MyClass>();
    std::weak_ptr<MyClass> weakPtr = ptr1;

    if (auto sharedPtr = weakPtr.lock()) {
        std::cout << "Object is still alive\n";
    } else {
        std::cout << "Object has been destroyed\n";
    }

    return 0;
}
```

### 4、介绍常见设计模式？

#### 创建型模式（Creational Patterns）

创建型模式关注于对象的创建过程，确保对象的创建和使用分离。

##### 1. 单例模式（Singleton Pattern）

确保一个类只有一个实例，并提供一个全局访问点。

```c++
class Singleton {
private:
    static Singleton* instance;
    Singleton() {} // 私有构造函数

public:
    static Singleton* getInstance() {
        if (instance == nullptr) {
            instance = new Singleton();
        }
        return instance;
    }
};

// 定义静态成员变量
Singleton* Singleton::instance = nullptr;
```

##### 2. 工厂方法模式（Factory Method Pattern）

定义一个创建对象的接口，但由子类决定实例化哪个类。工厂方法使一个类的实例化延迟到其子类。

```c++
class Product {
public:
    virtual void use() = 0;
};

class ConcreteProductA : public Product {
public:
    void use() override {
        std::cout << "Using Product A" << std::endl;
    }
};

class ConcreteProductB : public Product {
public:
    void use() override {
        std::cout << "Using Product B" << std::endl;
    }
};

class Creator {
public:
    virtual Product* factoryMethod() = 0;
};

class ConcreteCreatorA : public Creator {
public:
    Product* factoryMethod() override {
        return new ConcreteProductA();
    }
};

class ConcreteCreatorB : public Creator {
public:
    Product* factoryMethod() override {
        return new ConcreteProductB();
    }
};
```

##### 3. 抽象工厂模式（Abstract Factory Pattern）

提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。

```c++
class AbstractProductA {
public:
    virtual void use() = 0;
};

class AbstractProductB {
public:
    virtual void eat() = 0;
};

class ConcreteProductA1 : public AbstractProductA {
public:
    void use() override {
        std::cout << "Using Product A1" << std::endl;
    }
};

class ConcreteProductB1 : public AbstractProductB {
public:
    void eat() override {
        std::cout << "Eating Product B1" << std::endl;
    }
};

class AbstractFactory {
public:
    virtual AbstractProductA* createProductA() = 0;
    virtual AbstractProductB* createProductB() = 0;
};

class ConcreteFactory1 : public AbstractFactory {
public:
    AbstractProductA* createProductA() override {
        return new ConcreteProductA1();
    }
    AbstractProductB* createProductB() override {
        return new ConcreteProductB1();
    }
};
```

#### 结构型模式（Structural Patterns）

结构型模式关注类和对象的组合，确保一个类或对象的复杂结构更易于管理。

##### 1. 适配器模式（Adapter Pattern）

将一个类的接口转换成客户希望的另一个接口，使原本由于接口不兼容而不能一起工作的那些类可以一起工作。

```c++
class Target {
public:
    virtual void request() {
        std::cout << "Target request" << std::endl;
    }
};

class Adaptee {
public:
    void specificRequest() {
        std::cout << "Adaptee specific request" << std::endl;
    }
};

class Adapter : public Target {
private:
    Adaptee* adaptee;

public:
    Adapter(Adaptee* adaptee) : adaptee(adaptee) {}

    void request() override {
        adaptee->specificRequest();
    }
};
```

##### 2. 装饰者模式（Decorator Pattern）

动态地给对象添加职责，而不改变其接口。装饰者模式提供了比继承更灵活的扩展功能的方式。

```c++
class Component {
public:
    virtual void operation() = 0;
};

class ConcreteComponent : public Component {
public:
    void operation() override {
        std::cout << "ConcreteComponent operation" << std::endl;
    }
};

class Decorator : public Component {
protected:
    Component* component;

public:
    Decorator(Component* component) : component(component) {}

    void operation() override {
        component->operation();
    }
};

class ConcreteDecoratorA : public Decorator {
public:
    ConcreteDecoratorA(Component* component) : Decorator(component) {}

    void operation() override {
        Decorator::operation();
        addedBehavior();
    }

    void addedBehavior() {
        std::cout << "ConcreteDecoratorA added behavior" << std::endl;
    }
};
```

##### 3. 外观模式（Facade Pattern）

为子系统中的一组接口提供一个一致的界面，外观模式定义了一个高层接口，这个接口使得这一子系统更加容易使用。

```c++
class Subsystem1 {
public:
    void operation1() {
        std::cout << "Subsystem1 operation1" << std::endl;
    }
};

class Subsystem2 {
public:
    void operation2() {
        std::cout << "Subsystem2 operation2" << std::endl;
    }
};

class Facade {
private:
    Subsystem1* subsystem1;
    Subsystem2* subsystem2;

public:
    Facade() {
        subsystem1 = new Subsystem1();
        subsystem2 = new Subsystem2();
    }

    void operation() {
        subsystem1->operation1();
        subsystem2->operation2();
    }
};
```

#### 行为型模式（Behavioral Patterns）

行为型模式关注对象之间的责任分配和交互，描述对象怎样协作以及职责如何分配。

##### 1. 观察者模式（Observer Pattern）

定义对象间的一种一对多的依赖关系，使得每当一个对象改变状态，则所有依赖于它的对象都会得到通知并自动更新。

```c++
#include <vector>
#include <algorithm>

class Observer {
public:
    virtual void update() = 0;
};

class Subject {
private:
    std::vector<Observer*> observers;

public:
    void attach(Observer* observer) {
        observers.push_back(observer);
    }

    void detach(Observer* observer) {
        observers.erase(std::remove(observers.begin(), observers.end(), observer), observers.end());
    }

    void notify() {
        for (Observer* observer : observers) {
            observer->update();
        }
    }
};

class ConcreteObserver : public Observer {
private:
    Subject* subject;

public:
    ConcreteObserver(Subject* subject) : subject(subject) {
        subject->attach(this);
    }

    void update() override {
        std::cout << "Observer updated" << std::endl;
    }
};
```

##### 2. 策略模式（Strategy Pattern）

定义一系列算法，把它们一个个封装起来，并且使它们可互相替换。本模式使得算法可以独立于使用它的客户而变化。

```c++
class Strategy {
public:
    virtual void execute() = 0;
};

class ConcreteStrategyA : public Strategy {
public:
    void execute() override {
        std::cout << "ConcreteStrategyA execution" << std::endl;
    }
};

class ConcreteStrategyB : public Strategy {
public:
    void execute() override {
        std::cout << "ConcreteStrategyB execution" << std::endl;
    }
};

class Context {
private:
    Strategy* strategy;

public:
    void setStrategy(Strategy* strategy) {
        this->strategy = strategy;
    }

    void executeStrategy() {
        strategy->execute();
    }
};
```

##### 3. 命令模式（Command Pattern）

将一个请求封装为一个对象，从而使你可以用不同的请求对客户进行参数化；对请求排队或记录请求日志，以及支持可撤销的操作。

```c++
class Command {
public:
    virtual void execute() = 0;
};

class Receiver {
public:
    void action() {
        std::cout << "Receiver action" << std::endl;
    }
};

class ConcreteCommand : public Command {
private:
    Receiver* receiver;

public:
    ConcreteCommand(Receiver* receiver) : receiver(receiver) {}

    void execute() override {
        receiver->action();
    }
};

class Invoker {
private:
    Command* command;

public:
    void setCommand(Command* command) {
        this->command = command;
    }

    void executeCommand() {
        command->execute();
    }
};
```

### 5、单例模式具体实现？

单例模式确保一个类只有一个实例，并提供一个全局访问点。

#### 饿汉式单例（Eager Initialization）

饿汉式单例在类加载时就创建实例，线程安全，但如果实例初始化很重或者程序未使用到该实例，会造成资源浪费。

```c++
class Singleton {
private:
    static Singleton instance; // 静态实例

    // 私有构造函数，防止外部实例化
    Singleton() {}

public:
    // 禁用拷贝构造函数和赋值运算符
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;

    // 提供一个全局访问点
    static Singleton& getInstance() {
        return instance;
    }

    void doSomething() {
        // 业务方法
    }
};

// 初始化静态成员变量
Singleton Singleton::instance;
```

#### 懒汉式单例（Lazy Initialization）

懒汉式单例在第一次调用 `getInstance` 时创建实例，节省资源，但需要注意线程安全问题。

##### 非线程安全实现

```c++
class Singleton {
private:
    static Singleton* instance;

    Singleton() {}

public:
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;

    static Singleton* getInstance() {
        if (instance == nullptr) {
            instance = new Singleton();
        }
        return instance;
    }

    void doSomething() {
        // 业务方法
    }
};

// 初始化静态成员变量
Singleton* Singleton::instance = nullptr;
```

##### 线程安全实现（使用互斥锁）

```c++
#include <mutex>

class Singleton {
private:
    static Singleton* instance;
    static std::mutex mtx;

    Singleton() {}

public:
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;

    static Singleton* getInstance() {
        std::lock_guard<std::mutex> lock(mtx);
        if (instance == nullptr) {
            instance = new Singleton();
        }
        return instance;
    }

    void doSomething() {
        // 业务方法
    }
};

// 初始化静态成员变量
Singleton* Singleton::instance = nullptr;
std::mutex Singleton::mtx;
```

#### 双重检查锁定（Double-Checked Locking）

双重检查锁定是在懒汉式单例的基础上进一步优化，减少加锁的开销，但需要使用 `volatile` 关键字确保编译器不会对代码进行重排序。

```c++
#include <mutex>

class Singleton {
private:
    static Singleton* instance;
    static std::mutex mtx;

    Singleton() {}

public:
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;

    static Singleton* getInstance() {
        if (instance == nullptr) {
            std::lock_guard<std::mutex> lock(mtx);
            if (instance == nullptr) {
                instance = new Singleton();
            }
        }
        return instance;
    }

    void doSomething() {
        // 业务方法
    }
};

// 初始化静态成员变量
Singleton* Singleton::instance = nullptr;
std::mutex Singleton::mtx;
```

#### 使用 `std::call_once` 实现

C++11 引入的 `std::call_once` 可以保证只调用一次初始化代码，简化线程安全的单例实现。

```c++
#include <mutex>

class Singleton {
private:
    static Singleton* instance;
    static std::once_flag initFlag;

    Singleton() {}

public:
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;

    static Singleton* getInstance() {
        std::call_once(initFlag, []() {
            instance = new Singleton();
        });
        return instance;
    }

    void doSomething() {
        // 业务方法
    }
};

// 初始化静态成员变量
Singleton* Singleton::instance = nullptr;
std::once_flag Singleton::initFlag;
```

#### 使用局部静态变量

C++11 之后，局部静态变量的初始化是线程安全的，可以利用这一特性简化单例实现。

```c++
class Singleton {
private:
    Singleton() {}

public:
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;

    static Singleton& getInstance() {
        static Singleton instance;
        return instance;
    }

    void doSomething() {
        // 业务方法
    }
};
```

### 6、死锁发生原因，怎么防止？

死锁是指两个或多个进程或线程在争夺系统资源时互相等待，导致它们都无法继续执行的情况。死锁的发生需要满足以下四个条件，称为“必要条件”：

1. **互斥条件（Mutual Exclusion）**：

   资源不能被共享，每次只能一个进程使用。

2. **占有并等待条件（Hold and Wait）**：

   进程已经获得了一些资源，但在等待其他资源的同时，不释放已获得的资源。

3. **不可剥夺条件（No Preemption）**：

   已获得的资源不能被强制剥夺，只能由持有该资源的进程自愿释放。

4. **循环等待条件（Circular Wait）**：

   存在一个进程集合 {P1, P2, ..., Pn}，其中 P1 等待 P2 占有的资源，P2 等待 P3 占有的资源，直到 Pn 等待 P1 占有的资源，形成一个环形等待链。

#### 防止死锁的方法

防止死锁的方法主要有四种：

1. **破坏互斥条件**：

   使资源可以共享。例如，将一些资源变为可重入的资源。

2. **破坏占有并等待条件**：

   进程请求资源时不占有其他资源，可以采用以下方法：

   - 进程在开始执行时一次性申请所有需要的资源。
   - 进程在申请资源时，如果无法获得所有资源，则释放已占有的资源，稍后重新尝试。

3. **破坏不可剥夺条件**：

   允许资源被强制剥夺。例如：

   - 进程可以主动释放资源，稍后重新申请。
   - 如果某进程占有的资源被其他进程请求，可以强制该进程释放资源。

4. **破坏循环等待条件**：

   破坏循环等待条件可以通过以下方法：

   - **资源有序分配法**：为系统中的每类资源分配一个全局唯一的编号。进程必须按编号顺序申请资源，且只能按升序申请资源。
   - **一次性申请法**：进程在开始执行时一次性申请所有资源。
   - **银行家算法**：这是最著名的避免死锁的算法之一。它根据资源的当前可用量和进程的最大需求量，动态地分配资源，以避免死锁。

#### 死锁检测与解除

即使采取预防措施，死锁仍可能发生，因此我们还需要具备检测和解除死锁的能力。

1. **死锁检测**：
   - 使用资源分配图来检测死锁。资源分配图中的环路表示存在死锁。
   - 定期检查系统资源的分配状态，使用算法如银行家算法来检测潜在的死锁情况。
2. **死锁解除**：
   - **终止进程**：强制终止部分或全部死锁进程，释放它们占有的资源。
   - **撤销资源分配**：强制撤销部分进程的资源分配，让其他进程继续执行。

#### 实际编程中的防止死锁

在编写多线程程序时，使用正确的锁定顺序和避免嵌套锁定可以有效防止死锁。以下是一些常用的技巧：

1. **锁定顺序**：

   在多个线程需要多个锁时，所有线程必须按照相同的顺序获取锁。例如，线程A和线程B都需要锁L1和L2，那么它们必须都先获取L1再获取L2。

2. **尝试锁（try-lock）**：

   使用 `std::mutex::try_lock` 尝试获取锁，如果获取失败，则不会阻塞当前线程，而是可以选择稍后再试或采取其他措施。

3. **避免嵌套锁定**：

   尽量避免在一个锁的持有期间去获取另一个锁，或者在需要嵌套锁定时，严格控制锁的顺序。

4. **超时锁定**：

   使用支持超时的锁（如 `std::timed_mutex`），如果在指定时间内未能获取锁，则放弃获取，从而避免无限等待。

5. **锁的层次化管理**：

   为不同的资源定义不同的锁层次，严格按照层次获取锁，避免循环等待。

### 7、浏览器发送url到显示网页全过程？

1. 用户输入URL： 用户在浏览器的地址栏中输入URL（统一资源定位符），这是他们希望访问的网页的地址。
2. DNS解析： 浏览器首先需要解析URL中的主机名部分，以确定目标服务器的IP地址。这个过程称为DNS解析。如果浏览器有缓存，它可能首先查看本地缓存以获取IP地址。如果不在缓存中，浏览器将向本地DNS服务器发出请求，以获取目标主机的IP地址。本地DNS服务器可以向根DNS服务器、顶级域名服务器和权威DNS服务器逐级查询，最终找到目标主机的IP地址。
3. 建立TCP连接： 一旦浏览器获得了目标服务器的IP地址，它将尝试建立到该服务器的TCP连接。这个过程通常涉及三次握手，即浏览器向服务器发送一个连接请求，服务器确认请求并回复，然后浏览器再次确认。一旦建立了TCP连接，数据可以在浏览器和服务器之间进行可靠的双向通信。
4. 发起HTTP请求： 一旦TCP连接建立，浏览器会构建一个HTTP请求，该请求包括用户想要获取的特定页面的信息。HTTP请求通常包括请求方法（如GET、POST等）、请求头部和请求主体。
5. 服务器处理请求： 服务器接收到浏览器发送的HTTP请求后，会根据请求的内容执行相应的操作。这可能涉及查询数据库、处理业务逻辑、生成动态内容等。
6. 服务器发送HTTP响应： 服务器会生成一个HTTP响应，该响应包含请求的内容（通常是HTML页面）、状态码（表示请求是否成功、重定向等）以及响应头部信息。服务器还将此HTTP响应发送回浏览器。
7. 接收和渲染页面： 一旦浏览器收到HTTP响应，它会解析响应的内容，并根据HTML、CSS和JavaScript等数据渲染页面。这包括呈现文本、图像、链接和其他媒体，以及执行JavaScript代码。
8. 页面展示： 最终，浏览器将渲染后的页面呈现给用户，用户可以与页面进行交互。
9. 页面缓存： 浏览器通常会将页面的一些资源（如图像、样式表、脚本等）缓存，以便在用户再次访问相同页面时能够更快地加载。

### 8、tcp，udp区别？

#### 1. 基本概念

**TCP**：

- 全称是传输控制协议。
- 面向连接的协议，即在数据传输前需要建立连接。
- 提供可靠的、按顺序的、无差错的数据传输。
- 主要用于需要高可靠性的数据传输，如网页浏览、文件传输、电子邮件等。

**UDP**：

- 全称是用户数据报协议。
- 无连接的协议，即在数据传输前不需要建立连接。
- 提供不可靠的数据传输，数据包可能会丢失、重复或乱序。
- 主要用于需要快速传输且对可靠性要求不高的场景，如视频流、在线游戏、DNS查询等。

#### 2. 连接与数据传输

**TCP**：

- 连接建立：通过三次握手建立连接。
  1. 客户端发送SYN包。
  2. 服务器回应SYN-ACK包。
  3. 客户端回应ACK包，连接建立。
- **数据传输**：数据按顺序传输，使用序列号和确认机制确保数据包的到达和顺序。
- 连接终止：通过四次挥手断开连接。
  1. 一方发送FIN包表示终止连接。
  2. 对方回应ACK包。
  3. 对方发送FIN包表示同意断开连接。
  4. 一方回应ACK包，连接断开。

**UDP**：

- **无连接**：无需建立连接，直接发送数据。
- **数据传输**：数据以独立的报文形式发送，不保证顺序和可靠性。

#### 3. 可靠性

**TCP**：

- **可靠性**：通过确认应答（ACK）、重传机制、序列号等保证数据的可靠传输。
- **流量控制**：使用滑动窗口机制控制发送数据的速度，防止网络拥塞。
- **拥塞控制**：使用慢启动、拥塞避免、快重传和快恢复等算法控制网络拥塞。

**UDP**：

- **不可靠性**：没有确认应答和重传机制，数据包可能会丢失或乱序。
- **无流量控制和拥塞控制**：没有内置的流量控制和拥塞控制机制。

#### 4. 头部开销

**TCP**：

- **头部较大**：TCP头部通常是20字节，包含源端口、目的端口、序列号、确认号、数据偏移、保留位、控制位、窗口大小、校验和、紧急指针等字段。

**UDP**：

- **头部较小**：UDP头部通常是8字节，包含源端口、目的端口、长度和校验和等字段。

#### 5. 使用场景

**TCP**：

- **网页浏览**：需要可靠传输的HTTP/HTTPS协议。
- **文件传输**：如FTP、SFTP。
- **电子邮件**：如SMTP、IMAP、POP3。

**UDP**：

- **视频流**：如视频会议、直播。
- **在线游戏**：需要快速传输的实时数据。
- **DNS查询**：快速的域名解析。
- **VoIP**：实时语音通信。

### 9、想把udp变为可靠，怎么实现？

#### 1. 确认应答（Acknowledgment）

在发送数据包后，接收方需要发送确认应答（ACK）回给发送方，表示该数据包已成功接收。如果发送方在一定时间内没有收到ACK，则会重传该数据包。

#### 2. 重传机制（Retransmission）

发送方在发送数据包后启动一个定时器，如果在规定时间内没有收到接收方的ACK，则会重传该数据包。这个过程会持续直到接收到ACK或达到最大重传次数。

#### 3. 序列号（Sequence Number）

在每个数据包中添加一个序列号，以便接收方可以检测数据包的顺序并处理重复的数据包。序列号可以防止乱序问题，并确保数据按正确顺序组装。

#### 4. 超时处理（Timeout Handling）

在等待ACK的过程中，设置一个超时时间，如果超时则触发重传机制。

#### 5. 实现代码

以下是一个简单的可靠UDP实现示例代码，包含发送方和接收方的实现。

**可靠UDP发送方**（C++）：

```c++
#include <iostream>
#include <cstring>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>

#define PORT 8080
#define MAX_RETRIES 5
#define TIMEOUT 2 // seconds

struct Packet {
    int seq_num;
    char data[1024];
};

void reliable_send(int sockfd, sockaddr_in &server_addr, const char* message) {
    socklen_t addr_len = sizeof(server_addr);
    int seq_num = 0;
    Packet packet;
    packet.seq_num = seq_num;
    strcpy(packet.data, message);

    while (true) {
        sendto(sockfd, &packet, sizeof(packet), 0, (struct sockaddr*)&server_addr, addr_len);
        
        fd_set fds;
        FD_ZERO(&fds);
        FD_SET(sockfd, &fds);
        struct timeval tv;
        tv.tv_sec = TIMEOUT;
        tv.tv_usec = 0;

        int retval = select(sockfd + 1, &fds, NULL, NULL, &tv);
        if (retval == -1) {
            std::cerr << "Select error." << std::endl;
            return;
        } else if (retval) {
            int ack;
            recvfrom(sockfd, &ack, sizeof(ack), 0, (struct sockaddr*)&server_addr, &addr_len);
            if (ack == seq_num) {
                std::cout << "Acknowledgment received for seq_num: " << seq_num << std::endl;
                break;
            }
        } else {
            std::cerr << "Timeout, resending packet with seq_num: " << seq_num << std::endl;
        }
    }
}

int main() {
    int sockfd = socket(AF_INET, SOCK_DGRAM, 0);
    if (sockfd < 0) {
        std::cerr << "Socket creation failed." << std::endl;
        return 1;
    }

    sockaddr_in server_addr;
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(PORT);
    server_addr.sin_addr.s_addr = inet_addr("127.0.0.1");

    const char* message = "Hello, reliable UDP!";
    reliable_send(sockfd, server_addr, message);

    close(sockfd);
    return 0;
}
```

**可靠UDP接收方**（C++）：

```c++
#include <iostream>
#include <cstring>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>

#define PORT 8080

struct Packet {
    int seq_num;
    char data[1024];
};

int main() {
    int sockfd = socket(AF_INET, SOCK_DGRAM, 0);
    if (sockfd < 0) {
        std::cerr << "Socket creation failed." << std::endl;
        return 1;
    }

    sockaddr_in server_addr, client_addr;
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(PORT);
    server_addr.sin_addr.s_addr = INADDR_ANY;

    if (bind(sockfd, (const struct sockaddr *)&server_addr, sizeof(server_addr)) < 0) {
        std::cerr << "Bind failed." << std::endl;
        close(sockfd);
        return 1;
    }

    socklen_t addr_len = sizeof(client_addr);
    Packet packet;

    while (true) {
        recvfrom(sockfd, &packet, sizeof(packet), 0, (struct sockaddr*)&client_addr, &addr_len);
        std::cout << "Received packet with seq_num: " << packet.seq_num << ", data: " << packet.data << std::endl;

        int ack = packet.seq_num;
        sendto(sockfd, &ack, sizeof(ack), 0, (struct sockaddr*)&client_addr, addr_len);
        std::cout << "Acknowledgment sent for seq_num: " << ack << std::endl;
    }

    close(sockfd);
    return 0;
}
```

### 10、有碰到过core dump吗，怎么排查？

core dump（核心转储）是指当程序发生严重错误（如段错误）时，操作系统会将程序的内存映像保存到一个称为core文件的文件中，以便后续调试。排查core dump问题通常需要以下步骤：

1. **确认core文件生成**：首先要确认core文件是否生成。可以通过查看程序运行的目录或者使用`ulimit -c unlimited`设置允许生成core文件，并确保程序有足够的权限生成core文件。
2. **查看core文件**：可以使用`file core`命令查看core文件类型，使用`gdb <executable> core`命令打开core文件进行调试。
3. **使用GDB调试**：使用GDB调试core文件，可以查看导致程序崩溃的位置和原因。例如，可以使用`bt`命令查看函数调用栈，`info registers`命令查看寄存器的值，`list`命令查看导致错误的代码位置等。
4. **分析错误信息**：根据GDB输出的信息，可以确定导致core dump的具体原因。可能是空指针引用、内存越界、未初始化变量等。
5. **修复问题**：根据分析结果，修复程序中导致core dump的问题，可能需要修改代码逻辑、添加错误检查或者优化内存管理等。
6. **防止再次发生**：对程序进行全面的测试，确保修复后不会再次触发core dump。可以使用静态分析工具、动态检查工具等辅助排查。

### 11、会gdb调试吗？

#### 1. 编译程序时添加调试信息

在编译程序时，需要添加`-g`选项来生成调试信息，这样GDB才能正确解析程序的源代码和符号信息。例如：

```
gcc -g -o my_program my_program.c
```

#### 2. 启动GDB

启动GDB并指定要调试的可执行文件：

```
gdb ./my_program
```

#### 3. 设置断点

在需要调试的地方设置断点，可以是函数名、行号或者文件名加行号的形式。例如，在第10行设置断点：

```
(gdb) break 10
```

#### 4. 运行程序

使用`run`命令运行程序，程序会在设置的断点处停止执行：

```
(gdb) run
```

#### 5. 执行调试命令

在程序暂停时，可以使用多种命令来查看变量、调用堆栈等信息：

- `print <variable>`：打印变量的值。
- `backtrace`（简写为`bt`）：打印调用堆栈。
- `step`（简写为`s`）：单步执行，进入函数。
- `next`（简写为`n`）：单步执行，不进入函数。
- `continue`（简写为`c`）：继续执行直到下一个断点。
- `finish`：执行完当前函数并停止在调用该函数的地方。

#### 6. 修改变量值

在调试过程中，可以使用`set`命令修改变量的值：

```
(gdb) set variable_name = new_value
```

#### 7. 查看内存

使用`x`命令查看内存中的内容：

```
(gdb) x /<格式> <地址>
```

例如，查看地址0x123456处的4个字节的内容：

```
(gdb) x /4x 0x123456
```

#### 8. 退出GDB

调试完成后，可以使用`quit`命令退出GDB：

```
(gdb) quit
```

### 12、linux基本命令会哪些，介绍一下？



### 13、mysql和redis区别，关系型数据库和非关系型数据库区别？

#### MySQL vs. Redis

**MySQL**：

- **类型**：关系型数据库管理系统（RDBMS）。
- **数据模型**：采用表格模型，数据以表格的形式存储，表格之间通过关系进行连接。
- **查询语言**：结构化查询语言（SQL）。
- **数据持久性**：数据持久存储在硬盘上，支持事务处理，适用于需要保持数据完整性和一致性的场景。
- **应用场景**：适用于需要复杂查询、事务处理和数据一致性的应用，如企业应用、电子商务等。

**Redis**：

- **类型**：非关系型内存数据库（NoSQL）。
- **数据模型**：采用键值对（key-value）模型，数据以键值对的形式存储。
- **查询语言**：支持丰富的数据结构操作命令，如字符串、列表、集合、哈希表、有序集合等。
- **数据持久性**：支持持久化，可以将数据存储在硬盘上，也可以仅存储在内存中，适用于需要快速读写和高并发的场景。
- **应用场景**：适用于缓存、消息队列、实时数据分析等场景，能够提供快速读写和高并发访问。

#### 关系型数据库 vs. 非关系型数据库

**关系型数据库**：

- **数据模型**：采用表格模型，数据以表格的形式存储，表格之间通过关系进行连接。
- **数据一致性**：保持数据的一致性和完整性，支持事务处理。
- **查询语言**：使用结构化查询语言（SQL）进行数据查询和操作。
- **适用场景**：适用于需要复杂查询、事务处理和数据一致性的应用，如企业应用、电子商务等。

**非关系型数据库**：

- **数据模型**：不采用传统的表格模型，通常采用键值对、文档、列族、图形等数据模型。
- **数据一致性**：灵活性更高，可以根据需求选择强一致性或最终一致性。
- **查询语言**：不同类型的非关系型数据库使用不同的查询语言或数据操作接口。
- **适用场景**：适用于需要高性能、高可扩展性和灵活性的应用，如大数据处理、实时数据分析等。

### 14、redis基本数据结构？

1. **字符串（String）**：最基本的数据结构，可以存储字符串、整数或者浮点数。

   常用命令：SET、GET、INCR、DECR等。

2. **哈希表（Hash）**：类似于关联数组，存储字段和对应的值之间的映射。

   常用命令：HSET、HGET、HDEL、HGETALL等。

3. **列表（List）**：按照插入顺序存储多个元素的列表。

   常用命令：LPUSH、RPUSH、LPOP、RPOP、LRANGE等。

4. **集合（Set）**：无序且不重复的元素集合。

   常用命令：SADD、SREM、SISMEMBER、SMEMBERS等。

5. **有序集合（Sorted Set）**：类似于集合，但每个元素都关联一个分数（score），用于排序。

   常用命令：ZADD、ZREM、ZSCORE、ZRANGE等。

6. **位图（Bitmap）**：可以进行位级操作的数据结构，用于存储位的集合。

   常用命令：SETBIT、GETBIT、BITCOUNT、BITOP等。

### 15、手撕单例模式？

### 16、简单多线程，一个count两个线程轮流count++？

```c++
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx;  // 互斥锁，用于保护计数器
int count = 0;   // 共享的计数器

// 线程函数，对计数器进行增加操作
void incrementCount(int id) {
    for (int i = 0; i < 5; ++i) {
        mtx.lock();  // 加锁
        std::cout << "Thread " << id << " is incrementing count to " << ++count << std::endl;
        mtx.unlock();  // 解锁
        std::this_thread::sleep_for(std::chrono::milliseconds(100));  // 线程休眠
    }
}

int main() {
    // 创建两个线程
    std::thread t1(incrementCount, 1);
    std::thread t2(incrementCount, 2);

    // 等待线程执行完毕
    t1.join();
    t2.join();

    std::cout << "Final count is " << count << std::endl;

    return 0;
}
```

