## Dubbo

### 系统架构的演变

![](G:\学习笔记和资料\图片\1590413686(1).jpg)

#### ORM架构

​     单一应用架构：当网站流量很小时，只需一个应用，将所有功能都部署在一起，以减少部署节点和成本。

#### MVC架构

​      垂直应用架构：当访问量逐渐增大，将应用拆成互不相干的几个应用，以提升效率。

#### RPC服务架构

​     分布式服务架构： 当垂直应用越来越多，应用之间交互不可避免，将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心。服务之间互相调用

#### SOA架构

​     SOA解决多服务互相调用凌乱的问题，同时SOA又有一个名字，叫做服务治理。多个子系统直接相互交互，相互调用非常凌乱，于是就用到了我们的SOA架构帮助我们把服务之间调用的乱七八糟的关系给治理起来，然后提供一个统一的标准，把我们的服务治理成下图所示，以前我们的服务是互相交互，现在是只对数据总线进行交互，这样系统就变得统一起来。 Dubbo就是SOA服务治理方案的核心框架

##### 数据总线

​       每个服务需要根据约定向数据总线注册服务 ，数据总线里面一个key对应一个value，key指的是服务名，value则是服务的调度方式。数据总线还有一些高级应用，比如心跳检测，实现负载均衡。

### RPC 

​      就是指远程过程调用，是一种进程间的通信方式。，RPC就是从一台机器(客户端)上通过参数传递的方式调用另一台机器(服务器)上的一个函数或方法(可以统称为服务)并得到返回的结果。是程序员不必编写调用的细节。

#### RPC原理

![](G:\学习笔记和资料\图片\1590455547(1).jpg)

#### RPC调用流程

![image-20200526092644615](C:\Users\hava_a_good_time\AppData\Roaming\Typora\typora-user-images\image-20200526092644615.png)

1. 客户端调用
2. ClientStub 序列化，找到服务地址，发送消息
3. ServerStub 反序列化，调用本地服务
4. server处理服务，返回结果
5. serversStub序列化结果，发送消息
6. ClientStub反序列化，返回调用结果给客户端
7. 客户端得到返回结果 

Rpc框架的作用就是封装2-6步骤。决定RPC框架的性能取决于通信效率和序列化和反序列化机制，常见的RPC框架有Dubbo,gRpc

### Dubbo 介绍

​      基于 Java 的高性能的 RPC 分布式服务框架。适合于小数据量大并发的服务调用。层是使用Netty这样的NIO框架异步非阻塞通信框架，是基于TCP协议传输的，配合以Hession序列化完成RPC通信。

### Dubbo架构

![image-20200527090652604](C:\Users\hava_a_good_time\AppData\Roaming\Typora\typora-user-images\image-20200527090652604.png)

### Dubbo调用流程

1. 服务容器负责启动，加载，运行服务提供者。
2. 服务提供者在启动时，解析配置文件后将服务注册到注册中心
3. 服务消费者在启动时，向注册中心订阅自己所需的服务
4. 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
5. 服务消费者向注册中心获取服务提供者地址列表，并根据负载算法直接调用提供者。
6. 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心
**注意**：注册中心，服务提供者，服务消费者三者之间均为长连接，监控中心除外。注册中心通过长连接感知服务提供者的存在，服务提供者宕机，注册中心将立即推送事件通知消费者。注册中心和监控中心全部宕机，不影响已运行的提供者和消费者，消费者在本地缓存了提供者列表。注册中心和监控中心都是可选的，服务消费者可以直连服务提供者。

直连示例

```java
 @Reference(url = "127.0.0.1:20881")
 private UserService userService;
```

### Dubbo 核心的配置

| 标签                 | 用途         | 解释                                                         |
| -------------------- | ------------ | ------------------------------------------------------------ |
| <dubbo:service/>     | 服务配置     | 暴露一个服务，一个服务可以用多个协议暴露，也可以注册到多个注册中心 |
| <dubbo:protocol/>    | 协议配置     | 用于配置提供服务的协议信息                                   |
| <dubbo:application/> | 应用配置     | 用于配置当前应用信息，不管该应用是提供者还是消费者           |
| <dubbo:registry/>    | 注册中心配置 | 用于配置连接注册中心相关信息                                 |
| <dubbo:provider/>    | 提供方配置   |                                                              |
| <dubbo:consumer/>    | 消费方配置   |                                                              |
| <dubbo:monitor/>     | 监控中心配置 |                                                              |

