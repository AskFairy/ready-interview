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

equals 可靠，但效率低

hashcode 效率高、但hash冲突不可靠