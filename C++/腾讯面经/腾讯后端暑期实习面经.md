# 腾讯后端暑期实习面经

> 来源：https://www.nowcoder.com/feed/main/detail/456f7bc6b2264dd5944e69639f76c281

## 1、虚拟机和容器的区别是什么？

#### 基本定义

##### 虚拟机

通过虚拟化技术（如 VMware、VirtualBox、Hyper-V 或 KVM）在物理硬件上模拟出一台完整的计算机。它包括一个完整的操作系统（Guest OS）、应用程序以及所需的库和依赖项。虚拟机运行在虚拟化层（Hypervisor）之上，Hypervisor 负责管理物理硬件资源并分配给多个虚拟机。

##### 容器

一种轻量级的虚拟化技术，它将应用程序及其依赖项（如库、配置文件等）打包在一起，但不包含完整的操作系统。容器共享宿主操作系统的内核，通过容器引擎（如 Docker、Podman）进行管理和隔离。

#### 架构差异

##### 虚拟机

- 架构：物理硬件 → Hypervisor（如 Type 1 或 Type 2） → 虚拟机（每个虚拟机包含 Guest OS + 应用 + 依赖项）。
- 每个虚拟机都有独立的操作系统内核，导致资源占用较高。
- Hypervisor 模拟硬件（如虚拟 CPU、内存、磁盘），Guest OS 运行在这些虚拟硬件上。

##### 容器

- 架构：物理硬件 → 宿主操作系统 → 容器引擎（如 Docker） → 容器（仅包含应用 + 依赖项）。
- 容器直接使用宿主操作系统的内核，不需要额外的 Guest OS，因此更轻量。
- 通过命名空间（Namespace）和控制组（cgroups）实现隔离。

#### 工作原理

##### 虚拟机

 Hypervisor（虚拟机管理程序）将物理硬件抽象为多个虚拟硬件实例，每个虚拟机认为自己独占一套硬件。Guest OS 运行在虚拟硬件上，与宿主 OS 完全独立。例如，你可以在 Linux 宿主机上运行多个 Windows VM。

##### 容器

容器通过操作系统的特性（如 Linux 的 Namespace 和 cgroups）实现隔离：

- Namespace：隔离进程、文件系统、网络等，使每个容器有独立的运行环境。
- cgroups：限制资源使用（如 CPU、内存），防止容器之间相互干扰。 容器直接调用宿主内核的系统调用，无需模拟硬件，因此效率更高。

#### 主要区别

| 特性     | 虚拟机 (VM)                          | 容器 (Container)                       |
| -------- | ------------------------------------ | -------------------------------------- |
| 操作系统 | 每个 VM 有独立的 Guest OS            | 共享宿主 OS 的内核，无独立 OS          |
| 资源占用 | 较高（需要为每个 OS 分配资源）       | 较低（仅打包应用和依赖项）             |
| 启动速度 | 较慢（需要启动整个 OS）              | 很快（几秒甚至毫秒级别）               |
| 隔离性   | 强（完全隔离，独立的内核和硬件模拟） | 较弱（共享内核，依赖宿主 OS 隔离机制） |
| 可移植性 | 依赖虚拟化平台，迁移稍复杂           | 高（容器镜像可在支持的平台上运行）     |
| 安全性   | 高（内核隔离，漏洞影响范围小）       | 较低（共享内核，可能受宿主影响）       |
| 典型工具 | VMware, VirtualBox, KVM, Hyper-V     | Docker, Podman, Kubernetes             |

## 2、mysql聚集索引和二级索引？

#### 基本定义

##### 聚集索引

聚集索引是指数据行的物理存储顺序与索引的顺序一致。换句话说，表的数据本身就是按照聚集索引的键值排序存储的。一个表只能有一个聚集索引，因为数据的物理存储顺序只能有一种。

##### 二级索引

二级索引是非聚集索引，索引的键值和数据行的物理存储顺序无关。二级索引存储的是索引键值以及指向实际数据行的引用（通常是主键值）。一个表可以有多个二级索引。

#### 存储结构

##### 聚集索引

- 在 InnoDB 引擎中（MySQL 默认引擎），聚集索引的叶子节点直接存储整行数据。
- 主键（Primary Key）默认是聚集索引。如果表没有定义主键，InnoDB 会选择一个唯一非空索引作为聚集索引；如果没有这样的索引，InnoDB 会生成一个隐藏的 6 字节 ROWID 作为聚集索引。
- 数据和索引是“绑定”在一起的，查询聚集索引时不需要额外的查找，直接就能获取整行数据。

##### 二级索引

- 二级索引的叶子节点存储的是索引键值和指向数据的指针（通常是主键值）。
- 数据本身存储在聚集索引中，因此查询二级索引时，通常需要通过主键值再去聚集索引中查找完整数据（称为“回表”）。

#### 工作原理

假设有一个表 users，结构如下：

```
id (主键)  | name | age
1         | Alice | 25
2         | Bob   | 30
3         | Carol | 35
```

##### 聚集索引

假设 id 是主键（即聚集索引），表的数据按 id 的顺序物理存储：

```
id=1 | Alice | 25
id=2 | Bob   | 30
id=3 | Carol | 35
```

查询 SELECT * FROM users WHERE id = 2 时，InnoDB 直接定位到 id=2 的记录，获取整行数据，无需额外查找。

##### 二级索引

假设在 name 上创建了二级索引，索引结构可能是：

```
name=Alice | id=1
name=Bob   | id=2
name=Carol | id=3
```

查询 SELECT * FROM users WHERE name = 'Bob' 时：

1. 先在二级索引中找到 name='Bob'，得到 id=2。
2. 再到聚集索引中通过 id=2 查找完整数据（回表）。

如果查询只需 name 和 id（如 SELECT name, id FROM users WHERE name = 'Bob'），可以直接从二级索引获取数据，无需回表（称为“覆盖索引”）。

