# 未岚大陆秋招C++一面

> 来源：https://www.nowcoder.com/discuss/680756488276553728

### 1、fork和exec是怎么实现的？ 

#### 1. `fork` 系统调用

`fork` 系统调用用于创建一个新的进程，这个新进程被称为子进程，而调用 `fork` 的进程被称为父进程。子进程是父进程的一个副本，但在某些方面有所不同，比如进程 ID（PID）。

##### 实现步骤

1. 分配新的进程资源：
   - 内核为子进程分配新的进程控制块（PCB），包括进程 ID、状态、优先级等。
   - 分配新的内存空间，包括代码段、数据段、堆栈段等。
2. 复制父进程的资源：
   - 复制父进程的 PCB 信息，但更新子进程的 PID。
   - 复制父进程的代码段、数据段和堆栈段。现代操作系统通常使用 **写时复制（Copy-On-Write, COW）** 技术来优化这个过程。初始时，父进程和子进程共享相同的内存页面，只有当其中一个进程尝试写入某一页时，内核才会为该页创建一个副本。
   - 复制父进程的文件描述符表，使子进程继承父进程打开的文件。
3. 设置子进程的状态：
   - 将子进程的状态设置为就绪态，准备被调度执行。
   - 设置子进程的初始指令指针，使其从 `fork` 调用返回后继续执行。
4. 返回值：
   - `fork` 调用在父进程中返回子进程的 PID。
   - `fork` 调用在子进程中返回 0。
   - 如果 `fork` 调用失败，返回 -1，并设置 `errno` 以指示错误原因。

##### 代码示例

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

int main() {
    pid_t pid = fork();

    if (pid < 0) {
        fprintf(stderr, "Fork failed\n");
        return 1;
    } else if (pid == 0) {
        // 子进程
        printf("I am child, my pid is %d\n", getpid());
    } else {
        // 父进程
        printf("I am father, my pid is %d, child pid is %d\n", getpid(), pid);
    }

    return 0;
}
```

#### 2. `exec` 系统调用

`exec` 系列函数用于在当前进程中加载并执行新的程序，用新的程序替换当前进程的内存映像。`exec` 系列函数有多种变体，包括 `execl`、`execv`、`execle`、`execve`、`execlp` 和 `execvp`，它们的主要区别在于参数的传递方式和环境变量的处理。

##### 实现步骤

1. 加载新的程序：
   - 内核释放当前进程的代码段、数据段和堆栈段。
   - 从指定的路径加载新的程序的二进制文件。
   - 将新的程序的代码段、数据段和堆栈段加载到内存中。
2. 设置新的程序的入口点：
   - 将新的程序的入口点（通常是 `main` 函数）设置为当前进程的指令指针。
   - 设置新的程序的参数和环境变量。
3. 返回值：
   - `exec` 系列函数成功调用后不会返回，因为当前进程已经被新的程序替换。
   - 如果 `exec` 调用失败，返回 -1，并设置 `errno` 以指示错误原因。

##### 代码示例

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

int main() {
    pid_t pid = fork();

    if (pid < 0) {
        fprintf(stderr, "Fork failed\n");
        return 1;
    } else if (pid == 0) {
        // 子进程
        char *args[] = {"/bin/ls", "-l", NULL};
        execv("/bin/ls", args);

        // 如果 execv 失败，会返回到这里
        fprintf(stderr, "Exec failed\n");
        return 1;
    } else {
        // 父进程
        printf("I am father, my pid is %d, child pid is %d\n", getpid(), pid);
    }

    return 0;
}
```

### 2、select、poll、epoll的区别？LT和ET的区别？

#### 1. `select` 机制

`select` 是最早的I/O多路复用机制之一，几乎在所有的Unix系统上都有实现。它允许一个进程监视多个文件描述符，等待这些描述符中的任何一个变为可读、可写或发生异常。

##### 主要特点

- 文件描述符限制：`select` 有一个固定的文件描述符限制，通常是1024个。这个限制可以通过修改内核参数来增加，但依然存在。
- 轮询机制：`select` 采用轮询的方式，每次调用 `select` 时，都需要将所有感兴趣的文件描述符集合从用户态复制到内核态，并在内核态中逐个检查这些文件描述符的状态。
- 返回值：`select` 返回后，会修改传入的文件描述符集合，将所有就绪的文件描述符标记出来。应用程序需要遍历这些集合，找到所有就绪的文件描述符。

##### 优点

- 可移植性：`select` 几乎在所有的Unix系统上都有实现，具有很好的可移植性。
- 简单易用：`select` 的使用相对简单，易于理解和实现。

##### 缺点

- 性能问题：随着文件描述符数量的增加，`select` 的性能会显著下降，因为每次调用都需要遍历所有文件描述符。
- 文件描述符限制：`select` 的文件描述符限制较低，不适合处理大量文件描述符的场景。

#### 2. `poll` 机制

`poll` 是对 `select` 的改进，解决了 `select` 的一些局限性，尤其是在文件描述符数量上的限制。

##### 主要特点

- 无文件描述符限制：`poll` 没有文件描述符数量的限制，因为它使用链表来存储文件描述符，可以动态扩展。
- 轮询机制：`poll` 仍然采用轮询的方式，每次调用 `poll` 时，都需要将所有感兴趣的文件描述符集合从用户态复制到内核态，并在内核态中逐个检查这些文件描述符的状态。
- 返回值：`poll` 返回后，会修改传入的 `pollfd` 结构体数组，将所有就绪的文件描述符标记出来。应用程序需要遍历这些结构体，找到所有就绪的文件描述符。

##### 优点

- 无文件描述符限制：`poll` 可以处理更多的文件描述符，适用于大规模并发场景。
- 简单易用：`poll` 的使用相对简单，易于理解和实现。

##### 缺点

- 性能问题：尽管 `poll` 解决了文件描述符数量的限制，但其轮询机制仍然会导致性能下降，特别是当文件描述符数量较多时。

#### 3. `epoll` 机制

`epoll` 是Linux特有的I/O多路复用机制，设计目的是解决 `select` 和 `poll` 在处理大量文件描述符时的性能问题。

##### 主要特点

- 事件驱动：`epoll` 采用事件驱动的方式，只有当文件描述符准备好时，内核才会通知应用程序。这避免了轮询机制的性能开销。
- 无文件描述符限制：`epoll` 没有文件描述符数量的限制，可以处理大量的文件描述符。
- 内存映射：`epoll` 使用内存映射技术，减少了文件描述符在用户态和内核态之间的复制开销。
- 回调机制：`epoll` 为每个文件描述符注册了回调函数，当文件描述符准备好时，内核会调用这些回调函数，将就绪的文件描述符加入就绪队列中。

##### 优点

