# 华为od--C++面经

> 来源：https://www.nowcoder.com/discuss/580863971067019264

## 一面(50min)

## 项目

### 讲一下网络通信框架的原理及实现

## c++

### 1、说一下继承和多态？

1、继承：继承是指一个类（称为子类或派生类）可以继承另一个类（称为父类或基类）的属性和方法。子类可以访问父类的非私有成员（即公有成员和保护成员），并且可以在自己的定义中添加新的成员或重写父类的成员函数。继承可以实现代码的重用和扩展，使得代码更易于维护和理解。

```c++
class Animal {
public:
    void eat() {
        std::cout << "Animal is eating" << std::endl;
    }
};

class Dog : public Animal {
public:
    void bark() {
        std::cout << "Dog is barking" << std::endl;
    }
};

int main() {
    Dog dog;
    dog.eat();  // 继承了父类的eat()方法
    dog.bark();
    return 0;
}
```

2、多态：多态是指在运行时根据对象的实际类型来调用相应的方法，而不是根据引用或指针的静态类型。多态性是指同一个操作可以作用于不同类型的对象，并且可以根据对象的类型执行不同的行为。主要通过虚函数和函数重载实现。

- 编译时多态性（静态多态性）： 通过函数重载实现，编译器在编译时根据函数参数的类型和数量来选择调用合适的函数。这种多态性是在编译时解析的。
- 运行时多态性（动态多态性）： 通过虚函数和继承实现，允许在运行时根据对象的实际类型来调用适当的函数。这种多态性是在运行时解析的。

虚函数可以在父类中被声明为虚函数，子类可以根据需要覆盖（重写）这些虚函数。当通过父类的指针或引用调用虚函数时，会根据实际对象的类型来决定调用哪个版本的虚函数，从而实现多态。

```c++
class Animal {
public:
    virtual void eat() {
        std::cout << "Animal is eating" << std::endl;
    }
};

class Dog : public Animal {
public:
    void eat() override {
        std::cout << "Dog is eating" << std::endl;
    }
};

int main() {
    Animal* animal = new Dog();
    animal->eat();  // 实际调用的是Dog类的eat()方法
    delete animal;
    return 0;
}
```

### 2、说一下你常用的STL容器？

1. vector：动态数组，支持随机访问和尾部插入删除操作。它的底层实现是一块连续的内存空间，可以快速访问任意位置的元素。
2. list：双向链表，支持在任意位置进行插入删除操作，但不支持随机访问。由于链表的结构，插入和删除操作的时间复杂度为O(1)，但访问元素的时间复杂度为O(n)。
3. deque：双端队列，支持在队头和队尾进行插入删除操作，也支持随机访问。它的底层实现通常是一系列连续的内存块，可以在常数时间内对头部和尾部进行插入删除操作，但随机访问的性能略低于vector。
4. set：集合，内部使用红黑树实现，元素自动排序且唯一。插入、删除和查找操作的平均时间复杂度为O(log n)。
5. map：映射，内部使用红黑树实现，键值对自动排序且键唯一。插入、删除和查找操作的平均时间复杂度为O(log n)。
6. unordered_set：无序集合，内部使用哈希表实现，元素无序且唯一。插入、删除和查找操作的平均时间复杂度为O(1)。
7. unordered_map：无序映射，内部使用哈希表实现，键值对无序且键唯一。插入、删除和查找操作的平均时间复杂度为O(1)。

### 3、说说vector和list的区别？

1. 内部数据结构：
   - `vector`：使用动态数组作为内部数据结构，支持随机访问，即可以通过索引快速访问任意元素。插入和删除元素可能需要移动后续元素，因此在插入和删除操作上性能相对较低。
   - `list`：使用双向链表作为内部数据结构，插入和删除元素的性能很高，因为只需要调整节点的指针，而不需要移动元素。但是不支持随机访问，只能通过迭代器逐个遍历元素。
