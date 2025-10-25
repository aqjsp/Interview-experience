# 拼多多服务端C++开发一面

> 来源：https://www.nowcoder.com/share/jump/1728306134784

### 1、项目拷打

围绕项目问八股

### 2、STL你熟练吗？说说怎么设计的？（说了STL六大组件，我只回答4个，还有两个忘了）

#### STL组件

##### 1. 容器（Containers）

容器用于存储和管理数据。STL提供了多种类型的容器，每种容器都有其特定的用途和性能特点。

- **序列容器**：
  - `vector`：动态数组，支持随机访问，但在中间插入和删除元素效率较低。
  - `list`：双向链表，适合频繁的插入和删除操作。
  - `deque`：双端队列，支持两端高效的插入和删除操作。
  - `array`：固定大小的数组，支持随机访问。
- **关联容器**：
  - `set` 和 `multiset`：有序集合，支持唯一键或重复键。
  - `map` 和 `multimap`：有序映射，支持唯一键或重复键。
  - `unordered_set` 和 `unordered_multiset`：无序集合，基于哈希表实现。
  - `unordered_map` 和 `unordered_multimap`：无序映射，基于哈希表实现。

##### 2. 迭代器（Iterators）

迭代器用于遍历容器中的元素。它们提供了一种统一的方式来访问不同类型的容器。

- 输入迭代器：只能向前移动，用于读取数据。
- 输出迭代器：只能向前移动，用于写入数据。
- 前向迭代器：可以向前移动，支持多次遍历。
- 双向迭代器：可以向前和向后移动。
- 随机访问迭代器：支持任意位置的访问，类似于指针。

##### 3. 算法（Algorithms）

算法是对容器中的元素进行操作的函数。STL提供了大量的算法，这些算法都是泛型的，可以应用于任何支持相应迭代器的容器。

- **非修改性算法**：
  - `find`：查找指定值的元素。
  - `count`：计算指定值的元素个数。
  - `equal`：判断两个范围是否相等。
  - `for_each`：对每个元素执行指定的操作。
- **修改性算法**：
  - `sort`：对元素进行排序。
  - `reverse`：反转元素的顺序。
  - `remove`：移除指定值的元素。
  - `transform`：对每个元素应用一个函数。

##### 4. 仿函数（Function Objects）

仿函数是实现了`operator()`的类的对象，可以像普通函数一样调用。它们通常用于算法中，提供灵活的参数化行为。

- **一元函数对象**：接受一个参数并返回一个结果。
- **二元函数对象**：接受两个参数并返回一个结果。

常见的仿函数包括：

- `std::plus`：加法
- `std::minus`：减法
- `std::multiplies`：乘法
- `std::divides`：除法
- `std::modulus`：取模
- `std::negate`：取反
- `std::equal_to`：等于
- `std::not_equal_to`：不等于
- `std::greater`：大于
- `std::less`：小于
- `std::greater_equal`：大于等于
- `std::less_equal`：小于等于

##### 5. 适配器（Adapters）

适配器用于改变现有容器或函数的行为。STL提供了几种类型的适配器：

- **容器适配器**：
  - `stack`：后进先出（LIFO）的栈。
  - `queue`：先进先出（FIFO）的队列。
  - `priority_queue`：优先级队列。
- **函数适配器**：
  - `bind`：绑定函数和参数。
  - `mem_fn`：调用成员函数。
  - `not1` 和 `not2`：取反仿函数。

##### 6. 分配器（Allocators）

分配器负责管理内存的分配和释放。默认情况下，STL容器使用全局的`new`和`delete`操作符来管理内存，但也可以自定义分配器以适应特定的需求。

- 默认分配器：`std::allocator`
- 自定义分配器：可以通过继承`std::allocator`或实现自己的分配器来满足特定的内存管理需求。

##### 给个例子

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>
#include <stack>

