### 1、数据库的4个特性（不是事务的特性吗）

1. 原子性（Atomicity）：
   - 定义： 原子性指的是事务中的操作要么全部执行成功，要么全部执行失败。如果一个事务包含多个操作，如果其中任何一个操作失败，整个事务都应该被回滚（撤销），使数据库回到事务之前的状态。
   - 示例： 考虑从银行账户 A 转账到账户 B 的操作。如果转账操作成功但存入账户 B 失败，那么整个事务应该被回滚，以确保账户 A 和账户 B 的一致性。
2. 一致性（Consistency）：
   - 定义： 一致性确保事务执行前后数据库从一个一致性状态转移到另一个一致性状态。这意味着事务执行后，数据库应该满足所有预定义的完整性约束和规则。
   - 示例： 假设有一个数据库中存储了学生的成绩信息，包括总分不能为负数的约束。如果一个事务试图将某个学生的总分设置为负数，该事务应该被拒绝，因为它违反了完整性约束。
3. 隔离性（Isolation）：
   - 定义： 隔离性要求每个事务都应该与其他事务相互隔离，互不干扰。即使多个事务并发执行，每个事务也应该看到一个似乎是独占数据库的视图。
   - 示例： 假设两个事务同时尝试更新同一行数据，隔离性要求它们之间不应该互相干扰，一个事务的更新不应该影响到另一个事务。
4. 持久性（Durability）：
   - 定义： 持久性确保一旦事务提交，其结果将永久存储在数据库中，即使系统发生故障或崩溃，事务的结果也不会丢失。
   - 示例： 如果用户执行了一个转账操作，并且该操作已成功提交，持久性要求即使数据库服务器崩溃，该转账操作的结果也应该在数据库恢复后保留。

### 2、4特性为什么和事务相关？

1.  原子性要求事务中的操作要么全部成功执行，要么全部失败回滚。这是因为在一个事务中的操作通常相互依赖，只有当所有操作都成功时，事务的结果才能被提交。如果其中任何一个操作失败，整个事务必须回滚，以保持数据库的一致性。
2. 一致性确保事务执行前后数据库状态保持一致。因为事务执行通常会改变数据库中的数据。如果事务违反了数据库的完整性约束或规则，那么它应该被拒绝，以确保一致性。
3. 隔离性要求每个事务都应该独立运行，不受其他并发事务的干扰。因为在多用户并发访问数据库时，如果不处理好事务的隔离性，可能会导致数据不一致或竞态条件等问题。
4. 持久性确保一旦事务提交，其结果将永久存储在数据库中，即使系统发生故障或崩溃。因为事务的提交通常表示事务已经成功完成，它的结果应该得到持久保存，以防止数据丢失。

### 3、描述每个特性对应事务的场景？

1. 原子性适用于需要多个操作作为一个不可分割单元执行的事务场景。在这些场景中，要么所有操作都成功完成，要么所有操作都失败并回滚。

   场景： 考虑一个银行转账操作。一个事务可能包括从一个账户扣除金额并将相同金额存入另一个账户。在这种情况下，如果从一个账户扣除金额成功但将其存入另一个账户失败，整个事务必须回滚，以确保资金的一致性。

2.  一致性适用于需要确保事务在执行前后数据库保持一致性状态的场景。这意味着事务执行后，数据库应该满足所有预定义的完整性约束和规则。

   场景： 假设有一个数据库存储学生成绩信息，其中有一个完整性约束规定成绩不能为负数。如果一个事务试图将某个学生的成绩设置为负数，该事务应该被拒绝，以确保一致性。

3. 隔离性适用于多个事务并发执行的场景。它确保每个事务都应该与其他事务相互隔离，互不干扰。

   示例场景： 假设有多个用户并发地读取和更新银行账户。隔离性要求一个用户的操作不应该影响到其他用户的操作。例如，两个用户同时尝试更新同一账户的余额，隔离性要求它们之间不应该互相干扰。

4. 持久性确保一旦事务提交，其结果将永久存储在数据库中，即使系统发生故障或崩溃。

   示例场景： 当用户执行一个重要的数据库操作（如支付）并且该操作成功提交时，持久性要求即使数据库服务器在提交后崩溃，该支付操作的结果也应该在数据库恢复后仍然有效，以防止数据丢失。

### 4、每个场景都是用什么技术保证的？

