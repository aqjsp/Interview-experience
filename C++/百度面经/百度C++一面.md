# 百度C++一面

> 来源：https://www.nowcoder.com/discuss/727458381765558272

## 1、有用到c++的atomic吗？ atomic的内存序问题？ 

#### 什么是 std::atomic？

 C++11 引入的模板类，位于 <atomic> 头文件中，用于封装基本数据类型（如 int、bool）或自定义类型，提供原子操作。

##### 作用

- 确保多线程对变量的读写操作不被中断。
- 避免未定义行为（如数据竞争）。

##### 常见操作

- load()：原子读取。
- store()：原子写入。
- fetch_add()、fetch_sub()：原子增减。
- compare_exchange_weak/strong()：比较并交换（CAS）。

##### 基本使用

```c++
#include <iostream>
#include <atomic>
#include <thread>

std::atomic<int> counter(0); // 原子变量

void increment() {
    for (int i = 0; i < 1000; ++i) {
        counter.fetch_add(1); // 原子加1
    }
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);
    t1.join();
    t2.join();
    std::cout << "Counter: " << counter.load() << std::endl; // 输出 2000
    return 0;
}
```

如果没有 std::atomic，多线程对普通 int 的 ++ 操作可能导致结果不正确（如小于 2000）。fetch_add 保证原子性。

#### 内存序问题

`std::atomic` 的原子操作不仅保证操作本身的完整性，还涉及线程间内存访问的顺序。这种顺序由内存序控制，C++ 提供了 6 种内存序选项，定义在 std::memory_order 枚举中。内存序决定了操作的同步成本和性能。

##### 内存序的种类

1. memory_order_seq_cst（顺序一致性）：
   - 默认内存序，最严格。
   - 所有线程看到的内存操作顺序一致（全局总序）。
   - 类似单线程执行的直觉。
   - 开销：最高，可能触发硬件同步指令（如 x86 的 mfence）。
2. memory_order_acquire（获取序）：
   - 用于读取操作（如 load）。
   - 保证当前操作之后的内存操作不会被重排序到之前。
   - 用途：确保读取到最新值后，后续操作基于此值。
3. memory_order_release（释放序）：
   - 用于写入操作（如 store）。
   - 保证当前操作之前的内存操作不会被重排序到之后。
   - 用途：确保写入前所有操作完成，供其他线程读取。
4. memory_order_acq_rel（获取-释放序）：
   - 结合 acquire 和 release，用于读-改-写操作（如 fetch_add）。
   - 保证读到最新值并写入时同步。
5. memory_order_consume（消费序）：
   - 比 acquire 更弱，只同步依赖当前值的操作。
   - 用途：减少不必要同步（较少使用，因复杂且硬件支持有限）。
6. memory_order_relaxed（松散序）：
   - 最宽松，仅保证原子性，不强制线程间顺序。
   - 用途：性能优先，需手动同步。

## 2、linux命令，查看进程、内存、磁盘 ?

#### 查看进程

ps命令：显示进程信息。

常用选项：

- ps aux：列出所有进程（BSD 风格）。
- ps -ef：列出所有进程（System V 风格）。

top 命令：实时显示进程状态，类似任务管理器。

pidstat 命令：详细统计进程的 CPU、内存、I/O 使用。

#### 查看内存

free 命令：查看内存和交换区使用。

vmstat 命令：实时统计内存、CPU、I/O。

cat /proc/meminfo：查看详细内存信息。

#### 查看磁盘

df：显示磁盘空间使用。

du：统计目录或文件大小。

lsblk：列出块设备。

iostat：磁盘 I/O 性能。

## 3、设计模式原则、法则？

#### 原则

##### (1) 单一职责原则

一个类应该只有一个引起它变化的原因。

作用：减少耦合，提高内聚。

#####  (2) 开放-封闭原则

软件实体应可扩展但不可修改。

作用：支持新功能添加而不破坏现有代码。

##### (3) 里氏替换原则

子类可以替换基类而不影响程序正确性。

