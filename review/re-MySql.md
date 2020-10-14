### 基础

#### 数据库的三范式是什么？什么是反模式？

1. 原子性
2. 完全依赖主键
3. 直接依赖于主键

表多，join，性能

反设计模式：宽表

#### Mysql数据类型

#### varchar 则是一种可变长度的类型，（50）排序占内存少

#### int(11) 中的 11影响展示效果

#### 钱用decimal 

#### 自增主键，InnoDB类型重启丢失，MyISAM 类型记录不丢失

#### COUNT(*) ，InnoDB类型全表扫描，MyISAM 类型记录

#### InnoDB 的 [4 大特性](https://note.youdao.com/ynoteshare1/index.html?id=a6004953a0a7c80073ac74d8e76f1ebd&type=note)

- [插入缓冲](https://www.cnblogs.com/zhs0/p/10528520.html)(insert buffer/ change buffer) 
  - **非聚簇索引**
  - 插入缓存，合并
  - 减少随机IO，提升性能
- 二次写(double write)
  - 写入磁盘之前，缓存，崩溃恢复
- 自适应哈希索引(ahi)
  - 监控热数据，自适应变成hash索引
- 预读(read ahead)：提升IO性能
  - 线性预读：阀值，默认56，下一个extent放入**buffer pool**
  - 随机预读：当前extent中剩余的page，5.5废弃

#### MySQL 存储引擎：

- `InnoDB`：不支持事务、也不支持外键，优势是访问速度快
- `MyISAM`：事物，行级别锁，崩溃恢复。
- MRG_MYISAM：Merge存储引擎是**一组MyISAM表的组合**
- MEMORY：使用存在于**内存**中的内容来创建表。

#### MyISAM和InnoDB

- 事物
- 锁
- 崩溃恢复
- MVCC应对高并发





#### 字符集指的是⼀种从**⼆进制编码**到某类**字符符号**的**映射**。校对规则则是指某种**字符集下**的**排序规则**。



![image-20200927201915956](https://gitee.com//chenchong0817/picture/raw/master/Aaron/20200927201926.png)



### 索引

#### 好处

- **缩小查询数目**，加快检索速度，**降低IO成本**
- 数据**预先排好序**，排序成本、CPU消耗

#### 坏处

- **占用存储空间**
- **降低更新表的速度**

#### 场景

- **中大型表**
- 特大型的表，建立和使用索引的代价随着增长，建议**分库分表**

#### 索引类型

- 1、普通索引
- 2、唯一索引
- 3、主键索引
- 4、复合索引

#### **创建**原则

1.  `WHERE` 子句中的**列**，**或连接子句中的列**
2. **基数越大**，索引效果越好
3. 创建**复合索引，基数更大**
4. 避免**创建过多**的索引，**占空间，降低写效率**
5. **主键**尽可能**选择较短**的数据类型，可以有效**减少索引的磁盘**占用提高查询效率。
6. 对字符串进行索引，应该定制一个前缀长度，可以节省大量的索引空间。

#### **使用**注意事项

- 应尽量避免在 `WHERE` 子句中**对字段进行判断、表达式、函数操作**，优化器将**无法通过索引来确定将要命中的行数**，导致引擎放弃使用索引而进行全表扫描。
  - 判断： `!=` 或 `<>` 、`column IS NULL` 、 `OR`
- 不要在 `WHERE` 子句中的 `=` **左边**进行函数、算术运算或其他表达式运算，否则系统将可能无法正确使用索引。
- 复合索引**遵循前缀原则**。
- 如果 MySQL 评估使用索引比全表扫描更慢，会放弃使用索引。如果此时想要索引，可以在语句中**添加强制索引**。
- 列类型是**字符串类型**，查询时一定要给值**加引号**，否则索引失效。
- `LIKE` 查询，`%` 不能在前，因为无法使用索引。如果需要模糊匹配，可以使用全文索引。

####  以下三条 SQL 如何建索引，只建一条怎么建？

```
WHERE a = 1 AND b = 1
WHERE b = 1
WHERE b = 1 ORDER BY time DESC
```

- 以顺序 b , a, time 建立复合索引，`CREATE INDEX table1_b_a_time ON index_test01(b, a, time)`。
- 对于第一条 **SQL** ，因为最新 MySQL 版本会**优化** `WHERE` 子句后面的列顺序，以匹配复合索引顺序。

#### （重点）EXPLAIN

详细看看 [《MySQL explain 执行计划详细解释》](http://www.jfox.info/2017/mysql-explain执行计划详细解释.html) 。

![image-20201014205144535](https://gitee.com/chenchong0817/picture/raw/master/Aaron/image-20201014205144535.png)

具体解释如下

1. 执行id
   包含一组数字，id如果相同，可以认为是一组，从上往下顺序执行；在所有组中，id值越大，优先级越高，越先执行
2. **查询类型select_type**
   type显示的是访问类型，是较为重要的一个指标，结果值从好到坏依次是： 
   system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL ，一般来说，得**保证查询至少达到range级别，最好能达到ref**。
   - **ALL**: 扫描全表
   - **index**: 扫描全部索引树
   - **range**: 扫描部分索引，索引范围扫描，对索引的扫描开始于某一点，返回匹配值域的行，常见于between、<、>等的查询
   - ref: **非唯一性索引扫描**，返回匹配某个单独值的所有行。常见于使用非唯一索引即唯一索引的非唯一前缀进行的查找
   - eq_ref：**唯一性索引扫描**，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描
3. possible_keys
   指出MySQL能使用哪个索引在表中找到行，查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询使用
4. **key**：**显示MySQL在查询中实际使用的索引**，若没有使用索引，显示为NULL
   TIPS：查询中若使用了覆盖索引，则该索引仅出现在key列表中
5. key_len
   表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度
6. ref
   表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值
7. **rows**：
   表示MySQL根据表统计信息及索引选用情况，估算的找到所需的记录所需要读取的行
8. Extra
   包含不适合在其他列中显示但十分重要的额外信息

二、关于MySQL执行计划的局限总结如下：
1.EXPLAIN不会告诉你关于触发器、存储过程的信息或用户自定义函数对查询的影响情况
2.EXPLAIN**不考虑各种Cache**
3.EXPLAIN**不能显示MySQL在执行查询时所作的优化工作**
4.**部分统计信息是估算的，并非精确值**
5.EXPALIN只能解释SELECT操作，其他操作要重写为SELECT后查看执行计划 

#### 索引原理

[《MySQL索引背后的数据结构及算法原理》](http://blog.codinglabs.org/articles/theory-of-mysql-index.html)

在 MySQL 中，我们可以看到两种索引方式：

- B-Tree 索引。
- Hash 索引。

##### 磁盘的相关知识

系统从磁盘读取数据基本单位：磁盘块（block）

InnoDB 读取基本单位  page：16k > block

B-Tree 结构的数据可以让**系统高效的找到数据所在的磁盘块**。



**磁盘预读**

局部性原理：

当一个数据被用到时，其附近的数据也通常会马上被使用。

预读的长度一般为页（page）的整倍数

##### AVL-Tree（自平衡二叉树）

2-3树、红黑树等

##### B-Tree

B-Tree 中的每个节点根据实际情况可以包含大量的关键字信息和分支，如下图所示为一个 3 阶的 B-Tree：

[![B-Tree 的结构](http://static2.iocoder.cn/84ea509fa091a10add4e7614e6cb37db)](http://static2.iocoder.cn/84ea509fa091a10add4e7614e6cb37db)B-Tree 的结构

- **每个节点占用一个盘块的磁盘空间**
  - 一个节点上有两个升序排序的 key 和三个指向子树根节点的 point ，
  - point 存储的是子节点所在磁盘块的地址。
  - **两个 key 划分成的三个范围域**，对应三个 point 指向的子树的数据的范围域。

发现需要 3 次磁盘 I/O 操作，和 3 次内存查找操作。由于内存中的 key 是一个有序表结构，可以**利用二分法查找提高效率**。而 3 次磁盘 I/O 操作是影响整个 B-Tree 查找效率的决定因素**。B-Tree 相对于 AVLTree 缩减了节点个数**，使每次磁盘 I/O 取到内存的数据都发挥了作用，从而提高了查询效率。

#####  **B+Tree 索引**

将上一节中的 B-Tree 优化，由于 B+Tree 的非叶子节点只存储键值信息，假设每个磁盘块能存储 4 个键值及指针信息，则变成 B+Tree 后其结构如下图所示：

[![B+Tree 的结构](http://static2.iocoder.cn/259d196856a231aff5e3cf1505848af4)](http://static2.iocoder.cn/259d196856a231aff5e3cf1505848af4)B+Tree 的结构

B+Tree 是在 B-Tree 基础上的一种优化，**使其更适合实现外存储索引结构**，InnoDB存储引擎就是用 B+Tree 实现其索引结构。

- 有**数据记录节点都是按照键值大小顺序存放在同一层的叶子节点上，而非叶子节点上只存储 key 值信息**
- 去掉B-Tree节点的date数据，**key存储增多，减少树的深度（减少磁盘I/O次数）**

B+Tree 相对于 B-Tree 有几点不同：

- **非叶子节点只存储键值信息**。
  - key存储增多，减少树的深度（减少磁盘I/O次数）
- 所有叶子节点之间都有一个**链指针**。
  -  B+Tree 上有两个头指针，一个指向根节点，另一个指向关键字最小的叶子节点
  - 可以对 B+Tree 进行两种查找运算：一种是**对于主键**的**范围查找**和**分页查找**，另一种是从根节点开始，**进行随机查找**。
- **数据记录都存放在叶子节点中**。

推算：

- InnoDB 存储引擎中页的大小为 16KB，一般表的主键类型为 INT（占用4个字节） 或 BIGINT（占用8个字节），指针类型也一般为 4 或 8 个字节，也就是说一个页（B+Tree 中的一个节点）中大概存储 **16KB/(8B+8B)=1K 个键值**（因为是估值，为方便计算，这里的 K 取值为〖10〗^3）。也就是说一个深度为 3 的 B+Tree 索引可以维护10^3 *10^3* 10^3 = **10亿 条记录**。
- 实际情况中每个节点可能不能填充满，因此在数据库中，**B+Tree 的高度一般都在 2~4 层**。MySQL 的 `InnoDB` 存储引擎在设计时是**将根节点常驻内存**的，也就是说查找某一键值的行记录时最多只需要 1~3 次磁盘 I/O 操作。

##### **B-Tree 有哪些索引类型？**

索引类型分为**主键索引**和**非主键索引（辅助索引）**。

- **辅助索引**与聚集索引的区别在于辅助索引的**叶子节点**并不包含行记录的全部数据，而是存储相应行数据的**聚集索引键，即主键**。

当通过辅助索引来查询数据时，需要进过两步：

- 首先，InnoDB 存储引擎会遍历辅助索引找到主键。
- 然后，再通过主键在聚集索引中找到完整的行记录数据。

另外，InnoDB 通过**主键聚簇数据**，如果**没有定义主键**，会选择一个**唯一的非空索引代替**，如果没有这样的索引，会**隐式定义个主键作为聚簇索引**。

##### **聚簇索引的注意点有哪些？**

聚簇索引表**最大限度地提高了 I/O 密集型应用的性能**，但它也有以下几个限制：

- 1、**插入速度严重依赖于插入顺序**，按照主键的顺序插入是最快的方式，否则将会出现**页分裂**，严重影响性能。因此，对于 InnoDB 表，我们一般都会定义一个自增的 ID 列为主键。

  > 关于这一点，可能面试官会换一个问法。例如，为什么主键需要是**自增 ID** ，又或者为什么主键需要带有时间性关联。

- 2、**更新主键的代价很高**，因为将会导致被更新的行移动。

- 3、**回表：**二级索引访问需要两次索引查找，第一次找到主键值，第二次根据主键值找到行数据。

  > 当然，有一种情况可以**无需二次查找**，基于非主键索引查询，但是**查询字段只有主键 ID** ，那么在二级索引中就可以查找到。

- 4、**主键 ID 建议使用整型**。节点的数据可以存储更多主键 ID 。

##### 🦅 **什么是索引的最左匹配特性？**

当 B+Tree 的数据项是复合的数据结构，比如索引 `(name, age, sex)` 的时候，B+Tree 是按照从左到右的顺序来建立**组合索引搜索树**的（组合索引，就是三个索引在一个索引里面）。

- 比如当 `(张三, 20, F)` 这样的数据来检索的时候，B+Tree 会优先比较 name 来确定下一步的所搜方向，如果 name 相同再依次比较 age 和 sex ，最后得到检索的数据。
- 但当 `(20, F)` 这样的没有 name 的数据来的时候，B+Tree 就不知道下一步该查哪个节点，因为建立搜索树的时候 name 就是第一个比较因子，必须要先根据 name 来搜索才能知道下一步去哪里查询。
- 比如当 `(张三, F)` 这样的数据来检索时，B+Tree 可以用 name 来指定搜索方向，但下一个字段 age 的缺失，所以**只能把名字等于张三的数据都找到，然后再匹配性别是 F 的数据了。**

#### 🦅 **MyISAM 索引实现**

> 艿艿：注意，我们上面看到的都是 InnoDB 存储引擎下的索引实现。

MyISAM 索引的实现，和 InnoDB 索引的实现是一样使用 B+Tree ，**差别在于 MyISAM 索引文件和数据文件是分离的，索引文件仅保存数据记录的地址**。

1）主键索引：

MyISAM引 擎使用B+Tree作为索引结构，**叶节点的data域存放的是数据记录的地址**。下图是MyISAM主键索引的原理图：

[![主键索引](http://static2.iocoder.cn/d49d260fc1eb8f992df0401b70d70e3d)](http://static2.iocoder.cn/d49d260fc1eb8f992df0401b70d70e3d)主键索引

- 这里设表一共有三列，假设我们以 Col1 为主键，上图是一个 MyISAM 表的主索引（Primary key）示意。可以看出 MyISAM 的索引文件仅仅保存数据记录的地址。

2）辅助索引：

**在 MyISAM 中，主索引和辅助索引在结构上没有任何区别，只是主索引要求 key 是唯一的，而辅助索引的 key 可以重复。**如果我们在 Col2 上建立一个辅助索引，则此索引的结构如下图所示：

[![辅助索引](http://static2.iocoder.cn/2fb922405a35479fa99eb2de4708638c)](http://static2.iocoder.cn/2fb922405a35479fa99eb2de4708638c)辅助索引

- 同样也是一颗 B+Tree ，data 域保存数据记录的地址。因此，**MyISAM 中索引检索的算法为首先按照 B+Tree 搜索算法搜索索引，如果指定的 Key 存在，则取出其 data 域的值，然后以 data 域的值为地址，读取相应数据记录。**

MyISAM 的索引方式也叫做“**非聚集**”的，之所以这么称呼是为了与InnoDB 的聚集索引区分。

🦅 **MyISAM 索引与 InnoDB 索引的区别？**

- InnoDB 索引是聚簇索引，MyISAM 索引是非聚簇索引。

- InnoDB 的主键索引的叶子节点存储着行数据，因此主键索引非常高效。

- MyISAM 索引的叶子节点存储的是行数据地址，需要再寻址一次才能得到数据。

- InnoDB 非主键索引的叶子节点存储的是主键和其他带索引的列数据，因此查询时做到覆盖索引会非常高效。

  > **覆盖索引**，指的是基于非主键索引查询，但是查询字段只有主键 ID ，那么在二级索引中就可以查找到。
  >
  > 只需要在一棵索引树上就能获取SQL所需的所有列数据，无需回表，速度更快。
  >
  > 如何实现:
  >
  > 将被查询的字段，建立到联合索引里去。

### 事物

**什么是事务？事务的四大特性是什么？并发事务会带来哪些问题？有哪些解决方案？**

#### 什么是事物

多个操作执行，完全地执行，要么完全地不执行。

#### 四大特性（ACID）

**原子性**：对于其数据修改，**要么全都执行，要么全都不执行**

**一致性**：事务在完成时，必须使所有的数据都保持**一致状态**

**隔离性**：由**并发事务所作的修改**必须与任何其它并发事务操作**隔离**

**持久性**：事务完成之后，它对于系统的影响是永久性的，落地到磁盘。

#### 并发事务带来的问题

脏写问题：**多个事务**对同一行，然后基于**最初选定的值更新**该行时，会发生**丢失更新问题**。

脏读问题：**读取未提交的数据**（这条记录的数据就处于不一致状态）。

不可重复读问题：**一个事物**，在不同时间点**读取一条数据**的结果不同。

幻读问题：一个事物，在不同时间点**检索数据条目不同**。

#### 并发事务问题的解决方案（四种隔离级别）

**读未提交（Read Uncommitted）**：**允许脏读取，但不允许更新丢失（脏写）**。如果一个事务已经开始写数据，则另外一个事务则**不允许同时进行写操作**，但允许其他事务读此行数据。该隔离级别可以通过**“排他写锁”**实现。

- 可避免 脏写。
- 不可避免 脏读、不可重复读、幻读。

**读已提交（Read Committed）**：允许不可重复读取，但不允许脏读取。这可以通过**“瞬间共享读锁”和“排他写锁”实现**。读取数据的事务允许其他事务继续访问该行数据，但是**未提交的写事务将会禁止其他事务访问该行**。

- 可避免 脏读，
- 不可避免 不可重复读、幻读。Oracle采用读已提交。

**可重复读取（Repeatable Read）**：禁止不可重复读取和脏读取，但是**有时可能出现幻读数据**。这可以通过**“共享读锁”和“排他写锁”**实现。**读取数据的事务将会禁止写事务（但允许读事务），写事务则禁止任何其他事务**。

- 可避免 脏读、不可重复读， 不可避免 幻读。MySQL采用可重复读。

**序列化（Serializable）**：提供严格的事务隔离。它要求事务序列化执行，**事务只能一个接着一个地执行，不能并发执行**。仅仅通过“行级锁”是无法实现事务序列化的，必须通过其他机制保证新插入的数据不会被刚执行查询操作的事务访问到。

- 可避免 脏读、不可重复读、幻读情况的发生。

**Mysql默认的事务隔离级别是可重复读，用Spring开发程序时，如果不设置隔离级别默认用Mysql设置的隔离级别，如果Spring设置了就用已经设置的隔离级别**



### （重点）丁奇老师的 [《MySQL 实战 45 讲》](http://www.iocoder.cn/images/jikeshijian/MySQL实战45讲.jpg) 的「08 | 事务到底是隔离的还是不隔离的？」

## 【重点】请说说 MySQL 的锁机制？

表锁是日常开发中的常见问题，因此也是面试当中最常见的考察点，当多个查询同一时刻进行数据修改时，就会产生并发控制的问题。MySQL **的共享锁和排他锁，就是读锁和写锁**。

- 共享锁：不堵塞，多个用户**可以同时读一个资源**，互不干扰。
- 排他锁：一个**写锁会阻塞其他的读锁和写锁**，这样可以只允许一个用户进行写入，防止其他用户读取正在写入的资源。

🦅 **锁的粒度？**

- 表锁：系统开销最小，会锁定整张表，MyIsam 使用表锁。
- 行锁：**最大程度的支持并发处理**，但是也带来了最大的锁开销，InnoDB 使用行锁。

🦅 **什么是悲观锁？什么是乐观锁？**

1）悲观锁：保守态度，因此，**在整个数据处理过程中，将数据处于锁定状态**。悲观锁的实现，往往依靠**数据库提供的锁机制**。

> 艿艿：悲观锁，就是我们上面看到的共享锁和排他锁。

2）乐观锁

悲观锁大多数情况下依靠数据库的锁机制实现，据库性能的大量开销（什么样的开销）。

乐观锁，大多是基于数据版本（ Version ）记录机制实现。

> 艿艿：乐观锁，实际就是通过版本号，从而实现 CAS 原子性更新。

🦅 **什么是死锁？**

- 设置获得锁的超时时间。

  > 通过超时，至少保证最差最差最差情况下，可以有退出的口子。

- 按同一顺序访问对象。

  > 这个是最重要的方式。

🦅 **MySQL 中 InnoDB 引擎的行锁是通过加在什么上完成(或称实现)的？为什么是这样子的？？**

InnoDB 是基于索引来完成行锁。例如：`SELECT * FROM tab_with_index WHERE id = 1 FOR UPDATE` 。

- `FOR UPDATE` 可以根据条件来完成**行锁**锁定，并且 id 是有索引键的列,如果 id 不是索引键那么 InnoDB 将完成**表锁**，并发将无从谈起。

🦅 **关于熟悉 MySQL 的锁机制？**

> 艿艿：这个问题，艿艿也没特别研究，先 mark 在这里。

- gap 锁

- next-key 锁

- Innodb 的行锁是怎么实现的？

  > Innodb 的锁的策略为 next-key 锁，即 record lock + gap lock ，是通过在 index 上加 lock 实现的。
  >
  > - 如果 index 为 unique index ，则降级为 record lock 行锁。
  > - 如果是普通 index ，则为 next-key lock 。
  > - 如果没有 index ，则直接锁住全表，即表锁。

- MyISAM 的表锁是怎么实现的？

  > MyISAM 直接使用表锁。

## 【重要】MySQL 查询执行顺序？

MySQL 查询执行的顺序是：

```java
(1)     SELECT
(2)     DISTINCT <select_list>
(3)     FROM <left_table>
(4)     <join_type> JOIN <right_table>
(5)     ON <join_condition>
(6)     WHERE <where_condition>
(7)     GROUP BY <group_by_list>
(8)     HAVING <having_condition>
(9)     ORDER BY <order_by_condition>
(10)    LIMIT <limit_number>
```

具体的，可以看看 [《SQL 查询之执行顺序解析》](http://zouzls.github.io/2017/03/23/SQL查询之执行顺序解析/) 文章。

**HAVING与WHERE的区别**

- HAVING和WHERE都是用来**过滤数据**的，他们可以使用相同的运算符来进行数据过滤，不同的是：

- **WHERE发生在HAVING之前**，在执行HAVING之前，会先将不符合WHERE条件的数据过滤掉。**WHERE过滤的是行，而HAVING过滤的是分组聚集后的数据**。

## 【重要】聊聊 MySQL SQL 优化？

可以看看如下几篇文章：

- [《PHP 面试之 MySQL 查询优化》](https://www.jianshu.com/p/ab958a4823d1)
- [《【面试】【MySQL常见问题总结】【03】》](https://blog.csdn.net/DERRANTCM/article/details/51534411) 第 078、095、105 题

另外，除了从 SQL 层面进行优化，也可以从服务器硬件层面，进一步优化 MySQL 。具体可以看看 [《MySQL 数据库性能优化之硬件优化》](https://blog.csdn.net/bemavery/article/details/46241533) 。

## 【加分】什么是 MVCC ？

> 艿艿：这是一个面试的加分题，一些大厂比较喜欢问，例如蚂蚁金服。

多版本并发控制（MVCC），是一种用来**解决读-写冲突**的无锁并发控制，也就是为事务分配单向增长的时间戳，为每个修改保存一个版本，版本与事务时间戳关联，读操作只读该事务开始前的数据库的快照。 这样在读操作不用阻塞写操作，写操作不用阻塞读操作的同时，避免了脏读和不可重复读。

推荐可以看看如下资料：

- 沈询 [《在线分布式数据库原理与实践》](https://www.imooc.com/learn/272)

  > 一共 1 小时 53 分钟，有趣，牛逼，强烈推荐！！！

- 钟延辉

  - [《分布式数据库 MVCC 技术探秘 (1)》](https://mp.weixin.qq.com/s/sOxLZlXRYR-zZKStE7qAwg)
  - [《分布式数据库 MVCC 技术探秘(2): 混合逻辑时钟》](https://mp.weixin.qq.com/s/8lX3Gyq4J5vLHETtG01EdA)

## 编写 SQL 查询语句的考题合集

因为考题比较多，艿艿就不一一列举，瞄了一些还不错的文章，如下：

- [《10 道 MySQL 查询语句面试题》](https://www.yanxurui.cc/posts/mysql/2016-11-10-10-sql-interview-questions/)
- [《MySQL 开发面试题》](https://www.cnblogs.com/geaozhang/p/6839297.html)
- [《企业面试题｜最常问的 MySQL 面试题集合（二）》](https://juejin.im/entry/5b57ebdcf265da0f61320e6f)

## MySQL 数据库 CPU 飙升到 500% 的话，怎么处理？

当 CPU 飙升到 500% 时，先用操作系统命令 **top 命令**观察是不是 mysqld 占用导致的，如果不是，找出占用高的进程，并进行相关处理。

> 如果此时是 IO 压力比较大，可以使用 **iostat 命令**，定位是哪个进程占用了磁盘 IO 。

如果是 mysqld 造成的，使用 `show processlist` 命令，**看看里面跑的 Session 情况，是不是有消耗资源的 SQL 在运行**。

- 一般来说，肯定要 kill 掉这些线程(同时观察 CPU 使用率是否下降)，等进行相应的调整(比如说加索引、改 SQL 、改内存参数)之后，再重新跑这些 SQL。

> 也可以**查看 MySQL 慢查询日志**，看是否有慢 SQL 。

**也有可能是每个 SQL 消耗资源并不多，但是突然之间，有大量的 Session 连进来导致 CPU 飙升**，这种情况就需要跟应用一起来分析为何连接数会激增，再做出相应的调整，比如说限制连接数等。

# 运维

理解一遍，即使有蛮多不会，也不要担心太多。

## Innodb 的事务与日志的实现方式

🦅 **有多少种日志？**

- redo 日志
- undo 日志

🦅 **日志的存放形式？**

- redo：**在页修改的时候，先写到 redo log buffer 里面， 然后写到 redo log 的文件系统缓存里面(fwrite)，然后再同步到磁盘文件（fsync）**。
- undo：在 MySQL5.5 之前，undo 只能存放在 ibdata *文件里面， 5.6 之后，可以通过设置 innodb_undo_tablespaces 参数把 undo log 存放在 ibdata* 之外。

🦅 **事务是如何通过日志来实现的，说得越深入越好**

> 艿艿：这个流程的理解还是比较简单的，实际思考实现感觉还是蛮复杂的。

基本流程如下：

- 因为事务在修改页时，要先记 undo ，在记 undo 之前要记 undo 的 redo， 然后修改数据页，再记数据页修改的 redo。 redo（里面包括 undo 的修改）一定要比数据页先持久化到磁盘。
- 当事务需要回滚时，**因为有 undo**，可以把数据页回滚到前镜像的状态。
- 崩溃恢复时，如果 redo log 中事务没有对应的 commit 记录，那么需要用 undo 把该事务的修改回滚到事务开始之前。如果有 commit 记录，就用 redo 前滚到该事务完成时并提交掉。

## MySQL binlog 的几种日志录入格式以及区别

## MySQL 主从复制的流程是怎么样的？

MySQL 的主从复制是基于如下 3 个线程的交互（多线程复制里面应该是 4 类线程）：

- 1、**Master 上面的 binlog dump 线程，该线程负责将 master 的 binlog event 传到 slave 。**
- 2、**Slave 上面的 IO 线程**，该线程负责接收 Master 传过来的 binlog，并**写入 relay log** 。
- 3、Slave 上面的 **SQL 线程**，该线程负责读取 relay log 并执行。
- 4、如果是多线程复制，无论是 5.6 库级别的假多线程还是 MariaDB 或者 5.7 的真正的多线程复制， SQL 线程只做 coordinator ，只负责把 relay log 中的 binlog 读出来然后交给 worker 线程， woker 线程负责具体 binlog event 的执行。

🦅 **MySQL 如何保证复制过程中数据一致性？**



🦅 **MySQL 如何解决主从复制的延时性？**

- 5.7 是真正意义的多线程复制，它的原理是基于 group commit， 只要 master 上面的事务是 group commit 的，那 slave 上面也可以通过多个 worker线程去并发执行。 

🦅 **工作遇到的复制 bug 的解决方法？**

- 5.6 的多库复制有时候自己会停止，我们写了一个脚本重新 start slave 。

🦅 **你是否做过主从一致性校验，如果有，怎么做的，如果没有，你打算怎么做？**

- **主从一致性校验有多种工具** 例如 `checksum`、`mysqldiff`、pt-table-checksum 等。

## MySQL 有哪些日志？

- **错误日志**：记录了当 mysqld 启动和停止时，以及服务器在运行过程中发生任何严重错误时的相关信息。

  - ```java
    mysql> show variables like 'log_error'\G;
    ```

    

- **二进制文件**：记录了所有的 DDL（数据定义语言）语句和 DML（数据操纵语言）语句，不包括数据查询语句。语句以“事件”的形式保存，它描述了数据的更改过程。（定期删除日志，默认关闭）。

  > 就是我们上面看到的 **MySQL binlog 日志**。

- **查询日志**：**记录了客户端的所有语句，格式为纯文本格式，可以直接进行读取**。（log 日志中记录了所有数据库的操作，对于访问频繁的系统，此日志对系统性能的影响较大，建议关闭，默认关闭）。

- **慢查询日志**：**慢查询日志记录了包含所有执行时间超过参数long_query_time（单位：秒）所设置值的 SQL 语句的日志。（纯文本格式）**

  - 开启慢查询日志：

    ```java
    mysql> set global slow_query_log='ON';
    ```

  - 默认的慢查询日志文件目录

    ```java
    mysql> show variables like 'slow_query_log_file'\G;
    ```

  > 重要，一定要开启。

另外，错误日志和慢查询日志的详细解释，可以看看 [《MySQL 日志文件之错误日志和慢查询日志详解》](https://blog.csdn.net/xlgen157387/article/details/76019934) 文章。

## 聊聊 MySQL 监控？

**你是如何监控你们的数据库的？**

监控的工具有很多，例如 Zabbix ，Lepus ，我这里用的是 [Lepus](https://github.com/lijiapengsa/lepus) 。

## 对一个大表做在线 DDL ，怎么进行实施的才能尽可能降低影响？

使用 pt-online-schema-change ，具体可以看看 [《MySQL 大表在线 DML 神器–pt-online-schema-change》](https://blog.csdn.net/mchdba/article/details/76253016) 文章。

另外，还有一些其它的工具，胖友可以搜索下。

# 彩蛋

T T 知识点真多，胖友还是重点掌握文初说的几个，不要方，我们能赢。如下是艿艿在网络上找到过的一个 MySQL 问题清单（研发向）：

> - 3-1 数据库架构
> - 3-2 优化你的索引-运用二叉查找树
> - 3-3 优化你的索引-运用B树
> - 3-4 优化你的索引-运用B+树
> - 3-5 优化你的索引-运用Hash以及BitMap
> - 3-6 密集索引和稀疏索引的区别
> - 3-7 索引额外的问题之如何调优Sql
> - 3-8 索引额外问题之最左匹配原则的成因
> - 3-9 索引额外问题之索引是建立越多越好吗
> - 3-10 锁模块之MyISAM与InooDB关于锁方面的区别
> - 3-11 锁模块之MyISAM与InooDB关于锁方面的区别_2
> - 3-12 锁模块之数据库事务的四大特性
> - 3-13 锁模块之事务并发访问产生的问题以及事务隔离机制
> - 3-14 锁模块之事务并发访问产生的问题以及事务隔离机制_2
> - 3-15 锁模块之当前读和快照读
> - 3-16 锁模块之RR如何避免幻读
> - 3-17 锁模块小结
> - 3-18 关键语法讲解
> - 3-19 本章总结
> - 3-20 彩蛋之面试的三层架构

------

参考与推荐如下文章：

- ranjun940726 [《PHP 面试指南》](https://www.kancloud.cn/ranjun940726/php_interview)
- 紫葡萄0 [《MySQL 索引的使用和优化》](https://www.jianshu.com/p/07c42e310e67)
- Ddaidai [《【MySQL】20 个经典面试题》](https://www.jianshu.com/p/977a9e7d80b3)
- 瘦瘦鸭 [《MySQL 面试知识点总结》](https://segmentfault.com/a/1190000014316316)
- 立超的专栏 [《MyISAM 和 InnoDB 的索引实现》](https://www.cnblogs.com/zlcxbb/p/5757245.html)
- 时芥蓝 [《MySQL 面试之必会知识点》](https://www.jianshu.com/p/5052f6a454ef)
- derrantcm [《【面试】【MySQL常见问题总结】【04】》](https://blog.csdn.net/DERRANTCM/article/details/51535193)
- mrlapulga [《MySQL 经典面试题》](https://www.mrlapulga.com/?p=59) 提供的面试题，难的想哭。
- 小麦苗 [《MySQL 笔试面试题集合》](https://yq.aliyun.com/articles/283427) 全的想哭。

**文章目录**

1. 开发
   1. [为什么互联网公司一般选择 MySQL 而不是 Oracle?](http://svip.iocoder.cn/MySQL/Interview/#为什么互联网公司一般选择-MySQL-而不是-Oracle)
   2. [数据库的三范式是什么？什么是反模式？](http://svip.iocoder.cn/MySQL/Interview/#数据库的三范式是什么？什么是反模式？)
   3. [MySQL 有哪些数据类型？](http://svip.iocoder.cn/MySQL/Interview/#MySQL-有哪些数据类型？)
   4. [MySQL 有哪些存储引擎？](http://svip.iocoder.cn/MySQL/Interview/#MySQL-有哪些存储引擎？)
   5. [【重点】什么是索引？](http://svip.iocoder.cn/MySQL/Interview/#【重点】什么是索引？)
   6. [【重点】MySQL 索引的原理？](http://svip.iocoder.cn/MySQL/Interview/#【重点】MySQL-索引的原理？)
   7. [【重点】请说说 MySQL 的四种事务隔离级别？](http://svip.iocoder.cn/MySQL/Interview/#【重点】请说说-MySQL-的四种事务隔离级别？)
   8. [【重点】请说说 MySQL 的锁机制？](http://svip.iocoder.cn/MySQL/Interview/#【重点】请说说-MySQL-的锁机制？)
   9. [【重要】MySQL 查询执行顺序？](http://svip.iocoder.cn/MySQL/Interview/#【重要】MySQL-查询执行顺序？)
   10. [【重要】聊聊 MySQL SQL 优化？](http://svip.iocoder.cn/MySQL/Interview/#【重要】聊聊-MySQL-SQL-优化？)
   11. [【加分】什么是 MVCC ？](http://svip.iocoder.cn/MySQL/Interview/#【加分】什么是-MVCC-？)
   12. [编写 SQL 查询语句的考题合集](http://svip.iocoder.cn/MySQL/Interview/#编写-SQL-查询语句的考题合集)
   13. [MySQL 数据库 CPU 飙升到 500% 的话，怎么处理？](http://svip.iocoder.cn/MySQL/Interview/#MySQL-数据库-CPU-飙升到-500-的话，怎么处理？)
2. 运维
   1. [Innodb 的事务与日志的实现方式](http://svip.iocoder.cn/MySQL/Interview/#Innodb-的事务与日志的实现方式)
   2. [MySQL binlog 的几种日志录入格式以及区别](http://svip.iocoder.cn/MySQL/Interview/#MySQL-binlog-的几种日志录入格式以及区别)
   3. [MySQL 主从复制的流程是怎么样的？](http://svip.iocoder.cn/MySQL/Interview/#MySQL-主从复制的流程是怎么样的？)
   4. [聊聊 MySQL 备份方式？备份策略是怎么样的？](http://svip.iocoder.cn/MySQL/Interview/#聊聊-MySQL-备份方式？备份策略是怎么样的？)
   5. [聊聊 MySQL 集群?](http://svip.iocoder.cn/MySQL/Interview/#聊聊-MySQL-集群)
   6. [聊聊 MySQL 安全？](http://svip.iocoder.cn/MySQL/Interview/#聊聊-MySQL-安全？)
   7. [MySQL 有哪些日志？](http://svip.iocoder.cn/MySQL/Interview/#MySQL-有哪些日志？)
   8. [聊聊 MySQL 监控？](http://svip.iocoder.cn/MySQL/Interview/#聊聊-MySQL-监控？)
   9. [对一个大表做在线 DDL ，怎么进行实施的才能尽可能降低影响？](http://svip.iocoder.cn/MySQL/Interview/#对一个大表做在线-DDL-，怎么进行实施的才能尽可能降低影响？)
3. [彩蛋](http://svip.iocoder.cn/MySQL/Interview/#彩蛋)

© 2014 - 2020 芋道源码 | 

总访客数 841210 次 && 总访问量 3974138 次

[回到首页](http://svip.iocoder.cn/index)



https://mp.weixin.qq.com/s/Qfvfj9R_dJiV65s48IdLdQ

https://www.jianshu.com/u/b64b58a43e1c

