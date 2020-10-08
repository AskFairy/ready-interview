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

#### StringBuilder 源码分析

##### 初始化

默认16、str.length() + 16

##### 添加元素

容量检测、 扩容为当前 容积 * 2 + 2、当前`char[]`元素拷贝

- 添加元素的方式是基于 `System.arraycopy` 拷贝操作进行的，这是一个本地方法。

##### toString()

 `String` 的构造函数传递数组进行转换的，copy个新的String字符串返回

#### StringBuffer 源码分析

`StringBuffer` 与 `StringBuilder`，API的使用和底层实现上基本一致，维度不同的是 `StringBuffer`方法上加了 `synchronized` 🔒锁，所以它是线程安全的。

### String s = new String("xyz") 会创建几个对象？

可能是2两个，也可能是一个。

### 对象的深拷贝（深克隆）、浅拷贝（浅克隆）？

浅拷贝:

- 基本数据类型：复制
- 非基本数据类型：复制引用

深拷贝：

- 非基本数据类型：生产新对象
- clone() 方法
- 实现 Serializable 接口

### 变量的值传递和引用传递?

​	值传递：

- 对基本型变量而言，传递`变量的拷贝`
- 引用传递：是对于**对象型变量**而言的，传递的是该`对象地址（引用）`的一个`副本`。浅拷贝就是这种情况。

### 自动拆装箱

​	自动装箱和拆箱，就是`基本类型和引用类型之间的自动转换`。

为了解决

- 一个`函数`需要传递一个Object变量、

- 原始数据类型和 Java `泛型`并不能配合使用

- 解决之前JDK中不能直接地向`集合`( Collection )中放入原始类型值的问题

  总之为了让代码简练


### Object有哪些方法？

- 返回当前类信息的**getClass方法**，获取Class对象。一般用来反射。
- 用来对象对比的**equals方法**。
- 获取hash值的**hashCode方法**，一般用来和equals方法配合使用对比引用类型。
- 用来clone对象的**clone方法**。
- 将对象用字符串表示的**toString方法**。
- 用来线程间通信的**wait()、notify()、notifyAll()**方法。
  - 让线程进入阻塞状态的wait()方法
  - 用来唤醒该对象监视器的一个等待线程的notify()
  - 以及用来唤醒全部等待线程的notifyAll()方法。
- 以及垃圾收集器删除对象之前会调用的**finalize() 方法，可以用了对象的自救**。

### 类的实例化顺序？

先静态后非静，先父后子的顺序。

先加载父类静态变量，父类静态代码块，子类静态变量，子类静态代码块。

后加载父类非静态变量，父类构造函数，子类非静态变量，子类构造函数。



`注：类属性初始化时虚拟机是不会声明属性的同时赋值的，它会把所有属性和方法全部声明完了再从头按代码顺序赋值。`

`静态属性是在类被加载之后就放进静态存储区的,而非静态属性需要类被实例化之后才被创建`

#### 为什么先加载父类再加载子类？

​	子类随时会用到父类中的东西