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

### 10、C++中多态怎么实现的（我答对基类的虚函数进行重写然后使用父类的指针或引用来调用，他说这是应用层，底层怎么实现的，我答了虚函数表那一套）

### 11、虚函数表的本质是什么（指针数组）

### 12、里面存了什么东西？（静态区的函数地址）

### 13、手撕两个链表逆置相加（20min）