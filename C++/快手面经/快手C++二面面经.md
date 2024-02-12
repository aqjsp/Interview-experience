# 快手C++二面面经

> 来源：https://www.nowcoder.com/feed/main/detail/bd803b3753f343bf9750b7201414d0c9

## 八股

### 1、new malloc的区别 至少说出4点以上，在申请内存的时候都做了哪些工作 申请内存的过程是否需要初始化？

#### 区别

1. 语法：
   - `new` 是 C++ 中的关键字，用于动态分配内存，并调用构造函数初始化对象。
   - `malloc` 是 C 语言中的函数，用于动态分配内存，返回 `void*` 类型的指针。在 C++ 中也可以使用，但是需要强制类型转换。
2. 类型安全性：
   - `new` 是类型安全的，它会在分配内存时考虑到对象的类型，并调用相应的构造函数初始化对象。
   - `malloc` 返回的是 `void*` 类型的指针，需要手动进行类型转换，容易出现错误。
3. 内存分配失败处理：
   - `new` 在内存分配失败时会抛出 `std::bad_alloc` 异常。
   - `malloc` 在内存分配失败时会返回 `NULL`，需要检查返回值来判断是否分配成功。
4. 内存释放：
   - `new` 分配的内存可以使用 `delete` 关键字来释放，同时会调用对象的析构函数。
   - `malloc` 分配的内存需要使用 `free` 函数来释放，不会调用对象的析构函数。
5. 大小计算：
   - `new` 可以根据对象类型自动计算所需的内存大小。
   - `malloc` 需要手动计算所需的内存大小，并传递给函数。

#### 申请内存

1. 内存分配请求： 当程序需要动态分配内存时，通过调用相应的内存分配函数（如 `new`、`malloc`、`calloc` 等）向操作系统请求内存。
2. 内存管理器查找可用内存块： 内存管理器会在内存池中查找一个合适大小的可用内存块。
3. 分配内存： 如果找到合适的内存块，则将该内存块标记为已分配，并返回一个指向该内存块的指针。
4. 初始化内存（可选）： 在 C++ 中，使用 `new` 运算符分配的内存会调用对象的构造函数进行初始化；在 C 中，使用 `malloc` 或 `calloc` 分配的内存不会被初始化，其内容是未定义的。
5. 返回指针： 将指向分配内存的指针返回给程序，程序可以使用该指针来访问和操作分配的内存。

在 C++ 中，使用 `new` 运算符分配内存时，会调用对象的构造函数进行初始化，这意味着分配的内存已经被初始化。而在 C 中，使用 `malloc` 或 `calloc` 分配内存时，内存不会被初始化，其内容是未定义的。

### 2、delete 和 delete [] 区别 如何对调使用会发生什么事情？

1. 释放方式：
   - `delete` 用于释放使用 `new` 分配的单个对象的内存。
   - `delete[]` 用于释放使用 `new[]` 分配的数组对象的内存。
2. 释放过程：
   - `delete` 会调用被释放对象的析构函数，然后释放对象占用的内存。
   - `delete[]` 会先调用数组中每个元素的析构函数（如果有），然后释放整个数组占用的内存。
3. 对调用的影响：
   - 如果使用 `delete[]` 释放使用 `new` 分配的单个对象的内存，行为是未定义的，可能会导致程序崩溃或其他意外情况。
   - 如果使用 `delete` 释放使用 `new[]` 分配的数组对象的内存，行为也是未定义的，可能会导致程序崩溃或其他意外情况。

### 3、动态多态的虚函数内部原理， 子类继承父类在动态多态中会调用谁的虚方法？

动态多态是指在运行时根据对象的实际类型来调用相应的函数，而不是根据变量或指针的静态类型来调用函数。

1. 虚函数的内部原理：
   - 虚函数是通过虚函数表（vtable）来实现的。每个类有一个虚函数表，其中存储着该类的虚函数的地址。
   - 当一个类中包含虚函数时，编译器会在该类的对象中添加一个指向虚函数表的指针，称为虚函数指针（vptr）。
   - 在调用虚函数时，实际上是通过对象的虚函数指针找到对应的虚函数表，然后根据函数在虚函数表中的偏移量找到具体的函数地址进行调用。
