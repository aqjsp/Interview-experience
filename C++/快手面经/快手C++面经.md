> https://www.nowcoder.com/discuss/569974504026378240

# 快手秋招C++面经，

## 一面

### C++：

#### 1、派生类继承基类时、虚函数表内的函数是何时替换的？

在C++中，当派生类继承基类时，如果基类中有虚函数，派生类可以选择是否覆盖（override）这些虚函数。如果派生类覆盖了基类的虚函数，那么在派生类对象上调用该虚函数时，将会调用派生类中的实现，而不是基类中的实现。这种动态绑定的机制是通过虚函数表（vtable）来实现的。

虚函数表是用来实现动态多态的一种机制，每个包含虚函数的类都有一个对应的虚函数表。在编译阶段，编译器会为每个包含虚函数的类生成一个虚函数表，表中存储了指向各个虚函数实现的指针。当一个类被实例化时，会在对象的内存布局中分配一个指向其虚函数表的指针（通常被称为虚表指针或vptr），这样就能在运行时实现动态绑定。

当派生类继承基类时，如果派生类覆盖了基类的虚函数，编译器会在派生类的虚函数表中将该虚函数的指针指向派生类中的实现。这样，当通过基类指针或引用调用虚函数时，实际上会根据对象的实际类型在运行时查找对应的虚函数表，并调用正确的虚函数实现。

#### 2、指向派生类的基类指针、强转为 `void*` 再转为基类指针、此时调用虚函数会发生什么（正常）？

1. 转换为 `void*`：当将指向派生类的基类指针强制转换为 `void*` 类型时，指针的类型信息会丢失，但指针仍然指向原来的对象。
2. 再转换回基类指针：当将 `void*` 类型的指针转换回基类指针时，编译器会进行一次静态类型转换。这意味着编译器会假定这个指针是指向基类对象的，而不考虑它原本指向派生类对象。
3. 调用虚函数：如果这个基类中的虚函数在派生类中被覆盖（override），那么在运行时，由于编译器在转换回基类指针时假定这个指针指向的是基类对象，因此调用虚函数时会根据基类的虚函数表来确定调用哪个函数。这意味着即使实际上这个指针指向的是派生类对象，也会调用基类中的虚函数实现，而不是派生类中的实现。

### 算法：

#### 1、反转链表 `[l, r]` 区间内的所有节点、返回新链表的头节点

思路：

1. 先创建一个哑节点 `dummy`，并将其指向原链表的头节点，这样可以处理头节点可能被反转的情况。
2. 使用 `prev` 指针找到左边界的前一个节点，然后使用 `cur` 指针记录当前需要反转的节点。
3. 开始从左边界到右边界进行反转，每次将 `cur` 节点的 `next` 指针指向下一个节点的 `next`，然后将下一个节点的 `next` 指向反转后的链表头部，最后将 `prev` 的 `next` 指针指向下一个节点，完成一次反转。
4. 返回哑节点的 `next`，即为新链表的头节点。

参考代码：

```
#include <iostream>

struct ListNode {
    int val;
    ListNode *next;
    ListNode(int x) : val(x), next(nullptr) {}
};

ListNode* reverseBetween(ListNode* head, int left, int right) {
    if (left == right) return head;  // 如果 left 等于 right，无需反转，直接返回头节点

    // 创建一个哑节点作为新链表的头节点，并将其指向原链表的头节点
    ListNode* dummy = new ListNode(0);
    dummy->next = head;

    // 使用 prev 指针来记录左边界的前一个节点
    ListNode* prev = dummy;
    for (int i = 0; i < left - 1; ++i) {
        prev = prev->next;
    }

    // 使用 cur 指针来记录当前需要反转的节点
    ListNode* cur = prev->next;

    // 从左边界开始反转到右边界
    for (int i = 0; i < right - left; ++i) {
        ListNode* next = cur->next;  // 先记录下一个节点的指针
        cur->next = next->next;      // 将当前节点的 next 指针指向下一个节点的 next
        next->next = prev->next;     // 将下一个节点的 next 指针指向反转后的链表头部
        prev->next = next;           // 将 prev 的 next 指针指向下一个节点，完成一次反转
    }

    return dummy->next;  // 返回新链表的头节点
}

void printList(ListNode* head) {
    while (head) {
        std::cout << head->val << " ";
        head = head->next;
    }
    std::cout << std::endl;
}

int main() {
    ListNode* head = new ListNode(1);
    head->next = new ListNode(2);
    head->next->next = new ListNode(3);
    head->next->next->next = new ListNode(4);
    head->next->next->next->next = new ListNode(5);

    std::cout << "Original List: ";
    printList(head);

    int left = 2, right = 4;
    head = reverseBetween(head, left, right);

    std::cout << "Reversed List from " << left << " to " << right << ": ";
    printList(head);

    return 0;
}
```

