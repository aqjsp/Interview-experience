1、自我介绍

2、谈谈你印象较深的项目

3、C++中的虚函数和纯虚函数有什么区别，使用场景

虚函数（Virtual Function）：

1. 实现方式： 虚函数是在基类中声明并使用关键字 `virtual` 标记的函数，可以有默认实现，也可以被派生类重写。
2. 基类中有默认实现： 虚函数在基类中可以有默认的实现，如果派生类没有重写这个虚函数，将使用基类中的默认实现。
3. 可被调用： 虚函数可以在基类中被调用，也可以在派生类中被调用。基类的指针或引用可以指向派生类的对象，并调用派生类中的虚函数。

```C++
class Base {
public:
    virtual void virtualFunction() {
        // 默认实现...
    }
};

class Derived : public Base {
public:
    void virtualFunction() override {
        // 重写实现...
    }
};

Base* obj = new Derived();
obj->virtualFunction();  // 调用派生类中的虚函数
```

纯虚函数（Pure Virtual Function）：

1. 声明方式： 纯虚函数是在基类中声明并使用关键字 `virtual` 和 `= 0` 标记的函数，没有默认实现，必须在派生类中被重写。
2. 没有默认实现： 纯虚函数在基类中没有默认实现，派生类必须提供具体的实现。
3. 纯虚函数的类是抽象类： 包含纯虚函数的类被称为抽象类，无法被实例化。必须由派生类提供纯虚函数的实现。

```C++
class AbstractBase {
public:
    virtual void pureVirtualFunction() = 0;  // 纯虚函数
};

class ConcreteDerived : public AbstractBase {
public:
    void pureVirtualFunction() override {
        // 具体的实现...
    }
};

AbstractBase* obj = new ConcreteDerived();  // 可以使用指针或引用指向派生类的对象
obj->pureVirtualFunction();  // 调用派生类中的纯虚函数
```

使用场景：

虚函数：

1. 实现多态性： 虚函数是实现运行时多态性的关键。通过基类指针或引用指向派生类对象，可以根据实际对象的类型来调用相应的虚函数。

```C++
class Shape {
public:
    virtual void draw() const {
        // 具体的绘制操作...
    }
};

class Circle : public Shape {
public:
    void draw() const override {
        // 绘制圆形的操作...
    }
};

int main() {
    Shape* shape = new Circle();
    shape->draw();  // 调用派生类中的虚函数
    delete shape;
    return 0;
}
```

2. 提供默认实现： 虚函数在基类中可以提供默认的实现，如果派生类不重写，将使用基类的默认实现。

```C++
class Base {
public:
    virtual void functionWithDefault() {
        // 默认实现...
    }
};

class Derived : public Base {
    // 没有重写 functionWithDefault，使用基类的默认实现
};

int main() {
    Derived derived;
    derived.functionWithDefault();  // 调用基类的默认实现
    return 0;
}
```

纯虚函数：

1. 实现抽象类： 包含纯虚函数的类是抽象类，无法被实例化。它们提供一个接口，要求派生类提供具体的实现。

```C++
class AbstractShape {
public:
    virtual void draw() const = 0;  // 纯虚函数
};

class ConcreteCircle : public AbstractShape {
public:
    void draw() const override {
        // 具体的绘制圆形的操作...
    }
};

int main() {
    AbstractShape* shape = new ConcreteCircle();
    shape->draw();  // 调用派生类中的纯虚函数
    delete shape;
    return 0;
}
```

2. 接口定义： 纯虚函数可以用于定义接口，让多个类共享相同的接口，但提供不同的实现。

```C++
class Interface {
public:
    virtual void method() const = 0;  // 纯虚函数
};

class ConcreteClassA : public Interface {
public:
    void method() const override {
        // 具体的实现...
    }
};

class ConcreteClassB : public Interface {
public:
    void method() const override {
        // 具体的实现...
    }
};

int main() {
    Interface* objA = new ConcreteClassA();
    Interface* objB = new ConcreteClassB();
    objA->method();  // 调用 ConcreteClassA 中的实现
    objB->method();  // 调用 ConcreteClassB 中的实现
    delete objA;
    delete objB;
    return 0;
}
```

4、纯虚函数和虚函数必须在派生类中实现吗