- 高性能：`epoll` 的事件驱动机制使得它在处理大量文件描述符时性能优越。
- 无文件描述符限制：`epoll` 可以处理大量的文件描述符，适用于高并发场景。
- 内存映射：`epoll` 使用内存映射技术，减少了文件描述符在用户态和内核态之间的复制开销。

##### 缺点

- 复杂性：`epoll` 的使用相对复杂，需要更多的系统调用和数据结构管理。
- 平台限制：`epoll` 是Linux特有的机制，不具有跨平台的特性。

##### 水平触发（LT）

- 触发条件：只要文件描述符处于就绪状态，`epoll_wait` 就会返回。
- 特点：如果应用程序在 `epoll_wait` 返回后没有处理完所有就绪的文件描述符，下次调用 `epoll_wait` 时，这些文件描述符仍然会被返回。
- 适用场景：适用于需要精确控制的场景，例如处理少量数据或需要确保数据完整性的场景。

##### 边缘触发（ET）

- 触发条件：只有当文件描述符从非就绪状态变为就绪状态时，`epoll_wait` 才会返回。
- 特点：如果应用程序在 `epoll_wait` 返回后没有处理完所有就绪的文件描述符，下次调用 `epoll_wait` 时，这些文件描述符不会再次被返回，除非文件描述符的状态再次发生变化。
- 适用场景：适用于高并发、高吞吐量的场景，例如处理大量数据流的场景。ET模式要求应用程序在 `epoll_wait` 返回后立即处理所有就绪的文件描述符，否则可能会遗漏事件。

### 3、如何判断一个任务处理完成？

这里给大家列举两个比较常用的，需要了解更多，可自行学习。

#### 1. 使用 `std::condition_variable`

`std::condition_variable` 是C++标准库中用于线程间同步的工具。它允许一个线程等待某个条件变为真，而另一个线程在满足条件时通知等待的线程。

##### 实现步骤

1. 定义条件变量和互斥量：定义一个 `std::condition_variable` 和一个 `std::mutex`。
2. 设置条件：在任务完成时，设置一个共享的条件变量。
3. 等待条件：在主线程中使用 `std::condition_variable` 等待条件变为真。

##### 代码示例

```c
#include <iostream>
#include <thread>
#include <condition_variable>
#include <mutex>

std::condition_variable cv;
std::mutex mtx;
bool task_completed = false;

void task_with_condition_variable() {
    std::this_thread::sleep_for(std::chrono::seconds(2));
    std::cout << "任务完成" << std::endl;
    {
        std::lock_guard<std::mutex> lock(mtx);
        task_completed = true;
    }
    cv.notify_one();  // 通知等待的线程
}

int main() {
    std::thread t(task_with_condition_variable);

    std::unique_lock<std::mutex> lock(mtx);
    cv.wait(lock, [] { return task_completed; });  // 等待任务完成

    std::cout << "所有任务执行完成" << std::endl;

    t.join();
    return 0;
}
```

#### 2. 使用 `std::atomic` 变量

`std::atomic` 变量提供了原子操作，可以用于在多线程环境中安全地共享数据。通过设置一个 `std::atomic` 变量来表示任务的状态，主线程可以检查这个变量来判断任务是否完成。

##### 实现步骤

1. 定义 `std::atomic` 变量：定义一个 `std::atomic<bool>` 变量来表示任务的状态。
2. 设置任务状态：在任务完成时，设置 `std::atomic` 变量。
3. 检查任务状态：在主线程中检查 `std::atomic` 变量的值。

##### 代码示例

```c
#include <iostream>
#include <thread>
#include <atomic>

std::atomic<bool> task_completed(false);

void task_with_atomic() {
    std::this_thread::sleep_for(std::chrono::seconds(2));
    std::cout << "任务完成" << std::endl;
    task_completed = true;  // 设置任务完成
}

int main() {
    std::thread t(task_with_atomic);

    while (!task_completed.load()) {
        std::this_thread::yield();  // 让出CPU时间片
    }

    std::cout << "所有任务执行完成" << std::endl;

    t.join();
    return 0;
}
```

### 4、进程和线程的区别？

#### 1. 资源分配

- 进程：进程是操作系统中资源分配的基本单位。每个进程都有独立的地址空间、内存空间、文件描述符等资源。进程之间的资源是隔离的，一个进程的资源不会直接影响到另一个进程。
- 线程：线程是CPU调度的基本单位，但线程不拥有系统资源。线程是进程中的一个执行单元，同一进程中的多个线程共享进程的资源，如内存、文件描述符等。线程之间的资源共享使得线程之间的通信更为方便。

#### 2. 地址空间

- 进程：每个进程都有独立的地址空间。这意味着每个进程都有自己的虚拟地址空间，包括代码段、数据段、堆和栈。
- 线程：同一进程中的所有线程共享同一个地址空间。这意味着线程可以访问进程中的任何数据，但不能访问其他进程的数据。

#### 3. 通信

- 进程：进程之间的通信需要通过进程间通信（IPC）机制，如管道、消息队列、共享内存、信号量等。这些机制通常涉及到额外的系统调用和开销。
- 线程：同一进程中的线程可以直接访问共享的内存空间，因此线程之间的通信更为高效。线程可以通过共享变量、互斥锁等机制进行同步和通信。

#### 4. 调度

- 进程：在传统的操作系统中，进程是调度的基本单位。进程的调度涉及到更多的系统开销，因为需要保存和恢复进程的上下文。
- 线程：在引入线程的操作系统中，线程是调度的基本单位。线程的调度开销较小，因为线程只需要保存和恢复少量的寄存器和栈信息。

#### 5. 系统开销

- 进程：创建和销毁进程的开销较大。操作系统需要为每个进程分配和回收资源，如内存、文件描述符等。进程切换时也需要保存和恢复大量的上下文信息。
- 线程：创建和销毁线程的开销较小。操作系统只需要为线程分配少量的资源，如栈空间和寄存器。线程切换时的开销也较小，因为只需要保存和恢复少量的寄存器和栈信息。

#### 6. 并发性

- 进程：进程之间的并发性较低，因为进程之间的资源是隔离的，通信开销较大。
- 线程：线程之间的并发性较高，因为线程共享进程的资源，通信开销较小。同一进程中的多个线程可以并发执行，提高系统的吞吐量。

#### 7. 稳定性和安全性

- 进程：多进程程序更稳定，因为进程之间的资源是隔离的。一个进程的崩溃不会影响其他进程。
- 线程：多线程程序的稳定性较差，因为线程共享资源，一个线程的错误可能导致整个进程崩溃。

### 5、线程独有的资源是什么？

线程作为进程中的一个执行单元，虽然共享了进程的大部分资源，但也拥有自己独有的资源。这些资源确保了线程能够独立地执行任务，同时与其他线程保持一定的隔离。