## 二面

### C++ & Webserver：

#### 1、线程池实现步骤？

这里就讲讲正常的一个线程池的实现步骤。

1. 定义任务类：首先需要定义一个任务类，用于封装需要在线程池中执行的任务。任务类至少应该包含一个执行任务的方法，可以是一个函数指针或者是一个函数对象。

   ```
   class Task {
   public:
       virtual void execute() = 0;
   };
   ```

2. 定义线程池类：接下来定义线程池类，其中包含了线程池的管理逻辑，如线程的创建、销毁、任务的添加等。线程池类需要包含一个线程池容器，用于存放线程对象。

   ```
   #include <vector>
   #include <thread>
   #include <queue>
   #include <mutex>
   #include <condition_variable>
   
   class ThreadPool {
   public:
       ThreadPool(size_t numThreads);
       ~ThreadPool();
   
       void addTask(Task* task);
   
   private:
       std::vector<std::thread> workers;  // 线程池中的线程
       std::queue<Task*> tasks;            // 任务队列
       std::mutex queueMutex;              // 保护任务队列的互斥量
       std::condition_variable condition;  // 用于线程间通信的条件变量
       bool stop;                          // 标志线程池是否停止的标志位
   };
   ```

3. 实现线程池类的构造函数和析构函数：在构造函数中创建指定数量的线程，并启动这些线程；在析构函数中停止线程池中的所有线程。

   ```
   ThreadPool::ThreadPool(size_t numThreads) : stop(false) {
       for (size_t i = 0; i < numThreads; ++i) {
           workers.emplace_back([this] {
               while (true) {
                   Task* task = nullptr;
                   {
                       std::unique_lock<std::mutex> lock(queueMutex);
                       condition.wait(lock, [this] { return stop || !tasks.empty(); });
                       if (stop && tasks.empty()) return;
                       task = tasks.front();
                       tasks.pop();
                   }
                   task->execute();
                   delete task;
               }
           });
       }
   }
   
   ThreadPool::~ThreadPool() {
       {
           std::unique_lock<std::mutex> lock(queueMutex);
           stop = true;
       }
       condition.notify_all();
       for (std::thread& worker : workers) {
           worker.join();
       }
   }
   ```

4. 实现添加任务的方法：在线程池类中添加一个方法用于向任务队列中添加任务。

   ```
   void ThreadPool::addTask(Task* task) {
       {
           std::unique_lock<std::mutex> lock(queueMutex);
           tasks.push(task);
       }
       condition.notify_one();
   }
   ```

5. 使用线程池：最后，在主程序中使用定义好的线程池类来执行任务。

   ```
   int main() {
       ThreadPool pool(4);  // 创建一个包含4个线程的线程池
   
       // 添加任务到线程池
       for (int i = 0; i < 8; ++i) {
           pool.addTask(new YourTask());  // YourTask 是需要执行的任务类
       }
   
       // ...
   
       return 0;
   }
   ```

#### 2、存放线程执行任务的结构体或者类型是什么？（`std::function`）

在一个线程池中，通常需要一个结构体或者类型来表示线程执行的任务。这个结构体或者类型需要包含执行任务的信息，比如任务的具体内容、状态等。在C++中，可以使用函数指针、`std::function` 或者自定义的函数对象来表示任务。