作用：保证继承体系的语义一致性。

##### (4) 接口隔离原则

客户端不应依赖不需要的接口。

作用：避免接口臃肿，降低依赖。

##### (5) 依赖倒置原则

高层模块不依赖低层模块，二者依赖抽象。

作用：解耦模块，提升灵活性。

##### (6) 最少知识原则

只与直接关联的对象交互，减少耦合。

#### 法则

##### (1) KISS 法则

保持简单，避免过度设计。

作用：防止模式滥用导致代码复杂。

##### (2) DRY 法则

避免重复代码。

作用：提高代码复用性。

##### (3) YAGNI 法则

不要实现当前不需要的功能。

作用：避免过度设计。

##### (4) 高内聚低耦合

- 高内聚：模块内部功能集中。
- 低耦合：模块间依赖最小。

作用：提高可维护性和独立性。

## 5、子线程结束后如何通知主进程去执行一个方法？ 

## 4、条件变量知道吗？

一种同步原语，允许线程在某个条件不满足时等待（阻塞），并在条件满足时被其他线程唤醒。

作用：

- 等待：线程暂停执行，直到特定条件成立。
- 通知：当条件变化时，通知等待的线程继续执行。

#### 工作原理

##### 基本操作

1. wait（等待）：
   - 线程检查条件（如队列是否为空）。
   - 若条件不满足，调用 wait 阻塞自己，同时释放关联的锁。
   - 被唤醒后，自动重新获取锁。
2. notify / signal（通知）：当条件满足时，其他线程调用通知操作，唤醒一个或所有等待线程。
3. notify_all / broadcast（广播）：唤醒所有等待线程。

##### 工作流程（以生产者-消费者为例）

- 消费者：
  1. 加锁检查队列。
  2. 若队列为空，调用 wait 等待（释放锁）。
  3. 被唤醒后重新加锁，消费数据。
- 生产者：
  1. 加锁添加数据到队列。
  2. 调用 notify 唤醒消费者。
  3. 释放锁。

#### C++ 中的条件变量

生产者-消费者模型

```c++
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <queue>

std::mutex mtx;                // 互斥锁
std::condition_variable cv;    // 条件变量
std::queue<int> data_queue;    // 共享队列
bool finished = false;         // 结束标志

void producer() {
    for (int i = 1; i <= 5; ++i) {
        std::this_thread::sleep_for(std::chrono::milliseconds(100)); // 模拟生产
        {
            std::lock_guard<std::mutex> lock(mtx); // 加锁
            data_queue.push(i);
            std::cout << "Produced: " << i << std::endl;
        }
        cv.notify_one(); // 通知一个等待线程
    }
    {
        std::lock_guard<std::mutex> lock(mtx);
        finished = true; // 生产结束
    }
    cv.notify_all(); // 通知所有线程
}

void consumer() {
    while (true) {
        std::unique_lock<std::mutex> lock(mtx); // 灵活锁
        // 等待条件：队列不为空或生产结束
        cv.wait(lock, [] { return !data_queue.empty() || finished; });
        
        if (finished && data_queue.empty()) {
            break; // 结束
        }
        if (!data_queue.empty()) {
            int data = data_queue.front();
            data_queue.pop();
            std::cout << "Consumed: " << data << std::endl;
        }
        lock.unlock(); // 手动解锁（可选）
    }
}

int main() {
    std::thread prod(producer);
    std::thread cons(consumer);
    prod.join();
    cons.join();
    return 0;
}
```

输出：

```c++
Produced: 1
Consumed: 1
Produced: 2
Consumed: 2
Produced: 3
Consumed: 3
Produced: 4
Consumed: 4
Produced: 5
Consumed: 5
```

#### 与其他同步机制的对比