2. 子类继承父类的虚方法：
   - 当子类继承父类并重写（override）父类的虚函数时，子类会覆盖父类的虚函数，即子类的虚函数表中相应位置的函数指针会指向子类的虚函数。
   - 在动态多态中，如果通过父类的指针或引用调用虚函数，实际上会根据对象的实际类型找到对应的虚函数表，并调用该表中对应位置的函数。
   - 因此，无论是通过父类指针还是子类指针调用虚函数，都会调用对象的实际类型中的虚函数。

### 4、多线程在C++中保证线程安全的方式有哪些？

1. 互斥锁（Mutex）： 使用互斥锁可以保护临界区（一段代码或操作需要保证同一时刻只能由一个线程执行的区域），确保在同一时刻只有一个线程可以访问共享资源。常见的互斥锁包括 `std::mutex`、`std::recursive_mutex` 等。
2. 条件变量（Condition Variable）： 条件变量用于线程间的通信，允许一个线程等待另一个线程满足特定的条件。常与互斥锁配合使用，例如 `std::condition_variable`。
3. 原子操作（Atomic Operation）： 原子操作可以确保某个操作在多线程环境下是原子性的，不会被中断。C++ 提供了一系列的原子操作类型，如 `std::atomic`，用于操作原子类型的数据。
4. 读写锁（Read-Write Lock）： 读写锁允许多个线程同时读取共享资源，但只有一个线程能够写入共享资源。这可以提高多线程读取操作的并发性能。C++11 提供了 `std::shared_mutex`。
5. 线程局部存储（Thread-Local Storage）： 每个线程有自己独立的存储空间，可以避免多个线程访问同一全局变量而导致的竞态条件。可以使用 `thread_local` 关键字定义线程局部变量。
6. 使用锁的 RAII 封装： 使用 RAII（资源获取即初始化）技术可以确保在作用域结束时自动释放锁，避免忘记释放锁而导致的死锁等问题。可以使用 `std::lock_guard`、`std::unique_lock` 等封装互斥锁。

### 5、多线程只读操作的时候需要加锁吗？

在多线程环境中，只有读操作（不涉及修改共享数据）的情况下，通常情况下不需要加锁。这是因为多个线程同时读取共享数据不会导致数据的破坏或不一致性，因此不需要互斥访问。

但是，在某些情况下，即使是只读操作也需要考虑加锁的情况：

1. 确保数据一致性： 如果多个线程同时读取数据，但其中某些线程的操作依赖于其他线程的操作结果，为了保证数据的一致性，可能需要在读取数据时加锁。
2. 避免数据竞争： 即使是只读操作，如果多个线程同时访问同一块内存区域，并且其中某些线程可能会对该内存区域进行写操作，为了避免数据竞争，可能也需要加锁。
3. 提高性能： 在某些情况下，即使是只读操作，加锁也可以避免由于多个线程同时访问同一数据结构而导致的性能下降。例如，读写锁（Read-Write Lock）允许多个线程同时读取共享数据，但在有写操作时会阻塞其他读操作，从而提高了读操作的并发性能。

### 6、多个线程读 一个线程写需要加锁吗？

在多个线程同时读取数据，且只有一个线程写入数据的情况下，需要考虑加锁以保证数据的一致性和避免数据竞争。

如果不加锁，可能会出现以下问题：

1. 数据竞争： 多个线程同时读取数据，如果写入线程同时修改数据，可能会导致数据不一致或损坏。
2. 脏读取（Dirty Read）： 读取线程读取到了正在被写入线程修改的数据，导致读取的数据不正确。
3. 不可重复读取（Non-repeatable Read）： 读取线程多次读取同一数据，但在读取过程中数据被写入线程修改，导致每次读取的结果不一致。
4. 幻读（Phantom Read）： 读取线程读取了一组数据，但在读取过程中写入线程插入了新的数据，导致读取结果不一致。

为了避免这些问题，可以使用读写锁（Read-Write Lock）来实现。读写锁允许多个线程同时读取数据，但只允许一个线程写入数据。当有写入操作时，读取操作会被阻塞，确保写入操作的原子性和一致性。

另一种方法是使用互斥锁（Mutex）来保护数据的读写操作。在读取数据时获取共享（读）锁，在写入数据时获取独占（写）锁，确保在同一时刻只有一个线程可以写入数据，从而避免数据竞争和不一致性。

### 7、读写锁如何实现口述？

读写锁是一种用于控制对共享资源的访问的同步机制，允许多个线程同时读取共享资源，但只允许一个线程写入共享资源。读写锁分为读锁和写锁两种类型，线程在访问共享资源之前需要获取相应的锁。

读写锁的实现可以基于互斥锁和条件变量来完成。