#### 区别

| 特性          | 聚集索引 (Clustered Index)   | 二级索引 (Secondary Index)      |
| ------------- | ---------------------------- | ------------------------------- |
| 存储内容      | 叶子节点存储整行数据         | 叶子节点存储索引键值 + 主键指针 |
| 数量限制      | 一个表只能有一个             | 一个表可以有多个                |
| 数据顺序      | 数据按索引键物理排序         | 数据顺序与索引无关              |
| 查询效率      | 直接获取数据，效率高         | 可能需要回表，效率稍低          |
| 插入/更新成本 | 较高（需要维护数据物理顺序） | 较低（仅更新索引结构）          |

## 3、mysql的隔离模式？

事务隔离的目标是解决并发事务执行时可能出现的问题，主要包括以下三种异常：

- 脏读： 一个事务读取到另一个事务未提交的数据。如果后一个事务回滚，前一个事务读取的数据就变成了“脏数据”。
- 不可重复读：一个事务在执行期间多次读取同一行数据，但由于另一个事务修改并提交了该数据，导致前后读取结果不一致。
- 幻读：一个事务在执行期间多次查询某个范围的数据，但由于另一个事务插入或删除了数据，导致前后查询结果的行数不一致（像是“幻觉”）。

#### 四种隔离级别

MySQL InnoDB 支持以下四种隔离级别，从低到高分别是：

##### 读未提交

事务可以读取其他事务未提交的数据。

可能的问题：

- 脏读：读取到未提交的“脏数据”。
- 不可重复读和幻读也可能发生。

例子：

```sql
事务 A: UPDATE users SET age = 30 WHERE id = 1;  (未提交)
事务 B: SELECT age FROM users WHERE id = 1;     (读取到 age = 30)
事务 A: ROLLBACK;                              (回滚，age 恢复为原值)
```

事务 B 读取到了未提交的“脏数据”。

##### 读已提交

事务只能读取其他事务已提交的数据，解决了脏读问题。

可能的问题：

- 不可重复读：同一事务内多次读取同一行，数据可能被其他事务修改。
- 幻读也可能发生。

例子：

```sql
事务 A: SELECT age FROM users WHERE id = 1;     (读取 age = 25)
事务 B: UPDATE users SET age = 30 WHERE id = 1; (提交)
事务 A: SELECT age FROM users WHERE id = 1;     (读取 age = 30)
```

事务 A 两次读取结果不同，出现不可重复读。

##### 可重复读

保证事务在执行期间多次读取同一行数据时，结果一致，解决了不可重复读问题。这是 MySQL InnoDB 的默认隔离级别。

可能的问题：

- 幻读：在范围查询时，其他事务插入或删除数据可能导致“幻行”。

实现方式：

- 使用 MVCC：事务开始时生成一个一致性快照（Read View），后续读取都基于此快照。
- 结合间隙锁（Gap Lock）防止幻读（InnoDB 在此级别增强了幻读防护）。

例子：

```sql
事务 A: SELECT age FROM users WHERE id = 1;     (读取 age = 25)
事务 B: UPDATE users SET age = 30 WHERE id = 1; (提交)
事务 A: SELECT age FROM users WHERE id = 1;     (仍读取 age = 25)
```

事务 A 的读取结果保持一致，但如果涉及范围查询，可能遇到幻读：

```sql
事务 A: SELECT * FROM users WHERE age > 20;     (返回 2 行)
事务 B: INSERT INTO users (id, age) VALUES (4, 25); (提交)
事务 A: SELECT * FROM users WHERE age > 20;     (返回 3 行，幻读)
```

##### 串行化

最高隔离级别，所有事务串行执行，完全隔离，解决了脏读、不可重复读和幻读。

实现方式：通过加锁（表锁或行锁）强制事务顺序执行，禁止并发操作。

可能的问题：并发性能极低，可能导致锁冲突和死锁。

例子：

```sql
事务 A: SELECT * FROM users WHERE age > 20;     (加锁)
事务 B: INSERT INTO users (id, age) VALUES (4, 25); (被阻塞，直到 A 提交)
```

####  隔离级别对比

| 隔离级别        | 脏读 | 不可重复读 | 幻读 | 并发性能 |
| --------------- | ---- | ---------- | ---- | -------- |
| 读未提交        | 是   | 是         | 是   | 高       |
| 读已提交        | 否   | 是         | 是   | 中       |
| 可重复读 (默认) | 否   | 否         | 是*  | 中       |
| 串行化          | 否   | 否         | 否   | 低       |

## 4、undolog redolog binlog？

#### Undo Log（回滚日志）

undo log 是 InnoDB 存储引擎用来记录事务执行前的旧数据版本的日志，主要用于事务回滚和一致性读（如 MVCC，多版本并发控制）。

##### 作用

- 事务回滚：如果事务执行失败或手动回滚（如 ROLLBACK），undo log 提供旧数据，用于将数据库恢复到事务开始前的状态。
- 一致性读：在隔离级别（如读已提交、可重复读）下，通过 undo log 生成数据的历史版本快照，保证事务读取到一致的数据，而不受其他事务修改的影响。

##### 存储位置

- undo log 存储在 InnoDB 的 undo log segment 中，位于共享表空间（ibdata 文件）或独立的 undo tablespace（MySQL 5.6 及以上支持）。
- 分为两种类型：
  - Insert Undo Log：记录插入操作的旧数据，仅在回滚时使用，事务提交后可删除。
  - Update/Delete Undo Log：记录更新或删除操作的旧数据，用于 MVCC，可能长期保留。

##### 工作原理

假设表 users 中有一行数据 id=1, age=25：

- 事务 A 执行 UPDATE users SET age = 30 WHERE id = 1。
- InnoDB 将旧值 age=25 写入 undo log。
- 如果事务回滚，InnoDB 用 undo log 中的 age=25 恢复数据。
- 如果事务提交，undo log 可能被标记为可清理（但 MVCC 可能暂时保留）。

