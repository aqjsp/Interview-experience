来源：https://www.nowcoder.com/discuss/552166947891195904?sourceSSR=users

## 一面

### 1、指针和引用的区别? 

1. 定义：

   - 指针： 指针是一个包含变量地址的变量。通过指针，可以访问或修改存储在该地址上的值。
   - 引用： 引用是一个别名，它为一个已存在的变量提供了另一个名字。引用在创建时必须初始化，并且一旦初始化后，它将一直引用相同的对象。

2. 语法：

   - 指针： 使用 `*` 符号来声明指针，以及通过 `*` 来访问指针指向的值。

   - ```C++
     int x = 10;
     int *ptr = &x;  // 指针的声明和初始化int value = *ptr;  // 使用指针访问值
     ```

   - 引用： 使用 `&` 符号来声明引用，没有类似 `*` 的解引用符号。

   - ```C++
     int x = 10;
     int &ref = x;  // 引用的声明和初始化int value = ref;  // 直接使用引用访问值
     ```

3. 空值（NULL 或 nullptr）：

   - 指针： 可以是空值（nullptr 或 NULL），表示指针不指向任何有效的地址。
   - 引用： 引用必须在创建时初始化，并且不能为 null。

4. 地址操作：

   - 指针： 可以通过指针进行地址的算术操作，比如指针加法和减法。
   - 引用： 引用一旦初始化，不能改变引用的目标。

5. 多级间接引用：

   - 指针： 可以通过多级指针实现多级间接引用。
   - 引用： 引用本身不支持多级引用。

6. 数组：

   - 指针： 可以通过指针对数组进行遍历和操作。
   - 引用： 引用不直接支持数组的遍历，但可以通过指针和引用的结合来实现。

7. 传递给函数：

   - 指针： 通过指针可以实现函数的参数传递和返回。
   - 引用： 通过引用也可以实现函数的参数传递和返回，但语法上更简洁。

8. 使用场景：

   - 指针： 通常用于动态内存分配、数组操作、实现数据结构等。
   - 引用： 通常用于函数参数传递、返回引用值、以及在某些情况下取代指针使用。

### 2、面向对象？

关于 C++ 面向对象编程的一些基本概念：

1. 类和对象：

   - 类（Class）： 类是用户自定义的数据类型，用于封装数据和行为。类定义了一种抽象的数据类型，包括数据成员和成员函数。
   - 对象（Object）： 对象是类的实例，是具体的数据实体。通过类创建对象，对象包含了类中定义的数据和可以对数据执行的操作。

2. 封装：

   封装（Encapsulation）： 将类的实现细节隐藏起来，通过公有接口提供对类的访问。这样可以防止外部直接访问类的内部数据，只能通过定义的接口进行操作。

3. 继承：

   继承（Inheritance）： 允许一个类（子类/派生类）从另一个类（父类/基类）继承属性和行为。子类可以重用父类的成员，并且可以在其上添加新的成员或修改继承的成员。

4. 多态：

   多态（Polymorphism）： 多态允许使用一个基本类型的指针指向派生类型的对象，从而实现基类指针可以在运行时指向不同类型的对象。多态分为编译时多态（静态多态）和运行时多态（动态多态）。

5. 构造函数和析构函数：

   - 构造函数（Constructor）： 构造函数用于初始化对象的状态，在对象创建时自动调用。构造函数的名称与类名相同，没有返回类型。
   - 析构函数（Destructor）： 析构函数用于清理对象占用的资源，在对象销毁时自动调用。析构函数的名称是在类名前加上波浪号 `~`。

6. 成员函数和成员变量：

   - 成员函数（Member Function）： 类中定义的函数，可以操作对象的数据成员。成员函数可以是公有的、私有的或受保护的。
   - 成员变量（Member Variable）： 类中定义的数据成员，表示对象的状态。成员变量可以是公有的、私有的或受保护的。

7. 访问修饰符：

   - 公有（Public）： 成员在类的内部和外部均可访问。
   - 私有（Private）： 成员只能在类的内部访问。
   - 受保护（Protected）： 成员可以在类的内部和派生类中访问。

8. 静态成员：

   静态成员（Static Member）： 静态成员属于类而不是对象，对所有类的对象共享。静态成员可以是静态数据成员或静态成员函数。

9. 友元函数：

   友元函数（Friend Function）： 友元函数是不属于类成员函数的独立函数，但它可以访问类的私有成员。友元函数通过在类中声明并在类外定义来实现。

