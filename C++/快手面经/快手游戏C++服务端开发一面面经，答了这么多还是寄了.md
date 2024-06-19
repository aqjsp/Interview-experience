# 快手游戏C++服务端开发一面面经，答了这么多还是寄了。。。

> 来源：https://www.nowcoder.com/discuss/620359211850977280

## C++开发

### 1、类的静态成员函数的特点？

1. **无需对象实例**：静态成员函数不依赖于类的对象实例，可以直接通过类名调用。例如，`ClassName::StaticFunction()`。
2. **访问限制**：静态成员函数只能访问静态成员变量和其他静态成员函数，不能访问非静态成员变量和非静态成员函数。因为静态成员函数在调用时并不依赖于具体的对象实例，所以没有`this`指针。
3. **共享属性**：静态成员变量和静态成员函数是类的共享成员，所有对象实例共享同一份静态成员数据。这对于实现全局属性或全局行为非常有用。
4. **生命周期**：静态成员函数和静态成员变量在程序运行开始时分配内存，直到程序结束时才释放内存。它们的生命周期贯穿整个程序运行过程。
5. **内存分配**：静态成员变量存储在静态存储区中，而不是在对象的内存空间中。静态成员函数的代码段则与普通成员函数无异，但它们不依赖于对象实例。
6. **可以作为回调函数**：由于静态成员函数不需要对象实例，它们可以作为普通函数指针使用，适用于某些需要回调函数的场景。

给一个示例代码，静态成员函数的使用：

```
#include <iostream>
using namespace std;

class MyClass {
public:
    // 静态成员变量
    static int staticVar;

    // 静态成员函数
    static void staticFunction() {
        cout << "Static Function Called. staticVar = " << staticVar << endl;
        // 不能访问非静态成员变量或非静态成员函数
        // nonStaticFunction(); // 错误
    }

    // 非静态成员函数
    void nonStaticFunction() {
        cout << "Non-Static Function Called." << endl;
    }
};

// 静态成员变量的初始化
int MyClass::staticVar = 0;

int main() {
    // 通过类名调用静态成员函数
    MyClass::staticFunction();

    // 修改静态成员变量
    MyClass::staticVar = 10;

    // 通过类名再次调用静态成员函数
    MyClass::staticFunction();

    // 创建对象实例
    MyClass obj;

    // 通过对象实例调用静态成员函数（不推荐）
    obj.staticFunction();

    return 0;
}
```

输出：

```
Static Function Called. staticVar = 0
Static Function Called. staticVar = 10
Static Function Called. staticVar = 10
```

### 2、虚函数是怎么实现的？

#### 虚函数表（vtable）

虚函数表是一个函数指针数组，每个类都有一个虚函数表，用来存储该类的虚函数的地址。当一个类定义了虚函数时，编译器会为该类生成一个虚函数表。

#### 虚函数指针（vptr）

每个含有虚函数的类的对象中，编译器会自动添加一个虚函数指针（vptr），该指针指向该类的虚函数表。通过这个指针，对象可以在运行时找到正确的函数实现，从而实现动态绑定。

#### 实现过程

1. **类定义**：
   - 当一个类定义了虚函数，编译器会为该类生成一个虚函数表。
   - 虚函数表中存储了该类的所有虚函数的地址。
2. **对象创建**：
   - 当创建一个对象时，该对象会包含一个指向该类虚函数表的指针（vptr）。
   - vptr在对象构造时被初始化，指向相应的虚函数表。
3. **函数调用**：
   - 当通过基类指针或引用调用虚函数时，程序会通过对象的vptr找到虚函数表，然后根据虚函数表中的地址调用实际的函数实现。

#### 虚函数的动态绑定

虚函数的动态绑定是通过虚函数表和虚函数指针在运行时实现的。当调用虚函数时，程序会根据对象的vptr找到正确的函数地址，从而实现动态绑定。这使得C++可以在运行时根据实际对象类型调用相应的函数，实现多态性。

#### 虚函数的优点

- **多态性**：通过虚函数，可以在基类指针或引用的基础上调用子类的实现，实现多态性。
- **灵活性**：虚函数允许子类重写父类的方法，使得代码更具灵活性和可扩展性。