##### 特点

- 逻辑日志，记录的是操作的逆过程（如“将 age 从 30 改回 25”）。
- 与事务一致性密切相关，是 MVCC 的核心组件。

#### Redo Log（重做日志）

redo log 是 InnoDB 存储引擎用来记录事务修改后的物理数据变化的日志，主要用于崩溃恢复（Crash Recovery），确保已提交事务的持久性。

##### 作用

- 数据持久性：当事务提交时，redo log 先写入磁盘，即使内存中的数据页（Buffer Pool）尚未刷新到磁盘，崩溃后也能通过 redo log 恢复。
- 提高性能：通过先写日志、再异步刷盘的方式，减少直接写数据文件的 I/O 开销。

##### 存储位置

- 默认存储在磁盘上的 ib_logfile0 和 ib_logfile1 文件中（通常两个文件，循环使用）。
- 属于物理日志，记录的是数据页的修改（如“在某页的偏移量 X 处写入值 Y”）。

##### 工作原理

还是表 users，事务 A 执行 UPDATE users SET age = 30 WHERE id = 1：

- 修改操作先写入内存中的 Buffer Pool。
- 同时将修改记录（如“修改页面 X，偏移 Y，值从 25 变为 30”）写入 redo log buffer。
- 事务提交时，redo log buffer 刷入磁盘（ib_logfile）。
- 如果崩溃，InnoDB 重启时读取 redo log，重放已提交的修改。

##### 特点

- 顺序写磁盘，性能高（相比数据文件的随机写）。
- 日志文件大小固定，写满后循环覆盖（未刷盘的数据会先持久化）。
- 与 WAL（Write-Ahead Logging，预写日志）机制相关。

#### Binlog（二进制日志）

binlog 是 MySQL Server 层面的日志，记录所有对数据库的修改操作（如 INSERT、UPDATE、DELETE），主要用于主从复制和数据恢复。

##### 作用

- 主从复制：主库将 binlog 传输到从库，从库重放这些操作以保持数据一致。
- 数据恢复：结合全量备份和 binlog，可以实现时间点恢复（Point-in-Time Recovery, PITR）。

##### 存储位置

- 存储在磁盘上的二进制文件（如 mysql-bin.000001），文件名和位置可配置。
- 通过参数 log_bin 启用，默认关闭。

##### 工作原理

事务 A 执行 UPDATE users SET age = 30 WHERE id = 1：

- 操作完成后，Server 层将语句或数据变化写入 binlog（格式取决于配置）。
- 主库提交事务时，binlog 同步写入磁盘。
- 从库读取 binlog，执行相同的修改。

##### 格式

- Statement：记录 SQL 语句（如 UPDATE users SET age = 30 WHERE id = 1），简单但可能不精确。
- Row：记录具体的数据行变化（如“将 id=1 的 age 从 25 改为 30”），精确但占用空间大。
- Mixed：混合模式，根据情况选择 Statement 或 Row。

##### 特点

- 逻辑日志，记录操作语句或数据变化。
- 与存储引擎无关，MyISAM 也支持。
- 可用于审计和跨库同步。

#### 三者的区别

| 特性     | Undo Log                    | Redo Log                 | Binlog                   |
| -------- | --------------------------- | ------------------------ | ------------------------ |
| 所属层   | InnoDB 存储引擎             | InnoDB 存储引擎          | MySQL Server 层          |
| 作用     | 回滚、一致性读              | 崩溃恢复、持久性         | 主从复制、数据恢复       |
| 日志类型 | 逻辑日志（旧数据）          | 物理日志（数据页变化）   | 逻辑日志（语句或行变化） |
| 存储位置 | 表空间（ibdata 或单独文件） | ib_logfile 文件          | mysql-bin 文件           |
| 生命周期 | 事务提交后可清理            | 循环覆盖（刷盘后可重用） | 可配置保留时间或手动清理 |
| 是否必须 | 是（支持事务和 MVCC）       | 是（支持崩溃恢复）       | 否（可选，用于复制）     |

#### 联系

- 事务提交流程：

  1. 修改数据时，生成 undo log（支持回滚）。

  2. 将修改写入 redo log buffer（保证持久性）。

  3. 事务提交时，redo log 刷盘，同时记录 binlog（若启用）。

  4. binlog 刷盘后，事务完成。

- 一致性：redo log 确保物理一致性，undo log 确保逻辑一致性，binlog 用于外部同步。

- 崩溃恢复：redo log 重放提交事务，undo log 回滚未提交事务。

## 5、binlog的3种形式？

#### Statement（基于语句的日志格式，STATEMENT-BASED REPLICATION, SBR）

Statement 格式记录的是执行的原始 SQL 语句，而不是具体的行数据变化。

##### 工作原理

- 每次写操作（如 INSERT、UPDATE、DELETE）发生时，MySQL 将完整的 SQL 语句写入 binlog。
- 从库读取 binlog 并重新执行这些 SQL 语句，以实现数据同步。

例如，假设执行以下操作：

```sql
UPDATE users SET age = age + 1 WHERE id = 1;
```

binlog 记录：

```sql
UPDATE users SET age = age + 1 WHERE id = 1;
```

从库直接执行这条语句，更新对应的行。

#### Row（基于行的日志格式，ROW-BASED REPLICATION, RBR）

Row 格式记录的是具体的数据行变化，而不是 SQL 语句本身。它记录每行数据的“前镜像”（before image）和“后镜像”（after image）。

##### 工作原理

- 对于每次写操作，MySQL 将受影响的每一行数据的具体变化写入 binlog。
- 从库直接应用这些行级变化，无需重新执行 SQL。

例如，还是执行：

```sql
UPDATE users SET age = age + 1 WHERE id = 1;
```

假设原始数据是 id=1, age=25：

binlog 记录：