2. 空间复杂度：
   - `vector`：由于使用动态数组，可能会分配比实际元素数量更多的内存，因此空间复杂度相对高一些。
   - `list`：每个元素都需要额外的链表节点，因此通常占用的内存比`vector`更多。
3. 插入和删除操作：
   - `vector`：插入和删除元素的性能相对较差，特别是在中间位置，因为需要移动后续元素。
   - `list`：插入和删除元素的性能非常高，因为只需要调整节点的指针，不需要移动元素。
4. 随机访问：
   - `vector`：支持随机访问，可以通过索引快速访问元素。
   - `list`：不支持随机访问，只能通过迭代器逐个遍历元素。
5. 迭代器稳定性：
   - `vector`：在不发生插入和删除操作的情况下，迭代器的稳定性较好，可以一直有效。
   - `list`：在不发生删除当前元素的情况下，迭代器的稳定性较好。但如果在迭代过程中删除了当前元素，那么使用被删除元素的迭代器会导致未定义行为。
6. 适用场景：
   - 使用`vector`当需要快速随机访问元素，并且不经常进行插入和删除操作时。
   - 使用`list`当需要频繁进行插入和删除操作，而随机访问操作不是主要需求时。

## 八股

### TCP和UDP的区别及其应用？

1. 连接性：
   - TCP是面向连接的协议，建立连接需要经过三次握手，通信双方在传输数据前需要先建立连接，然后进行可靠的数据传输，最后释放连接。
   - UDP是无连接的协议，通信双方无需建立连接，直接发送数据包。因此，UDP的通信速度比TCP快，但不可靠。
2. 可靠性：
   - TCP提供可靠的数据传输，通过序列号、确认和重传机制保证数据的可靠性，可以保证数据的顺序性和完整性。
   - UDP不保证数据的可靠性，数据包可能丢失、重复或乱序，不提供重传机制。
3. 数据量：
   - TCP适用于大量数据传输，对数据的大小和频率没有限制，适合对数据完整性要求高的场景，如文件传输、网页访问等。
   - UDP适用于少量数据的传输，对数据的大小和频率有一定限制，适合对实时性要求高的场景，如视频、音频流传输、在线游戏等。
4. 应用场景：
   - TCP常用于需要可靠性和顺序性的场景，如HTTP、HTTPS、FTP等应用层协议。
   - UDP常用于实时性要求高、数据量小且可以容忍丢包的场景，如实时视频、音频传输、在线游戏等。

## 算法

### 1、快排和归并排序，常见排序算法的应用场景？

#### 快排

**快排逻辑**：

1. 从数组中选择一个基准元素。这个选择可以影响快速排序的性能。通常，选择第一个元素、最后一个元素或随机元素作为基准元素。
2. 将数组中的其他元素与基准元素进行比较，将小于基准元素的元素放在其左边，大于基准元素的元素放在其右边。这一过程称为分割或划分。
3. 对左右两个分割后的子数组进行递归排序。重复上述过程，直到子数组的大小为1或0，因为已经是有序的。
4. 递归排序完成后，所有子数组已经有序，最后合并这些子数组。

**时间复杂度**：

- 平均情况下，快速排序的时间复杂度为O(n * log n)。
- 在最坏情况下，当每次选择的基准元素都是数组中的最小或最大元素时，时间复杂度可能达到O(n^2)。为了避免最坏情况，可以采用随机选择基准元素的策略。

**空间复杂度**：

快速排序的主要消耗是递归调用的栈空间，因此空间复杂度取决于递归的深度。在最坏情况下，空间复杂度为O(n)，在平均情况下通常是O(log n)。

**稳定性**：

快速排序通常是不稳定的，因为相等元素的相对顺序可能会在排序过程中改变。例如，如果有两个相等的元素，快速排序可能会交换它们的位置。

**示例代码**：