#### 1. 线程ID

每个线程都有一个唯一的标识符，称为线程ID。线程ID用于区分同一个进程中的不同线程，确保操作系统能够正确地管理和调度这些线程。

#### 2. 栈区

每个线程都有自己的栈区。栈区用于存储函数调用时的局部变量、函数参数、返回地址等信息。栈区是线程私有的，确保了每个线程在执行函数时不会干扰其他线程的局部变量。

#### 3. 程序计数器（PC）

程序计数器（Program Counter，PC）是一个寄存器，用于指示线程当前执行的指令位置。当线程被调度时，操作系统会保存当前线程的PC值，以便在下次调度时恢复执行。每个线程都有自己的PC，确保了线程可以独立地执行代码。

#### 4. 寄存器组

线程在执行时会使用一系列寄存器来存储中间计算结果、地址等信息。这些寄存器在不同的线程之间是独立的，确保了每个线程的计算不会受到其他线程的影响。当线程被切换时，操作系统会保存当前线程的寄存器状态，以便在下次调度时恢复。

#### 5. 错误返回码

线程在执行系统调用或其他操作时，可能会遇到错误并设置错误返回码。由于同一个进程中可能有多个线程同时运行，一个线程设置的错误返回码可能会被另一个线程覆盖。因此，每个线程都有自己的错误返回码变量，确保了错误处理的准确性。

#### 6. 线程优先级

线程优先级决定了线程在调度时的优先顺序。虽然优先级高的线程并不一定会先执行，但优先级高的线程有更高的机会被调度。每个线程都有自己的优先级，确保了线程调度的灵活性。

#### 7. 线程局部存储（TLS）

线程局部存储（Thread Local Storage，TLS）是一种特殊的存储机制，允许每个线程拥有自己的全局变量副本。这些变量在所有线程中看起来是全局的，但实际上每个线程都有自己的独立副本，不会互相干扰。TLS在多线程编程中非常有用，可以避免线程之间的数据竞争。

### 6、读写锁什么时候上锁？

读写锁（Read-Write Lock）是一种用于控制对共享资源访问的同步机制，特别适用于读操作远多于写操作的场景。读写锁允许多个读线程同时访问共享资源，但只允许一个写线程访问资源，从而提高程序的并发性能。读写锁在特定条件下上锁，以确保数据的一致性和线程的安全性。

#### 1. 读锁的上锁条件

- 没有写锁被持有：当没有其他线程持有写锁时，读线程可以获取读锁。多个读线程可以同时持有读锁，允许多个读线程并发访问共享资源。
- 没有写线程在等待写锁：在某些实现中，如果已经有写线程在等待写锁，新的读线程可能会被阻塞，以防止写饥饿（即写线程长时间无法获取锁）。

#### 2. 写锁的上锁条件

- 没有读锁被持有：当没有其他线程持有读锁时，写线程可以获取写锁。写锁是独占的，确保在写操作期间没有其他线程（无论是读线程还是写线程）访问共享资源。
- 没有写锁被其他线程持有：写锁也是互斥的，确保在任何时候只有一个写线程可以持有写锁。

#### 3. 上锁的具体时机

- 读操作：
  - 在读操作开始前，读线程会尝试获取读锁。如果当前没有写锁被持有，读线程可以成功获取读锁并继续执行读操作。
  - 如果当前有写锁被持有或有写线程在等待写锁，读线程将被阻塞，直到写锁被释放且没有写线程在等待写锁。
  - 读操作完成后，读线程会释放读锁，允许其他读线程或写线程获取锁。
- 写操作：
  - 在写操作开始前，写线程会尝试获取写锁。如果当前没有读锁或写锁被持有，写线程可以成功获取写锁并继续执行写操作。
  - 如果当前有读锁或写锁被持有，写线程将被阻塞，直到所有读锁和前一个写锁被释放。
  - 写操作完成后，写线程会释放写锁，允许其他读线程或写线程获取锁。

#### 4. 非阻塞上锁

- 尝试获取读锁：`pthread_rwlock_tryrdlock`函数尝试获取读锁，如果当前没有写锁被持有，读线程可以成功获取读锁并继续执行读操作。如果当前有写锁被持有或有写线程在等待写锁，函数会立即返回`EBUSY`，表示获取锁失败。
- 尝试获取写锁：`pthread_rwlock_trywrlock` 函数尝试获取写锁，如果当前没有读锁或写锁被持有，写线程可以成功获取写锁并继续执行写操作。如果当前有读锁或写锁被持有，函数会立即返回 `EBUSY`，表示获取锁失败。

### 7、什么是死锁？只有两个线程时候，会发生死锁吗？

#### 什么是死锁？

死锁是指在多线程或多进程环境中，两个或多个进程或线程因为争夺资源而陷入的一种互相等待的状态，导致这些进程或线程都无法继续执行。在这种状态下，每个进程或线程都在等待其他进程或线程释放它所持有的资源，而自身又不释放已持有的资源，形成了一种永久的阻塞状态。

#### 死锁的四个必要条件

死锁的发生**必须同时满足**以下四个必要条件：

1. 互斥条件：一个资源一次只能被一个进程或线程占用，其他进程或线程必须等待当前占用者释放资源。
2. 请求与保持条件：一个进程或线程已经保持了至少一个资源，但又提出了新的资源请求，而该资源已被其他进程或线程占有，此时请求者被阻塞，但对自己已获得的资源保持不放。
3. 不可剥夺条件：进程或线程所获得的资源在未使用完毕之前，不能被其他进程或线程强行剥夺，只能由获得该资源的进程或线程自行释放。
4. 循环等待条件：存在一个进程或线程的循环等待链，即每个进程或线程都在等待下一个进程或线程所占有的资源，形成一个闭环。

#### 两个线程时会发生死锁吗？

即使只有两个线程，也有可能发生死锁。

给个例子：

```c
#include <iostream>
#include <thread>
#include <mutex>
#include <chrono>

class DeadlockExample {
private:
    std::mutex lock1;
    std::mutex lock2;

public:
    void method1() {
        std::lock_guard<std::mutex> guard1(lock1);
        std::cout << "Thread 1: Holding lock 1..." << std::endl;
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
        
        // 这里会导致死锁
        std::lock_guard<std::mutex> guard2(lock2);
        std::cout << "Thread 1: Holding lock 1 & 2..." << std::endl;
    }

    void method2() {
        std::lock_guard<std::mutex> guard2(lock2);
        std::cout << "Thread 2: Holding lock 2..." << std::endl;
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
        
        // 这里会导致死锁
        std::lock_guard<std::mutex> guard1(lock1);
        std::cout << "Thread 2: Holding lock 2 & 1..." << std::endl;
    }

    void run() {
        std::thread t1(&DeadlockExample::method1, this);
        std::thread t2(&DeadlockExample::method2, this);

        t1.join();
        t2.join();
    }
};

int main() {
    DeadlockExample example;
    example.run();
    return 0;
}
```