### 3、多态？

多态性是指同一个操作可以作用于不同类型的对象，并且可以根据对象的类型执行不同的行为。多态性通过虚函数和函数重载实现。

- 编译时多态性（静态多态性）： 通过函数重载实现，编译器在编译时根据函数参数的类型和数量来选择调用合适的函数。这种多态性是在编译时解析的。
- 运行时多态性（动态多态性）： 通过虚函数和继承实现，允许在运行时根据对象的实际类型来调用适当的函数。这种多态性是在运行时解析的。

### 4、继承的实现原理？

在C++中，继承的实现原理主要基于两个概念：基类（父类）和派生类（子类）。

1. 类的成员和成员函数继承：
   - 派生类继承了基类的成员变量和成员函数，包括公有、保护和私有的。
   - 派生类可以通过访问权限来限制对基类成员的访问。

```C++
class Base {
public:
    int publicVar;
protected:
    int protectedVar;
private:
    int privateVar;
};

class Derived : public Base {// Derived 类继承了 Base 类的 publicVar, protectedVar, 但不继承 privateVarpublic:void someFunction() {
        publicVar = 10;       // 可以访问基类的公有成员
        protectedVar = 20;    // 可以访问基类的保护成员// privateVar = 30;    // 无法访问基类的私有成员
    }
};
```

2. 构造函数和析构函数的调用：

   - 派生类的构造函数负责调用基类的构造函数，确保基类部分正确初始化。

   - 派生类的析构函数负责调用基类的析构函数，确保基类部分正确清理。

```C++
class Base {
public:Base() {// 构造函数的实现
    }
    ~Base() {// 析构函数的实现
    }
};

class Derived : public Base {
public:Derived() : Base() {// 派生类构造函数调用基类构造函数
    }
    ~Derived() {// 派生类析构函数调用基类析构函数
    }
};
```

3. 虚函数和多态性：

   - C++ 使用虚函数表（vtable）和虚函数指针来实现动态绑定，支持运行时多态性。

   - 虚函数在基类中声明为 `virtual`，并在派生类中重新定义。

```C++
class Base {
public:virtual void someFunction() {// 虚函数的默认实现
    }
};

class Derived : public Base {
public:void someFunction() override {// 派生类重新定义的虚函数
    }
};
```

4. 访问权限控制：

派生类可以通过访问权限控制来限制对基类成员的访问。`public` 继承保持原有的访问权限，而 `protected` 和 `private` 继承则改变访问权限。

```C++
class Base {
public:
    int publicVar;
protected:
    int protectedVar;
private:
    int privateVar;
};

class DerivedPublic : public Base {
    // publicVar 是 publi
    // protectedVar 是 protected
    // privateVar 是不可见的
};

class DerivedProtected : protected Base {
    // publicVar 是 protected
    // protectedVar 是 protected
    // privateVar 是不可见的
};

class DerivedPrivate : private Base {
    // publicVar 是 private
    // protectedVar 是 private
    // privateVar 是不可见的
};
```

### 5、STL？

STL 主要包含以下几个组件：

1. 容器（Containers）：

   容器是用来存储数据的数据结构。STL提供了多种容器，包括向量（`vector`）、链表（`list`）、双端队列（`deque`）、集合（`set`）、映射（`map`）、堆栈（`stack`）、队列（`queue`）等。每种容器都有其特定的特性和适用场景。

2. 算法（Algorithms）：

   算法包括了一系列常见的操作，例如排序、查找、遍历等。这些算法可以用于不同类型的容器，提供了一种统一的处理方式。使用算法，开发者可以不关心底层容器的具体实现，从而更加专注于问题的逻辑。

3. 迭代器（Iterators）：

   迭代器提供了一种访问容器元素的统一接口。通过迭代器，可以逐个遍历容器中的元素，使得算法可以适用于各种不同类型的容器。迭代器的设计模式是STL的核心之一。

4. 函数对象（Functors）：

   函数对象是可调用对象，它可以像函数一样被调用。STL中的算法通常可以接受函数对象作为参数，提供了更灵活的算法实现方式。函数对象可以通过重载函数调用运算符 `operator()` 来实现。

5. 适配器（Adapters）：

   适配器是一种用于改变容器或迭代器接口的工具。例如，栈和队列的适配器可以将其他容器转化为栈或队列的接口，迭代器的适配器可以改变迭代器的行为。

