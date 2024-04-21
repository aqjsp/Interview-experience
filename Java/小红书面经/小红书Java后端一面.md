# 小红书Java后端一面

> https://www.nowcoder.com/feed/main/detail/26dfce0f1c1944bc9842cd92987dbd5a

### 1、spring boot主要是相比spring的framework有哪些区别？

Spring Boot 是 Spring Framework 的一个扩展，主要是为了简化 Spring 应用程序的开发和部署。

##### 区别

1. 简化配置：Spring Boot 提供了一种约定大于配置的方式，通过自动配置（auto-configuration）和起步依赖（starter dependencies），大部分情况下无需手动配置，开发者可以更专注于业务逻辑的开发。
2. 内嵌容器：Spring Boot 可以打包为可执行的 JAR 文件，并且内嵌了 Tomcat、Jetty、Undertow 等 Web 服务器，使得应用程序可以独立运行，无需外部容器。
3. 自动配置：Spring Boot 根据应用程序的依赖和配置，自动配置 Spring 应用程序的各个组件，如数据源、事务管理、日志、安全等，减少了手动配置的工作量。
4. 起步依赖：Spring Boot 提供了一系列的起步依赖，可以方便地引入常用的功能模块，如 Web 开发、数据访问、消息队列等，简化了项目的依赖管理。
5. 监控和管理：Spring Boot 提供了 Actuator 模块，可以方便地监控和管理应用程序，包括健康检查、性能指标、配置信息等。
6. 外部化配置：Spring Boot 支持将应用程序的配置信息外部化，可以使用属性文件、YAML 文件、环境变量等方式进行配置，使得应用程序更易于配置和管理。
7. 集成测试支持：Spring Boot 提供了一套完整的集成测试支持，可以方便地进行单元测试和集成测试。

### 2、传统的spring framework和spring boot启动方式的区别？

**传统的 Spring Framework 启动方式**：

- 在传统的 Spring Framework 中，应用程序通常是通过在 web.xml 文件中配置 `ContextLoaderListener` 或 `DispatcherServlet` 来启动 Spring 容器的。
- 配置 `ContextLoaderListener` 会在应用程序启动时加载 Spring 的根上下文（ApplicationContext），用于管理应用程序中的 bean。
- 配置 `DispatcherServlet` 则会在应用程序启动时加载一个或多个 Spring 的 Web 上下文（WebApplicationContext），用于处理 web 请求。