| 机制                | 优点               | 缺点                   | 适用场景                |
| ------------------- | ------------------ | ---------------------- | ----------------------- |
| 条件变量            | 高效等待，灵活通知 | 需配合锁，复杂性高     | 生产者-消费者，状态同步 |
| 互斥锁（Mutex）     | 简单，保护共享数据 | 无等待机制，仅互斥     | 简单并发访问            |
| 信号量（Semaphore） | 支持计数，简单同步 | 语义较弱，无条件检查   | 资源计数                |
| 忙等待（Spinlock）  | 低延迟（短临界区） | 浪费 CPU，适合极短等待 | 高性能实时系统          |

## 5、智能指针讲一下？

#### (1)` std::auto_ptr`（已废弃）

##### 原理

- 所有权单一：auto_ptr 管理一个动态分配的对象，指针被复制时，原指针失去所有权（置为 nullptr）。
- 实现：通过拷贝构造函数和赋值操作符转移所有权。

##### 示例代码

```c++
#include <iostream>
#include <memory>

void example_auto_ptr() {
    std::auto_ptr<int> ptr1(new int(10));
    std::cout << *ptr1 << std::endl; // 输出 10

    std::auto_ptr<int> ptr2 = ptr1;  // 转移所有权
    std::cout << (ptr1.get() == nullptr) << std::endl; // 输出 1（ptr1 为空）
    std::cout << *ptr2 << std::endl; // 输出 10
}
```

##### 特性

- 转移语义：赋值时所有权转移，原指针失效。
- 析构：对象超出作用域时自动释放。

##### 问题

- 不可预测性：赋值后原指针变空，可能导致逻辑错误。
- 不支持容器：转移语义与 STL 容器（如 std::vector）不兼容。
- 无线程安全：未考虑多线程场景。

#### (2) `std::unique_ptr`

独占所有权的智能指针，确保单一对象只有一个管理者。

##### 原理

- 独占所有权：通过移动语义（move semantics）转移所有权，禁止拷贝。
- 实现：基于 RAII（资源获取即初始化），析构时自动释放内存。

##### 示例

```c++
#include <iostream>
#include <memory>

void example_unique_ptr() {
    // 创建 unique_ptr
    std::unique_ptr<int> ptr1 = std::make_unique<int>(10); // C++14 推荐
    std::cout << *ptr1 << std::endl; // 输出 10

    // 转移所有权
    std::unique_ptr<int> ptr2 = std::move(ptr1);
    std::cout << (ptr1.get() == nullptr) << std::endl; // 输出 1
    std::cout << *ptr2 << std::endl; // 输出 10

    // ptr2 超出作用域时自动释放
}
```

##### 特性

- 移动语义：

  - 不能拷贝（拷贝构造函数删除），只能移动。
  - 使用 std::move 显式转移所有权。

- 自定义删除器：支持指定删除函数（如释放文件句柄）。

  ```c++
  auto deleter = [](FILE* fp) { fclose(fp); };
  std::unique_ptr<FILE, decltype(deleter)> fp(fopen("test.txt", "r"), deleter);
  ```

- 高效：无额外开销，仅封装裸指针。

##### 优点

- 安全性：杜绝意外拷贝导致的双重释放。
- 灵活性：支持数组（unique_ptr<T[]>）和自定义删除器。
- 性能：与裸指针接近，无引用计数。

#### (3) `std::shared_ptr`

共享所有权的智能指针，允许多个指针管理同一对象。

##### 原理

- 引用计数：内部维护一个计数器，记录有多少 shared_ptr 指向同一对象。
- 析构：引用计数减为 0 时释放内存。

##### 示例

```c++
#include <iostream>
#include <memory>

void example_shared_ptr() {
    std::shared_ptr<int> ptr1 = std::make_shared<int>(20); // C++14 推荐
    std::cout << *ptr1 << " (count: " << ptr1.use_count() << ")" << std::endl; // 输出 20 (count: 1)

    {
        std::shared_ptr<int> ptr2 = ptr1; // 共享所有权
        std::cout << *ptr2 << " (count: " << ptr2.use_count() << ")" << std::endl; // 输出 20 (count: 2)
    }
    std::cout << *ptr1 << " (count: " << ptr1.use_count() << ")" << std::endl; // 输出 20 (count: 1)

    // ptr1 超出作用域时释放
}
```