#### Dubbo配置案列-生产者

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        http://dubbo.apache.org/schema/dubbo        http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <!-- 提供方应用信息，用于计算依赖关系 -->
    <dubbo:application name="provider"/>

    <!-- 使用multicast广播注册中心暴露服务地址 -->
    <!--<dubbo:registry address="multicast://224.5.6.7:1234"/>-->

    <!-- 使用Zookeeper注册服务地址 -->
    <dubbo:registry address="zookeeper://127.0.0.1:2181"/>

    <!--指定通信规则dubbo。 用dubbo协议在20880端口暴露服务 -->
    <dubbo:protocol name="dubbo" port="20880"/>

    <!-- 和本地bean一样实现服务 -new-->
    <bean id="userServiceImpl" class="com.hww.service.impl.UserServiceImpl"/>
    <!-- 声明需要暴露的服务接口 -->
    <dubbo:service interface="com.hww.service.UserService" ref="userServiceImpl" version="1.0.0" />

    <!--统一配置服务提供者-->
    <dubbo:registry check="true"/>

    <dubbo:provider timeout="5000"/>
   <!--监控中心配置-->
    <dubbo:monitor address="127.0.0.1:7070"/>
</beans>
```



####    Dubbo配置案列-消费者

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd
		http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd ">
    <!--配置优先级：
     1.方法>接口>全局配置
     2.级别一样的话消费方>提供方
    -->
    <dubbo:application name="consumer"/>
  
    <!--注册中心地址-->
    <dubbo:registry address="zookeeper://127.0.0.1:2181"/>
   
    <!--启动时检查注册中心是否存在-->
    <dubbo:registry check="false"/>

    <!--接口配置：
    check="false" :启动不检查依赖的服务是否存在
    version="1.0.0" 指定版本号 
     -->
    <dubbo:reference id="userService" interface="com.hww.service.UserService" check="true" version="1.0.0" timeout="2000">
        <!--方法配置 幂等方法可以添加重试-->
        <dubbo:method name="getUserAddressList" timeout="1000" retries="2"/>
    </dubbo:reference>

    <!--全局统一配置消费者
    check="false" -依赖的服务启动不检查 ， 	缺省值：true。
    timeout="3000"-远程服务调用超时时间(毫秒) ，缺省值：1000。
    retries="2"-远程服务调用重试次数，不包括第一次调用，不需要重试请设为0 ,缺省值=2
    -->
        
    <dubbo:consumer check="false" timeout="3000" retries="2"/>
    <!--监控中心配置-->
    <dubbo:monitor address="127.0.0.1:7070"/>
</beans>

```



### Dubbo集群容错(服务提供者集群)

![](G:\学习笔记和资料\图片\1590416737(1).jpg)

#### 集群工作过程

1. 服务消费者初始化期间，集群 Cluster 实现类为服务消费者创建 Cluster Invoker 实例
2. 服务消费者进行远程调用时。 Cluster Invoker 首先会调用 Directory 的 list 方法列举 Invoker 列表。Directory 的用途是保存 Invoker，可简单类比为 List<Invoker>。其实现类 RegistryDirectory 是一个动态服务目录，可感知注册中心配置的变化，它所持有的 Invoker 列表会随着注册中心内容的变化而变化。
3. 调用 Router 的 route 方法进行路由，过滤掉不符合路由规则的 Invoker。
4. 通过 LoadBalance 从 Invoker 列表中选择一个 Invoker。
5. 最后 Cluster Invoker 会将参数传给 LoadBalance 选择出的 Invoker 实例的 invoke 方法，进行真正的远程调用。

#### 容错方式

| Failover Cluster | 失败自动切换，自动重试其他服务器(默认) |
| ---------------- | -------------------------------------- |
| Failfast Cluster | 快速失败，立即报错，只发起一次调用     |
| Failsafe Cluster | 失败安全，出现失败，自动忽略           |

### 启动时检查

​       Dubbo 缺省会在启动时检查依赖的服务是否可用，不可用时会抛出异常，阻止 Spring 初始化完成，默认 check="true"，可以通过 check="false" 关闭检查。消费者在启动时，会去注册中心拉取服务列表，缓存在本地，后面即使注册中心挂了，也不会影响正常使用。

```java
 <dubbo:consumer check="false" timeout="3000" retries="2" ></dubbo:consumer>
```

### 一个服务接口多种实现

​       当一个接口有多种实现时，可以用 group 属性来分组，服务提供方和消费方都指定同一个 group 即可