```
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/applicationContext.xml</param-value>
</context-param>

<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/dispatcher-servlet.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

**Spring Boot 启动方式**：

- Spring Boot 的启动方式更加简单，主要通过 `SpringApplication` 类的静态 `run` 方法来启动应用程序，不需要额外的配置文件。
- Spring Boot 会自动扫描应用程序中的类，并根据约定进行自动配置，包括自动配置 Spring 容器、自动配置 Web 应用程序、自动配置数据库等。

```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

}
```

### 3、JVM内存结构？程序计数器是线程安全的吗？

##### 内存结构

1. 程序计数器（Program Counter Register）：
   - 程序计数器是一块较小的内存空间，可以看作是当前线程所执行的字节码的行号指示器。
   - 在多线程环境下，每个线程都有自己独立的程序计数器，互不影响。
2. Java 虚拟机栈（JVM Stack）：
   - 每个线程在创建时都会被分配一个 Java 虚拟机栈，用于存储方法执行的栈帧。
   - 每个方法被调用时都会创建一个栈帧，用于存储局部变量、操作数栈、动态链接、方法出口等信息。
   - 栈帧的大小在编译时确定，可以动态扩展，但不能超过栈的最大容量。
3. 本地方法栈（Native Method Stack）：
   - 本地方法栈类似于 Java 虚拟机栈，用于执行 native 方法。
   - 每个线程都有自己独立的本地方法栈。
4. Java 堆（Java Heap）：
   - Java 堆是所有线程共享的内存区域，用于存储对象实例和数组。
   - Java 堆在启动时就被分配好，可以动态扩展，但受到最大堆大小的限制。
5. 方法区（Method Area）：
   - 方法区也是所有线程共享的内存区域，用于存储类的结构信息、静态变量、常量池等数据。
   - 方法区在启动时就被分配好，可以动态扩展，但受到最大方法区大小的限制。
6. 运行时常量池（Runtime Constant Pool）：
   - 运行时常量池是方法区的一部分，用于存储编译期生成的各种字面量和符号引用。
   - 运行时常量池在方法区中，也受到方法区大小的限制。
7. 直接内存（Direct Memory）：
   - 直接内存并不是 JVM 运行时数据区的一部分，但被频繁地用于 NIO 操作，通过 ByteBuffer 分配。

程序计数器是线程私有的，每个线程有自己独立的程序计数器，存储着当前线程执行的字节码行号。在多线程环境下，不同线程的程序计数器互不影响，因此可以认为程序计数器是线程安全的。

### 4、一些垃圾处理器有没有了解过？说下CMS和G1两个垃圾处理器的区别？

垃圾处理器是负责管理和回收内存中无用对象的组件，Java 中的垃圾处理器主要包括 CMS（Concurrent Mark-Sweep）和 G1（Garbage-First）两种。

##### 区别

1. CMS（Concurrent Mark-Sweep）：
   - 一种以最短停顿时间为目标的垃圾收集器，适用于对延迟敏感的应用程序。
   - 使用并发标记和并发清除算法，可以在主线程运行的同时执行部分垃圾收集工作，从而减少停顿时间。
   - 缺点是可能会产生碎片，当碎片过多时可能会触发 Full GC，导致较长的停顿时间。
   - 适用于需要快速响应时间的应用程序，但不适合内存大、生成大量垃圾的应用程序。
2. G1（Garbage-First）：
   - 一种面向服务端应用的垃圾收集器，它试图在吞吐量和停顿时间之间取得平衡。
   - G1 使用分代收集算法，将堆内存划分为多个区域（Region），每个区域可以是 Eden 区、Survivor 区或 Old 区。
   - 通过优先回收垃圾比例较高的区域（Garbage-First），以减少碎片和提高回收效率。
   - 特点是可以在一定程度上控制停顿时间，适用于大内存应用程序和对延迟要求较高的场景。

CMS 更注重最短停顿时间，适用于对响应速度要求高的应用程序；而 G1 则更注重在吞吐量和停顿时间之间取得平衡，适用于需要平衡性能和延迟的场景。

### 5、MySQL的ACID实现原理，通过什么样的设计去解决可能出现的问题？

1. **原子性（Atomicity）**：事务是一个不可分割的工作单位，要么全部执行，要么全部不执行。
   - MySQL 使用日志（Redo Log 和 Undo Log）来实现原子性。Redo Log 记录事务所做的修改，Undo Log 用于事务回滚。
   - 当事务提交时，首先将 Redo Log 写入磁盘，然后将修改的数据写入磁盘；如果事务失败，使用 Undo Log 进行回滚。
2. **一致性（Consistency）**：事务执行前后，数据库从一个一致性状态转换到另一个一致性状态。
   - MySQL 使用锁机制和 MVCC（多版本并发控制）来实现一致性。MVCC 可以保证读取到的数据是事务开始时的一致状态。
3. **隔离性（Isolation）**：多个事务并发执行时，每个事务都应该感觉不到其他事务的存在。
   - MySQL 使用锁和事务隔离级别来实现隔离性。事务隔离级别包括读未提交（Read Uncommitted）、读已提交（Read Committed）、可重复读（Repeatable Read）和串行化（Serializable）。
4. **持久性（Durability）**：一旦事务提交，则其所做的修改会永久保存在数据库中。
   - MySQL 使用 Redo Log 和 Undo Log 来实现持久性。Redo Log 用于在事务提交后将修改的数据写入磁盘，保证数据的持久性。

### 6、MVCC实现原理？

MVCC（Multi-Version Concurrency Control）是数据库系统中常用的一种并发控制机制，用于实现事务的隔离性。

实现原理：

1. **版本号**：每个数据行都会保存多个版本，每个版本都有一个唯一的版本号或时间戳。当一个事务开始时，系统会为该事务分配一个唯一的版本号或时间戳。
2. **数据结构**：对于每个数据行，通常会有一个主版本（Current Version）和多个历史版本（Historical Versions）。主版本是最新的数据版本，而历史版本是之前的数据版本，用于支持事务的隔离性。
3. **读操作**：当一个事务需要读取数据时，数据库系统根据事务的版本号或时间戳，决定是读取主版本还是历史版本。如果事务读取的是主版本，则可以读取到最新的数据；如果事务读取的是历史版本，则可以读取到事务开始时的数据。
4. **写操作**：当一个事务需要修改数据时，数据库系统会为该事务创建一个新的版本，并将修改后的数据写入新版本中。同时，原始数据行的主版本仍然保留，用于其他事务的读取。
5. **事务隔离级别**：MVCC 可以支持不同的事务隔离级别，如读已提交（Read Committed）、可重复读（Repeatable Read）等。不同的隔离级别决定了事务读取数据时所使用的版本。

MVCC 的实现可以提高数据库系统的并发性能，因为它允许读操作不会被写操作阻塞，同时保证了事务的隔离性。但是，MVCC 也会增加存储空间的消耗，因为需要保存多个数据版本。

### 7、假如说你现在需要连接一个不存在的IP地址，客户端和服务端会做哪些事情？

需要考虑是目标IP地址和客户端IP地址是不是在同一个局域网，不在同一个局域网下会先发到路由器，包括在数据链路层上有什么过程。

1. **客户端**：
   - 客户端会首先尝试解析该 IP 地址对应的主机名（如果有提供的话）。
   - 如果无法解析主机名或者解析失败，则客户端会直接尝试连接该 IP 地址。
   - 客户端连接函数（如 `connect()`）会立即返回一个错误，指示连接失败。
   - 客户端可能会尝试重新连接，或者根据具体情况进行错误处理。
2. **服务端**：
   - 如果服务端的 IP 地址是被动监听的，即服务端在该 IP 地址上等待连接请求，那么服务端会继续等待连接，因为服务端并不关心客户端是否存在。
   - 当服务端收到连接请求时，会尝试向客户端发送连接确认信息（如果协议需要），但由于客户端不存在，这些信息将无法到达客户端。

### 8、AQS？

AQS（AbstractQueuedSynchronizer）是 Java 中用于实现同步器（Synchronizer）的抽象基类，它提供了一种基于 FIFO 等待队列的同步器框架，可以用来构建各种同步器，如 ReentrantLock、Semaphore、CountDownLatch 等。

AQS 的主要特点和实现原理：

1. **同步状态（State）**：AQS 中定义了一个整型变量表示同步状态，可以用来表示资源的数量或者锁的持有情况等。通过修改同步状态来实现同步器的功能。
2. **等待队列（Wait Queue）**：AQS 中维护了一个 FIFO 的等待队列，用于存放因获取锁而阻塞的线程。等待队列中的线程会被挂起，并且按照先进先出的顺序唤醒。
3. **独占模式和共享模式**：AQS 支持两种同步模式，即独占模式和共享模式。独占模式用于实现互斥锁，只允许一个线程访问共享资源；共享模式用于实现信号量等，允许多个线程同时访问共享资源。
4. **AQS 的主要方法**：
   - `acquire(int arg)`：尝试获取同步状态，如果获取失败则加入等待队列并挂起线程。
   - `release(int arg)`：释放同步状态，并唤醒等待队列中的下一个线程。
   - `tryAcquire(int arg)`：尝试获取同步状态，成功则返回 true，失败则返回 false，不会阻塞线程。
   - `tryRelease(int arg)`：尝试释放同步状态，成功则返回 true，失败则返回 false。
5. **AQS 的子类**：AQS 是一个抽象类，需要子类实现一些抽象方法来定义具体的同步逻辑。常见的子类包括：
   - `ReentrantLock.Sync`：ReentrantLock 的同步器实现。
   - `Semaphore.Sync`：Semaphore 的同步器实现。
   - `CountDownLatch.Sync`：CountDownLatch 的同步器实现。

### 9、synchronized和reentrantLock的区别？

1. **使用方式**：
   - `synchronized` 是 Java 关键字，可以用于修饰方法或代码块，实现对方法或代码块的同步。
   - `ReentrantLock` 是一个类，需要显式地创建对象，并在需要同步的代码块中使用 `lock()` 和 `unlock()` 方法。
2. **可中断性**：
   - `synchronized` 在等待锁的过程中，是不可中断的，即无法被其他线程打断。
   - `ReentrantLock` 提供了 `lockInterruptibly()` 方法，可以在等待锁的过程中响应中断。
3. **公平性**：
   - `synchronized` 是非公平锁，无法保证等待的线程按照请求锁的顺序获取锁。
   - `ReentrantLock` 可以选择创建公平锁，保证等待时间最长的线程会先获得锁。
4. **灵活性**：
   - `ReentrantLock` 提供了更多的灵活性，比如可以实现公平锁、可重入锁、超时锁等功能。
   - `synchronized` 的功能相对较少，只能实现基本的同步功能。
5. **性能**：
   - 在低竞争情况下，`synchronized` 的性能可能更好，因为它是 JVM 内置的关键字，而 `ReentrantLock` 是通过 Java 类实现的。
   - 在高竞争和大量线程访问时，`ReentrantLock` 的性能可能更好，因为它提供了更多的同步控制选项，可以更灵活地调整同步策略。

### 10、MySQL中的锁机制？

1. **表级锁**：
   - 表级锁是最粗粒度的锁，可以用来锁定整张表。
   - MySQL 中的表级锁包括读锁（`READ`）和写锁（`WRITE`）。
   - 读锁（共享锁）允许多个事务同时读取数据，但不允许写入数据，适用于读密集型操作。
   - 写锁（排它锁）只允许一个事务独占访问表，其他事务无法读取或写入数据，适用于写密集型操作。
2. **行级锁**：
   - 行级锁是最细粒度的锁，可以用来锁定表中的某一行数据。
   - MySQL 中的行级锁包括共享锁（`S`）和排它锁（`X`）。
   - 共享锁允许多个事务同时读取同一行数据，但不允许其他事务对该行数据进行写操作。
   - 排它锁只允许一个事务独占访问某一行数据，其他事务无法读取或写入该行数据。
3. **锁的使用**：
   - 在 MySQL 中，可以使用 `LOCK TABLES` 命令来锁定表，使用 `UNLOCK TABLES` 命令来释放表级锁。
   - 也可以使用 `SELECT ... FOR UPDATE` 或 `SELECT ... LOCK IN SHARE MODE` 语句来获取行级锁。
4. **死锁**：
   - 当多个事务相互等待对方持有的锁时，可能会发生死锁。
   - MySQL 中的 InnoDB 存储引擎会自动检测和处理死锁，通常会选择其中一个事务作为死锁牺牲者，回滚该事务并释放锁。

### 11、行锁加锁的基本单位是什么？了解Next-Key Lock吗？Next-Key Lock退化为行锁的过程？

在 MySQL 中，行锁的基本单位是索引记录。当使用 `SELECT ... FOR UPDATE` 或 `SELECT ... LOCK IN SHARE MODE` 对数据行加锁时，实际上是对索引记录进行加锁。

**Next-Key Lock** 是 MySQL 中一种特殊的行锁，用于解决幻读（Phantom Read）的问题。Next-Key Lock 组合了索引记录的锁和索引间隙（Index Gap）的锁，保证了在事务执行期间不会有其他事务插入或修改涉及到的索引范围。

Next-Key Lock 退化为行锁的过程：

1. 当一个事务对一个索引记录加锁时，MySQL 会检查该记录是否存在。如果存在，则加锁成功；如果不存在，则加锁失败。
2. 如果加锁失败，MySQL 会将索引间隙也加锁，即加锁范围为“当前索引记录的前一个记录到下一个记录之间的间隙”。这个间隙就是 Next-Key Lock。
3. 如果事务后续插入了一条新的记录，而这条新记录的主键值恰好在前述间隙范围内，那么该事务将等待直到前一个事务完成并释放锁。
4. 当发生上述情况时，Next-Key Lock 会退化为行锁，即只锁定新增记录。这样可以防止幻读问题的发生。

### 12、算法：无重复字符的最长子串

LeetCode.3

#### 思路

1. 使用两个指针 `left` 和 `right` 分别表示当前子串的左右边界，初始时都指向字符串的开头。
2. 使用一个哈希表 `hash_map` 来存储字符和字符在当前子串中的位置。
3. 遍历字符串，每次移动右指针 `right`，并将当前字符加入哈希表中。
4. 如果当前字符已经在哈希表中，并且其位置在 `left` 和 `right` 之间（即在当前子串中），则更新 `left` 的位置为该字符在哈希表中的位置加 1，表示将左边界移动到重复字符的下一个位置。
5. 在遍历过程中不断更新最长子串的长度。

#### 参考代码

##### C++

```
#include <iostream>
#include <unordered_map>
#include <algorithm>