#### 死锁分析

- 线程1 (`method1`) 先获取 `lock1`，然后尝试获取 `lock2`。
- 线程2 (`method2`) 先获取 `lock2`，然后尝试获取 `lock1`。

当 线程1 持有 `lock1` 并等待 `lock2` 时，线程2 持有 `lock2` 并等待 `lock1`。这时，两个线程都进入了等待状态，形成了死锁。

#### 如何避免死锁？

避免死锁的方法主要有以下几种：

1. 破坏互斥条件：尽量减少资源的互斥使用，例如通过资源共享或资源复制。
2. 破坏请求与保持条件：要求进程或线程在请求新的资源之前，必须释放已持有的所有资源。
3. 破坏不可剥夺条件：允许系统在必要时剥夺进程或线程的资源。
4. 破坏循环等待条件：确保资源的请求顺序是固定的，避免形成循环等待。

### 8、栈和堆的区别？

#### 1. 管理方式

- 栈：
  - 自动管理：栈的内存分配和释放由操作系统自动完成，无需程序员手动干预。
  - 局部作用域：栈主要用于存储函数调用时的局部变量、函数参数和返回地址等。
  - 生命周期：栈中的数据在其所在的函数调用结束后自动被销毁。
- 堆：
  - 手动管理：堆的内存分配和释放需要程序员手动控制，使用 `malloc`、`free`（C语言）或 `new`、`delete`（C++语言）等操作。
  - 全局作用域：堆主要用于存储动态分配的内存，如对象、数组等。
  - 生命周期：堆中的数据在程序结束前一直存在，除非程序员显式释放。

#### 2. 空间大小

- 栈：栈的大小通常较小，具体大小取决于操作系统和编译器。例如，64位Windows默认栈大小为1MB，64位Linux默认栈大小为10MB。
- 堆：堆的大小通常较大，理论上可以达到虚拟内存的大小，具体大小取决于系统的可用内存。

#### 3. 生长方向

- 栈：栈的内存地址从高地址向低地址生长。
- 堆：堆的内存地址从低地址向高地址生长。

#### 4. 分配方式

- 栈：
  - 静态分配：栈支持静态分配，例如函数中的局部变量。
  - 动态分配：栈也支持动态分配，例如使用 `alloca` 函数，但这些动态分配的内存仍然由操作系统管理，不需要手动释放。
- 堆：
  - 动态分配：堆只支持动态分配，例如使用 `malloc`、`new` 等函数分配内存，需要手动释放。

#### 5. 分配效率

- 栈：栈的分配和释放非常高效，因为操作系统提供了专门的寄存器和指令来管理栈。
- 堆：堆的分配和释放效率较低，因为需要进行复杂的内存管理操作，容易产生内存碎片。

#### 6. 存放内容

- 栈：
  - 局部变量：函数调用时的局部变量、函数参数、返回地址等。
  - 栈帧：每个函数调用都会在栈上创建一个栈帧，用于存储该函数的局部变量和参数。
- 堆：
  - 动态数据：动态分配的内存，如对象、数组等。
  - 全局数据：全局变量和静态变量通常不存放在堆中，而是存放在数据段或BSS段。

#### 7. 使用场景

- 栈：
  - 函数调用：栈主要用于函数调用时的参数传递、局部变量存储等。
  - 递归：递归函数的调用栈也是通过栈来实现的。
- 堆：
  - 动态内存分配：堆主要用于需要动态分配内存的场景，如对象的创建、大型数据结构的存储等。
  - 优先队列：堆数据结构（如二叉堆）常用于实现优先队列。

### 9、讲一下智能指针，各自的区别？

#### 1. `std::auto_ptr`（已废弃）

- 所有权模型：`std::auto_ptr` 采用独占所有权模型，即同一时间只有一个 `std::auto_ptr` 可以拥有某个对象。
- 复制和赋值：复制和赋值操作会转移所有权，原 `std::auto_ptr` 将不再拥有对象。
- 局限性：
  - 不安全：复制和赋值操作会转移所有权，容易导致悬挂指针。
  - 不支持数组：`std::auto_ptr` 不支持管理数组，使用 `delete` 而不是 `delete[]` 释放内存。
  - 已废弃：C++11已将其废弃，建议使用 `std::unique_ptr`。

#### 2. `std::unique_ptr`

- 所有权模型：`std::unique_ptr` 也采用独占所有权模型，同一时间只有一个 `std::unique_ptr` 可以拥有某个对象。
- 复制和赋值：`std::unique_ptr` 禁止复制和赋值操作，但支持移动语义。可以通过 `std::move` 将所有权从一个 `std::unique_ptr` 转移到另一个。
- 安全性：`std::unique_ptr` 更安全，避免了 `std::auto_ptr` 的所有权转移问题。
- 支持数组：`std::unique_ptr` 支持管理数组，可以使用 `std::unique_ptr<int[]>` 来管理数组。
- 性能：`std::unique_ptr` 的性能较高，因为它的实现非常轻量级，只包含一个指针。

#### 3. `std::shared_ptr`

- 所有权模型：`std::shared_ptr` 采用共享所有权模型，多个 `std::shared_ptr` 可以同时拥有同一个对象。
- 引用计数：`std::shared_ptr` 使用引用计数来管理对象的生命周期。每当有一个新的 `std::shared_ptr` 指向同一个对象时，引用计数加1；当一个 `std::shared_ptr` 被销毁或重置时，引用计数减1。当引用计数为0时，对象被自动删除。
- 线程安全：引用计数的操作是线程安全的，但对象的访问不是线程安全的。
- 性能：由于需要维护引用计数，`std::shared_ptr` 的性能相对较低，但仍然非常有用，特别是在需要共享资源的场景中。

#### 4. `std::weak_ptr`

- 所有权模型：`std::weak_ptr` 是一个弱引用智能指针，它不增加对象的引用计数。
- 用途：`std::weak_ptr` 主要用于解决 `std::shared_ptr` 之间的循环引用问题。`std::weak_ptr` 可以观察一个对象是否还存在，但不能直接访问对象。
- 使用方法：要访问 `std::weak_ptr` 管理的对象，需要先调用 `lock` 方法，返回一个 `std::shared_ptr`。如果对象已经不存在，`lock` 方法将返回一个空的 `std::shared_ptr`。
- 性能：`std::weak_ptr` 的性能较低，因为它需要检查对象是否仍然存在。