##### 特性

- 引用计数：

  - use_count() 返回当前引用数。
  - 线程安全：计数器操作是原子的，但对象访问需额外同步。

- 自定义删除器：

  ```c++
  std::shared_ptr<int> ptr(new int(30), [](int* p) { delete p; std::cout << "Deleted\n"; });
  ```

- 支持数组：C++17 起支持 shared_ptr<T[]>。

##### 优点

- 共享性：多个指针可安全引用同一对象。
- 自动化：引用计数为 0 时自动释放。

##### 缺点

- 性能开销：引用计数增加内存和计算成本。
- 循环引用：可能导致内存泄漏（需用 weak_ptr 解决）。

#### (4) `std::weak_ptr`

辅助 shared_ptr，解决循环引用问题，不拥有对象所有权。

##### 原理

- 弱引用：不增加引用计数，仅观察 shared_ptr 管理的对象。
- 检查有效性：通过 lock() 获取 shared_ptr，若对象已释放则返回空。

##### 示例用法

```c++
#include <iostream>
#include <memory>

class Node {
public:
    std::shared_ptr<Node> next;
    std::weak_ptr<Node> prev; // 避免循环引用
    ~Node() { std::cout << "Node destroyed\n"; }
};

void example_weak_ptr() {
    std::shared_ptr<Node> node1 = std::make_shared<Node>();
    std::shared_ptr<Node> node2 = std::make_shared<Node>();

    node1->next = node2;
    node2->prev = node1; // weak_ptr 不增加引用计数

    std::cout << "node1 count: " << node1.use_count() << std::endl; // 输出 1
    std::cout << "node2 count: " << node2.use_count() << std::endl; // 输出 1

    // node1 和 node2 超出作用域时正常析构
}
```

##### 特性

- 不控制生命周期：不影响引用计数。
- 访问方式：
  - lock()：返回 shared_ptr，若对象已释放则为空。
  - expired()：检查对象是否已销毁。
- 解决循环引用：
  - shared_ptr 互相引用时，计数永不为 0。
  - 用 weak_ptr 打破循环。

##### 优点

- 安全性：避免内存泄漏。
- 灵活性：动态检查资源可用性。

##### 缺点

- 依赖性：必须与 shared_ptr 搭配。
- 复杂性：使用需额外检查。

#### 四种智能指针对比

| 智能指针   | 所有权       | 引用计数 | 拷贝 | 主要用途               | 状态       |
| ---------- | ------------ | -------- | ---- | ---------------------- | ---------- |
| auto_ptr   | 独占         | 无       | 转移 | 简单独占管理（已废弃） | C++17 移除 |
| unique_ptr | 独占         | 无       | 移动 | 独占资源，工厂模式     | 推荐使用   |
| shared_ptr | 共享         | 有       | 拷贝 | 共享资源，多线程       | 常用       |
| weak_ptr   | 无（弱引用） | 无       | 拷贝 | 解决循环引用，临时观察 | 辅助工具   |

## 6、动态库和静态库的区别？

#### 基本概念

##### 静态库

一组目标文件（.o 文件）的集合，通常以 .a（Linux）或 .lib（Windows）为后缀，在程序链接时被直接嵌入到可执行文件中。

##### 动态库

独立的可执行文件，通常以 .so（Linux, Shared Object）或 .dll（Windows, Dynamic Link Library）为后缀，在程序运行时动态加载。

#### 生成与使用

##### 静态库

1. 生成：

   - 编译源文件为目标文件：

     ```c
     g++ -c math.cpp -o math.o
     ```

   - 打包成静态库：

     ```
     ar rcs libmath.a math.o
     ```

2. 链接：

   ```
   g++ main.cpp -L. -lmath -o myprogram
   ```

   -L.：指定库路径。

   -lmath：链接 libmath.a。

##### 动态库

