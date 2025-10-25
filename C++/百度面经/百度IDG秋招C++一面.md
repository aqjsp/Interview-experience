# 百度IDG秋招C++一面

> 来源：https://www.nowcoder.com/feed/main/detail/47263f74efb24dacaabc86596cfc5433

## C++篇

### 1、既然了解C++，请说出C++虚函数的定义？

在C++中，虚函数是一种允许在基类中定义一个函数，并在派生类中重新定义它的机制。通过虚函数，C++实现了运行时多态性，即通过基类指针或引用调用派生类的函数版本。虚函数是通过在函数声明前加上关键字 `virtual` 来实现的。

#### 虚函数的定义和使用

##### 基本定义

虚函数在基类中使用 `virtual` 关键字进行声明：

```c++
class Base {
public:
    virtual void show() {
        std::cout << "Base class show function" << std::endl;
    }
    virtual ~Base() {}  // 虚析构函数，确保正确调用派生类的析构函数
};

class Derived : public Base {
public:
    void show() override {  // 使用 override 关键字可以提高代码的可读性和安全性
        std::cout << "Derived class show function" << std::endl;
    }
};
```

##### 使用虚函数实现多态

通过基类指针或引用调用虚函数时，将会根据实际对象的类型调用相应的函数版本：

```c++
int main() {
    Base* b = new Derived();
    b->show();  // 输出 "Derived class show function"
    delete b;
    return 0;
}
```

#### 虚函数的机制

虚函数的实现依赖于虚函数表（VTable）和虚函数指针（VPTR）。每个含有虚函数的类（以及从该类派生的所有类）都有一个虚函数表。虚函数表是一个函数指针数组，每个指针指向该类的一个虚函数的实现。

- **VPTR（虚函数指针）**：在含有虚函数的对象中，编译器会在对象内存布局中添加一个指针（VPTR），指向该类的虚函数表。
- **VTable（虚函数表）**：这是一个函数指针数组，每个指针指向该类的一个虚函数的实现。在多态调用中，编译器通过 VPTR 查找对象对应的虚函数表，并调用相应的函数。

#### 虚函数的优点

- 实现运行时多态性：通过虚函数，C++可以在运行时根据对象的实际类型调用相应的函数版本。
- 提高代码的可扩展性：虚函数使得类的继承和重用更加灵活，可以在派生类中重新定义基类中的函数，而不需要修改基类。

#### 虚函数的缺点

- 额外的内存开销：虚函数表和虚函数指针会增加类的内存占用。
- 函数调用开销：虚函数调用需要通过虚函数表进行间接调用，略微增加了函数调用的时间开销。

### 2、C++面向对象三大特性是什么？

#### 1. 封装

定义：封装是将数据和操作数据的函数（方法）绑定在一起，形成一个独立的单元（类），并将这些数据隐藏起来，只通过对象的方法来访问和修改数据。

优点：

- 数据保护：通过访问控制修饰符（`private`, `protected`, `public`）保护数据不被外部代码直接访问和修改。
- 代码组织：将相关的数据和操作封装在一个类中，提高代码的可读性和维护性。
- 灵活性：通过公开的接口（`public` 方法），类的内部实现可以随时更改，而不影响外部代码。

示例：

```c++
class Rectangle {
private:
    double length;
    double width;

public:
    void setLength(double len) {
        length = len;
    }

    void setWidth(double wid) {
        width = wid;
    }

    double getLength() {
        return length;
    }

    double getWidth() {
        return width;
    }

    double area() {
        return length * width;
    }
};
```

#### 2. 继承

定义：继承是指一个类（子类或派生类）从另一个类（基类或父类）获取属性和行为（方法），并可以添加新的属性和行为或重写基类的方法。

优点：

- 代码复用：通过继承，子类可以重用基类中的代码，减少代码重复。
- 扩展性：可以在子类中扩展基类的功能。
- 多态性：通过继承实现多态性。

示例：

```c++
class Shape {
public:
    void setColor(std::string c) {
        color = c;
    }

    std::string getColor() {
        return color;
    }

private:
    std::string color;
};

class Circle : public Shape {
public:
    void setRadius(double r) {
        radius = r;
    }

    double getRadius() {
        return radius;
    }

private:
    double radius;
};
```

#### 3. 多态

