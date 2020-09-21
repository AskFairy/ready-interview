### Dubbo框架里的微服务组件

#### 服务发布与引用

Dubbo 框架主要是使用 XML 配置方式

##### 1.服务端发布过程

```xml

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        http://dubbo.apache.org/schema/dubbo        http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
 
    <!-- 提供方应用信息，用于计算依赖关系 -->
    <dubbo:application name="hello-world-app"  />
 
    <!-- 使用multicast广播注册中心暴露服务地址 -->
    <dubbo:registry address="multicast://224.5.6.7:1234" />
 
    <!-- 用dubbo协议在20880端口暴露服务 -->
    <dubbo:protocol name="dubbo" port="20880" />
 
    <!-- 声明需要暴露的服务接口 -->
    <dubbo:service interface="com.alibaba.dubbo.demo.DemoService" ref="demoService" />
 
    <!-- 和本地bean一样实现服务 -->
    <bean id="demoService" class="com.alibaba.dubbo.demo.provider.DemoServiceImpl" />
</beans>
```

- “dubbo:service”开头的配置项声明了服务提供者要发布的接口，

- “dubbo:protocol”开头的配置项声明了服务提供者要发布的接口的协议以及端口号

Dubbo 会把以上配置项解析成下面的 URL 格式：

`dubbo://host-ip:20880/com.alibaba.dubbo.demo.DemoService`

然后基于[扩展点自适应机制](http://dubbo.incubator.apache.org/zh-cn/docs/dev/SPI.html)，通过 URL 的“`dubbo://`”协议头识别，就会调用 `DubboProtocol` 的 `export()` 方法，打开服务端口 20880，就可以把服务 demoService 暴露到 20880 端口了。

- `扩展点自适应机制:`

##### 2.服务引用

服务消费者的 XML 配置

```java

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        http://dubbo.apache.org/schema/dubbo        http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
 
    <!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
    <dubbo:application name="consumer-of-helloworld-app"  />
 
    <!-- 使用multicast广播注册中心暴露发现服务地址 -->
    <dubbo:registry address="multicast://224.5.6.7:1234" />
 
    <!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
    <dubbo:reference id="demoService" interface="com.alibaba.dubbo.demo.DemoService" />
</beans>
```

其中“dubbo:reference”开头的配置项声明了服务消费者要引用的服务，Dubbo 会把以上配置项解析成下面的 URL 格式：

`dubbo://com.alibaba.dubbo.demo.DemoService`

然后基于`扩展点自适应机制`，通过 URL 的“`dubbo://`”协议头识别，就会调用 `DubboProtocol` 的 `refer()` 方法，得到服务 demoService 引用，完成服务引用过程。

#### 服务注册与发现

##### 服务提供者注册服务的过程

继续以前面服务提供者的 XML 配置为例，其中“dubbo://registry”开头的配置项声明了注册中心的地址，Dubbo 会把以上配置项解析成下面的 URL 格式：

```java
registry://multicast://224.5.6.7:1234/com.alibaba.dubbo.registry.RegistryService?export=URL.encode("dubbo://host-ip:20880/com.alibaba.dubbo.demo.DemoService")
```

然后基于扩展点自适应机制，通过 URL 的“`registry://`”协议头识别，就会调用 `RegistryProtocol` 的 `export()` 方法，将 export 参数中的提供者 URL，注册到注册中心。

##### 服务消费者发现服务的过程

同样以前面服务消费者的 XML 配置为例，其中“`dubbo://registry`”开头的配置项声明了注册中心的地址，跟服务注册的原理类似，Dubbo 也会把以上配置项解析成下面的 URL 格式：

```java
registry://multicast://224.5.6.7:1234/com.alibaba.dubbo.registry.RegistryService?refer=URL.encode("consummer://host-ip/com.alibaba.dubbo.demo.DemoService")
```

然后基于扩展点自适应机制，通过 URL 的“`registry://`”协议头识别，就会调用 `RegistryProtocol` 的 `refer()` 方法，基于 refer 参数中的条件，查询服务 demoService 的地址。