6. 空间配置器（Allocators）：

   空间配置器负责管理内存的分配和释放。STL中的容器在实现时通常使用了空间配置器，可以通过自定义空间配置器来满足特定的需求。

这里就不详细赘述各个模块具体的内容了。

### 6、map中使用自定义数据结构，需要做哪些工作？：重载运算符

1. 比较函数： `std::map` 默认使用 `<` 运算符进行键的比较。如果你的自定义数据结构作为 `map` 的键，确保实现了 `<` 运算符或提供了比较函数（函数对象），以便 `map` 能够正确比较键的大小。

```C++
struct CustomStruct {
    int key;
    // other members

    bool operator<(const CustomStruct& other) const {
        return key < other.key;
    }
};
```

2. 哈希函数（可选）： 如果你希望使用 `std::unordered_map`，那么你需要提供哈希函数。这个哈希函数可以帮助提高插入和查找的性能。

```C++
struct CustomStructHash {
    std::size_t operator()(const CustomStruct& obj) const {
        return std::hash<int>{}(obj.key);
    }
};
```

3. 自定义比较函数对象（可选）： 如果默认的 `<` 运算符或者自定义的比较函数不满足你的需求，你可以使用自定义的比较函数对象。

```C++
struct CustomCompare {
    bool operator()(const CustomStruct& a, const CustomStruct& b) const {
        // your custom comparison logic
    }
};
```

4. 初始化对象： 在使用自定义数据结构的实例作为 `map` 的键之前，确保这些实例已经被正确初始化。

5. 注意拷贝语义： 确保自定义数据结构正确支持拷贝构造函数和赋值运算符，因为`map` 在内部会进行键的拷贝。

```C++
struct CustomStruct {
    int key;
    // other members

    // copy constructor
    CustomStruct(const CustomStruct& other) : key(other.key) {
        // copy other members
    }

    // copy assignment operator
    CustomStruct& operator=(const CustomStruct& other) {
        if (this != &other) {
            key = other.key;
            // copy other members
        }
        return *this;
    }
};
```

6. 确保键是可比较的： 如果你在自定义数据结构中使用了其他自定义类型，确保这些类型也满足比较要求。

### 7、C++11？

1. 自动类型推导（Auto）： 允许编译器推导变量的类型，使代码更加简洁。

```C++
auto x = 5; // x的类型将被推导为int
```

2. 范围-based for 循环： 简化了对容器元素的遍历。

```C++
std::vector<int> numbers = {1, 2, 3, 4, 5};
for (const auto& num : numbers) {
    // 使用num
}
```

3. 智能指针： 引入了`std::shared_ptr`和`std::unique_ptr`等智能指针，用于管理动态分配的内存，帮助防止内存泄漏。

```C++
std::shared_ptr<int> sharedPtr = std::make_shared<int>(42);
```

4. Lambda 表达式： 允许在函数内部定义匿名函数，提高代码可读性和灵活性。

```C++
auto add = [](int a, int b) { return a + b; };
```

5. nullptr： 引入了空指针常量`nullptr`，用于替代传统的空指针`NULL`。

```C++
int* ptr = nullptr;
```

6. 强制类型转换（Type Casting）： 引入了`static_cast`、`dynamic_cast`、`const_cast`、`reinterpret_cast`等更安全和灵活的类型转换操作符。

```C++
double x = 3.14;
int y = static_cast<int>(x);
```

7. 右值引用和移动语义： 支持通过右值引用实现移动语义，提高了对临时对象的处理效率。

```C++
std::vector<int> getVector() {
    // 返回一个临时vector
    return std::vector<int>{1, 2, 3};
}

std::vector<int> numbers = getVector(); // 使用移动语义
```

8. 新的容器和算法： 引入了新的容器，如`std::unordered_map`、`std::unordered_set`，以及一些新的算法。

```C++
std::unordered_map<int, std::string> myMap = {{1, "one"}, {2, "two"}};
```

9. 线程支持（std::thread）： 提供了原生的多线程支持，使得并发编程更加方便。

```C++
#include <thread>

void myFunction() {
    // 线程执行的代码
}

int main() {
    std::thread t(myFunction);
    t.join(); // 等待线程结束
    return 0;
}
```

### 8、lambda表达式用过吗？实现原理？

Lambda表达式是C++11引入的一个特性，用于创建匿名函数。Lambda表达式的语法如下：