来个例子：

```
#include <functional>

// 使用 std::function 来表示任务
struct Task {
    std::function<void()> function;

    // 构造函数
    Task(const std::function<void()>& f) : function(f) {}

    // 执行任务的方法
    void execute() {
        if (function) {
            function();
        }
    }
};
```

#### 3、线程 A 如何向线程 B 发起异步请求并获取到处理结果、接口是什么？

在 C++ 中，线程 A 可以向线程 B 发起异步请求并获取处理结果的一种常见方式是使用 `std::future` 和 `std::promise`。这种方法允许线程 A 发起异步任务，并在需要时等待线程 B 完成任务并获取结果。

使用 `std::future` 和 `std::promise` 实现线程 A 向线程 B 发起异步请求并获取处理结果的简单示例：

```
#include <iostream>
#include <future>
#include <thread>

void asyncTask(std::promise<int>& promiseObj) {
    // 模拟一个耗时的异步任务
    std::this_thread::sleep_for(std::chrono::seconds(2));

    // 设置 promise 的值，表示任务完成
    promiseObj.set_value(42);
}

int main() {
    // 创建一个 promise 对象和一个 future 对象
    std::promise<int> promiseObj;
    std::future<int> futureObj = promiseObj.get_future();

    // 在另一个线程中执行异步任务
    std::thread worker(asyncTask, std::ref(promiseObj));
    worker.detach();  // 让 worker 线程在后台运行

    // 在主线程中等待异步任务的结果
    std::cout << "Waiting for result..." << std::endl;
    int result = futureObj.get();  // 阻塞等待任务完成并获取结果
    std::cout << "Result: " << result << std::endl;

    return 0;
}
```

示例中，线程 A（主线程）创建了一个 `std::promise` 对象 `promiseObj` 和一个与之关联的 `std::future` 对象 `futureObj`。然后，线程 A 启动了一个新的线程（线程 B），并将 `promiseObj` 作为参数传递给异步任务函数 `asyncTask`。异步任务函数中通过 `promiseObj.set_value()` 设置了异步任务的结果。

在主线程中，通过 `futureObj.get()` 方法阻塞等待异步任务的完成，并获取到任务的结果。这样，线程 A 就能够向线程 B 发起异步请求并获取处理结果了。

#### 4、介绍一下智能指针？

1. `std::unique_ptr`：
   - `std::unique_ptr` 用于管理独占所有权的对象，即同一时间只能有一个 `std::unique_ptr` 指向一个对象。
   - 当 `std::unique_ptr` 被销毁时，它所指向的对象也会被销毁，这样可以确保资源的正确释放。
   - `std::unique_ptr` 不支持拷贝和赋值操作，但可以通过 `std::move` 来转移所有权。
   - 适合用于管理局部对象或者作为容器元素的指针。
2. `std::shared_ptr`：
   - `std::shared_ptr` 用于管理共享所有权的对象，即多个 `std::shared_ptr` 可以指向同一个对象。
   - 内部通过引用计数来管理资源的生命周期，当最后一个指向对象的 `std::shared_ptr` 被销毁时，对象会被释放。
   - 支持拷贝和赋值操作，内部使用引用计数来追踪对象的引用情况。
   - 适合用于多个对象共享同一资源的情况，比如多个对象共享同一个动态分配的对象。
3. `std::weak_ptr`：
   - `std::weak_ptr` 是 `std::shared_ptr` 的一种辅助工具，用于解决 `std::shared_ptr` 的循环引用问题。
   - `std::weak_ptr` 本身不增加引用计数，它只是观察 `std::shared_ptr` 的引用计数，并提供了一种机制来检测对象是否已经被释放。
   - 可以通过 `std::weak_ptr` 的 `lock` 方法获取一个指向对象的 `std::shared_ptr`，如果对象已经被释放，则返回一个空的 `std::shared_ptr`。
   - 适合用于解决 `std::shared_ptr` 循环引用导致的内存泄漏问题。