虚函数（Virtual Function）：

1. 实现可选： 派生类可以选择是否在其中实现基类中声明的虚函数。
2. 默认实现： 如果派生类没有提供虚函数的实现，将使用基类中的默认实现（如果有的话）。
3. 重写： 如果在派生类中提供了虚函数的实现，可以使用 `override` 关键字进行标记。

```C++
class Base {
public:
    virtual void virtualFunction() {
        // 默认实现...
    }
};

class Derived : public Base {
public:
    void virtualFunction() override {
        // 派生类中的实现...
    }
};
```

纯虚函数（Pure Virtual Function）：

1. 必须实现： 派生类必须提供纯虚函数的具体实现，否则派生类也会成为抽象类，无法被实例化。
2. 没有默认实现： 纯虚函数在基类中没有默认的实现。
3. 纯虚函数的类是抽象类： 包含纯虚函数的类被称为抽象类，无法被实例化。

```C++
class AbstractBase {
public:
    virtual void pureVirtualFunction() = 0;  // 纯虚函数
};

class ConcreteDerived : public AbstractBase {
public:
    void pureVirtualFunction() override {
        // 具体的实现...
    }
};
```

5、说说抽象类

抽象类是一种在C++中通过使用纯虚函数（Pure Virtual Function）来定义接口的特殊类。抽象类的实例不能被直接实例化，它的主要目的是为了通过派生类提供具体的实现，并强制子类提供特定的接口。

以下是抽象类的一些特点和使用方式：

特点：

1. 包含纯虚函数： 抽象类包含至少一个纯虚函数，通过在声明中使用 `= 0` 来标记。

```C++
class AbstractClass {
public:
    virtual void pureVirtualFunction() = 0;
};
```

2. 无法实例化： 抽象类无法被直接实例化，它的主要作用是作为一个接口，定义一组纯虚函数，要求派生类提供具体的实现。

3. 可以包含普通函数： 抽象类可以包含普通的成员函数，不一定都是纯虚函数。

```C++
class AbstractClass {
public:
    virtual void pureVirtualFunction() = 0;
    void regularFunction() {
        // 具体的实现...
    }
};
```

使用方式：

1. 作为接口定义： 抽象类通常用于定义一组接口，要求派生类提供具体的实现。这样可以实现多态性，通过基类指针或引用调用派生类的实现。

```C++
AbstractClass* obj = new ConcreteDerived();
obj->pureVirtualFunction();  // 调用派生类中的纯虚函数
delete obj;
```

2. 包含数据成员： 抽象类可以包含数据成员，不限制只有纯虚函数。

```C++
class AbstractShape {
protected:
    double width;
    double height;

public:
    AbstractShape(double w, double h) : width(w), height(h) {}

    virtual double area() const = 0;  // 纯虚函数
};
```

3. 派生类实现接口： 派生类必须提供抽象类中纯虚函数的具体实现，否则派生类也会成为抽象类。

```C++
class ConcreteCircle : public AbstractShape {
public:
    ConcreteCircle(double radius) : AbstractShape(radius, radius) {}

    double area() const override {
        return 3.14 * width * height;
    }
};
```

6、C++的内联函数，和普通函数有什么区别

内联函数：

1. 定义方式： 内联函数的定义通常在类声明中，或者在函数定义前使用 `inline` 关键字。内联函数的定义会被编译器尽量地替换函数调用处的实际代码。

```C++
// 内联函数的定义
inline int add(int a, int b) {
    return a + b;
}
```

2. 编译时替换： 编译器会尽量将内联函数的调用处直接替换为函数体的代码，而不进行函数调用的开销。

3. 适用场景： 内联函数适用于短小的函数体，频繁调用的函数，以减小函数调用的开销。

4. 多次定义的问题： 内联函数的定义通常放在头文件中，如果在多个文件中引用同一个头文件，可能导致多次定义的问题。通常需要使用 `inline` 函数时，建议将函数定义放在头文件中，并使用头文件保护（`#pragma once` 或者 `#ifndef` 和 `#define`）来防止多次定义。

普通函数：

1. 定义方式： 普通函数的定义通常在类声明之外，或者在一个独立的源文件中。

```C++
// 普通函数的定义
int add(int a, int b) {
    return a + b;
}
```

