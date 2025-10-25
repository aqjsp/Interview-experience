# 腾讯天美一面凉经

> 来源：https://www.nowcoder.com/discuss/574991853695631360

## C++

### 1、全局静态变量和函数静态变量的初始化顺序？

1. 全局静态变量的初始化顺序：
   - 在同一个编译单元中，全局静态变量的初始化顺序是按照它们在代码中的出现顺序进行的。这意味着先定义的静态变量会先初始化，后定义的静态变量会后初始化。
   - 在不同的编译单元中，全局静态变量的初始化顺序是不确定的。这意味着一个编译单元中的全局静态变量可能会在另一个编译单元中的全局静态变量之前或之后初始化，具体取决于编译器和链接器的实现。
2. 函数静态变量的初始化顺序：
   - 函数内部的静态变量的初始化顺序与它们在函数内部的声明顺序相同。这意味着先声明的静态变量会先初始化，后声明的静态变量会后初始化。
   - 函数内部的静态变量只会在第一次调用函数时初始化，之后的调用不会重新初始化。

### 2、谈一下对多态的理解？

1. 编译时多态（静态多态）：

   编译时多态是通过函数重载和运算符重载来实现的。在编译时，编译器根据调用的函数或运算符的参数类型和数量来确定具体调用哪个函数或运算符，这就是编译时多态。例如：

   ```
   int add(int a, int b) {
       return a + b;
   }
   
   double add(double a, double b) {
       return a + b;
   }
   
   int result1 = add(1, 2); // 调用第一个 add 函数
   double result2 = add(1.5, 2.5); // 调用第二个 add 函数
   ```

2. 运行时多态（动态多态）：

   运行时多态是通过继承和虚函数来实现的。在运行时，程序根据对象的实际类型来确定调用哪个函数，这就是运行时多态。在 C++ 中，通过在基类中声明虚函数，在派生类中重写（override）这些虚函数，然后通过基类指针或引用来调用这些函数，就可以实现运行时多态。例如：

   ```
   class Animal {
   public:
       virtual void makeSound() {
           cout << "Animal makes a sound" << endl;
       }
   };
   
   class Dog : public Animal {
   public:
       void makeSound() override {
           cout << "Dog barks" << endl;
       }
   };
   
   class Cat : public Animal {
   public:
       void makeSound() override {
           cout << "Cat meows" << endl;
       }
   };
   
   int main() {
       Animal* animal1 = new Dog();
       Animal* animal2 = new Cat();
   
       animal1->makeSound(); // 输出 "Dog barks"
       animal2->makeSound(); // 输出 "Cat meows"
   
       delete animal1;
       delete animal2;
   
       return 0;
   }
   ```

   例子中，Animal 类中的 makeSound() 函数是虚函数，在 Dog 和 Cat 类中进行了重写。在 main 函数中，通过 Animal 类型的指针调用 makeSound() 函数时，实际调用的是指向的对象的类型对应的函数，这就是运行时多态。

### 3、虚函数表的存储位置？

在 C++ 中，虚函数表（virtual function table，简称 vtable）是用于实现动态多态性的一种机制，它存储了类的虚函数的地址。虚函数表通常是针对每个包含虚函数的类生成的，每个对象的内存中都有一个指向其所属类的虚函数表的指针（通常被称为虚指针），用于在运行时确定调用哪个函数。

虚函数表的存储位置通常是在每个对象的内存布局中，具体位置取决于编译器和操作系统的实现。一般来说，虚函数表位于对象的内存布局的起始位置或者末尾，但并不是标准规定。虚函数表本身是一个指针数组，其中存储了指向每个虚函数的指针。

给个简化的示意图，展示一个包含虚函数的类的内存布局：

```
  +----------------+
  | 虚函数表指针   |  --> 指向虚函数表的指针
  +----------------+
  | 其他成员变量   |  --> 类的其他成员变量
  +----------------+

  虚函数表：
  +----------------+
  | 虚函数指针 1   |  --> 指向第一个虚函数的指针
  +----------------+
  | 虚函数指针 2   |  --> 指向第二个虚函数的指针
  +----------------+
  | ...            |
  +----------------+
```