### 10、弱引用计数的来源？

在 `std::shared_ptr` 中，除了维护一个引用计数来记录共享所有权的数量外，还有一个弱引用计数（weak reference count）。弱引用计数用于管理 `std::weak_ptr` 的生命周期。

#### 引用计数（Shared Count）

- 作用：记录当前有多少个 `std::shared_ptr` 指向同一个对象。
- 增减：
  - 当创建一个新的 `std::shared_ptr` 指向同一个对象时，引用计数加1。
  - 当一个 `std::shared_ptr` 被销毁或重置时，引用计数减1。
- 对象销毁：当引用计数为0时，表示没有 `std::shared_ptr` 指向该对象，对象会被自动删除。

#### 弱引用计数（Weak Count）

- 作用：记录当前有多少个 `std::weak_ptr` 指向同一个对象。
- 增减：
  - 当创建一个新的 `std::weak_ptr` 指向同一个对象时，弱引用计数加1。
  - 当一个 `std::weak_ptr` 被销毁或重置时，弱引用计数减1。
- 控制块销毁：当引用计数和弱引用计数都为0时，表示没有 `std::shared_ptr` 和 `std::weak_ptr` 指向该对象，控制块（control block）会被销毁。

#### 控制块（Control Block）

`std::shared_ptr` 和 `std::weak_ptr` 都依赖于一个控制块来管理引用计数和弱引用计数。控制块是一个内部数据结构，通常包含以下信息：

- 指向对象的指针：实际管理的对象的指针。
- 引用计数：记录 `std::shared_ptr` 的数量。
- 弱引用计数：记录 `std::weak_ptr` 的数量。

#### 弱引用计数的作用

1. 管理 `std::weak_ptr` 的生命周期：
   - 当所有 `std::shared_ptr` 都被销毁后，对象会被删除，但 `std::weak_ptr` 仍然可能存在。
   - 弱引用计数确保在最后一个 `std::weak_ptr` 被销毁之前，控制块不会被销毁，从而允许 `std::weak_ptr` 通过 `lock` 方法检查对象是否仍然存在。
2. 避免控制块过早销毁：
   - 如果没有弱引用计数，当最后一个 `std::shared_ptr` 被销毁时，控制块会被立即销毁，即使还有 `std::weak_ptr` 存在。
   - 弱引用计数确保控制块在最后一个 `std::weak_ptr` 被销毁之前保持有效。

#### 示例代码

```c
#include <iostream>
#include <memory>

void weak_ptr_example() {
    std::shared_ptr<int> sharedPtr(new int(42));
    std::weak_ptr<int> weakPtr = sharedPtr;

    // 引用计数为1，弱引用计数为1
    std::cout << "sharedPtr use count: " << sharedPtr.use_count() << std::endl;
    std::cout << "weakPtr use count: " << weakPtr.use_count() << std::endl;

    // 创建一个新的 shared_ptr，引用计数加1
    std::shared_ptr<int> sharedPtr2 = sharedPtr;
    std::cout << "sharedPtr use count: " << sharedPtr.use_count() << std::endl;
    std::cout << "weakPtr use count: " << weakPtr.use_count() << std::endl;

    // 销毁 sharedPtr，引用计数减1
    sharedPtr.reset();
    std::cout << "sharedPtr2 use count: " << sharedPtr2.use_count() << std::endl;
    std::cout << "weakPtr use count: " << weakPtr.use_count() << std::endl;

    // 销毁 sharedPtr2，引用计数为0，对象被删除
    sharedPtr2.reset();
    std::cout << "sharedPtr2 use count: " << sharedPtr2.use_count() << std::endl;
    std::cout << "weakPtr use count: " << weakPtr.use_count() << std::endl;

    // 尝试访问对象
    std::shared_ptr<int> lockedPtr = weakPtr.lock();
    if (lockedPtr) {
        std::cout << "*lockedPtr = " << *lockedPtr << std::endl;
    } else {
        std::cout << "Object has been deleted." << std::endl;
    }

    // 销毁 weakPtr，弱引用计数为0，控制块被销毁
    weakPtr.reset();
}

int main() {
    weak_ptr_example();
    return 0;
}
```

#### 输出

```c
sharedPtr use count: 1
weakPtr use count: 1
sharedPtr use count: 2
weakPtr use count: 1
sharedPtr2 use count: 1
weakPtr use count: 1
sharedPtr2 use count: 0
weakPtr use count: 1
Object has been deleted.
```

### 11、了解make_shared吗？weak_ptr会导致线程不安全吗？

#### 1. `std::make_shared`

`std::make_shared` 是 C++11 引入的一个模板函数，用于更高效地创建 `std::shared_ptr` 实例。它通过一次内存分配同时创建对象和控制块，从而减少了内存分配的次数，提高了性能。

##### 优点

- 单一内存分配：`std::make_shared` 在一次内存分配中同时创建对象和控制块，减少了内存碎片，提高了内存使用效率。
- 性能：相比直接使用 `new` 创建对象并通过构造函数传递给 `std::shared_ptr`，`std::make_shared` 更高效。
- 异常安全：`std::make_shared` 在内存分配失败时会抛出异常，确保不会留下未初始化的指针。

##### 语法

```c
template <class T, class... Args>
shared_ptr<T> make_shared(Args&&... args);
```

##### 示例代码

```c
#include <iostream>
#include <memory>

class MyClass {
public:
    MyClass(int value) : value(value) {
        std::cout << "MyClass constructor called with value: " << value << std::endl;
    }
    ~MyClass() {
        std::cout << "MyClass destructor called" << std::endl;
    }
    void printValue() const {
        std::cout << "Value: " << value << std::endl;
    }
private:
    int value;
};

int main() {
    std::shared_ptr<MyClass> ptr = std::make_shared<MyClass>(42);
    ptr->printValue();
    return 0;
}
```

#### 2. `std::weak_ptr` 的线程安全性

`std::weak_ptr` 本身并不直接管理对象的生命周期，而是作为 `std::shared_ptr` 的一个弱引用。`std::weak_ptr` 的主要用途是避免循环引用和实现观察者模式。

在多线程环境中，`std::weak_ptr` 的使用需要注意以下几点：

##### 引用计数

- 引用计数：`std::shared_ptr` 的引用计数操作是线程安全的，这意味着多个线程可以同时读取同一个 `std::shared_ptr` 而不会出现问题。
- 弱引用计数：`std::weak_ptr` 的弱引用计数也是线程安全的，多个线程可以同时读取同一个 `std::weak_ptr` 而不会出现问题。

##### `lock` 方法

- **`lock` 方法**：`std::weak_ptr` 的 `lock` 方法用于将 `std::weak_ptr` 转换为 `std::shared_ptr`。这个操作是线程安全的，但需要注意的是，`lock` 方法返回的 `std::shared_ptr` 必须在同一个线程中使用，以避免数据竞争。

