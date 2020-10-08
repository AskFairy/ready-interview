### Java8新特性

- 接口引进了`默认方法`
- `函数式接口`，以此为基础引入增加了`Lambda` 表达式来简化开发。
  - `Lambda` 表达式：访问局部变量，final，保护数据的一致性
  - 在 Lambda 表达式中对成员变量和静态变量拥有读写权限。，
- **四饼调用**
- 并且`内置`了很多函数式接口。内置函数( Comparator 、 Runnable 、断言、Function、Supplier生产者（空构、方对象不参方法）、消费者（空构、方对象参方法）)
- 用来包装对象，防止空指针异常的`Optionals`
- **Stream流**：Filter 过滤、Sorted 排序、Map 转换（替换对象对象或者属性）、Match 匹配、Count 计数、Reduce:合并
- Map是不支持 Stream 流的，但是，我们可以对其 key, values, entry 使用 流操作
- 日期 Date API：
- Annotations 注解：Java8中的注释是可重复的

### 重载和重写

多态

重载：编译期、不关注返回值、同一类

重写：运行期、父子类

### == 、hashcode 和 equals (比较、可靠、效率)、

比较

== 比较内存

hashcode 和 equals  比较对象

equals 可靠，但效率低，默认==、自定义重写

hashcode 效率高、但hash冲突不可靠

在引用类型比较的时候，需要重写equals和hashcode。首先用hashCode()去对比，再利用equals 兜底验证。

HashMap对比Key就是这么干的，运用逻辑运算符的短路性质，先对hash值，与上  （内存地址 或 equals ）。

### 8种基本类型

byte、1byte等于8bit。

boolean、1byte, 只有 0,1

short、2byte

char、2byte  范围 0~2^16-1

int、4byte，范围是-2^31~2^31

long、8byte  

float 4byte   单精度浮点数有效数字8位   -2^128~+2^128

double、8byte 双精度浮点数有效数字16位  -2^1024~+2^1024



范围是由指数的位数来决定的，**float指数8，double指数11**

公式：（浮点）数值 =    （符号位）尾数   ×   底数 ^ 指数

#### char 型变量中能不能存贮一个中文汉字？为什么？

看在什么语言了，**汉字在Unicode编码下占两个字节**，C语言char型只有一个字节不能存储。Java占两个字节可以存储。

### String、StringBuilder、StringBuffer总结

String不可变对象

`StringBuilder`、`StringBuffer`，因为*没有发生多线程竞争也就没有锁升级*，所以两个类耗时几乎相同，当然在单线程下更推荐使用 `StringBuilder` 。

for循环会被会被JVM编译期优化成 `StringBuilder` ，但所有的字符串链接操作，都`需要实例化一次StringBuilder`

#### intern()方法

向字符串常量池中注册

- 字符串常量池：
  - *有点像HashMap的哈希桶+链表的数据结构*，用来存放字符串
  - `StringTable` 是一个固定长度的数组 `1009` 个大小