示意图中，一个对象的内存布局包括了一个指向虚函数表的指针和其他成员变量。虚函数表本身是一个指针数组，每个元素指向对应虚函数的实际实现。当调用一个虚函数时，程序会根据对象的虚函数表指针找到对应的虚函数表，然后根据函数在虚函数表中的索引找到实际的函数实现进行调用。

### 4、运行时刻能把虚函数表拿出来吗？

在程序运行时，虚函数表是存储在内存中的，但通常情况下，C++ 并没有提供直接的语言级别的手段来访问或操作虚函数表。虚函数表的内部结构和存储位置是由编译器决定的，不同的编译器和不同的平台可能有不同的实现方式。

虚函数表存储在对象的内存布局中，通常是作为对象的第一个成员，即对象的起始位置处。每个包含虚函数的类都有自己的虚函数表，当对象被创建时，编译器会在对象的内存中插入一个指向类的虚函数表的指针（通常被称为虚指针），用于在运行时进行动态绑定。

由于虚函数表的内部结构和存储位置是由编译器决定的，并且 C++ 标准并没有规定如何访问虚函数表，因此在标准的 C++ 中，通常无法直接从程序中访问虚函数表。试图直接访问虚函数表可能会导致不可移植的行为，并且可能会破坏程序的健壮性和可维护性。

### 5、虚函数表指针存储位置？

在 C++ 中，对于包含虚函数的类的每个对象，都会有一个指向其所属类的虚函数表（vtable）的指针，通常被称为虚指针（vptr）。这个指针是由编译器在编译阶段自动插入到类的对象中的。

虚指针的存储位置通常位于对象内存布局的开始部分，也就是说，在对象内存的最前面。具体来说，虚指针通常存储在对象内存的起始位置，作为对象的第一个成员。这样做的目的是为了在运行时能够快速地找到对象所属类的虚函数表，从而实现动态多态性。

当一个对象被创建时，编译器会在对象内存中插入这个虚指针，并将其指向该对象所属类的虚函数表。这个虚指针在运行时被用来进行动态绑定，即在调用虚函数时，会根据虚指针指向的虚函数表来确定实际调用的函数。

### 6、拿到虚函数表地址后，是否可以改写虚函数表的内容？

在常规的情况下，虚函数表（vtable）通常是存储在程序的只读数据段（例如 .rodata 段）中的，这意味着虚函数表的内容是只读的，不能被修改。如果尝试修改只读数据段的内容，通常会导致操作系统抛出段错误（segmentation fault）或访问权限错误（access violation），从而导致程序异常终止。

虚函数表的只读性是由操作系统和硬件的内存保护机制来保证的，这样设计的目的是为了确保程序的安全性和稳定性。虚函数表中存储的是类的虚函数地址，这些地址在程序运行过程中是不会改变的，因此也不需要被修改。

## 操作系统

### 1、说一下虚拟内存？

虚拟内存是一种计算机内存管理技术，它通过将物理内存（RAM）和磁盘空间结合起来，为每个进程提供了一种看上去是连续的内存空间，称为虚拟地址空间。虚拟内存的概念使得每个进程都拥有自己独立的内存空间，从而使得每个进程都可以运行在自己的地址空间中，互不干扰。

特点：