#### 虚函数的缺点

- **开销**：使用虚函数需要额外的内存开销来存储虚函数表和虚函数指针，同时在运行时也有一定的性能开销，因为需要通过虚函数表查找函数地址。
- **不支持内联**：虚函数通常不能内联，因为它们的调用在编译时无法确定具体的函数实现。

### 3、一个对象最多只有一个虚函数指针吗？

#### 单一继承情况下的虚函数指针

在单一继承的情况下，每个包含虚函数的类的对象会有一个vptr，这个vptr指向该类的虚函数表（vtable）。

```
class Base {
public:
    virtual void show() {
        cout << "Base class show function" << endl;
    }
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    void show() override {
        cout << "Derived class show function" << endl;
    }
};
```

上述例子中，无论是`Base`类还是`Derived`类的对象，都会有一个vptr指向它们各自的虚函数表。

#### 多重继承情况下的虚函数指针

在多重继承的情况下，每个对象可能会有多个虚函数指针。这是因为每个基类都有自己的虚函数表，而每个继承自多个基类的类的对象必须维护多个虚函数指针，以指向各个基类的虚函数表。

```
class Base1 {
public:
    virtual void show1() {
        cout << "Base1 show1 function" << endl;
    }
    virtual ~Base1() = default;
};

class Base2 {
public:
    virtual void show2() {
        cout << "Base2 show2 function" << endl;
    }
    virtual ~Base2() = default;
};

class Derived : public Base1, public Base2 {
public:
    void show1() override {
        cout << "Derived show1 function" << endl;
    }
    void show2() override {
        cout << "Derived show2 function" << endl;
    }
};
```

上述例子中，`Derived`类从`Base1`和`Base2`继承而来。因此，`Derived`类的对象将包含两个vptr，一个指向`Base1`的虚函数表，另一个指向`Base2`的虚函数表。

#### 单一继承

对于单一继承，一个类只有一个基类，因此对象只需一个vptr。

```
Base* ptr = new Derived();
ptr->show(); // 调用Derived的show函数
```

#### 多重继承

对于多重继承，一个类有多个基类，每个基类都有自己的虚函数表，因此对象需要多个vptr。

```
Derived* d = new Derived();
Base1* b1 = d;
Base2* b2 = d;

b1->show1(); // 调用Derived的show1函数
b2->show2(); // 调用Derived的show2函数
```

这种情况下，`d`对象有两个vptr，一个指向`Base1`的虚函数表，另一个指向`Base2`的虚函数表。

### 4、友元函数能作为虚函数吗？

#### 友元函数

友元函数是一种非成员函数，但它可以访问类的私有成员和保护成员。友元函数是通过在类内声明`friend`关键字来指定的。友元函数的主要目的是为了提供一种机制，使非成员函数可以访问类的私有数据。

```
class MyClass {
private:
    int data;
public:
    MyClass(int value) : data(value) {}
    
    // 声明友元函数
    friend void showData(const MyClass& obj);
};

// 友元函数定义
void showData(const MyClass& obj) {
    cout << "Data: " << obj.data << endl;
}

int main() {
    MyClass obj(10);
    showData(obj); // 调用友元函数，输出 "Data: 10"
    return 0;
}
```

在这个例子中，`showData`函数是`MyClass`类的友元函数，它能够访问`MyClass`对象的私有成员`data`。

#### 虚函数

虚函数是类的成员函数，通过在基类中使用`virtual`关键字声明。虚函数的主要目的是实现多态性，使得通过基类指针或引用调用派生类的重写函数。

```
class Base {
public:
    virtual void show() {
        cout << "Base class show function" << endl;
    }
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    void show() override {
        cout << "Derived class show function" << endl;
    }
};

int main() {
    Base* bptr = new Derived();
    bptr->show(); // 输出 "Derived class show function"
    delete bptr;
    return 0;
}
```

在这个例子中，通过基类指针`bptr`调用虚函数`show`，实现了运行时的多态性。

#### 友元函数不能是虚函数的原因