给个伪代码示例：

```
// 读写锁结构体
struct ReadWriteLock {
    pthread_mutex_t mutex;         // 用于保护读写锁的互斥锁
    pthread_cond_t readCondition;  // 读取条件变量
    pthread_cond_t writeCondition; // 写入条件变量
    int readers;                   // 当前正在读取的线程数
    int writers;                   // 当前正在写入的线程数
    int pendingWriters;            // 等待写入的线程数
};

// 初始化读写锁
void init_rwlock(ReadWriteLock* rwlock) {
    pthread_mutex_init(&rwlock->mutex, NULL);
    pthread_cond_init(&rwlock->readCondition, NULL);
    pthread_cond_init(&rwlock->writeCondition, NULL);
    rwlock->readers = 0;
    rwlock->writers = 0;
    rwlock->pendingWriters = 0;
}

// 加读锁
void read_lock(ReadWriteLock* rwlock) {
    pthread_mutex_lock(&rwlock->mutex);
    while (rwlock->writers > 0 || rwlock->pendingWriters > 0) {
        pthread_cond_wait(&rwlock->readCondition, &rwlock->mutex);
    }
    rwlock->readers++;
    pthread_mutex_unlock(&rwlock->mutex);
}

// 释放读锁
void read_unlock(ReadWriteLock* rwlock) {
    pthread_mutex_lock(&rwlock->mutex);
    rwlock->readers--;
    if (rwlock->readers == 0 && rwlock->pendingWriters > 0) {
        pthread_cond_signal(&rwlock->writeCondition);
    }
    pthread_mutex_unlock(&rwlock->mutex);
}

// 加写锁
void write_lock(ReadWriteLock* rwlock) {
    pthread_mutex_lock(&rwlock->mutex);
    rwlock->pendingWriters++;
    while (rwlock->readers > 0 || rwlock->writers > 0) {
        pthread_cond_wait(&rwlock->writeCondition, &rwlock->mutex);
    }
    rwlock->pendingWriters--;
    rwlock->writers++;
    pthread_mutex_unlock(&rwlock->mutex);
}

// 释放写锁
void write_unlock(ReadWriteLock* rwlock) {
    pthread_mutex_lock(&rwlock->mutex);
    rwlock->writers--;
    if (rwlock->pendingWriters > 0) {
        pthread_cond_signal(&rwlock->writeCondition)
    } else {
        pthread_cond_broadcast(&rwlock->readCondition);
    }
    pthread_mutex_unlock(&rwlock->mutex);
}
```

### 8、8大排序方法的时间复杂度？ 口述归并排序和快排

| 排序算法 | 平均时间复杂度 | 最好情况  | 最坏情况  | 空间复杂度 | 稳定性 |
| -------- | -------------- | --------- | --------- | ---------- | ------ |
| 冒泡排序 | O(n^2)         | O(n)      | O(n^2)    | O(1)       | 稳定   |
| 快速排序 | O(nlogn)       | O(nlogn)  | O(n^2)    | O(logn)    | 不稳定 |
| 选择排序 | O(n^2)         | O(n^2)    | O(n^2)    | O(1)       | 不稳定 |
| 插入排序 | O(n^2)         | O(n)      | O(n^2)    | O(1)       | 稳定   |
| 希尔排序 | O(nlogn)       | O(log^2n) | O(log^2n) | O(1)       | 不稳定 |
| 归并排序 | O(nlogn)       | O(nlogn)  | O(nlogn)  | O(n)       | 稳定   |
| 堆排序   | O(nlogn)       | O(nlogn)  | O(nlogn)  | O(1)       | 不稳定 |
| 计数排序 | O(n+k)         | O(n+k)    | O(n+k)    | O(k)       | 稳定   |
| 桶排序   | O(n+k)         | O(n+k)    | O(n^2)    | O(n+k)     | 稳定   |
| 基数排序 | O(nxk)         | O(nxk)    | O(nxk)    | O(n+k)     | 稳定   |

#### 归并排序

采用分治法（Divide and Conquer）策略。它的基本思想是将待排序的序列分成两部分，分别对这两部分进行排序，然后将两个已排序的部分合并成一个有序序列。

下面是归并排序的详细步骤：

1. 分解（Divide）： 将待排序的序列分成两个长度相等（或相差最多 1）的子序列，找到序列的中间位置。
2. 解决（Conquer）： 递归地对左右两个子序列进行归并排序，直到子序列的长度为 1 或 0。
3. 合并（Merge）： 将两个已排序的子序列合并成一个有序序列。合并过程需要额外的空间来存储临时序列。