```C++
#include <iostream>
#include <vector>
#include <stack>

// 交换两个元素
void swap(int& a, int& b) {
    int temp = a;
    a = b;
    b = temp;
}

// 分割函数，返回基准元素的索引
int partition(std::vector<int>& arr, int low, int high) {
    int pivot = arr[high];  // 选择最后一个元素作为基准
    int i = (low - 1);     // 初始化较小元素的索引

    for (int j = low; j <= high - 1; j++) {
        if (arr[j] < pivot) {
            i++;
            swap(arr[i], arr[j]);
        }
    }
    swap(arr[i + 1], arr[high]);
    return (i + 1);
}

// 非递归快速排序
void quickSort(std::vector<int>& arr, int low, int high) {
    std::stack<int> stack;
    stack.push(low);
    stack.push(high);

    while (!stack.empty()) {
        high = stack.top();
        stack.pop();
        low = stack.top();
        stack.pop();

        int pivotIndex = partition(arr, low, high);

        if (pivotIndex - 1 > low) {
            stack.push(low);
            stack.push(pivotIndex - 1);
        }

        if (pivotIndex + 1 < high) {
            stack.push(pivotIndex + 1);
            stack.push(high);
        }
    }
}

int main() {
    std::vector<int> arr = {12, 4, 5, 6, 7, 3, 1, 15, 2, 8};
    int arrSize = arr.size();

    std::cout << "Original array: ";
    for (int i = 0; i < arrSize; i++) {
        std::cout << arr[i] << " ";
    }
    std::cout << std::endl;

    quickSort(arr, 0, arrSize - 1);

    std::cout << "Sorted array: ";
    for (int i = 0; i < arrSize; i++) {
        std::cout << arr[i] << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

#### 归并排序

归并排序（Merge Sort）是一种经典的分治算法，它的基本思想是将原始数组递归地分割成更小的子数组，然后将这些子数组归并（合并）成一个有序数组。

具体步骤如下：

1. 分割：将待排序的数组递归地分割成两个子数组，直到每个子数组只包含一个元素为止。
2. 归并：将两个有序的子数组合并成一个有序数组。合并过程中，比较两个子数组的第一个元素，将较小的元素放入临时数组中，然后移动相应的指针，继续比较下一个元素，直到其中一个子数组的元素全部放入临时数组中，再将另一个子数组的剩余元素依次放入临时数组中。
3. 复制：将临时数组中的元素复制回原始数组的对应位置，完成一次归并。
4. 递归：对两个子数组分别进行递归的归并排序，直到所有子数组都只包含一个元素为止。

归并排序是一种稳定的排序算法，其时间复杂度为O(nlogn)，空间复杂度为O(n)。由于归并排序是一种分治算法，因此其适用于大规模数据的排序，且具有良好的稳定性和可靠性。

```c++
#include <iostream>
#include <vector>