1. **非成员函数**：友元函数不是类的成员函数，而虚函数必须是类的成员函数。虚函数需要在类的虚函数表中有一个条目，以便在运行时进行动态绑定，而友元函数不在类的范围内，因此无法包含在虚函数表中。
2. **访问机制**：虚函数是通过对象的vptr指向的虚函数表来实现动态绑定的，而友元函数没有这种机制。友元函数只是一个普通的函数，它不依赖于具体的对象实例，没有vptr指向它。
3. **多态性要求**：虚函数的主要目的是为了实现多态性，使得基类指针或引用能够调用派生类的重写函数。友元函数不具备这种需求和功能。

### 5、内联成员函数可以是虚函数吗？

#### 内联函数

内联函数是一种提示编译器将函数调用展开为函数体内的代码，以避免函数调用的开销。内联函数通过在函数定义前加上`inline`关键字来声明。内联函数通常用于短小、频繁调用的函数。

#### 虚函数

虚函数是用于实现多态性的成员函数，允许通过基类指针或引用调用派生类的重写函数。虚函数通过在基类中使用`virtual`关键字声明。

#### 内联函数和虚函数的结合

内联函数可以是虚函数，但在实际使用中存在一些限制和考虑。以下是详细的解释：

1. **虚函数的内联属性**：
   - 虚函数可以在类定义中声明为内联函数。然而，虚函数在基类中的内联声明不会自动使其在派生类中内联。派生类重写的虚函数需要显式地内联。
   - 内联只是对编译器的建议，编译器可能会忽略内联建议，特别是在虚函数的情况下。
2. **多态性和内联的冲突**：
   - 虚函数的多态性依赖于运行时的动态绑定，即通过虚函数表（vtable）在运行时确定调用哪个函数实现。这种动态绑定在某些情况下会阻碍内联展开，因为内联展开需要在编译时确定调用目标。
   - 如果通过基类指针或引用调用虚函数，编译器在编译时无法确定实际调用的函数实现，因此不能内联展开。
3. **特殊情况下的内联**：
   - 如果编译器在编译时能够确定虚函数的实际类型（例如，通过具体对象调用而不是通过基类指针或引用），那么编译器可能会内联该虚函数。
   - 以下示例展示了这种情况：

```
#include <iostream>

class Base {
public:
    virtual void show() {
        std::cout << "Base class show function" << std::endl;
    }
};

class Derived : public Base {
public:
    inline void show() override {
        std::cout << "Derived class show function" << std::endl;
    }
};

int main() {
    Derived d;
    d.show(); // 可能被内联展开

    Base* bptr = &d;
    bptr->show(); // 不会被内联展开，因为是通过基类指针调用
    return 0;
}
```

### 6、模板函数可以是虚函数吗？

#### 模板函数

模板函数是一种函数，它允许我们在编写代码时使用泛型类型。模板函数在编译时会根据提供的具体类型生成具体的函数实例。

```
template<typename T>
void show(T value) {
    std::cout << value << std::endl;
}
```

#### 虚函数

虚函数是用于实现运行时多态性的成员函数，允许基类指针或引用调用派生类的重写函数。虚函数依赖于虚函数表（vtable）和虚函数指针（vptr）来在运行时动态绑定函数调用。

```
class Base {
public:
    virtual void show() {
        std::cout << "Base class show function" << std::endl;
    }
};

class Derived : public Base {
public:
    void show() override {
        std::cout << "Derived class show function" << std::endl;
    }
};
```

#### 模板函数与虚函数的区别

1. **编译时与运行时**：
   - 模板函数在编译时实例化，具体类型在编译时确定，生成对应的函数代码。
   - 虚函数在运行时动态绑定，通过虚函数表和虚函数指针确定调用哪个具体的函数实现。
2. **函数表机制**：
   - 虚函数依赖于虚函数表，而模板函数在编译时生成具体的函数实例，无法在编译时生成统一的虚函数表结构。
   - 由于模板函数是在编译时生成具体实例，而不是在类定义时确定，所以无法将其地址放入虚函数表中。

#### 为什么模板函数不能是虚函数