1. 虚拟地址空间：每个进程都有自己的虚拟地址空间，它是一个连续的地址空间，从逻辑上看，每个进程都拥有整个地址空间。这使得每个进程都可以使用相同的地址空间来访问内存，而不需要关心物理内存的实际分配情况。
2. 分页机制：虚拟内存将物理内存分割成固定大小的页面（page），通常是4KB或者更大。虚拟地址空间也被划分成与物理页面相同大小的虚拟页面（virtual page）。当进程访问虚拟内存时，操作系统会将虚拟页面映射到物理页面，这样就实现了虚拟内存到物理内存的映射。
3. 分页调度：当物理内存不足时，操作系统可以将不常用的页面从物理内存移动到硬盘上的交换空间（swap space）中，这样就释放了物理内存供其他进程使用。需要使用被换出的页面时，操作系统会将其重新加载到物理内存中。
4. 内存保护：虚拟内存可以实现内存保护机制，每个页面都有相应的访问权限位（如读、写、执行权限），当进程试图访问不允许的内存区域时，操作系统会产生一个异常，从而阻止进程访问非法内存。
5. 共享内存：虚拟内存还可以实现共享内存的机制，多个进程可以将同一个物理页面映射到它们各自的虚拟地址空间中，从而实现了进程间的共享内存。

### 2、Linux上有个二进制程序一直在运行，修改代码后重新编译把原来的二进制程序覆盖了，会怎么样？

1. 已经运行的程序不会立即受到影响：正在运行的程序会继续使用原来的二进制文件，直到它被终止并重新启动。这是因为当一个程序被加载到内存中运行时，操作系统会将其内容加载到内存中，并且不会再次读取磁盘上的文件，因此即使文件被修改，运行中的程序也不会立即受到影响。
2. 新的程序会在下次启动时生效：当你终止当前运行的程序并重新启动它时，操作系统会加载新的二进制程序，并且新的程序将会生效。这意味着你的修改将会在下一次程序启动时生效。
3. 注意文件的锁定和权限：在 Linux 系统上，如果一个文件正在被使用，例如正在运行的程序正在读取它，那么你不能直接修改或删除这个文件，因为文件被锁定了。此外，你需要有足够的权限来修改这个文件，否则你将无法进行修改操作。

### 3、进程占用的内存比较多，该怎么调试是什么情况？

当一个进程占用的内存比较多时，可能会遇到内存泄漏（memory leak）或者内存使用不当等问题。

1. 使用内存分析工具：可以使用专门的内存分析工具来检测内存泄漏和内存使用情况。例如，Valgrind 是一个常用的开源工具，它可以检测内存泄漏和内存错误，并提供详细的报告。
2. 监控系统资源：使用系统监控工具（如 top、htop、ps 等）来监视进程的内存使用情况。通过观察进程的内存占用情况和系统资源的使用情况，可以初步判断进程是否存在内存泄漏或者内存使用异常的情况。
3. 检查代码中的内存分配和释放：仔细检查代码中的内存分配和释放操作，确保每次分配内存后都能正确释放，避免内存泄漏。特别注意在循环中分配内存而未释放的情况，以及在异常情况下未释放内存的情况。
4. 分析堆栈和内存转储：在发生内存泄漏或内存使用异常时，可以通过分析进程的堆栈信息和内存转储（core dump）来定位问题。堆栈信息可以帮助你找到内存分配和释放的位置，而内存转储可以提供进程崩溃时的内存状态，有助于定位问题的根本原因。
5. 使用代码审查和静态分析工具：进行代码审查可以帮助发现潜在的内存泄漏和内存使用问题。另外，一些静态分析工具可以帮助检测代码中可能存在的内存问题，提前发现潜在的风险。

### 4、定位到一个进程的内存比较异常，该如何进一步查找为什么内存会异常？

1. 使用内存分析工具进行检测：使用专门的内存分析工具（如Valgrind、AddressSanitizer等）对进程进行内存分析，以检测是否存在内存泄漏或者内存错误。这些工具可以提供详细的内存使用情况报告，帮助定位问题的具体原因。
2. 检查内存分配和释放的逻辑：仔细检查代码中的内存分配和释放逻辑，确保每次分配的内存都能够正确释放。特别关注循环中的内存分配和释放，以及异常情况下的内存处理逻辑。
3. 分析内存使用模式：观察进程的内存使用模式，包括内存的分配和释放情况、内存占用的变化趋势等。通过分析内存使用模式，可以发现是否存在内存泄漏或者内存占用异常的情况。
4. 检查第三方库和系统调用：如果进程使用了第三方库或者系统调用，需要检查这些部分是否存在内存使用不当的情况。有些第三方库可能存在内存泄漏或者内存占用过高的问题，需要特别注意。
5. 分析核心转储（core dump）：如果进程发生了崩溃，可以分析核心转储文件以获取进程崩溃时的内存状态。核心转储文件包含了进程崩溃时的内存快照，可以帮助定位问题的根本原因。
6. 使用日志和调试信息：在代码中添加日志和调试信息，记录内存分配和释放的情况，以及内存使用过程中的关键参数和状态。这些信息可以帮助你更好地理解进程的内存使用情况，并定位问题。

