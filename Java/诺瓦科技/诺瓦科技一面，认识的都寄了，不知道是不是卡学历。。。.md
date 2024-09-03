# 诺瓦科技一面，认识的都寄了，不知道是不是卡学历。。。



> 来源：https://www.nowcoder.com/share/jump/1724944277139

### 1、Java 面向对象？

#### 1. 抽象（Abstraction）

抽象是指隐藏对象的复杂实现细节，只对外提供必要的接口。这有助于减少复杂性，让用户只关注高层次的操作。

- **类（Class）**：类是一个蓝图，它定义了对象的属性和行为。通过类可以创建对象，每个对象都是该类的实例。
- **接口（Interface）**：接口是一种特殊的类，它只包含方法的声明，不包含方法的实现。类通过实现接口来提供具体的方法实现。接口的主要作用是定义一组公共的行为规范，类通过实现这些接口来约束自身行为。

```
abstract class Animal {
    abstract void makeSound(); // 抽象方法，没有方法体
}

class Dog extends Animal {
    @Override
    void makeSound() {
        System.out.println("Woof");
    }
}
```

#### 2. 封装（Encapsulation）

封装是将对象的状态（属性）和行为（方法）捆绑在一起，并隐藏对象的内部实现细节。封装通过访问修饰符（`private`, `protected`, `public`）来控制对类成员的访问，从而保护对象的状态。

- **私有属性（Private Fields）**：使用 `private` 关键字将属性设为私有，防止外部直接访问。
- **公共方法（Public Methods）**：通过提供公共的 getter 和 setter 方法来控制对属性的访问。

```
public class Person {
    private String name;  // 私有属性
    private int age;

    public String getName() {  // 公共 getter 方法
        return name;
    }

    public void setName(String name) {  // 公共 setter 方法
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        if (age > 0) {
            this.age = age;
        }
    }
}
```

#### 3. 继承（Inheritance）

继承是面向对象编程的一种机制，它允许一个类继承另一个类的属性和方法，从而实现代码的重用和扩展。通过继承，可以创建一个更具体的子类，继承一个更抽象的父类的特征。

- **父类（Superclass）**：提供基本功能和属性的类。
- **子类（Subclass）**：继承父类的属性和方法，可以扩展和覆盖父类的方法。

```
class Animal {
    void eat() {
        System.out.println("This animal eats food");
    }
}

class Dog extends Animal {
    void bark() {
        System.out.println("Woof");
    }
}
```

#### 4. 多态（Polymorphism）

多态性是指相同的接口可以有不同的实现方式。在 Java 中，多态性通过方法重载（overloading）和方法重写（overriding）实现。

- **方法重载（Overloading）**：在同一个类中，方法名相同但参数列表不同。
- **方法重写（Overriding）**：在子类中重新定义父类中的方法。

```
class Animal {
    void makeSound() {
        System.out.println("Some sound");
    }
}

class Dog extends Animal {
    @Override
    void makeSound() {
        System.out.println("Woof");
    }
}

public class Main {
    public static void main(String[] args) {
        Animal myAnimal = new Dog();  // 动态多态性
        myAnimal.makeSound();  // 输出 "Woof"
    }
}
```

### 2、Object 常用的方法？

#### 1. `equals(Object obj)`

- **作用**: 比较两个对象是否相等。默认实现是比较两个对象的内存地址（引用相等性）。
- **用法**: 需要在子类中重写以实现自定义的相等性逻辑（例如，比较对象的内容）。

```
public class Person {
    private String name;
    
    public Person(String name) {
        this.name = name;
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;  // 检查引用相等性
        if (obj == null || getClass() != obj.getClass()) return false;  // 检查是否为相同类
        Person person = (Person) obj;
        return name.equals(person.name);  // 自定义内容相等性
    }
}
```

#### 2. `hashCode()`

- **作用**: 返回对象的哈希码值，哈希码是支持散列存储（例如 `HashMap`）的机制。
- **用法**: 如果重写了 `equals()` 方法，就必须重写 `hashCode()` 方法，以确保相等的对象具有相同的哈希码。

```
@Override
public int hashCode() {
    return name.hashCode();
}
```

#### 3. `toString()`

- **作用**: 返回对象的字符串表示。默认实现是返回对象的类名和其内存地址的哈希码。
- **用法**: 通常在子类中重写以提供更有意义的字符串表示（例如，用于调试或日志记录）。

```
@Override
public String toString() {
    return "Person{name='" + name + "'}";
}
```

#### 4. `getClass()`

- **作用**: 返回对象的运行时类信息。
- **用法**: 用于反射操作或在运行时获取类的详细信息。

```
Person person = new Person("Alice");
Class<?> clazz = person.getClass();
System.out.println(clazz.getName());  // 输出: Person
```

#### 5. `clone()`

- **作用**: 创建并返回对象的一个浅拷贝。此方法会抛出 `CloneNotSupportedException`，所以对象必须实现 `Cloneable` 接口。
- **用法**: 用于对象复制。重写此方法时需注意深拷贝与浅拷贝的区别。

```
public class Person implements Cloneable {
    private String name;
    
    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();  // 使用 Object 的 clone 方法
    }
}
```

#### 6. `finalize()`

- **作用**: 当垃圾回收器确定没有对该对象的更多引用时，由对象的垃圾收集器调用。用来在对象被垃圾收集之前进行清理操作。
- **用法**: 一般不推荐使用，因其不确定性和性能影响。更好的做法是使用 `try-with-resources` 或显式的资源管理。

```
@Override
protected void finalize() throws Throwable {
    try {
        // 清理操作
    } finally {
        super.finalize();
    }
}
```

#### 7. `wait()`、`notify()`、`notifyAll()`