1. **模板函数在编译时实例化**：
   - 模板函数在使用时根据具体类型实例化，这意味着编译器在编译期生成特定类型的函数代码。在类定义时，编译器无法知道所有可能的模板实例化，因此无法创建虚函数表条目。
2. **虚函数表的固定结构**：
   - 虚函数表在类定义时创建并固定下来，包含指向虚函数的指针。模板函数实例化是在编译期进行的，与虚函数表的运行时动态绑定机制不兼容。

### 7、模板函数是怎么实现的，为什么能实现多态？

#### 模板函数的实现

模板函数的实现是通过模板实例化完成的。当我们定义一个模板函数时，并不会立即生成函数代码。只有在使用该模板函数时（即实例化模板函数时），编译器才会生成对应的具体函数代码。

```
template<typename T>
void show(T value) {
    std::cout << value << std::endl;
}
```

当我们调用这个模板函数时，

```
show(10);     // T 实例化为 int
show(3.14);   // T 实例化为 double
show("Hello"); // T 实例化为 const char*
```

编译器会生成以下具体的函数实例：

```
void show(int value) {
    std::cout << value << std::endl;
}

void show(double value) {
    std::cout << value << std::endl;
}

void show(const char* value) {
    std::cout << value << std::endl;
}
```

#### 编译时多态性

模板函数通过模板实例化实现编译时多态性，这意味着在编译期生成不同类型的函数实例。编译时多态性主要依赖于编译期的类型推导和模板实例化机制。

```
template<typename T>
T add(T a, T b) {
    return a + b;
}

int main() {
    int x = add(1, 2);       // 生成 add(int, int)
    double y = add(1.1, 2.2); // 生成 add(double, double)
    return 0;
}
```

### 8、有用过智能指针吗，有哪些类型，分别是什么区别？

#### 1. `std::unique_ptr`

`std::unique_ptr`是独占所有权的智能指针，确保一个对象在同一时间只有一个智能指针指向它。

- **特点**：

  - 独占所有权，不能复制，只能移动。
  - 在离开作用域时自动释放所指向的对象。
  - 提供轻量级的所有权转移机制（移动语义）。

- **使用场景**：

  - 适用于确定某个对象只会有一个所有者的情况。
  - 用于资源管理，例如文件句柄、网络连接等。

- **示例代码**：

  ```
  #include <memory>
  #include <iostream>
  
  void uniquePtrExample() {
      std::unique_ptr<int> p1(new int(10));
      std::cout << *p1 << std::endl;
  
      // std::unique_ptr<int> p2 = p1; // 错误，不能复制
      std::unique_ptr<int> p2 = std::move(p1); // 移动p1到p2
      if (!p1) {
          std::cout << "p1 is null" << std::endl;
      }
  }
  ```

#### 2. `std::shared_ptr`

`std::shared_ptr`是共享所有权的智能指针，可以有多个智能指针指向同一个对象。对象会在最后一个引用离开作用域时被销毁。

- **特点**：

  - 共享所有权，通过引用计数实现。
  - 可以复制和赋值，引用计数会相应增加或减少。
  - 在所有共享指针离开作用域后自动释放所指向的对象。

- **使用场景**：

  - 适用于多个所有者共享同一个资源的情况。
  - 适用于需要动态内存管理但不确定对象生命周期的情况。

- **示例代码**：

  ```
  #include <memory>
  #include <iostream>
  
  void sharedPtrExample() {
      std::shared_ptr<int> p1(new int(20));
      std::shared_ptr<int> p2 = p1; // p1和p2共享所有权
      std::cout << *p1 << ", " << *p2 << std::endl;
      std::cout << "use count: " << p1.use_count() << std::endl;
  }
  ```

#### 3. `std::weak_ptr`

`std::weak_ptr`是一种不控制对象生命周期的智能指针，与`std::shared_ptr`配合使用，解决循环引用的问题。

- **特点**：

  - 不影响对象的引用计数。
  - 可以从`std::shared_ptr`创建，弱引用不会增加引用计数。
  - 需要使用`std::weak_ptr`的`lock`方法提升为`std::shared_ptr`以访问对象。