// 合并两个有序子数组
void merge(std::vector<int>& arr, int left, int mid, int right) {
    int n1 = mid - left + 1;  // 计算左子数组的长度
    int n2 = right - mid;     // 计算右子数组的长度

    std::vector<int> L(n1), R(n2);  // 创建临时数组存储左右子数组

    // 将数据复制到临时数组中
    for (int i = 0; i < n1; i++)
        L[i] = arr[left + i];
    for (int j = 0; j < n2; j++)
        R[j] = arr[mid + 1 + j];

    int i = 0, j = 0, k = left;
    // 合并两个子数组
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

    // 将剩余的元素复制回原始数组
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

// 归并排序
void mergeSort(std::vector<int>& arr, int left, int right) {
    if (left < right) {
        int mid = left + (right - left) / 2;
        mergeSort(arr, left, mid);       // 对左半部分进行归并排序
        mergeSort(arr, mid + 1, right);  // 对右半部分进行归并排序
        merge(arr, left, mid, right);    // 合并两个有序子数组
    }
}

int main() {
    std::vector<int> arr = {12, 11, 13, 5, 6, 7};
    int n = arr.size();

    mergeSort(arr, 0, n - 1);  // 对整个数组进行归并排序

    std::cout << "Sorted array is: ";
    for (int i = 0; i < n; i++)
        std::cout << arr[i] << " ";
    std::cout << std::endl;

    return 0;
}
```

#### 使用场景

1. 冒泡排序：适用于数据量较小的情况，实现简单，但效率较低。在实际应用中很少使用。
2. 选择排序：适用于数据量较小且对稳定性没有要求的情况。由于其不稳定的特性，不适合用于排序对象。
3. 插入排序：适用于数据量较小且基本有序的情况。由于其稳定性和对部分有序数据的高效处理，常用于对小规模数据进行排序。
4. 归并排序：适用于任意规模的数据，稳定性好，但空间复杂度较高。常用于外部排序或对大规模数据的排序。
5. 快速排序：适用于任意规模的数据，性能优越，但对初始数据的分布较敏感。常用于内部排序、大规模数据的排序和快速查找。
6. 堆排序：适用于大规模数据的排序，实现简单，但不稳定。常用于内部排序和优先队列的实现。

### 2、LC394字符串解码手撕？

#### 思路

1. 定义两个辅助栈 `nums` 和 `strs`，一个用于存储重复次数，一个用于存储待解码的字符串。
2. 遍历输入字符串 `s` 中的每个字符。
3. 对于每个字符 `ch`：
   - 如果 `ch` 是数字，则表示重复次数，将其转换为数字并累加到 `num` 变量中。
   - 如果 `ch` 是字母，则直接将其加入当前待解码的字符串 `curr` 中。
   - 如果 `ch` 是左括号 `[`，表示需要进行解码操作，此时需要将当前的重复次数 `num` 入栈，并将当前待解码的字符串 `curr` 入栈，并重置 `num` 和 `curr`。
   - 如果 `ch` 是右括号 `]`，表示需要进行重复操作，此时需要从 `nums` 栈顶获取重复次数，并从 `strs` 栈顶获取待解码的字符串，进行重复后更新 `curr`。
4. 最终返回 `curr`，即为解码后的字符串。

#### 参考代码

```c++
#include <iostream>
#include <stack>
#include <string>

class Solution {
public:
    std::string decodeString(std::string s) {
        std::stack<int> nums;  // 存储重复次数
        std::stack<std::string> strs;  // 存储待解码的字符串
        std::string curr;  // 用于临时存储当前待解码的字符串
        int num = 0;  // 用于存储重复次数

        for (char ch : s) {
            if (isdigit(ch)) {
                num = num * 10 + (ch - '0');  // 计算重复次数
            } else if (isalpha(ch)) {
                curr += ch;  // 字母直接加入当前字符串
            } else if (ch == '[') {
                nums.push(num);  // 重复次数入栈
                num = 0;  // 重置重复次数
                strs.push(curr);  // 当前字符串入栈
                curr = "";  // 重置当前字符串
            } else if (ch == ']') {
                int repeatTimes = nums.top();  // 获取重复次数
                nums.pop();  // 弹出重复次数
                std::string temp = strs.top();  // 获取待解码的字符串
                strs.pop();  // 弹出待解码的字符串
                for (int i = 0; i < repeatTimes; ++i) {
                    temp += curr;  // 重复当前字符串
                }
                curr = temp;  // 更新当前字符串
            }
        }

        return curr;  // 返回最终解码结果
    }
};

int main() {
    Solution solution;
    std::string s = "3[a2[c]]";
    std::string decodedString = solution.decodeString(s);
    std::cout << "Decoded string: " << decodedString << std::endl;
    return 0;
}
```

## 二面(1h)

## 项目

### 学校导师带着做的还是自学的，独立完成的吗

## C++

### 1、C++面向对象的三大特性？

1. 封装（Encapsulation）：

   封装是指将数据和操作数据的方法封装在一个类中，隐藏类的内部实现细节，只对外部暴露必要的接口。通过封装，可以实现信息隐藏和保护数据的安全性，使得类的实现细节对外部用户透明，用户只需关注类的接口和功能。

2. 继承（Inheritance）：

   继承是指一个类（称为子类或派生类）可以继承另一个类（称为父类或基类）的属性和方法。子类可以使用父类的成员变量和成员函数，并且可以在此基础上扩展新的属性和方法。通过继承，可以实现代码的重用性和层次化设计，提高代码的可维护性和扩展性。

3. 多态（Polymorphism）：

   多态是指同一种操作作用于不同的对象上时，可以产生不同的行为。在C++中，多态可以通过虚函数（virtual function）和函数重载（function overloading）来实现。虚函数允许子类重写父类的方法，实现运行时多态性；函数重载允许同名函数根据参数类型或个数的不同而具有不同的行为，实现编译时多态性。多态性可以提高代码的灵活性和可扩展性，使得程序更加易于维护和理解。

### 2、C++的多态what, why？

多态：多态是指在运行时根据对象的实际类型来调用相应的方法，而不是根据引用或指针的静态类型。多态性是指同一个操作可以作用于不同类型的对象，并且可以根据对象的类型执行不同的行为。主要通过虚函数和函数重载实现。

- 编译时多态性（静态多态性）： 通过函数重载实现，编译器在编译时根据函数参数的类型和数量来选择调用合适的函数。这种多态性是在编译时解析的。
- 运行时多态性（动态多态性）： 通过虚函数和继承实现，允许在运行时根据对象的实际类型来调用适当的函数。这种多态性是在运行时解析的。

虚函数可以在父类中被声明为虚函数，子类可以根据需要覆盖（重写）这些虚函数。当通过父类的指针或引用调用虚函数时，会根据实际对象的类型来决定调用哪个版本的虚函数，从而实现多态。

```c++
class Animal {
public:
    virtual void eat() {
        std::cout << "Animal is eating" << std::endl;
    }
};

class Dog : public Animal {
public:
    void eat() override {
        std::cout << "Dog is eating" << std::endl;
    }
};

int main() {
    Animal* animal = new Dog();
    animal->eat();  // 实际调用的是Dog类的eat()方法
    delete animal;
    return 0;
}
```

### 3、重载和重写(覆盖)的区别？

1. 重载（Overloading）：

   - 定义：在同一个作用域内，允许多个函数或方法拥有相同的名称，但参数列表不同（包括参数类型、个数或顺序）。

   - 特点：

     - 重载的函数或方法在编译时根据调用时的参数类型和个数确定具体调用哪个版本，即静态多态（编译期决议）。

     - 可以包括同名函数或方法的不同版本，例如：

       ```c++
       int add(int a, int b);
       float add(float a, float b);
       ```

   - 重载不涉及父子类关系，只是在同一个类中或者同一作用域内进行名称相同但参数列表不同的函数或方法定义。

2. 重写（Override）：

   - 定义：子类重新定义（覆盖）了父类的虚函数（virtual function），并且参数列表和返回类型必须与父类的虚函数完全相同。
   - 特点：
     - 重写发生在父类和子类之间，子类重新定义了父类的虚函数，具有继承关系。
     - 当通过父类指针或引用调用虚函数时，会根据实际对象的类型来确定调用的是子类的函数还是父类的函数，即动态多态（运行期决议）。
     - 重写可以实现子类对父类行为的改变和扩展。

总结：

- 重载和重写都是实现多态的手段，但针对的对象和场景不同。
- 重载是在一个类中或者同一作用域内实现同名函数或方法的多个版本，根据参数列表的不同进行区分。
- 重写是子类重新定义了父类的虚函数，实现了继承时的多态性。

### 4、你常用的STL容器？

### 5、半圆形继承(多继承)，ABC是半圆形继承的关系并且AB都有成员函数D，C成员调用D时优先访问谁的？

## 八股

### 1、7层网络模型，每层举个协议的例子？

1. 物理层：
   - 功能：负责传输比特流，定义传输媒介的物理特性。
   - 例子：Ethernet、Wi-Fi、光纤传输等。
2. 数据链路层：
   - 功能：在相邻节点间传输数据帧，进行错误检测和纠正。
   - 例子：Ethernet（IEEE 802.3）、Wi-Fi（IEEE 802.11）、PPP（Point-to-Point Protocol）。
3. 网络层：
   - 功能：负责在不同网络间进行路由选择和数据传输。
   - 例子：IP（Internet Protocol）、IPv4、IPv6。
4. 传输层：
   - 功能：提供端到端的数据传输服务，确保数据可靠性和有序性。
   - 例子：TCP（Transmission Control Protocol）、UDP（User Datagram Protocol）。
5. 会话层：
   - 功能：建立、管理和终止会话连接，控制数据交换的方式和顺序。
   - 例子：NetBIOS（Network Basic Input/Output System）。
6. 表示层：
   - 功能：处理数据的格式转换、加密和解密，确保数据在不同系统间的兼容性。
   - 例子：ASCII（American Standard Code for Information Interchange）、JPEG、GIF。
7. 应用层：
   - 功能：提供用户应用程序访问网络的接口，实现特定的网络服务。
   - 例子：HTTP（Hypertext Transfer Protocol）、FTP（File Transfer Protocol）、SMTP（Simple Mail Transfer Protocol）。

### 2、你常用的同步机制，临界区如何用的？

1. 条件变量（Condition Variable）：

   - 功能：条件变量用于线程间的通信，当一个线程需要等待某个条件满足时可以进入等待状态，直到其他线程通知条件满足并唤醒它。
   - 使用方法：使用 `wait()` 方法使线程进入等待状态，使用 `notify_one()` 或 `notify_all()` 方法通知其他线程条件已满足。

   示例代码：

   ```c++
   #include <iostream>
   #include <thread>
   #include <mutex>
   #include <condition_variable>
   
   std::mutex mtx;  // 创建互斥锁
   std::condition_variable cv;  // 创建条件变量
   
   void worker() {
       std::unique_lock<std::mutex> lock(mtx);
       cv.wait(lock);  // 等待条件满足
       std::cout << "Condition met, thread wakes up" << std::endl;
   }
   
   int main() {
       std::thread t(worker);
       std::this_thread::sleep_for(std::chrono::seconds(1));
       cv.notify_one();  // 唤醒等待的线程
       t.join();
       return 0;
   }
   ```

2. 信号量（Semaphore）：

   - 功能：信号量用于控制对一定数量的资源的访问，可以用来解决生产者-消费者问题等并发场景。
   - 使用方法：使用 `wait()` 方法获取资源，使用 `post()` 方法释放资源。

   示例代码：

   ```c++
   #include <iostream>
   #include <thread>
   #include <mutex>
   #include <condition_variable>
   
   std::mutex mtx;  // 创建互斥锁
   std::condition_variable cv;  // 创建条件变量
   int count = 0;  // 计数器
   
   void producer() {
       while (true) {
           std::unique_lock<std::mutex> lock(mtx);
           cv.wait(lock, [](){ return count < 10; });  // 等待资源可用
           count++;
           std::cout << "Produced, count = " << count << std::endl;
           cv.notify_all();  // 通知消费者
       }
   }
   
   void consumer() {
       while (true) {
           std::unique_lock<std::mutex> lock(mtx);
           cv.wait(lock, [](){ return count > 0; });  // 等待资源可用
           count--;
           std::cout << "Consumed, count = " << count << std::endl;
           cv.notify_all();  // 通知生产者
       }
   }
   
   int main() {
       std::thread t1(producer);
       std::thread t2(consumer);
       t1.join();
       t2.join();
       return 0;
   }
   ```

3. 原子操作（Atomic Operations）：

   - 功能：原子操作是一种不会被中断的操作，可以确保在多线程环境下对共享资源的原子访问，避免了竞态条件（Race Condition）。
   - 使用方法：使用 `std::atomic` 类模板包装需要原子操作的变量，可以使用原子操作函数或操作符来进行操作。

   示例代码：

   ```c++
   #include <iostream>
   #include <thread>
   #include <atomic>
   
   std::atomic<int> count(0);  // 原子变量
   
   void increment() {
       for (int i = 0; i < 1000000; ++i) {
           count++;  // 原子递增操作
       }
   }
   
   void decrement() {
       for (int i = 0; i < 1000000; ++i) {
           count--;  // 原子递减操作
       }
   }
   
   int main() {
       std::thread t1(increment);
       std::thread t2(decrement);
       t1.join();
       t2.join();
       std::cout << "Final count: " << count << std::endl;
       return 0;
   }
   ```

4. 临界区（Critical Section）：

- 临界区是一段代码，同时只能有一个线程执行，用于保护共享资源或临界区代码段。
- 临界区的实现可以使用互斥锁来保护，在进入临界区前先获取互斥锁，退出临界区时释放互斥锁，从而确保同一时间只有一个线程可以进入临界区。

5. 互斥锁（Mutex）：

- 互斥锁是一种同步机制，用于保护临界区或共享资源，确保同一时间只有一个线程可以访问。

- 在C++中，可以使用标准库中的 `std::mutex` 类来实现互斥锁。

- 互斥锁的基本使用步骤如下：

  - 创建一个 `std::mutex` 对象，用于保护临界区或共享资源。
  - 在进入临界区前，使用 `lock()` 方法获取互斥锁，表示进入临界区。
  - 在退出临界区时，使用 `unlock()` 方法释放互斥锁，表示退出临界区。

- 示例代码如下所示：

  ```c++
  #include <iostream>
  #include <thread>
  #include <mutex>
  
  std::mutex mtx;  // 创建互斥锁
  
  void criticalSection() {
      mtx.lock();  // 获取互斥锁
      std::cout << "Enter critical section" << std::endl;
      // 执行临界区代码
      std::cout << "Exit critical section" << std::endl;
      mtx.unlock();  // 释放互斥锁
  }
  
  int main() {
      std::thread t1(criticalSection);
      std::thread t2(criticalSection);
      t1.join();
      t2.join();
      return 0;
  }
  ```

在实际应用中，临界区的设计要注意以下几点：

- 临界区应该尽量小，只包含必要的共享资源访问代码。
- 临界区内不要包含可能导致阻塞的操作，避免出现死锁情况。
- 在退出临界区之前，确保所有异常情况都得到处理，以免资源泄露或其他问题。

### 3、三次握手过程？

在建立连接之前，Client处于CLOSED状态，而Server处于LISTEN的状态。

1. 第一次握手：客户端主动给服务端发送一个SYN报文，并携带自己的初始化序列号一起发送给服务端。此时客户端处于一个SYN_SEND的状态。
2. 第二次握手：服务端收到客户端发来的SYN报文之后，就会以自己的SYN报文作为应答，然后将自己的初始化序列号发送给客户端，并且会将客户端的初始化序列号+1作为自己的ACK值发送给客户端，以表示自己已经收到了客户端的SYN报文。此时服务端处于一个SYN_RECV的状态。
3. 第三次握手：客户端收到服务端发来的SYN报文之后，会把服务端的初始化序列号+1作为ACK值发送给服务端，用来表示自己已经收到了服务端发来的SYN报文。此时客户端处于一个ESTABLISHED的状态。

![三次握手](https://cdn.jsdelivr.net/gh/aqjsp/photos/202402110052745.png)

### 4、进程和线程的区别？

1. 定义：
   - 进程（Process）：是程序在执行过程中的一个实例，是操作系统资源分配的基本单位，每个进程有独立的内存空间。
   - 线程（Thread）：是进程中的一个执行单元，是CPU调度和分派的基本单位，同一进程内的多个线程共享相同的内存空间。
2. 资源分配：
   - 进程独享资源：每个进程有独立的内存空间、文件描述符、进程控制块等系统资源。
   - 线程共享资源：同一进程内的多个线程共享相同的内存空间和文件描述符等资源。
3. 创建和销毁：
   - 进程创建和销毁较为复杂：创建新进程需要分配独立的内存空间、建立进程控制块等，销毁进程需要释放资源并通知操作系统。
   - 线程创建和销毁较为简单：在同一进程内部创建新线程只需分配线程控制块即可，销毁线程也比较轻量级。
4. 调度和切换：
   - 进程调度和切换开销大：进程切换涉及到切换内存空间、文件描述符表等，开销较大。
   - 线程调度和切换开销小：线程切换只需切换线程控制块和栈即可，开销较小。
5. 通信方式：
   - 进程间通信复杂：需要使用进程间通信（IPC）机制，如管道、信号量、消息队列等。
   - 线程间通信简单：由于线程共享相同的内存空间，可以直接通过共享内存、全局变量等方式进行通信。
6. 并发性：
   - 进程并发性较差：由于进程间资源独立，进程之间的通信和同步需要较多的系统开销。
   - 线程并发性较好：由于线程共享相同的内存空间，线程之间的通信和同步更加方便快捷。

## 算法

### LC853车队、分析时间复杂度

#### 思路

1. 初始化变量：

   - 获取车辆数量 `n`，创建一个空的栈 `stack` 和一个空的向量 `times`，用于存储每辆车到达目的地的时间。
   - 创建一个元组列表 `cars`，用于存储每辆车的位置和速度。

2. 计算到达时间：

   遍历每辆车，计算它们到达目的地所需的时间，并将这些时间存储在 `times` 向量中。计算方法为：目的地距离除以速度。

3. 按位置排序：

   将车辆按照位置从大到小排序，这样排序后，位置越大的车辆越靠近目的地。

4. 遍历车辆：

   从第一辆车开始，逐个比较每辆车的到达时间和栈顶车辆的到达时间：

   - 如果当前车辆的到达时间小于栈顶车辆的到达时间，说明当前车辆可以追上栈顶车辆，不形成新车队，将当前车辆的到达时间压入栈中。
   - 如果当前车辆的到达时间大于或等于栈顶车辆的到达时间，说明当前车辆无法追上栈顶车辆，形成新车队，将栈顶车辆的到达时间弹出。

5. 返回车队数量：

   最终栈中存储的就是形成的车队，栈的大小即为车队的数量。

#### 参考代码

```c++
#include <vector>
#include <algorithm>
#include <iostream>

using namespace std;

class Solution {
public:
    int carFleet(int target, vector<int>& position, vector<int>& speed) {
        int n = position.size();
        vector<double> times(n);  // 存储到达目的地的时间
        vector<pair<int, int>> cars(n);  // 存储车辆的位置和速度
        for (int i = 0; i < n; ++i) {
            cars[i] = make_pair(position[i], speed[i]);
        }
        // 按照车辆位置从大到小排序
        sort(cars.begin(), cars.end(), greater<>());

        vector<double> stack;  // 用于存储车辆的到达时间
        for (int i = 0; i < n; ++i) {
            // 计算到达目的地的时间
            times[i] = (double)(target - cars[i].first) / cars[i].second;
        }
        for (int i = 0; i < n; ++i) {
            double m = times[i];
            // 如果前面有车辆的到达时间大于等于当前车辆，则当前车辆无法追上前面的车辆，形成新车队
            if (!stack.empty() && stack.back() >= times[i]) {
                m = stack.back();  // 更新到达时间为前一辆车的到达时间
                stack.pop_back();  // 弹出前一辆车的到达时间
            }
            stack.push_back(m);  // 将当前车辆的到达时间压入栈中
        }
        return stack.size();  // 返回车队数量
    }
};

int main() {
    Solution sol;
    vector<int> position = {10, 8, 0, 5, 3};
    vector<int> speed = {2, 4, 1, 1, 3};
    int target = 12;
    cout << sol.carFleet(target, position, speed) << endl;  // 输出：3
    return 0;
}
```