```C++
[capture](parameters) -> return_type {
    // 函数体
}
```

其中：

- `capture` 用于捕获外部变量，可以是值捕获（`[var]`）、引用捕获（`[&var]`）、混合捕获（`[var, &var]`）等。
- `parameters` 是函数参数列表。
- `return_type` 是返回类型。
- `->` 用于指定返回类型。
- `{}` 内为函数体。

Lambda表达式的实现原理可以简单概括如下：

1. 生成匿名类型： 编译器会生成一个匿名类型，用于存储Lambda表达式的数据成员。
2. 生成函数调用运算符： 编译器会为Lambda表达式生成一个函数调用运算符 `operator()` 的重载，这是Lambda表达式的实际执行体。
3. 捕获外部变量： 如果有捕获外部变量，编译器会生成匿名类型的成员变量，用于存储捕获的外部变量。

Lambda表达式的本质是一个可调用对象，可以像函数一样被调用。编译器为Lambda表达式生成的匿名类型使其具有函数对象的特性

### 9、引用计数知道吗？

引用计数是一种内存管理技术，用于跟踪指定内存块的引用次数。在引用计数中，每个被管理的对象都有一个与之相关联的计数器，用于记录当前有多少个指针指向该对象。当创建一个指向对象的指针时，引用计数加一；当销毁一个指向对象的指针时，引用计数减一。当引用计数降为零时，表示没有任何指针指向该对象，可以安全地释放该对象的内存。

引用计数的基本思想是在对象上维护一个计数器，通过增加和减少计数器的值来追踪对象的引用情况。引用计数有以下优点和缺点：

优点：

1. 简单：引用计数是一种简单直观的内存管理方式。
2. 实时性：当最后一个引用被销毁时，对象可以立即被释放。

缺点：

1. 循环引用：引用计数难以处理循环引用的情况，即两个或多个对象互相引用，导致计数永不为零，对象无法被释放。
2. 开销：每次引用计数的修改都需要额外的开销，可能导致性能损失。

在C++中，可以通过使用智能指针来实现引用计数。`std::shared_ptr` 就是一种使用引用计数的智能指针，它会在对象上维护一个计数器，通过增加和减少计数器的值来管理对象的生命周期。

### 10、类中的引用计数使用什么实现？

在C++中，类中引用计数通常使用智能指针来实现。其中，`std::shared_ptr` 是一种常见的智能指针，它使用引用计数来管理对象的生命周期。

下面是一个简单的示例，演示了如何在类中使用 `std::shared_ptr` 实现引用计数：

```C++
#include <iostream>
#include <memory>

class MyClass {
public:
    MyClass() {
        std::cout << "MyClass constructor" << std::endl;
    }

    ~MyClass() {
        std::cout << "MyClass destructor" << std::endl;
    }
};

class RefCountedClass {
public:
    RefCountedClass() : refCount(new int(0)), data(new MyClass()) {
        std::cout << "RefCountedClass constructor" << std::endl;
        (*refCount)++;
    }

    // 使用智能指针时，不再需要手动释放资源
    ~RefCountedClass() {
        std::cout << "RefCountedClass destructor" << std::endl;
    }

    // 复制构造函数，增加引用计数
    RefCountedClass(const RefCountedClass& other) : refCount(other.refCount), data(other.data) {
        std::cout << "RefCountedClass copy constructor" << std::endl;
        (*refCount)++;
    }

    // 赋值运算符重载，增加引用计数
    RefCountedClass& operator=(const RefCountedClass& other) {
        if (this != &other) {
            std::cout << "RefCountedClass copy assignment operator" << std::endl;
            refCount = other.refCount;
            data = other.data;
            (*refCount)++;
        }
        return *this;
    }

private:
    std::shared_ptr<int> refCount; // 引用计数
    std::shared_ptr<MyClass> data; // 实际数据
};

int main() {
    {
        RefCountedClass obj1; // 创建对象1，引用计数为1
        {
            RefCountedClass obj2 = obj1; // 创建对象2，引用计数增加为2
        } // 对象2 离开作用域，引用计数减少为1
    } // 对象1 离开作用域，引用计数减少为0，自动释放资源

    return 0;
}
```

这个示例中，`RefCountedClass` 类使用 `std::shared_ptr` 来管理 `MyClass` 类的实例。在构造函数中，引用计数初始化为1，每当对象被复制或赋值时，引用计数增加。当对象离开作用域时，引用计数减少。使用智能指针能够更方便地管理资源，避免手动释放内存的麻烦。

