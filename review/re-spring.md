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

# REST

## 资源是什么?

资源是指数据在 REST 架构中如何显示的。将实体作为资源公开 ，它允许客户端通过 HTTP 方法如：[GET](http://javarevisited.blogspot.sg/2012/03/get-post-method-in-http-and-https.html), [POST](http://www.java67.com/2014/08/difference-between-post-and-get-request.html),[PUT](http://www.java67.com/2016/09/when-to-use-put-or-post-in-restful-web-services.html), DELETE 等读，写，修改和创建资源。

## 什么是安全的 REST 操作?

REST 接口是通过 HTTP 方法完成操作。

- 一些HTTP操作是安全的，如 GET 和 HEAD ，它不能在服务端修改资源
- 换句话说，PUT,POST 和 DELETE 是不安全的，因为他们能修改服务端的资源。

所以，是否安全的界限，在于**是否修改**服务端的资源。

## 什么是幂等操作? 为什么幂等操作如此重要?

有一些HTTP方法，如：GET，不管你使用多少次它都能产生相同的结果，在没有任何一边影响的情况下，发送多个 GET 请求到相同的[URI](http://www.java67.com/2013/01/difference-between-url-uri-and-urn.html) 将会产生相同的响应结果。因此，这就是所谓**幂等**操作。

换句话说，[POST方法不是幂等操作](http://javarevisited.blogspot.sg/2016/05/what-are-idempotent-and-safe-methods-of-HTTP-and-REST.html) ，因为如果发送多个 POST 请求，它将在服务端创建不同的资源。但是，假如你用PUT更新资源，它将是幂等操作。

甚至多个 PUT 请求被用来更新服务端资源，将得到相同的结果。你可以通过 Pluralsight 学习 [HTTP Fundamentals](http://pluralsight.pxf.io/c/1193463/424552/7490?u=https%3A%2F%2Fwww.pluralsight.com%2Fcourses%2Fxhttp-fund) 课程来了解 HTTP 协议和一般的 HTTP 的更多幂等操作。

## REST 是可扩展的或说是协同的吗?

是的，[REST](http://javarevisited.blogspot.sg/2015/08/difference-between-soap-and-restfull-webservice-java.html) 是可扩展的和可协作的。它既不托管一种特定的技术选择，也不定在客户端或者服务端。你可以用 [Java](http://javarevisited.blogspot.sg/2017/11/top-5-free-java-courses-for-beginners.html), [C++](http://www.java67.com/2018/02/5-free-cpp-courses-to-learn-programming.html), [Python](http://www.java67.com/2018/02/5-free-python-online-courses-for-beginners.html), 或 [JavaScript](http://www.java67.com/2018/04/top-5-free-javascript-courses-to-learn.html) 来创建 RESTful Web 服务，也可以在客户端使用它们。

我建议你读一本关于REST接口的书来了解更多，如：[RESTful Web Services](http://javarevisited.blogspot.sg/2017/02/top-5-books-to-learn-rest-and-restful-web-services-in-java.html) 。

> 艿艿：所以这里的“可拓展”、“协同”对应到我们平时常说的，“跨语言”、“语言无关”。

## REST 用哪种 HTTP 方法呢?

REST 能用任何的 HTTP 方法，但是，最受欢迎的是：

- 用 GET 来检索服务端资源
- 用 POST 来创建服务端资源
- [用 PUT 来更新服务端资源](http://javarevisited.blogspot.sg/2016/04/what-is-purpose-of-http-request-types-in-RESTful-web-service.html#axzz56WGunSwy)
- 用 DELETE 来删除服务端资源。

恰好，这四个操作，对上我们日常逻辑的 CRUD 操作。

> 艿艿：经常能听到胖友抱怨自己做的都是 CRUD 的功能。看了这个面试题，有没觉得原来 CRUD 也能玩的稍微高级一点？！
>
> 实际上，每个 CRUD 也是可以通过不断的打磨，玩的很高级。例如说 DDD 领域驱动，完整的单元测试，可扩展的设计。

## 删除的 HTTP 状态返回码是什么 ?

在删除成功之后，您的 REST API 应该返回什么状态代码，并没有严格的规则。它可以返回 200 或 204 没有内容。

- 一般来说，如果删除操作成功，响应主体为空，返回 [204](http://www.netingcn.com/http-status-204.html) 。
- 如果删除请求成功且响应体不是空的，则返回 200 。

## REST API 是无状态的吗?

**是的**，REST API 应该是无状态的，因为它是基于 HTTP 的，它也是无状态的。

REST API 中的请求应该包含处理它所需的所有细节。它**不应该**依赖于以前或下一个请求或服务器端维护的一些数据，例如会话。

**REST 规范为使其无状态设置了一个约束，在设计 REST API 时，您应该记住这一点**。

## REST安全吗? 你能做什么来保护它?

安全是一个宽泛的术语。它可能意味着消息的安全性，这是通过认证和授权提供的加密或访问限制提供的。

REST 通常不是安全的，但是您可以通过使用 Spring Security 来保护它。

- 至少，你可以通过在 Spring Security 配置文件中使用 HTTP 来启用 HTTP Basic Auth 基本认证。
- 类似地，如果底层服务器支持 HTTPS ，你可以使用 HTTPS 公开 REST API 。

## RestTemplate 的优势是什么?

在 Spring Framework 中，RestTemplate 类是 [模板方法模式](http://www.java67.com/2012/09/top-10-java-design-pattern-interview-question-answer.html) 的实现。跟其他主流的模板类相似，如 JdbcTemplate 或 JmsTempalte ，它将在客户端简化跟 RESTful Web 服务的集成。正如在 RestTemplate 例子中显示的一样，你能非常容易地用它来调用 RESTful Web 服务。

> 艿艿：当然，实际场景我还是更喜欢使用 [OkHttp](http://square.github.io/okhttp/) 作为 HTTP 库，因为更好的性能，使用也便捷，并且无需依赖 Spring 库。

## HttpMessageConverter 在 Spring REST 中代表什么?

HttpMessageConverter 是一种[策略接口](http://www.java67.com/2014/12/strategy-pattern-in-java-with-example.html) ，它指定了一个转换器，它可以转换 HTTP 请求和响应。Spring REST 用这个接口转换 HTTP 响应到多种格式，例如：JSON 或 XML 。

每个 HttpMessageConverter 实现都有一种或几种相关联的MIME协议。Spring 使用 `"Accept"` 的标头来确定客户端所期待的内容类型。

然后，它将尝试找到一个注册的 HTTPMessageConverter ，它能够处理特定的内容类型，并使用它将响应转换成这种格式，然后再将其发送给客户端。

如果胖友对 HttpMessageConverter 不了解，可以看看 [《Spring 中 HttpMessageConverter 详解》](https://leokongwq.github.io/2017/06/14/spring-MessageConverter.html) 。

## 如何创建 HttpMessageConverter 的自定义实现来支持一种新的请求/响应？

我们仅需要创建自定义的 AbstractHttpMessageConverter 的实现，并使用 WebMvcConfigurerAdapter 的 `#extendMessageConverters(List> converters)` 方法注中册它，该方法可以生成一种新的请求 / 响应类型。

具体的示例，可以学习 [《在 Spring 中集成 Fastjson》](https://github.com/alibaba/fastjson/wiki/在-Spring-中集成-Fastjson) 文章。

## @PathVariable 注解，在 Spring MVC 做了什么? 为什么 REST 在 Spring 中如此有用？

`@PathVariable` 注解，是 Spring MVC 中有用的注解之一，它允许您从 URI 读取值，比如查询参数。它在使用 Spring 创建 RESTful Web 服务时特别有用，因为在 REST 中，资源标识符是 URI 的一部分。

具体的使用示例，胖友如果不熟悉，可以看看 [《Spring MVC 的 @RequestParam 注解和 @PathVariable 注解的区别》](https://blog.csdn.net/cx361006796/article/details/52829759) 。

# 666. 彩蛋

文末的文末，艿艿还是那句话！！！！还是非常推荐胖友去撸一撸 Spring MVC 的源码，特别是如下两篇：

- [《精尽 Spring MVC 源码分析 —— 组件一览》](http://svip.iocoder.cn/Spring-MVC/Components-intro/)
- [《精尽 Spring MVC 源码分析 —— 请求处理一览》](http://svip.iocoder.cn/Spring-MVC/DispatcherServlet-process-request-intro/)

参考和推荐如下文章：

- [《排名前 20 的 REST 和 Spring MVC 面试题》](http://www.spring4all.com/article/1445)
- [《跟着 Github 学习 Restful HTTP API 的优雅设计》](http://www.iocoder.cn/Fight/Learn-Restful-HTTP-API-design-from-Github/)

**文章目录**

1. Spring MVC
   1. [Spring MVC 框架有什么用？](http://svip.iocoder.cn/Spring-MVC/Interview/#Spring-MVC-框架有什么用？)
   2. [介绍下 Spring MVC 的核心组件？](http://svip.iocoder.cn/Spring-MVC/Interview/#介绍下-Spring-MVC-的核心组件？)
   3. [描述一下 DispatcherServlet 的工作流程？](http://svip.iocoder.cn/Spring-MVC/Interview/#描述一下-DispatcherServlet-的工作流程？)
   4. [@Controller 注解有什么用？](http://svip.iocoder.cn/Spring-MVC/Interview/#Controller-注解有什么用？)
   5. [@RestController 和 @Controller 有什么区别？](http://svip.iocoder.cn/Spring-MVC/Interview/#RestController-和-Controller-有什么区别？)
   6. [@RequestMapping 注解有什么用？](http://svip.iocoder.cn/Spring-MVC/Interview/#RequestMapping-注解有什么用？)
   7. [@RequestMapping 和 @GetMapping 注解的不同之处在哪里？](http://svip.iocoder.cn/Spring-MVC/Interview/#RequestMapping-和-GetMapping-注解的不同之处在哪里？)
   8. [返回 JSON 格式使用什么注解？](http://svip.iocoder.cn/Spring-MVC/Interview/#返回-JSON-格式使用什么注解？)
   9. [介绍一下 WebApplicationContext ？](http://svip.iocoder.cn/Spring-MVC/Interview/#介绍一下-WebApplicationContext-？)
   10. [Spring MVC 的异常处理？](http://svip.iocoder.cn/Spring-MVC/Interview/#Spring-MVC-的异常处理？)
   11. [Spring MVC 有什么优点？](http://svip.iocoder.cn/Spring-MVC/Interview/#Spring-MVC-有什么优点？)
   12. [Spring MVC 怎样设定重定向和转发 ？](http://svip.iocoder.cn/Spring-MVC/Interview/#Spring-MVC-怎样设定重定向和转发-？)
   13. [Spring MVC 的 Controller 是不是单例？](http://svip.iocoder.cn/Spring-MVC/Interview/#Spring-MVC-的-Controller-是不是单例？)
   14. [Spring MVC 和 Struts2 的异同？](http://svip.iocoder.cn/Spring-MVC/Interview/#Spring-MVC-和-Struts2-的异同？)
   15. [详细介绍下 Spring MVC 拦截器？](http://svip.iocoder.cn/Spring-MVC/Interview/#详细介绍下-Spring-MVC-拦截器？)
   16. [Spring MVC 的拦截器可以做哪些事情？](http://svip.iocoder.cn/Spring-MVC/Interview/#Spring-MVC-的拦截器可以做哪些事情？)
   17. [Spring MVC 的拦截器和 Filter 过滤器有什么差别？](http://svip.iocoder.cn/Spring-MVC/Interview/#Spring-MVC-的拦截器和-Filter-过滤器有什么差别？)
2. REST
   1. [REST 代表着什么?](http://svip.iocoder.cn/Spring-MVC/Interview/#REST-代表着什么)
   2. [资源是什么?](http://svip.iocoder.cn/Spring-MVC/Interview/#资源是什么)
   3. [什么是安全的 REST 操作?](http://svip.iocoder.cn/Spring-MVC/Interview/#什么是安全的-REST-操作)
   4. [什么是幂等操作? 为什么幂等操作如此重要?](http://svip.iocoder.cn/Spring-MVC/Interview/#什么是幂等操作-为什么幂等操作如此重要)
   5. [REST 是可扩展的或说是协同的吗?](http://svip.iocoder.cn/Spring-MVC/Interview/#REST-是可扩展的或说是协同的吗)
   6. [REST 用哪种 HTTP 方法呢?](http://svip.iocoder.cn/Spring-MVC/Interview/#REST-用哪种-HTTP-方法呢)
   7. [删除的 HTTP 状态返回码是什么 ?](http://svip.iocoder.cn/Spring-MVC/Interview/#删除的-HTTP-状态返回码是什么)
   8. [REST API 是无状态的吗?](http://svip.iocoder.cn/Spring-MVC/Interview/#REST-API-是无状态的吗)
   9. [REST安全吗? 你能做什么来保护它?](http://svip.iocoder.cn/Spring-MVC/Interview/#REST安全吗-你能做什么来保护它)
   10. [RestTemplate 的优势是什么?](http://svip.iocoder.cn/Spring-MVC/Interview/#RestTemplate-的优势是什么)
   11. [HttpMessageConverter 在 Spring REST 中代表什么?](http://svip.iocoder.cn/Spring-MVC/Interview/#HttpMessageConverter-在-Spring-REST-中代表什么)
   12. [如何创建 HttpMessageConverter 的自定义实现来支持一种新的请求/响应？](http://svip.iocoder.cn/Spring-MVC/Interview/#如何创建-HttpMessageConverter-的自定义实现来支持一种新的请求-响应？)
   13. [@PathVariable 注解，在 Spring MVC 做了什么? 为什么 REST 在 Spring 中如此有用？](http://svip.iocoder.cn/Spring-MVC/Interview/#PathVariable-注解，在-Spring-MVC-做了什么-为什么-REST-在-Spring-中如此有用？)
3. [666. 彩蛋](http://svip.iocoder.cn/Spring-MVC/Interview/#666-彩蛋)

© 2014 - 2020 芋道源码 | 

总访客数 839198 次 && 总访问量 3971993 次

[回到首页](http://svip.iocoder.cn/index)