```java
<dubbo:reference id="feedbackIndexService" group="feedback" interface="com.xxx.IndexService" />
<dubbo:reference id="memberIndexService" group="member" interface="com.xxx.IndexService" />
```

### Dubbo服务多版本

​      可以用版本号（version），一个服务多个不同版本注册到注册中心，版本号不同的服务相互间不引用。这个和服务分组的概念有一点类似。

  服务提供者配置：

```
<dubbo:service interface="com.foo.BarService" version="1.0.0" />
```
服务消费者配置：

```
<dubbo:reference id="barService" interface="com.foo.BarService" version="1.0.0" />
```

### Dubbo对结果进行缓存

​       Dubbo 提供了声明式缓存，用于加速热门数据的访问速度，以减少用户加缓存的工作量

#### 缓存类型

1.  lru 基于最近最少使用原则删除多余缓存，保持最热的数据被缓存。
2. `threadlocal` 当前线程缓存，比如一个页面渲染，用到很多 portal，每个 portal 都要去查用户信息，通过线程缓存，可以减少这种多余访问。

```
<dubbo:reference interface="com.foo.BarService" cache="lru" />
```

### Dubbo有哪些注册中心

1. Zookeeper注册中心：采用Zookeeper的watch机制实现服务的变更；
2. redis注册中心： 采用key/Map存储，key存储服务名，Map中key存储服务URL，value服务过期时间。基于redis的发布/订阅模式通知数据变更。
3. Multicast注册中心：广播地址，进行服务注册和发现

### Dubbo支持的通讯协议
| dubbo://     | 缺省协议，采用单一长连接和 NIO 异步通讯，适合于小数据量大并发的服务调用，使用TCP协议 |
| ------------ | ------------------------------------------------------------ |
| http://      | 基于 HTTP 表单的远程调用协议，短连接，多连接                 |
| redis://     | 基于 Redis实现的 RPC 协议                                    |
| memcached:// | 基于 memcached 实现的 RPC 协议                               |

​     Dubbo 允许配置多协议，在不同服务上支持不同协议或者同一服务上同时支持多种协议。

#### 不同服务不同协议

​      不同服务在性能上适用不同协议进行传输，比如大数据用短连接协议，小数据大并发用长连接协议

```java
 <dubbo:application name="world"  />
 <dubbo:registry id="registry" address="10.20.141.150:9090" username="admin" password="hello1234" />
    <!-- 多协议配置 -->
    <dubbo:protocol name="dubbo" port="20880" />
    <dubbo:protocol name="rmi" port="1099" />
    <!-- 使用dubbo协议暴露服务 -->
    <dubbo:service interface="com.hww.Service" version="1.0.0" ref="helloService" protocol="dubbo" />
    <!-- 使用rmi协议暴露服务 -->
    <dubbo:service interface="com.hww.Service" version="1.0.0" ref="demoService" protocol="rmi"/> 
</beans>
```

#### 多协议暴露服务

​      需要与 http 客户端互操作

```java

  <dubbo:application name="world"  />
  <dubbo:registry id="registry" address="10.20.141.150:9090" username="admin" password="hello1234" />
    <!-- 多协议配置 -->
    <dubbo:protocol name="dubbo" port="20880" />
    <dubbo:protocol name="hessian" port="8080" />
    <!-- 使用多个协议暴露服务 -->
    <dubbo:service id="helloService" interface="com.hww.Service" version="1.0.0" protocol="dubbo,hessian" />
</beans>
```



### Dubbo的负载均衡策略

| 策略                       | 解释                                                         |
| -------------------------- | ------------------------------------------------------------ |
| Random LoadBalance         | 按权重设置随机概率                                           |
| RoundRobin LoadBalance     | 轮循，按公约后的权重设置轮循比率                             |
| LeastActive LoadBalance    | 最少活跃调用数，相同活跃数的随机，活跃数指调用前后计数差，使慢的提供者收到更少请求 |
| ConsistentHash LoadBalance | 一致性Hash，相同参数的请求总是发到同一提供者                 |

配置示列

生产者：

```
<dubbo:service interface="..." loadbalance="roundrobin" />
```

消费者：

```
<dubbo:reference interface="..." loadbalance="roundrobin" />
```

### Consumer异步调用

​    基于 NIO 的非阻塞实现并行调用，客户端不需要启动多线程即可完成并行调用多个远程服务，相对多线程开销较小。

#### CompletableFuture实现异步调用

需要服务提供者事先定义CompletableFuture签名的服务