### 11、为什么要用指针？

1. 动态内存分配：指针允许动态地分配和释放内存，这是静态变量和普通成员变量无法做到的。动态内存分配使得在运行时根据需要分配内存，而不是在编译时确定。
2. 灵活性和多态性：指针支持多态性，允许使用基类指针指向派生类对象，从而实现多态。这对于实现抽象类和接口非常有用，使得代码更加灵活和可扩展。
3. 数据结构的灵活性：指针可以用于构建复杂的数据结构，如链表、树等。通过指针，可以轻松实现动态数据结构，而静态变量或普通成员变量通常需要在编译时确定大小。
4. 函数和回调：指针可用于传递函数或回调，实现函数指针。这在实现回调机制、事件处理等场景中非常有用。
5. 传递引用和修改实参：通过指针，可以传递变量的地址，使得函数能够修改传入的实际参数。这是通过引用传递和修改实参的一种手段。   

### 12、QT的object类作为所有控件的基类，做了哪些工作，发挥了什么作用？

1. 父子关系管理： `QObject` 支持对象之间的父子关系。当一个对象作为另一个对象的子对象时，它的生命周期将由父对象管理。当父对象销毁时，会自动销毁其所有子对象，从而简化内存管理。
2. 信号和槽系统： `QObject` 是信号和槽系统的基础。通过继承 `QObject`，对象可以使用 Qt 的信号和槽机制进行事件通信，实现松耦合的设计。这是 Qt 中实现事件驱动编程的核心机制。
3. 元对象系统（Meta-Object System）： Qt 使用元对象系统来实现一些高级功能，如动态属性、反射、信号和槽的动态连接等。`QObject` 提供了元对象宏（`Q_OBJECT`）和 `moc` 工具，用于生成元对象代码。
4. 动态属性和元数据： `QObject` 支持动态属性，允许在运行时向对象添加属性。这为对象的扩展和自定义提供了一种机制。同时，通过元对象系统，可以获取对象的元数据，包括类名、属性、方法等信息。
5. 对象名称： 每个 `QObject` 都可以有一个对象名称，通过 `QObject` 的 `objectName()` 方法获取。对象名称在对象查找和识别时很有用，特别是在图形界面编程中。
6. 线程安全： `QObject` 提供了一些线程安全的功能，如 `QObject::moveToThread()` 方法，可以将对象移动到另一个线程。这对于在多线程应用程序中管理对象非常有用。

### 13、算法题：整型转字符串？判断负号情况、传参数用引用。

```C++
#include <iostream>
#include <string>

std::string itoa(int num) {
    // 特殊情况处理：整数为 0
    if (num == 0) {
        return "0";
    }

    std::string result;  // 用于存储转换后的字符串
    bool isNegative = false;

    // 处理负数
    if (num < 0) {
        isNegative = true;
        num = -num;
    }

    // 逐位提取数字并添加到字符串中
    while (num > 0) {
        char digit = '0' + num % 10;  // 取得最低位的数字并转换为字符
        result = digit + result;       // 将字符添加到字符串的前面
        num /= 10;                     // 去掉已经处理的最低位
    }

    // 处理负号
    if (isNegative) {
        result = '-' + result;
    }

    return result;
}

int main() {
    // 测试
    int num1 = 123;
    int num2 = -456;
    int num3 = 0;

    std::cout << "itoa(" << num1 << ") = " << itoa(num1) << std::endl;
    std::cout << "itoa(" << num2 << ") = " << itoa(num2) << std::endl;
    std::cout << "itoa(" << num3 << ") = " << itoa(num3) << std::endl;

    return 0;
}
```

## 二面

### 1、Qt信号与槽机制原理？

1. 信号（Signal）：

   - 信号是一种特殊的成员函数，用于通知对象发生了某个特定的事件。
   - 信号由 `signals` 关键字声明，没有函数体，只是一个函数声明的形式。
   - 信号可以带有参数，参数的数量和类型是灵活的。

2. 槽（Slot）：

   - 槽是接收信号的特殊成员函数，用于响应特定的事件。
   - 槽由 `slots` 关键字声明，也是一个函数声明的形式。
   - 槽的参数要和信号的参数一致。

3. 连接（Connect）：

   - 信号与槽通过连接建立关联，这是通过 `QObject::connect` 函数完成的。
   - 当信号被发射时，与之连接的槽会被调用。

