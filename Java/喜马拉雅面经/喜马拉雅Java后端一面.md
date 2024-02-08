# 喜马拉雅Java后端一面

> 来源：https://www.nowcoder.com/feed/main/detail/b6e4d63041ac4c2088264090cd333abf

## 八股

### 1、一般用什么IO流？

常用的 IO 流主要分为两大类：字节流和字符流。字节流用于处理二进制数据，而字符流用于处理文本数据。

1. 字节流：
   - `FileInputStream`：用于从文件中读取字节流。
   - `FileOutputStream`：用于向文件中写入字节流。
   - `ByteArrayInputStream`：用于从字节数组中读取字节流。
   - `ByteArrayOutputStream`：用于向字节数组中写入字节流。
   - `BufferedInputStream`：提供了缓冲功能，可以提高读取字节流的效率。
   - `BufferedOutputStream`：提供了缓冲功能，可以提高写入字节流的效率。
2. 字符流：
   - `FileReader`：用于从文件中读取字符流。
   - `FileWriter`：用于向文件中写入字符流。
   - `BufferedReader`：提供了缓冲功能，可以提高读取字符流的效率。
   - `BufferedWriter`：提供了缓冲功能，可以提高写入字符流的效率。

在选择 IO 流时，通常根据需求和处理的数据类型来决定使用字节流还是字符流。如果处理的是文本数据，推荐使用字符流；如果处理的是二进制数据或者需要与网络或外部设备进行交互，推荐使用字节流。

### 2、什么时候用ArrayList，什么时候用HashMap？

1. ArrayList：
   - 当需要按照顺序存储一组元素，并且需要频繁地根据索引来访问和修改元素时，通常使用 `ArrayList`。
   - `ArrayList` 内部使用数组实现，可以根据索引快速访问元素，但在插入和删除操作时需要移动元素，因此适用于读取操作频繁、插入和删除操作较少的场景。
2. HashMap：
   - 当需要按照键值对存储一组数据，并且需要根据键快速查找、插入和删除数据时，通常使用 `HashMap`。
   - `HashMap` 内部使用哈希表实现，可以快速查找、插入和删除键值对，但不保证元素的顺序，适用于需要高效查找和修改数据的场景。

### 3、HashMap底层结构？

1. 数组：`HashMap` 内部维护一个数组，称为哈希桶数组（或者称为哈希表），用于存储键值对。
2. 链表或红黑树：每个哈希桶中可能存储多个键值对，当多个键映射到同一个桶时，这些键值对会以链表或红黑树的形式存储在桶中。在 JDK 8 及之后的版本中，当链表长度超过一定阈值（默认为 8）时，会将链表转换为红黑树，以提高查找效率。
3. 哈希函数：`HashMap` 使用哈希函数将键映射到哈希桶数组的索引位置。哈希函数的设计要尽量避免碰撞（多个键映射到同一个桶），以保证哈希表的性能。
4. 负载因子和扩容：`HashMap` 中有两个重要参数：负载因子（load factor）和容量（capacity）。负载因子表示哈希表的填充程度，当填充程度超过负载因子时，会触发扩容操作，即重新计算哈希桶数组的大小，并将原有的键值对重新分配到新的桶中，以保持哈希表的性能。
5. 键值对：`HashMap` 中存储的元素是键值对，其中键是唯一的，值可以重复。当插入新的键值对时，根据键的哈希值确定存储位置，并将键值对存储在对应的桶中。

### 4、Set查询为什么快？

`Set` 的查询操作快主要是因为它的实现类在内部使用了哈希表（`HashMap`）来存储元素。

1. 哈希表存储：`HashSet` 内部使用哈希表来存储元素，哈希表的查询操作时间复杂度为 O(1)，即平均情况下只需要常数时间就能找到元素。哈希表使用哈希函数将元素的键值映射到哈希表的索引位置上，无需进行比较操作，直接根据哈希值就能定位到存储位置。
2. 哈希函数设计：良好设计的哈希函数能够将元素均匀地分布在哈希表中，降低碰撞的概率。Java 中的 `HashSet` 使用的是对象的 `hashCode()` 方法作为哈希函数，该方法返回对象的哈希码，通常会根据对象的内部状态计算出一个整数哈希码。
3. 数组存储：哈希表内部采用数组结构存储元素，根据哈希值直接计算出元素在数组中的存储位置，因此查询时不需要遍历整个集合，只需计算哈希值并访问对应位置即可。
4. 不保证顺序：由于哈希表不保证元素的顺序，所以在查询时不需要按顺序遍历集合，可以通过哈希函数直接定位元素。

