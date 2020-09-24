

## 基础



### ThreadLocal

如果共享数据的代码是能保证在同一个线程中执行，即**一个变量只要被某个线程独享**。

我们可以通过**ThreadLocal类来将变量存储到线程本地**，来解决线程安全问题。

#### 应用场景

1. `SimpleDateFormat`
   - SimpleDateFormat(下面简称sdf)类内部有一个Calendar对象引用,它用来储存和这个sdf相关的日期信息如果你的sdf是个static的, 那么**多个thread** 之间就会共享这个sdf, 同时也是共享这个Calendar引用,**对这个Calendar同时操作就会导致读写不一致的情况**,常见的异常有: 1.数组下标越界; 2. input为空字符串。
   - 可以使用ThreadLocal 分别为每个线程创建独立的 SimpleDateFormat。
2. 基于[谷歌`Dapper`](https://bigbully.github.io/Dapper-translation/)论文实现**非入侵全链路追踪**，使用 `ThreadLocal` 记录方法执行ID。
3. MDC日志框架

#### 实现原理

每一个线程的`Thread对象中都有一个ThreadLocalMap对象`，其中存储了一组`以ThreadLocal.threadLocalHashCode为键`，`以本地线程变量为值`的K-V值对。

`ThreadLocal对象`就是当前线程的ThreadLocalMap的`访问入口`，`每一个 ThreadLocal对象`都包含了一个独一无二的`threadLocalHashCode`值，使用这个值就可以在线程K-V值对中找回对应的本地线程变量。

它是一个弱引用实现，当

`ThreadLocal对象`就是当前线程的ThreadLocalMap的`访问入口`，`每一个 ThreadLocal对象`都包含了一个独一无二的`threadLocalHashCode`值，使用这个值就可以在线程K-V值对中找回对应的本地线程变量。



## 并发工具类

产生背景、应用场景以及实现原理

### Lock和Condition

Java SDK 并发包通过 `Lock` 和 `Condition` 两个接口来实现管程，其中 **Lock 用于解决互斥问题，Condition 用于解决同步问题**。

#### 产生背景

Java 语言本身提供的 synchronized 也是管程的一种实现，那为什么还要在 SDK 里提供Lock实现呢？