1. 原子性通常通过事务日志（Transaction Log）来实现。数据库管理系统（DBMS）会在事务开始之前记录当前数据库状态，然后在事务执行期间记录所有操作。如果事务中的任何操作失败或事务被回滚，DBMS可以使用事务日志来还原数据库到事务开始时的状态。
2. 一致性通常依赖于数据库中的完整性约束和规则来实现。这包括主键约束、唯一性约束、外键约束以及其他自定义约束。在事务执行之前和之后，DBMS会检查这些约束，如果事务违反了任何约束，将拒绝该事务。
3. 隔离性通常通过并发控制机制来实现，以确保多个并发事务之间不会相互干扰。这些机制包括锁定（Locking）和多版本并发控制（MVCC）等。
   - 锁定（Locking）： 在锁定机制下，事务可以锁定数据，阻止其他事务对同一数据的访问，直到锁被释放。这确保了数据的一致性，但也可能导致死锁（Deadlock）问题。
   - 多版本并发控制（MVCC）： 在MVCC下，每个事务都可以看到数据库的快照，而不是实际数据。每个事务在开始时创建一个快照，其他事务对数据的更改不会影响到正在执行的事务。这提供了更高的并发性，并减少了死锁问题。
4. 持久性通过数据库的持久化机制来实现，确保一旦事务提交，其结果将永久存储在数据库中。这包括将事务日志写入持久存储介质（如硬盘）以及确保在系统故障后能够正确恢复数据库。

### 5、自己写项目代码的时候怎么用到这些特性的 6、事务隔离性？

1. 读未提交（Read Uncommitted）： 这是最低的隔离级别，它允许一个事务读取另一个事务未提交的数据。这种级别可能导致脏读（Dirty Read），也就是读取到了未提交的、可能会回滚的数据。
2. 读已提交（Read Committed）： 在这个隔离级别下，一个事务只能读取已经提交的数据。这样可以防止脏读，但仍然可能出现不可重复读和幻读的情况。
3. 可重复读（Repeatable Read）： 这个隔离级别确保在一个事务执行期间，同一数据的读取结果是一致的，即使其他事务修改了该数据也不影响。这可以防止脏读和不可重复读，但仍然可能出现幻读。
4. 串行化（Serializable）： 这是最高的隔离级别，它确保所有事务按照串行顺序执行，不会出现并发问题。这可以防止脏读、不可重复读和幻读，但对性能有较大的影响，因为它限制了并发性。

### 7、了解什么引擎？

1. InnoDB： InnoDB 是 MySQL 数据库系统的默认存储引擎，也是一个非常流行的开源事务性存储引擎。它支持事务、行级锁定、外键约束和崩溃恢复等功能，适用于需要强大的事务支持和数据完整性的应用程序。
2. MyISAM： MyISAM 是另一个 MySQL 存储引擎，不支持事务和行级锁定，但对于读密集型应用程序和具有高并发读取请求的场景效果良好。它适用于日志、缓存和其他不需要事务支持的应用。
3. PostgreSQL： PostgreSQL 使用了一种名为 MVCC（多版本并发控制）的存储引擎，支持高级的数据类型、完整性约束和复杂查询。它被广泛用于需要高度定制化和复杂数据模型的应用程序。
4. SQLite： SQLite 是一个嵌入式数据库引擎，适用于轻量级应用和移动应用。它的数据库以单个文件的形式存储，不需要独立的数据库服务器进程。
5. Oracle Database： Oracle 使用了自家开发的存储引擎，支持高度可扩展性、企业级特性和大规模应用的要求。它是一种强大而复杂的数据库引擎，广泛用于企业级应用。
6. SQL Server： Microsoft SQL Server 使用了一种名为 SQL Server 数据引擎（SQL Server Database Engine）的存储引擎，支持事务、分布式事务、强大的查询优化和大规模数据处理。
7. MongoDB： MongoDB 是一个文档型数据库，使用 BSON（一种二进制 JSON 格式）存储数据。它的存储引擎支持水平扩展和灵活的数据模型。

### 8、MyISAM和InnoDB的区别？

1. 事务支持：
   - MyISAM： MyISAM 不支持事务。它的操作是基于表级锁定的，这意味着在一个事务中的某个操作可能会锁定整个表，导致其他事务被阻塞。
   - InnoDB： InnoDB 支持事务。它使用行级锁定，允许多个事务并发执行而不会互相阻塞。这使得 InnoDB 更适合需要高度事务支持的应用程序。
2. 锁定机制：
   - MyISAM： MyISAM 使用表级锁定。这意味着在读取或修改数据时，会锁定整个表，而其他事务必须等待锁释放才能执行操作。
   - InnoDB： InnoDB 使用行级锁定。这意味着只锁定需要访问的行，允许其他事务同时访问表中的不同行数据，提高了并发性。
3. 外键约束：
   - MyISAM： MyISAM 不支持外键约束。这意味着数据库不会强制执行外键关系，开发人员需要自己来维护数据的完整性。
   - InnoDB： InnoDB 支持外键约束。它可以确保关联表之间的数据完整性，当尝试插入或更新数据时，如果违反了外键关系，会触发错误并拒绝操作。
