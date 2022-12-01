# **实现一个简单的RPC框架**



## [1]什么是RPC？原理是什么？

#### (1-1)什么是RPC

RPC远程过程调用，主要关注远程调用而非本地调用。

为什么要RPC：两个不同服务器上的服务提供的方法不在同一个内存空间，需要通过网络编程传递所需要的参数。方法调用的结果需要网络编程来接受。

RPC能帮助做什么：RPC可以帮助我们调用远程计算机上的某个服务的方法，就像调用本地方法一样简单。并且，不需要了解底层网络编程的具体细节。

#### (1-2)RPC原理

核心由6个部分实现

1.客户端（服务消费端）：调用远程方法的一端。

2.客户端Stub桩：本质上是一个代理类，把调用方法、类、方法参数等信息传到服务端。

3.网络传输：把你调用的方法信息（参数等）传到服务端，

4.服务端Stub桩：不是代理类。服务端 Stub（桩)做的事情就是根据客户端传过来的 RPC 请求去找到对应的目标类和目标方法执行,然后拿回来执行结果.然后再通过网络传输将结果返回给客户端。

5.服务端（服务提供端）：提供远程方法的一端。

![img](https://www.yuque.com/api/filetransfer/images?url=http%3A%2F%2Fmy-blog-to-use.oss-cn-beijing.aliyuncs.com%2F18-12-6%2F37345851.jpg&sign=3e74c7dc34e427a81a470b333ab1a4fc2e33a3f410400c5ff07b69ab9d83bf96)

1.服务消费端：以本地调用方式调用远程服务；

2.客户端Stub：接收到调用后，负责将方法、参数等组装成能够进行网络传输的消息体（序列化） RpcRequest；

3.客户端Stub：找到远程服务地址，并将消息发送到服务提供端；

4.服务端Stub：收到消息，将消息反序列化为Java对象：RpcRequest；

5.服务端Stub：根据RpcRequest中的类、方法、方法参数等信息调用本地方法；

6.服务端Stub：得到方法执行结果并将组装成能够进行网络传输的的消息体（RpcRespons）发送至消费方

7.客户端Stub：收到消息，将消息反序列化为Java对象RpcResponse，得到最终结果



## [2]常见RPC框架介绍

这里说的RPC框架指的是可以让客户端直接调用服务端方法，如：Dubbo、Motan、gRPC。如果和HTTP协议打交道，解析和封装HTTP请求和响应，则不属于RPC框架，如Feign。

(2-1)Dubbo

Apache Dubbo 是一款微服务框架，为大规模微服务实践提供高性能RPC通信、流量治理、可观测性等解决方案。





## [3]如何自己实现一个RPC框架？

RPC框架不仅要提供服务发现功能，还要提供负载均衡、容错等。

最简单RPC实现⬇️：服务提供端Server向注册中心注册服务，服务消费者Client向注册中心注册服务，然后通过网络请求服务提供端Server。

![img](https://www.yuque.com/api/filetransfer/images?url=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Fbbcad419b2e3ecac19d2becb0a764cc7.png&sign=b5bad7c446385617a19d5e3e49bbf5946875eb16b0813d53354575bc7a19cd0d)

Dubbo框架：

<img src="https://www.yuque.com/api/filetransfer/images?url=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Fce5576fd8e7ef8aa2ef1a6f03f99544a.png&sign=0622f648911b3e4324fac5c0e632cc4d5538598376c540a18c22fb03799725d4" alt="img" style="zoom:50%;" />

完整RPC框架：

![img](https://www.yuque.com/api/filetransfer/images?url=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F55d67e70fac5f7f2f97742acb5713fb5.png&sign=05c839b793f0987bc87053a9cc1cf916b30029bf8c8ae65b46567910a191e511)

#### (3-1)最基本的RPC框架需要的组件：

##### (3-1-1)注册中心

使用Zookeeper或者Nacos甚至Redis。

Zookeeper提供了高可用、高性能、稳定的分布式数据一致性解决方案，通常被用于实现诸如数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master选举、分布式锁和分布式队列。并且，Zookeeper将数据保存到内存中，性能优秀，“读”多于“写”的情况下，性能较高。

注册中心负责地址的注册与查找，相当于目录服务。服务端启动的时候将服务名称及其对应地址（ip+port）注册到注册中心，服务消费端根据服务名称找到其对应的服务地址。有了服务地址以后，服务消费端就可以通过网络请求服务端了。



![img](https://www.yuque.com/api/filetransfer/images?url=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Fa3365377759aa5c796153caa314ad147.png&sign=5b8debcec958a701820a677d295822e0a1abe0ee6dc36d9668184954ca289677)

Dubbo框架图结点说明：

​	Provider：暴露服务的服务提供方

​	Consumer：调用远程服务的服务消费方

​	Registry：服务注册与发现的注册中心

​	Monitor：统计服务的调用次数和调用时间的监控中心

​	Container：运行服务的容器

调用关系：

​	1.服务容器负责启动、加载、运行服务的提供者

​	2.服务提供者在启动时，向注册中心注册自己提供的服务

​	3.服务消费者在启动时，向注册中心订阅自己所需的服务

​	4.注册中心返回服务提供者地址列表给服务消费者，如有变更，注册中心将基于长链接推送变更数据给消费者

​	5.服务消费者，从地址列表中，基于软负载均衡算法，选一台提供者进行调用

​	6.服务消费者和提供者，在内存中累计调用次数和调用时间，定时发送数据到中心

##### (3-1-2)网络传输

使用基于NIO的网络编程框架Netty:

​	1.Netty是一个基于NIO的client-server框架，可以快速简单开发网络应用程序

​	2.极大的简化了TCP和UDP套接字服务器等网络编程，性能和安全性更好

​	3.支持多种协议，FTP、SMTP、HTTP等各种二进制和基于文本的传统协议

(3-1-3)序列化和反序列化

网络传输的数据必须是二进制的，需要将Java对象序列化为二进制数据。

![img](https://www.yuque.com/api/filetransfer/images?url=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Fd774fb3f7d6f71d06e700ab237b0d260.png&sign=7e4405f8261241fb21e466ccc42b8dbd16fa885bc48777125d0c374a7ec97ae4)

不推荐使用Java自带序列化，java.io.Serializable接口，其不支持跨语言调用且性能差。常用的序列化方法有：hessian kryo protostuff

##### (3-1-4)动态代理

代理模式：给某一个对象提供一个代理对象，并由代理对象代替真实对象做一些事，安全校验、日志打印等。

RPC主要目的就是调用远程方法像调用本地方法一样，不需要关心远程方法的调用细节。

通过动态代理来屏蔽远程方法的底层细节。

https://javaguide.cn/java/basis/proxy.html#_1-%E4%BB%A3%E7%90%86%E6%A8%A1%E5%BC%8F

##### (3-1-5)负载均衡



##### (3-1-6)传输协议

需要设计一个私有RPC协议，这个协议是客户端和服务端消费的基础。协议中定义：传输数据的类型、每种类型的数据所占字节。

标准的RPC协议包括：

​	· 魔数：通常是4个字节，主要为了筛选来到服务端的数据包，服务端首先取出前4个字节进行比较，以此能够最快辨别出这个数据包是否是遵循自定义协议的，若是无效数据包则可以关闭连接以节省资源。

​	· 序列化编号：标识序列化的方式，比如是使用Java自带的序列化，还是json\kryo等序列化方式

​	· 消息体长度：



#### (3-2)实现RPC的前置知识

##### (3-2-1)Java

​	· 动态代理机制

​	· 序列化机制及序列化框架对比

​	· 线程池

​	· CompletableFuture

##### (3-2-2)Netty

​	· 使用Netty进行网络传输

​	· ByteBuf

​	· Netty粘包拆包

​	· Netty长连接和心跳机制

##### (3-2-3)Zookeeper

​	· 基本概念

​	· 数据结构

#### (3-3)总结

实现一个RPC框架至少需要包括：

​	· 注册中心：负责地址注册与查找，相当于目录服务

​	· 网络传输：通过网络请求传递目标类和参数到服务提供端

​	· 序列化和反序列化： 

​	· 动态代理：屏蔽远程调用的底层细节

​	· 负载均衡：

​	· 传输协议：



## [4]序列化介绍以及序列化协议选择

#### (4-1)什么是序列化和反序列化

序列化：将数据结构或对象转换成二进制字节流

反序列化：二进制字节流转换成数据结构或对象

序列化和反序列化场景：

​	· 网络传输

​	· 存储到文件

​	· 存储到数据库

​	· 存储到内存

序列化的主要目的是将数据存储到数据库、内存、文件系统或通过网络传输。

![img](https://www.yuque.com/api/filetransfer/images?url=https%3A%2F%2Fmy-blog-to-use.oss-cn-beijing.aliyuncs.com%2F2020-8%2Fa478c74d-2c48-40ae-9374-87aacf05188c.png&sign=d94defe4f0e0d2d445d2d061f751199585b252720bc6b2e26fb2010bf7e6c1c7)

序列化协议对应TCP/IP协议中的表示层即4曾模型中的应用层。

#### (4-2)常见序列化协议

##### (4-2-1)Java自带序列化方式

```java
@AllArgsConstructor
@NoArgsConstructor
@Getter
@Builder
@ToString
public class RpcRequest implements Serializable{
  private static final long serialVersionID = 1234567996543123;
  private String requestID;
  private String interfaceName;
  private String methodName;
  private Object[] parameters;
  private Class<?>[] paramTypes;
  private RpcMessageTypeEnum rpcMessageTypeEnum;
}
```



##### (4-2-2)Kryo

高性能序列化/反序列化工具，变长存储特性使用了字节码生成机制，拥有较高的运行速度和较小的字节码体积。

```java
```







## [5]Socket 网络通信实战

<img src="https://www.yuque.com/api/filetransfer/images?url=https%3A%2F%2Fmy-blog-to-use.oss-cn-beijing.aliyuncs.com%2F2020-8%2F6cb857df-c438-4831-a0f2-966344b02213.png&sign=6a50ffca9aa3a137977fd60cd2277bcbb51d269a2b3d0a2617de13ac34237e10" alt="img" style="zoom:50%;" />

#### (5-1)什么是socket套接字

应用程序可以通过它发送或接收数据。套接字socket=IP地址:端口号

要通过互联网进行通信，至少需要一对套接字：

​	服务器端的 Server Socket

​	客户端的 Client Socket

Java开发中使用socket时，常用的类包含在java.net中：	

​	Socket用于客户端

​	ServerSocket用于服务端

#### (5-2)socket网络通信过程



<img src="https://www.yuque.com/api/filetransfer/images?url=https%3A%2F%2Fguide-blog-images.oss-cn-shenzhen.aliyuncs.com%2F2020-8%2F4bbc8291-3819-47e0-9de1-5f6d4c80fed3-20200802201645361.png&sign=90903490f920a47465fb1c67b31c19e7b6d5d58e310c5761cdb69443cc503b3a" alt="img" style="zoom:60%;" />

服务器端：

​	1.创建`ServerSocket`对象并绑定IP地址和端口号，`server.bind(new InetSocketAddress(host, port))`

​	2.通过`accept()`方法监听客户端请求

​	3.连接建立后，通过输入流读取客户端发送的信息

​	4.通过输出流向客户端发送响应信息

​	5.关闭资源

客户端：

​	1.创建`Socket`对象并指定服务器的IP地址和端口号，`socket.connect(inetSocketAddress)`

​	2.建立连接后，通过输出流向服务器发送信息

​	3.通过输入流获得相应信息

​	4.关闭资源

#### (5-3)socket网络通信实战代码

```java
```

`ServerSocket`中的`accept()`方法是阻塞方法，也就是说调用该方法等待客户端连接请求时会阻塞，直到收到客户端发起的连接请求。

需要同时管理多个客户端请求时，需要开启多个线程，

```java

```

<img src="https://www.yuque.com/api/filetransfer/images?url=https%3A%2F%2Fguide-blog-images.oss-cn-shenzhen.aliyuncs.com%2F2020-8%2F7b55fbf1-efb7-4341-b8c3-084ad690c45e.png&sign=b6cc326ebcc2f3e2069b0419227d149bd6ad9b81bda9be92842d011d89b318e7" alt="img" style="zoom:67%;" />

但是这种情况会造成资源浪费，即使使用线程池也会浪费资源。因为底层是同步阻塞的BIO模型。

改进方法：使用Java1.4之后提供NIO，同步非阻塞IO模型。使用基于NIO的网络编程框架Neety。

客户端：

```java

```

发送的消息实体：

```java
```



先启动服务端，再运行客户端，观察控制台输出：



## [6]Netty从入门到网络通信实战

#### (6-1)Netty简介

​	1.Netty是一个基于NIO的cs架构，使用它可以快速开发网络应用程序；

​	2.极大的简化了UDP和TCP套接字服务器等网络编程，性能和安全性更好；

​	3.支持多种协议，FTP、SMTP、HTTP以及各种二进制和基于文本的传统协议

#### (6-2)Netty特点

​	·统一的API，支持多种传输类型，阻塞或非阻塞的

​	·简单而强大的线程模型

​	·自带编码器解决TCP粘包/拆包问题

​	·真正的无连接数据包套接字支持

​	·比直接使用Java核心API拥有更高的吞吐量、更低的延迟、更低的资源消耗、更少的内存复制

​	·安全性较好，有完整的SSL/TLS以及StartTLS支持

​	·社区活跃

​	·成熟稳定，许多大型开源项目都用到了Netty，如：Dubbo\RocketMQ

#### (6-3)Netty应用

NIO可以做的事情，Netty都可以做，并且可以做的更好，主要用来做网络通信

·**作为RPC框架的网络通信工具**

·**实现一个自己的HTTP** 

·**实现一个即时通讯系统** 

·**消息推送系统** 

#### (6-4)使用Netty的开源项目



#### (6-5)Netty使用kryo序列化传输对象实战代码

kryo不支持没有无参构造的对象进行反序列化，如果希望某个对象被kryo进行序列化操作，需要有相应的无参构造。

##### (6-5-1)**传输实体类**

客户端将`RpcRequest`类型的对象发送到服务端，服务端处理后，将`RpcResponse`对象返回给客户端。

客户端请求：



服务端响应：



##### (6-5-2)**客户端**

初始化客户端：客户端中有一个用于向服务端发送消息的sendMessage()方法，通过这个方法可以将消息，也就是RpcRequest对象发送到服务端，并且可以同步获得返回结果RpcRespose.

```java
```

sendMessage()分析：

​	1.首先初始化了一个Bootstrap

​	2.通过Bootstrap连接到服务端

​	3.通过Channel向服务端发送RpcRequest

​	4.发送成功后，阻塞等待，直到Channel关闭

​	5.拿到服务端返回到结果RpcResponse

**自定义 ChannelHandler 处理服务端消息**

```java
```

`NettyClientHandler`用于读取服务端发送过来的`RpcResponse`对象，并将其保存到`AttributeMap`，`AttributeMap`是可以看作是`Channel`的共享数据源。

通过channel和key将数据读取出来

```java
```

AttributeMap 类似于Map 

```java
```



##### (6-5-3)**服务端**

初始化服务端：`NettyServer`主要作用就是开启了一个服务端用于接收客户端的请求并处理

```java
```

自定义ChannelHandler处理客户端消息

```java
```

##### (6-5-4)**编码器**

自定义编码器：

​	`NettyKryoEncoder`是自定义编码器，负责处理“出站”消息，将消息格式转化为字节数组，写入容器`ByteBuf`中

```java
```

自定义解码器：

​	`NettyKryoDecoder`是自定义解码器，负责处理“入站”消息，从`ByteBuf`中读取到业务对象对应的字节序列，然后将字节序列转换为业务对象

```java
```

自定义序列化接口

​	`Serializer` 接口中的两个方法，一个用于序列化，一个用于反序列化

```java
```

实现序列化接口

​	自定义的kryo序列化实现类

```java

```

​	自定义序列化异常类

```java
```



##### (6-5-5)功能测试





## [7]静态代理+JDK/CGLIB 动态代理实战

#### (7-1)代理模式

代理模式的主要作用是扩展目标对象的功能，

#### (7-2)静态代理



#### (7-3)动态代理

​	动态代理不需要对每个目标类都单独创建一个代理类，并且也不需要我们必须实现接口，我们可以直接使用代理类(CHLIB代理机制)

​	从JVM角度来说，动态代理是在运行时动态生成类字节码，并加载到JVM中。

​	Java实现动态代理的方式有JDK动态代理、CGLIB代理。本次使用JDK动态代理。



##### (7-3-1)JDK动态代理

Java动态代理中`InvocationHandler`和`Proxy`是核心

​	`Proxy`中使用频率最高的是`newProxyInstance()`

```java
```

方法共有3个参数：

​	1.loader：类加载器，用于加载代理对象

​	2.interfaces：被代理类实现的一些接口

​	3.h：实现了InvocationHandler的对象

要实现动态代理，还必须实现`InvocationHandler`来自定义处理逻辑，动态代理对象调用一个方法时，这个方法的调用就会被转到`InvocationHandler`的`invoke`方法

```java
```

`invoke()`方法有三个参数：

​	1.proxy：动态生成的代理类

​	2.method：与代理类对象调用的方法相对应

​	3.args：当前方法的参数

通过`Proxy`中的`newProxyInstance()`创建的代理对象在调用方法的时候，实际会调用到实现`InvocationHandler`接口的类的`invoke()`方法。

JDK动态代理类使用步骤：

​	1.定义一个接口及实现类

​	2.自定义`InvocationHandler`并重写`invoke()`，在`invoke()`我们调用原生方法并自定义一些处理逻辑

​	3.通过`Proxy.newProxyInstance()`创建代理对象



代码部分





##### (7-3-2)CGLIB动态代理



#### (7-4)静态代理与动态代理对比







## [8]ZooKeeper常用命令+ Curator使用详解



## [9]RPC 框架代码分析之网络传输模块



## [10]RPC 框架代码分析之注册中心模块



## [11]RPC 框架代码分析之其他模块

## [12]（优化）使用CompletableFuture优化接受服务提供端返回结果