- **作用**: 这些方法用于线程间通信。`wait()` 会让当前线程等待，直到另一个线程调用同一对象的 `notify()` 或 `notifyAll()` 方法。
- **用法**: 必须在同步块或同步方法内调用，否则会抛出 `IllegalMonitorStateException`。

```
synchronized (obj) {
    obj.wait();  // 当前线程等待
    obj.notify();  // 唤醒一个等待线程
    obj.notifyAll();  // 唤醒所有等待线程
}
```

### 3、TCP如何保证可靠？

#### 1. 连接建立

TCP 使用三次握手过程来建立连接，以确保双方都准备好进行通信。这三次握手的过程：

![三次握手](https://cdn.jsdelivr.net/gh/aqjsp/Pictures/image-20240829234826848.png)

1. **SYN**: 客户端向服务器发送一个 SYN（Synchronize）报文，表示请求建立连接。
2. **SYN-ACK**: 服务器收到 SYN 后，返回一个 SYN-ACK（Synchronize Acknowledgment）报文，表示同意建立连接并确认收到客户端的 SYN。
3. **ACK**: 客户端收到 SYN-ACK 后，发送一个 ACK（Acknowledgment）报文，表示确认连接建立。

通过三次握手，TCP 确保了双方都收到了连接请求，并为即将开始的数据传输建立了可靠的连接。

#### 2. 数据传输和确认

TCP 使用序列号（Sequence Number）和确认号（Acknowledgment Number）来跟踪数据的传输和确认状态。

- **序列号**：每个 TCP 数据包都有一个序列号，它表示该包在整个数据流中的位置。序列号用于确保数据包按顺序到达。
- **确认号**：接收方发送一个 ACK 包，包含下一个期望收到的字节的序列号，确认它已成功接收到所有先前的数据包。

这种机制确保了发送方能够知道接收方是否成功接收了数据。如果数据包丢失或损坏，接收方不会发送相应的确认，发送方将重传数据。

#### 3. 重传机制

TCP 实现了一个超时和重传机制，以确保丢失的数据包能够重新发送。

- **超时计时器**：发送方在发送一个数据包后启动一个计时器，如果在指定时间内没有收到确认，发送方会重传该数据包。
- **快速重传**：如果接收方接收到一个乱序的数据包（例如，由于丢失的包），它会发送重复的 ACK（称为冗余 ACK）。当发送方收到三个重复的 ACK 后，它就会立即重传丢失的数据包，而不需要等待超时。

#### 4. 流量控制

TCP 使用滑动窗口（Sliding Window）机制来控制数据的流动，确保发送方不会淹没接收方。

- **滑动窗口**：发送方维护一个窗口，表示它可以在等待 ACK 的同时发送的最大数据量。窗口大小由接收方确定，并包含在每个 ACK 包中。
- **接收窗口：接收方通过广告接收窗口的大小来告知发送方它可以接受的数据量，这样发送方就不会发送超过接收方处理能力的数据量。

流量控制确保了数据传输的稳定性，避免了网络拥塞和接收方缓冲区溢出。

#### 5. 拥塞控制

TCP 拥塞控制机制有助于防止网络拥塞，这些机制包括：

- **慢启动（Slow Start）**：发送方在刚开始传输数据时逐步增加拥塞窗口（Congestion Window，CWND）的大小，从小数据量开始发送，并逐渐加速，避免一开始就发送大量数据导致网络拥塞。
- **拥塞避免（Congestion Avoidance）**：当 CWND 超过慢启动阈值（ssthresh）时，进入拥塞避免阶段，CWND 增长速度变缓。
- **快速重传与快速恢复（Fast Retransmit and Fast Recovery）**：通过快速重传，发送方在检测到丢包时立即重传丢失的数据包，而不等待超时。快速恢复则允许 CWND 部分恢复，而不是完全回退到慢启动阶段。

#### 6. 数据校验（Checksum）

TCP 在每个数据包的头部包含一个校验和字段，用于检测数据传输过程中的错误。发送方计算校验和并将其包含在 TCP 报文头中，接收方收到数据后重新计算校验和并与报文中的校验和进行比较。如果校验和不匹配，数据包被认为已损坏，接收方将丢弃该包，并等待发送方重传。

#### 7. 连接终止（Four-way Handshake）

TCP 使用四次挥手过程来优雅地终止连接，确保所有数据都已成功传输：

![四次挥手](https://cdn.jsdelivr.net/gh/aqjsp/Pictures/image-20240830000155209.png)

1. **FIN**: 一方（如客户端）发送一个 FIN（Finish）报文，表示它已经完成发送数据。
2. **ACK**: 另一方（如服务器）收到 FIN 后，发送一个 ACK 报文，表示确认收到。
3. **FIN**: 服务器也发送一个 FIN 报文，表示它也完成了数据发送。
4. **ACK**: 客户端收到 FIN 后，发送最后一个 ACK，表示确认连接关闭。

这种机制确保双方都完成了数据传输并同意关闭连接。

### 4、单例模式？

#### 1. 懒汉式

##### 线程不安全

懒汉式单例模式在第一次调用时实例化对象。该方式实现简单，但不是线程安全的。

```
public class Singleton {
    private static Singleton instance;

    private Singleton() {
        // 私有构造函数
    }

    public static Singleton getInstance() {
        if (instance == null) {  // 第一次调用时才创建实例
            instance = new Singleton();
        }
        return instance;
    }
}
```

缺点: 在多线程环境下，可能会创建多个实例。

##### 线程安全

通过在 `getInstance` 方法上加同步锁（`synchronized`）来确保线程安全，但每次获取实例时都会进行同步，开销较大。

```
public class Singleton {
    private static Singleton instance;

    private Singleton() {
        // 私有构造函数
    }

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

缺点: 虽然线程安全，但效率低下，特别是在多次调用 `getInstance` 时。

#### 2. 饿汉式

饿汉式在类加载时就创建实例。这种方式天生是线程安全的，但可能会造成资源浪费。

```
public class Singleton {
    private static final Singleton instance = new Singleton();

    private Singleton() {
        // 私有构造函数
    }

    public static Singleton getInstance() {
        return instance;
    }
}
```

优点: 实现简单，线程安全。

缺点: 如果这个单例没有被使用，实例仍然会被创建，造成内存浪费。

#### 3. 双重检查锁

双重检查锁机制是在 `getInstance` 方法中减少使用同步块的范围，仅在实例未被创建时才同步，这样既保证了线程安全，又提高了效率。

```
public class Singleton {
    private static volatile Singleton instance;  // 使用 volatile 关键字保证线程间的可见性

    private Singleton() {
        // 私有构造函数
    }

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

优点: 线程安全，延迟加载，效率较高。

注意: 使用 `volatile` 关键字是为了防止指令重排序问题，确保在多线程环境下的安全性。

#### 4. 静态内部类

静态内部类方式利用了类加载机制来保证线程安全，同时实现了延迟加载。

```
public class Singleton {
    private Singleton() {
        // 私有构造函数
    }

    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```

优点: 实现简单，线程安全，延迟加载。`SingletonHolder` 类只有在 `getInstance` 方法被调用时才会被加载，确保了单例的延迟初始化。

#### 5. 枚举单例

使用枚举来实现单例是最佳的方法之一，因为它不仅能防止反射攻击，还能防止反序列化重新创建对象。

```
public enum Singleton {
    INSTANCE;

    public void doSomething() {
        // 自定义方法
    }
}
```

优点: 线程安全，防止反射攻击和反序列化。

### 5、jdk jvm jre 区别？

#### 1. JDK（Java Development Kit）

JDK（Java 开发工具包） 是用于开发 Java 应用程序的工具包。它包含了编写、编译、调试和运行 Java 应用程序所需的所有工具和资源。

**JDK 的组成部分**:

- JRE（Java 运行环境）: 用于运行 Java 程序的环境，包括 JVM 和 Java 核心类库。
- Java 编译器（`javac`）: 将 Java 源代码（.java 文件）编译为字节码（.class 文件）。
- 其他开发工具: 包括 `javadoc`（用于生成文档的工具）、`jar`（用于打包 Java 类文件的工具）、`jdb`（Java 调试器）等。

**用途**: JDK 是面向开发者的工具，用于编写和编译 Java 程序。每个 Java 开发者需要安装 JDK 以便开发和测试他们的 Java 应用。

#### 2. JRE（Java Runtime Environment）

JRE（Java 运行环境） 是用于运行 Java 应用程序的环境。它提供了 Java 程序在不同操作系统上执行所需的库和其他组件。

**JRE 的组成部分**:

- JVM（Java 虚拟机）: 执行 Java 字节码的虚拟机。
- 核心类库: 支持 Java 程序运行的类库（例如 `java.lang`、`java.util` 等）。

**用途**: JRE 是面向用户的环境，用于运行 Java 应用程序。对于不需要开发 Java 程序的普通用户来说，安装 JRE 即可运行 Java 应用程序。

#### 3. JVM（Java Virtual Machine）

JVM（Java 虚拟机） 是 JRE 的一个重要组成部分，它是一个虚拟机进程，负责将 Java 字节码（.class 文件）转换为机器码，以便操作系统执行。

**JVM 的主要功能**:

- 加载 Java 字节码。
- 验证 字节码的正确性和安全性。
- 解释/编译 字节码为机器码（即时编译器，JIT 编译）。
- 管理内存（包括堆内存和栈内存）和垃圾回收。
- 提供运行时环境：包括线程管理、安全性检查等。

**特点**:

- 平台无关性: JVM 是使 Java 具有跨平台特性的关键，因为同一个 Java 字节码可以在不同平台的 JVM 上运行。

**用途**: JVM 是 Java 程序的执行环境，负责将平台无关的 Java 字节码转换为特定平台的机器码，并执行这些代码。

#### 简而言之：

1. 如果你是 **Java 开发者**，你需要 **JDK**。
2. 如果你只是想 **运行 Java 应用程序**，你只需要 **JRE**。
3. **JVM** 是 Java 程序运行的核心，是 JRE 中的一部分，负责 Java 字节码的执行。

### 6、java 内存模型？

Java 内存模型（Java Memory Model，JMM）定义了 Java 程序在并发执行时的行为。它描述了多线程之间如何共享数据以及如何确保数据一致性。JMM 规范确保了 Java 程序在不同平台和 JVM 实现上的一致性。

#### 1. 主内存和工作内存

- 主内存：是所有 Java 变量（实例字段、静态字段和数组元素）的存储区域。主内存是线程之间共享的内存区域。所有线程都可以访问主内存中的数据。
- 工作内存：每个线程都有自己的工作内存，也称为线程栈。工作内存存储了主内存中变量的副本（缓存）。线程只能直接操作其工作内存中的数据，不能直接读写主内存的数据。线程对变量的所有操作（读取和赋值）都必须在工作内存中进行，而不是直接操作主内存中的变量。

线程间通信：线程通过各自的工作内存和主内存之间的数据同步来实现间接通信。一个线程将其工作内存中变量的修改刷新到主内存，另一个线程再从主内存中读取这个变量的最新值。

#### 2. 原子性、可见性和有序性

- 原子性：原子性保证一个操作不可被中断。即使是多线程环境下，一个操作一旦开始，就不会被其他线程打断。例如，Java 中的基本数据类型（`int`、`char` 等）赋值操作是原子的。
- 可见性：可见性保证了当一个线程修改了变量的值，新值对其他线程是立即可见的。Java 提供了 `volatile` 关键字来保证变量的可见性。被声明为 `volatile` 的变量会直接写入主内存，而每次读取时都会从主内存中读取最新的值。
- 有序性：有序性保证了程序的执行顺序按代码的顺序执行。Java 中的指令可能会因为编译器优化和 CPU 指令重排序而改变执行顺序。`synchronized` 和 `volatile` 关键字可以用来禁止指令重排序。

#### 3. `happens-before` 原则

**`happens-before`** 是 Java 内存模型中最重要的规则之一。它用来定义多线程环境中操作之间的顺序关系，确保数据的一致性和可见性。

常见的 `happens-before` 规则包括：

1. 程序次序规则：在一个线程内，按照代码的顺序，前面的操作 `happens-before` 于后面的操作。
2. 锁定规则：对一个锁的解锁 `happens-before` 于随后对这个锁的加锁。
3. volatile 变量规则：对一个 `volatile` 变量的写操作 `happens-before` 于后续对这个 `volatile` 变量的读操作。
4. 线程启动规则：线程的 `start()` 方法调用 `happens-before` 于该线程的每一个操作。
5. 线程终止规则：线程中的所有操作 `happens-before` 于对此线程的 `join()` 方法的返回。
6. 传递性：如果 `A happens-before B`，且 `B happens-before C`，那么 `A happens-before C`。

`happens-before` 规则确保了在多线程环境下，代码的执行是有序的和可见的。

#### 4. 指令重排序

指令重排序 是指编译器和处理器在不改变程序执行结果的前提下，可以重新排序指令的执行顺序。指令重排序有助于优化程序性能，但可能会导致多线程环境下的并发问题。

Java 提供了 `volatile` 关键字和 `synchronized` 关键字来禁止指令重排序：

- **`volatile`**：禁止将其修饰的变量的读写操作与其他内存操作重排序。
- **`synchronized`**：在进入同步块时，清空工作内存，在执行同步块代码时，强制把主内存中变量的最新值重新加载到工作内存中；在退出同步块时，把工作内存中修改的变量刷新到主内存中。

### 7、Java 流有哪些？

在 Java 中，流（Stream）是用于输入和输出操作的核心类，它们提供了一种高效和灵活的方式来处理数据流。Java 中的流主要分为 **字节流** 和 **字符流** 两大类，这两类又可以分为 **输入流** 和 **输出流**。此外，从 Java 8 开始，引入了 **Java Stream API**，用于对集合进行操作。

#### 1. 字节流

字节流用于处理二进制数据，适合于所有类型的 I/O，包括文本文件、图像、音频、视频等。

- **输入字节流（InputStream）**: 从数据源中读取数据。它是字节输入流的基类。

  常见的子类:

  - `FileInputStream`：用于从文件中读取数据。
  - `ByteArrayInputStream`：用于从字节数组中读取数据。
  - `ObjectInputStream`：用于从流中反序列化对象。
  - `BufferedInputStream`：为另一个输入流添加缓冲功能，提高读取效率。

- **输出字节流（OutputStream）**: 向数据目的地写入数据。它是字节输出流的基类。

  常见的子类:

  - `FileOutputStream`：用于向文件中写入数据。
  - `ByteArrayOutputStream`：用于向字节数组中写入数据。
  - `ObjectOutputStream`：用于将对象序列化，并写入输出流中。
  - `BufferedOutputStream`：为另一个输出流添加缓冲功能，提高写入效率。

```
try (FileInputStream fis = new FileInputStream("input.txt");
     FileOutputStream fos = new FileOutputStream("output.txt")) {
    int data;
    while ((data = fis.read()) != -1) {
        fos.write(data);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

#### 2. 字符流

字符流专门用于处理字符数据，适合于处理文本文件。它们以 16 位的 Unicode 字符为单位来读取和写入数据，因此适用于国际化的字符集。

- **输入字符流（Reader）**: 从字符输入流中读取字符。它是字符输入流的基类。

  常见的子类:

  - `FileReader`：用于从文件中读取字符数据。
  - `StringReader`：用于从字符串中读取字符数据。
  - `BufferedReader`：为另一个字符输入流添加缓冲功能，提高读取效率。

- **输出字符流（Writer）**: 向字符输出流中写入字符。它是字符输出流的基类。

  常见的子类:

  - `FileWriter`：用于向文件中写入字符数据。
  - `StringWriter`：用于向字符串中写入字符数据。
  - `BufferedWriter`：为另一个字符输出流添加缓冲功能，提高写入效率。

```
try (FileReader fr = new FileReader("input.txt");
     FileWriter fw = new FileWriter("output.txt")) {
    int data;
    while ((data = fr.read()) != -1) {
        fw.write(data);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

#### 3. Java Stream API（Java 8 引入）

Java 8 引入的 Stream API 是一个用于处理集合数据的高效工具。它支持对集合的操作（如过滤、映射、归约、排序等），并提供了一种声明性方式（类似 SQL 语句）来处理数据流。

- **创建流**：
  - `Stream.of(T... values)`：通过一组值创建流。
  - `Arrays.stream(T[] array)`：通过数组创建流。
  - `Collection.stream()`：通过集合创建流。
- **常见的操作**：
  - 中间操作: 生成一个新的流，惰性执行。常见的中间操作包括 `filter`、`map`、`flatMap`、`sorted`、`distinct`、`limit`、`skip`。
  - 终端操作: 触发流的执行并生成结果。常见的终端操作包括 `forEach`、`collect`、`reduce`、`count`、`allMatch`、`anyMatch`、`noneMatch`、`findFirst`、`findAny`。

```
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");
names.stream()
     .filter(name -> name.startsWith("C"))
     .map(String::toUpperCase)
     .forEach(System.out::println); // 输出：CHARLIE
```

### 8、String StringBuilder StringBuffer 区别？

#### 1. `String`

- 不可变性: `String` 是不可变的对象（immutable）。一旦创建了一个 `String` 对象，其内容是不可改变的。
- 字符串常量池: JVM 会将 `String` 对象的字面量存储在字符串常量池中，以便复用相同的字符串，提高效率。
- 适用场景: `String` 适用于字符串内容固定不变的场景。例如，处理配置文件、日志信息、常量字符串等。

```
String str1 = "Hello";
String str2 = "Hello"; // str1 和 str2 引用同一个字符串常量池中的 "Hello"
str1 = str1 + " World"; // 生成一个新的字符串对象 "Hello World"
```

上面的代码会创建两个 `String` 对象，一个是 `"Hello"`，另一个是 `"Hello World"`。`str1` 在拼接操作后会指向新的字符串对象，而原来的 `"Hello"` 字符串仍然存在（如果没有其他引用，最终会被垃圾回收）。

#### 2. `StringBuilder`

- 可变性: `StringBuilder` 是可变的对象（mutable）。可以直接对 `StringBuilder` 对象进行修改，而不会生成新的对象。
- 线程不安全: `StringBuilder` 不是线程安全的，即它的操作没有进行同步。多个线程操作同一个 `StringBuilder` 对象时可能会产生线程安全问题。
- 性能: 由于 `StringBuilder` 不进行同步操作，其性能优于 `StringBuffer`（特别是在单线程环境下）。对于频繁修改字符串内容的操作，如循环拼接字符串等，`StringBuilder` 更为高效。
- 适用场景: `StringBuilder` 适用于在单线程中执行大量字符串拼接或修改操作的场景。

```
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World"); // 直接在原对象上修改
System.out.println(sb.toString()); // 输出: Hello World
```

#### 3. `StringBuffer`

- 可变性: 与 `StringBuilder` 类似，`StringBuffer` 也是可变的对象，可以修改其内容。
- 线程安全: `StringBuffer` 是线程安全的，因为它的所有操作都是同步的（synchronized），即在多线程环境下可以安全地使用。
- 性能: 由于 `StringBuffer` 的操作是同步的，因此在单线程环境下，它的性能不如 `StringBuilder`。但在多线程环境下，如果需要操作同一个字符串对象，`StringBuffer` 则能保证线程安全。
- 适用场景: `StringBuffer` 适用于在多线程中需要安全地操作字符串的场景。

```
StringBuffer sb = new StringBuffer("Hello");
sb.append(" World"); // 线程安全的操作
System.out.println(sb.toString()); // 输出: Hello World
```

#### 总结

| 特性       | `String`                       | `StringBuilder`                  | `StringBuffer`                   |
| ---------- | ------------------------------ | -------------------------------- | -------------------------------- |
| 可变性     | 不可变                         | 可变                             | 可变                             |
| 线程安全性 | 线程安全                       | 线程不安全                       | 线程安全                         |
| 使用场景   | 字符串内容不变的场景           | 单线程中需要大量修改字符串的场景 | 多线程中需要大量修改字符串的场景 |
| 性能       | 较低（每次修改都会创建新对象） | 较高（不进行同步操作）           | 较低（同步操作有开销）           |

### 9、final 关键字？

在 Java 中，`final` 关键字用于限制变量、方法或类的修改，确保其不被改变或重写。它可以应用于变量、方法和类，具有不同的作用。

#### 1. `final` 变量

- 不可修改性: 使用 `final` 修饰的变量在初始化后不可再更改。意味着一旦为 `final` 变量赋值，该变量就不能再被重新赋值。
- 常量: 对于基本数据类型，`final` 变量一旦赋值，其值不能再改变。对于引用数据类型，`final` 引用一旦初始化后不能再指向其他对象，但其指向的对象的内容是可以改变的。

```
final int CONSTANT_VALUE = 100;  // 基本数据类型的 final 变量
CONSTANT_VALUE = 200;  // 编译错误，不能修改 final 变量的值

final StringBuilder sb = new StringBuilder("Hello");
sb.append(" World");  // 允许，引用类型的内容可以改变
sb = new StringBuilder("New");  // 编译错误，不能更改 final 引用的指向
```

#### 2. `final` 方法

防止重写: 使用 `final` 修饰的方法不能被子类重写（override）。这对设计不可改变的行为非常有用，确保该方法的实现不会被子类更改。

```
class Parent {
    public final void display() {
        System.out.println("This is a final method.");
    }
}

class Child extends Parent {
    // 编译错误，不能重写 final 方法
    public void display() {
        System.out.println("Trying to override.");
    }
}
```

在上面的示例中，`Parent` 类的 `display` 方法被标记为 `final`，因此子类 `Child` 不能重写它。

#### 3. `final` 类

防止继承: 使用 `final` 修饰的类不能被继承。这对于设计不可扩展的类非常有用，尤其是当你希望确保类的实现不被更改时（如安全或系统级的关键类）。

```
final class FinalClass {
    // Class implementation
}

// 编译错误，不能继承 final 类
class SubClass extends FinalClass {
    // SubClass implementation
}
```

在上面的示例中，`FinalClass` 是一个 `final` 类，因此无法被其他类继承。

#### 4. `final` 关键字的使用场景

- 不可修改的常量: 使用 `final` 变量来定义不可改变的常量，尤其是公共常量时通常使用 `public static final` 修饰。
- 确保安全和不变性: 使用 `final` 方法和类来防止方法被重写和类被继承，这对安全性和不变性至关重要。
- 性能优化: 编译器和 JVM 可以针对 `final` 关键字进行优化。例如，`final` 方法在编译时可以直接被内联（inlined），以提高运行时性能。

#### 5. 其他

- **`final` 参数**: 可以在方法的参数列表中使用 `final` 关键字，表示在方法内部不能更改参数的引用。

  ```
  public void print(final int number) {
      // number = 5; // 编译错误，不能修改 final 参数
      System.out.println(number);
  }
  ```

- **`final` 和 `static` 的组合**: 常见于定义常量时使用，如 `public static final int MAX_SIZE = 100;`。`static` 用于表示类级别共享的常量，而 `final` 用于表示常量的值不可改变。

### 10、equals 的特性？

在 Java 中，`equals` 方法是用于比较两个对象是否“相等”的方法。`equals` 方法是定义在 `java.lang.Object` 类中的，这意味着所有 Java 对象都可以使用这个方法。

#### 1. `equals` 方法的默认行为

默认情况下，`Object` 类中的 `equals` 方法是比较两个对象的引用地址（即内存地址），只有当两个引用指向同一个对象时，`equals` 方法才会返回 `true`。这就意味着，除非一个类重写了 `equals` 方法，否则它不会按照对象的内容来比较。

```
Object obj1 = new Object();
Object obj2 = new Object();
System.out.println(obj1.equals(obj2)); // 输出: false

Object obj3 = obj1;
System.out.println(obj1.equals(obj3)); // 输出: true
```

在这个例子中，`obj1` 和 `obj2` 是两个不同的 `Object` 实例，尽管它们的内容相同，但 `equals` 方法比较的是它们的引用地址，因此返回 `false`。`obj3` 和 `obj1` 引用的是同一个对象，所以返回 `true`。

#### 2. `equals` 方法的特性

根据 Java 文档中的规定，`equals` 方法需要满足以下几个特性：

- 自反性 (Reflexive): 对于任何非空引用 `x`，`x.equals(x)` 应该返回 `true`。即一个对象必须等于它自己。
- 对称性 (Symmetric): 对于任何非空引用 `x` 和 `y`，如果 `x.equals(y)` 返回 `true`，那么 `y.equals(x)` 也应该返回 `true`。即两个对象彼此相等的关系是对称的。
- 传递性 (Transitive): 对于任何非空引用 `x`、`y` 和 `z`，如果 `x.equals(y)` 返回 `true`，并且 `y.equals(z)` 返回 `true`，那么 `x.equals(z)` 也应该返回 `true``。
- 一致性 (Consistent): 对于任何非空引用 `x` 和 `y`，在没有修改对象的前提下，多次调用 `x.equals(y)` 应该返回相同的结果。
- 非空性 (Non-nullity): 对于任何非空引用 `x`，`x.equals(null)` 应该返回 `false`。

#### 3. 正确重写 `equals` 方法

为了实现一个合适的 `equals` 方法，通常需要根据类的实际内容来判断两个对象是否相等。例如，在字符串类 `String` 中，`equals` 方法已经被重写来比较字符串的内容。

**重写 `equals` 方法的步骤：**

1. 使用 `==` 检查参数是否为当前对象:如果是，直接返回 `true`。
2. 使用 `instanceof` 检查参数是否为当前类的实例:如果不是，直接返回 `false`。
3. 转换参数为正确的类型:对参数进行强制类型转换。
4. 逐个比较类中的每个重要字段:
   - 使用 `==` 比较基本数据类型。
   - 使用 `equals` 比较引用数据类型。

```
public class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true; // 1. 检查是否为同一个引用
        if (obj == null || getClass() != obj.getClass()) return false; // 2. 检查是否为同一类的实例

        Person person = (Person) obj; // 3. 强制类型转换
        return age == person.age && name.equals(person.name); // 4. 比较重要字段
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age); // 推荐同时重写 hashCode 方法
    }
}
```

#### 4. 重写 `equals` 方法时的注意事项

- **同时重写 `hashCode` 方法**: Java 中 `equals` 和 `hashCode` 方法有一个通用的契约：如果两个对象通过 `equals` 方法被认为是相等的，那么它们必须具有相同的 `hashCode`。如果你重写了 `equals` 方法，一定要记得同时重写 `hashCode` 方法，否则会在使用哈希表（如 `HashMap` 和 `HashSet`）时出现问题。
- **对 `null` 的处理**: 在 `equals` 方法中，必须要确保当参数为 `null` 时返回 `false`。
- **遵循 `equals` 方法的特性**: 重写 `equals` 方法时，要确保遵循自反性、对称性、传递性、一致性和非空性等规则。

### 11、归并排序？

采用分治法（Divide and Conquer）来实现排序。它具有良好的性能，时间复杂度为 O(nlog⁡n)O(n \log n)O(nlogn)，且稳定。

#### 基本原理

归并排序的核心思想是将一个大的未排序数组分解成两个较小的未排序数组，递归地对这两个数组进行排序，然后将排序好的两个数组合并成一个有序数组。具体步骤如下：

1. **分解（Divide）**: 将待排序的数组从中间分成两半。
2. **递归排序（Conquer）**: 递归地对这两半进行归并排序，直到每个子数组只包含一个元素（即自然有序）。
3. **合并（Merge）**: 将两个已排序的子数组合并成一个有序数组。

#### 实现步骤

1. 分解阶段:如果数组的长度大于1，则将其分成两半，递归地对每一半进行归并排序。
2. 合并阶段:合并两个已排序的子数组，生成一个新的排序数组。这个合并过程将两个有序的数组逐个比较元素并将其排序放入新数组中。

#### 代码

```
public class MergeSort {

    // 主方法，负责调用归并排序
    public static void mergeSort(int[] array) {
        if (array.length < 2) {
            return; // 基本情况：数组为空或只有一个元素
        }
        int mid = array.length / 2;
        int[] left = new int[mid];
        int[] right = new int[array.length - mid];

        // 拷贝数组的左半部分
        System.arraycopy(array, 0, left, 0, mid);
        // 拷贝数组的右半部分
        System.arraycopy(array, mid, right, 0, array.length - mid);

        // 递归排序左半部分
        mergeSort(left);
        // 递归排序右半部分
        mergeSort(right);

        // 合并已排序的左右部分
        merge(array, left, right);
    }

    // 合并两个已排序的数组
    private static void merge(int[] array, int[] left, int[] right) {
        int i = 0; // 左数组的索引
        int j = 0; // 右数组的索引
        int k = 0; // 主数组的索引

        // 合并左数组和右数组
        while (i < left.length && j < right.length) {
            if (left[i] <= right[j]) {
                array[k++] = left[i++];
            } else {
                array[k++] = right[j++];
            }
        }

        // 复制左数组剩余的元素
        while (i < left.length) {
            array[k++] = left[i++];
        }

        // 复制右数组剩余的元素
        while (j < right.length) {
            array[k++] = right[j++];
        }
    }

    public static void main(String[] args) {
        int[] array = {38, 27, 43, 3, 9, 82, 10};
        System.out.println("Original array:");
        for (int num : array) {
            System.out.print(num + " ");
        }
        System.out.println();

        mergeSort(array);

        System.out.println("Sorted array:");
        for (int num : array) {
            System.out.print(num + " ");
        }
    }
}
```

#### 归并排序的优缺点

##### 优点

1. 时间复杂度: 归并排序的时间复杂度为 O(nlog⁡n)O(n \log n)O(nlogn)，在最坏情况下也能保持这个时间复杂度，因此非常稳定。
2. 稳定性: 归并排序是稳定的排序算法，能保持相同元素的相对顺序。
3. 适用于大数据量: 归并排序在处理大数据量时表现良好，特别是在外部排序（如排序文件）中表现优越。

##### 缺点

1. 空间复杂度: 归并排序需要额外的空间来存储临时数组，空间复杂度为 O(n)O(n)O(n)。
2. 实现复杂性: 相比于某些其他排序算法，归并排序的实现相对复杂，需要处理额外的空间和递归。

### 12、http tcp 区别？

#### 1. 协议层次

- **HTTP**: 应用层协议。HTTP 是一个在应用层（第七层）工作的协议，主要用于客户端和服务器之间的通信，特别是 Web 浏览器和 Web 服务器之间的数据传输。
- **TCP**: 传输层协议。TCP 是一个在传输层（第四层）工作的协议，负责在网络中的两个主机之间可靠地传输数据。

#### 2. 功能

- **HTTP**:
  - 用途: 主要用于从 Web 服务器向客户端（如浏览器）传输数据（如网页、图片、视频等）。
  - 无状态: HTTP 协议本身是无状态的，即每个请求都是独立的，不保留之前的请求和响应状态。虽然 HTTP 协议无状态，但可以通过会话机制（如 cookies 和 sessions）来维持状态。
  - 请求/响应模型: HTTP 使用请求/响应模型，客户端发送请求，服务器返回响应。
- **TCP**:
  - 用途: 提供可靠的、面向连接的数据传输服务。TCP 在发送数据时会确保数据的完整性和顺序。
  - 有状态: TCP 连接是有状态的，必须先建立连接，然后才能传输数据，数据传输完成后连接才会断开。
  - 数据流: TCP 传输的是数据流，数据在发送时被拆分成小的数据包（segment），并在接收端重新组装。

#### 3. 可靠性

- **HTTP**:
  - 可靠性: HTTP 的可靠性取决于底层的传输层协议（通常是 TCP）。HTTP 自身并不处理数据的可靠性和顺序问题。
- **TCP**:
  - 可靠性: TCP 提供可靠的数据传输。它通过以下机制来保证数据的可靠性：
  - 数据重传: 如果数据丢失或损坏，TCP 会重新发送数据。
  - 顺序保证: 确保数据按照正确的顺序到达接收端。
  - 流量控制: 控制数据发送的速度，以防止接收端被淹没。
  - 拥塞控制: 调节网络流量以防止网络拥塞。

#### 4. 连接

- **HTTP**:
  - 连接: HTTP 是一个无连接的协议（在 HTTP/1.0 中）。HTTP/1.1 引入了持久连接（Keep-Alive），允许在同一个 TCP 连接上发送多个 HTTP 请求/响应，提高了效率。
- **TCP**:
  - 连接: TCP 是一个面向连接的协议。通信开始之前必须先建立连接（通过三次握手），通信结束后必须断开连接（通过四次挥手）。

#### 5. 数据传输

- **HTTP**:
  - 数据单位: HTTP 传输的是请求和响应消息，消息由头部和主体组成。
  - 数据格式: HTTP 数据是以文本格式（如 HTML、JSON、XML）传输的。
- **TCP**:
  - 数据单位: TCP 传输的是数据包（segment），每个数据包都有一个头部和负载。
  - 数据格式: TCP 传输的是原始的字节流，不关心数据的具体格式。

#### 6. 使用示例

- **HTTP**: 用于 Web 浏览器与 Web 服务器之间的通信。例如，当你在浏览器中输入网址并加载网页时，浏览器通过 HTTP 协议与 Web 服务器进行通信，发送请求并接收响应。
- **TCP**: 用于确保可靠的底层数据传输。例如，当你通过浏览器下载文件时，HTTP 协议通过 TCP 连接进行数据传输，确保文件完整地传输到你的计算机。

### 13、tcp 拥塞控制算法？

TCP 拥塞控制算法用于管理和避免网络拥塞，确保网络的高效和稳定。TCP 的拥塞控制算法通过动态调整数据发送速率来避免网络过载，从而提高网络的可靠性和性能。

#### 1. 慢启动 (Slow Start)

- 目的: 解决网络初期的拥塞问题。
- 工作原理:
  - 在连接建立时，TCP 使用一个小的拥塞窗口（Congestion Window, cwnd）开始发送数据。
  - 每收到一个确认（ACK），cwnd 就增加一个新的数据包的大小。
  - 这个增长是指数级的（即每个往返时间（RTT）增长一倍），直到达到一个阈值（慢启动阈值，ssthresh）或出现丢包。
- 特点: 慢启动算法可以快速增长窗口大小以利用可用带宽，但可能会导致网络拥塞。

#### 2. 拥塞避免 (Congestion Avoidance)

- **目的**: 控制数据发送速率，避免网络拥塞。
- **工作原理**:
  - 当 cwnd 达到慢启动阈值时，进入拥塞避免阶段。
  - 在这个阶段，cwnd 以线性方式增长：每经过一个 RTT，cwnd 增加一个数据包的大小。
  - 这种线性增长减少了拥塞的风险，并在网络条件变坏时逐渐降低数据发送速率。

#### 3. 快速重传 (Fast Retransmit)

- **目的**: 快速处理丢包问题。
- **工作原理**:
  - 当发送方收到三个相同的 ACK（即三个重复确认），说明数据包可能丢失。
  - 发送方会立即重传丢失的数据包，而不是等待超时。
  - 这种方法可以减少丢包造成的延迟，提高网络的响应速度。

#### 4. 快速恢复 (Fast Recovery)

- **目的**: 提高网络性能，处理网络拥塞后的恢复。
- **工作原理**:
  - 当快速重传触发时，TCP 进入快速恢复阶段。
  - 在快速恢复阶段，慢启动阈值（ssthresh）被设置为 cwnd 的一半，并且 cwnd 设置为 ssthresh 的值。
  - 在此阶段，拥塞窗口不会回到初始值，而是以更慢的速率（线性增长）恢复。
  - 这种方式可以避免在数据丢失后恢复过程过长，减少数据传输的延迟。

#### 5. TCP 拥塞控制算法的演变

随着网络环境的变化，TCP 拥塞控制算法也经历了一些演变和优化。下面是几种常见的拥塞控制算法：

- **TCP Tahoe**: 包含慢启动、拥塞避免、快速重传和快速恢复机制。它在发生丢包时会将 cwnd 重置为 1，并重新开始慢启动。
- **TCP Reno**: 在 TCP Tahoe 的基础上引入了更优化的快速恢复机制。TCP Reno 在快速恢复阶段不将 cwnd 重新设置为 1，而是维持在拥塞避免阶段的值，以加快恢复速度。
- **TCP NewReno**: 改进了 TCP Reno，增强了快速恢复的能力，能够更有效地处理多个丢包的情况。
- **TCP Vegas**: 基于延迟进行拥塞控制。它通过监测往返时间（RTT）的变化来检测拥塞，并动态调整发送速率。
- **TCP CUBIC**: 针对高带宽和长延迟网络进行了优化。它使用一种立方函数来计算拥塞窗口的增长，以便在高带宽-延迟网络中更有效地利用带宽。
- **TCP BBR**: 基于带宽-延迟产品（Bandwidth-Delay Product, BDP）进行拥塞控制。它通过测量带宽和延迟来动态调整发送速率，以达到最佳网络利用率。

### 14、NIO BIO AIO？

#### 1. BIO（Blocking I/O）

##### 特点

- 阻塞 I/O：在 BIO 中，每一个 I/O 操作都是阻塞的，意味着在进行 I/O 操作时，线程会被挂起，直到操作完成。
- 每连接一个线程：每个连接都会分配一个独立的线程来处理，这可能会导致资源消耗较高，尤其是在连接数很多时。

##### 工作原理

- 当客户端发起连接时，服务器接受连接请求，并为每个客户端创建一个新的线程来处理 I/O 操作。
- 线程在进行读写操作时会被阻塞，直到数据可用或操作完成。

##### 优缺点

- 优点:实现简单，代码易于理解和编写。
- 缺点:
  - 线程数量随着连接数增加而增加，可能导致线程资源耗尽。
  - 不适合高并发环境，因为线程开销大，无法有效利用系统资源。

#### 2. NIO（Non-blocking I/O）

##### 特点

- 非阻塞 I/O：在 NIO 中，I/O 操作是非阻塞的，线程可以继续执行其他任务，而不需要等待 I/O 操作完成。
- 选择器：NIO 使用 `Selector` 来管理多个通道（Channel），可以在一个线程中处理多个连接，降低了线程的使用。

##### 工作原理

- Channel: NIO 使用通道（Channel）来进行数据传输，通道可以是文件通道、套接字通道等。
- Selector: 使用选择器（Selector）来轮询多个通道的状态，确定哪些通道准备好进行 I/O 操作。
- Buffer: 数据的读写是通过缓冲区（Buffer）进行的。

##### 优缺点

- 优点:
  - 能够处理大量并发连接，线程开销较小，适合高并发场景。
  - 通过选择器（Selector）实现了单线程处理多个连接，提高了资源利用率。
- 缺点:编程复杂度较高，相比 BIO，需要处理更多的细节和状态管理。

#### 3. AIO（Asynchronous I/O）

##### 特点

- 异步 I/O：AIO 中的 I/O 操作是完全异步的，线程在发起 I/O 操作后，不需要等待操作完成，可以立即继续执行其他任务。
- 回调机制：AIO 使用回调机制来处理 I/O 完成后的操作，操作完成时会通知相应的回调方法。

##### 工作原理

- 异步 I/O 操作: 客户端发起异步 I/O 操作后，操作立即返回。系统会在操作完成时通过回调函数通知应用程序。
- 回调: 应用程序不需要轮询或阻塞等待 I/O 操作完成，而是注册一个回调函数，在 I/O 操作完成时系统会自动调用。

##### 优缺点

- 优点:
  - 高效的 I/O 处理，能够充分利用系统资源，适合处理大量并发 I/O 操作。
  - 代码中不需要显式地管理线程和阻塞，编程模型更符合异步编程的特点。
- 缺点:
  - 编程复杂度较高，回调函数可能导致代码难以理解和维护。
  - 目前在 Java 中的支持较为有限，主要集中在 Java 7 及以后的版本中。