4. 崩溃恢复：
   - MyISAM： MyISAM 在崩溃后需要进行表级别的修复，可能会导致数据丢失。
   - InnoDB： InnoDB 支持崩溃恢复，它会记录事务日志，可以在崩溃后还原事务的一致性状态，减少数据丢失的风险。
5. 性能特点：
   - MyISAM： MyISAM 在读取大量静态数据时性能较高，适用于读密集型操作。但对于写入和更新频繁的应用程序，性能可能受到影响，特别是在并发环境下。
   - InnoDB： InnoDB 更适合需要事务支持、写入频繁和并发操作的应用程序。它的性能在高并发写入的情况下通常更稳定。

### 9、InoodbB+树细节？

先来说说B树：

B 树不要和二叉树混淆，B 树不是二叉树，而是一种自平衡树数据结构。 它维护有序数据并允许以对数时间进行搜索，顺序访问，插入和删除。B 树是二叉搜索树的一般化，因为 B 树的节点可以有两个以上的子节点。

与其他自平衡二进制搜索树不同，B 树非常适合读取和写入相对较大的数据块（如光盘）的存储系统。它通常用于数据库和文件系统，例如 mysql 的 InnoDB 引擎使用的数据结构就是 B 树的变形 B+ 树。

B 树是一种平衡的多分树，通常我们说 m 阶的 B 树，它必须满足如下条件：

- 每个节点最多只有 m 个子节点。
- 每个非叶子节点（除了根）具有至少 ⌈m/2⌉ 子节点。
- 如果根不是叶节点，则根至少有两个子节点。
- 具有 k 个子节点的非叶节点包含 k -1 个键。
- 所有叶子都出现在同一水平，没有任何信息（高度一致）。

![image-20240110011557251](E:\GitHub\Interview-experience\深信服面经\assets\image-20240110011557251.png)

B 树的阶，指的是 B 树中节点的子节点数目的最大值。例如在上图的书中，「13,16,19」拥有的子节点数目最多，一共有四个子节点（灰色节点）。所以该 B 树的阶为 4，该树称为 4 阶 B 树。在实际应用中，B 树应用于 MongoDb 的索引。

再来说说B+树：

B+ 树是应文件系统所需而产生的 B 树的变形树。B+ 树的特征：

有 m 个子树的中间节点包含有 m 个元素（B 树中是 k-1 个元素），每个元素不保存数据，只用来索引。

所有的叶子结点中包含了全部关键字的信息，及指向含有这些关键字记录的指针，且叶子结点本身依关键字的大小自小而大的顺序链接。而 B 树的叶子节点并没有包括全部需要查找的信息。

所有的非终端结点可以看成是索引部分，结点中仅含有其子树根结点中最大（或最小）关键字。而 B 树的非终节点也包含需要查找的有效信息。例如下图中的根节点 8 是左子树中最大的元素，15 是右子树中最大的元素。

![image-20240110011610728](E:\GitHub\Interview-experience\深信服面经\assets\image-20240110011610728.png)

与 B 树相比，B+ 树有着如下的好处：

1. B+ 树的磁盘读写代价更低

   B+ 树的内部结点并没有指向关键字具体信息的指针，所以其内部结点相对 B 树更小。如果把所有同一内部结点的关键字存放在同一盘块中，那么盘块所能容纳的关键字数量也越多，所以一次性读入内存中的需要查找的关键字也就越多。相对来说 IO 读写次数也就降低了，查找速度就更快了。

2. B+ 树查询效率更加稳定

   由于非终结点并不是最终指向文件内容的结点，而只是叶子结点中关键字的索引。所以 B+ 树中任何关键字的查找必须走一条从根结点到叶子结点的路。所有关键字查询的路径长度相同，导致每一个数据的查询效率相当。而对于 B 树来说，因为其每个节点都存具体的数据，因此其查询速度可能更快，但是却并不稳定。

3. B+ 树便于范围查询（最重要的原因，范围查找是数据库的常态）

   B 树在提高了 IO 性能的同时，并没有解决元素遍历效率低下的问题。为了解决这个问题，B+ 树应用而生。B+ 树只需要去遍历叶子节点就可以实现整棵树的遍历。在数据库中基于范围的查询是非常频繁的，因此 MySQL 的 Innodb 引擎就使用了 B+ 树作为其索引的数据结构。

最后再说InnoDB中的B+树索引：