```
public interface AsyncService {
    CompletableFuture<String> sayHello(String name);
}
```

生产者配置

```java
<dubbo:reference id="asyncService" timeout="10000" interface="com.alibaba.dubbo.samples.async.api.AsyncService"/>
```

消费者调用远程服务代码

```java
// 调用直接返回CompletableFuture
CompletableFuture<String> future = asyncService.sayHello("async call request");
// 增加回调
future.whenComplete((v, t) -> {
    if (t != null) {
        t.printStackTrace();
    } else {
        System.out.println("Response: " + v);
    }
});
// 早于结果输出
System.out.println("Executed before response return.");
```

#### 使用RpcContext

消费者配置

```java
<dubbo:reference id="asyncService" interface="org.apache.dubbo.samples.governance.api.AsyncService">
      <dubbo:method name="sayHello" async="true" />
</dubbo:reference>
```

消费者调用远程服务代码

```java
// 此调用会立即返回null
asyncService.sayHello("world");
// 拿到调用的Future引用，当结果返回后，会被通知和设置到此Future
CompletableFuture<String> helloFuture = RpcContext.getContext().getCompletableFuture();
// 为Future添加回调
helloFuture.whenComplete((retValue, exception) -> {
    if (exception == null) {
        System.out.println(retValue);
    } else {
        exception.printStackTrace();
    }
});
```

### Provider异步执行

Provider端异步执行和Consumer端异步调用是相互独立的，可以任意正交组合两端配置

- Consumer同步 - Provider同步
- Consumer异步 - Provider同步
- Consumer同步 - Provider异步
- Consumer异步 - Provider异步

#### CompletableFuture签名的接口实现异步执行

服务接口定义：

```
public interface AsyncService {
    CompletableFuture<String> sayHello(String name);
}
```

服务的实现

```java
public class AsyncServiceImpl implements AsyncService {
    @Override
    public CompletableFuture<String> sayHello(String name) {
        RpcContext savedContext = RpcContext.getContext();
        // supplyAsync提供自定义线程池，避免使用JDK公用线程池
        return CompletableFuture.supplyAsync(() -> {
            System.out.println(savedContext.getAttachment("consumer-key1"));
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "async response from provider.";
        });
    }
}
```

​     通过`return CompletableFuture.supplyAsync()`，业务执行已从Dubbo线程切换到业务线程，避免了对Dubbo线程池的阻塞。

#### AsyncContext实现异步执行

 服务接口定义

```java
public interface AsyncService {
    String sayHello(String name);
}
```

服务暴露，和普通服务完全一致：

```java
<bean id="asyncService" class="org.apache.dubbo.samples.governance.impl.AsyncServiceImpl"/>
<dubbo:service interface="org.apache.dubbo.samples.governance.api.AsyncService" ref="asyncService"/>
```

服务实现：

```java
public class AsyncServiceImpl implements AsyncService {
    public String sayHello(String name) {
        final AsyncContext asyncContext = RpcContext.startAsync();
        new Thread(() -> {
            // 如果要使用上下文，则必须要放在第一句执行
            asyncContext.signalContextSwitch();
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            // 写回响应
            asyncContext.write("Hello " + name + ", response from provider.");
        }).start();
        return null;
    }
}
```

### Dubbo原理

#### Dubbo暴露服务过程和标签解析

​      ioc容器启动时，spring解析配置文件里的标签，DubboNamespaceHandler(Dubbo名称空间处理器)，调用init方法根据不同的标签创建DubboBeanDefinitionParser。 spring解析配置文件里的标签，有一个总接口BeanDefinitionParser。DubboBeanDefinitionParser 实现了 Spring 的 BeanDefinitionParser，通过重写 parse() 方法实现将标签解析为对应的 JavaBean进行保存， 额外注意的是对于<dubbo-service></dubbo-service>标签，会解析为对应的ServiceBean,ServiceBean的解析涉及到服务暴露的过程。因为ServiceBean 实现了InitializingBean重写了afterPropertiesSet()方法，实现了ApplicationListener<ContextRefreshedEvent>(应用监听器)j监听ioc容器加载完成事件，重写了onApplicationEvent()方法。

afterPropertiesSet()方法主要是保存<dubbo:provider>标签信息。也就是服务提供者的全局配置信息

onApplicationEvent()方法主要是调用export()方法暴露服务，先获取执行器，然后使用Protocol有DubboProtocol 和RegistryProtocol DubboProtocol 主要是启动netty监听端口，RegistryProtocol主要是将服务地址注册进zookeeper中