4. 元对象系统（Meta-Object System）：

   - Qt 使用元对象系统来实现信号与槽的机制。
   - 在编译阶段，Qt 的元对象编译器（MOC）会解析源代码，生成与信号和槽相关的元对象信息。
   - 运行时，这些元对象信息用于建立信号与槽的连接，实现动态的事件处理。

5. 线程安全：

   - Qt 的信号与槽机制支持多线程应用，可以在不同线程中发射信号和调用槽。
   - 在多线程环境下，Qt 提供了 `Qt::QueuedConnection` 连接类型，确保槽函数在接收信号的槽对象所属的线程中执行。

6. 自定义信号与槽：

   Qt 支持自定义信号和槽，使得用户可以定义自己的事件和处理逻辑。

给个例子：

Qt 中使用信号与槽：

```C++
#include <QObject>
#include <QTimer>
#include <QDebug>

class MyClass : public QObject {
    Q_OBJECT  // 宏，用于启用元对象系统支持

public slots:
    void handleTimeout() {
        qDebug() << "Timeout!";
    }
};

int main() {
    MyClass obj;

    // 创建定时器
    QTimer timer;
    
    // 连接定时器的 timeout 信号与 MyClass 对象的 handleTimeout 槽
    QObject::connect(&timer, SIGNAL(timeout()), &obj, SLOT(handleTimeout()));

    // 设置定时器间隔为 1000 毫秒
    timer.setInterval(1000);

    // 启动定时器
    timer.start();

    return 0;
}
```

### 2、排序算法？

1. **快速排序（Quick Sort）：**

- 思想： 通过一趟排序将待排序记录分割成独立的两部分，其中一部分的所有记录都比另一部分的记录小，然后递归地对两部分进行排序。
- C++ 示例：

```C++
#include <algorithm>

std::vector<int> arr = {4, 2, 7, 1, 9, 5};
std::sort(arr.begin(), arr.end()); // 使用 STL 的 std::sort
```

2. **归并排序（Merge Sort）：**

- 思想： 将待排序序列分成两个有序的子序列，再对子序列进行归并排序，最后合并这两个有序子序列。
- C++ 示例：

```C++
#include <algorithm>

std::vector<int> arr = {4, 2, 7, 1, 9, 5};
std::merge_sort(arr.begin(), arr.end()); // 自定义归并排序
```

3. **插入排序（Insertion Sort）：**

- 思想： 将一个记录插入到已经排好序的有序表中，从而得到一个新的、记录数增加1的有序表。
- C++ 示例：

```C++
#include <algorithm>

std::vector<int> arr = {4, 2, 7, 1, 9, 5};
std::insertion_sort(arr.begin(), arr.end()); // 自定义插入排序
```

4. **冒泡排序（Bubble Sort）：**

- 思想： 依次比较相邻的两个元素，将较大的元素交换到右侧，通过多次的遍历，将最大的元素移动到最右侧。
- C++ 示例：

```C++
#include <algorithm>

std::vector<int> arr = {4, 2, 7, 1, 9, 5};
std::bubble_sort(arr.begin(), arr.end()); // 自定义冒泡排序
```

5. **堆排序（Heap Sort）：**

- 思想： 利用堆这种数据结构进行排序。将待排序的序列构建成一个最大堆，然后将堆顶元素与末尾元素交换，再重新调整堆，如此循环直至整个序列有序。
- C++ 示例：

```C++
#include <algorithm>

std::vector<int> arr = {4, 2, 7, 1, 9, 5};
std::make_heap(arr.begin(), arr.end()); // 创建最大堆
std::sort_heap(arr.begin(), arr.end()); // 堆排序
```

6. **选择排序（Selection Sort）：**

- 思想： 每次从待排序的序列中选择最小（或最大）的元素，将其与序列的第一个元素交换，然后在剩下的元素中选择最小（或最大）的元素，与序列的第二个元素交换，如此循环。
- C++ 示例：

```C++
#include <algorithm>

std::vector<int> arr = {4, 2, 7, 1, 9, 5};
std::selection_sort(arr.begin(), arr.end()); // 自定义选择排序
```

7. **计数排序（Counting Sort）：**

- 思想： 统计待排序序列中每个元素出现的次数，然后根据统计信息将元素放回原序列的正确位置。
- C++ 示例：

