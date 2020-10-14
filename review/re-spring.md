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

### （重点）Spring MVC 的工作流程

- （1） 客户端发送请求，请求到达 DispatcherServlet 主控制器。
  （2） DispatcherServlet 控制器调用 HandlerMapping 处理。
  （3） HandlerMapping 负责维护请求和 Controller 组件对应关系。 HandlerMapping 根据请求调用对应的 Controller 组件处理。
  （4） 执行 Controller 组件的业务处理，需要访问数据库，可以调用 DAO 等组件。
  （5）Controller 业务方法处理完毕后，会返回一个 ModelAndView 对象。该组件封装了模型数据和视图标识。
  （6）Servlet 主控制器调用 ViewResolver 组件，根据 ModelAndView 信息处理。定位视图资源，生成视图响应信息。
  （7）控制器将响应信息给用户输出。

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

　　①拦截器是基于java的反射机制的，而过滤器是基于函数回调。
　　②拦截器不依赖与servlet容器，过滤器依赖与servlet容器。**

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