### 5、HashMap为什么要从链表转换为红黑树？

`HashMap` 在 JDK 8 及之后的版本中引入了链表转换为红黑树的优化机制，主要是为了提高在发生哈希冲突时的查询效率。

1. 降低碰撞带来的性能影响：当哈希表中的桶（存储位置）中存在大量的键值对时，可能会发生哈希冲突，即多个键映射到同一个桶中。如果使用链表存储冲突的键值对，当链表长度过长时，查询效率会降低，因为需要遍历整个链表才能找到目标元素。
2. 提高查询效率：将链表转换为红黑树后，可以在最坏情况下将查找操作的时间复杂度从 O(n) 降低到 O(log n)，即使在极端情况下，即红黑树高度为 log n 时，仍能保持较高的查询效率。
3. 优化大容量的情况：在哈希表容量较大（默认超过 64 个桶）且桶中的链表长度超过一定阈值（默认为 8）时，会触发链表转换为红黑树的优化策略。这样可以避免在大容量的哈希表中由于链表过长而导致查询效率下降的问题。
4. 适应性更强：红黑树作为自平衡二叉搜索树，能够保持较为平衡的高度，对于大容量的哈希表来说更加稳定，能够保持较高的查询效率。

### 6、什么时候使用线程？

在 Java 中，使用线程可以实现并发执行，提高程序的性能和效率。

1. 异步任务：当需要执行耗时的操作，但又不希望阻塞主线程时，可以使用线程执行这些操作，如网络请求、文件读写等。
2. 并发处理：当需要同时处理多个任务时，可以使用多线程来并发执行这些任务，提高处理速度。例如，多线程处理大量的数据计算、图像处理等。
3. 事件驱动：当需要响应用户输入或其他事件时，可以使用线程来处理这些事件，保持程序的响应性。例如，GUI 应用程序中的事件处理、服务器端的请求处理等。
4. 定时任务：当需要定期执行某些任务时，可以使用线程来定时触发执行。例如，定时器任务、定时清理等。
5. 资源共享：当多个线程需要共享资源时，可以使用线程来管理资源的访问和同步。例如，多个线程同时访问共享数据结构、共享文件等。

### 7、使用线程池有什么好处？

1. 资源管理：线程池可以限制同时执行的线程数量，避免因为创建过多线程而导致系统资源不足的问题。通过控制线程数量，可以更好地管理系统的资源利用率。
2. 性能提升：线程池可以重用线程，避免了频繁创建和销毁线程的开销，提高了线程的利用率。这样可以减少系统在创建和销毁线程上的时间和资源消耗，从而提升系统的性能。
3. 响应速度：由于线程池中的线程是预先创建好的，可以立即执行任务，而不需要等待新线程的创建，因此可以更快地响应任务的到来，提高系统的响应速度。
4. 线程管理：线程池提供了一些管理和监控线程的方法，可以方便地管理线程的状态、执行情况和异常处理，提高了对线程的管理和控制能力。
5. 减少资源竞争：线程池可以通过合理的调度算法来避免线程之间的资源竞争，提高了并发处理的效率和稳定性。

### 8、给十个线程，添加到线程池里面，怎么判断他全部执行完毕？

1. 使用计数器：创建一个初始值为 10 的计数器，每个线程执行完毕时将计数器减一，当计数器减为 0 时表示所有线程执行完毕。
2. 使用 `CountDownLatch`：创建一个 `CountDownLatch` 对象，初始值为 10，每个线程执行完毕时调用 `countDown()` 方法，当 `CountDownLatch` 的值减为 0 时表示所有线程执行完毕。
3. 使用 `ExecutorService` 的 `awaitTermination()` 方法：使用 `ExecutorService` 的 `submit()` 方法提交任务，然后调用 `shutdown()` 方法关闭线程池，并使用 `awaitTermination(timeout, unit)` 方法等待所有任务执行完毕，超时时间可以根据需要设置。

示例代码：