```sql
Before: id=1, age=25
After:  id=1, age=26
```

从库直接将 id=1 的 age 从 25 更新为 26。

#### Mixed（混合日志格式，MIXED-MODE REPLICATION）

Mixed 格式是 Statement 和 Row 的混合模式，MySQL 根据操作类型动态选择记录方式。通常优先使用 Statement，但在特定情况下切换为 Row。

##### 工作原理

- 默认记录 SQL 语句（Statement）。
- 遇到可能导致主从不一致的操作时，切换为记录行变化（Row）。
- 切换的典型场景包括：
  - 使用非确定性函数（如 NOW()、UUID()）。
  - 使用 INSERT ... ON DUPLICATE KEY UPDATE 或 LOAD DATA INFILE。
  - 执行涉及存储过程、触发器或复杂语句的操作。

示例：

- 执行 INSERT INTO logs (time) VALUES (NOW());：

  binlog 切换为 Row 格式，记录具体行数据（如 time=2025-03-05 12:00:00）。

- 执行 UPDATE users SET age = 30 WHERE id = 1;：

  binlog 使用 Statement 格式，记录语句本身。

#### 三种格式对比表

| 特性     | Statement          | Row                | Mixed                |
| -------- | ------------------ | ------------------ | -------------------- |
| 记录内容 | SQL 语句           | 具体行变化         | 动态切换（语句或行） |
| 日志大小 | 小                 | 大                 | 中等                 |
| 一致性   | 较低（可能不一致） | 高（完全一致）     | 高（智能优化）       |
| 可读性   | 高（SQL 可读）     | 低（二进制）       | 中等（混合）         |
| 从库性能 | 较低（需执行 SQL） | 高（直接应用变化） | 中等（视情况而定）   |
| 适用场景 | 简单操作、审计     | 高一致性、复杂操作 | 通用场景             |

## 6、乐观锁悲观锁？

#### 基本定义

##### 悲观锁

- 假设：假定并发冲突很常见，认为多个事务同时访问数据时很可能会发生冲突。
- 策略：在操作数据前，先对数据加锁，确保其他事务无法同时访问或修改，直到当前事务完成并释放锁。

##### 乐观锁

- 假设：假定并发冲突很少发生，认为大多数情况下多个事务不会同时修改同一数据。
- 策略：不提前加锁，操作时直接读取数据，提交时检查数据是否被其他事务修改过，若冲突则回滚或重试。

#### 工作原理

##### 悲观锁

实现方式：

- 通过数据库的锁机制（如行锁、表锁）实现。
- 常见 SQL 关键字：SELECT ... FOR UPDATE（在 MySQL 中）。

流程：

1. 事务开始时，锁定目标数据（例如，锁定某行）。
2. 其他事务试图访问被锁定的数据时会被阻塞，直到锁释放。
3. 当前事务完成（提交或回滚），释放锁。

示例：

```sql
BEGIN;
SELECT * FROM users WHERE id = 1 FOR UPDATE; -- 加行锁
UPDATE users SET age = 30 WHERE id = 1;
COMMIT;
```

事务执行期间，其他事务无法修改 id=1 的行。

##### 乐观锁

实现方式：

- 通常通过版本号（version）或时间戳（timestamp）字段实现。
- 不依赖数据库锁，而是通过应用程序逻辑控制。

流程：

1. 读取数据时，记录当前的版本号。
2. 更新数据时，检查版本号是否一致。
3. 如果一致，则更新成功并递增版本号；否则，说明数据已被修改，事务失败。