归并排序是稳定的排序算法，时间复杂度为 O(n log n)，空间复杂度为 O(n)，其中 n 是待排序序列的长度。

```
#include <iostream>
#include <vector>
using namespace std;

// 合并两个有序数组
void merge(vector<int>& arr, int left, int mid, int right) {
    int n1 = mid - left + 1;
    int n2 = right - mid;

    // 创建临时数组来存储左右两部分的元素
    vector<int> L(n1), R(n2);

    // 将元素复制到临时数组 L 和 R 中
    for (int i = 0; i < n1; i++) {
        L[i] = arr[left + i];
    }
    for (int j = 0; j < n2; j++) {
        R[j] = arr[mid + 1 + j];
    }

    // 合并临时数组到 arr[left..right]
    int i = 0, j = 0, k = left;
    while (i < n1 && j < n2) {
        if (L[i] <= R[j]) {
            arr[k] = L[i];
            i++;
        } else {
            arr[k] = R[j];
            j++;
        }
        k++;
    }

    // 将剩余的元素复制到 arr 中
    while (i < n1) {
        arr[k] = L[i];
        i++;
        k++;
    }
    while (j < n2) {
        arr[k] = R[j];
        j++;
        k++;
    }
}

// 归并排序主函数
void mergeSort(vector<int>& arr, int left, int right) {
    if (left < right) {
        // 找到中间点
        int mid = left + (right - left) / 2;

        // 递归地对左右两部分进行排序
        mergeSort(arr, left, mid);
        mergeSort(arr, mid + 1, right);

        // 合并两个有序数组
        merge(arr, left, mid, right);
    }
}

int main() {
    // 输入
    vector<int> arr = {12, 11, 13, 5, 6, 7};

    // 输出排序前的数组
    cout << "排序前的数组：" << endl;
    for (int num : arr) {
        cout << num << " ";
    }
    cout << endl;

    // 归并排序
    mergeSort(arr, 0, arr.size() - 1);

    // 输出排序后的数组
    cout << "排序后的数组：" << endl;
    for (int num : arr) {
        cout << num << " ";
    }
    cout << endl;

    return 0;
}
```

#### 快排

基本思想是通过一趟排序将待排序的数据分割成独立的两部分，其中一部分的所有数据都比另一部分的所有数据都要小，然后再按此方法对这两部分数据分别进行快速排序，整个排序过程可以递归进行，以此达到整个数据变成有序序列。

以下是快速排序的详细步骤及示例代码：

1. 选择一个基准元素（pivot），通常选择数组的第一个元素。
2. 设定两个指针，左边一个指向数组的起始位置，右边一个指向数组的末尾。
3. 移动左指针直到找到一个比基准元素大的元素，移动右指针直到找到一个比基准元素小的元素，然后交换这两个元素，重复这个过程，直到左指针超过了右指针。
4. 将基准元素与右指针指向的元素交换位置，这样基准元素就位于右指针的位置，这个位置左边的元素都比基准元素小，右边的元素都比基准元素大。
5. 对基准元素左边的子数组和右边的子数组分别进行快速排序，直到整个数组有序。

##### 递归

```
#include <iostream>
#include <vector>
using namespace std;

// 分区函数，将数组分为比基准元素小和大的两部分
int partition(vector<int>& arr, int low, int high) {
    int pivot = arr[low]; // 选取第一个元素作为基准元素
    int left = low, right = high;
    while (left < right) {
        // 从右向左找到第一个小于基准元素的位置
        while (left < right && arr[right] >= pivot) {
            right--;
        }
        arr[left] = arr[right]; // 将找到的小于基准元素的值放到左指针位置
        // 从左向右找到第一个大于基准元素的位置
        while (left < right && arr[left] <= pivot) {
            left++;
        }
        arr[right] = arr[left]; // 将找到的大于基准元素的值放到右指针位置
    }
    arr[left] = pivot; // 将基准元素放到最终位置
    return left;
}

// 快速排序函数
void quickSort(vector<int>& arr, int low, int high) {
    if (low < high) {
        int pivotPos = partition(arr, low, high); // 分区
        quickSort(arr, low, pivotPos - 1); // 对左子数组排序
        quickSort(arr, pivotPos + 1, high); // 对右子数组排序
    }
}

int main() {
    vector<int> arr = {12, 11, 13, 5, 6, 7};
    cout << "排序前的数组：" << endl;
    for (int num : arr) {
        cout << num << " ";
    }
    cout << endl;

    quickSort(arr, 0, arr.size() - 1);

    cout << "排序后的数组：" << endl;
    for (int num : arr) {
        cout << num << " ";
    }
    cout << endl;

    return 0;
}
```