- **使用场景**：

  - 适用于需要观察一个共享对象，但不想影响其生命周期的情况。
  - 解决`std::shared_ptr`的循环引用问题。

- **示例代码**：

  ```
  #include <memory>
  #include <iostream>
  
  void weakPtrExample() {
      std::shared_ptr<int> p1 = std::make_shared<int>(30);
      std::weak_ptr<int> wp = p1; // 创建弱引用
  
      std::shared_ptr<int> p2 = wp.lock(); // 提升为共享引用
      if (p2) {
          std::cout << *p2 << std::endl;
          std::cout << "use count: " << p2.use_count() << std::endl;
      }
  }
  ```

#### 智能指针的对比

| 智能指针类型      | 所有权     | 引用计数 | 主要特点                                                     | 使用场景                                     |
| ----------------- | ---------- | -------- | ------------------------------------------------------------ | -------------------------------------------- |
| `std::unique_ptr` | 独占所有权 | 无       | 独占所有权，不能复制，只能移动                               | 确定只有一个所有者，避免资源泄漏             |
| `std::shared_ptr` | 共享所有权 | 有       | 共享所有权，通过引用计数管理对象生命周期                     | 多个所有者共享资源，动态内存管理             |
| `std::weak_ptr`   | 弱引用     | 无       | 与`std::shared_ptr`配合使用，不增加引用计数，解决循环引用问题 | 观察共享对象，不影响其生命周期，解决循环引用 |

### 9、noexcept关键字了解吗？

`noexcept`可以用于声明一个函数不会抛出异常。有两种形式：

1. **无条件形式**：明确表示函数不会抛出异常。
2. **条件形式**：基于一个布尔表达式来决定函数是否不会抛出异常。

#### 无条件形式

无条件形式的`noexcept`声明一个函数在任何情况下都不会抛出异常。

```
void func() noexcept {
    // This function is guaranteed not to throw an exception
}
```

#### 条件形式

条件形式的`noexcept`根据一个布尔表达式来决定函数是否不会抛出异常。常见的用法是通过类型特性来判断。

```
template<typename T>
void func(T t) noexcept(noexcept(T())) {
    // This function will not throw an exception if T's constructor does not throw
}
```

#### 使用`noexcept`的原因

1. **性能优化**：标记为`noexcept`的函数可以进行更多的优化，例如在调用这些函数时不需要生成处理异常的代码路径。
2. **提高可预测性**：当一个函数标记为`noexcept`时，调用者可以确信该函数不会抛出异常，从而简化错误处理逻辑。
3. **提高安全性**：在某些场景下（例如在析构函数中），抛出异常会导致程序异常终止。使用`noexcept`可以避免这种情况。

#### 示例代码

以下是一些示例代码，展示了如何使用`noexcept`关键字：

##### 简单的无条件`noexcept`

```
void doWork() noexcept {
    // Guaranteed not to throw an exception
}

int main() {
    doWork();
    return 0;
}
```

##### 带有条件的`noexcept`

```
#include <type_traits>

template<typename T>
void maybeThrow(T t) noexcept(std::is_nothrow_copy_constructible<T>::value) {
    // Will not throw if T's copy constructor is noexcept
}

int main() {
    int a = 5;
    maybeThrow(a); // OK, int is nothrow copy constructible

    std::string s = "Hello";
    maybeThrow(s); // OK, std::string's copy constructor may throw
    return 0;
}
```

#### 与其他C++特性的结合

##### 与移动构造函数和移动赋值运算符的结合

在实现移动构造函数和移动赋值运算符时，使用`noexcept`可以避免不必要的性能开销。

```
class MyClass {
public:
    MyClass(MyClass&& other) noexcept {
        // Move constructor
    }

    MyClass& operator=(MyClass&& other) noexcept {
        // Move assignment operator
        return *this;
    }
};
```

##### 标准库中的`noexcept`

C++标准库中很多函数和运算符都使用了`noexcept`来优化性能和行为。例如，标准库容器在执行某些操作（如`std::vector`的移动操作）时会检查元素类型的构造函数和赋值运算符是否是`noexcept`的。

### 10、完美转发了解吗？

#### 万能引用（Universal References）