### 5、LeetCode460 LFU缓存

问题描述：

请你为 最不经常使用（LFU）缓存算法缓存算法设计并实现数据结构。

实现 `LFUCache` 类：

- `LFUCache(int capacity)` - 用数据结构的容量 `capacity` 初始化对象
- `int get(int key)` - 如果键 `key` 存在于缓存中，则获取键的值，否则返回 `-1` 。
- `void put(int key, int value)` - 如果键 `key` 已存在，则变更其值；如果键不存在，请插入键值对。当缓存达到其容量 `capacity` 时，则应该在插入新项之前，移除最不经常使用的项。在此问题中，当存在平局（即两个或更多个键具有相同使用频率）时，应该去除 **最久未使用** 的键。

为了确定最不常使用的键，可以为缓存中的每个键维护一个 **使用计数器** 。使用计数最小的键是最久未使用的键。

当一个键首次插入到缓存中时，它的使用计数器被设置为 `1` (由于 put 操作)。对缓存中的键执行 `get` 或 `put` 操作，使用计数器的值将会递增。

函数 `get` 和 `put` 必须以 `O(1)` 的平均时间复杂度运行。

思路：

1. 使用哈希表 `keyMap` 存储键值对，键是键值对的键，值是指向双向链表中节点的指针。
2. 使用哈希表 `freqMap` 存储键的使用频率，键是键值对的键，值是该键的使用频率。
3. 使用哈希表 `freqList` 存储每个频率对应的双向链表，键是使用频率，值是对应的双向链表头节点。
4. 使用变量 `minFreq` 记录当前最小的使用频率，用于删除最不经常使用的

参考代码：