1. 生成：

   - 编译为位置无关代码（PIC）：

     ```
     g++ -fPIC -c math.cpp -o math.o
     ```

   - 创建动态库：

     ```
     g++ -shared -o libmath.so math.o
     ```

2. 链接：

   ```
   g++ main.cpp -L. -lmath -o myprogram
   ```

   运行时需指定库路径：

   ```
   export LD_LIBRARY_PATH=.:$LD_LIBRARY_PATH
   ./myprogram
   ```

#### 区别

##### (1) 链接时机

静态库：

- 在编译链接阶段嵌入到可执行文件中。
- 可执行文件包含库代码，成为独立实体。

动态库：

- 在运行时动态加载。
- 可执行文件仅记录动态库的引用，需操作系统支持。

##### (2) 文件大小

静态库：

- 可执行文件较大，因包含所有库代码。
- 示例：libmath.a 10KB，程序变大 10KB。

动态库：

- 可执行文件较小，仅记录符号表。
- 动态库独立存在，多个程序可共享。

##### (3) 内存使用

静态库：

- 每个程序加载时，库代码占用独立内存。
- 示例：10 个程序用同一静态库，内存占用 10 份。

动态库：

- 动态库加载到内存后，多个程序共享同一份。
- 示例：10 个程序共享 libmath.so，内存仅一份。

##### (4) 优缺点

| 特性 | 静态库                                                  | 动态库                                            |
| ---- | ------------------------------------------------------- | ------------------------------------------------- |
| 优点 | 1. 自包含，无运行时依赖<br>2. 启动快<br>3. 移植性强     | 1. 文件小，节省磁盘<br>2. 内存共享<br>3. 更新方便 |
| 缺点 | 1. 文件大，重复嵌入<br>2. 更新需重编译<br>3. 内存占用高 | 1. 运行时依赖<br>2. 加载稍慢<br>3. 版本兼容问题   |

## 7、虚函数，虚函数表，虚函数指针讲一下？

#### 虚函数

##### 定义

- 虚函数是 C++ 中用 virtual 关键字声明的成员函数，允许在基类指针或引用调用时，根据实际对象类型执行派生类的实现。
- 作用：实现动态绑定（Dynamic Binding），支持多态。

##### 语法

```c++
class Base {
public:
    virtual void func() {
        std::cout << "Base::func" << std::endl;
    }
};

class Derived : public Base {
public:
    void func() override { // override 可选，C++11 增强可读性
        std::cout << "Derived::func" << std::endl;
    }
};
```

##### 特性

- 动态绑定：
  - 非虚函数：编译时绑定（静态绑定），调用基类版本。
  - 虚函数：运行时绑定，调用实际对象类型的版本。
- 继承性：一旦基类函数声明为 virtual，派生类中同名函数自动为虚函数。
- 析构函数：基类析构函数通常声明为虚函数，避免通过基类指针删除对象时只调用基类析构。

##### 示例代码

```c++
#include <iostream>
class Base {
public:
    virtual void func() { std::cout << "Base\n"; }
    virtual ~Base() = default; // 虚析构
};

class Derived : public Base {
public:
    void func() override { std::cout << "Derived\n"; }
};

int main() {
    Base* ptr = new Derived();
    ptr->func(); // 输出 "Derived"
    delete ptr;  // 正确调用 Derived 和 Base 的析构
    return 0;
}
```

#### 虚函数表

##### 定义

- 一个由编译器为每个具有虚函数的类生成的函数指针数组，存储该类所有虚函数的地址。
- 作用：在运行时根据对象类型查找正确的函数地址，实现多态。

##### 实现原理

- 生成：
  - 编译器在编译时为每个类创建一个静态的虚函数表。
  - 表中每个条目指向类的虚函数实现。
- 存储：虚函数表是全局唯一的，每个类一份，位于程序的只读数据段（.rodata）。
- 结构：如果基类有 2 个虚函数，派生类覆盖 1 个，则派生类的 vtable 继承并更新对应条目。

#### 虚函数指针

##### 定义