using namespace std;

int lengthOfLongestSubstring(string s) {
    unordered_map<char, int> hash_map; // 哈希表，用于存储字符和字符在当前子串中的位置
    int left = 0, right = 0; // 左右指针，表示当前子串的左右边界
    int max_length = 0; // 最长子串的长度
    int n = s.size(); // 字符串的长度

    while (right < n) {
        char c = s[right]; // 当前字符
        // 如果当前字符已经在哈希表中，并且其位置在 left 和 right 之间（即在当前子串中）
        if (hash_map.find(c) != hash_map.end() && hash_map[c] >= left) {
            left = hash_map[c] + 1; // 更新 left 的位置为重复字符的下一个位置
        }
        hash_map[c] = right; // 将当前字符加入哈希表中
        max_length = max(max_length, right - left + 1); // 更新最长子串的长度
        right++; // 移动右指针
    }

    return max_length;
}

int main() {
    string s = "abcabcbb";
    cout << lengthOfLongestSubstring(s) << endl; // 输出：3

    s = "bbbbb";
    cout << lengthOfLongestSubstring(s) << endl; // 输出：1

    s = "pwwkew";
    cout << lengthOfLongestSubstring(s) << endl; // 输出：3

    return 0;
}
```

##### Java

```
import java.util.HashMap;