### 12、讲一些stl中的容器和各自特点？

今天再仔细给大家总结一遍。

大致分为三类：**序列容器**、**关联容器**和**容器适配器**。

#### 1. 序列容器（Sequence Containers）

序列容器是按顺序存储元素的容器，支持随机访问或顺序访问。

##### 1.1 `std::vector`

- 特点：
  - 动态数组：底层实现为动态数组，存储空间连续。
  - 随机访问：支持随机访问，访问元素的时间复杂度为 O(1)。
  - 插入/删除：在尾部插入/删除元素的时间复杂度为 O(1)，在中间或头部插入/删除元素的时间复杂度为 O(n)。
  - 空间效率：会预先分配一些额外空间，以减少重新分配的次数。
- 适用场景：
  - 需要随机访问且频繁在尾部进行操作的场景。
  - 如果频繁在中间或头部插入/删除元素，则不适用该容器。

##### 1.2 `std::deque`

- 特点：
  - 双端队列：底层实现为分段连续的存储块，支持在首部和尾部高效插入/删除元素。
  - 随机访问：支持随机访问，访问元素的时间复杂度为 O(1)。
  - 插入/删除：在首部和尾部插入/删除元素的时间复杂度为 O(1)，在中间插入/删除元素的时间复杂度为 O(n)。
  - 空间效率：会预先分配一些额外空间，以减少重新分配的次数。
- 适用场景：
  - 需要随机访问且频繁在首部和尾部进行操作的场景。
  - 如果频繁在中间插入/删除元素，则不适用该容器。

##### 1.3 `std::list`

- 特点：
  - 双向链表：底层实现为双向链表，存储空间不连续。
  - 顺序访问：不支持随机访问，只能通过迭代器进行访问。
  - 插入/删除：在任意位置插入/删除元素的时间复杂度为 O(1)。
  - 空间效率：每个元素都需要分配额外的空间来存储前后指针。
- 适用场景：
  - 需要在任意位置频繁插入/删除操作的场景。

##### 1.4 `std::forward_list`

- 特点：
  - 单向链表：底层实现为单向链表，存储空间不连续。
  - 顺序访问：不支持随机访问，只能通过迭代器进行访问。
  - 插入/删除：在任意位置插入/删除元素的时间复杂度为 O(1)。
  - 空间效率：每个元素只需要分配额外的空间来存储后继指针。
- 适用场景：
  - 需要在任意位置频繁插入/删除操作，且对内存使用有较高要求的场景。

##### 1.5 `std::array`

- 特点：
  - 固定大小数组：底层实现为固定大小的数组，存储空间连续。
  - 随机访问：支持随机访问，访问元素的时间复杂度为 O(1)。
  - 插入/删除：不支持动态插入/删除元素。
  - 空间效率：固定大小，没有额外的空间开销。
- 适用场景：
  - 需要固定大小数组且支持随机访问的场景。

#### 2. 关联容器（Associative Containers）

关联容器是按特定顺序存储元素的容器，通常用于快速查找。

##### 2.1 `std::set`

- 特点：
  - 有序集合：底层实现为红黑树，存储空间不连续。
  - 唯一性：每个元素都是唯一的，不允许重复。
  - 插入/删除/查找：插入、删除和查找元素的时间复杂度为 O(log n)。
- 适用场景：
  - 需要有序集合且元素不重复的场景。

##### 2.2 `std::multiset`

- 特点：
  - 有序集合：底层实现为红黑树，存储空间不连续。
  - 允许多重性：允许元素重复。
  - 插入/删除/查找：插入、删除和查找元素的时间复杂度为 O(log n)。
- 适用场景：
  - 需要有序集合且元素允许重复的场景。

##### 2.3 `std::map`

- 特点：
  - 有序映射：底层实现为红黑树，存储空间不连续。
  - 键值对：存储键值对，键是唯一的，不允许重复。
  - 插入/删除/查找：插入、删除和查找元素的时间复杂度为 O(log n)。
- 适用场景：
  - 需要有序键值对且键不重复的场景。

##### 2.4 `std::multimap`

- 特点：
  - 有序映射：底层实现为红黑树，存储空间不连续。
  - 键值对：存储键值对，键允许重复。
  - 插入/删除/查找：插入、删除和查找元素的时间复杂度为 O(log n)。
- 适用场景：
  - 需要有序键值对且键允许重复的场景。

##### 2.5 `std::unordered_set`

- 特点：
  - 无序集合：底层实现为哈希表，存储空间不连续。
  - 唯一性：每个元素都是唯一的，不允许重复。
  - 插入/删除/查找：插入、删除和查找元素的平均时间复杂度为 O(1)。
- 适用场景：
  - 需要无序集合且元素不重复的场景。

##### 2.6 `std::unordered_multiset`

- 特点：
  - 无序集合：底层实现为哈希表，存储空间不连续。
  - 允许多重性：允许元素重复。
  - 插入/删除/查找：插入、删除和查找元素的平均时间复杂度为 O(1)。
- 适用场景：
  - 需要无序集合且元素允许重复的场景。

##### 2.7 `std::unordered_map`

- 特点：
  - 无序映射：底层实现为哈希表，存储空间不连续。
  - 键值对：存储键值对，键是唯一的，不允许重复。
  - 插入/删除/查找：插入、删除和查找元素的平均时间复杂度为 O(1)。
- 适用场景：
  - 需要无序键值对且键不重复的场景。

##### 2.8 `std::unordered_multimap`

- 特点：
  - 无序映射：底层实现为哈希表，存储空间不连续。
  - 键值对：存储键值对，键允许重复。
  - 插入/删除/查找：插入、删除和查找元素的平均时间复杂度为 O(1)。
- 适用场景：
  - 需要无序键值对且键允许重复的场景。

#### 3. 容器适配器（Container Adapters）

容器适配器是基于其他容器实现的，提供了一些特定的功能。

##### 3.1 `std::stack`

- 特点：
  - 后进先出（LIFO）：基于 `std::deque` 或其他容器实现，只允许在一端进行插入和删除操作。
  - 操作：`push`、`pop`、`top`。
- 适用场景：
  - 需要后进先出数据结构的场景。

##### 3.2 `std::queue`

- 特点：
  - 先进先出（FIFO）：基于 `std::deque` 或其他容器实现，允许在一端插入，在另一端删除。
  - 操作：`push`、`pop`、`front`、`back`。
- 适用场景：
  - 需要先进先出数据结构的场景。

##### 3.3 `std::priority_queue`

- 特点：
  - 优先级队列：基于 `std::vector` 或其他容器实现，元素按优先级排序，最高优先级的元素始终在队列前端。
  - 操作：`push`、`pop`、`top`。
