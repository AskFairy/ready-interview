# Spring 整体

## 什么是 Spring Framework？

Spring 是一个**轻量级**的**开源**应用框架，**降低应用程序开发的复杂度，可以集成其他框架**。

## Spring Framework 中有多少个模块，它们分别是什么？

**1、Spring core：核心容器**

主要实现**控制反转IoC和依赖注入DI、Bean配置以及加载**。

**2、Spring AOP：Spring面向切面编程**

**3、Spring context：Spring上下文**

Spring上下文是一个配置文件，向Spring框架提供上下文信息。

**4、Spring DAO**

Spring 中的DAO提供一致的方式访问数据库

**5、Spring ORM（Object Relation Mapper）对象关系映射模块**

**6、Spring Web模块**

Web模块建立在应用程序上下文模块之上，为基于Web的应用程序提供了上下文。Web层使用Web层框架，可选的，可以是Spring自己的MVC框架，或者提供的Web框架，如Struts等。

**7、Spring MVC**

MVC框架是一个全功能的构建Web应用程序的MVC实现。



## （重点）使用 Spring 框架能带来哪些好处？



- **DI 依赖注入**
- **轻量级**
- **面向切面编程(AOP)**
- **集成主流框架**
- **Web MVC 框架**
- **事务管理**

带来相应的缺点：

- 把很多 JavaEE 的东西封装了，在满足快速开发高质量程序的同时，**隐藏了实现细节**。


## （重点）Spring 框架中都用到了哪些设计模式？