```
import java.util.concurrent.*;

public class ThreadPoolExample {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(10);
        CountDownLatch latch = new CountDownLatch(10);

        for (int i = 0; i < 10; i++) {
            executor.submit(() -> {
                // 执行任务
                System.out.println(Thread.currentThread().getName() + " is running");
                latch.countDown(); // 任务执行完毕，计数器减一
            });
        }

        try {
            latch.await(); // 等待所有任务执行完毕
            System.out.println("All threads have finished");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        executor.shutdown(); // 关闭线程池
    }
}
```

### 9、为什么会存在线程安全问题，给出具体的例子？

线程安全问题是指在多线程环境下，由于线程间的竞争和并发访问共享资源，可能导致程序出现不可预料的错误或结果不符合预期的情况。

主要原因包括：

1. 竞态条件：当多个线程同时访问共享资源，并且对资源的操作顺序会影响最终的结果时，就会出现竞态条件。例如，多个线程同时对同一个变量进行读取和修改操作，由于操作顺序不确定，可能导致结果出现不一致的情况。
2. 数据不一致：多个线程同时访问共享的数据结构，如果没有正确地同步操作，可能会导致数据的不一致性。例如，一个线程正在对数据结构进行修改，而另一个线程同时读取该数据结构，可能会读取到不一致的数据。
3. 死锁：多个线程因为互相等待对方释放资源而导致的无限等待状态。例如，线程 A 持有资源 X，等待获取资源 Y；线程 B 持有资源 Y，等待获取资源 X，这样就会导致死锁。
4. 资源竞争：多个线程同时竞争某个资源，如果没有合适的同步机制，可能会导致资源的不合理分配和使用。例如，多个线程同时竞争文件写入权限，可能会导致文件写入顺序混乱或数据丢失。

给个例子：

```
public class RaceConditionExample {
    private static int counter = 0;

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    counter++; // 竞态条件，多个线程同时对 counter 进行递增操作
                }
            }).start();
        }

        try {
            Thread.sleep(1000); // 等待所有线程执行完毕
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("Counter: " + counter); // 可能得到的结果不是预期的 100000
    }
}
```

### 10、sychronized锁住方法是锁住了对象还是锁住了什么？

当使用 `synchronized` 关键字修饰一个方法时，实际上是对当前对象的实例进行加锁。具体来说，`synchronized` 锁住的是当前对象实例的监视器（monitor），也就是当前对象的内置锁（也称为对象锁）。每个对象实例都有一个与之关联的监视器，用于实现对象级别的同步。

当一个线程进入一个 `synchronized` 方法时，它会尝试获取当前对象实例的监视器。如果该监视器没有被其他线程占用，则该线程可以顺利执行方法内部的代码；如果该监视器已经被其他线程占用，那么该线程将被阻塞，直到获取到监视器为止。

### 11、git基本使用，你常用什么命令？



## 算法

### 1、数字反转：Long：1234-&gt;4321，不准转换为string

#### 思路

1. 取数字的每一位：利用求余运算（`%`）可以取得数字的个位数，然后利用整除运算（`/`）去掉最后一位，得到剩余的数字。
2. 构建反转后的数字：从最后一位开始，依次将每一位数字乘以 10 的幂次方，加到结果上，得到反转后的数字。
3. 处理负数：如果原数字是负数，则在计算过程中要保持负号，并在最后结果中添加负号。

#### 参考代码

```
public class ReverseNumber {
    public static long reverse(long num) {
        long reversed = 0;
        boolean isNegative = num < 0;
        num = Math.abs(num); // 取绝对值进行反转

        while (num > 0) {
            long digit = num % 10; // 取得当前数字的个位数
            reversed = reversed * 10 + digit; // 构建反转后的数字
            num /= 10; // 去掉最后一位
        }

        return isNegative ? -reversed : reversed;
    }

    public static void main(String[] args) {
        long num = 1234;
        System.out.println(reverse(num)); // 输出 4321
    }
}
```

### 2、字符串反转：“ hello world ”-&gt;”world hello”

#### 思路

1. 分割字符串：使用 `split()` 方法按空格将字符串分割成字符串数组。
2. 反转字符串数组：使用两个指针，分别指向字符串数组的头部和尾部，交换两个指针指向的字符串，然后头指针向后移动一位，尾指针向前移动一位，继续交换，直到两个指针相遇。
3. 连接字符串数组：使用 `String.join()` 方法将反转后的字符串数组连接成一个字符串，每个字符串之间用空格分隔。

