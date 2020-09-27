#### 1. 数据库的三范式是什么？什么是反模式？

1. 原子性
2. 完全依赖主键
3. 直接依赖于主键

表多，join，性能

反设计模式：宽表

#### Mysql数据类型

#### varchar 则是一种可变长度的类型，（50）排序占内存少

#### int(11) 中的 11影响展示效果

#### 金钱decimal 

#### 自增主键，InnoDB类型重启丢失，MyISAM 类型记录不丢失

#### COUNT(*) ，InnoDB类型全表扫描，MyISAM 类型记录

#### InnoDB 的 [4 大特性](https://note.youdao.com/ynoteshare1/index.html?id=a6004953a0a7c80073ac74d8e76f1ebd&type=note)

- [插入缓冲](https://www.cnblogs.com/zhs0/p/10528520.html)(insert buffer/ change buffer) 
  - 非聚簇索引
  - 插入缓存，合并
  - 减少随机IO，提升性能
- 二次写(double write)
  - 写入磁盘之前，缓存，崩溃恢复
- 自适应哈希索引(ahi)
  - 监控热数据，自适应hash
- 预读(read ahead)：提升IO性能
  - 线性预读：下一个extent
  - 随机预读：当前extent中剩余的page

#### MySQL 存储引擎：

- `InnoDB`
- `MyISAM`
- MRG_MYISAM
- MEMORY

