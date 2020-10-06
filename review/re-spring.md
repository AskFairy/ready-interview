### 拦截器和过滤器

https://www.cnblogs.com/panxuejun/p/7715917.html

#### IOC

tomcat在启动的时候，直接会启动 spring容器

spring loC, spring容器，根据Xm配置，或者是你的注解，去实例化你的一些bean对象，然后根据xm配置或者注解，去对bean对象之间的引用关系，去进行依赖注入，某个bean依赖了另外一个bean

底层的核心技术，反射，他会通过反射的技术，直接根据你的类去自己构建对应的对象出来，用的就是反射技术spring Ioc，

系统的类与类之间彻底的解耦合现在这套比较高大上的一点系统里，有几十个类都使用了@ Resource

#### 反射

<img src="https://gitee.com//chenchong0817/picture/raw/master/Aaron/20200928202111.png" alt="image-20200928202109635" style="zoom:50%;" />

1. getDeclaredFiled 仅能获取类本身的属性成员（包括私有、共有、保护） 
2. getField 仅能获取类(及其父类可以自己测试) public属性成员

#### 注解注入

<img src="https://gitee.com//chenchong0817/picture/raw/master/Aaron/20200928202954.png" alt="image-20200928202951915" style="zoom:50%;" />

### ICO容器



![image-20200928204832272](https://gitee.com//chenchong0817/picture/raw/master/Aaron/20200928205135.png)

![image-20200928205153278](https://gitee.com//chenchong0817/picture/raw/master/Aaron/20200928205155.png)

3. 监听器模式

<img src="upload\image-20200928205517338.png" alt="image-20200928205517338" style="zoom:33%;" />

![image-20200928211039519](https://gitee.com//chenchong0817/picture/raw/master/Aaron/20200928211041.png)

- BeanFactory：模板工厂
- FactoryBean：个性化定制Bean

#### 核心方法

![image-20200928211433401](https://gitee.com//chenchong0817/picture/raw/master/Aaron/20200928211435.png)





1. 先读取环境值：

![image-20200928211259988](https://gitee.com//chenchong0817/picture/raw/master/Aaron/20200928211303.png)

2. 再得到工厂BeanFactory
3. BeanFactoryPostProcessor
4.  BeanPostProcessor（AOP在这实现）
5. 事件，观察者模式
6. 开始对象实例化