定义：多态是指同一个函数名或操作符可以在不同的上下文中有不同的实现方式。多态有两种形式：编译时多态（函数重载和运算符重载）和运行时多态（虚函数）。

优点：

- 接口统一：可以通过基类接口处理不同的子类对象。
- 灵活性和可扩展性：在不修改已有代码的情况下，通过新增子类来实现新的功能。

示例：

```c++
class Shape {
public:
    virtual void draw() {
        std::cout << "Drawing a shape" << std::endl;
    }
};

class Circle : public Shape {
public:
    void draw() override {
        std::cout << "Drawing a circle" << std::endl;
    }
};

class Rectangle : public Shape {
public:
    void draw() override {
        std::cout << "Drawing a rectangle" << std::endl;
    }
};

void displayShape(Shape* shape) {
    shape->draw();
}

int main() {
    Shape* s1 = new Circle();
    Shape* s2 = new Rectangle();

    displayShape(s1); // 输出 "Drawing a circle"
    displayShape(s2); // 输出 "Drawing a rectangle"

    delete s1;
    delete s2;

    return 0;
}
```

### 3、多态是什么？多态具体操作是什么？

多态（Polymorphism）是面向对象编程中的一个重要概念，指的是同一个操作作用于不同的对象，可以有不同的行为表现。具体来说，多态性使得可以通过统一的接口来访问不同类的对象，这些对象可以有不同的实现方式，从而使得程序能够根据对象的实际类型来调用相应的方法。

#### 多态的实现方式

在面向对象编程中，多态性主要通过两种方式实现：编译时多态（静态多态）和运行时多态（动态多态）。

##### 1. 编译时多态（静态多态）

编译时多态通过函数重载（Overloading）和运算符重载（Operator Overloading）来实现。

- 函数重载：同一个函数名可以有多个定义，编译器根据参数的类型、顺序、数量来决定调用哪个函数。

```c++
class Calculator {
public:
    int add(int a, int b) {
        return a + b;
    }

    double add(double a, double b) {
        return a + b;
    }
};
```

在上述示例中，`add` 方法被重载了，可以处理整数和浮点数类型的参数。

- 运算符重载：对C++中的运算符（如`+`, `-`, `*`, `/` 等）进行重载，使其能够用于类对象。

```c++
class Complex {
public:
    Complex operator+(const Complex& other) {
        Complex result;
        result.real = this->real + other.real;
        result.imaginary = this->imaginary + other.imaginary;
        return result;
    }

private:
    double real;
    double imaginary;
};
```

通过重载 `+` 运算符，使得 `Complex` 类对象可以直接使用 `+` 运算符进行相加操作。

##### 2. 运行时多态（动态多态）

运行时多态通过虚函数（Virtual Function）和继承来实现。

- 虚函数：在基类中声明虚函数，在派生类中可以重写（覆盖）这些虚函数，通过基类指针或引用调用这些虚函数时，根据实际对象的类型来决定调用哪个函数。

```c++
class Shape {
public:
    virtual void draw() {
        std::cout << "Drawing a shape" << std::endl;
    }
};

class Circle : public Shape {
public:
    void draw() override {
        std::cout << "Drawing a circle" << std::endl;
    }
};

class Rectangle : public Shape {
public:
    void draw() override {
        std::cout << "Drawing a rectangle" << std::endl;
    }
};

int main() {
    Shape* shape1 = new Circle();
    Shape* shape2 = new Rectangle();

    shape1->draw(); // 输出 "Drawing a circle"
    shape2->draw(); // 输出 "Drawing a rectangle"

    delete shape1;
    delete shape2;

    return 0;
}
```

### 4、继承的public、protected、private三个关键字说一下有什么用？

#### 1. `public` 继承

当一个类以 `public` 方式继承另一个类时，基类的 `public` 成员在派生类中保持为 `public`，基类的 `protected` 成员在派生类中保持为 `protected`，基类的 `private` 成员在派生类中依然不可访问。

```c++
class Base {
public:
    int publicMember;
protected:
    int protectedMember;
private:
    int privateMember;
};

class Derived : public Base {
    // publicMember 是 public
    // protectedMember 是 protected
    // privateMember 是不可访问的
};
```

#### 2. `protected` 继承