- 适用场景：
  - 需要按优先级处理元素的场景。

### 13、讲一下deque的底层实现？

见上

### 14、红黑树的特点？哈希表的特点？如何解决哈希冲突？

#### 红黑树的特点

红黑树（Red-Black Tree）是一种自平衡的二叉查找树，通过控制节点的颜色和路径的黑高来保证查找、插入、删除的效率。

1. 节点颜色：每个节点要么是红色，要么是黑色。
2. 根节点：根节点总是黑色的。
3. 叶子节点：每个叶子节点（即空节点）都是黑色的。
4. 红色节点的子节点：两个红色节点不能相邻，即红色节点的子节点必须是黑色的。
5. 黑色节点的路径：从任意节点到其每个叶子节点的所有路径都包含相同数量的黑色节点。
6. 平衡性：红黑树的黑色节点路径长度相同，保证了树的高度为 O(log n)，从而保证了查找、插入和删除操作的时间复杂度为 O(log n)。
7. 应用场景：红黑树适用于需要有序性和高效查找、插入和删除操作的场景，例如数据库索引、符号表等。

#### 哈希表的特点

哈希表（Hash Table）是一种数据结构，通过哈希函数将键值映射到表中的一个位置来访问记录，以加快查找的速度。

1. 快速访问：通过哈希函数将键值转换为数组索引，可以在 O(1) 时间内完成查找、插入和删除操作。
2. 空间换时间：哈希表需要预先分配一定的内存空间，但通过牺牲空间来换取时间上的高效。
3. 无序性：哈希表中的元素是无序的，因为它们是根据哈希函数的结果存储的。
4. 哈希冲突：由于哈希函数的输出范围有限，不同的键值可能会映射到同一个位置，这种现象称为哈希冲突。解决哈希冲突是哈希表设计中的一个重要问题。
5. 应用场景：哈希表适用于需要高速查找、插入和删除操作的场景，例如缓存、集合、映射等。

#### 解决哈希冲突的方法

哈希冲突是哈希表使用中不可避免的问题，但通过合理的解决方法，可以有效地管理和缓解冲突的影响。

1. 链地址法（Separate Chaining）：
   - 将所有哈希值相同的元素存储在一个链表中。每个哈希桶都连接一个链表，当发生冲突时，新元素被添加到相应的链表中。
   - 优点：实现简单，不会出现堆积现象，非同义词不会发生冲突。
   - 缺点：需要额外的空间来存储链表指针。
2. 开放地址法（Open Addressing）：
   - 当发生冲突时，通过某种探查方法在哈希表中寻找下一个空闲位置来存储元素。常见的探查方法包括：
     - 线性探测（Linear Probing）：按顺序查找下一个空闲位置。
     - 二次探测（Quadratic Probing）：按二次多项式查找下一个空闲位置。
     - 双重哈希（Double Hashing）：使用第二个哈希函数来确定步长。
   - 优点：不需要额外的空间来存储链表指针。
   - 缺点：容易出现堆积现象，导致性能下降。
3. 再哈希法（Rehashing）：
   - 当发生冲突时，使用第二个哈希函数重新计算哈希值，直到找到一个不冲突的位置。
   - 优点：减少冲突的概率。
   - 缺点：增加了计算时间。
4. 建立公共溢出区：
   - 将哈希表分为基本表和溢出表，将发生冲突的元素存储在溢出表中。
   - 优点：可以灵活处理冲突。
   - 缺点：需要额外的空间来存储溢出表。

### 15、讲一下右值引用？

右值引用是 C++11 引入的一个重要特性，旨在优化临时对象的处理，避免不必要的深拷贝操作，从而提高程序的性能。右值引用通过引入新的引用类型 `T&&`，允许程序员直接操作临时对象，实现了移动语义（Move Semantics）和完美转发（Perfect Forwarding）。

#### 1. 左值与右值

在理解右值引用之前，需要先了解左值（Lvalue）和右值（Rvalue）的概念。

- 左值（Lvalue）：

  - 指的是有明确存储位置的对象，可以取地址，通常表示持久存在的变量。
  - 特点：可以出现在赋值操作符的左边，可以通过 `&` 取地址。

  ```c
  int x = 5;  // x 是左值
  int& ref = x;  // ref 是左值引用
  ```

- 右值（Rvalue）：

  - 指的是临时对象或字面量，不能取地址，通常表示临时的、短暂存在的值。
  - 特点：只能出现在赋值操作符的右边，不能通过 `&` 取地址。

  ```c
  int y = 10;  // 10 是右值
  int z = x + 5;  // x + 5 是右值
  ```

#### 2. 左值引用与右值引用

- 左值引用（Lvalue Reference）：使用 `T&` 表示，只能绑定到左值。

  ```c
  int x = 5;
  int& ref = x;  // 合法
  int& ref2 = 10;  // 非法，10 是右值
  ```

- 右值引用（Rvalue Reference）：使用 `T&&` 表示，只能绑定到右值。

  ```c
  int x = 5;
  int&& rref = 10;  // 合法
  int&& rref2 = x;  // 非法，x 是左值
  ```

#### 3. 移动语义（Move Semantics）

移动语义允许将资源从一个对象“移动”到另一个对象，而不是进行深拷贝。这在处理临时对象时特别有用，可以显著提高性能。

- 移动构造函数（Move Constructor）：使用右值引用来接收资源，将其转移到当前对象中。

  ```c
  class MyClass {
      int* data;
  public:
      // 构造函数
      MyClass(int size) : data(new int[size]) {}
  
      // 移动构造函数
      MyClass(MyClass&& other) noexcept : data(other.data) {
          other.data = nullptr;  // 将资源转移给新对象
      }
  
      // 析构函数
      ~MyClass() {
          delete[] data;
      }
  };
  ```

- 移动赋值运算符（Move Assignment Operator）：当一个对象被赋值时，检测是否可以使用右值引用来移动资源，而不是拷贝资源。

  ```c
  class MyClass {
      int* data;
  public:
      // 构造函数
      MyClass(int size) : data(new int[size]) {}
  
      // 移动构造函数
      MyClass(MyClass&& other) noexcept : data(other.data) {
          other.data = nullptr;
      }
  
      // 移动赋值运算符
      MyClass& operator=(MyClass&& other) noexcept {
          if (this != &other) {
              delete[] data;  // 释放当前资源
              data = other.data;  // 将资源转移给当前对象
              other.data = nullptr;
          }
          return *this;
      }
  
      // 析构函数
      ~MyClass() {
          delete[] data;
      }
  };
  ```

#### 4. `std::move` 与 `std::forward`