public class Main {
    public static int lengthOfLongestSubstring(String s) {
        HashMap<Character, Integer> map = new HashMap<>(); // 哈希表，用于存储字符和字符在当前子串中的位置
        int left = 0, right = 0; // 左右指针，表示当前子串的左右边界
        int maxLength = 0; // 最长子串的长度

        while (right < s.length()) {
            char c = s.charAt(right); // 当前字符
            // 如果当前字符已经在哈希表中，并且其位置在 left 和 right 之间（即在当前子串中）
            if (map.containsKey(c) && map.get(c) >= left) {
                left = map.get(c) + 1; // 更新 left 的位置为重复字符的下一个位置
            }
            map.put(c, right); // 将当前字符加入哈希表中
            maxLength = Math.max(maxLength, right - left + 1); // 更新最长子串的长度
            right++; // 移动右指针
        }

        return maxLength;
    }

    public static void main(String[] args) {
        String s = "abcabcbb";
        System.out.println(lengthOfLongestSubstring(s)); // 输出：3

        s = "bbbbb";
        System.out.println(lengthOfLongestSubstring(s)); // 输出：1

        s = "pwwkew";
        System.out.println(lengthOfLongestSubstring(s)); // 输出：3
    }
}
```

##### Python

```
def lengthOfLongestSubstring(s: str) -> int:
    char_map = {}  # 哈希表，用于存储字符和字符在当前子串中的位置
    left = 0  # 左指针，表示当前子串的左边界
    max_length = 0  # 最长子串的长度

    for right in range(len(s)):
        if s[right] in char_map and char_map[s[right]] >= left:
            left = char_map[s[right]] + 1  # 更新左边界为重复字符的下一个位置
        char_map[s[right]] = right  # 将当前字符加入哈希表中
        max_length = max(max_length, right - left + 1)  # 更新最长子串的长度

    return max_length

# 测试样例
s = "abcabcbb"
print(lengthOfLongestSubstring(s))  # 输出：3

s = "bbbbb"
print(lengthOfLongestSubstring(s))  # 输出：1

s = "pwwkew"
print(lengthOfLongestSubstring(s))  # 输出：3
```