- **代理模式** — 在 `AOP` 中被用的比较多。
- **单例模式** — 在 Spring 配置文件中定义的 **Bean 默认为单例模式**。
- **模板方法** — 用来解决代码重复的问题。比如 [RestTemplate](http://howtodoinjava.com/2015/02/20/spring-restful-client-resttemplate-example/)、JmsTemplate、JdbcTemplate 。
- **工厂模式、抽象工厂模式** — BeanFactory 用来创建对象的实例。

当然，感兴趣的胖友，觉得不过瘾，可以看看艿艿基友知秋写的几篇文章：

- [《Spring 框架中的设计模式(一)》](http://www.iocoder.cn/Spring/DesignPattern-1)
- [《Spring 框架中的设计模式(二)》](http://www.iocoder.cn/Spring/DesignPattern-2)
- [《Spring 框架中的设计模式(三)》](http://www.iocoder.cn/Spring/DesignPattern-3)
- [《Spring 框架中的设计模式(四)》](http://www.iocoder.cn/Spring/DesignPattern-4)
- [《Spring 框架中的设计模式(五)》](http://www.iocoder.cn/Spring/DesignPattern-5)

# Spring IoC

## 什么是 Spring IoC 容器？



Spring 框架的核心是 Spring IoC 容器。容器过读取提供的**配置元数据** Bean Definition **创建** Bean 对象，并使用**依赖注入**来管理管理的**完整生命周期**。

- 该配置元数据 Bean Definition 可以通过 `XML`，Java `注解`或 `Java Config 代码`**提供**。
- 反射注入属性

## 什么是依赖注入？

就是一种将调用者与被调用者分离的思想，在依赖注入中，你**不必主动**、手动创建对象，但**必须描述**如何创建它们。然后，再由 IoC 容器将它们装配在一起。

## IoC 和 DI 有什么区别？



**IoC是目的（思想），DI是手段**。IoC是指让生成类的方式由传统方式（new）反过来，既程序员不调用new,需要类的时候由框架注入（DI）

## 可以通过多少种方式完成依赖注入？

通常，依赖注入可以通过**三种**方式完成，即：

> 上面一个问题的三种方式的英文，下面是三种方式的中文。

- 接口注入
- **构造函数注入**
- **setter 注入**

| 构造函数注入                   | setter 注入                    |
| :----------------------------- | :----------------------------- |
| 没有部分注入                   | 有部分注入                     |
| 不会覆盖 setter 属性           | 会覆盖 setter 属性             |
| 任意修改都会**创建一个新实例** | 任意修改**不会创建一个新实例** |
| 适用于设置很**多**属性         | 适用于设置**少量**属性         |

- 实际场景下，**setting 注入使用的更多。**

## Spring 中有多少种 IoC 容器？

Spring 提供了**两种( 不是“个” ) IoC 容器**，分别是 `BeanFactory`、`ApplicationContext` 。

**BeanFactory**

BeanFactory ，就像一个包含 Bean 集合的工厂类。它会在客户端要求时**实例化 Bean 对象**。

**ApplicationContext（应用上下文）**

ApplicationContext 接口**扩展**了 BeanFactory 接口，它在 BeanFactory 基础上提供了一些额外的功能，**管理生命周期，自定义初始化**等。

另外，BeanFactory 也被称为**低级**容器，而 ApplicationContext 被称为**高级**容器。

1. **低级容器 加载配置文件（从 XML，数据库，Applet）**，并解析成 BeanDefinition 到低级容器中。getBean 的操作都是在低级容器里操作的
   -  加载配置文件，解析成 BeanDefinition 放在 Map 里。
   - 调用 getBean 的时候，从 BeanDefinition 所属的 Map 里，拿出 Class 对象进行实例化，同时，如果有依赖关系，将递归调用 getBean 方法 —— 完成依赖注入。
2. 加载成功后，高级容器启动高级功能，例如**接口回调，监听器，自动实例化单例**，发布事件等等功能。

## 请介绍下常用的 BeanFactory 容器？

BeanFactory 最常用的是 `XmlBeanFactory` 。它可以根据 XML 文件中定义的内容，创建相应的 Bean。

## 请介绍下常用的 ApplicationContext 容器？

以下是三种较常见的 ApplicationContext 实现方式：

- 1、ClassPathXmlApplicationContext ：从 ClassPath 的 XML 配置文件中读取上下文，并生成上下文定义。
- 2、FileSystemXmlApplicationContext ：由文件系统中的XML配置文件读取上下文。ApplicationContext context = new FileSystemXmlApplicationContext(“bean.xml”);
- 3、XmlWebApplicationContext ：由 Web 应用的XML文件读取上下文。例如我们在 Spring MVC 使用的情况。
- 目前我们更多的是使用 Spring Boot 为主，所以使用的是第四种 ApplicationContext 容器，`ConfigServletWebServerApplicationContext` 。

## 列举一些 IoC 的一些好处？

- 它将**最小化应用程序中的代码量**。
- 它以最小的影响和最少的侵入机制促进**松耦合**。
- 它**支持即时的实例化和延迟加载 Bean 对象**。
- 它将使您的应用程序易于测试，因为它不需要单元测试用例中的任何单例或 JNDI 查找机制。

## 简述 Spring IoC 的实现机制？

简单来说，Spring 中的 IoC 的实现原理，就是**工厂模式**，通过**反射机制**创建对象。代码如下：

在基友 [《面试问烂的 Spring IoC 过程》](http://www.iocoder.cn/Fight/Interview-poorly-asked-Spring-IOC-process-1/) 的文章中，把 Spring IoC 相关的内容，讲的非常不错。

只需要低级容器就可以实现，2 个步骤：

a. 加载配置文件，解析成 BeanDefinition 放在 Map 里。

b. 调用 getBean 的时候，从 BeanDefinition 所属的 Map 里，拿出 Class 对象进行实例化，同时，如果有依赖关系，将递归调用 getBean 方法 —— 完成依赖注入。

上面就是 Spring 低级容器（BeanFactory）的 IoC。

至于高级容器 ApplicationContext，他包含了低级容器的功能，当句话，他不仅仅是 IoC。他支持不同信息源头，支持 BeanFactory 工具类，支持层级容器，支持访问文件资源，支持事件发布通知，支持接口回调等等。

## Spring 框架中有哪些不同类型的事件？

Spring 的 ApplicationContext 提供了**支持事件和代码中监听器的功能**。

我们可以创建 Bean 用来监听在 ApplicationContext 中发布的事件。如果一个 Bean 实现了 ApplicationListener 接口，当一个ApplicationEvent 被发布以后，Bean 会自动被通知。

1. **上下文更新事件**（ContextRefreshedEvent）：该事件会在ApplicationContext 被初始化或者更新时发布。也可以在调用ConfigurableApplicationContext 接口中的 `#refresh()` 方法时被触发。
2. **上下文开始事件**（ContextStartedEvent）：当容器调用ConfigurableApplicationContext 的 `#start()` 方法开始/重新开始容器时触发该事件。
3. **上下文停止事件**（ContextStoppedEvent）：当容器调用 ConfigurableApplicationContext 的 `#stop()` 方法停止容器时触发该事件。
4. **上下文关闭事件**（ContextClosedEvent）：当ApplicationContext 被关闭时触发该事件。容器被关闭时，其管理的所有单例 Bean 都被销毁。
5. **请求处理事件**（RequestHandledEvent）：在 Web应用中，当一个HTTP **请求（request）结束触发**该事件。
6. 通过扩展 ApplicationEvent 类来开发**自定义**的事件。

# Spring Bean

## 什么是 Spring Bean ？

- Bean 由 Spring IoC 容器实例化，配置，装配和管理。
- Bean 是基于用户提供给 IoC 容器的**配置元数据 Bean Definition 创建**。

## Spring 有哪些配置方式

单纯从 Spring Framework 提供的方式，一共有三种：

- 1、XML 配置文件。

- 2、注解配置。

- 3、Java Config 配置。

  Spring 的 Java 配置是通过使用 @Bean 和 @Configuration 来实现。

  - Spring Boot 使用的就是

## Spring 支持几种 Bean Scope（作用域） ？

> 艿艿，这个是一个比较小众的题目，简单了解即可。

Spring Bean 支持 5 种 Scope ，分别如下：

- `Singleton` - 每个 Spring IoC 容器仅有一个单 Bean 实例。**默认**
- `Prototype` - 线程每次请求都会产生一个新的实例。
- `Request` - 每一次 HTTP 请求都会产生一个新的 Bean 实例，并且该 Bean 仅在当前 HTTP 请求内有效。
- `Session` - 每一个的 Session 都会产生一个新的 Bean 实例，同时该 Bean 仅在当前 HTTP Session 内有效。
- `global session/Application` - 每一个 Web Application 都会产生一个新的 Bean ，同时该 Bean 仅在当前 Web Application 内有效。
- 自定义 Bean Scope

## （重点）Spring Bean 在容器的生命周期是什么样的？

Spring Bean 的**初始化**流程如下：

- 实例化 Bean 对象

  - Spring 容器根据配置中的 Bean Definition(定义)中**实例化** Bean 对象。

    > Bean Definition 可以通过 XML，Java 注解或 Java Config 代码提供。

  - Spring 使用依赖注入**填充**所有属性，如 Bean 中所定义的配置。

- Aware 相关的属性，注入到 Bean 对象

  - 如果 Bean 实现 **BeanNameAware** 接口，则工厂通过传递 Bean 的 beanName 来调用 `#setBeanName(String name)` 方法。
  - 如果 Bean 实现 **BeanFactoryAware** 接口，工厂通过传递自身的实例来调用 `#setBeanFactory(BeanFactory beanFactory)` 方法。

- 调用相应的方法，进一步初始化 Bean 对象

  - 如果存在与 Bean 关联的任何 **BeanPostProcessor** 们，则调用 `#preProcessBeforeInitialization(Object bean, String beanName)` 方法。
  - 如果 Bean 实现 **InitializingBean** 接口，则会调用 `#afterPropertiesSet()` 方法。
  - 如果为 Bean 指定了 **init** 方法（例如 `` 的 `init-method` 属性），那么将调用该方法。
  - 如果存在与 Bean 关联的任何 **BeanPostProcessor** 们，则将调用 `#postProcessAfterInitialization(Object bean, String beanName)` 方法。

Spring Bean 的**销毁**流程如下：

- 如果 Bean 实现 **DisposableBean** 接口，当 spring 容器关闭时，会调用 `#destroy()` 方法。
- 如果为 bean 指定了 **destroy** 方法（例如 `` 的 `destroy-method` 属性），那么将调用该方法。

整体如下图：[![流程图](http://static2.iocoder.cn/images/Spring/2018-12-24/03.jpg)](http://static2.iocoder.cn/images/Spring/2018-12-24/03.jpg)流程图

无意中，艿艿又翻到一张有趣的整体图，如下图：

[![流程图](http://static2.iocoder.cn/images/Spring/2018-12-24/08.png)](http://static2.iocoder.cn/images/Spring/2018-12-24/08.png)流程图

## 什么是 Spring 的内部 bean？

只有将 Bean **仅**用作另一个 Bean 的属性时，才能将 Bean 声明为内部 Bean。

- 为了定义 Bean，Spring 提供基于 XML 的配置元数据在 ``或 `` 中提供了 ``元素的使用。
- 内部 Bean 总是**匿名**的，并且它们总是作为**原型 Prototype** 。

## 什么是 Spring 装配？

当 Bean 在 Spring 容器中组合在一起时，它被称为**装配**或 **Bean 装配**。

> 装配，和上文提到的 DI 依赖注入，实际是一个东西。

**自动装配有哪些方式？**

Spring 容器能够自动装配 Bean 。

自动装配的不同模式：

- **no** - 这是默认设置，表示没有自动装配。应使用显式 Bean 引用进行装配。
- **byName** - 它根据 Bean 的名称注入对象依赖项。它匹配并装配其属性与 XML 文件中由相同名称定义的 Bean 。
- 【最常用】**byType** - 它根据类型注入对象依赖项。如果属性的类型与 XML 文件中的一个 Bean 类型匹配，则匹配并装配属性。
- 构造函数
- autodetect - 首先容器尝试通过构造函数使用 autowire 装配，如果不能，则尝试通过 byType 自动装配。

## 解释什么叫延迟加载？

默认情况下，容器启动之后会将所有作用域为**单例**的 Bean 都创建好，但是有的业务场景我们并不需要它提前都创建好。此时，我们可以在Bean 中设置 `lzay-init = "true"` 。

## Spring 框架中的单例 Bean 是线程安全的么？

**不是线程安全的，作用域，共享变量**：Spring 框架并没有对[单例](http://howtodoinjava.com/2012/10/22/singleton-design-pattern-in-java/) Bean 进行任何多线程的封装处理。仅提供根据配置，创建单例 Bean 或多例 Bean 的功能。但实际上，大部分的 Spring Bean 并没有可变的状态(比如Serview 类和 DAO 类)，所以在某种程度上说 Spring 的单例 Bean 是线程安全的。

如果你的 Bean 有多种状态的话，就需要自行保证线程安全。最浅显的解决办法，就是将多态 Bean 的作用域( Scope )由 Singleton 变更为 Prototype 。

## （重点）Spring Bean 怎么解决循环依赖的问题？

https://blog.csdn.net/u010853261/article/details/77940767

# Spring 注解

## 什么是基于注解的容器配置？

不使用 XML 来描述 Bean 装配，开发人员通过在相关的类，方法或字段声明上使用**注解**将配置移动到组件类本身。它可以作为 XML 设置的替代方案。例如：

Spring 的 Java 配置是通过使用 `@Bean` 和 `@Configuration` 来实现。

## [@Configuration和@Component区别](https://blog.csdn.net/ctwy291314/article/details/104428533)

`@Configuration`注解中是包含`@Component`注解的

`@Configuration`就相当于Spring配置文件中的``标签，里面可以配置bean

概括就是 @Configuration 中所有带 @Bean 注解的**方法都会被动态代理**，因此调用该方法返回的都是**同一个实例**。

其工作原理是：如果方式是首次被调用那么原始的方法体会被执行并且结果对象会**被注册到Spring上下文中，之后所有的对该方法的调用仅仅只是从Spring上下文中取回该对象返回给调用者**。

@Component，只是纯JAVA方式的调用，多次调用该方法返回的是**不同的对象实例**。

## 如何在 Spring 中启动注解装配？

默认情况下，Spring 容器中未打开注解装配。使用 Spring Boot ，默认情况下已经开启。

## @Component, @Controller, @Repository, @Service 有何区别？

- `@Component` ：它将 Java 类标记为 Bean 。它是任何 Spring 管理组件的**通用**构造型。
- `@Controller` ：它将一个类标记为 Spring Web MVC **控制器**。
- `@Service` ：此注解是组件注解的特化。它不会对 `@Component` 注解提供任何其他行为。您可以在**服务层**类中使用 @Service 而不是 `@Component` ，因为它以更好的方式指定了意图。
- `@Repository` ：这个注解是具有类似用途和功能的 `@Component` 注解的特化。它为 **DAO** 提供了额外的好处。它将 DAO 导入 IoC 容器，并使未经检查的异常有资格转换为 Spring DataAccessException 。

## @Required 注解有什么用？

`@Required` 注解，应用于 Bean 属性 setter 方法。

- 此注解仅指示必须在配置时使用 Bean 定义中的显式属性值或使用自动装配填充受影响的 Bean 属性。
- 如果尚未填充受影响的 Bean 属性，则容器将抛出 BeanInitializationException 异常。

示例代码如下：

```
public class Employee {

    private String name;
    
    @Required
    public void setName(String name){
        this.name=name;
    }
    
    public string getName(){
        return name;
    }
    
}
```

- T T 貌似平时很少用这个注解噢。

## @Autowired 注解有什么用？

`@Autowired` 注解，可以更准确地控制应该在何处以及**如何进行自动装配bean**。

- 此注解用于在 setter 方法，构造函数，具有任意名称或多个参数的属性或方法上自动装配 Bean。
- 默认情况下，它是**类型**驱动的注入。

## @Autowired 与@Resource的区别

@Autowired默认按**类型装配**（这个注解是属业spring的）

@Resource（这个注解属于J2EE的），默认**按照名称进行装配**，名称可以通过name属性进行指定。当**找不到与名称匹配的bean时才按照类型进行装配**。

## @Qualifier 注解有什么用？

当你创建多个**相同类型**的 Bean ，并希望仅使用属性装配**其中一个** Bean 时，您可以使用 `@Qualifier` 注解和 `@Autowired` 通过指定 ID 应该装配哪个**确切的** Bean 来消除歧义。

# Spring AOP

AOP 的工作重心在于**如何将增强编织目标对象的连接点上**, 这里包含两个工作:

1. 如何通过 PointCut 和 Advice 定位到特定的 **JoinPoint** 上。
2. 如何在 Advice 中编写切面代码。

> Spring AOP 的面试题中，大多数都是概念题，主要是对切面的理解。概念点主要有：
>
> - AOP：面向切面编程，切面、动态代理
>   - `ProxyFactoryBean`代理工厂对象
>   - `TransactionProxyFactoryBean`事务代理工厂对象
> - Aspect：
>   - 用 @Aspect 注解的类就是切面
>   - Aspect 由 **PointCut** 和 **Advice** 组成。
> - JoinPoint：切点
>   - 一个方法的执行。
>   - 或者是一个异常的处理。
> - PointCut：**PointCut 是匹配 JoinPoint 的条件**。
> - Advice：特定 JoinPoint 处的 Aspect 所**采取的动作称为 Advice** 。
> - Target：织入 Advice 的**目标对象**。
> - AOP Proxy
> - Weaving

- 在阅读完 [「Spring AOP」](http://svip.iocoder.cn/Spring/Interview/#) 的面试题后，在回过头思考下这些概念点，到底理解了多少。注意，不是背，理解！

非常推荐阅读如下两篇文章：

- [《彻底征服 Spring AOP 之理论篇》](https://segmentfault.com/a/1190000007469968)
- [《彻底征服 Spring AOP 之实战篇》](https://segmentfault.com/a/1190000007469982)

[![流程图](http://static2.iocoder.cn/images/Spring/2018-12-24/04.jpg)](http://static2.iocoder.cn/images/Spring/2018-12-24/04.jpg)流程图



## 关于 JoinPoint 和 PointCut 的区别

1. 首先，Advice 通过 PointCut 查询需要被织入的 JoinPoint 。
2. 然后，Advice 在查询到 JoinPoint **上**执行逻辑。

## 什么是 Advice ？

Advice ，**通知**。

- 特定 JoinPoint 处的 Aspect 所**采取的动作称为 Advice** 。

**有哪些类型的 Advice？**

- Before - 这些类型的 Advice 在 JoinPoint 方法之前执行，并使用 `@Before` 注解标记进行配置。
- After Returning - 这些类型的 Advice 在连接点方法正常执行后执行，并使用 `@AfterReturning` 注解标记进行配置。
- After Throwing - 这些类型的 Advice 仅在 JoinPoint 方法通过抛出异常退出并使用 `@AfterThrowing` 注解标记配置时执行。
- After Finally - 这些类型的 Advice 在连接点方法之后执行，无论方法退出是正常还是异常返回，并使用 `@After` 注解标记进行配置。
- Around - 这些类型的 Advice 在连接点之前和之后执行，并使用 `@Around` 注解标记进行配置。

## 什么是 Target ？

Target ，织入 Advice 的**目标对象**。目标对象也被称为 **Advised Object** 。

- 因为 Spring AOP 使用运行时代理的方式来实现 Aspect ，因此 Advised Object 总是一个代理对象(Proxied Object) 。
- **注意, Advised Object 指的不是原来的对象，而是织入 Advice 后所产生的代理对象**。
- Advice + Target Object = Advised Object = Proxy 。

## AOP 有哪些实现方式？

实现 AOP 的技术，主要分为两大类：

- ① **静态代理** - 指使用 AOP 框架提供的命令进行编译，从而**在编译阶段**就可生成 AOP 代理类，因此也称为编译时增强。

  - 编译时编织（特殊编译器实现）

  - 类加载时编织（特殊的类加载器实现）。

- ② **动态代理** - 在**运行时在内存中“临时”生成 AOP 动态代理类**，因此也被称为运行时增强。目前 Spring 中使用了两种动态代理库：

  - JDK 动态代理：反射+接口
  - CGLIB：继承，子类覆盖

那么 Spring 什么时候使用 JDK 动态代理，什么时候使用 CGLIB 呢？

- 如果被代理的目标对象实现了至少一个接口，则会使用 JDK 动态代理。所有该目标类型实现的接口都讲被代理。
- 若该目标对象没有实现任何接口，则创建一个 CGLIB 代理。

或者，我们来换一个解答答案：

Spring AOP 中的动态代理主要有两种方式，

- JDK 动态代理

  JDK 动态代理通过**反射**来接收被代理的类，并且要求被代理的类**必须实现一个接口**。JDK动态代理的核心是 InvocationHandler 接口和 Proxy 类。

- CGLIB 动态代理

  **如果目标类没有实现接口**，那么 Spring AOP 会选择使用 CGLIB 来动态代理目标类。
  CGLIB（Code Generation Library），是一个代码生成的类库，可以在**运行时动态的生成某个类的子类**，注意，CGLIB 是通过**继承的方式做的动态代理**，因此如果某个类被标记为 `final` ，那么它是无法使用 CGLIB 做动态代理的。

## 什么是编织（Weaving）？

- 在 Spring AOP 中，编织在**运行时执行，即动态代理**。请参考下图：[![Proxy](http://static2.iocoder.cn/images/Spring/2018-12-24/05.jpg)](http://static2.iocoder.cn/images/Spring/2018-12-24/05.jpg)Proxy

## Spring 如何使用 AOP 切面？

在 Spring AOP 中，有两种方式配置 AOP 切面：

- 基于 **XML** 方式的切面实现。
- 基于 **注解** 方式的切面实现。

# Spring Transaction

非常推荐阅读如下文章：

- [《可能是最漂亮的 Spring 事务管理详解》](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247484702&idx=1&sn=c04261d63929db09ff6df7cadc7cca21&chksm=fa497aafcd3ef3b94082da7bca841b5b7b528eb2a52dbc4eb647b97be63a9a1cf38a9e71bf90&token=165108535&lang=zh_CN#rd)

## 什么是事务？

事务就是对一系列的数据库操作（比如插入多条数据）进行**统一的提交**回滚操作，如果插入成功，那么**一起成功**，如果中间有一条出现异常，那么**回滚**之前的所有操作。

这样可以**防止出现脏数据，防止数据库数据出现问题**。

## 事务的特性指的是？

指的是 **ACID** ，如下图所示：

[![事务的特性](http://static2.iocoder.cn/images/Spring/2018-12-24/06.png)](http://static2.iocoder.cn/images/Spring/2018-12-24/06.png)事务的特性

1. **原子性** Atomicity ：
2. **一致性** Consistency ：在事务开始之前和事务结束以后，**数据库的完整性没有被破坏**。这表示写入的资料必须完全符合所有的预设[约束](https://zh.wikipedia.org/wiki/数据完整性)、[触发器](https://zh.wikipedia.org/wiki/触发器_(数据库))、[级联回滚](https://zh.wikipedia.org/w/index.php?title=级联回滚&action=edit&redlink=1)等。
3. **隔离性** Isolation ：数据库允许**多个并发事务**同时对其数据进行读写和修改的能力，隔离性**可以防止多个事务并发执行时由于交叉执行而导致数据的不一致**。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。
4. **持久性** Durability ：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

## 列举 Spring 支持的事务管理类型？

目前 Spring 提供两种类型的事务管理：

- **声明式**事务：通过使用注解或基于 XML 的配置事务，从而事务管理与业务代码分离。
  - Spring Boot + 注解的**声明式**事务
- **编程式**事务：通过编码的方式实现事务管理，需要在代码中显式的调用事务的获得、提交、回滚。它为您提供极大的灵活性，但维护起来非常困难。

实际场景下，我们一般使用 Spring Boot + 注解的**声明式**事务。

## Spring 事务如何和不同的数据持久层框架做集成？

1. 定义责事务管理的接口PlatformTransactionManager
   - 返回的是 TransactionStatus 对象方法
   - 提交事务方法`#commit(TransactionStatus status)`
   - 回滚事务方法`#rollback(TransactionStatus status)`、
2. 基于**模板方法**，抽象子类AbstractPlatformTransactionManager
   - #doCommit(TransactionStatus status)
   - #doRollback(TransactionStatus status)
3. 最后，不同的数据持久层框架，会有其对应的 `PlatformTransactionManager` 实现类

[![事务的特性](http://static2.iocoder.cn/images/Spring/2018-12-24/07.png)](http://static2.iocoder.cn/images/Spring/2018-12-24/07.png)http://svip.iocoder.cn/MyBatis/Spring-Integration-4/)

## 为什么在 Spring 事务中不能切换数据源？

因为，在 Spring 的事务管理中，**所使用的数据库连接会和当前线程所绑定**，即使我们设置了另外一个数据源，使用的还是当前的数据源连接。

## @Transactional 注解有哪些属性？如何使用？

`@Transactional` 注解的**属性**如下：事务管理器，**传播行为**,**隔离级别**,**读写或只读**事务，默认**读写**,**事务超时时间**设置（秒）,导致/不导致回滚数组等

- 一般情况下，我们直接使用 `@Transactional` 的**所有属性默认值**即可。

具体**用法**如下：

- `@Transactional` 可以作用于**接口、接口方法、类以及类方法**上。
- 虽然 `@Transactional` 注解可以作用于接口、接口方法、类以及类方法上，但是 Spring 建议不要在接口或者接口方法上使用该注解，因为这只有在使用基于接口的代理时它才会生效。另外， **`@Transactional` 注解应该只被应用到 `public` 方法上，这是由 Spring AOP 的本质决定的**。如果你在 `protected`、`private` 或者默认可见性的方法上使用 `@Transactional` 注解，这将被忽略，也不会抛出任何异常。**这一点，非常需要注意**。

## 什么是事务的隔离级别？分成哪些隔离级别？

 * 【Spring 独有】使用后端数据库默认的隔离级别
 * MySQL 默认采用的 REPEATABLE_READ隔离级别
 * Oracle 默认采用的 READ_COMMITTED隔离级别

## 什么是事务的传播级别？分成哪些传播级别？

事务的**传播行为**，指的是**当前带有事务配置的方法，需要怎么处理事务**。

- 。*通过事务的传播级别，Spring 才知道如何处理事务，是创建一个新事务呢，还是继续使用当前的事务*。


在 TransactionDefinition 接口中，定义了**三类七种**传播级别**0-6**。代码如下：

支持当前事务: 存在继续用，不存在创建新的、非事物、抛异常

不支持当前事物：存在挂起，新事物、非事物，非事物运行，存在就抛异常

存在新事物当嵌套事物、没有新事物

```java
// TransactionDefinition.java

// ========== 支持当前事务的情况 ========== 

/**
 * 如果当前存在事务，则使用该事务。
 * 如果当前没有事务，则创建一个新的事务。
 */
int PROPAGATION_REQUIRED = 0;
/**
 * 如果当前存在事务，则使用该事务。
 * 如果当前没有事务，则以非事务的方式继续运行。
 */
int PROPAGATION_SUPPORTS = 1;
/**
 * 如果当前存在事务，则使用该事务。
 * 如果当前没有事务，则抛出异常。
 */
int PROPAGATION_MANDATORY = 2;

// ========== 不支持当前事务的情况 ========== 

/**
 * 创建一个新的事务。
 * 如果当前存在事务，则把当前事务挂起。
 */
int PROPAGATION_REQUIRES_NEW = 3;
/**
 * 以非事务方式运行。
 * 如果当前存在事务，则把当前事务挂起。
 */
int PROPAGATION_NOT_SUPPORTED = 4;
/**
 * 以非事务方式运行。
 * 如果当前存在事务，则抛出异常。
 */
int PROPAGATION_NEVER = 5;

// ========== 其他情况 ========== 

/**
 * 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行。
 * 如果当前没有事务，则等价于 {@link TransactionDefinition#PROPAGATION_REQUIRED}
 * 设置回滚到，嵌套事物回滚不影响
 */
int PROPAGATION_NESTED = 6;
```



## 什么是事务的只读属性？

事务的只读属性是指，对事务性资源进行只读操作或者是读写操作。

- 只读事务来保证读记录的数据一致性。

## 什么是事务的回滚规则？

回滚规则，定义了哪些异常会**导致事务回滚**而哪些不会。

- **默认情况下，事务只有遇到运行期异常时才会回滚，而在遇到检查型异常时不会回滚**
- 但是你可以声明事务在遇到特定的检查型异常时像遇到运行期异常那样回滚。同样，你还可以声明事务遇到特定的异常不回滚，即使这些异常是运行期异常。

注意，事务的回滚规则，并不是数据库事务规范中的名词，**而是 Spring 自身所定义的**。

## 简单介绍 TransactionStatus ？

TransactionStatus 接口，记录事务的状态，不仅仅包含事务本身，还包含事务的其它信息。

## 使用 Spring 事务有什么优点？

1. 通过 PlatformTransactionManager ，**为不同的数据层持久框架提供统一的 API** ，无需关心到底是原生 JDBC、Spring JDBC、JPA、Hibernate 还是 MyBatis 。
2. 通过使用**声明式事务，使业务代码和事务管理的逻辑分离**，更加清晰。

从倾向上来说，艿艿比较喜欢**注解** + 声明式事务。

## Spring 支持哪些 ORM 框架？

- Hibernate
- JPA
  - JPA规范本质上就是一种ORM规范，注意不是ORM框架——因为JPA并未提供ORM实现，它只是制订了一些规范，提供了一些编程的API接口，但具体实现则由服务厂商来提供实现
- MyBatis

可能会有胖友说，不是应该还有 Spring JDBC 吗。注意，Spring JDBC 不是 ORM 框架。













### 拦截器和过滤器

https://www.cnblogs.com/lynn16/p/10691224.html

https://www.cnblogs.com/panxuejun/p/7715917.html

- **拦截器**是基于java的**反射机制**的，而过滤器是基于**函数回调**。
  - 拦截是AOP的一种实现策略，动态代理的方式调用。
- **拦截器不依赖与servlet容器**，过滤器依赖与servlet容器。
- **拦截器可以获取IOC容器中的各个bean**，而过滤器就不行，这点很重要，在拦截器里注入一个service，可以调用业务逻辑。
- 触发时机不同：
  - 过滤器是在请求进入容器后，但请求进入servlet之前进行预处理的。请求结束返回也是，是在servlet处理完后，返回给前端之前。
    - **过滤器包裹住servlet，servlet包裹住拦截器**。
- **拦截器是spring容器**的，是spring支持的，
- 拦截器功在对请求权限鉴定方面确实很有用处

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

# Spring MVC

## Spring MVC 框架有什么用？

Spring Web MVC 框架快速灵活开发 **Web 应用程序**。

MVC 模式有助于分离应用程序的不同方面，如输入逻辑，业务逻辑和 UI 逻辑，同时在所有这些元素之间提供松散耦合。

## 介绍下 Spring MVC 的核心组件？

Spring MVC 一共有[九大核心组件](https://blog.csdn.net/qq_43716366/article/details/107308013)，分别是：

- MultipartResolver：文件解析器，用于**处理上传请求**
- LocaleResolver：LocaleResolver用于从request解析出Locale，Locale就是**zh-cn之类**（国际化），表示一个区域，有了这个就可以对不同区域的用户显示不同的结果
- ThemeResolver：用于**解析主题**。SpringMVC中一个主题对应一个properties文件，里面存放着跟当前主题相关的所有资源、如图片、css样式等。
- HandlerMapping：处理映射器，根据需要干的活找到相应的工具
- HandlerAdapter：处理适配器，使用工具干活的人
- HandlerExceptionResolver：处理**异常**解析，根据异常设置ModelAndView
- RequestToViewNameTranslator：请求**视图名称翻译器,**从request中获取ViewName就是RequestToViewNameTranslator要做的事情了
- ViewResolver：视图解析器，String类型的视图名和Locale解析为View类型的视图
- FlashMapManager：用来管理FlashMap的，FlashMap主要用在redirect中传递参数。

最关键的只有 `HandlerMapping + HandlerAdapter + HandlerExceptionResolver 。`

关于每个组件的说明，直接看 [《精尽 Spring MVC 源码分析 —— 组件一览》](http://svip.iocoder.cn/Spring-MVC/Components-intro/) 。

## 描述一下  Spring MVC 的工作流程？

DispatcherServlet 的工作流程可以用一幅图来说明：

[![DispatcherServlet 的工作流程](https://blog-pictures.oss-cn-shanghai.aliyuncs.com/15300766829012.jpg)](https://blog-pictures.oss-cn-shanghai.aliyuncs.com/15300766829012.jpg)（1） 客户端发送请求，请求到达 `DispatcherServlet` 主控制器。
（2） DispatcherServlet 控制器调用 `HandlerMapping` 处理。
（3） HandlerMapping 负责维护请求和 Controller 组件对应关系。 HandlerMapping 根据请求调用对应的 Controller 组件处理。
（4） 执行 Controller 组件的业务处理，需要访问数据库，可以调用 DAO 等组件。
（5）Controller 业务方法处理完毕后，会返回一个 `ModelAndView` 对象。该组件封装了模型数据和视图标识。
（6）Servlet 主控制器调用 `ViewResolver 组件`，根据 ModelAndView 信息处理。定位视图资源，生成视图响应信息。
（7）控制器将响应信息给用户输出。



**但是 Spring MVC 的流程真的一定是酱紫么**？

前后端已经进行分离了，所以 Spring MVC 只负责 **M**odel 和 **C**ontroller 两块，而将 **V**iew 移交给了前端。

那么变成什么样了呢？在步骤 ③ 中，如果 Handler(Controller) 执行完后，如果判断方法有 `@ResponseBody` 注解，则直接将结果JSON写回给用户( 浏览器 )。

但是 HTTP 是不支持返回 Java POJO 对象的，所以需要将结果使用 [HttpMessageConverter](http://svip.iocoder.cn/Spring-MVC/HandlerAdapter-5-HttpMessageConverter/) 进行转换后，才能返回。例如说，大家所熟悉的 [FastJsonHttpMessageConverter](https://github.com/alibaba/fastjson/wiki/在-Spring-中集成-Fastjson) ，将 POJO 转换成 JSON 字符串返回。

## @Controller 注解有什么用？

`@Controller` 注解，**它将一个类标记为 Spring Web MVC 控制器Controller 。**

## @RestController 和 @Controller 有什么区别？

`@RestController` 注解，在 `@Controller` 基础上，增加了 `@ResponseBody` 注解，更加适合目前前后端分离的架构下，提供 Restful API ，返回例如 JSON 数据格式。当然，返回什么样的数据格式，根据客户端的 `"ACCEPT"` 请求头来决定。

## @RequestMapping 注解有什么用？

`@RequestMapping` 注解，用于**将特定 HTTP 请求方法映射到将处理相应请求的控制器中的特定类/方法**。此注释可应用于两个级别：

- 类级别：映射请求的 URL。
- 方法级别：映射 URL 以及 HTTP 请求方法。

## @RequestMapping 和 @GetMapping 注解的不同之处在哪里？

- `@RequestMapping` 可注解在类和方法上；`@GetMapping` 仅可注册在方法上。
- `@RequestMapping` 可进行 GET、POST、PUT、DELETE 等请求方法；`@GetMapping` 是 `@RequestMapping` 的 GET 请求方法的特例，目的是为了提高清晰度。

## 返回 JSON 格式使用什么注解？

可以使用 **`@ResponseBody`** 注解，或者使用包含 `@ResponseBody` 注解的 **`@RestController`** 注解。

当然，还是需要配合相应的支持 JSON 格式化的 HttpMessageConverter 实现类。例如，Spring MVC 默认使用 MappingJackson2HttpMessageConverter 。

## 介绍一下 WebApplicationContext ？

WebApplicationContext 是实现ApplicationContext接口的子类，专门为 WEB 应用准备的。

- 它允许从相对于 Web 根目录的路径中**加载配置文件**，**完成初始化 Spring MVC 组件的工作**。
- **从 WebApplicationContext 中，可以获取 ServletContext 引用，整个 Web 应用上下文对象将作为属性放置在 ServletContext 中，以便 Web 应用环境可以访问 Spring 上下文**。

## （谷粒商城）Spring MVC 的异常处理？

一般情况下，我们使用 `@ExceptionHandler` 注解来实现过异常的处理，

## Spring MVC 有什么优点？

1. 使用真的真的真的非常**方便**，开发简单
2. 提供**拦截器机制**，可以方便的对请求进行拦截处理。
3. 提供**异常机制**，可以方便的对异常做统一处理。

## Spring MVC 怎样设定重定向和转发 ？

- 结果转发：在返回值的前面加 `"forward:/"` 。
- 重定向：在返回值的前面加上 `"redirect:/"` 。

当然，目前前后端分离之后，我们作为后端开发，已经很少有机会用上这个功能了。

## Spring MVC 的 Controller 是不是单例？

绝绝绝大多数情况下，Controller 是**单例**。

那么，Controller 里一般不建议存在**共享的变量**。实际场景下，艿艿也没碰到需要使用共享变量的情况。

## Spring MVC 和 Struts2 的异同？

1. 入口

   不同

   - Spring MVC 的入门是一个 Servlet **控制器**。
   - Struts2 入门是一个 Filter **过滤器**。

2. 配置映射

   不同，

   - Spring MVC 是基于**方法**开发，传递参数是通过**方法形参**，一般设置为**单例**。
   - Struts2 是基于**类**开发，传递参数是通过**类的属性**，只能设计为**多例**。

## 详细介绍下 Spring MVC 拦截器？

`org.springframework.web.servlet.HandlerInterceptor` ，拦截器接口。代码如下：

HandlerInterceptor接口





- 一共有三个方法，分别为：

  - `#preHandle(...)` 方法，调用 Controller 方法之**前**执行。

  - `#postHandle(...)` 方法，调用 Controller 方法之**后**执行。

  - `#afterCompletion(...)`方法，**处理完 Controller 方法返回结果之**

    **后执行**。

    - 例如，页面渲染后。
    - **当然，要注意，无论调用 Controller 方法是否成功，都会执行**。

- 总结来说：

  - `#preHandle(...)` 方法，按拦截器定义**顺序**调用。若任一拦截器返回 `false` ，则 Controller 方法不再调用。
  - `#postHandle(...)` 和 `#afterCompletion(...)` 方法，按拦截器定义**逆序**调用。
  - `#postHandler(...)` 方法，在调用 Controller 方法之**后**执行。
  - `#afterCompletion(...)` 方法，只有该拦截器在 `#preHandle(...)` 方法返回 `true` 时，才能够被调用，且一定会被调用。为什么“且一定会被调用”呢？即使 `#afterCompletion(...)` 方法，按拦截器定义**逆序**调用时，前面的拦截器发生异常，后面的拦截器还能够调用，**即无视异常**。



## Spring MVC 的拦截器可以做哪些事情？

拦截器能做的事情非常非常非常多，例如：

- 记录访问**日志**。
- 记录异常日志。
- **鉴权**，需要登陆的请求操作，拦截未登陆的用户。

## Spring MVC 的拦截器和 Filter 过滤器有什么差别？



- **功能相同**：拦截器和 Filter都能实现相应的功能，谁也不比谁强。
- **容器不同**：拦截器构建在 Spring MVC 体系中；Filter 构建在 Servlet 容器之上。
- **使用便利性不同**：拦截器提供了三个方法，分别在不同的时机执行；**过滤器仅提供一个方法**，当然也能实现拦截器的执行时机的效果，就是麻烦一些。

另外，😈 再补充一点小知识。我们会发现，**拓展性好的框架，都会提供相应的拦截器或过滤器机制**，方便的我们做一些拓展。例如：

- Dubbo 的 Filter 机制。
- Spring Cloud Gateway 的 Filter 机制。
- Struts2 的拦截器机制。

## 什么是安全的 REST 操作?

所以，是否安全的界限，在于**是否修改**服务端的资源。

## REST 是可扩展的或说是协同的吗?



所以这里的“可拓展”、“协同”对应到我们平时常说的，“**跨语言”、“语言无关”**。

## 删除的 HTTP 状态返回码是什么 ?

- 一般来说，如果删除操作成功，响应主体为空，返回 [204](http://www.netingcn.com/http-status-204.html) 。
- 如果删除请求成功且响应体不是空的，则返回 200 。

## REST API 是无状态的吗?

**是的**，REST API 应该是无状态的，因为它是基于 HTTP 的，它也是无状态的。

它**不应该**依赖于以前或下一个请求或服务器端维护的一些数据，例如会话。

## REST安全吗? 你能做什么来保护它?

安全是一个宽泛的术语。它可能意味着消息的安全性，这是通过**认证和授权**提供的加密或访问限制提供的。

REST 通常不是安全的，但是您可以通过使用 `Spring Security` 来保护它。

- 至少，你可以通过在 Spring Security 配置文件中使用 HTTP 来启用 HTTP Basic Auth 基本认证。
- 类似地，如果底层服务器支持 HTTPS ，你可以使用 HTTPS 公开 REST API 。

## @PathVariable 注解，在 Spring MVC 做了什么? 为什么 REST 在 Spring 中如此有用？

`@PathVariable` 注解，是 Spring MVC 中有用的注解之一，**它允许您从 URI 读取值**，@RequestParam注解和@PathVariable注解的区别，从字面上可以看出**前者是获取请求里边携带的参数；后者是获取请求路径里边的变量参数。**

# Spring Boot 

# 核心技术篇

## Spring Boot 是什么？

[Spring Boot](https://github.com/spring-projects/spring-boot) 是 Spring 的**子项目**，开发者可以快速配置 Spring 项目，引入各种 Spring MVC、Spring Transaction、Spring AOP、MyBatis 等等框架，而无需不断重复编写繁重的 Spring 配置，降低了 Spring 的使用成本。

> 艿艿：犹记当年，Spring XML 为主的时代，大晚上各种搜索 Spring 的配置，苦不堪言。现在有了 Spring Boot 之后，生活真美好。

Spring Boot 提供了各种 Starter 启动器，提供标准化的默认配置。例如：

- [`spring-boot-starter-web`](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web/2.1.1.RELEASE) 启动器，可以快速配置 Spring MVC 。
- [`mybatis-spring-boot-starter`](https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter/1.3.2) 启动器，可以快速配置 MyBatis 。

并且，Spring Boot 基本已经一统 Java 项目的开发，大量的开源项目都实现了其的 Starter 启动器。例如：

- [`incubator-dubbo-spring-boot-project`](https://github.com/apache/incubator-dubbo-spring-boot-project) 启动器，可以快速配置 Dubbo 。
- [`rocketmq-spring-boot-starter`](https://github.com/maihaoche/rocketmq-spring-boot-starter) 启动器，可以快速配置 RocketMQ 。

## Spring Boot 提供了哪些核心功能？

- 1、独立运行 Spring 项目

  Spring Boot 可以以 jar 包形式独立运行，运行一个 Spring Boot 项目只需要通过 `java -jar xx.jar` 来运行。

- 2、内嵌 Servlet 容器

  Spring Boot 可以选择内嵌 Tomcat、Jetty 或者 Undertow，这样我们无须以 war 包形式部署项目。

  > 第 2 点是对第 1 点的补充，在 Spring Boot 未出来的时候，大多数 Web 项目，是打包成 war 包，部署到 Tomcat、Jetty 等容器。

- 3、提供 Starter 简化 Maven 配置

  Spring 提供了一系列的 starter pom 来简化 Maven 的依赖加载。例如，当你使用了 `spring-boot-starter-web` ，会自动加入如下依赖：[![`spring-boot-starter-web` 的 pom 文件](http://static2.iocoder.cn/images/Spring/2018-12-26/01.png)](http://static2.iocoder.cn/images/Spring/2018-12-26/01.png)`spring-boot-starter-web` 的 pom 文件

- 4、[自动配置 Spring Bean](https://www.jianshu.com/p/ddb6e32e3faf)

  Spring Boot 检测到特定类的存在，就会针对这个应用做一定的配置，进行自动配置 Bean ，这样会极大地减少我们要使用的配置。

  当然，Spring Boot 只考虑大多数的开发场景，并不是所有的场景，若在实际开发中我们需要配置Bean ，而 Spring Boot 没有提供支持，则可以自定义自动配置进行解决。

- 5、[准生产的应用监控](https://blog.csdn.net/wangshuang1631/article/details/72810412)

  Spring Boot 提供基于 HTTP、JMX、SSH 对运行时的项目进行监控。

- 6、无代码生成和 XML 配置

  Spring Boot 没有引入任何形式的代码生成，它是使用的 Spring 4.0 的条件 `@Condition` 注解以实现根据条件进行配置。同时使用了 Maven /Gradle 的**依赖传递解析机制**来实现 Spring 应用里面的自动配置。

  > 第 6 点是第 3 点的补充。

## Spring Boot 有什么优缺点？

> 艿艿：任何技术栈，有优点必有缺点，没有银弹。
>
> 另外，这个问题的回答，我们是基于 [《Spring Boot浅谈(是什么/能干什么/优点和不足)》](https://blog.csdn.net/fly_zhyu/article/details/76407830) 整理，所以胖友主要看下这篇文章。

**Spring Boot 的优点**

> 艿艿：优点和 [「Spring Boot 提供了哪些核心功能？」](http://svip.iocoder.cn/Spring-Boot/Interview/#) 问题的答案，是比较重叠的。

- 1、使【编码】变简单。
- 2、使【配置】变简单。
- 3、使【部署】变简单。
- 4、使【监控】变简单。

**Spring Boot 的缺点**

> 艿艿：如下的缺点，基于 [《Spring Boot浅谈(是什么/能干什么/优点和不足)》](https://blog.csdn.net/fly_zhyu/article/details/76407830)，考虑的出发点是把 Spring Boot 作为微服务的框架的选型的角度进行考虑。

- 1、没有提供相应的【服务发现和注册】的配套功能。

  > 艿艿：当然，实际上 Spring Boot 本身是不需要提供这样的功能。服务发现和注册的功能，是在 Spring Cloud 中进行提供。

- 2、自身的 acturator 所提供的【监控功能】，也需要与现有的监控对接。

- 3、没有配套的【安全管控】方案。

  > 艿艿：关于这一点，艿艿也有点迷糊，Spring Security 是可以比较方便的集成到 Spring Boot 中，所以不晓得这里的【安全管控】的定义是什么。所以这一点，面试的时候回答，可以暂时先省略。

- 4、对于 REST 的落地，还需要自行结合实际进行 URI 的规范化工作

  > 艿艿：这个严格来说，不算缺点。本身，是规范的范畴。

所以，上面的缺点，严格来说可能不太适合在面试中回答。艿艿认为，Spring Boot 的缺点主要是，因为自动配置 Spring Bean 的功能，我们可能无法知道，哪些 Bean 被进行创建了。这个时候，如果我们想要自定义一些 Bean ，可能存在冲突，或者不知道实际注入的情况。

## Spring Boot、Spring MVC 和 Spring 有什么区别？

Spring 的完整名字，应该是 Spring Framework 。它提供了多个模块，Spring IoC、Spring AOP、Spring MVC 等等。所以，Spring MVC 是 Spring Framework 众多模块中的一个。

而 Spring Boot 是构造在 Spring Framework 之上的 Boot 启动器，旨在更容易的配置一个 Spring 项目。

总结说来，如下图所示：[![Spring Boot 对比 Spring MVC 对比 Spring ？](http://static2.iocoder.cn/images/Spring/2018-12-26/02.png)](http://static2.iocoder.cn/images/Spring/2018-12-26/02.png)Spring Boot 对比 Spring MVC 对比 Spring ？

## Spring Boot 中的 Starter 是什么？

比较**通俗**的说法：

> FROM [《Spring Boot 中 Starter 是什么》](https://www.cnblogs.com/EasonJim/p/7615801.html)
>
> 比如我们要在 Spring Boot 中引入 Web MVC 的支持时，我们通常会引入这个模块 `spring-boot-starter-web` ，而这个模块如果解压包出来会发现里面什么都没有，只定义了一些 **POM** 依赖。如下图所示：[![`spring-boot-starter-web`](http://static2.iocoder.cn/images/Spring/2018-12-26/03.png)](http://static2.iocoder.cn/images/Spring/2018-12-26/03.png)`spring-boot-starter-web`
>
> 经过研究，Starter 主要用来简化依赖用的。比如我们之前做MVC时要引入日志组件，那么需要去找到log4j的版本，然后引入，现在有了Starter之后，直接用这个之后，log4j就自动引入了，也不用关心版本这些问题。

比较**书名**的说法：

> FROM [《Spring Boot Starter 介绍》](http://www.importnew.com/27101.html)
>
> 依赖管理是任何复杂项目的关键部分。以手动的方式来实现依赖管理不太现实，你得花更多时间，同时你在项目的其他重要方面能付出的时间就会变得越少。
>
> Spring Boot Starter 就是为了解决这个问题而诞生的。Starter **POM** 是一组方便的依赖描述符，您可以将其包含在应用程序中。您可以获得所需的所有 Spring 和相关技术的一站式服务，无需通过示例代码搜索和复制粘贴依赖。

## Spring Boot 常用的 Starter 有哪些？

- `spring-boot-starter-web` ：提供 Spring MVC + 内嵌的 Tomcat 。
- `spring-boot-starter-data-jpa` ：提供 Spring JPA + Hibernate 。
- `spring-boot-starter-data-redis` ：提供 Redis 。
- `mybatis-spring-boot-starter` ：提供 MyBatis 。

## 创建一个 Spring Boot Project 的最简单的方法是什么？

Spring Initializr 是创建 Spring Boot Projects 的一个很好的工具。打开 `"https://start.spring.io/"` 网站，我们可以看到 Spring Initializr 工具，如下图所示：

[![Spring Initializr](http://static2.iocoder.cn/images/Spring/2018-12-26/04.png)](http://static2.iocoder.cn/images/Spring/2018-12-26/04.png)Spring Initializr

- 图中的每一个**红线**，都可以填写相应的配置。相信胖友都很熟悉，就不哔哔了。
- 点击生 GenerateProject ，生成 Spring Boot Project 。
- 将项目导入 IDEA ，记得选择现有的 Maven 项目。

------

当然，我们以前使用 IDEA 创建 Spring 项目的方式，也一样能创建 Spring Boot Project 。Spring Initializr 更多的是，提供一个便捷的工具。

## 如何统一引入 Spring Boot 版本？

**目前有两种方式**。

① 方式一：继承 `spring-boot-starter-parent` 项目。配置代码如下：

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.1.RELEASE</version>
</parent>
```

② 方式二：导入 spring-boot-dependencies 项目依赖。配置代码如下：

```
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>1.5.1.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

**如何选择？**

因为一般我们的项目中，都有项目自己的 Maven parent 项目，所以【方式一】显然会存在冲突。所以实际场景下，推荐使用【方式二】。

详细的，推荐阅读 [《Spring Boot 不使用默认的 parent，改用自己的项目的 parent》](https://blog.csdn.net/rainbow702/article/details/55046298) 文章。

另外，在使用 Spring Cloud 的时候，也可以使用这样的方式。

## 运行 Spring Boot 有哪几种方式？

- 1、打包成 Fat Jar ，直接使用 `java -jar` 运行。目前主流的做法，推荐。
- 2、在 IDEA 或 Eclipse 中，直接运行应用的 Spring Boot 启动类的 `#main(String[] args)` 启动。适用于开发调试场景。
- 3、如果是 Web 项目，可以打包成 War 包，使用外部 Tomcat 或 Jetty 等容器。

## 如何打包 Spring Boot 项目？

通过引入 `spring-boot-maven-plugin` 插件，执行 `mvn clean package` 命令，将 Spring Boot 项目打成一个 Fat Jar 。后续，我们就可以直接使用 `java -jar` 运行。

关于 `spring-boot-maven-plugin` 插件，更多详细的可以看看 [《创建可执行 jar》](https://qbgbook.gitbooks.io/spring-boot-reference-guide-zh/II. Getting started/11.5. Creating an executable jar.html) 。

## 如果更改内嵌 Tomcat 的端口？

- 方式一，修改 `application.properties` 配置文件的 `server.port` 属性。

  ```
  server.port=9090
  ```

- 方式二，通过启动命令增加 `server.port` 参数进行修改。

  ```
  java -jar xxx.jar --server.port=9090
  ```

当然，以上的方式，不仅仅适用于 Tomcat ，也适用于 Jetty、Undertow 等服务器。

## 如何重新加载 Spring Boot 上的更改，而无需重新启动服务器？

一共有三种方式，可以实现效果：

- 【推荐】`spring-boot-devtools` 插件。注意，这个工具需要配置 IDEA 的自动编译。

- Spring Loaded 插件。

  > Spring Boot 2.X 后，官方宣布不再支持 Spring Loaded 插件 的更新，所以基本可以无视它了。

- [JRebel](https://www.jianshu.com/p/bab43eaa4e14) 插件，需要付费。

关于如何使用 `spring-boot-devtools` 和 Spring Loaded 插件，胖友可以看看 [《Spring Boot 学习笔记：Spring Boot Developer Tools 与热部署》](https://segmentfault.com/a/1190000014488100) 。

## Spring Boot 的配置文件有哪几种格式？

Spring Boot 目前支持两种格式的配置文件：

- `.properties` 格式。示例如下：

  ```
  server.port = 9090
  ```

- `.yaml` 格式。示例如下：

  ```
  server:
      port: 9090
  ```

------

可能有胖友不了解 **YAML 格式**？

YAML 是一种人类可读的数据序列化语言，它通常用于配置文件。

- 与 Properties 文件相比，如果我们想要在配置文件中添加复杂的属性 YAML 文件就更加**结构化**。从上面的示例，我们可以看出 YAML 具有**分层**配置数据。

- 当然 YAML 在 Spring 会存在一个缺陷，

  `@PropertySource`

   

  注解不支持读取 YAML 配置文件，仅支持 Properties 配置文件。

  - 不过这个问题也不大，可以麻烦一点使用 [`@Value`](https://blog.csdn.net/lafengwnagzi/article/details/74178374) 注解，来读取 YAML 配置项。

实际场景下，艿艿相对比较喜欢使用 Properties 配置文件。个人喜欢~当然，YAML 已经越来越流行了。

## Spring Boot 默认配置文件是什么？

对于 Spring Boot 应用，默认的配置文件根目录下的 **application** 配置文件，当然可以是 Properties 格式，也可以是 YAML 格式。

可能有胖友说，我在网上看到面试题中，说还有一个根目录下的 **bootstrap** 配置文件。这个是 Spring Cloud 新增的启动配置文件，[需要引入 `spring-cloud-context` 依赖后，才会进行加载](https://my.oschina.net/freeskyjs/blog/1843048)。它的特点和用途主要是：

> 参考 [《Spring Cloud 中配置文件名 bootstrap.yml 和 application.yml 区别》](https://my.oschina.net/neverforget/blog/1525947) 文章。

- 【特点】因为 bootstrap 由父 ApplicationContext 加载，比 application 优先加载。
- 【特点】因为 bootstrap 优先于 application 加载，所以不会被它覆盖。
- 【用途】使用配置中心 Spring Cloud Config 时，需要在 bootstrap 中配置配置中心的地址，从而实现父 ApplicationContext 加载时，从配置中心拉取相应的配置到应用中。

另外，[《Appendix A. Common application properties》](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html) 中，有 application 配置文件的通用属性列表。

## Spring Boot 如何定义多套不同环境配置？

可以参考 [《Spring Boot 教程 - Spring Boot Profiles 实现多环境下配置切换》](https://blog.csdn.net/top_code/article/details/78570047) 一文。

但是，需要考虑一个问题，生产环境的配置文件的安全性，显然我们不能且不应该把生产的配置放到项目的 Git 仓库中进行管理。那么应该怎么办呢？

- 方案一，生产环境的配置文件放在生产环境的服务器中，以 `java -jar myproject.jar --spring.config.location=/xxx/yyy/application-prod.properties` 命令，设置 参数 `spring.config.location` 指向配置文件。
- 方案二，使用 Jenkins 在执行打包，配置上 Maven Profile 功能，使用服务器上的配置文件。😈 整体来说，和【方案一】的差异是，将配置文件打包进了 Jar 包中。
- 方案三，使用配置中心。

## Spring Boot 配置加载顺序？

在 Spring Boot 中，除了我们常用的 application 配置文件之外，还有：

- 系统环境变量
- 命令行参数
- 等等…

参考 [《Externalized Configuration》](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html) 文档，我们整理顺序如下：

1. ```
   spring-boot-devtools
   ```

    

   依赖的

    

   ```
   spring-boot-devtools.properties
   ```

    

   配置文件。

   > 这个灰常小众，具体说明可以看看 [《Spring Boot参考文档（12）开发者工具》](https://blog.csdn.net/u011499747/article/details/71746325) ，建议无视。

2. 单元测试上的

    

   ```
   @TestPropertySource
   ```

    

   和

    

   ```
   @SpringBootTest
   ```

    

   注解指定的参数。

   > 前者的优先级高于后者。可以看看 [《Spring、Spring Boot 和TestNG 测试指南 - @TestPropertySource》](https://segmentfault.com/a/1190000010854607) 一文。

3. 命令行指定的参数。例如 `java -jar springboot.jar --server.port=9090` 。

4. 命令行中的 `spring.application.json` 指定参数。例如 `java -Dspring.application.json='{"name":"Java"}' -jar springboot.jar` 。

5. ServletConfig 初始化参数。

6. ServletContext 初始化参数。

7. JNDI 参数。例如 `java:comp/env` 。

8. Java 系统变量，即 `System#getProperties()` 方法对应的。

9. 操作系统环境变量。

10. RandomValuePropertySource 配置的 `random.*` 属性对应的值。

11. Jar **外部**的带指定 profile 的 application 配置文件。例如 `application-{profile}.yaml` 。

12. Jar **内部**的带指定 profile 的 application 配置文件。例如 `application-{profile}.yaml` 。

13. Jar **外部** application 配置文件。例如 `application.yaml` 。

14. Jar **内部** application 配置文件。例如 `application.yaml` 。

15. 在自定义的 `@Configuration` 类中定于的 `@PropertySource` 。

16. 启动的 main 方法中，定义的默认配置。即通过 `SpringApplication#setDefaultProperties(Map defaultProperties)` 方法进行设置。

嘿嘿，是不是很多很长，不用真的去记住。

- 一般来说，面试官不会因为这个题目回答的不好，对你扣分。
- 实际使用时，做下测试即可。
- 每一种配置方式的详细说明，可以看看 [《Spring Boot 参考指南（外部化配置）》](https://segmentfault.com/a/1190000015069140) 。

## Spring Boot 有哪些配置方式？

和 Spring 一样，一共提供了三种方式。

- 1、XML 配置文件。

  Bean 所需的依赖项和服务在 XML 格式的配置文件中指定。这些配置文件通常包含许多 bean 定义和特定于应用程序的配置选项。它们通常以 bean 标签开头。例如：

  ```
  <bean id="studentBean" class="org.edureka.firstSpring.StudentBean">
      <property name="name" value="Edureka"></property>
  </bean>
  ```

- 2、注解配置。

  您可以通过在相关的类，方法或字段声明上使用注解，将 Bean 配置为组件类本身，而不是使用 XML 来描述 Bean 装配。默认情况下，Spring 容器中未打开注解装配。因此，您需要在使用它之前在 Spring 配置文件中启用它。例如：

  ```
  <beans>
  <context:annotation-config/>
  <!-- bean definitions go here -->
  </beans>
  ```

- 3、Java Config 配置。

  Spring 的 Java 配置是通过使用 @Bean 和 @Configuration 来实现。

  - `@Bean` 注解扮演与 `` 元素相同的角色。

  - `@Configuration` 类允许通过简单地调用同一个类中的其他 `@Bean` 方法来定义 Bean 间依赖关系。

  - 例如：

    ```
    @Configuration
    public class StudentConfig {
        
        @Bean
        public StudentBean myStudent() {
            return new StudentBean();
        }
        
    }
    ```

    - 是不是很熟悉 😈

目前主要使用 **Java Config** 配置为主。当然，三种配置方式是可以混合使用的。例如说：

- Dubbo 服务的配置，艿艿喜欢使用 XML 。
- Spring MVC 请求的配置，艿艿喜欢使用 `@RequestMapping` 注解。
- Spring MVC 拦截器的配置，艿艿喜欢 Java Config 配置。

------

另外，现在已经是 Spring Boot 的天下，所以更加是 **Java Config** 配置为主。

## Spring Boot 的核心注解是哪个？

```
package cn.iocoder.skywalking.web01;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Web01Application {

    public static void main(String[] args) {
        SpringApplication.run(Web01Application.class, args);
    }

}
```

- `@SpringBootApplication` 注解，就是 Spring Boot 的核心注解。

`org.springframework.boot.autoconfigure.@SpringBootApplication` 注解的代码如下：

```
// SpringBootApplication.java

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
    @AliasFor(
        annotation = EnableAutoConfiguration.class
    )
    Class<?>[] exclude() default {};

    @AliasFor(
        annotation = EnableAutoConfiguration.class
    )
    String[] excludeName() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackages"
    )
    String[] scanBasePackages() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackageClasses"
    )
    Class<?>[] scanBasePackageClasses() default {};
}
```

- 它组合了 3 个注解，详细说明，胖友看看 [《Spring Boot 系列：@SpringBootApplication 注解》](https://blog.csdn.net/claram/article/details/75125749) 。

- `@Configuration` 注解，指定类是 **Bean 定义**的配置类。

  > `@Configuration` 注解，来自 `spring-context` 项目，用于 Java Config ，不是 Spring Boot 新带来的。

- `#ComponentScan` 注解，扫描指定包下的 Bean 们。

  > `@ComponentScan` 注解，来自 `spring-context` 项目，用于 Java Config ，不是 Spring Boot 新带来的。

- `@EnableAutoConfiguration` 注解，打开自动配置的功能。如果我们想要关闭某个类的自动配置，可以设置注解的 `exclude` 或 `excludeName` 属性。

  > `@EnableAutoConfiguration` 注解，来自 `spring-boot-autoconfigure` 项目，**它才是 Spring Boot 新带来的**。

## 什么是 Spring Boot 自动配置？

在 [「Spring Boot 的核心注解是哪个？」](http://svip.iocoder.cn/Spring-Boot/Interview/#) 中，我们已经看到，使用 `@@EnableAutoConfiguration` 注解，打开 Spring Boot 自动配置的功能。具体如何实现的，可以看看如下两篇文章：

- [《@EnableAutoConfiguration 注解的工作原理》](https://www.jianshu.com/p/464d04c36fb1) 。
- [《一个面试题引起的 Spring Boot 启动解析》](https://juejin.im/post/5b679fbc5188251aad213110)
- 建议，能一边调试，一边看这篇文章。调试很简单，任一搭建一个 Spring Boot 项目即可。

如下是一个比较简单的总结：

1. Spring Boot 在启动时扫描项目所依赖的 jar 包，寻找包含`spring.factories` 文件的 jar 包。
2. 根据 `spring.factories` 配置加载 AutoConfigure 类。
3. 根据 [`@Conditional` 等条件注解](http://svip.iocoder.cn/Spring-Boot/Interview/Spring Boot 条件注解) 的条件，进行自动配置并将 Bean 注入 Spring IoC 中。

## Spring Boot 有哪几种读取配置的方式？

Spring Boot 目前支持 **2** 种读取配置：

1. `@Value` 注解，读取配置到属性。最最最常用。

   > 另外，支持和 `@PropertySource` 注解一起使用，指定使用的配置文件。

2. `@ConfigurationProperties` 注解，读取配置到类上。

   > 另外，支持和 `@PropertySource` 注解一起使用，指定使用的配置文件。

详细的使用方式，可以参考 [《Spring Boot 读取配置的几种方式》](https://aoyouzi.iteye.com/blog/2422837) 。

## 使用 Spring Boot 后，项目结构是怎么样的呢？

我们先来说说项目的分层。一般来说，主流的有两种方式：

- 方式一，`controller`、`service`、`dao` 三个包，每个包下面添加相应的 XXXController、YYYService、ZZZDAO 。
- 方式二，按照业务模块分包，每个包里面放 Controller、Service、DAO 类。例如，业务模块分成 `user`、`order`、`item` 等等包，在 `user` 包里放 UserController、UserService、UserDAO 类。

那么，使用 Spring Boot 的项目怎么分层呢？艿艿自己的想法

- 现在项目都会进行服务化分拆，每个项目不会特别复杂，所以建议使用【方式一】。
- 以前的项目，大多是单体的项目，动则项目几万到几十万的代码，当时多采用【方式二】。

下面是一个简单的 Spring Boot 项目的 Demo ，如下所示：[![Spring Boot 项目的 Demo](http://static2.iocoder.cn/images/Spring/2018-12-26/05.png)](http://static2.iocoder.cn/images/Spring/2018-12-26/05.png)Spring Boot 项目的 Demo

## 如何在 Spring Boot 启动的时候运行一些特殊的代码？

如果需要在 SpringApplication 启动后执行一些特殊的代码，你可以实现 ApplicationRunner 或 CommandLineRunner 接口，这两个接口工作方式相同，都只提供单一的 run 方法，该方法仅在 `SpringApplication#run(...)` 方法**完成之前调用**。

一般情况下，我们不太会使用该功能。如果真需要，胖友可以详细看看 [《使用 ApplicationRunner 或 CommandLineRunner 》](https://qbgbook.gitbooks.io/spring-boot-reference-guide-zh/IV. Spring Boot features/23.8 Using the ApplicationRunner or CommandLineRunner.html) 。

## Spring Boot 2.X 有什么新特性？

1. 起步 JDK 8 和支持 JDK 9
2. 第三方库的升级
3. Reactive Spring
4. HTTP/2 支持
5. 配置属性的绑定
6. Gradle 插件
7. Actuator 改进
8. 数据支持的改进
9. Web 的改进
10. 支持 Quartz 自动配置
11. 测试的改进
12. 其它…

详细的说明，可以看看 [《Spring Boot 2.0系列文章(二)：Spring Boot 2.0 新特性详解》](http://www.54tianzhisheng.cn/2018/03/06/SpringBoot2-new-features) 。

# 整合篇

## 如何将内嵌服务器换成 Jetty ？

默认情况下，`spring-boot-starter-web` 模块使用 Tomcat 作为内嵌的服务器。所以需要去除对 `spring-boot-starter-tomcat` 模块的引用，添加 `spring-boot-starter-jetty` 模块的引用。代码如下：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion> <!-- 去除 Tomcat -->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency> <!-- 引入 Jetty -->
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

## Spring Boot 中的监视器 Actuator 是什么？

`spring-boot-actuator` 提供 Spring Boot 的监视器功能，可帮助我们访问生产环境中正在运行的应用程序的**当前状态**。

- 关于 Spring Boot Actuator 的教程，可以看看 [《Spring Boot Actuator 使用》](https://www.jianshu.com/p/af9738634a21) 。
- 上述教程是基于 Spring Boot 1.X 的版本，如果胖友使用 Spring Boot 2.X 的版本，你将会发现 `/beans` 等 Endpoint 是不存在的，参考 [《Spring boot 2 - Actuator endpoint, where is /beans endpoint》](https://stackoverflow.com/questions/49174700/spring-boot-2-actuator-endpoint-where-is-beans-endpoint) 问题来解决。

**安全性**

Spring Boot 2.X 默认情况下，`spring-boot-actuator` 产生的 Endpoint 是没有安全保护的，但是 Actuator 可能暴露敏感信息。

所以一般的做法是，引入 `spring-boot-start-security` 依赖，使用 Spring Security 对它们进行安全保护。

## 如何集成 Spring Boot 和 Spring MVC ？

1. 引入 `spring-boot-starter-web` 的依赖。

2. 实现 WebMvcConfigurer 接口，可添加自定义的 Spring MVC 配置。

   > 因为 Spring Boot 2 基于 JDK 8 的版本，而 JDK 8 提供 `default` 方法，所以 Spring Boot 2 废弃了 WebMvcConfigurerAdapter 适配类，直接使用 WebMvcConfigurer 即可。

   ```
   // WebMvcConfigurer.java
   public interface WebMvcConfigurer {
   
       /** 配置路径匹配器 **/
       default void configurePathMatch(PathMatchConfigurer configurer) {}
       
       /** 配置内容裁决的一些选项 **/
       default void configureContentNegotiation(ContentNegotiationConfigurer configurer) { }
   
       /** 异步相关的配置 **/
       default void configureAsyncSupport(AsyncSupportConfigurer configurer) { }
   
       default void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) { }
   
       default void addFormatters(FormatterRegistry registry) {
       }
   
       /** 添加拦截器 **/
       default void addInterceptors(InterceptorRegistry registry) { }
   
       /** 静态资源处理 **/
       default void addResourceHandlers(ResourceHandlerRegistry registry) { }
   
       /** 解决跨域问题 **/
       default void addCorsMappings(CorsRegistry registry) { }
   
       default void addViewControllers(ViewControllerRegistry registry) { }
   
       /** 配置视图解析器 **/
       default void configureViewResolvers(ViewResolverRegistry registry) { }
   
       /** 添加参数解析器 **/
       default void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
       }
   
       /** 添加返回值处理器 **/
       default void addReturnValueHandlers(List<HandlerMethodReturnValueHandler> handlers) { }
   
       /** 这里配置视图解析器 **/
       default void configureMessageConverters(List<HttpMessageConverter<?>> converters) { }
   
       /** 配置消息转换器 **/
       default void extendMessageConverters(List<HttpMessageConverter<?>> converters) { }
   
      /** 配置异常处理器 **/
       default void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) { }
   
       default void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) { }
   
       @Nullable
       default Validator getValidator() { return null; }
   
       @Nullable
       default MessageCodesResolver getMessageCodesResolver() {  return null; }
   
   }
   ```

------

在使用 Spring MVC 时，我们一般会做如下几件事情：

1. 实现自己项目需要的拦截器，并在 WebMvcConfigurer 实现类中配置。可参见 [MVCConfiguration](https://github.com/YunaiV/oceans/blob/2a2d3746905f1349e260e88049e7e28346c7648f/bff/webapp-bff/src/main/java/cn/iocoder/oceans/webapp/bff/config/MVCConfiguration.java) 类。
2. 配置 `@ControllerAdvice` + `@ExceptionHandler` 注解，实现全局异常处理。可参见 [GlobalExceptionHandler](https://github.com/YunaiV/oceans/blob/2a2d3746905f1349e260e88049e7e28346c7648f/bff/webapp-bff/src/main/java/cn/iocoder/oceans/webapp/bff/config/GlobalExceptionHandler.java) 类。
3. 配置 `@ControllerAdvice` ，实现 ResponseBodyAdvice 接口，实现全局统一返回。可参见 [GlobalResponseBodyAdvice](https://github.com/YunaiV/oceans/blob/2a2d3746905f1349e260e88049e7e28346c7648f/bff/webapp-bff/src/main/java/cn/iocoder/oceans/webapp/bff/config/GlobalResponseBodyAdvice.java) 。

当然，有一点需要注意，WebMvcConfigurer、ResponseBodyAdvice、`@ControllerAdvice`、`@ExceptionHandler` 接口，都是 Spring MVC 框架自身已经有的东西。

- `spring-boot-starter-web` 的依赖，帮我们解决的是 Spring MVC 的依赖以及相关的 Tomcat 等组件。

## 如何集成 Spring Boot 和 Spring Security ？

目前比较主流的安全框架有两个：

1. Spring Security
2. Apache Shiro

对于任何项目来说，安全认证总是少不了，同样适用于使用 Spring Boot 的项目。相对来说，Spring Security 现在会比 Apache Shiro 更流行。

Spring Boot 和 Spring Security 的配置方式比较简单：

1. 引入 `spring-boot-starter-security` 的依赖。
2. 继承 WebSecurityConfigurerAdapter ，添加**自定义**的安全配置。

当然，每个项目的安全配置是不同的，需要胖友自己选择。更多详细的使用，建议认真阅读如下文章：

- [《Spring Boot中 使用 Spring Security 进行安全控制》](http://blog.didispace.com/springbootsecurity/) ，快速上手。
- [《Spring Security 实现原理与源码解析系统 —— 精品合集》](http://www.iocoder.cn/Spring-Security/good-collection/) ，深入源码。

另外，安全是一个很大的话题，感兴趣的胖友，可以看看 [《Spring Boot 十种安全措施》](https://www.jdon.com/49653) 一文。

## 如何集成 Spring Boot 和 Spring Security OAuth2 ？

参见 [《Spring Security OAuth2 入门》](http://www.iocoder.cn/Spring-Security/OAuth2-learning/) 文章，内容有点多。

## 如何集成 Spring Boot 和 JPA ？

1. 引入 `spring-boot-starter-data-jpa` 的依赖。
2. 在 application 配置文件中，加入 JPA 相关的少量配置。当然，数据库的配置也要添加进去。
3. 具体编码。

详细的使用，胖友可以参考：

- [《一起来学 SpringBoot 2.x | 第六篇：整合 Spring Data JPA》](http://www.iocoder.cn/Spring-Boot/battcn/v2-orm-jpa/)

有两点需要注意：

- Spring Boot 2 默认使用的数据库连接池是 [HikariCP](https://github.com/brettwooldridge/HikariCP) ，目前最好的性能的数据库连接池的实现。
- `spring-boot-starter-data-jpa` 的依赖，使用的默认 JPA 实现是 Hibernate 5.X 。

## 如何集成 Spring Boot 和 MyBatis ？

1. 引入 `mybatis-spring-boot-starter` 的依赖。
2. 在 application 配置文件中，加入 MyBatis 相关的少量配置。当然，数据库的配置也要添加进去。
3. 具体编码。

详细的使用，胖友可以参考：

- [《一起来学 SpringBoot 2.x | 第七篇：整合 Mybatis》](http://www.iocoder.cn/Spring-Boot/battcn/v2-orm-mybatis/)

## 如何集成 Spring Boot 和 RabbitMQ ？

1. 引入 `spring-boot-starter-amqp` 的依赖
2. 在 application 配置文件中，加入 RabbitMQ 相关的少量配置。
3. 具体编码。

详细的使用，胖友可以参考：

- [《一起来学 SpringBoot 2.x | 第十二篇：初探 RabbitMQ 消息队列》](http://www.iocoder.cn/Spring-Boot/battcn/v2-queue-rabbitmq/)
- [《一起来学 SpringBoot 2.x | 第十三篇：RabbitMQ 延迟队列》](http://www.iocoder.cn/Spring-Boot/battcn/v2-queue-rabbitmq-delay/)

## 如何集成 Spring Boot 和 Kafka ？

1. 引入 `spring-kafka` 的依赖。
2. 在 application 配置文件中，加入 Kafka 相关的少量配置。
3. 具体编码。

详细的使用，胖友可以参考：

- [《Spring Boot系列文章（一）：SpringBoot Kafka 整合使用》](http://www.54tianzhisheng.cn/2018/01/05/SpringBoot-Kafka/)

## 如何集成 Spring Boot 和 RocketMQ ？

1. 引入 `rocketmq-spring-boot` 的依赖。
2. 在 application 配置文件中，加入 RocketMQ 相关的少量配置。
3. 具体编码。

详细的使用，胖友可以参考：

- [《我用这种方法在 Spring 中实现消息的发送和消费》](http://www.iocoder.cn/RocketMQ/start/spring-boot-example)

## Spring Boot 支持哪些日志框架？

Spring Boot 支持的日志框架有：

- Logback
- Log4j2
- Log4j
- Java Util Logging

默认使用的是 Logback 日志框架，也是目前较为推荐的，具体配置，可以参见 [《一起来学 SpringBoot 2.x | 第三篇：SpringBoot 日志配置》](http://www.iocoder.cn/Spring-Boot/battcn/v2-config-logs/) 。

因为 Log4j2 的性能更加优秀，也有人在生产上使用，可以参考 [《Spring Boot Log4j2 日志性能之巅》](https://www.jianshu.com/p/f18a9cff351d) 配置。

# 666. 彩蛋

😈 看完之后，复习复习 Spring Boot 美滋滋。有一种奇怪的感觉，把面试题写成了 Spring 的学习指南。

当然，如果胖友有新的面试题，欢迎在星球一起探讨补充。

参考和推荐如下文章：

- 我有面试宝典 [《[经验分享\] Spring Boot面试题总结》](http://www.wityx.com/post/242_1_1.html)
- Java 知音 [《Spring Boot 面试题精华》](https://cloud.tencent.com/developer/article/1348086)
- 祖大帅 [《一个面试题引起的 Spring Boot 启动解析》](https://juejin.im/post/5b679fbc5188251aad213110)
- 大胡子叔叔_ [《Spring Boot + Spring Cloud 相关面试题》](https://blog.csdn.net/panhaigang123/article/details/79587612)
- 墨斗鱼博客 [《20 道 Spring Boot 面试题》](https://www.mudouyu.com/article/26)
- 夕阳雨晴 [《Spring Boot Starter 的面试题》](https://blog.csdn.net/sun1021873926/article/details/78176354)

**文章目录**

1. 核心技术篇
   1. [Spring Boot 是什么？](http://svip.iocoder.cn/Spring-Boot/Interview/#Spring-Boot-是什么？)
   2. [Spring Boot 提供了哪些核心功能？](http://svip.iocoder.cn/Spring-Boot/Interview/#Spring-Boot-提供了哪些核心功能？)
   3. [Spring Boot 有什么优缺点？](http://svip.iocoder.cn/Spring-Boot/Interview/#Spring-Boot-有什么优缺点？)
   4. [Spring Boot、Spring MVC 和 Spring 有什么区别？](http://svip.iocoder.cn/Spring-Boot/Interview/#Spring-Boot、Spring-MVC-和-Spring-有什么区别？)
   5. [Spring Boot 中的 Starter 是什么？](http://svip.iocoder.cn/Spring-Boot/Interview/#Spring-Boot-中的-Starter-是什么？)
   6. [Spring Boot 常用的 Starter 有哪些？](http://svip.iocoder.cn/Spring-Boot/Interview/#Spring-Boot-常用的-Starter-有哪些？)
   7. [创建一个 Spring Boot Project 的最简单的方法是什么？](http://svip.iocoder.cn/Spring-Boot/Interview/#创建一个-Spring-Boot-Project-的最简单的方法是什么？)
   8. [如何统一引入 Spring Boot 版本？](http://svip.iocoder.cn/Spring-Boot/Interview/#如何统一引入-Spring-Boot-版本？)
   9. [运行 Spring Boot 有哪几种方式？](http://svip.iocoder.cn/Spring-Boot/Interview/#运行-Spring-Boot-有哪几种方式？)
   10. [如何打包 Spring Boot 项目？](http://svip.iocoder.cn/Spring-Boot/Interview/#如何打包-Spring-Boot-项目？)
   11. [如果更改内嵌 Tomcat 的端口？](http://svip.iocoder.cn/Spring-Boot/Interview/#如果更改内嵌-Tomcat-的端口？)
   12. [如何重新加载 Spring Boot 上的更改，而无需重新启动服务器？](http://svip.iocoder.cn/Spring-Boot/Interview/#如何重新加载-Spring-Boot-上的更改，而无需重新启动服务器？)
   13. [Spring Boot 的配置文件有哪几种格式？](http://svip.iocoder.cn/Spring-Boot/Interview/#Spring-Boot-的配置文件有哪几种格式？)
   14. [Spring Boot 默认配置文件是什么？](http://svip.iocoder.cn/Spring-Boot/Interview/#Spring-Boot-默认配置文件是什么？)
   15. [Spring Boot 如何定义多套不同环境配置？](http://svip.iocoder.cn/Spring-Boot/Interview/#Spring-Boot-如何定义多套不同环境配置？)
   16. [Spring Boot 配置加载顺序？](http://svip.iocoder.cn/Spring-Boot/Interview/#Spring-Boot-配置加载顺序？)
   17. [Spring Boot 有哪些配置方式？](http://svip.iocoder.cn/Spring-Boot/Interview/#Spring-Boot-有哪些配置方式？)
   18. [Spring Boot 的核心注解是哪个？](http://svip.iocoder.cn/Spring-Boot/Interview/#Spring-Boot-的核心注解是哪个？)
   19. [什么是 Spring Boot 自动配置？](http://svip.iocoder.cn/Spring-Boot/Interview/#什么是-Spring-Boot-自动配置？)
   20. [Spring Boot 有哪几种读取配置的方式？](http://svip.iocoder.cn/Spring-Boot/Interview/#Spring-Boot-有哪几种读取配置的方式？)
   21. [使用 Spring Boot 后，项目结构是怎么样的呢？](http://svip.iocoder.cn/Spring-Boot/Interview/#使用-Spring-Boot-后，项目结构是怎么样的呢？)
   22. [如何在 Spring Boot 启动的时候运行一些特殊的代码？](http://svip.iocoder.cn/Spring-Boot/Interview/#如何在-Spring-Boot-启动的时候运行一些特殊的代码？)
   23. [Spring Boot 2.X 有什么新特性？](http://svip.iocoder.cn/Spring-Boot/Interview/#Spring-Boot-2-X-有什么新特性？)
2. 整合篇
   1. [如何将内嵌服务器换成 Jetty ？](http://svip.iocoder.cn/Spring-Boot/Interview/#如何将内嵌服务器换成-Jetty-？)
   2. [Spring Boot 中的监视器 Actuator 是什么？](http://svip.iocoder.cn/Spring-Boot/Interview/#Spring-Boot-中的监视器-Actuator-是什么？)
   3. [如何集成 Spring Boot 和 Spring MVC ？](http://svip.iocoder.cn/Spring-Boot/Interview/#如何集成-Spring-Boot-和-Spring-MVC-？)
   4. [如何集成 Spring Boot 和 Spring Security ？](http://svip.iocoder.cn/Spring-Boot/Interview/#如何集成-Spring-Boot-和-Spring-Security-？)
   5. [如何集成 Spring Boot 和 Spring Security OAuth2 ？](http://svip.iocoder.cn/Spring-Boot/Interview/#如何集成-Spring-Boot-和-Spring-Security-OAuth2-？)
   6. [如何集成 Spring Boot 和 JPA ？](http://svip.iocoder.cn/Spring-Boot/Interview/#如何集成-Spring-Boot-和-JPA-？)
   7. [如何集成 Spring Boot 和 MyBatis ？](http://svip.iocoder.cn/Spring-Boot/Interview/#如何集成-Spring-Boot-和-MyBatis-？)
   8. [如何集成 Spring Boot 和 RabbitMQ ？](http://svip.iocoder.cn/Spring-Boot/Interview/#如何集成-Spring-Boot-和-RabbitMQ-？)
   9. [如何集成 Spring Boot 和 Kafka ？](http://svip.iocoder.cn/Spring-Boot/Interview/#如何集成-Spring-Boot-和-Kafka-？)
   10. [如何集成 Spring Boot 和 RocketMQ ？](http://svip.iocoder.cn/Spring-Boot/Interview/#如何集成-Spring-Boot-和-RocketMQ-？)
   11. [Spring Boot 支持哪些日志框架？](http://svip.iocoder.cn/Spring-Boot/Interview/#Spring-Boot-支持哪些日志框架？)
3. [666. 彩蛋](http://svip.iocoder.cn/Spring-Boot/Interview/#666-彩蛋)

© 2014 - 2020 芋道源码 | 

总访客数 839469 次 && 总访问量 3972296 次

[回到首页](http://svip.iocoder.cn/index)