当一个类以 `protected` 方式继承另一个类时，基类的 `public` 成员和 `protected` 成员在派生类中都变成 `protected`，基类的 `private` 成员在派生类中依然不可访问。

```c++
class Derived : protected Base {
    // publicMember 是 protected
    // protectedMember 是 protected
    // privateMember 是不可访问的
};
```

#### 3. `private` 继承

当一个类以 `private` 方式继承另一个类时，基类的 `public` 成员和 `protected` 成员在派生类中都变成 `private`，基类的 `private` 成员在派生类中依然不可访问。

```c++
class Derived : private Base {
    // publicMember 是 private
    // protectedMember 是 private
    // privateMember 是不可访问的
};
```

#### 进一步说明

1. `public` 继承：这种继承方式通常用于表示 "is-a" 关系。例如，`Bird` 是一个 `Animal`。使用 `public` 继承可以确保基类的接口在派生类中仍然可用。
2. `protected` 继承：这种继承方式较少使用，它用于当你想要限制基类的接口在派生类的外部不可见，但在派生类的内部和进一步派生的类中仍然可见时。例如，`Component` 是 `Entity` 的一个受保护的部分，但外部不应直接访问 `Component`。
3. `private` 继承：这种继承方式通常用于表示 "implemented-in-terms-of" 关系，即派生类是基类的实现细节，而不是一种类型关系。例如，`Stack` 可以通过 `List` 实现，但 `Stack` 并不表示一种 `List`。

### 5、override 和 overload关系是什么？

#### Overloading (重载)

**重载**是指在同一个作用域中定义多个同名函数，但这些函数的参数列表不同（参数的个数或类型不同）。重载是编译时的行为，编译器通过参数列表来区分不同的函数。

##### 示例：

```c++
class Example {
public:
    void func(int a) {
        std::cout << "Function with int parameter" << std::endl;
    }

    void func(double a) {
        std::cout << "Function with double parameter" << std::endl;
    }

    void func(int a, double b) {
        std::cout << "Function with int and double parameters" << std::endl;
    }
};

int main() {
    Example ex;
    ex.func(10);          // 调用 void func(int)
    ex.func(10.5);        // 调用 void func(double)
    ex.func(10, 10.5);    // 调用 void func(int, double)
    return 0;
}
```

在上面的例子中，`func` 函数被重载了三次，每次的参数列表不同。

#### Overriding (重写)

**重写**是指在派生类中重新定义基类中已存在的虚函数。重写是运行时的行为，通常用于实现多态性。派生类中的函数必须与基类中的虚函数具有相同的名称、参数列表和返回类型。

为了显式地表明派生类中的函数是重写基类中的虚函数，C++11引入了`override`关键字。

##### 示例：

```c++
class Base {
public:
    virtual void show() {
        std::cout << "Base class show function" << std::endl;
    }
};

class Derived : public Base {
public:
    void show() override {  // 重写基类的虚函数
        std::cout << "Derived class show function" << std::endl;
    }
};

int main() {
    Base* basePtr;
    Derived d;
    basePtr = &d;
    basePtr->show();  // 调用 Derived 类的 show 函数
    return 0;
}
```

在上面的例子中，`Derived` 类中的 `show` 函数重写了 `Base` 类中的 `show` 函数。

#### Overloading 与 Overriding 的区别与关系

- **定义范围**：
  - 重载：在同一个类或同一个作用域中定义多个同名函数，参数列表不同。
  - 重写：在派生类中重新定义基类中的虚函数，参数列表必须相同。
- **关键字**：
  - 重载：不需要任何特殊关键字。
  - 重写：可以使用`override`关键字来显式地标明。
- **行为**：
  - 重载：是编译时多态，通过函数签名（名称和参数列表）区分不同的函数。
  - 重写：是运行时多态，通过虚函数机制实现，基类的指针或引用调用重写的函数。
- **作用**：
  - 重载：提高函数的灵活性和可读性，使函数可以处理不同类型或数量的参数。
  - 重写：实现多态性，使基类的指针或引用可以调用派生类中的实现。

### 6、子类访问父类能访问哪些属性？

可以参考上面三个关键字。

## 计算机基础篇

### 1、TCP与UDP的区别是什么分别说出来？

#### TCP（传输控制协议）