万能引用是指模板参数的类型推导时可以同时表示左值引用和右值引用的引用。万能引用的形式是`T&&`，其中`T`是模板参数。

```
template<typename T>
void func(T&& arg) {
    // arg 是万能引用
}
```

#### 完美转发的实现

完美转发通过结合万能引用和`std::forward`来实现。`std::forward`根据传入的参数类型决定是将参数转发为左值引用还是右值引用。

##### `std::forward`的用法

`std::forward`是一个标准库函数，用于保留参数的左值或右值性质。以下是一个例子：

```
#include <utility>
#include <iostream>

void process(int& lref) {
    std::cout << "Lvalue reference" << std::endl;
}

void process(int&& rref) {
    std::cout << "Rvalue reference" << std::endl;
}

template<typename T>
void wrapper(T&& arg) {
    process(std::forward<T>(arg)); // 使用std::forward进行完美转发
}

int main() {
    int x = 10;
    wrapper(x);        // Lvalue reference
    wrapper(10);       // Rvalue reference
    return 0;
}
```

#### 完美转发的优势

1. **避免不必要的拷贝**：通过完美转发，能够避免不必要的对象拷贝，提高程序性能。
2. **保留参数特性**：完美转发保留了参数的左值或右值属性，使得被调用函数能够正确处理参数。

### 11、vector底层是怎么实现的，是线程安全的吗？

`std::vector` 的底层数据结构是一个连续的动态数组，元素在内存中排列成一系列的单元。这使得通过索引来访问元素非常高效，因为它可以通过内存指针和偏移量直接计算出元素的地址。

C++标准并没有规定`std::vector`必须是线程安全的。因此，在多线程环境中，如果有一个线程正在修改`std::vector`，而另一个线程同时对其进行访问（读取或修改），就有可能导致竞态条件（Race Condition）和数据竞争（Data Race）。为了确保在多线程环境下的安全使用，可以使用互斥锁（Mutex）或其他线程同步机制来保护`std::vector`的访问。

### 12、说一下最小堆的插入和删除的过程？

#### 最小堆的插入操作

1. **插入元素**：首先将新元素插入到堆的末尾（即数组的最后一个位置）。
2. **上浮操作**：然后将新元素与其父节点进行比较。如果新元素的值小于父节点的值，则交换它们的位置，继续向上比较直到满足最小堆的性质为止。

插入操作的时间复杂度为O(log n)，其中n为堆中元素的个数。

#### 最小堆的删除操作

1. **删除根节点**：首先将根节点（堆顶元素）删除。为了保持完全二叉树的结构，通常将堆的最后一个元素移动到根节点的位置。
2. **下沉操作**：然后将新的根节点与其子节点中较小的节点进行比较。如果新根节点的值大于子节点的值，则交换它们的位置，继续向下比较直到满足最小堆的性质为止。

删除操作的时间复杂度也是O(log n)，其中n为堆中元素的个数。

#### 示例代码

