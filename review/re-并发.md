

## 基础



### ThreadLocal

如果共享数据的代码是能保证在同一个线程中执行，即**一个变量只要被某个线程独享**。

我们可以通过**ThreadLocal类来将变量存储到线程本地**，来解决线程安全问题。

`数据结构`、`拉链存储`、`斐波那契散列`、`神奇的0x61c88647`、`弱引用Reference`、`过期key探测清理和启发式清理`等等。

#### 应用场景

1. `SimpleDateFormat`
   - SimpleDateFormat(下面简称sdf)类内部有一个Calendar对象引用,它用来储存和这个sdf相关的日期信息如果你的sdf是个static的, 那么**多个thread** 之间就会共享这个sdf, 同时也是共享这个Calendar引用,**对这个Calendar同时操作就会导致读写不一致的情况**,常见的异常有: 1.数组下标越界; 2. input为空字符串。
   - 可以使用ThreadLocal 分别为每个线程创建独立的 SimpleDateFormat。
2. 基于[谷歌`Dapper`](https://bigbully.github.io/Dapper-translation/)论文实现**非入侵全链路追踪**，使用 `ThreadLocal` 记录方法执行ID。
3. MDC日志框架

#### 实现原理

每一个线程的`Thread对象中都有一个ThreadLocalMap对象`，

ThreadLocalMap对象底层采用数组结构Entry[]存储数据，Entry[]是一个弱引用实现，当Key就是ThreadLocal对象、不是强引用时，发生GC时就会被垃圾回收。

`ThreadLocal对象`就是当前线程的ThreadLocalMap的`访问入口`，

**为了让数据更加散列，减少哈希碰撞** `ThreadLocal` 使用的就是 **斐波那契（Fibonacci）散列法 + 拉链法存储数据到数组结构中**。

`f(k) = (k * 2654435769) >> 28`

只要实例化一个 `ThreadLocal` 对象，就会获取一个相应的哈希值`threadLocalHashCode`值。

**添加时**

并通过`int i = key.threadLocalHashCode & (len-1);`获取数组下标的。

- 插入时待插入的下标，是空位置直接插入。

- 待插入的下标，不为空，key 相同，直接更新

- 待插入的下标，不为空，key 不相同，拉链法寻址

- 不为空，key 不相同，碰到过期key。碰到这种情况，`ThreadLocal` 会进行探测清理过期key。

添加可能涉及到元素扩容

扩容时，先进行启发式清理，并且如果数组中元素大于2 / 3 需要重新rehash() 

- 启发式清理：添加新元素或删除另一个过时元素时，调用方法扫描一段范围内的单元格，会找到所有垃圾，找到垃圾后（k == null）后，调用探测式清理，探测器清理返回下一个垃圾的位置。
- 探测式清理，是**以当前遇到的 GC 元素开始，向后不断的清理。直到遇到 null 为止，才停止 rehash 计算**。通过**重新散列**位于staleSlot和下一个null插槽之间的任何可能冲突的条目**来清除陈旧的条目**。进行清理过期元素，并把后续位置的元素，前移。

rehash() 时先探测石清理所有元素

如果清理后元素还大于threshold的3 / 4进行扩容操作

数组容量扩大原理的2倍，遍历所有元素，重新计算Hash，

**获取元素**

1. **直接定位**到，没有哈希冲突，直接返回元素即可。
2. **没有直接定**位到了，key不同，需要**拉链式寻找**。
3. **没有直接定位到**了，key不同，拉链式寻找，**遇到GC清理元素，需要探测式清理，再寻找元素。**









## 并发工具类

产生背景、应用场景以及实现原理

### Lock和Condition

Java SDK 并发包通过 `Lock` 和 `Condition` 两个接口来实现管程，其中 **Lock 用于解决互斥问题，Condition 用于解决同步问题**。

#### 产生背景

Java 语言本身提供的 synchronized 也是管程的一种实现，那为什么还要在 SDK 里提供Lock实现呢？