1. 连接导向：
   - TCP 是一种面向连接的协议。在数据传输之前，必须先建立连接，然后进行数据传输，传输结束后再释放连接。
   - 使用三次握手建立连接（SYN，SYN-ACK，ACK），四次挥手释放连接（FIN，ACK，FIN-ACK，ACK）。
2. 可靠性：TCP 提供可靠的数据传输。它通过序列号、确认应答、重传机制、流量控制和拥塞控制来保证数据的可靠性和顺序性。
3. 顺序保证：TCP 保证数据按照发送顺序到达接收端，并且不会出现丢失、重复或乱序。
4. 适用场景：适用于需要确保数据完整性和顺序的应用，如文件传输、电子邮件、Web 浏览等。
5. 传输效率：由于提供了可靠性保证和数据顺序性，TCP 在传输效率上相对于UDP略低，尤其是在网络拥塞时，TCP 的拥塞控制可能会导致传输速率的下降。
6. 头部开销：TCP 头部相对较大，包含序列号、确认号、窗口大小等信息，因此头部开销较大。

#### UDP（用户数据报协议）

1. 无连接：UDP 是一种无连接的协议，通信双方在传输数据时不需要先建立连接，也不需要断开连接。
2. 不可靠性：UDP 不保证数据传输的可靠性，数据包可能丢失、重复或者无序到达。
3. 适用场景：
   - 适用于实时性要求高、对可靠性要求较低的应用，如音频、视频流媒体、在线游戏等。
   - 因为不需要建立和释放连接，UDP 可以减少一些开销，传输速度相对较快。
4. 传输效率：UDP 在传输效率上通常比TCP高，因为它不需要等待连接建立和拥塞控制的反馈。
5. 头部开销：UDP 头部相对较小，只包含源端口、目标端口、长度和校验和等基本信息，头部开销小。

### 2、TCP三握四挥具体讲一下（典中典）

#### 三次握手