##### 非递归

```
#include <iostream>
#include <vector>
#include <stack>
using namespace std;

// 分区函数，将数组分为比基准元素小和大的两部分
int partition(vector<int>& arr, int low, int high) {
    int pivot = arr[low]; // 选取第一个元素作为基准元素
    int left = low, right = high;
    while (left < right) {
        // 从右向左找到第一个小于基准元素的位置
        while (left < right && arr[right] >= pivot) {
            right--;
        }
        arr[left] = arr[right]; // 将找到的小于基准元素的值放到左指针位置
        // 从左向右找到第一个大于基准元素的位置
        while (left < right && arr[left] <= pivot) {
            left++;
        }
        arr[right] = arr[left]; // 将找到的大于基准元素的值放到右指针位置
    }
    arr[left] = pivot; // 将基准元素放到最终位置
    return left;
}

// 非递归快速排序函数
void quickSort(vector<int>& arr, int low, int high) {
    stack<pair<int, int>> stk; // 用于存储待排序子数组的边界
    stk.push(make_pair(low, high)); // 将整个数组的边界压入栈中

    while (!stk.empty()) {
        pair<int, int> curr = stk.top();
        stk.pop();
        int l = curr.first, r = curr.second;
        if (l < r) {
            int pivotPos = partition(arr, l, r); // 分区
            stk.push(make_pair(l, pivotPos - 1)); // 左半部分压栈
            stk.push(make_pair(pivotPos + 1, r)); // 右半部分压栈
        }
    }
}

int main() {
    vector<int> arr = {12, 11, 13, 5, 6, 7};
    cout << "排序前的数组：" << endl;
    for (int num : arr) {
        cout << num << " ";
    }
    cout << endl;

    quickSort(arr, 0, arr.size() - 1);

    cout << "排序后的数组：" << endl;
    for (int num : arr) {
        cout << num << " ";
    }
    cout << endl;

    return 0;
}
```

### 9、map 和multimap unordered_map区别 为什么要有 unordered_map 使用场景是什么，这三者访问元素的时间复杂度 底层实现？

#### 区别

1. 底层数据结构不同：
   - `std::map` 和 `std::multimap` 底层通常基于红黑树实现，保证了元素的有序性。
   - `std::unordered_map` 底层基于哈希表实现，没有元素的顺序保证，但能够提供更快的查找速度（平均 O(1)）。
2. 元素的唯一性：
   - `std::map` 中的键值对是唯一的，如果插入已存在的键，则会替换原有的值。
   - `std::multimap` 中的键可以重复，允许多个键对应不同的值。
   - `std::unordered_map` 也要求键的唯一性，但哈希冲突时会使用链表或其他方式解决，因此允许在常数时间内插入重复的键。
3. 查找效率：
   - `std::map` 和 `std::multimap` 的查找效率为 O(log n)，因为基于红黑树实现。
   - `std::unordered_map` 的查找效率为平均 O(1)，最坏情况下为 O(n)（发生哈希冲突时）。
4. 内存占用：
   - `std::map` 和 `std::multimap` 需要额外的内存来存储红黑树结构，因此可能占用更多的内存。
   - `std::unordered_map` 通常占用较少的内存，但在哈希冲突较多时可能会占用较多内存来维护链表等结构。
5. 遍历顺序：
   - `std::map` 和 `std::multimap` 遍历时按照键的顺序（升序）进行。
   - `std::unordered_map` 没有固定的遍历顺序，取决于哈希表的实现和当前的负载因子。

#### 使用场景

1. 快速查找：`unordered_map` 的底层实现是哈希表，因此查找元素的平均时间复杂度为 O(1)，对于大量数据的快速查找非常高效。
2. 存储键值对：`unordered_map` 可以存储键值对，并且保证键的唯一性。这在需要建立键与值之间的映射关系时非常有用，比如字典、符号表等。
3. 去重：由于 `unordered_map` 的键是唯一的，可以用来去重，即只保留不重复的元素。
4. 缓存：`unordered_map` 可以用作缓存，存储计算结果以避免重复计算。
5. 替代数组：在一些情况下，`unordered_map` 可以替代数组来存储数据，尤其是当键的范围比较大或者不连续时。

