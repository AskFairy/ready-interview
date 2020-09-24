

## 基础



## 并发工具类

产生背景、应用场景以及实现原理

### Lock和Condition

Java SDK 并发包通过 `Lock` 和 `Condition` 两个接口来实现管程，其中 **Lock 用于解决互斥问题，Condition 用于解决同步问题**。

#### 产生背景

Java 语言本身提供的 synchronized 也是管程的一种实现，那为什么还要在 SDK 里提供Lock实现呢？