- 虚函数指针是每个对象内部的一个隐藏指针，指向该对象所属类的虚函数表。
- 作用：在运行时通过 vptr 查找 vtable，再调用正确的虚函数。

##### 实现原理

- 位置：
  - vptr 通常位于对象内存布局的开头（由编译器决定）。
  - 每个对象实例都有一个 vptr，指向其类的 vtable。
- 初始化：
  - 在对象构造时，编译器在构造函数中设置 vptr。
  - 派生类构造时，覆盖基类的 vptr。

## 8、继承和组合 ？

#### 继承

##### 定义

- 继承是指一个类（子类/派生类）从另一个类（父类/基类）派生，继承其属性和方法，从而复用父类的代码并扩展功能。
- 关键字：C++ 中使用 class Derived : public Base。

##### 原理

- is-a 关系：子类是父类的一种具体类型（如“狗是动物”）。
- 实现：
  - 子类自动获得父类的非私有成员（public 和 protected）。
  - 可通过虚函数实现多态。

##### 示例代码

```c++
#include <iostream>

class Animal {
public:
    void eat() {
        std::cout << "Animal eats" << std::endl;
    }
    virtual void sound() {
        std::cout << "Animal makes sound" << std::endl;
    }
};

class Dog : public Animal {
public:
    void sound() override { // 覆盖父类方法
        std::cout << "Dog barks" << std::endl;
    }
    void fetch() { // 子类特有方法
        std::cout << "Dog fetches" << std::endl;
    }
};

int main() {
    Dog dog;
    dog.eat();   // 输出 "Animal eats"
    dog.sound(); // 输出 "Dog barks"
    dog.fetch(); // 输出 "Dog fetches"

    Animal* animalPtr = &dog;
    animalPtr->sound(); // 多态，输出 "Dog barks"
    return 0;
}
```

#### 组合

##### 定义

- 组合是指一个类通过包含其他类的对象（成员变量）来使用其功能，构建“has-a”关系。
- 关键字：无特定语法，通过成员变量实现。

##### 原理

- has-a 关系：对象拥有其他对象作为组成部分（如“汽车有引擎”）。
- 实现：
  - 类内部实例化或引用其他类的对象。
  - 通过调用成员对象的方法实现功能。

##### 示例代码

```c++
#include <iostream>

class Engine {
public:
    void start() {
        std::cout << "Engine starts" << std::endl;
    }
};

class Car {
private:
    Engine engine; // 组合：Car 包含 Engine
public:
    void drive() {
        engine.start(); // 使用 Engine 的功能
        std::cout << "Car drives" << std::endl;
    }
};

int main() {
    Car car;
    car.drive(); // 输出 "Engine starts" 和 "Car drives"
    return 0;
}
```

#### 区别表

| 特性     | 继承                 | 组合                       |
| -------- | -------------------- | -------------------------- |
| 关系类型 | is-a（是一种）       | has-a（拥有）              |
| 耦合性   | 高（子类依赖父类）   | 低（通过接口交互）         |
| 复用方式 | 直接继承父类代码     | 通过成员对象调用           |
| 多态性   | 支持（虚函数）       | 不直接支持（需接口或代理） |
| 灵活性   | 较低（绑定父类实现） | 较高（可替换成员）         |
| 代码结构 | 层次结构，可能复杂   | 扁平结构，清晰             |
| 维护性   | 可能因父类变化受影响 | 改动局部，不易影响整体     |

## 9、讲一下stl中map和unordered_map的区别？

#### 基本定义

##### `std::map`

- 头文件：<map>
- 定义：有序关联容器，键值对按键（key）排序存储。
- 底层实现：红黑树（Red-Black Tree），一种自平衡二叉搜索树。

##### `std::unordered_map`

- 头文件：<unordered_map>
- 定义：无序关联容器，键值对根据键的哈希值存储。
- 底层实现：哈希表（Hash Table）。

#### 底层实现原理

##### `std::map` - 红黑树