#### 访问元素的时间复杂度和底层实现：

- `std::map` 和 `std::multimap` 的查找、插入和删除操作的平均时间复杂度为 O(log n)，因为它们是基于红黑树实现的，红黑树保证了元素的有序性。
- `std::unordered_map` 的查找、插入和删除操作的平均时间复杂度为 O(1)，最坏情况下为 O(n)，这是因为 `unordered_map` 使用哈希表来实现，哈希表可以在平均情况下提供常数时间的性能，但在发生哈希冲突时，会退化到链表或其他方式来解决冲突，导致最坏情况下的时间复杂度为 O(n)。

## 手撕： 

### 1、IPV4地址字符串转化为 32整型数字？

#### 思路

1. 将 IPv4 地址字符串按照点号 "." 分割成四个部分。
2. 将每个部分转换为整数，并将它们按照从左到右的顺序拼接起来，得到一个 32 位的整数。

举个例子，如果有一个 IPv4 地址字符串 "192.168.1.1"，它可以按照上述步骤转换为整数：

1. 分割成四个部分：192, 168, 1, 1。
2. 转换为整数并拼接：192 << 24 | 168 << 16 | 1 << 8 | 1。

```
#include <iostream>
#include <sstream>
#include <vector>

// 将IPv4地址字符串转换为32位整数
uint32_t ipToInteger(const std::string& ip) {
    // 创建一个向量来存储IP地址的各个部分
    std::vector<int> parts;
    // 使用字符串流来处理IP地址字符串
    std::stringstream ss(ip);
    std::string part;
    // 使用点号分割字符串，并将各个部分转换为整数并存储到向量中
    while (std::getline(ss, part, '.')) {
        parts.push_back(std::stoi(part));
    }
    // 将各个部分合并为一个32位整数并返回
    return (parts[0] << 24) | (parts[1] << 16) | (parts[2] << 8) | parts[3];
}

// 这里将stoi也实现一下
int stoi(const std::string& str) {
    int result = 0;
    for (char c : str) {
        if (c < '0' || c > '9') {
            // 非数字字符，返回错误值 0
            return 0;
        }
        // 将字符转换为数字，并累加到结果中
        result = result * 10 + (c - '0');
    }
    // 返回转换后的整数值
    return result;
}

int main() {
    std::string ip = "192.168.1.1";
    // 调用函数将IP地址字符串转换为整数
    uint32_t result = ipToInteger(ip);
    // 打印转换结果
    std::cout << "IP: " << ip << " => Integer: " << result << std::endl;
    return 0;
}
```

### 2、词频统计 保证次数相同基础上优先字母排序打印 ACM模式？

#### 问题描述

写一个 bash 脚本以统计一个文本文件 `words.txt` 中每个单词出现的频率。

为了简单起见，你可以假设：

- `words.txt`只包括小写字母和 `' '` 。
- 每个单词只由小写字母组成。
- 单词间由一个或多个空格字符分隔。

**示例:**

假设 `words.txt` 内容如下：

```
the day is sunny the the
the sunny is is
```

你的脚本应当输出（以词频降序排列）：

```
the 4
is 3
sunny 2
day 1
```

**说明:**

- 不要担心词频相同的单词的排序问题，每个单词出现的频率都是唯一的。
- 你可以使用一行 [Unix pipes](http://tldp.org/HOWTO/Bash-Prog-Intro-HOWTO-4.html) 实现吗？

我在LC上看到这里实际上考察的是一个对Linux命令的使用，如果有别的方式大家可以去学一下。

我就使用Linux命令给大家解答一下

```
# 使用 tr 命令将空格替换为换行符，然后使用 sort 命令排序，再使用 uniq -c 统计每个单词的频率，最后再次排序
cat words.txt | tr -s ' ' '\n' | sort | uniq -c | sort -nr | awk '{print $2, $1}'
```

这个命令的具体执行流程如下：

1. `cat words.txt`：读取文件 words.txt 的内容。
2. `tr -s ' ' '\n'`：将空格替换为换行符，即将每个单词分隔到一行。
3. `sort`：对单词进行排序。
4. `uniq -c`：统计每个单词出现的次数，并在前面加上频率。
5. `sort -nr`：按照频率降序排序。
6. `awk '{print $2, $1}'`：使用 awk 输出单词和频率，其中 $2 是单词，$1 是频率。