- `std::move`：将左值强制转换为右值引用，以便调用移动构造函数或移动赋值运算符。

  ```c
  MyClass obj1(10);
  MyClass obj2 = std::move(obj1);  // 调用移动构造函数
  ```

- `std::forward`：用于完美转发，保持参数的原始类型和值类别。

  ```c
  template<typename T>
  void wrapper(T&& arg) {
      process(std::forward<T>(arg));  // 保持参数的原始类型和值类别
  }
  ```

#### 5. 完美转发（Perfect Forwarding）

完美转发允许模板函数在转发参数时保持参数的原始类型和值类别，这对于实现移动语义和精确的函数封装非常重要。

```c
#include <iostream>
#include <utility>

void process(int& x) {
    std::cout << "process(int&) - Lvalue reference" << std::endl;
}

void process(int&& x) {
    std::cout << "process(int&&) - Rvalue reference" << std::endl;
}

template<typename T>
void wrapper(T&& arg) {
    process(std::forward<T>(arg));  // 保持参数的原始类型和值类别
}

int main() {
    int a = 10;
    wrapper(a);  // 调用 process(int&)
    wrapper(20); // 调用 process(int&&)
    return 0;
}
```

### 16、什么时候使用移动语义？

见上。

### 17、讲一下多态？怎么调用虚函数的？

#### 1. 编译时多态（静态多态）

编译时多态是指在编译阶段就确定了函数或操作的具体实现。最典型的例子是函数重载（Function Overloading）和模板（Templates）。

##### 1.1 函数重载（Function Overloading）

函数重载是指在同一作用域内，可以有多个同名函数，但它们的参数列表必须不同（参数个数、类型或顺序不同）。

```c
#include <iostream>

// 函数重载
void display(int value) {
    std::cout << "Display int: " << value << std::endl;
}

void display(double value) {
    std::cout << "Display double: " << value << std::endl;
}

int main() {
    display(10);    // 调用 void display(int)
    display(10.5);  // 调用 void display(double)
    return 0;
}
```

##### 1.2 模板（Templates）

模板允许编写泛型代码，可以在编译时生成特定类型的代码。模板可以应用于函数和类。

```c
#include <iostream>

// 函数模板
template<typename T>
void display(T value) {
    std::cout << "Display: " << value << std::endl;
}

int main() {
    display(10);    // 实例化为 void display<int>(int)
    display(10.5);  // 实例化为 void display<double>(double)
    return 0;
}
```

#### 2. 运行时多态（动态多态）

运行时多态是指在运行时根据对象的实际类型来决定调用哪个方法。在 C++ 中，运行时多态主要通过虚函数（Virtual Functions）和继承（Inheritance）来实现。

##### 2.1 虚函数（Virtual Functions）

虚函数是在基类中使用 `virtual` 关键字声明的成员函数，允许子类重写这些函数。通过基类的指针或引用调用虚函数时，实际调用的是派生类中重写的方法。

```c
#include <iostream>

class Base {
public:
    virtual void display() {
        std::cout << "Display from Base" << std::endl;
    }
};

class Derived : public Base {
public:
    void display() override {
        std::cout << "Display from Derived" << std::endl;
    }
};

void callDisplay(Base& obj) {
    obj.display();
}

int main() {
    Base b;
    Derived d;

    callDisplay(b);  // 调用 Base::display()
    callDisplay(d);  // 调用 Derived::display()

    Base* ptr = new Derived();
    ptr->display();  // 调用 Derived::display()

    delete ptr;
    return 0;
}
```

##### 2.2 纯虚函数与抽象类

纯虚函数是没有具体实现的虚函数，形式为 `virtual void func() = 0;`。包含纯虚函数的类称为抽象类，不能实例化，只能被继承。

```c
#include <iostream>

class AbstractBase {
public:
    virtual void display() = 0;  // 纯虚函数
};

class ConcreteDerived : public AbstractBase {
public:
    void display() override {
        std::cout << "Display from ConcreteDerived" << std::endl;
    }
};

int main() {
    // AbstractBase ab;  // 错误：不能实例化抽象类
    ConcreteDerived cd;
    cd.display();  // 调用 ConcreteDerived::display()

    AbstractBase* ptr = new ConcreteDerived();
    ptr->display();  // 调用 ConcreteDerived::display()

    delete ptr;
    return 0;
}
```

#### 3. 调用虚函数的过程

调用虚函数的过程涉及虚函数表（Virtual Table, VTable）和虚函数指针（Virtual Table Pointer, VPtr）。

##### 3.1 虚函数表（VTable）

- 定义：每个包含虚函数的类都会生成一个虚函数表，其中存储着该类中所有虚函数的地址。
- 作用：在运行时，通过虚函数表查找并调用正确的虚函数。

##### 3.2 虚函数指针（VPtr）

- 定义：在对象的内存布局中，编译器会添加一个额外的指针，称为虚函数指针，指向该对象对应的虚函数表。
- 作用：通过虚函数指针，程序能够在运行时确定调用哪个虚函数。

#### 3.3 调用过程

1. 编译阶段：
   - 编译器确定对象的类型，并生成相应的虚函数表。
   - 对象的内存布局中包含一个虚函数指针，指向其对应的虚函数表。
2. 运行阶段：
   - 通过基类指针或引用调用虚函数时，编译器会使用虚函数指针查找虚函数表。
   - 根据虚函数表中的地址，调用相应的虚函数。

### 18、手撕，128. 最长连续序列

#### 思路

1. 构建哈希表：将数组中的所有元素插入到一个 `unordered_set` 中，以便我们可以在 O(1) 时间内检查某个元素是否存在。
2. 查找连续序列：
   - 遍历数组中的每个元素，检查该元素是否是一个连续序列的起点。一个元素是连续序列的起点当且仅当它的前一个元素不在哈希表中。
   - 如果当前元素是连续序列的起点，我们从该元素开始，逐个检查后续的连续元素，并计算连续序列的长度。
   - 更新最长连续序列的长度。

#### 参考代码

```c
#include <vector>
#include <unordered_set>
#include <algorithm>
using namespace std;

class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        if (nums.empty()) return 0;

        unordered_set<int> num_set(nums.begin(), nums.end());
        int longestStreak = 0;

        for (int num : num_set) {
            // 检查当前元素是否是连续序列的起点
            if (num_set.find(num - 1) == num_set.end()) {
                int currentNum = num;
                int currentStreak = 1;

                // 从当前元素开始，逐个检查后续的连续元素
                while (num_set.find(currentNum + 1) != num_set.end()) {
                    currentNum += 1;
                    currentStreak += 1;
                }

                // 更新最长连续序列的长度
                longestStreak = max(longestStreak, currentStreak);
            }
        }

        return longestStreak;
    }
};
```