1. 树结构： InnoDB 的 B+ 树是一个多层次的树结构，包含根节点、内部节点和叶子节点。每个节点都可以包含多个键值对（通常是数据行的主键和对应的数据指针）。
2. 排序和平衡： B+ 树中的键值对按照键的顺序进行排序，这使得范围查询非常高效。树是自平衡的，意味着在插入和删除操作后，树的平衡会被恢复，以保持较低的搜索复杂度。
3. 叶子节点： B+ 树的叶子节点包含了数据行的实际内容，这使得 B+ 树非常适合用作索引结构，因为索引通常需要快速查找并检索数据行。叶子节点按照主键的顺序排列，可以支持范围查询。
4. 内部节点： 内部节点用于引导搜索，它们包含键的范围信息，以指导下一步搜索的方向。内部节点通常不包含实际数据，它们仅包含指向下一级节点的指针。
5. 主键索引： InnoDB 的 B+ 树的主键索引是聚集索引（Clustered Index），这意味着数据行的实际内容存储在主键索引的叶子节点上。这种设计使得主键索引非常高效，因为查询主键时可以直接访问数据行。
6. 辅助索引： InnoDB 也支持辅助索引（Secondary Index）。辅助索引的叶子节点包含了主键值，通过主键值可以快速定位到实际的数据行。这也使得辅助索引非常高效。
7. 页面和缓冲池： InnoDB 使用一个缓冲池（Buffer Pool）来管理内存中的数据页。数据页是 B+ 树的基本单位，它们包含了多个键值对。缓冲池用于缓存数据页，以减少磁盘 I/O 操作。
8. 锁和事务： InnoDB 的 B+ 树支持多版本并发控制（MVCC），允许不同事务同时读取和修改数据。它使用各种锁来控制并发访问，以确保数据的一致性和隔离性。

### 10、面都是理论，你自己实现过一个B+树吗？？？？         破防*1

实现一个完整的 B+ 树是一个相当复杂的任务，需要涉及到节点的插入、删除、查找，以及平衡调整等许多细节。

阿Q在这里给出一个简单的示例，一些比较细节的就不写了，供大家参考：

```C
#include <iostream>
#include <vector>

const int M = 4; // B+ 树的阶数，可根据需求调整

struct Node {
    std::vector<int> keys;
    Node* children[M + 1];
    bool is_leaf;
};

class BPlusTree {
public:
    BPlusTree() {
        root = nullptr;
    }

    void insert(int key) {
        if (!root) {
            root = new Node;
            root->is_leaf = true;
            root->keys.push_back(key);
        } else {
            insert_recursive(root, key);
        }
    }

    bool search(int key) {
        return search_recursive(root, key);
    }

private:
    Node* root;

    void insert_recursive(Node* node, int key) {
        if (node->is_leaf) {
            // 插入到叶子节点的有序位置
            int i = 0;
            while (i < node->keys.size() && key > node->keys[i]) {
                i++;
            }
            node->keys.insert(node->keys.begin() + i, key);

            // 检查是否需要分裂
            if (node->keys.size() > M) {
                split(node);
            }
        } else {
            // 在合适的子节点中递归插入
            int i = 0;
            while (i < node->keys.size() && key > node->keys[i]) {
                i++;
            }
            insert_recursive(node->children[i], key);
        }
    }

    void split(Node* node) {
        Node* new_node = new Node;
        new_node->is_leaf = node->is_leaf;

        // 将一半的键值移动到新节点
        int mid = node->keys.size() / 2;
        for (int i = mid; i < node->keys.size(); i++) {
            new_node->keys.push_back(node->keys[i]);
        }
        node->keys.resize(mid);

        // 更新父节点
        if (!node->is_leaf) {
            for (int i = mid; i < node->keys.size() + 1; i++) {
                new_node->children.push_back(node->children[i]);
            }
            node->children.resize(mid + 1);
        }

        // 如果是根节点，创建新的根节点
        if (node == root) {
            Node* new_root = new Node;
            new_root->is_leaf = false;
            new_root->keys.push_back(new_node->keys[0]);
            new_root->children.push_back(node);
            new_root->children.push_back(new_node);
            root = new_root;
        } else {
            // 否则，将中间值插入到父节点
            insert_recursive(get_parent(root, node), new_node->keys[0]);
        }
    }

    Node* get_parent(Node* current, Node* child) {
        // 递归查找父节点
        for (int i = 0; i < current->children.size(); i++) {
            if (current->children[i] == child) {
                return current;
            }
            if (!current->children[i]->is_leaf) {
                Node* result = get_parent(current->children[i], child);
                if (result != nullptr) {
                    return result;
                }
            }
        }
        return nullptr;
    }

    bool search_recursive(Node* node, int key) {
        if (!node) {
            return false;
        }
        int i = 0;
        while (i < node->keys.size() && key > node->keys[i]) {
            i++;
        }
        if (i < node->keys.size() && key == node->keys[i]) {
            return true;
        } else if (node->is_leaf) {
            return false;
        } else {
            return search_recursive(node->children[i], key);
        }
    }
};

int main() {
    BPlusTree bptree;
    bptree.insert(10);
    bptree.insert(20);
    bptree.insert(5);
    bptree.insert(15);

    std::cout << "Search 10: " << (bptree.search(10) ? "Found" : "Not Found") << std::endl;
    std::cout << "Search 25: " << (bptree.search(25) ? "Found" : "Not Found") << std::endl;

    return 0;
}
```