```
#include <iostream>
#include <vector>

class MinHeap {
private:
    std::vector<int> heap;

    // 上浮操作
    void heapifyUp(int index) {
        int parent = (index - 1) / 2;
        while (index > 0 && heap[index] < heap[parent]) {
            std::swap(heap[index], heap[parent]);
            index = parent;
            parent = (index - 1) / 2;
        }
    }

    // 下沉操作
    void heapifyDown(int index) {
        int left = 2 * index + 1;
        int right = 2 * index + 2;
        int smallest = index;

        if (left < heap.size() && heap[left] < heap[smallest]) {
            smallest = left;
        }
        if (right < heap.size() && heap[right] < heap[smallest]) {
            smallest = right;
        }

        if (smallest != index) {
            std::swap(heap[index], heap[smallest]);
            heapifyDown(smallest);
        }
    }

public:
    // 插入操作
    void insert(int value) {
        heap.push_back(value);
        heapifyUp(heap.size() - 1);
    }

    // 删除操作
    int extractMin() {
        if (heap.empty()) {
            throw std::out_of_range("Heap is empty");
        }

        int root = heap.front();
        heap[0] = heap.back();
        heap.pop_back();
        heapifyDown(0);

        return root;
    }

    // 获取堆顶元素（最小元素）
    int getMin() {
        if (heap.empty()) {
            throw std::out_of_range("Heap is empty");
        }
        return heap.front();
    }

    // 获取堆的大小
    size_t size() {
        return heap.size();
    }

    // 判断堆是否为空
    bool empty() {
        return heap.empty();
    }
};

int main() {
    MinHeap heap;
    heap.insert(3);
    heap.insert(2);
    heap.insert(1);
    heap.insert(15);
    heap.insert(5);

    std::cout << "Heap size: " << heap.size() << std::endl;
    while (!heap.empty()) {
        std::cout << heap.extractMin() << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

### 13、解决哈希冲突有哪些方法？

1. 拉链法（Chaining）：
   - 拉链法是一种开放地址法，它将多个映射到同一槽位的元素存储在一个链表（或其他数据结构）中。
   - 当哈希冲突发生时，元素被添加到链表中。在查找、插入和删除元素时，首先找到对应的槽位，然后在链表中搜索或操作。
   - 拉链法的优点是简单且易于实现，但在链表变得很长时，性能可能会下降。
2. 线性探测法（Linear Probing）：
   - 线性探测法是一种封闭地址法，当哈希冲突发生时，它会查找下一个可用的槽位。
   - 如果槽位被占用，就线性地查找下一个槽位，直到找到一个空槽位或达到表的末尾。
   - 线性探测法可能导致聚集现象，即连续的槽位会被多次占用，降低性能。因此，通常需要解决二次聚集或更复杂的聚集问题。
3. 二次探测法（Quadratic Probing）：
   - 二次探测法是线性探测法的变种，它使用二次函数来查找下一个可用的槽位，以减少聚集现象。
   - 二次探测法的探测步长是一个二次函数的结果，通常是 `c1 * i^2 + c2 * i`，其中 `c1` 和 `c2` 是常数，`i` 是探测的次数。
   - 二次探测法的性能通常比线性探测法好，但仍然可能出现聚集。
4. 双重哈希（Double Hashing）：
   - 双重哈希是一种封闭地址法，它使用两个不同的哈希函数来确定下一个探测位置。
   - 当哈希冲突发生时，它首先使用第一个哈希函数来计算下一个槽位，如果该槽位已被占用，则使用第二个哈希函数来计算下一个槽位。
   - 双重哈希法可以减少聚集问题，但需要精心选择和设计哈希函数。
5. 再哈希（Rehashing）：
   - 当哈希表中的负载因子（已占用槽位数与总槽位数之比）达到一定阈值时，可以进行再哈希。
   - 再哈希是将哈希表的大小扩展一倍，并重新将所有元素插入新的表中。这可以减小负载因子，减少哈希冲突的可能性。
6. 建立一个好的哈希函数：
   - 最好的方法是设计一个能够尽可能减少冲突的哈希函数。好的哈希函数应该均匀地分散键，使哈希表的槽位均匀利用。
7. 分离链接哈希表（Separate Chaining Hash Table）：
   - 这种方法是将哈希表的每个槽位都构建为一个独立的数据结构，如链表、树或其他哈希表。这样，即使发生冲突，也能够高效地存储多个值。

### 14、快速排序是稳定排序吗，插入排序是稳定排序吗？



## 操作系统

### 15、用户态和内核态的区别？除了系统调用还有哪些情况会切换用户态和内核态？

### 16、零拷贝了解吗？

### 17、知道什么是写时拷贝吗？

### 18、知道什么是缺页中断吗？

### 19、常见的页面替换算法有哪些？

## 网络

### 20、每次https请求的过程中会发生什么？

### 21、tls握手的过程详细讲一下？

### 22、具体有哪些对称加密算法和非对称加密算法？

### 23、你知道什么是mtu吗？

### 24、针对游戏开发你觉得tcp和udp哪个更好？

### 25、你说udp允许一定的丢包的话，那玩家释放的技能被丢包了造成不好的体验了怎么办？

## 手撕

### 26、有一个链表，比如说12345，然后希望重排序成15243？