```c++
#include <unordered_map>
#include <list>

using namespace std;

class LFUCache {
private:
    int capacity;
    int minFreq;
    unordered_map<int, pair<int, int>> keyMap; // 键值对的哈希表 key -> (value, freq)
    unordered_map<int, list<int>::iterator> keyPos; // 键在 freqList 中的位置
    unordered_map<int, list<int>> freqList; // 频率对应的双向链表 freq -> list of keys with this freq
    unordered_map<int, int> freqMap; // 键的使用频率 key -> freq

public:
    LFUCache(int capacity) {
        this->capacity = capacity;
        minFreq = 0;
    }

    int get(int key) {
        if (keyMap.find(key) == keyMap.end()) {
            return -1;
        }

        int value = keyMap[key].first;
        int freq = keyMap[key].second;
        freqList[freq].erase(keyPos[key]); // 从旧频率链表中删除
        freqMap[key]++;
        freqList[freqMap[key]].push_back(key); // 添加到新频率链表
        keyPos[key] = --freqList[freqMap[key]].end(); // 更新在 freqList 中的位置

        if (freqList[minFreq].empty()) {
            minFreq++;
        }

        return value;
    }

    void put(int key, int value) {
        if (capacity <= 0) {
            return;
        }

        if (keyMap.find(key) != keyMap.end()) {
            keyMap[key].first = value; // 如果键存在，更新值
            get(key); // 更新频率
            return;
        }

        if (keyMap.size() >= capacity) {
            int oldKey = freqList[minFreq].front(); // 移除最不经常使用的键
            freqList[minFreq].pop_front();
            keyMap.erase(oldKey);
            keyPos.erase(oldKey);
            freqMap.erase(oldKey);
        }

        minFreq = 1; // 重置 minFreq
        keyMap[key] = {value, 1};
        freqMap[key] = 1;
        freqList[1].push_back(key);
        keyPos[key] = --freqList[1].end();
    }
};

int main() {
    // 创建一个容量为 2 的 LFU 缓存
    LFUCache lfu(2);
    
    // 执行 put 操作
    lfu.put(1, 1);   // cache=[1,_], cnt(1)=1
    lfu.put(2, 2);   // cache=[2,1], cnt(2)=1, cnt(1)=1
    
    // 执行 get 操作
    int value1 = lfu.get(1); // 返回 1，cache=[1,2], cnt(2)=1, cnt(1)=2
    
    // 执行 put 操作
    lfu.put(3, 3);   // 去除键 2 ，因为 cnt(2)=1 ，使用计数最小
                     // cache=[3,1], cnt(3)=1, cnt(1)=2
    
    // 执行 get 操作
    int value2 = lfu.get(2); // 返回 -1（未找到）
    
    // 执行 get 操作
    int value3 = lfu.get(3); // 返回 3，cache=[3,1], cnt(3)=2, cnt(1)=2
    
    // 执行 put 操作
    lfu.put(4, 4);   // 去除键 1 ，1 和 3 的 cnt 相同，但 1 最久未使用
                     // cache=[4,3], cnt(4)=1, cnt(3)=2
    
    // 执行 get 操作
    int value4 = lfu.get(1); // 返回 -1（未找到）
    
    // 执行 get 操作
    int value5 = lfu.get(3); // 返回 3，cache=[3,4], cnt(4)=1, cnt(3)=3
    
    // 执行 get 操作
    int value6 = lfu.get(4); // 返回 4，cache=[3,4], cnt(4)=2, cnt(3)=3
    
    return 0;
}
```





1. 安装 Docker：首先，需要在所有节点上安装 Docker，Kubernetes 默认使用 Docker 作为容器运行时。可以使用以下命令安装 Docker：

   ```bash
   sudo apt-get update
   sudo apt-get install -y docker.io
   ```

2. 安装 kubeadm、kubelet 和 kubectl：在所有节点上安装 Kubernetes 控制平面组件 kubeadm、kubelet 和命令行工具 kubectl。可以使用以下命令安装：

   ```bash
   sudo apt-get update
   sudo apt-get install -y apt-transport-https ca-certificates curl
   curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
   sudo add-apt-repository "deb https://apt.kubernetes.io/ kubernetes-xenial main"
   sudo apt-get update
   sudo apt-get install -y kubeadm kubelet kubectl
   sudo apt-mark hold kubeadm kubelet kubectl
   ```

3. 初始化 Master 节点：选择一个节点作为 Master 节点，然后使用 `kubeadm init` 命令初始化 Kubernetes 控制平面。例如：

   ```bash
   sudo kubeadm init --pod-network-cidr=10.244.0.0/16
   ```

   这个命令会下载必要的容器镜像并启动 Kubernetes 控制平面组件。在初始化完成后，会输出一个类似的命令，用于在其他节点上加入集群。

4. 配置 kubectl：在 Master 节点上，运行以下命令配置 kubectl，以便将其连接到 Kubernetes 集群：

   ```bash
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```

5. 加入 Worker 节点（可选）：如果你有额外的节点作为 Worker 节点，可以使用之前 `kubeadm init` 命令输出的加入命令将它们加入集群。例如：

   ```bash
   sudo kubeadm join <Master_IP>:<Master_Port> --token <Token> --discovery-token-ca-cert-hash <Hash>
   ```

6. 安装网络插件（可选）：如果你需要网络插件来为 Pod 提供网络功能，可以选择安装一个网络插件。例如，安装 Calico 网络插件：

   ```bash
   kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
   ```

7. 验证集群状态：使用以下命令验证 Kubernetes 集群的状态：

   ```bash
   kubectl get nodes
   ```

   如果一切正常，你应该会看到 Master 节点和（如果有的话）Worker 节点的状态为 Ready。