2. 独立的函数调用： 普通函数的调用会产生实际的函数调用开销，即函数调用时会跳转到函数体执行，然后再返回。

3. 适用场景： 普通函数适用于复杂的函数体，不经常调用的函数，或者需要在不同的源文件中共享的函数。

4. 避免多次定义： 普通函数的定义通常放在源文件中，通过函数声明放在头文件中，可以在多个文件中引用而不产生多次定义的问题。

区别：

- 内联函数适用于短小的函数体，频繁调用的情况，以减小函数调用的开销。
- 普通函数适用于复杂的函数体，不经常调用的函数，或者需要在不同的源文件中共享的函数。
- 内联函数的调用处会被编译器尽量替换为函数体的实际代码，而普通函数会产生实际的函数调用开销。

7、C++指针和引用的区别

1. 定义和声明：

- 指针： 指针是一个变量，其值是另一个变量的地址。指针需要通过 `*` 运算符来声明和使用。

```C++
int x = 10;
int *ptr = &x;  // 指针的声明和初始化
```

- 引用： 引用是一个别名，即某个变量的别名。引用在声明时使用 `&` 运算符。

```C++
int x = 10;
int &ref = x;  // 引用的声明和初始化
```

2. 语法和操作符：

- 指针： 指针使用 `*` 运算符来访问所指向的变量。

```C++
int x = 10;
int *ptr = &x;
*ptr = 20;  // 通过指针修改 x 的值
```

- 引用： 引用在声明时使用 `&` 运算符，但在使用时不需要。

```C++
int x = 10;
int &ref = x;
ref = 20;  // 直接修改 x 的值，无需使用运算符
```

3. 空值（Nullability）：

- 指针： 指针可以是空指针（`nullptr` 或 `NULL`），即不指向任何有效的地址。

```C++
int *ptr = nullptr;  // 空指针
```

- 引用： 引用必须在声明时初始化，并且一旦引用被初始化，它将一直引用同一个变量，不存在空引用的概念。

```C++
int x = 10;
int &ref = x;  // 引用初始化
```

4. 多重引用和指针：

- 指针： 指针可以重新赋值指向其他变量或空。

```C++
int x = 10;
int y = 20;
int *ptr = &x;
ptr = &y;  // 指针重新指向 y
```

- 引用： 引用在初始化后不能再引用其他变量。

```C++
int x = 10;
int y = 20;
int &ref = x;
ref = y;  // 修改了 x 的值，但 ref 仍然引用 x
```

5. 数组和函数参数：

- 指针： 指针可以用于遍历数组和作为函数参数传递。

```C++
int arr[5] = {1, 2, 3, 4, 5};
int *ptr = arr;  // 指向数组的第一个元素
void foo(int *ptr) {
    // 函数接受指针参数
}
```

- 引用： 引用不直接用于遍历数组，但可以作为函数参数传递。

```C++
void foo(int &ref) {
    // 函数接受引用参数
}
```

8、指针常量和常量指针的区别

1. 常量指针

定义：具有只能够读取内存中数据，却不能够修改内存中数据的属性的指针，称为指向常量的指针，简称常量指针。

声明：const int * p; int const * p;

**注**：可以将一个常量的地址赋值给一个对应类型的常量指针，因为常量指针不能够通过指针修改内粗数据。只能防止通过指针引用修改内存中的数据，并不保护指针所指向的对象。

2. 指针常量

定义：指针常量是指指针所指向的位置不能改变，即指针本身是一个常量，但是指针所指向的内容可以改变。

声明：int * const p=&a;

**注**：指针常量必须在声明的同时对其初始化，不允许先声明一个指针常量随后再对其赋值，这和声明一般的常量是一样的。

9、C++内存分配的方式有几种

1. 栈内存分配（Stack Allocation）：

   - 栈内存是由编译器自动分配和释放的，用于存储局部变量和函数调用信息。

   - 栈是一块连续的内存区域，遵循"先进后出"的原则。

   - 栈内存的生命周期由作用域控制，当变量超出作用域时，自动释放。

```C++
void exampleFunction() {
    int localVar = 42;  // 栈上分配的局部变量
    // ...
}  // localVar 在此出作用域时自动释放
```