示例：

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    age INT,
    version INT
);
```

操作：

```sql
-- 事务 A
SELECT age, version FROM users WHERE id = 1; -- 读取 age=25, version=1
-- 事务 B 修改了数据
UPDATE users SET age = 26, version = 2 WHERE id = 1 AND version = 1;
-- 事务 A 尝试更新
UPDATE users SET age = 30, version = 2 WHERE id = 1 AND version = 1; -- 失败，因 version 已变为 2
```

#### 主要区别

| 特性     | 悲观锁                    | 乐观锁                     |
| -------- | ------------------------- | -------------------------- |
| 并发假设 | 冲突频繁                  | 冲突罕见                   |
| 锁机制   | 提前加锁                  | 不加锁，提交时验证         |
| 实现方式 | 数据库锁（FOR UPDATE 等） | 版本号、时间戳等逻辑控制   |
| 性能开销 | 高（锁阻塞降低并发性）    | 低（无锁，但冲突时需重试） |
| 适用场景 | 高冲突、高一致性需求      | 低冲突、高并发读写         |
| 复杂度   | 简单（依赖数据库）        | 稍复杂（需应用层逻辑）     |

## 7、mysql怎么实现悲观锁？

#### 使用 SELECT ... FOR UPDATE（行级排他锁）

对查询的行加排他锁（X Lock），阻止其他事务读写这些行，直到当前事务结束。

##### 语法

```sql
SELECT column_list FROM table_name WHERE condition FOR UPDATE;
```

##### 示例

假设表 users：

```sql
id | name | age
1  | Alice| 25
2  | Bob  | 30
```

事务 A：

```sql
BEGIN;
SELECT * FROM users WHERE id = 1 FOR UPDATE; -- 加排他锁
UPDATE users SET age = 26 WHERE id = 1;
COMMIT;
```

事务 A 执行 FOR UPDATE 时，id=1 的行被锁定。

事务 B 若尝试：

```sql
SELECT * FROM users WHERE id = 1 FOR UPDATE;
```

会被阻塞，直到事务 A 提交或回滚。

#### 使用 SELECT ... LOCK IN SHARE MODE（行级共享锁）

对查询的行加共享锁（S Lock），允许多个事务读取，但阻止写入，直到当前事务结束。

##### 语法

```sql
SELECT column_list FROM table_name WHERE condition LOCK IN SHARE MODE;
```

##### 示例

事务 A：

```sql
BEGIN;
SELECT * FROM users WHERE id = 1 LOCK IN SHARE MODE; -- 加共享锁
-- 读取 age=25
COMMIT;
```

事务 B 可以同时执行：

```sql
SELECT * FROM users WHERE id = 1 LOCK IN SHARE MODE; -- 允许读
```

但事务 B 若尝试：

```sql
UPDATE users SET age = 30 WHERE id = 1; -- 被阻塞
```

会被阻塞，直到事务 A 结束。

#### 使用 LOCK TABLES（表级锁）

对整个表加锁，分为读锁（READ）和写锁（WRITE）。

##### 语法

```sql
LOCK TABLES table_name [READ | WRITE];
UNLOCK TABLES;
```

##### 示例

读锁：

```sql
LOCK TABLES users READ;
SELECT * FROM users;
UNLOCK TABLES;
```

其他事务可以读，但不能写。

写锁：

```sql
LOCK TABLES users WRITE;
UPDATE users SET age = age + 1;
UNLOCK TABLES;
```

其他事务无法读写。

## 8、SELECT...FOR UPDATE和 SELECT...LOCK IN SHARE MODE语句？

上面已讲过。

## 9、场景题：redis实现滑动窗口？

这里大家可以借助AI工具

## 10、 redis的持久化？

#### RDB（快照持久化）

RDB 是 Redis 的默认持久化方式，通过定期将内存中的数据集以二进制快照的形式保存到磁盘上，生成一个压缩的 .rdb 文件。

##### 工作原理

触发机制：

- 手动触发：

  - SAVE：同步保存，阻塞 Redis 主进程。
  - BGSAVE：异步保存，fork 一个子进程执行快照操作，主进程继续处理请求。

- 自动触发：

  通过配置文件中的 save 指令设置，例如：

  ```sql
  save 900 1    # 900 秒内至少 1 次键变更
  save 300 10   # 300 秒内至少 10 次键变更
  save 60 10000 # 60 秒内至少 10000 次键变更
  ```

  Redis 检测到满足条件时，自动调用 BGSAVE。

保存过程：

1. Redis 调用 fork() 创建子进程。
2. 子进程将内存数据写入临时文件（.rdb.tmp）。
3. 保存完成后，临时文件替换旧的 .rdb 文件。

文件位置：由配置文件中的 dir 和 dbfilename 参数指定，默认是 ./redis.rdb。

##### 数据恢复

重启 Redis 时，若检测到 .rdb 文件，自动加载到内存中，恢复数据。

#### AOF（追加式持久化）

AOF 通过将每次写操作（修改数据的命令）追加到日志文件中保存，记录 Redis 的操作历史，从而实现持久化。重启时通过重放这些命令恢复数据。

##### 工作原理

记录方式：

- 每次执行写命令（如 SET、DEL、INCR）时，命令以 Redis 协议格式追加到 AOF 文件。

- 示例：执行 SET key value，AOF 文件记录：

  ```
  *3\r\n$3\r\nSET\r\n$3\r\nkey\r\n$5\r\nvalue\r\n
  ```

同步策略（由 appendfsync 参数控制）：

- always：每次写命令后立即同步到磁盘，数据最安全，但性能最低。
- everysec：每秒同步一次，默认选项，平衡性能和安全性（最多丢失 1 秒数据）。
- no：由操作系统决定同步时机，性能最高，但数据丢失风险大。

文件位置：由 dir 和 appendfilename 参数指定，默认是 ./appendonly.aof。

##### 重写机制（AOF Rewrite）

问题：AOF 文件会随写操作不断增长，可能变得很大。

解决：Redis 提供 AOF 重写功能，优化冗余命令，生成更紧凑的文件。

- 手动触发：BGREWRITEAOF。

- 自动触发：由参数控制：

  ```sql
  auto-aof-rewrite-percentage 100  # 文件增长 100% 时重写
  auto-aof-rewrite-min-size 64mb  # 文件至少 64MB 时才重写
  ```

重写过程：

1. fork 子进程。
2. 子进程根据当前内存数据生成新的 AOF 文件（只记录最终状态，例如将多次 INCR 合并为一个 SET）。
3. 主进程继续追加新命令到旧 AOF 文件和缓冲区。
4. 重写完成后，新文件替换旧文件，缓冲区内容追加到新文件。

##### 数据恢复

重启 Redis 时，若启用 AOF（appendonly yes），逐条执行 AOF 文件中的命令，重建数据集。

## 11、开启持久化模式会不会导致redis变慢？

会。

#### RDB

RDB 通过定期生成内存快照（.rdb 文件）实现持久化，通常由 BGSAVE 命令触发，fork 一个子进程完成保存。

影响：

- fork 和快照保存可能导致短暂延迟（毫秒级），高内存高频场景更明显。
- 通常较小，除非频繁触发或内存超大。

#### AOF

AOF 将每次写命令追加到日志文件（.aof），并根据 appendfsync 参数决定同步策略：

- always：每次写后同步。
- everysec：每秒同步（默认）。
- no：操作系统决定。

影响：

- always：显著变慢，写性能下降严重。
- everysec：轻微变慢，影响可接受。
- no：几乎不影响。

## 12、redis为什么快？

Redis（Remote Dictionary Server）以其极高的性能著称，常被用作缓存、消息队列和实时数据存储。它的“快”并非单一因素的结果，而是多个设计和实现上的优势共同作用的成果。

从下面这几个方面聊聊：

#### 内存存储（In-Memory Storage）

##### 原理

- Redis 是一个内存数据库，数据主要存储在 RAM 中，而不是磁盘上。
- 内存访问速度比磁盘快几个数量级：
  - RAM 访问延迟：纳秒级（~10-100 ns）。
  - 磁盘访问延迟：毫秒级（机械硬盘 ~10ms，SSD ~0.1ms）。

##### 影响

- 所有读写操作都在内存中完成，避免了磁盘 I/O 的瓶颈。
- 即使开启持久化（RDB 或 AOF），核心操作仍基于内存，磁盘写入是异步或后台处理的。

#### 单线程事件循环（Single-Threaded Event Loop）

##### 原理

- Redis 使用单线程处理客户端请求，避免了多线程带来的上下文切换和锁竞争开销。
- 采用事件驱动模型（基于 I/O 多路复用，如 epoll/select），通过非阻塞 I/O 处理多个客户端连接。

##### 工作方式

- 事件循环：
  1. 监听客户端连接和命令（网络事件）。
  2. 将命令放入队列。
  3. 单线程按顺序处理队列中的命令。
- 非阻塞：网络 I/O 和命令执行分离，单个线程高效利用 CPU。

##### 影响

- 无锁竞争：无需线程同步（如互斥锁），消除了多线程模型的性能损耗。
- 上下文切换少：单线程避免了线程切换的开销（每次切换 ~微秒级）。
- 简单高效：逻辑清晰，指令执行顺序可预测。

#### 高效的数据结构

##### 原理

- Redis 提供了丰富的数据结构（如 String、Hash、List、Set、Sorted Set），并针对每种结构进行了底层优化。
- 这些数据结构使用高效的内存编码和算法，减少计算和内存开销。

##### 具体实现

- SDS（Simple Dynamic String）：
  - 替代 C 原生字符串，预分配空间，记录长度，避免频繁 realloc 和 strlen。
  - 读写复杂度：O(1)。
- Hash 表：使用渐进式 rehash，均摊扩容成本，查询/插入复杂度接近 O(1)。
- Skip List（跳跃表）：用于 Sorted Set，查找/插入复杂度 O(log n)，比平衡树更轻量。
- 压缩列表（ziplist）：小型 List/Set/Hash 使用紧凑内存布局，节省空间并加速遍历。

##### 影响

- 数据操作复杂度低，执行效率高。
- 内存使用率高，减少浪费。

#### I/O 多路复用（Multiplexing）

##### 原理

- Redis 使用 epoll（Linux）、kqueue（BSD）或 select 等机制处理大量并发连接。
- 单线程通过事件循环监听多个 socket，异步处理网络请求。

##### 工作方式

- 一个线程管理成千上万客户端连接。
- 当 socket 可读/可写时，触发事件，Redis 处理对应请求。

##### 影响

- 高并发：支持数十万客户端同时连接，而无需为每个连接分配线程。
- 低开销：网络层效率高，减少系统资源占用。

#### 为什么比其他数据库快？

- 对比传统磁盘数据库（如 MySQL）：
  - MySQL 依赖磁盘 I/O 和复杂事务，单次查询延迟 1-10ms。
  - Redis 全内存操作，延迟 < 1ms。
- 对比其他内存数据库（如 Memcached）：
  - Memcached 单线程、无复杂数据结构，仅支持 KV。
  - Redis 提供丰富数据结构和持久化，性能仍接近 Memcached。

## 13、对网络安全相关了解吗？知道什么攻击？

#### 恶意软件攻击 (Malware Attack)

恶意软件（Malware）是旨在破坏、窃取或控制系统的恶意代码的总称，包括病毒（Virus）、蠕虫（Worm）、木马（Trojan）、勒索软件（Ransomware）等。

原理：

- 病毒：依附于合法程序，需用户交互（如打开附件）传播。
- 蠕虫：自我复制，利用网络漏洞自动传播，无需用户干预。
- 木马：伪装成合法软件，诱导用户安装后窃取数据或开启后门。
- 勒索软件：加密用户数据，要求支付赎金以解密。

影响：数据丢失、系统瘫痪、财务损失。

#### 钓鱼攻击 (Phishing Attack)

一种社会工程攻击，通过伪装成可信实体（如银行、公司）诱骗用户提供敏感信息（如密码、信用卡号）。

原理：

- 发送伪造的电子邮件、短信或网站链接，诱导用户点击或输入信息。
- 常结合鱼叉式钓鱼（Spear Phishing），针对特定目标定制内容。

影响：身份盗窃、账户被盗。

#### 拒绝服务攻击 (Denial-of-Service, DoS) 和 分布式拒绝服务攻击 (DDoS)

- 定义：
  - DoS：通过大量请求淹没目标系统，使其无法响应合法用户。
  - DDoS：利用多个受控设备（如僵尸网络）发起分布式攻击。
- 原理：
  - 消耗目标的带宽、CPU 或内存资源。
  - 常见手段包括 SYN 洪水、UDP 洪水、HTTP 请求轰炸。
- 影响：服务中断、网站宕机。

#### 中间人攻击 (Man-in-the-Middle, MitM)

攻击者在通信双方之间拦截并可能篡改数据。

原理：

- 通过 ARP 欺骗、DNS 劫持或伪造 Wi-Fi 热点窃听流量。
- 可窃取未加密数据或注入恶意内容。

影响：敏感数据泄露、会话劫持。

#### SQL 注入攻击 (SQL Injection)

攻击者通过向数据库输入恶意 SQL 代码，获取未授权数据或操控数据库。

原理：

- 利用未正确验证的用户输入（如表单字段），执行非预期查询。
- 示例输入：' OR '1'='1 可绕过登录验证。

影响：数据库泄露、数据篡改。

#### 密码攻击 (Password Attack)

尝试破解用户密码以获取访问权限。

- 原理：
  - 暴力破解 (Brute Force)：尝试所有可能组合。
  - 字典攻击 (Dictionary Attack)：使用常见密码列表。
  - 凭据填充 (Credential Stuffing)：利用泄露的用户名密码对尝试登录。
- 影响：账户被盗、权限提升。

## 14、ping的时候用的什么协议？

在计算机网络中，ping 是一种常用的网络诊断工具，用于测试两台主机之间的连通性和往返时间（RTT，Round-Trip Time）。它主要依赖 ICMP（Internet Control Message Protocol，互联网控制消息协议） 来实现。

#### ICMP

- ICMP 是 TCP/IP 协议族中的一种网络层协议，设计用于在 IP 网络中传输控制消息，而不是用户数据。
- 它通常与 IP 协议（Internet Protocol）配合使用，IP 负责数据包的传输，而 ICMP 负责网络诊断和错误报告。

#### ping 的工作原理

ping 使用 ICMP 的 Echo Request（回显请求） 和 Echo Reply（回显应答） 消息来测试网络连通性。

发送 Echo Request：

- 本地主机（发起 ping 的设备）构造一个 ICMP Echo Request 消息。
- 该消息包含一个唯一的标识符（Identifier）和序列号（Sequence Number），用于匹配请求和应答。
- 数据包封装在 IP 数据包中，发送到目标主机。

目标主机响应：

- 如果目标主机在线且允许 ICMP 响应，它会收到 Echo Request。
- 目标主机构造一个 ICMP Echo Reply 消息，将原始请求的数据原样返回。
- 应答数据包通过 IP 路由回传到源主机。

计算 RTT：

- 源主机收到 Echo Reply 后，根据发送时间和接收时间计算往返时间（RTT）。

- 输出结果，如：

  ```
  Reply from www.baidu.com: bytes=32 time=15ms TTL=117
  ```

## 15、你在浏览器中输入https://www.某网址.com并访问时，经历了什么过程？

#### URL 解析

##### 过程

- 输入 https://www.xxx.com。
- 浏览器解析 URL，分解为以下部分：
  - 协议：https（超文本传输安全协议）。
  - 主机名：www.xxx.com（域名）。
  - 路径：默认 /（未指定具体路径时）。
  - 端口：隐式为 443（HTTPS 默认端口）。

##### 细节

- 浏览器会检查输入是否合法（如语法错误）。
- 如果省略协议（如只输入 www.xxx.com），现代浏览器通常默认补全为 https://（优先于 http://）。
- 如果输入的是搜索关键词而非 URL，浏览器会将其交给默认搜索引擎处理。

#### DNS 域名解析

##### 过程

- 浏览器需要将域名 www.xxx.com 转换为服务器的 IP 地址，因为网络通信基于 IP。
- DNS 解析过程如下：
  1. 检查浏览器缓存：浏览器先查看自身 DNS 缓存是否有 www.xxx.com 的 IP。
  2. 检查操作系统缓存：若无，查询本机 hosts 文件和系统 DNS 缓存（如 Windows 的 ipconfig /displaydns）。
  3. 本地 DNS 服务器：若仍无，向本地 DNS 服务器（通常由 ISP 提供，或如 8.8.8.8 的公共 DNS）发起请求。
  4. 递归解析：
     - 本地 DNS 服务器向根域名服务器（.）查询。
     - 根服务器返回 .com 的顶级域名服务器 (TLD) 地址。
     - TLD 服务器返回 xxx.com 的权威域名服务器地址。
     - 权威服务器返回 www.xxx.com 的 IP 地址（如 93.184.216.34）。
  5. 返回结果：本地 DNS 服务器将 IP 返回给操作系统和浏览器，并缓存结果。

##### 细节

- 记录类型：通常查询 A 记录（IPv4 地址）或 AAAA 记录（IPv6 地址）。
- 时间：DNS 解析通常耗时几毫秒到几十毫秒，取决于缓存命中率和网络延迟。
- CDN：若网站使用内容分发网络（如 Cloudflare），DNS 可能返回就近节点的 IP。

#### TCP 连接建立

##### 过程

- 浏览器获取 IP 地址后，发起与目标服务器（93.184.216.34:443）的 TCP 连接。
- TCP 三次握手：
  1. SYN：客户端发送 SYN 包（同步序列号），请求连接。
  2. SYN-ACK：服务器回复 SYN-ACK 包（确认客户端请求并发送自己的序列号）。
  3. ACK：客户端发送 ACK 包，确认连接建立。

##### 细节

- 端口：HTTPS 使用 443 端口（HTTP 默认 80）。
- 可靠性：TCP 确保数据有序、无损传输。
- 时间：三次握手通常耗时一个往返时间（RTT），如 10-50ms。

#### TLS 安全握手

##### 过程

- 因为使用 HTTPS，浏览器与服务器在 TCP 连接之上建立 TLS（传输层安全协议）加密通道。

- TLS 握手步骤：

  1. Client Hello：客户端发送支持的 TLS 版本（如 TLS 1.3）、加密算法（如 AES）、随机数等。

  2. Server Hello：服务器选择 TLS 版本和加密算法，返回随机数和证书（含公钥）。

  3. 证书验证：

     浏览器验证服务器证书：

     - 检查证书是否由受信任的 CA（证书颁发机构，如 Let’s Encrypt）签发。
     - 验证域名匹配（www.xxx.com）。
     - 检查证书有效期。

  4. 密钥交换：

     - 客户端生成会话密钥（Session Key），用服务器公钥加密后发送。
     - 服务器用私钥解密，双方协商出对称密钥。

  5. 握手完成：双方交换 “Finished” 消息，确认加密通道建立。

#### HTTP 请求发送

##### 过程

- 浏览器通过加密的 TLS 通道发送 HTTP 请求：

  ```
  GET / HTTP/1.1
  Host: www.xxx.com
  User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) ...
  Accept: text/html,application/xhtml+xml,...
  Connection: keep-alive
  ```

- 请求包括：

  - 方法：GET（获取资源）。
  - 路径：/（默认主页）。
  - 头信息：如 Host、User-Agent 等。

##### 细节

- HTTP/1.1：支持 keep-alive，复用 TCP 连接。
- HTTP/2：若服务器支持，可并行发送多个请求，效率更高。

#### 服务器响应

##### 过程

服务器接收请求后：

1. 解密：用 TLS 解密请求。

2. 路由：根据 Host 和路径（如 /）定位资源。

3. 处理：运行后端逻辑（如查询数据库、生成动态页面）。

4. 响应：构造 HTTP 响应：

   ```
   HTTP/1.1 200 OK
   Content-Type: text/html
   Content-Length: 1234
   ...
   <html><body>Hello, World!</body></html>
   ```

##### 细节

- 状态码：
  - 200 OK：成功。
  - 404 Not Found：资源不存在。
  - 301 Moved Permanently：重定向。
- 时间：取决于服务器性能和业务逻辑复杂性（几毫秒到几秒）。

#### 页面渲染

##### 过程

浏览器将 HTML 渲染为可视页面：

1. 构建 DOM 树：解析 HTML，生成文档对象模型。
2. 加载 CSS：下载并解析 <link> 中的 CSS，构建 CSSOM（CSS 对象模型）。
3. 渲染树：结合 DOM 和 CSSOM，确定页面结构和样式。
4. 布局（Layout）：计算元素位置和大小。
5. 绘制（Painting）：将像素渲染到屏幕。
6. 执行 JavaScript：加载并运行 <script> 中的代码，可能修改 DOM。

##### 细节

- 并行加载：浏览器同时下载 CSS、JS、图片等资源。
- 阻塞：<script> 默认阻塞渲染，可用 async 或 defer 优化。
- 时间：渲染时间从几十毫秒到几秒，视页面复杂性。

## 16、让用户访问旧网址时自动跳转到新网址该怎么做？

实现方法主要涉及 HTTP 重定向，可以从客户端、服务器端或 DNS 层面实现。

#### HTTP 重定向的基本原理

HTTP 重定向通过服务器返回特定的状态码（3xx 系列）通知客户端跳转到新地址。常见的重定向状态码包括：

- 301 Moved Permanently：永久重定向，告诉浏览器和搜索引擎新地址是永久替代旧地址。
- 302 Found：临时重定向，适合短期跳转，不影响搜索引擎索引。

##### 工作流程

1. 用户访问旧网址。
2. 服务器返回重定向响应（如 301 或 302），包含新网址（在 Location 头中）。
3. 浏览器接收响应，自动跳转到新网址。
4. 用户看到新网址的内容。

## 17、手撕归并排序？

归并排序通过将数组递归地分成小块，分别排序后再合并，最终得到有序数组。

#### 归并排序的原理

归并排序的核心是“分”和“治”：

- 分（Divide）：将数组递归地分成两半，直到每个子数组只剩一个元素（天然有序）。
- 治（Conquer）：将两个有序子数组合并成一个更大的有序数组。

##### 工作流程

1. 如果数组长度大于 1，将其分成两部分（通常取中间点）。
2. 递归地对左右两部分进行归并排序。
3. 将排序好的左右子数组合并为一个有序数组。

##### 特点

- 稳定性：归并排序是稳定的排序算法（相同元素的相对顺序不变）。
- 空间需求：需要额外的辅助空间来合并子数组。

#### 实现步骤

##### (1) 划分函数（mergeSort）

- 输入：待排序数组、起始索引、结束索引。
- 逻辑：
  - 如果起始索引 < 结束索引（子数组长度 > 1），计算中间点。
  - 递归调用自身，分别排序左半部分和右半部分。
  - 调用合并函数处理排序后的子数组。

##### (2) 合并函数（merge）

- 输入：数组、左子数组范围（left 到 mid）、右子数组范围（mid+1 到 right）。
- 逻辑：
  - 创建临时数组存储合并结果。
  - 比较左右子数组的元素，按顺序放入临时数组。
  - 将临时数组拷贝回原数组。

#### 参考代码（C++）

```c++
#include <iostream>
#include <vector>