- 数据结构：
  - 红黑树是一种自平衡二叉搜索树，保证树的深度差不超过两倍。
  - 每个节点存储键值对（std::pair<const Key, T>）。
- 排序：
  - 键按升序（默认使用 std::less<Key>）排列。
  - 支持自定义比较器（如 std::greater<Key>）。
- 操作：插入、删除、查找的时间复杂度为 O(log n)，因为需要遍历树的高度。

##### `std::unordered_map` - 哈希表

- 数据结构：
  - 使用哈希表，键通过哈希函数映射到桶（bucket）。
  - 每个桶可能包含链表或动态数组（处理冲突）。
- 无序：键不排序，存储位置由哈希值决定。
- 操作：平均时间复杂度为 O(1)，但最坏情况（哈希冲突严重）为 O(n)。

#### 时间复杂度

| 操作 | `std::map` | `std::unordered_map` |
| ---- | ---------- | -------------------- |
| 插入 | O(log n)   | O(1) 平均，O(n) 最坏 |
| 删除 | O(log n)   | O(1) 平均，O(n) 最坏 |
| 查找 | O(log n)   | O(1) 平均，O(n) 最坏 |
| 遍历 | O(n)       | O(n)                 |

- `map`：稳定且一致，因红黑树高度平衡。
- `unordered_map`：依赖哈希函数质量，冲突多时退化为线性。

## 10、一个pair对象是否可以作为map的key？ 

先给结论，可以，std::pair 默认支持 <，满足 std::map 的键要求。

#### std::map 对键的要求

std::map 是一个有序关联容器，其底层实现是红黑树。

为了在红黑树中维护键的顺序，std::map 对键类型（Key）有以下要求：

- 可比较：键类型必须支持严格弱序（Strict Weak Ordering）的比较操作。
- 默认比较器：std::map 使用 std::less<Key>，要求键支持 < 运算符。
- 自定义比较器：可以通过模板参数指定其他比较器。

#### std::pair 的比较机制

std::pair 是一个包含两个成员（first 和 second）的结构体，STL 为其提供了默认的比较运算符，包括 <。

##### std::pair 的 < 运算符

比较规则：

- 先比较 first：
  - 如果 first1 < first2，则 pair1 < pair2。
  - 如果 first1 > first2，则 pair1 > pair2。
- 若 first 相等，则比较 second：如果 second1 < second2，则 pair1 < pair2。

字典序（Lexicographical Order）：类似字母序，先比较第一个元素，相等时再比较第二个元素。

##### 条件

- std::pair 的 first 和 second 类型必须支持 < 运算符。
- 常见类型（如 int、std::string、double）默认支持。

## 11、stl迭代器了解吗？ 

迭代器是一种对象，提供了访问容器元素的方式，类似于广义的指针。

作用：

- 遍历容器元素。
- 提供统一的接口，使算法（如 std::sort、std::find）与容器无关。

#### 迭代器的分类

##### (1) 输入迭代器

- 特点：只读，单向遍历，只能前进（++）。
- 操作：*it（读）、it++、it == it2。
- 用途：从输入流（如 std::istream_iterator）读取数据。

##### (2) 输出迭代器

- 特点：只写，单向遍历，只能前进。
- 操作：*it = value（写）、it++。
- 用途：向输出流（如 std::ostream_iterator）写入数据。

##### (3) 前向迭代器

- 特点：读写，单向遍历，支持多次遍历。
- 操作：*it（读写）、it++、it == it2。
- 用途：适用于单向容器（如 std::forward_list）。

##### (4) 双向迭代器

- 特点：读写，双向遍历（前进 ++ 和后退 --）。
- 操作：*it、it++、it--、it == it2。
- 用途：适用于双向容器（如 std::list、std::map）。

##### (5) 随机访问迭代器

- 特点：读写，支持随机访问（跳跃式移动）。
- 操作：*it、it++、it--、it + n、it - n、it[n]、it < it2。
- 用途：适用于连续存储容器（如 std::vector、std::array）。

#### 实现原理

##### 底层实现