#### 参考代码

```
public class ReverseWords {
    public static String reverseWords(String s) {
        String[] words = s.trim().split("\\s+"); // 按空格分割字符串，去除首尾空格
        int left = 0, right = words.length - 1; // 定义两个指针，分别指向数组的头部和尾部

        while (left < right) {
            String temp = words[left]; // 交换两个指针指向的字符串
            words[left] = words[right];
            words[right] = temp;
            left++; // 头指针向后移动一位
            right--; // 尾指针向前移动一位
        }

        return String.join(" ", words); // 将字符串数组连接成一个字符串，每个字符串之间用空格分隔
    }

    public static void main(String[] args) {
        String s = " hello world ";
        System.out.println(reverseWords(s)); // 输出 "world hello"
    }
}
```

### 3、螺旋矩阵

#### 问题描述

给你一个 `m` 行 `n` 列的矩阵 `matrix` ，请按照 **顺时针螺旋顺序** ，返回矩阵中的所有元素。

**示例 1：**

![img](https://raw.githubusercontent.com/aqjsp/Pictures/main/202402082302104.jpg)

```
输入：matrix = [[1,2,3],[4,5,6],[7,8,9]]
输出：[1,2,3,6,9,8,7,4,5]
```

**示例 2：**

![img](https://raw.githubusercontent.com/aqjsp/Pictures/main/202402082302222.jpg)

```
输入：matrix = [[1,2,3,4],[5,6,7,8],[9,10,11,12]]
输出：[1,2,3,4,8,12,11,10,9,5,6,7]
```

**提示：**

- `m == matrix.length`
- `n == matrix[i].length`
- `1 <= m, n <= 10`
- `-100 <= matrix[i][j] <= 100`

#### 思路

1. 定义边界：初始化四个变量 `top`、`bottom`、`left`、`right` 分别表示当前螺旋的上边界、下边界、左边界和右边界，初始值分别为 0、`m - 1`、0、`n - 1`。
2. 螺旋遍历：不断循环遍历，按照顺时针的顺序依次遍历矩阵的元素，并更新边界。
   - 从左到右遍历 `matrix[top][left:right + 1]`，遍历完成后将 `top` 加 1。
   - 从上到下遍历 `matrix[top:bottom + 1][right]`，遍历完成后将 `right` 减 1。
   - 从右到左遍历 `matrix[bottom][right:left - 1:-1]`，遍历完成后将 `bottom` 减 1。
   - 从下到上遍历 `matrix[bottom:top - 1:-1][left]`，遍历完成后将 `left` 加 1。
3. 结束条件：当 `top > bottom` 或 `left > right` 时，表示已经遍历完所有元素。

#### 参考代码

```
import java.util.ArrayList;
import java.util.List;

public class SpiralMatrix {
    public static List<Integer> spiralOrder(int[][] matrix) {
        List<Integer> result = new ArrayList<>();
        if (matrix == null || matrix.length == 0 || matrix[0].length == 0) {
            return result;
        }

        int m = matrix.length;
        int n = matrix[0].length;
        int top = 0, bottom = m - 1, left = 0, right = n - 1;

        // 循环遍历螺旋顺序
        while (top <= bottom && left <= right) {
            // 从左到右
            for (int j = left; j <= right; j++) {
                result.add(matrix[top][j]);
            }
            top++; // 上边界下移一行

            // 从上到下
            for (int i = top; i <= bottom; i++) {
                result.add(matrix[i][right]);
            }
            right--; // 右边界左移一列

            // 从右到左
            if (top <= bottom) { // 检查是否越界
                for (int j = right; j >= left; j--) {
                    result.add(matrix[bottom][j]);
                }
                bottom--; // 下边界上移一行
            }

            // 从下到上
            if (left <= right) { // 检查是否越界
                for (int i = bottom; i >= top; i--) {
                    result.add(matrix[i][left]);
                }
                left++; // 左边界右移一列
            }
        }

        return result;
    }

    public static void main(String[] args) {
        int[][] matrix = {{1, 2, 3}, {4, 5, 6}, {7, 8, 9}};
        System.out.println(spiralOrder(matrix)); // 输出 [1, 2, 3, 6, 9, 8, 7, 4, 5]
    }
}
```