int main() {
    // 创建一个vector容器
    std::vector<int> numbers = {5, 2, 8, 1, 9};

    // 使用迭代器遍历容器
    for (auto it = numbers.begin(); it != numbers.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    // 使用算法对容器进行操作
    std::sort(numbers.begin(), numbers.end()); // 排序
    std::reverse(numbers.begin(), numbers.end()); // 反转

    // 查找元素
    auto pos = std::find(numbers.begin(), numbers.end(), 8);
    if (pos != numbers.end()) {
        std::cout << "Found 8 at position " << std::distance(numbers.begin(), pos) << std::endl;
    } else {
        std::cout << "8 not found" << std::endl;
    }

    // 使用仿函数
    std::vector<int> result(numbers.size());
    std::transform(numbers.begin(), numbers.end(), result.begin(), std::negate<int>());

    for (const auto& num : result) {
        std::cout << num << " ";
    }
    std::cout << std::endl;

    // 使用容器适配器
    std::stack<int> s;
    for (const auto& num : numbers) {
        s.push(num);
    }

    while (!s.empty()) {
        std::cout << s.top() << " ";
        s.pop();
    }
    std::cout << std::endl;

    return 0;
}
```

### 3、适配器是什么知道吗 （变相提醒我剩余的组件，顺利回答）

### 4、仿函数是什么（变相提醒我剩余的组件，顺利回答）

### 5、STL空间配置器有几级知道吗？

STL的空间配置器（Allocator）在SGI STL实现中采用了双层级配置器的设计，以优化内存分配和释放的效率，同时减少内存碎片。

#### 1. 一级配置器（Primary Allocator）

一级配置器主要用于处理较大块的内存分配。它直接使用C语言的内存管理函数`malloc`和`free`来分配和释放内存。当需要分配的内存块大于128字节时，一级配置器会被调用。

##### 主要特点：

- 直接调用C函数：使用`malloc`和`free`等C语言函数进行内存分配和释放。
- 处理大块内存：适用于大于128字节的内存块。
- 简单高效：由于直接调用系统函数，实现简单且高效。

#### 2. 二级配置器（Secondary Allocator）

二级配置器主要用于处理较小块的内存分配。它使用内存池（Memory Pool）和自由链表（Free List）的机制来管理内存，从而减少内存碎片和提高分配效率。当需要分配的内存块小于或等于128字节时，二级配置器会被调用。

##### 主要特点：

- 内存池：一次性从系统中申请一大块内存，然后将这块内存划分为多个小块，用于后续的分配请求。
- 自由链表：维护一个链表，记录可用的小内存块。当有新的分配请求时，从自由链表中取出一个可用的内存块；当有内存块被释放时，将其放回自由链表。
- 减少内存碎片：通过内存池和自由链表的机制，减少了频繁分配和释放小块内存带来的内存碎片问题。
- 提高效率：减少了频繁调用系统内存管理函数的开销，提高了内存分配和释放的效率。

#### 具体实现

##### 一级配置器的实现

一级配置器的实现相对简单，主要依赖于C语言的内存管理函数。

```c++
template <class T>
class primary_allocator {
public:
    typedef T value_type;
    typedef T* pointer;
    typedef const T* const_pointer;
    typedef T& reference;
    typedef const T& const_reference;
    typedef size_t size_type;
    typedef ptrdiff_t difference_type;

    pointer allocate(size_type n, const void* hint = 0) {
        void* p = malloc(n * sizeof(T));
        if (!p) throw std::bad_alloc();
        return static_cast<pointer>(p);
    }

    void deallocate(pointer p, size_type n) {
        free(p);
    }

    void construct(pointer p, const T& value) {
        new(p) T(value);
    }

    void destroy(pointer p) {
        p->~T();
    }
};
```

##### 二级配置器的实现

二级配置器的实现较为复杂，涉及内存池和自由链表的管理。

```c++
template <class T>
class secondary_allocator {
public:
    typedef T value_type;
    typedef T* pointer;
    typedef const T* const_pointer;
    typedef T& reference;
    typedef const T& const_reference;
    typedef size_t size_type;
    typedef ptrdiff_t difference_type;

    pointer allocate(size_type n, const void* hint = 0) {
        if (n > 128 / sizeof(T)) {
            return primary_allocator<T>().allocate(n, hint);
        }

        // 从内存池中分配内存
        return static_cast<pointer>(chunk_alloc(n * sizeof(T), n));
    }

    void deallocate(pointer p, size_type n) {
        if (n > 128 / sizeof(T)) {
            primary_allocator<T>().deallocate(p, n);
            return;
        }

        // 将内存块放回自由链表
        obj* q = (obj*)p;
        q->free_list_link = free_list[freelist_index(n)].free_list_link;
        free_list[freelist_index(n)].free_list_link = q;
    }

    void construct(pointer p, const T& value) {
        new(p) T(value);
    }

    void destroy(pointer p) {
        p->~T();
    }

private:
    union obj {
        union obj* free_list_link;
        char client_data[1];
    };

    static obj* free_list[16];
    static size_t freelist_index(size_t bytes) {
        return (bytes + sizeof(obj) - 1) / sizeof(obj) - 1;
    }

    static obj* chunk_alloc(size_t size, int& nobjs) {
        obj* chunk = (obj*)malloc(size * nobjs);
        if (!chunk) {
            nobjs = 1;
            return (obj*)malloc(size);
        }

        obj* current_obj = chunk;
        obj* next_obj = 0;
        int i;
        for (i = 0; i < nobjs - 1; ++i) {
            current_obj->free_list_link = next_obj;
            current_obj = next_obj;
            next_obj = (obj*)((char*)current_obj + size);
        }
        current_obj->free_list_link = next_obj;
        free_list[freelist_index(size)].free_list_link = chunk;
        return chunk;
    }
};

obj* secondary_allocator<>::free_list[16] = {0};
```

#### 总结

- 一级配置器：处理大于128字节的内存块，直接使用`malloc`和`free`。
- 二级配置器：处理小于或等于128字节的内存块，使用内存池和自由链表来管理内存，减少内存碎片和提高效率。

### 6、为什么malloc有内存池，空间配置器有内存池，你应用层还要再额外使用内存池？（基于项目问）

#### 1. 更细粒度的控制

- 定制化需求：不同的应用程序对内存管理的需求不同。例如，某些应用程序可能需要频繁分配和释放特定大小的小内存块，而这些需求可能无法完全通过现有的内存池机制满足。通过在应用层实现内存池，可以针对特定需求进行优化。
- 减少系统调用：即使`malloc`和STL的空间配置器使用了内存池，它们仍然可能需要进行系统调用（如`brk`或`mmap`）来获取大块内存。应用层的内存池可以进一步减少这种系统调用的频率，提高性能。

#### 2. 减少内存碎片

- 固定大小的内存块：应用层的内存池可以预先分配固定大小的内存块，避免了不同大小的内存块交错分配导致的外部碎片问题。这对于频繁分配和释放小内存块的应用特别有用。
- 内存块的复用：通过在应用层管理内存池，可以更精细地控制内存块的复用，减少内存碎片的产生。

#### 3. 提高缓存命中率

- 局部性原理：将频繁访问的对象分配到连续的内存块中，可以利用CPU缓存的局部性原理，提高缓存命中率，从而提升程序的性能。
- 预分配：应用层的内存池可以在程序启动时预分配一大块内存，确保这些内存块在物理上是连续的，进一步提高缓存效率。

#### 4. 优化特定场景

- 高性能需求：在高性能计算、实时系统和嵌入式系统中，对内存分配的性能要求非常高。应用层的内存池可以针对这些特定场景进行优化，减少内存分配和释放的开销。
- 多线程环境：在多线程环境中，全局的内存分配器（如`malloc`）可能会成为竞争热点，导致性能瓶颈。应用层的内存池可以实现线程本地的内存管理，减少锁的竞争，提高并发性能。

#### 5. 减少内存泄漏

- 自动管理：应用层的内存池可以实现更严格的内存管理机制，例如在对象生命周期结束时自动释放内存，减少内存泄漏的风险。
- 池化技术：通过池化技术，可以将内存分配和释放的操作封装在一个模块中，减少手动管理内存的复杂性和错误。

给个简单的应用层内存池的实现示例，展示了如何在应用层实现内存池以优化内存管理：

```c++
#include <iostream>
#include <vector>
#include <mutex>

class MemoryPool {
public:
    MemoryPool(size_t block_size, size_t num_blocks)
        : block_size_(block_size), num_blocks_(num_blocks) {
        pool_.resize(num_blocks);
        for (size_t i = 0; i < num_blocks_; ++i) {
            pool_[i] = malloc(block_size_);
            free_list_.push_back(static_cast<char*>(pool_[i]));
        }
    }

    ~MemoryPool() {
        for (auto ptr : pool_) {
            free(ptr);
        }
    }

    void* allocate() {
        std::lock_guard<std::mutex> lock(mutex_);
        if (free_list_.empty()) {
            // 如果没有可用的内存块，可以从系统中申请更多
            void* ptr = malloc(block_size_);
            pool_.push_back(ptr);
            return ptr;
        }
        void* ptr = free_list_.back();
        free_list_.pop_back();
        return ptr;
    }

    void deallocate(void* ptr) {
        std::lock_guard<std::mutex> lock(mutex_);
        free_list_.push_back(static_cast<char*>(ptr));
    }

private:
    size_t block_size_;
    size_t num_blocks_;
    std::vector<void*> pool_;
    std::vector<char*> free_list_;
    std::mutex mutex_;
};

int main() {
    MemoryPool pool(128, 100); // 创建一个内存池，管理100个128字节的内存块

    void* ptr1 = pool.allocate();
    void* ptr2 = pool.allocate();

    // 使用内存块...

    pool.deallocate(ptr1);
    pool.deallocate(ptr2);

    return 0;
}
```

尽管`malloc`和STL的空间配置器已经实现了内存池机制，但在某些应用场景中，应用层额外使用内存池仍然是必要的。通过在应用层实现内存池，可以更细粒度地控制内存管理，减少内存碎片，提高缓存命中率，优化特定场景，减少内存泄漏，从而提升程序的整体性能和稳定性。

### 7、malloc的机制(我就答了brk, mmap没说清楚)

#### 1. 内存池机制

- 减少系统调用：`malloc` 通常会预先向操作系统申请一大块内存，然后自己管理这块内存。这样可以减少频繁的系统调用（如 `brk` 和 `mmap`），提高性能。
- 内存管理：`malloc` 使用内存池来管理已分配的内存块，通过链表或其他数据结构来跟踪空闲和已分配的内存块。

#### 2. 系统调用

- `brk` 和 `sbrk`：`brk` 和 `sbrk` 系统调用用于调整进程的堆大小。`brk` 设置程序的堆顶指针，而 `sbrk` 则相对于当前堆顶指针增加或减少堆的大小。`malloc` 通常在需要更多内存时使用 `brk` 或 `sbrk` 来扩展堆。
- `mmap`：`mmap` 系统调用用于将文件或设备映射到内存中，也可以用于分配匿名内存。`malloc` 在需要大块内存时通常使用 `mmap`，因为 `mmap` 分配的内存可以直接映射到文件或设备，避免了堆的碎片问题。

#### 3. 内存块管理

- 内存块结构：每个内存块通常包含元数据，如块的大小、是否已分配等。这些元数据用于管理内存块的分配和释放。
- 链表：`malloc` 使用链表来管理空闲内存块。当需要分配内存时，`malloc` 会遍历链表，查找合适的空闲块。如果找到合适的块，就将其从链表中移除并返回给用户。如果找不到合适的块，`malloc` 会请求更多的内存。
- 合并空闲块：当内存块被释放时，`malloc` 会检查相邻的内存块是否也是空闲的，如果是，则将它们合并成一个更大的空闲块，以减少内存碎片。

#### 4. 分配算法

- 最佳适配：`malloc` 可以使用最佳适配算法，即在空闲链表中查找最接近所需大小的空闲块。
- 首次适配：`malloc` 也可以使用首次适配算法，即在空闲链表中查找第一个足够大的空闲块。
- 下次适配：`malloc` 还可以使用下次适配算法，即从上次查找的位置开始查找合适的空闲块。

#### 5. 内存池的具体实现

##### ptmalloc

- 多线程支持：`ptmalloc` 是 GNU C 库中 `malloc` 的实现，支持多线程。它使用多个内存池（分配区）来减少锁的竞争，提高并发性能。
- 主分配区和从分配区：每个进程有一个主分配区，使用 `brk` 从堆中分配内存。每个线程可以有多个从分配区，使用 `mmap` 从内存映射区域分配内存。
- 内存块（Chunk）：`ptmalloc` 将内存块组织成双向链表，每个块包含元数据和用户数据。`malloc` 从链表中分配内存块，`free` 将内存块返回到链表中。
- Bins：`ptmalloc` 使用 `bins` 来管理不同大小的内存块。`bins` 是一个数组，每个元素是一个链表，管理特定大小范围的内存块。`fast bins` 用于管理小于 64 字节的小内存块，`small bins` 用于管理 64 到 512 字节的内存块，`large bins` 用于管理大于 512 字节的内存块。

#### 6. 内存释放

- 延迟释放：`malloc` 可以延迟释放内存块，即将内存块返回到内存池中，而不是立即释放给操作系统。这样可以减少频繁的系统调用，提高性能。
- 内存池释放：当内存池中的所有内存块都被释放时，`malloc` 可以将整个内存池释放给操作系统。

#### 7. 内存碎片管理

- 内存碎片：频繁的内存分配和释放会导致内存碎片，即内存中存在许多小的空闲块，但无法满足大块内存的分配请求。`malloc` 通过合并空闲块和使用 `mmap` 分配大块内存来减少内存碎片。
- 内存池扩展：当内存池中的空闲块不足以满足新的内存请求时，`malloc` 会扩展内存池，请求更多的内存。

#### 8. 性能优化

- 快速路径：`malloc` 通常会实现快速路径，即对于常见的情况（如小内存块的分配和释放），使用优化的算法和数据结构，减少不必要的检查和操作。
- 缓存友好：`malloc` 会尽量将频繁访问的内存块分配到连续的地址空间，以提高缓存命中率。

#### 示例

给一个简化的 `malloc` 实现示例，展示了基本的内存池和链表管理机制：

```c++
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct Chunk {
    size_t size;
    struct Chunk *next;
    int is_free;
} Chunk;

Chunk *head = NULL;

void *my_malloc(size_t size) {
    Chunk *current = head;
    while (current != NULL) {
        if (current->is_free && current->size >= size) {
            current->is_free = 0;
            return (void *)(current + 1);
        }
        current = current->next;
    }

    size_t total_size = sizeof(Chunk) + size;
    Chunk *new_chunk = (Chunk *)sbrk(total_size);
    if (new_chunk == (void *)-1) {
        return NULL;
    }

    new_chunk->size = size;
    new_chunk->is_free = 0;
    new_chunk->next = head;
    head = new_chunk;

    return (void *)(new_chunk + 1);
}

void my_free(void *ptr) {
    if (ptr == NULL) {
        return;
    }

    Chunk *chunk = (Chunk *)ptr - 1;
    chunk->is_free = 1;

    // 合并相邻的空闲块
    Chunk *prev = NULL;
    Chunk *current = head;
    while (current != NULL) {
        if (current->is_free && current != chunk) {
            if (prev != NULL && prev->is_free && (char *)prev + prev->size + sizeof(Chunk) == (char *)current) {
                prev->size += current->size + sizeof(Chunk);
                prev->next = current->next;
                if (current == head) {
                    head = prev;
                }
                current = prev;
            }
        }
        prev = current;
        current = current->next;
    }
}

int main() {
    void *ptr1 = my_malloc(100);
    void *ptr2 = my_malloc(200);

    printf("ptr1: %p\n", ptr1);
    printf("ptr2: %p\n", ptr2);

    my_free(ptr1);
    my_free(ptr2);

    return 0;
}
```

### 8、你刚刚说brk只是移动了task_struct中的brk指针，实际没有分配物理内存，那么如果真的访问了系统怎么做的？（回答了缺页中断、mmu映射那一套）

`brk` 系统调用只是将进程的堆顶指针（`brk` 指针）移动到更高的地址，这一步操作仅涉及到虚拟内存的分配，并没有实际分配物理内存。这意味着，当你使用 `malloc` 请求一小块内存（通常小于128KB）时，`malloc` 会调用 `brk` 或 `sbrk` 来扩展堆的大小，但这一步并不会立即导致物理内存的分配。

#### 虚拟内存与物理内存的映射

当程序实际访问通过 `brk` 分配的虚拟内存时，操作系统会处理以下步骤：

1. 缺页中断（Page Fault）：
   - 当程序尝试访问一个尚未映射到物理内存的虚拟地址时，会触发缺页中断。缺页中断是硬件检测到无效的内存访问时产生的异常。
   - 缺页中断处理程序会捕获这个异常，并检查虚拟地址是否在合法的虚拟地址范围内（例如，是否在 `brk` 指针所指示的范围内）。
2. 分配物理页：
   - 如果虚拟地址是合法的，但尚未映射到物理内存，操作系统会分配一个物理页。
   - 操作系统会在内存管理单元（MMU）中建立虚拟地址到物理地址的映射关系。这个映射关系通常存储在页表中。
3. 更新页表：
   - 操作系统会更新页表，将新的物理页与虚拟地址关联起来。
   - 页表项（PTE）会包含物理页的地址、访问权限（读、写、执行）等信息。
4. 恢复执行：缺页中断处理完成后，程序会恢复执行，继续访问该虚拟地址。此时，访问的是已经映射到物理内存的地址，不会再触发缺页中断。

#### 示例流程

假设一个进程调用了 `malloc(30K)`，`malloc` 调用 `brk` 将堆顶指针向上移动 30K。此时，虚拟内存已经分配，但没有物理内存与之对应。当程序实际访问这块内存时，会发生以下步骤：

1. 程序访问内存：

```c++
char *ptr = malloc(30000);  // 调用 malloc 分配 30K 内存
*ptr = 'A';  // 尝试写入一个字符
```

2. 触发缺页中断：当程序尝试写入 `*ptr` 时，CPU 发现该地址尚未映射到物理内存，触发缺页中断。

3. 处理缺页中断：

   - 缺页中断处理程序检查虚拟地址是否在合法范围内。

   - 如果合法，操作系统分配一个物理页，并更新页表。

4. 恢复执行：缺页中断处理完成后，程序继续执行，写入操作成功。

#### 优点

- 延迟分配：这种方式允许操作系统延迟分配物理内存，直到真正需要时才分配，节省了物理内存资源。
- 按需分配：每次只分配实际需要的物理页，避免了不必要的内存浪费。

#### 缺点

- 性能开销：频繁的缺页中断会增加系统的开销，尤其是在大量小内存块的分配和访问场景中。
- 复杂性：这种机制增加了内存管理的复杂性，需要操作系统和硬件的紧密配合。

### 9、Linux内核中内存怎么管理的？（回答了伙伴系统、大页分配，他不太满意）

#### 1. 物理内存管理

##### 1.1 物理内存模型

Linux内核通过将物理内存划分为页（Page）来管理。每个页的大小通常是4KB。内核使用 `struct page` 结构体来描述每个物理页的状态信息，如页的状态（脏、锁定）、引用计数等。

##### 1.2 内存区域（Zone）

为了更好地管理不同类型的内存，Linux内核将物理内存划分为不同的区域（Zone）。常见的内存区域包括：

- ZONE_DMA：用于直接内存访问（DMA）的内存区域。
- ZONE_NORMAL：常规内存区域，内核可以直接映射。
- ZONE_HIGHMEM：高地址内存区域，主要用于32位系统，内核不能直接映射这部分内存。

##### 1.3 内存分配器

Linux内核提供了多种内存分配器来管理物理内存：

- 页分配器（Buddy Allocator）：用于分配连续的物理页。它通过将物理内存划分为不同大小的块（伙伴），并使用链表来管理这些块，从而实现高效的内存分配和回收。
- SLAB分配器：用于分配小块内存。SLAB分配器通过预分配内存块（slab）来减少内存碎片，并提高内存分配的效率。每个slab包含多个相同大小的对象。

#### 2. 虚拟内存管理

##### 2.1 虚拟地址空间

Linux内核将虚拟地址空间划分为用户空间和内核空间。在32位系统中，通常将4GB的虚拟地址空间划分为3GB的用户空间和1GB的内核空间。在64位系统中，虚拟地址空间更大，但划分方式类似。

##### 2.2 页表

虚拟地址到物理地址的映射通过页表（Page Table）实现。页表由多个层次的表组成，每个表项（PTE）包含物理页的地址和访问权限等信息。页表由操作系统维护，MMU（内存管理单元）使用页表来完成虚拟地址到物理地址的转换。

##### 2.3 虚拟内存区域（VMA）

每个进程的虚拟地址空间被划分为多个虚拟内存区域（VMA），每个VMA表示一个连续的虚拟地址范围。VMA包含关于该区域的详细信息，如起始地址、结束地址、访问权限等。

#### 3. 内存分配与回收

##### 3.1 内存分配

Linux内核提供了多种内存分配函数：

- kmalloc：分配连续的物理内存，返回虚拟地址。
- vmalloc：分配虚拟地址连续但物理地址不连续的内存。
- get_free_page：分配一个物理页。
- alloc_pages：分配多个连续的物理页。

#### 3.2 内存回收

Linux内核通过多种机制回收不再使用的内存：

- LRU（最近最少使用）算法：将最近最少使用的页标记为可回收，以便在内存紧张时优先回收。
- Swap：将不经常使用的页换出到磁盘上的交换分区，以释放物理内存。
- 内存压缩：通过压缩内存中的数据来减少内存使用。

#### 4. 内存管理机制

##### 4.1 缺页中断处理

当进程访问一个未映射到物理内存的虚拟地址时，会触发缺页中断。缺页中断处理程序会分配一个物理页，并更新页表，建立虚拟地址到物理地址的映射。

##### 4.2 页面替换算法

Linux内核使用多种页面替换算法来决定哪些页应该被换出到磁盘。常见的算法包括：

- LRU（最近最少使用）
- CLOCK：基于时钟的页面替换算法
- Working Set：基于工作集的页面替换算法

##### 4.3 内存碎片管理

Linux内核通过多种机制来减少内存碎片：

- 伙伴系统：通过合并相邻的空闲页来减少外部碎片。
- SLAB分配器：通过预分配内存块来减少内部碎片。

#### 5. 内存管理优化

##### 5.1 内存预分配

为了减少缺页中断的频率，Linux内核支持内存预分配。例如，`mmap` 系统调用可以预先分配虚拟地址空间，实际的物理内存分配在首次访问时进行。

##### 5.2 内存压缩

在内存紧张时，Linux内核可以通过压缩内存中的数据来减少内存使用。例如，`zram` 驱动可以将内存中的数据压缩并存储在内存中，从而释放物理内存。

##### 5.3 内存回收策略

Linux内核通过多种策略来回收不再使用的内存，包括：

- 主动回收：定期扫描内存，回收不活跃的页。
- 被动回收：在内存紧张时，通过缺页中断处理程序回收内存。

### 10、C++中多态怎么实现的？

#### 编译时多态（静态多态）

编译时多态是指在编译阶段就确定了函数调用的方式。C++中最常见的编译时多态是通过函数重载（Function Overloading）和运算符重载（Operator Overloading）来实现的。

##### 1. 函数重载（Function Overloading）

函数重载是指在同一个作用域内定义多个同名函数，但这些函数的参数列表不同（参数的数量、类型或顺序不同）。编译器根据传入的参数类型和数量来选择合适的函数版本进行调用。

```c++
void print(int x) {
    std::cout << "Integer: " << x << std::endl;
}

void print(double x) {
    std::cout << "Double: " << x << std::endl;
}

int main() {
    print(10);      // 调用 void print(int)
    print(3.14);    // 调用 void print(double)
    return 0;
}
```

##### 2. 运算符重载（Operator Overloading）

运算符重载是指重新定义某个运算符在特定类型对象上的行为。通过运算符重载，可以使用标准的运算符符号来操作自定义的数据类型，使代码更直观和自然。

```c++
class Complex {
public:
    double real, imag;

    Complex(double r, double i) : real(r), imag(i) {}

    // 重载加法运算符
    Complex operator+(const Complex& other) const {
        return Complex(real + other.real, imag + other.imag);
    }

    // 重载输出运算符
    friend std::ostream& operator<<(std::ostream& os, const Complex& c) {
        os << c.real << " + " << c.imag << "i";
        return os;
    }
};

int main() {
    Complex c1(1.0, 2.0);
    Complex c2(3.0, 4.0);
    Complex c3 = c1 + c2;  // 使用重载的加法运算符
    std::cout << c3 << std::endl;  // 输出 4 + 6i
    return 0;
}
```

#### 运行时多态（动态多态）

##### 1. 虚函数（Virtual Functions）

###### 1.1 虚函数的定义

虚函数是在基类中使用 `virtual` 关键字声明的成员函数。虚函数允许派生类重写（Override）基类中的同名函数，从而实现多态行为。

```c++
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
```

###### 1.2 虚函数的重写

派生类中的虚函数必须与基类中的虚函数具有相同的函数签名（函数名、参数列表和返回类型），才能构成重写。重写的关键在于 `override` 关键字，它用于显式地表示派生类中的函数重写了基类中的虚函数。

```c++
class Derived : public Base {
public:
    void display() override {
        std::cout << "Display from Derived" << std::endl;
    }
};
```

##### 2. 动态绑定（Dynamic Binding）

###### 2.1 动态绑定的实现

动态绑定是指在运行时根据对象的实际类型来调用相应的函数。在C++中，动态绑定通过虚函数表（Virtual Table, vtable）和虚表指针（Virtual Table Pointer, vptr）来实现。

- 虚函数表（vtable）：每个类都有一个虚函数表，表中存储了该类所有虚函数的地址。
- 虚表指针（vptr）：每个对象都有一个虚表指针，指向其所属类的虚函数表。

###### 2.2 动态绑定的过程

1. 对象创建：当创建一个对象时，编译器会在对象的内存布局中添加一个虚表指针，该指针指向类的虚函数表。
2. 函数调用：当通过基类指针或引用调用虚函数时，编译器会通过虚表指针查找虚函数表，找到实际的函数地址，从而调用派生类中的函数。

```c++
Base* basePtr = new Derived();
basePtr->display();  // 输出 "Display from Derived"
```

##### 3. 构成多态的条件

###### 3.1 继承关系

多态依赖于类之间的继承关系。至少需要一个基类和一个或多个派生类。派生类从基类继承属性和方法，并可以重写基类的方法。

###### 3.2 虚函数

基类中必须有一个或多个虚函数。这些虚函数是用 `virtual` 关键字声明的。虚函数允许派生类重写它们，从而实现多态性。

###### 3.3 基类指针或引用

必须通过基类的指针或引用调用虚函数。这是实现多态性的关键，因为通过基类指针或引用调用虚函数时，程序会根据实际指向的对象类型调用对应的方法。

##### 4. 析构函数的多态

###### 4.1 虚析构函数

如果基类的析构函数是虚函数，那么派生类的析构函数也会被视为虚函数。这样可以确保在通过基类指针删除派生类对象时，能够正确调用派生类的析构函数，从而避免内存泄漏。

```c++
class Base {
public:
    virtual ~Base() {
        std::cout << "Base destructor" << std::endl;
    }
};

class Derived : public Base {
public:
    ~Derived() {
        std::cout << "Derived destructor" << std::endl;
    }
};

int main() {
    Base* basePtr = new Derived();
    delete basePtr;  // 输出 "Derived destructor" 和 "Base destructor"
    return 0;
}
```

##### 5. 抽象类（Abstract Class）

抽象类是包含一个或多个纯虚函数（Pure Virtual Function）的类。纯虚函数是在基类中声明但没有实现的虚函数，形式为 `virtual ReturnType FunctionName() = 0;`。抽象类不能被实例化，但可以作为基类被派生。

```c++
class AbstractClass {
public:
    virtual void doSomething() = 0;  // 纯虚函数
};

class ConcreteClass : public AbstractClass {
public:
    void doSomething() override {
        std::cout << "Doing something in ConcreteClass" << std::endl;
    }
};
```

##### 6. 协变返回类型（Covariant Return Types）

协变允许派生类的虚函数返回类型与基类的虚函数返回类型不同，但必须是基类返回类型的派生类。

```c++
class Base {};
class Derived : public Base {};

class BaseClass {
public:
    virtual Base* getBase() {
        return new Base();
    }
};

class DerivedClass : public BaseClass {
public:
    virtual Derived* getBase() override {
        return new Derived();
    }
};
```

### 11、虚函数表的本质是什么？

主要用于解决在继承层次结构中的虚函数调用问题。**虚函数表本质上是一个函数指针数组**，其中每个条目都是指向虚函数的指针。通过虚函数表，编译器和程序能够在运行时确定调用哪个版本的虚函数。

#### 1. 虚函数表的定义

虚函数表是一个静态数组，每个包含虚函数的类都有自己的虚函数表。虚函数表中存储了该类中所有虚函数的地址。虚函数表在编译时生成，并且每个类只有一个虚函数表。

#### 2. 虚表指针（vptr）

每个对象都有一个虚表指针（Virtual Table Pointer，简称 vptr），指向其所属类的虚函数表。虚表指针通常位于对象的内存布局的最前面，由编译器自动插入。

#### 3. 虚函数表的结构

虚函数表的结构如下：

- 虚函数表：一个函数指针数组，每个条目指向一个虚函数的实现。
- 虚表指针：每个对象中包含一个指针，指向其所属类的虚函数表。

#### 4. 虚函数表的工作原理

##### 4.1 对象创建

当创建一个对象时，编译器会在对象的内存布局中插入一个虚表指针，并将其初始化为指向该类的虚函数表。

```c++
class Base {
public:
    virtual void func1() { std::cout << "Base::func1" << std::endl; }
    virtual void func2() { std::cout << "Base::func2" << std::endl; }
};

class Derived : public Base {
public:
    void func1() override { std::cout << "Derived::func1" << std::endl; }
    void func2() override { std::cout << "Derived::func2" << std::endl; }
};

int main() {
    Base* basePtr = new Derived();
    return 0;
}
```

上述代码中，`Derived` 对象的内存布局如下：

```c++
+-----------------+
| vptr            |  -> 指向 Derived 的虚函数表
+-----------------+
| ...             |
+-----------------+
```

##### 4.2 虚函数调用

当通过基类指针或引用调用虚函数时，编译器会通过虚表指针查找虚函数表，找到实际的函数地址，从而调用派生类中的函数。

```c++
basePtr->func1();  // 输出 "Derived::func1"
basePtr->func2();  // 输出 "Derived::func2"
```

具体步骤如下：

1. 获取虚表指针：通过基类指针 `basePtr` 获取对象的虚表指针 `vptr`。
2. 查找虚函数表：通过虚表指针 `vptr` 查找虚函数表。
3. 定位函数地址：根据虚函数的索引在虚函数表中找到对应的函数地址。
4. 调用函数：根据找到的函数地址调用相应的虚函数。

#### 5. 虚函数表的生成

虚函数表的生成由编译器在编译阶段完成。编译器会为每个包含虚函数的类生成一个虚函数表，并将虚函数的地址填充到表中。

#### 6. 虚函数表的继承

在继承关系中，派生类会继承基类的虚函数表，并在其中添加自己新增的虚函数的地址。如果派生类重写了基类的虚函数，虚函数表中的相应条目会被更新为指向派生类的实现。

```c++
class Base {
public:
    virtual void func1() { std::cout << "Base::func1" << std::endl; }
    virtual void func2() { std::cout << "Base::func2" << std::endl; }
};

class Derived : public Base {
public:
    void func1() override { std::cout << "Derived::func1" << std::endl; }
    void func2() override { std::cout << "Derived::func2" << std::endl; }
    virtual void func3() { std::cout << "Derived::func3" << std::endl; }
};

// 基类的虚函数表
// +-----------------+
// | Base::func1     |
// +-----------------+
// | Base::func2     |
// +-----------------+

// 派生类的虚函数表
// +-----------------+
// | Derived::func1  |
// +-----------------+
// | Derived::func2  |
// +-----------------+
// | Derived::func3  |
// +-----------------+
```

#### 7. 虚函数表的内存开销

由于每个对象都需要一个虚表指针，因此虚函数表会增加对象的内存开销。对于每个包含虚函数的类，编译器会生成一个虚函数表，这也会占用一定的内存。然而，这种开销通常是可接受的，因为虚函数表带来的多态性和灵活性远大于其内存开销。

#### 8. 虚函数表的性能影响

虚函数调用涉及到两次间接寻址：一次是通过虚表指针找到虚函数表，另一次是通过虚函数表找到函数地址。这比直接调用非虚函数稍微慢一些，但在大多数情况下，这种性能差异是可以忽略不计的。

### 12、手撕两个链表逆置相加

#### 思路

1. 链表逆序存储：由于链表中的数字是逆序存储的，我们可以逐位相加两个链表的数字，类似于手动加法时从右到左加。
2. 进位处理：在相加过程中，如果某一位的和大于等于10，就需要产生进位。
3. 构建新链表：将每位的和（考虑进位）存储到新的链表中。
4. 遍历两个链表：同时遍历两个链表，直到两个链表都遍历完，并处理任何剩余的进位。

#### 参考代码

```c++
#include <iostream>
using namespace std;

// 定义链表节点结构
struct ListNode {
    int val; // 节点值
    ListNode* next; // 指向下一个节点的指针
    ListNode(int x) : val(x), next(nullptr) {} // 构造函数
};

class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode* dummyHead = new ListNode(0); // 创建一个虚拟头节点
        ListNode* current = dummyHead; // 用于构建新链表
        int carry = 0; // 进位初始化为 0

        // 遍历两个链表
        while (l1 != nullptr || l2 != nullptr || carry != 0) {
            int sum = carry; // 从进位开始
            if (l1 != nullptr) { // 如果 l1 不为空
                sum += l1->val; // 加上 l1 当前位的值
                l1 = l1->next; // 移动到 l1 的下一个节点
            }
            if (l2 != nullptr) { // 如果 l2 不为空
                sum += l2->val; // 加上 l2 当前位的值
                l2 = l2->next; // 移动到 l2 的下一个节点
            }
            
            carry = sum / 10; // 计算新的进位
            current->next = new ListNode(sum % 10); // 创建新节点存储当前位的和
            current = current->next; // 移动到新节点
        }

        return dummyHead->next; // 返回新链表的头节点
    }
};

// 辅助函数用于打印链表
void printList(ListNode* head) {
    while (head) {
        cout << head->val << " ";
        head = head->next;
    }
    cout << endl;
}

// 主函数示例
int main() {
    // 创建第一个链表 l1 = [2, 4, 3] (表示数字 342)
    ListNode* l1 = new ListNode(2);
    l1->next = new ListNode(4);
    l1->next->next = new ListNode(3);

    // 创建第二个链表 l2 = [5, 6, 4] (表示数字 465)
    ListNode* l2 = new ListNode(5);
    l2->next = new ListNode(6);
    l2->next->next = new ListNode(4);

    // 创建 Solution 对象并调用 addTwoNumbers
    Solution solution;
    ListNode* result = solution.addTwoNumbers(l1, l2);

    // 打印结果链表
    cout << "Result: ";
    printList(result); // 输出: [7, 0, 8] (表示数字 807)

    return 0;
}
```

