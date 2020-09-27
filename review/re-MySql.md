InnoDB 的 [4 大特性](https://note.youdao.com/ynoteshare1/index.html?id=a6004953a0a7c80073ac74d8e76f1ebd&type=note)

- [插入缓冲](https://www.cnblogs.com/zhs0/p/10528520.html)(insert buffer/ change buffer) 
  - 非聚簇索引
  - 插入缓存，合并
  - 减少随机IO，提升性能
- 二次写(double write)
- 自适应哈希索引(ahi)
- 预读(read ahead)