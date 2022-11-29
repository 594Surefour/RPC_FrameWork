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

(3-1-1)注册中心

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

(3-1-2)网络传输

使用基于NIO的网络编程框架Netty:

​	1.Netty是一个基于NIO的client-server框架，可以快速简单开发网络应用程序

​	2.极大的简化了TCP和UDP套接字服务器等网络编程，性能和安全性更好

​	3.支持多种协议，FTP、SMTP、HTTP等各种二进制和基于文本的传统协议

(3-1-3)序列化和反序列化

网络传输的数据必须是二进制的，需要将Java对象序列化为二进制数据。

![img](https://www.yuque.com/api/filetransfer/images?url=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Fd774fb3f7d6f71d06e700ab237b0d260.png&sign=7e4405f8261241fb21e466ccc42b8dbd16fa885bc48777125d0c374a7ec97ae4)

不推荐使用Java自带序列化，java.io.Serializable接口，其不支持跨语言调用且性能差。常用的序列化方法有：hessian kryo protostuff

(3-1-4)动态代理



(3-1-5)负载均衡

(3-1-6)传输协议



#### (3-2)实现RPC的前置知识







## [4]序列化介绍以及序列化协议选择ibay8y

## [5]Socket 网络通信实战



## [6]Netty从入门到网络通信实战



## [7]静态代理+JDK/CGLIB 动态代理实战



## [8]ZooKeeper常用命令+ Curator使用详解



## [9]RPC 框架代码分析之网络传输模块



## [10]RPC 框架代码分析之注册中心模块



## [11]RPC 框架代码分析之其他模块

## [12]（优化）使用CompletableFuture优化接受服务提供端返回结果