![三次握手](https://cdn.jsdelivr.net/gh/aqjsp/Pictures/image-20240806010331027.png)

在建立连接之前，Client处于CLOSED状态，而Server处于LISTEN的状态。

1. 第一次握手：客户端主动给服务端发送一个SYN报文，并携带自己的初始化序列号一起发送给服务端。此时客户端处于一个SYN_SEND的状态。
2. 第二次握手：服务端收到客户端发来的SYN报文之后，就会以自己的SYN报文作为应答，然后将自己的初始化序列号发送给客户端，并且会将客户端的初始化序列号+1作为自己的ACK值发送给客户端，以表示自己已经收到了客户端的SYN报文。此时服务端处于一个SYN_RECV的状态。
3. 第三次握手：客户端收到服务端发来的SYN报文之后，会把服务端的初始化序列号+1作为ACK值发送给服务端，用来表示自己已经收到了服务端发来的SYN报文。此时客户端处于一个ESTABLISHED的状态。

#### 四次挥手

![四次挥手](https://cdn.jsdelivr.net/gh/aqjsp/Pictures/image-20240806010407777.png)

1. 第一次挥手（FIN-1）：
   - 客户端发送一个 FIN 报文段给服务器，表示客户端已经没有数据要发送了，请求关闭连接。
   - 客户端进入 FIN_WAIT_1 状态，等待服务器的确认。
2. 第二次挥手（ACK）：
   - 服务器收到客户端的 FIN 报文段后，发送一个 ACK 报文段作为应答，表示已经接收到了客户端的关闭请求。
   - 服务器进入 CLOSE_WAIT 状态，等待自己的数据发送完毕。
3. 第三次挥手（FIN-2）：
   - 服务器发送一个 FIN 报文段给客户端，表示服务器也没有数据要发送了，请求关闭连接。
   - 服务器进入 LAST_ACK 状态，等待客户端的确认。
4. 第四次挥手（ACK）：
   - 客户端收到服务器的 FIN 报文段后，发送一个 ACK 报文段作为应答，表示已经接收到了服务器的关闭请求。
   - 客户端进入 TIME_WAIT 状态，等待可能出现的延迟数据。
   - 服务器收到客户端的 ACK 报文段后，完成关闭，进入 CLOSED 状态。
   - 客户端在 TIME_WAIT 状态结束后，关闭连接，进入 CLOSED 状态。

### 3、请你讲一下浏览器登录网址，中间所有的过程？

#### 1. 输入网址和检查缓存

- 用户在浏览器中输入 URL（例如，http://www.xxx.com）。
- 浏览器首先检查本地缓存（包括浏览器缓存、系统缓存等），看看是否有对应的 DNS 记录或网页内容。

#### 2. DNS 解析

如果缓存中没有找到，浏览器向 DNS 服务器发送请求，以解析域名为 IP 地址。

- 浏览器首先检查本地 DNS 缓存。
- 如果本地没有，浏览器会向操作系统发起请求，操作系统检查自己的缓存。
- 若系统缓存也没有，操作系统会向配置的 DNS 服务器发送请求。
- DNS 服务器递归查询，最终返回域名对应的 IP 地址。

#### 3. 建立连接

浏览器获得 IP 地址后，通过 IP 地址和目标服务器建立连接。

- 对于 HTTP，浏览器通过三次握手建立 TCP 连接。
- 对于 HTTPS，首先进行 TCP 三次握手，然后进行 TLS/SSL 握手以建立安全连接。

#### 4. 发送 HTTP/HTTPS 请求

连接建立后，浏览器构造 HTTP/HTTPS 请求并发送给服务器。

- 请求行：包含请求方法（GET、POST 等）、URL 和 HTTP 版本。
- 请求头：包含主机名、用户代理、接受的内容类型等信息。
- 请求体：包含数据（对于 GET 请求通常为空，对于 POST 请求则包含提交的数据）。

#### 5. 服务器处理请求

服务器接收到请求后，处理请求，通常包括：

- 检查请求方法。
- 验证权限。
- 查询数据库或其他存储系统获取数据。
- 处理动态内容（例如，运行后端代码）。
- 构建响应。

#### 6. 服务器返回响应

服务器将处理结果作为响应返回给浏览器，响应包括：

- 响应行：包含 HTTP 版本、状态码和状态描述。
- 响应头：包含内容类型、内容长度、服务器信息等。
- 响应体：包含实际的网页内容（HTML、CSS、JavaScript、图片等）。

#### 7. 浏览器处理响应

浏览器接收响应后，检查响应状态码。

- 如果状态码是 200，表示成功。
- 如果状态码是 3xx，表示重定向，浏览器会重新发起请求。
- 如果状态码是 4xx 或 5xx，表示错误，浏览器会显示错误页面。

#### 8. 渲染页面

- 浏览器解析 HTML 文档，构建 DOM 树。
- 解析 CSS，构建 CSSOM 树。
- 解析 JavaScript，并执行脚本。
- 根据 DOM 树和 CSSOM 树构建渲染树。
- 布局渲染树，计算每个节点的位置和大小。
- 绘制页面，将内容显示在屏幕上。

### 4、OSI网络参考七层模型都是什么？

![OSI七层模型](https://cdn.jsdelivr.net/gh/aqjsp/Pictures/image-20240806010912681.png)

1. **物理层（Physical Layer）**

   - 功能：传输原始的比特流，通过物理媒介（如电缆、光纤、无线电波）在设备间传递比特。

   - 设备：集线器、网卡、物理电缆。

   - 协议和标准：IEEE 802.3（以太网）、IEEE 802.11（Wi-Fi）。

2. **数据链路层（Data Link Layer）**

   - 功能：将数据打包成帧，进行节点间的数据传输，检测并纠正物理层传输中的错误。

   - 设备：交换机、桥接器。

   - 协议和标准：Ethernet、PPP、HDLC、802.11。

3. **网络层（Network Layer）**

   - 功能：负责数据包的路由选择和转发，提供逻辑地址（如IP地址）。

   - 设备：路由器。

   - 协议和标准：IP（IPv4、IPv6）、ICMP、IGMP。

4. **传输层（Transport Layer）**

   - 功能：提供端到端的通信服务，确保数据传输的完整性和可靠性（如流量控制、错误检测和纠正）。

   - 协议和标准：TCP、UDP、SCTP。

5. **会话层（Session Layer）**

   - 功能：管理和控制通信会话，负责建立、维护和终止会话。

   - 协议和标准：NetBIOS、RPC。

6. **表示层（Presentation Layer）**

   - 功能：负责数据的格式化、加密和解密、数据压缩和解压缩。

   - 协议和标准：JPEG、MPEG、SSL/TLS。

7. **应用层（Application Layer）**

   - 功能：为应用程序提供网络服务接口，支持网络应用（如电子邮件、文件传输、网络浏览等）。

   - 协议和标准：HTTP、FTP、SMTP、DNS、SNMP。

### 5、TCP除了你介绍的还有什么机制？

#### 1. 数据重传机制

TCP 使用超时重传机制和快速重传来确保数据的可靠传输。每当发送数据时，发送方会启动一个定时器，如果在超时时间内没有收到确认应答（ACK），发送方会重传数据包。快速重传机制允许在收到重复的 ACK 时，立即重传丢失的数据包，而不必等待超时。

#### 2. 顺序控制

TCP 确保数据包的顺序。每个数据包都有一个序列号，接收方根据序列号重新排列数据包，确保数据的顺序正确。

#### 3. 流量控制

使用滑动窗口协议来控制流量。接收方通过广告窗口大小告诉发送方它能够接收的缓冲区大小，从而防止发送方过快地发送数据，避免接收方的缓冲区溢出。

#### 4. 连接管理

TCP 使用三次握手（Three-Way Handshake）来建立连接，确保双方都准备好进行数据传输。四次挥手（Four-Way Handshake）用于正常关闭连接，确保所有数据都被传输并且连接被正确关闭。

#### 5. 拥塞控制

TCP 拥塞控制机制通过调整数据传输速率来防止网络拥塞。主要算法包括慢启动、拥塞避免、快速重传和快速恢复等。慢启动算法逐渐增加窗口大小来适应网络条件，而拥塞避免算法在达到一定程度后进行缓慢的窗口增加。

#### 6. 数据完整性

TCP 使用校验和来确保数据在传输过程中没有被损坏。每个 TCP 数据包都包含一个校验和字段，用于验证数据的完整性。如果校验和计算结果不匹配，接收方会丢弃数据包并请求重传。

#### 7. 全双工通信

TCP 支持全双工通信，允许数据同时在两个方向上流动。这意味着通信的两端都可以同时发送和接收数据。

#### 8. 流量整形

TCP 使用流量整形来平滑数据流量，减少网络突发流量对网络的影响。通过调整数据发送的速率和数据包的大小，减少对网络的压力。

#### 9. 连接复用

TCP 支持在同一个连接上进行多个请求和响应，减少了频繁建立和断开连接的开销。

#### 10. 数据包分段与重组

TCP 将大块数据分割成适合网络传输的小段，并在接收端重新组装成原始数据。这使得 TCP 能够在不同的网络条件下处理不同大小的数据块。

#### 11. 流量窗口

TCP 使用流量窗口来调整发送数据的速率，基于接收方的处理能力来控制发送的数据量。

### 6、https是什么？

HTTPS（HyperText Transfer Protocol Secure）是一种在 HTTP（超文本传输协议）基础上增加了安全层的协议，用于保护网络通信的安全性。HTTPS 通过加密技术确保数据在客户端和服务器之间的传输是安全的，防止数据在传输过程中被窃取或篡改。

#### 主要特性

1. **加密**：

   - 数据加密：HTTPS 使用加密算法（如 SSL/TLS）对数据进行加密。加密过程确保数据在传输过程中无法被第三方读取或篡改。
   - 密钥交换：在建立 HTTPS 连接时，客户端和服务器会通过加密协议交换密钥，这些密钥用于加密和解密传输的数据。

2. **数据完整性**：

   消息认证码（MAC）：HTTPS 使用消息认证码来确保数据的完整性。消息认证码能检测数据在传输过程中是否被篡改。

3. **身份验证**：

   数字证书：HTTPS 使用数字证书来验证服务器的身份。数字证书由受信任的证书颁发机构（CA）签发，客户端可以通过验证证书来确保连接的是合法的服务器。

4. **防止中间人攻击**：

   中间人攻击保护：由于数据是加密的，攻击者无法轻易地解密和篡改传输中的数据，从而防止了中间人攻击。

#### HTTPS 的工作原理

1. **建立连接**：

   客户端请求：客户端（通常是浏览器）发起 HTTPS 请求，连接到服务器的指定端口（通常是 443）。

2. **服务器响应**：

   证书交换：服务器响应客户端的请求，并发送其数字证书给客户端。数字证书包含公钥和服务器的身份信息。

3. **验证证书**：

   证书验证：客户端验证服务器的数字证书，确保证书由受信任的证书颁发机构签发，并且证书没有过期或被撤销。

4. **密钥交换**：

   生成会话密钥：客户端和服务器通过使用证书中包含的公钥交换密钥，生成一个对称加密的会话密钥。这个会话密钥用于后续的数据加密和解密。

5. **加密通信**：

   数据加密：使用会话密钥对传输的数据进行加密。加密数据通过网络传输，保证数据的安全性。

6. **数据解密**：

   数据解密：接收方使用相同的会话密钥对加密的数据进行解密，恢复数据的原始内容。

#### HTTPS 与 HTTP 的区别

- **加密**：HTTP 是明文传输的，不加密数据；HTTPS 使用加密技术保护数据的安全性。
- **端口**：HTTP 使用端口 80，而 HTTPS 使用端口 443。
- **性能开销**：HTTPS 由于加密和解密的过程，通常会比 HTTP 有更高的性能开销。

#### HTTPS 的优势

- **数据保护**：防止数据在传输过程中被窃取或篡改。
- **身份验证**：确保与正确的服务器建立连接，防止钓鱼攻击。
- **隐私保护**：保护用户的隐私，防止第三方窥探用户的活动。

## 编程题

### 1、请你介绍什么叫快速排序，quickSort给定数组将其中元素进行快速排序并打印。

快速排序（QuickSort）是一种高效的排序算法，它采用分治法（Divide and Conquer）的策略来排序。

#### 基本思想

1. 选择一个基准元素（pivot）：从数组中选择一个元素作为基准。
2. 分区（Partition）：将数组重新排列，使得所有小于基准元素的元素排在基准元素的左侧，所有大于基准元素的元素排在基准元素的右侧。
3. 递归排序：对基准元素左右两边的子数组递归地应用快速排序。

#### 快速排序的步骤

1. 选择基准：可以选择数组的第一个元素、最后一个元素、随机一个元素，或使用更复杂的选择方法（如“三数取中”）来作为基准。
2. 分区操作：重排数组，使得基准元素的左边都是比基准小的元素，右边都是比基准大的元素。
3. 递归调用：对基准元素左边和右边的子数组分别进行快速排序。

#### 时间复杂度

- 最优时间复杂度：`O(n log n)`，当每次分区操作都能将数组均匀地分成两部分时。
- 平均时间复杂度：`O(n log n)`，在大多数情况下。
- 最坏时间复杂度：`O(n^2)`，当选择的基准元素总是数组中的最大或最小元素时（即数组已经是有序的）。

#### C++实现

```c++
#include <iostream>
#include <vector>
using namespace std;

// 快速排序的分区函数
int partition(vector<int>& arr, int low, int high) {
    int pivot = arr[high];  // 选择基准元素
    int i = low - 1;  // i是小于基准元素的区域的最后一个元素的索引

    // 遍历数组，进行分区操作
    for (int j = low; j < high; ++j) {
        if (arr[j] < pivot) {  // 如果当前元素小于基准元素
            ++i;
            swap(arr[i], arr[j]);  // 交换元素
        }
    }
    swap(arr[i + 1], arr[high]);  // 将基准元素放到正确的位置
    return i + 1;  // 返回基准元素的索引
}

// 快速排序的递归函数
void quickSort(vector<int>& arr, int low, int high) {
    if (low < high) {
        int pi = partition(arr, low, high);  // 获取基准元素的索引
        quickSort(arr, low, pi - 1);  // 对左半部分进行排序
        quickSort(arr, pi + 1, high);  // 对右半部分进行排序
    }
}

// 打印数组
void printArray(const vector<int>& arr) {
    for (int num : arr) {
        cout << num << " ";
    }
    cout << endl;
}

int main() {
    vector<int> array = {10, 7, 8, 9, 1, 5};  // 示例数组

    cout << "Original array: ";
    printArray(array);  // 打印原始数组

    quickSort(array, 0, array.size() - 1);  // 调用快速排序

    cout << "Sorted array: ";
    printArray(array);  // 打印排序后的数组

    return 0;
}
```

## 反问