### 11、慢查询出现的原因？怎么解决慢查询？

指在数据库中执行的查询操作需要较长时间才能完成。

下边我给出一些常见的导致慢查询的原因，以及如何解决它们：

1. 复杂的查询： 查询语句本身可能非常复杂，包含多个连接、子查询、聚合函数或大量的过滤条件。这会导致数据库需要更多的计算资源和时间来执行查询。

- 解决方法： 优化查询语句，尽量简化查询逻辑。使用适当的索引来加速查询，并确保查询中的字段都是需要的字段，避免不必要的数据检索。

2. 缺乏索引： 如果查询中的字段没有合适的索引，数据库将不得不进行全表扫描，这通常是慢查询的主要原因之一。

- 解决方法： 分析查询语句的执行计划，确定缺少的索引，并创建适当的索引以加速查询。但要注意，过多的索引也可能导致性能下降，因此需要谨慎添加索引。

3. 数据量过大： 数据表中包含大量的数据记录时，即使使用了索引，查询也可能变得很慢。

- 解决方法： 使用分页查询或者限制返回结果集的大小，避免一次性检索大量数据。另外，可以考虑使用分区表或归档数据来管理大量数据。

4. 锁和并发问题： 如果多个查询同时访问同一数据，可能会导致锁冲突和阻塞，从而使查询变慢。

- 解决方法： 优化数据库的并发控制机制，使用合适的事务隔离级别，减少锁的竞争。使用索引来减少锁定的范围。

5. 硬件和资源限制： 数据库服务器的硬件性能不足、内存不足或者磁盘 I/O 较慢都可能导致查询变慢。

- 解决方法： 升级硬件、增加内存、使用更快的存储设备或者优化数据库服务器的配置，以满足查询的资源需求。

6. 存储引擎选择： 不同的存储引擎在处理查询性能上有差异，选择不合适的存储引擎可能导致慢查询。

- 解决方法： 根据应用程序的需求和负载特性选择适当的存储引擎。例如，InnoDB 适合需要事务支持和并发控制的应用，而 MyISAM 适合读密集型操作。

7. 不合理的数据库设计： 数据库表结构和关联可能不合理，导致查询效率低下。

- 解决方法： 重新设计数据库结构，考虑合适的范式和关联，以优化数据存储和查询效率。

8. 缓存未命中： 如果查询的数据已被缓存，查询速度会很快，但如果缓存未命中，需要从磁盘读取数据，会导致慢查询。

- 解决方法： 使用缓存机制（如缓存服务器或应用程序层缓存）来提高查询命中率，减少对数据库的请求。

9. 不合理的查询频率： 频繁地执行相同或类似的查询可能导致数据库负载过重，影响性能。

- 解决方法： 优化应用程序代码，减少不必要的查询。使用查询缓存或者调整查询频率。

10. 数据库统计信息不准确： 数据库管理系统依赖于统计信息来生成查询计划，如果统计信息不准确，可能导致性能下降。

- 解决方法： 定期更新数据库的统计信息，以确保查询优化器可以生成合适的执行计划。

### 12、tcp/udp的区别？

1. 连接性：
   - TCP：TCP 是一种面向连接的协议。在建立通信之前，它需要在通信双方之间建立连接，进行三次握手（SYN、SYN-ACK、ACK）和四次挥手（FIN、ACK、FIN-ACK、ACK）来确保数据可靠地传输。这意味着TCP提供了可靠的数据传输，数据包按顺序到达，并且可以重传丢失的数据包。
   - UDP：UDP 是一种无连接的协议，它不需要在通信之前建立连接。每个UDP数据包都是独立的，发送者不会等待接收者的确认或重传丢失的数据包。这使得UDP的传输速度较快，但不提供可靠性保证。
2. 可靠性：
   - TCP：TCP 提供了可靠性传输。它使用序号、确认和重传机制来确保数据的可靠交付。如果数据包丢失或损坏，TCP会负责重新传输，直到数据被正确接收。
   - UDP：UDP 不提供可靠性保证。它发送数据包后不会等待确认，也不会重传数据包。因此，UDP可以更快地传输数据，但不能确保数据的可靠性。