2. 堆内存分配（Heap Allocation）：

   - 堆内存是由程序员手动分配和释放的，使用 `new` 和 `delete`（或 `malloc` 和 `free`）等关键字来管理。

   - 堆是一块动态分配的内存区域，大小不固定。

   - 堆上分配的内存需要手动释放，否则可能导致内存泄漏。

```C++
int* dynamicVar = new int;  // 堆上分配的动态变量
// 使用 dynamicVar
delete dynamicVar;  // 释放堆上分配的内存
```

3. 全局/静态内存分配（Global/Static Allocation）：

   - 全局变量和静态变量在程序启动时就会分配内存，它们的生命周期在整个程序运行期间。

   - 全局变量存储在全局数据段，静态变量存储在静态数据段。

```C++
int globalVar;  // 全局变量
static int staticVar;  // 静态变量
```

4. 常量区内存分配（Constant Data Allocation）：

   - 常量数据通常存储在常量区，包括字符串常量等。

   - 常量区的数据是只读的，不能修改。

```C++
const char* str = "Hello, World!";  // 字符串常量
```

10、TCP三次握手的过程

1. 第一次握手 - SYN（同步）：
   - 客户端发送一个TCP报文，将标志位SYN（Synchronize）设置为1，表示请求建立连接。
   - 客户端选择一个初始序列号（ISN）并将其放入报文中。
2. 第二次握手 - SYN + ACK（同步 + 确认）：
   - 服务器接收到客户端的SYN报文后，如果同意建立连接，会发送一个带有SYN和ACK标志位的报文回复。
   - 服务器也会选择一个初始序列号并将其放入报文中。
3. 第三次握手 - ACK（确认）：
   - 客户端收到服务器的SYN + ACK报文后，向服务器发送一个带有ACK标志位的报文。
   - 该报文中的序列号被设置为收到的SYN + ACK报文的序列号加1。
   - 服务器收到客户端的ACK报文后，确认连接建立。

11、给定一个字符串 s ，请你找出其中不含有重复字符的最长子串的长度。

> 示例 1:
>
> 输入: s = "abcabcbb"
>
> 输出: 3
>
> 解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3

使用滑动窗口（双指针）的方法来解决这个问题。

首先使用滑动窗口来维护当前的子串，它由左指针 `left` 和右指针 `right` 组成。初始时，两个指针都指向字符串的开头。

然后，我们使用 `unordered_set` 数据结构来存储当前子串中出现过的字符，这样可以快速判断一个字符是否重复出现。我们不断地移动右指针 `right`，如果当前字符不在 `charSet` 中，就将其加入集合并更新最大长度；如果当前字符已经在集合中，就移动左指针 `left`，并从集合中删除相应字符，直到当前子串不再包含重复字符为止。

最终，返回的 `maxLength` 就是最长不含重复字符的子串的长度。

```C
#include <iostream>
#include <string>
#include <unordered_set>

int lengthOfLongestSubstring(std::string s) {
    int n = s.length();
    int maxLength = 0;
    int left = 0, right = 0;
    std::unordered_set<char> charSet;

    while (right < n) {
        char currentChar = s[right];
        if (charSet.find(currentChar) == charSet.end()) {
            charSet.insert(currentChar);
            maxLength = std::max(maxLength, right - left + 1);
            right++;
        } else {
            charSet.erase(s[left]);
            left++;
        }
    }

    return maxLength;
}

int main() {
    std::string s = "abcabcbb"; // 例如，输入字符串 "abcabcbb"
    int length = lengthOfLongestSubstring(s);
    std::cout << "最长不含重复字符的子串长度是：" << length << std::endl;
    return 0;
}
```

时间复杂度是 O(N)，其中 N 是输入字符串的长度。

结果输出：

![img](https://vnkshn64w3.feishu.cn/space/api/box/stream/download/asynccode/?code=ZDE1OGQzNGQ0MzJkMTczYmYxZGM4YWQzNzYxZTU3MTRfcllYQjJTR3JvUERNMm1HSE1iZnh6SWd5bzFIWm5DZmVfVG9rZW46UmpaVWJOazlmbzRPRTd4T2xNeGNwWkpUbmRRXzE3MDMwOTA1NzY6MTcwMzA5NDE3Nl9WNA)