// 合并两个有序子数组
void merge(std::vector<int>& arr, int left, int mid, int right) {
    // 计算左右子数组的大小
    int n1 = mid - left + 1;  // 左子数组长度
    int n2 = right - mid;     // 右子数组长度

    // 创建临时数组
    std::vector<int> leftArr(n1);
    std::vector<int> rightArr(n2);

    // 将数据拷贝到临时数组
    for (int i = 0; i < n1; i++) {
        leftArr[i] = arr[left + i];
    }
    for (int j = 0; j < n2; j++) {
        rightArr[j] = arr[mid + 1 + j];
    }

    // 合并两个子数组
    int i = 0;    // 左子数组索引
    int j = 0;    // 右子数组索引
    int k = left; // 原数组合并位置

    while (i < n1 && j < n2) {
        if (leftArr[i] <= rightArr[j]) { // 保持稳定性，使用 <=
            arr[k] = leftArr[i];
            i++;
        } else {
            arr[k] = rightArr[j];
            j++;
        }
        k++;
    }

    // 拷贝剩余元素（如果有）
    while (i < n1) {
        arr[k] = leftArr[i];
        i++;
        k++;
    }
    while (j < n2) {
        arr[k] = rightArr[j];
        j++;
        k++;
    }
}

// 递归划分数组
void mergeSort(std::vector<int>& arr, int left, int right) {
    if (left < right) { // 子数组长度大于 1
        int mid = left + (right - left) / 2; // 防止溢出
        mergeSort(arr, left, mid);          // 排序左半部分
        mergeSort(arr, mid + 1, right);     // 排序右半部分
        merge(arr, left, mid, right);       // 合并结果
    }
}

// 打印数组
void printArray(const std::vector<int>& arr) {
    for (int num : arr) {
        std::cout << num << " ";
    }
    std::cout << std::endl;
}

int main() {
    std::vector<int> arr = {38, 27, 43, 3, 9, 82, 10};
    std::cout << "Original array: ";
    printArray(arr);

    mergeSort(arr, 0, arr.size() - 1);

    std::cout << "Sorted array: ";
    printArray(arr);

    return 0;
}
```