3. 数据流量控制：
   - TCP：TCP 使用流量控制机制来避免网络拥塞。它通过窗口控制来限制发送方发送数据的速率，以确保网络不会过载。
   - UDP：UDP 不提供流量控制机制，发送方可以以任何速度发送数据，但可能导致网络拥塞。
4. 顺序保证：
   - TCP：TCP 提供了顺序保证，即发送的数据包按照发送顺序到达接收端，并以相同的顺序交付给应用程序。
   - UDP：UDP 不提供顺序保证，发送的数据包可以以任何顺序到达接收端，并且不保证按照发送顺序交付给应用程序。
5. 头部开销：
   - TCP：TCP 的头部较大，包含序号、确认号、窗口大小等信息，通常会导致较大的头部开销。
   - UDP：UDP 的头部相对较小，包含源端口和目标端口等基本信息，头部开销较小。
6. 适用场景：
   - TCP：TCP 适用于需要可靠数据传输和顺序交付的应用，如网页浏览、电子邮件、文件传输等。
   - UDP：UDP 适用于对数据传输延迟要求较低，可以容忍丢失的应用，如实时音视频通话、在线游戏、DNS 查询等。

### 13、我看你主要是java，那如何用socket自己实现一个tcp    ？？？？我是java的，，这不是c++的webserver吗?

阿Q这里给出一个最简单的示例，以更好的说明问题。

TCP服务器：

```C
#include <iostream>
#include <string>
#include <cstring>
#include <cstdlib>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>

int main() {
    // 创建服务器套接字
    int server_socket = socket(AF_INET, SOCK_STREAM, 0);
    if (server_socket == -1) {
        std::cerr << "Error: Could not create socket." << std::endl;
        return 1;
    }

    // 绑定服务器地址和端口
    sockaddr_in server_address;
    server_address.sin_family = AF_INET;
    server_address.sin_port = htons(12345);
    server_address.sin_addr.s_addr = INADDR_ANY;
    if (bind(server_socket, (struct sockaddr*)&server_address, sizeof(server_address)) == -1) {
        std::cerr << "Error: Could not bind to address." << std::endl;
        return 1;
    }

    // 监听连接
    if (listen(server_socket, 5) == -1) {
        std::cerr << "Error: Could not listen on socket." << std::endl;
        return 1;
    }

    std::cout << "Waiting for a client to connect..." << std::endl;

    // 接受客户端连接
    sockaddr_in client_address;
    socklen_t client_address_size = sizeof(client_address);
    int client_socket = accept(server_socket, (struct sockaddr*)&client_address, &client_address_size);
    if (client_socket == -1) {
        std::cerr << "Error: Could not accept client connection." << std::endl;
        return 1;
    }

    std::cout << "Client connected." << std::endl;

    // 接收和发送数据
    char buffer[1024];
    std::string message = "Hello, Client!";
    send(client_socket, message.c_str(), message.size(), 0);

    close(client_socket);
    close(server_socket);

    return 0;
}
```

TCP客户端：

```C
#include <iostream>
#include <cstring>
#include <cstdlib>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>

int main() {
    // 创建客户端套接字
    int client_socket = socket(AF_INET, SOCK_STREAM, 0);
    if (client_socket == -1) {
        std::cerr << "Error: Could not create socket." << std::endl;
        return 1;
    }

    // 设置服务器地址和端口
    sockaddr_in server_address;
    server_address.sin_family = AF_INET;
    server_address.sin_port = htons(12345);
    if (inet_pton(AF_INET, "127.0.0.1", &(server_address.sin_addr)) == -1) {
        std::cerr << "Error: Invalid address." << std::endl;
        return 1;
    }

    // 连接到服务器
    if (connect(client_socket, (struct sockaddr*)&server_address, sizeof(server_address)) == -1) {
        std::cerr << "Error: Could not connect to server." << std::endl;
        return 1;
    }

    // 接收和显示服务器发送的数据
    char buffer[1024];
    int bytes_received = recv(client_socket, buffer, sizeof(buffer), 0);
    if (bytes_received == -1) {
        std::cerr << "Error: Could not receive data." << std::endl;
        return 1;
    }

    buffer[bytes_received] = '\0';
    std::cout << "Server says: " << buffer << std::endl;

    close(client_socket);

    return 0;
}
```

服务器和客户端分别创建了套接字，绑定地址（服务器端），连接到服务器（客户端），并进行数据的接收和发送。 14、RPC远端调用 代理

RPC（远程过程调用）是一种分布式系统中的通信方式，它允许一个计算机程序调用另一个计算机上的函数或方法，就像调用本地函数一样。为了实现RPC，通常会使用代理（Proxy）来处理远程调用。