```C++
#include <algorithm>

std::vector<int> arr = {4, 2, 7, 1, 9, 5};
std::counting_sort(arr.begin(), arr.end()); // 自定义计数排序
```

8. **桶排序（Bucket Sort）：**

- 思想： 将待排序序列划分为若干个桶，每个桶内部再分别排序，最后将各个桶的元素合并得到有序序列。
- C++ 示例：

```C++
#include <algorithm>

std::vector<int> arr = {4, 2, 7, 1, 9, 5};
std::bucket_sort(arr.begin(), arr.end()); // 自定义桶排序
```

9. **基数排序（Radix Sort）：**

- 思想： 将待排序序列按照每一位进行排序，从最低位到最高位，最终得到有序序列。
- C++ 示例：

```C++
#include <algorithm>

std::vector<int> arr = {4, 2, 7, 1, 9, 5};
std::radix_sort(arr.begin(), arr.end()); // 自定义基数排序
```

### 3、稳定的算法、不稳定的算法都有哪些？

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

### 4、快排稳定吗，选择排序稳定吗？

快速排序（Quick Sort）在一般情况下是不稳定的，因为它涉及到元素的交换操作。在排序的过程中，相同元素的相对位置可能会改变。

选择排序（Selection Sort）也是一种不稳定的排序算法。在选择排序的每一轮中，通过不断选择最小（或最大）的元素，将其放置到已排序部分的末尾。这个过程中，相同元素的相对位置可能发生改变，因此选择排序也不是稳定的排序算法。

### 5、让你设计一个线程池，你会如何实现？

设计一个线程池涉及到多个方面，包括线程的创建与销毁、任务的提交与执行、线程间的通信等。以下是一个简单的线程池设计思路：

1. 线程池的结构： 创建一个线程池类，其中包含一个任务队列和一定数量的工作线程。
2. 任务类： 创建一个任务类，用于表示需要在线程池中执行的具体任务。任务类中应包含任务的执行逻辑。
3. 任务队列： 使用一个线程安全的队列来存储待执行的任务。当有新的任务提交时，将任务加入任务队列。
4. 线程管理： 创建一定数量的工作线程，这些线程会循环地从任务队列中取任务并执行。线程执行完一个任务后，继续尝试获取并执行下一个任务。
5. 线程同步： 使用互斥锁等机制来保护任务队列，防止多个线程同时访问导致数据竞争。
6. 任务执行： 工作线程从任务队列中获取任务，执行任务的执行逻辑。执行完任务后，线程可以等待新任务或者被销毁，具体取决于线程池的设计。
7. 线程池的生命周期管理： 提供线程池的初始化、销毁等方法，确保线程池的正常运行和释放占用的资源。

给个例子：

```C++
#include <iostream>
#include <vector>
#include <queue>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <functional>

class ThreadPool {
public:
    ThreadPool(size_t numThreads) : stop(false) {
        for (size_t i = 0; i < numThreads; ++i) {
            workers.emplace_back([this] {
                while (true) {
                    std::function<void()> task;

                    {
                        std::unique_lock<std::mutex> lock(queueMutex);
                        condition.wait(lock, [this] { return stop || !tasks.empty(); });

                        if (stop && tasks.empty()) {
                            return;
                        }

                        task = std::move(tasks.front());
                        tasks.pop();
                    }

                    task();
                }
            });
        }
    }

    ~ThreadPool() {
        {
            std::unique_lock<std::mutex> lock(queueMutex);
            stop = true;
        }

        condition.notify_all();

        for (std::thread &worker : workers) {
            worker.join();
        }
    }

    template<class F>
    void enqueue(F&& f) {
        {
            std::unique_lock<std::mutex> lock(queueMutex);
            tasks.emplace(std::forward<F>(f));
        }

        condition.notify_one();
    }

private:
    std::vector<std::thread> workers;
    std::queue<std::function<void()>> tasks;
    std::mutex queueMutex;
    std::condition_variable condition;
    bool stop;
};

int main() {
    ThreadPool pool(4);

    for (int i = 0; i < 8; ++i) {
        pool.enqueue([i] {
            std::cout << "Task " << i << " executed by thread " << std::this_thread::get_id() << std::endl;
        });
    }

    // Sleep to allow threads to finish
    std::this_thread::sleep_for(std::chrono::seconds(2));

    return 0;
}
```

### 6、线程数量可能多了或者少了，怎么解决这个问题？