- 指针封装：对于 std::vector，迭代器可能是原始指针（T*）的封装。
- 自定义类：对于复杂容器（如 std::list、std::map），迭代器是封装了节点指针的类。

##### 接口要求

- 容器通过 begin() 和 end() 返回迭代器。
- 迭代器必须实现必要的运算符（*、++ 等），根据类别不同而变化。

#### 基本用法

```c++
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};

    // 使用迭代器遍历
    for (auto it = vec.begin(); it != vec.end(); ++it) {
        std::cout << *it << " "; // 输出 1 2 3 4 5
    }
    std::cout << std::endl;

    // 修改元素
    for (auto it = vec.begin(); it != vec.end(); ++it) {
        *it *= 2;
    }

    // 范围 for（底层基于迭代器）
    for (int x : vec) {
        std::cout << x << " "; // 输出 2 4 6 8 10
    }
    return 0;
}
```

#### 迭代器失效场景

##### (1) 容量改变

`std::vector`：`push_back`可能触发重新分配，所有迭代器失效。

```c++
std::vector<int> vec = {1, 2};
auto it = vec.begin();
vec.push_back(3); // it 可能失效
```

##### (2) 元素删除

`std::list`：删除元素只使指向该元素的迭代器失效。

`std::map`：删除使指向该键的迭代器失效，其他不受影响。

##### (3) 插入

`std::unordered_map`：插入可能触发 rehash，所有迭代器失效。

#### 避免失效

1. 使用返回值更新迭代器：

   ```c++
   auto it = vec.erase(it); // 返回下一个有效迭代器
   ```

2. 避免在循环中修改容器容量。

## 12、vector动态扩容是怎么实现的？

## 13、完美转发了解吗？

完美转发是指在函数模板中，将传入的参数以其原始值类别（左值或右值）无损地转发给另一个函数。

目标：

- 保留参数的左值/右值属性。
- 避免不必要的拷贝。

C++11 引入右值引用和转发引用，结合 `std::forward`，实现了完美转发。

#### 值类别

左值：有名字、可取地址的对象（如变量）。

右值：临时对象、无名字（如字面量、函数返回值）。

```c++
int x = 10;    // x 是左值
int&& r = 20;  // 20 是右值，r 是右值引用
```

#### 右值引用（&&）

绑定到右值，用于移动语义。

```c++
void func(int&& r) {}
func(42); // OK
```

#### 转发引用

在模板中，T&&（当 T 是模板参数）是转发引用，可以绑定左值或右值。

规则：

- 若传入左值，T 推导为 T&。
- 若传入右值，T 推导为 T。

```c++
template<typename T>
void func(T&& arg) {} // 转发引用
int x = 10;
func(x);   // T 推导为 int&
func(20);  // T 推导为 int
```

#### `std::forward`

将参数按其原始值类别转发。

## 14、算法：写一个C++类

这里给大家带有二分查找的 C++ 类。

```c++
#include <iostream>
#include <vector>

class BinarySearch {
public:
    // 构造函数：传入有序数据
    BinarySearch(const std::vector<int>& sortedData) : data(sortedData) {}

    // 查找目标元素的下标，未找到返回 -1
    int search(int target) const {
        int left = 0;
        int right = data.size() - 1;
        while (left <= right) {
            // 防止溢出
            int mid = left + (right - left) / 2;
            if (data[mid] == target) {
                return mid;
            } else if (data[mid] < target) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
        return -1; // 未找到
    }

private:
    std::vector<int> data; // 存储有序数据
};

int main() {
    // 构造有序数组
    std::vector<int> sortedArray = {1, 3, 5, 7, 9, 11, 13, 15};

    // 创建 BinarySearch 对象
    BinarySearch bs(sortedArray);

    // 查找目标数字
    int target = 7;
    int index = bs.search(target);

    if (index != -1)
        std::cout << "找到 " << target << "，下标为 " << index << std::endl;
    else
        std::cout << "没有找到 " << target << std::endl;

    return 0;
}
```