RPC工作原理：

1. 客户端调用： 客户端应用程序通过本地的客户端代理（Client Proxy）调用远程服务器上的服务或方法。
2. 序列化： 客户端代理将调用参数序列化为字节流，以便通过网络发送。
3. 网络传输： 序列化的参数通过网络传输到远程服务器。
4. 服务端接收： 远程服务器接收到请求，并将字节流反序列化为参数。
5. 服务端调用： 服务器上的服务实现（Server Object）执行相应的操作，可能涉及计算、数据库查询等。
6. 序列化响应： 服务器将响应序列化为字节流。
7. 网络传输响应： 序列化的响应通过网络传输回客户端。
8. 客户端接收： 客户端代理接收到响应，将字节流反序列化为结果。
9. 客户端返回： 客户端应用程序获得最终结果，就像调用本地函数一样。

代理在RPC中扮演了重要的角色，它负责处理本地调用和远程调用之间的交互。代理分为客户端代理和服务器端代理：

- 客户端代理（Client Proxy）： 客户端应用程序使用客户端代理来调用远程服务。客户端代理负责将调用参数序列化为字节流，并将字节流发送到远程服务器。它还负责接收服务器的响应，将响应反序列化为结果，并将结果返回给客户端应用程序。
- 服务器端代理（Server Proxy）： 服务器端代理在远程服务器上接收来自客户端的请求，负责将请求的参数反序列化，并将调用传递给实际的服务实现。它还将服务的响应序列化为字节流，并将响应发送回客户端。

### 15、负载均衡？

什么是负载均衡？

单个服务器解决不了，增加服务器数量， 然后将请求分发到各个服务器上，将原先请求集中到单个服务器上的情况改为将请求分发到多个服务器上，将负载分发到不同的服务器上，也就是所说的负载均衡。

实现原理：

1. 请求分发： 负载均衡器接收到来自客户端的请求后，根据预定义的算法和策略，将请求分发给一个或多个后端服务器。
2. 服务器选择： 负载均衡器选择目标服务器的方法可以包括轮询（Round Robin）、加权轮询、最少连接、IP散列、负载权重等。不同的方法适用于不同的应用场景。
3. 健康检查： 负载均衡器会定期检查后端服务器的健康状态，如果某台服务器出现故障或不可用，负载均衡器会自动将流量路由到其他健康的服务器。

好处：

- 提高性能： 负载均衡允许请求分布到多个服务器，从而减轻单个服务器的负载，提高整体性能。这对于处理大量请求的应用程序和网站特别有用。
- 提高可用性： 如果某个服务器发生故障，负载均衡器可以自动将流量路由到其他正常运行的服务器，从而提高了系统的可用性和容错性。
- 实现横向扩展： 通过向系统添加新的服务器，负载均衡可以帮助实现横向扩展，以应对不断增加的流量和用户。
- 优化资源利用率： 负载均衡器可以确保服务器资源被充分利用，减少资源浪费，提高效率。

应用场景：

1. Web服务器负载均衡： 在Web应用程序中，负载均衡器可用于将HTTP请求分发给多个Web服务器，以提供更高的网站性能和可用性。
2. 应用程序负载均衡： 用于将应用程序请求分发给多个应用程序服务器，例如应用程序服务器集群。
3. 数据库负载均衡： 用于将数据库查询分发给多个数据库服务器，以平衡数据库负载和提高查询性能。
4. 流媒体服务负载均衡： 用于将流媒体流（如音频和视频）分发给多个媒体服务器，以提供流畅的媒体播放体验。
5. 分布式计算负载均衡： 用于分发计算任务到多个计算节点，以加速大规模计算。

负载均衡器的类型：

- 硬件负载均衡器： 这是一种专用硬件设备，设计用于处理高负载的网络流量。它通常具有高吞吐量和低延迟，适用于大规模企业和数据中心。
- 软件负载均衡器： 这是一种运行在通用硬件或虚拟机上的软件应用程序，通过软件实现负载均衡功能。它具有灵活性和可扩展性，适用于小型和中型环境，以及云计算环境。

### 16、二叉树中序非递归遍历？

实现步骤：

1. 初始化一个空栈，用于存放待访问的节点。
2. 从根节点开始，将根节点及其左子树上的所有节点依次入栈，直到遇到叶子节点为止。
3. 出栈一个节点，访问该节点。
4. 如果出栈的节点有右子树，将右子树中的所有节点依次入栈，重复步骤2。
5. 重复步骤3和步骤4，直到栈为空。

示例代码：