4. `std::auto_ptr`（C++11 之前）：
   - `std::auto_ptr` 用于管理动态分配的对象，在 C++11 中已被废弃，不推荐使用。
   - `std::auto_ptr` 具有独占所有权，不支持拷贝构造和拷贝赋值操作，但支持移动语义。
   - 在 C++11 中被 `std::unique_ptr` 替代，因为 `std::unique_ptr` 具有更好的语义和性能。

#### 5、了解哪些设计模式（单例、工厂、建造者）

这里就简单说一下题主给的这几个设计模式吧。

##### 单例模式：

主要用于确保一个类只有一个实例，并提供一个全局访问点来访问该实例。

1. **饿汉式单例模式**（线程不安全）：

   - 在类的静态成员变量中直接创建实例，并在类的静态方法中返回该实例。
   - 这种方式在程序启动时就会创建单例对象，无论是否需要使用，可能会导致资源浪费。
   - 不适合在多线程环境下使用，因为没有进行线程安全的处理。

   ```
   class Singleton {
   public:
       static Singleton& getInstance() {
           static Singleton instance;
           return instance;
       }
   
       // 防止拷贝构造和赋值操作
       Singleton(const Singleton&) = delete;
       Singleton& operator=(const Singleton&) = delete;
   
   private:
       Singleton() {}  // 私有化构造函数，禁止外部创建实例
   };
   ```

2. **懒汉式单例模式**（线程不安全）：

   - 在第一次调用时才创建单例对象，避免了在程序启动时就创建对象的资源浪费。
   - 不适合在多线程环境下使用，因为没有进行线程安全的处理。

   ```
   class Singleton {
   public:
       static Singleton& getInstance() {
           static Singleton instance;
           return instance;
       }
   
       // 防止拷贝构造和赋值操作
       Singleton(const Singleton&) = delete;
       Singleton& operator=(const Singleton&) = delete;
   
   private:
       Singleton() {}  // 私有化构造函数，禁止外部创建实例
   };
   ```

3. 懒汉式单例模式（线程安全）：

   - 使用加锁的方式保证在多线程环境下也能正常工作，但会影响性能。
   - 在 `getInstance` 方法中加锁，避免了多个线程同时创建实例的问题。

   ```
   #include <mutex>
   
   class Singleton {
   public:
       static Singleton& getInstance() {
           std::lock_guard<std::mutex> lock(mutex);
           static Singleton instance;
           return instance;
       }
   
       // 防止拷贝构造和赋值操作
       Singleton(const Singleton&) = delete;
       Singleton& operator=(const Singleton&) = delete;
   
   private:
       Singleton() {}  // 私有化构造函数，禁止外部创建实例
       static std::mutex mutex;
   };
   
   std::mutex Singleton::mutex;
   ```

4. Meyers' Singleton（线程安全）：（不常用）

   - 利用 C++11 的特性，在静态变量的初始化阶段进行初始化，保证了线程安全性。
   - 使用静态局部变量的特性，在第一次调用 `getInstance` 方法时才进行实例化。

   ```
   class Singleton {
   public:
       static Singleton& getInstance() {
           static Singleton instance;
           return instance;
       }
   
       // 防止拷贝构造和赋值操作
       Singleton(const Singleton&) = delete;
       Singleton& operator=(const Singleton&) = delete;
   
   private:
       Singleton() {}  // 私有化构造函数，禁止外部创建实例
       ~Singleton() {}
   };
   ```

   



#### 6、泛型编程使用经验

### 项目：

#### 1、为什么要使用 cgroup 进行绑核、有没有了解过其他方案（namespace）

#### 2、C++ 中 CPU 绑核的 API 是什么（`sched_setaffinity`）

#### 3、如果车端 CPU 开销超过原有的 11 个核怎么办

### 场景题：

#### 1、按行和按列遍历二维数组在性能上的差异

#### 2、死循环中执行 `i++` 自增操作后睡眠 1ms、CPU 占用是怎么样的

### 算法：

#### 1、手撕 shared_ptr

#### 2、平衡二叉树（No. 110）