1. 自适应调整： 在线程池中引入自适应机制，根据任务队列的长度、处理任务的速度等动态地调整线程数量。如果任务队列积压较多，可以增加线程数量以加速处理；如果任务队列较空闲，可以减少线程数量以节省资源。
2. 定期调整： 定期检查任务队列的状态和系统资源使用情况，根据这些信息调整线程数量。例如，可以每隔一定时间检查一次，并根据任务队列长度和线程执行速度来动态调整。
3. 弹性线程池： 实现一个弹性线程池，可以根据系统负载自动伸缩。这可以通过监控系统的 CPU 使用率、内存使用率等指标，动态调整线程池的大小。
4. 任务执行时间监控： 统计每个任务的执行时间，根据任务执行时间的长短来判断是否需要增加或减少线程数量。执行时间较长的任务可能需要更多的线程来并发执行。

### 7、有哪些IO多路复用技术？

1. select： 是Unix/Linux系统下的多路复用IO函数，通过select函数可以同时监控多个文件描述符的可读、可写和异常等事件。缺点是效率较低，受到文件描述符数量的限制。
2. poll： 与select类似，也可以用于监控多个文件描述符。poll没有文件描述符数量的限制，但是在大量文件描述符时性能仍然不高。
3. epoll： 是Linux特有的多路复用IO函数，是select和poll的增强版。通过epoll可以监听大量的文件描述符，且性能随着文件描述符数量的增加而线性增长。epoll使用回调机制，将活跃的文件描述符放入一个事件表中，而不是像select和poll一样每次都遍历整个文件描述符集合。

### 8、select和epoll的区别？

1. 效率：
   - `select` 的效率较低，因为每次调用 `select` 都需要线性扫描所有的文件描述符，时间复杂度为 O(n)。
   - `epoll` 使用了回调机制，只有活跃的文件描述符会被放入一个事件表中，因此性能较好，且随着文件描述符数量的增加而线性增长。
2. 文件描述符数量限制：
   - `select` 的文件描述符数量存在限制，通常被限制在 1024 个左右，因为它是使用位图实现的，一个字节有 8 位，因此 1024 个描述符需要 1024 / 8 = 128 字节的位图。
   - `epoll` 没有这样的限制，它可以支持非常大的并发连接。
3. 触发方式：
   - `select` 和 `poll` 是水平触发的，即只要文件描述符状态发生变化就会通知应用程序，应用程序需要反复调用 `select` 或 `poll`。
   - `epoll` 支持水平触发和边缘触发两种方式。边缘触发表示只有在状态变化的瞬间才会通知应用程序。
4. API 的差异：
   - `select` 和 `poll` 使用相似的 API，需要不断地传递文件描述符集合，并在返回时检查每个文件描述符的状态。
   - `epoll` 使用三个 API 函数：`epoll_create` 创建一个 epoll 实例，`epoll_ctl` 用于注册和删除文件描述符，`epoll_wait` 用于等待事件的发生。

### 9、中序遍历非递归实现（迭代）

思路：

1. 从根节点开始，一直访问左子树，同时将经过的节点入栈。
2. 当左子树访问完毕（为空）时，弹出栈顶元素，访问该节点，并转向其右子树，然后重复步骤1。
3. 直到栈为空且当前节点为空时，遍历结束。

参考代码：

```C++
#include <iostream>
#include <stack>

// 二叉树的节点结构
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

void inorderTraversal(TreeNode* root) {
    std::stack<TreeNode*> nodeStack;
    TreeNode* current = root;

    while (current != nullptr || !nodeStack.empty()) {
        // 将左子树入栈
        while (current != nullptr) {
            nodeStack.push(current);
            current = current->left;
        }

        // 访问节点并转向右子树
        current = nodeStack.top();
        nodeStack.pop();
        std::cout << current->val << " "; // 访问节点
        current = current->right;
    }
}

int main() {
    // 构建一个二叉树
    TreeNode* root = new TreeNode(1);
    root->left = new TreeNode(2);
    root->right = new TreeNode(3);
    root->left->left = new TreeNode(4);
    root->left->right = new TreeNode(5);

    // 中序遍历
    std::cout << "Inorder Traversal: ";
    inorderTraversal(root);

    return 0;
}
```

### 10、反问

岗位侧重点？更侧重于业务功能实现，围绕音视频编解码的应用（在线课程直播之类的）

学习建议？QT原理、数据结构算法要会用，知道什么场景下用什么数据结构