```C
#include <iostream>
#include <stack>

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

void inOrderTraversal(TreeNode* root) {
    if (root == nullptr) {
        return;
    }

    std::stack<TreeNode*> s;
    TreeNode* currentNode = root;

    while (!s.empty() || currentNode != nullptr) {
        // 将当前节点及其左子树上的所有节点入栈
        while (currentNode != nullptr) {
            s.push(currentNode);
            currentNode = currentNode->left;
        }

        // 出栈一个节点，访问该节点
        currentNode = s.top();
        s.pop();
        std::cout << currentNode->val << " ";

        // 切换到右子树，重复上述过程
        currentNode = currentNode->right;
    }
}

int main() {
    // 创建一个二叉树示例
    TreeNode* root = new TreeNode(1);
    root->left = new TreeNode(2);
    root->right = new TreeNode(3);
    root->left->left = new TreeNode(4);
    root->left->right = new TreeNode(5);

    std::cout << "Inorder traversal: ";
    inOrderTraversal(root);
    std::cout << std::endl;

    return 0;
}
```

### 17、最长无重复字串？

使用滑动窗口算法。

示例代码：

```C
#include <iostream>
#include <string>
#include <unordered_map>

int lengthOfLongestSubstring(const std::string& s) {
    // 使用哈希表来存储字符和它们的最后出现位置
    std::unordered_map<char, int> charIndexMap;

    int maxLength = 0; // 记录最长无重复子串的长度
    int windowStart = 0; // 滑动窗口的起始位置

    for (int windowEnd = 0; windowEnd < s.length(); ++windowEnd) {
        char currentChar = s[windowEnd];

        // 如果当前字符在窗口内重复出现，并且重复位置在窗口的起始位置之后
        if (charIndexMap.find(currentChar) != charIndexMap.end() && charIndexMap[currentChar] >= windowStart) {
            // 更新窗口的起始位置，使其跳过重复字符
            windowStart = charIndexMap[currentChar] + 1;
        }

        // 将当前字符的位置记录到哈希表中
        charIndexMap[currentChar] = windowEnd;

        // 计算当前窗口的长度，并更新最大长度
        int currentLength = windowEnd - windowStart + 1;
        maxLength = std::max(maxLength, currentLength);
    }

    return maxLength;
}

int main() {
    std::string input = " ";
    std::cin >> input;
    int result = lengthOfLongestSubstring(input);
    std::cout << "Length of longest substring without repeating characters: " << result << std::endl;
    return 0;
}
```

### 18、出现次数最多的十个ip ？？？破防*3  

使用一个哈希表来统计每个 IP 地址出现的次数，然后选择前十个出现次数最多的 IP 地址。

```C
#include <iostream>
#include <vector>
#include <unordered_map>
#include <algorithm>

// 自定义结构用于存储 IP 地址和其出现次数
struct IPCount {
    std::string ip;
    int count;

    IPCount(const std::string& ip, int count) : ip(ip), count(count) {}

    // 用于排序的比较函数，按照出现次数逆序排序
    bool operator<(const IPCount& other) const {
        return count > other.count;
    }
};

std::vector<std::string> findTopTenIPs(const std::vector<std::string>& ipList) {
    std::unordered_map<std::string, int> ipCountMap;

    // 统计每个 IP 地址的出现次数
    for (const std::string& ip : ipList) {
        ipCountMap[ip]++;
    }

    // 将统计结果存储到自定义结构 IPCount 中
    std::vector<IPCount> ipCounts;
    for (const auto& entry : ipCountMap) {
        ipCounts.push_back(IPCount(entry.first, entry.second));
    }

    // 对 IPCount 结构进行排序，按照出现次数逆序排序
    std::sort(ipCounts.begin(), ipCounts.end());

    // 提取前十个出现次数最多的 IP 地址
    std::vector<std::string> topTenIPs;
    for (int i = 0; i < std::min(10, static_cast<int>(ipCounts.size())); ++i) {
        topTenIPs.push_back(ipCounts[i].ip);
    }

    return topTenIPs;
}

int main() {
    // 示例 IP 地址列表
    std::vector<std::string> ipList = {
        "192.168.1.1",
        "192.168.1.2",
        "192.168.1.1",
        "192.168.1.3",
        "192.168.1.2",
        "192.168.1.4",
        "192.168.1.5",
        "192.168.1.6",
        "192.168.1.4",
        "192.168.1.7",
        // 添加更多 IP 地址
    };

    std::vector<std::string> topTenIPs = findTopTenIPs(ipList);

    std::cout << "Top 10 IP Addresses:" << std::endl;
    for (const std::string& ip : topTenIPs) {
        std::cout << ip << std::endl;
    }

    return 0;
}
```

来源：https://www.nowcoder.com/feed/main/detail/d16f02f5dde7463fae2fe6af544316e5