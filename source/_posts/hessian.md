title: hessian 的简单介绍
date: 2014-01-15 10:11:09
tags:
- 协议
categories: 基础
---

### 一、什么是Hessian
>Hessian 是一个基于 binary-RPC 实现的远程通讯 library。使用二进制传输数据。Hessian通常通过Web应用来提供服务，通过接口暴露。Servlet和Spring的DispatcherServlet都可以把请求转发给Hessian服务。由以下两种方式提供，分别为：com.caucho.hessian.server.HessianServlet、org.springframework.web.servlet.DispatcherServlet。
 
关于hessian的7个问题：

1. 是基于什么协议实现的？
	
	基于Binary-RPC协议实现。

2. 怎么发起请求？
    
    需通过Hessian本身提供的API来发起请求。

3. 怎么 将请求转化为符合协议的格式的？
 <!-- more -->   
    Hessian通过其自定义的串行化机制将请求信息进行序列化，产生二进制流。
4. 使用什么 传输协议传输？
    
    Hessian基于Http协议进行传输。
5. 响应端基于什么机制来接收请求？
   
    响应端根据Hessian提供的API来接收请求。
6.怎么将流还原为传输格式的？
   
    Hessian根据其私有的串行化机制来将请求信息进行反序列化，传递给使用者时已是相应的请求信息对象了。
7. 处理完毕后怎么回应？
   
   处理完毕后直接返回，hessian将结果对象进行序列化，传输至调用端。
### 二、hessian的优缺点

**优点：**

* 简单易用，面向接口，通过接口暴露服务，jar包只有200、300k，不需要配置防火墙
* 效率高，复杂对象序列化速度仅次于RMI，简单对象序列化优于RMI，二进制传输
* 多语言支持：wiki、Java、Flash/Flex、Python、C++、.NET C#、PHP、Ruby、Objective-C
* 可与spring集成，配置简单，HessianServiceExporte
 
**缺点：**

* 缺乏安全机制，传输没有加密处理
* 异常机制不完善，总是报一些错误，错误原因也是千奇百怪，提示信息不足
* 事务处理欠缺
* 版本问题，spring 2.5.6对照3.1.3版，spring 3对照4.0及以上版本，需要使用spring MVC
 
### 三、各个通讯协议对比：
通讯效率测试结果:

RMI > Httpinvoker >= Hessian >> Burlap >> Web service
       
1. RMI 是 Java 首选远程调用协议，非常高效稳定，特别是在数据结构复杂，数据量大的情况下，与其他通讯协议的差距尤为明显。但不能跨语言。
2. HttpInvoker 使用 java 的序列化技术传输对象，与 RMI 在本质上是一致的。从效率上看，两者也相差无几， HttpInvoker 与 RMI 的传输时间基本持平。
3. Hessian 在传输少量对象时，比 RMI 还要快速高效，但传输数据结构复杂的对象或大量数据对象时，较 RMI 要慢 20% 左右。但这只是在数据量特别大，
数据结构很复杂的情况下才能体现出来，中等或少量数据时， Hessian并不比RMI慢。 Hessian 的好处是精简高效，可以跨语言使用，而且协议规范公开，
我们可以针对任意语言开发对其协议的实现。另外， Hessian与WEB服务器结合非常好，借助WEB服务器的成熟功能，在处理大量用户并发访问时会有很大优势，在资源分配，
线程排队，异常处理等方面都可以由成熟的WEB服务器保证。而 RMI 本身并不提供多线程的服务器。而且，RMI 需要开防火墙端口， Hessian 不用。
4. Burlap 采用 xml 格式传输。仅在传输 1 条数据时速度尚可，通常情况下，它的毫时是 RMI 的 3 倍。
5. Web Service 的效率低下是众所周知的，平均来看， Web Service 的通讯毫时是 RMI 的 10 倍。
 
### 四、基本应用流程
**客户端必须具备以下几点：**

* java客户端包含Hessian.jar的包。
* 具有和服务器端结构一样的接口。
* 利用HessianProxyFactory调用远程接口。
* 使用spring方式需要配置HessianProxyFactoryBean
* 注意：使用resin容器时，resin已经包含了hessian.jar包
JAVA服务器端必须具备以下几点：
* 包含Hessian的jar包。
* 设计一个接口，用来给客户端调用。
* 实现该接口的功能。
* 配置web.xml，配好相应的servlet。
* 对象必须实现Serializable 接口。
* 对于spring方式DispatcherServlet拦截url,HessianServiceExporter提供Bean服务
 
### 五、几种Remoting实现的比较
Spring支持的Remoting实现技术是非常多的，虽然Spring屏蔽了这些技术使用上的差异，但是选择一个合适的Remoting技术仍然对系统有非常积极的作用，下面就来讲述这些实现技术的优缺点。

1. RMI：
   RMI使用Java的序列化机制实现调用及返回值的编组(marshal)与反编组(unmarshal)，可以使用任何可序列化的对象作为参数和返回值。其缺点是RMI只能通过RMI协议来进行访问，无法通过HTTP协议访问，无法穿透防火墙。
2. Hessian：
   Hessian也是将网络传输的对象转换为二进制流通过Http进行传递，不过它是使用自己的序列化机制实现的编组与反编组，其支持的数据类型是有限制的，不支持复杂的对象。Hessian的优点是可以透过防火墙。
3. Burlap：
Burlap是将网络传输的对象转换为XML文本格式通过Http进行传递，支持的对象与Hessian相比更少。XML一般比二进制流占 用空间大，在网络上传递所需要的时间比二进制流长，XML的解析过程也会耗用更多的内存。Burlap可以穿透防火墙，而且由于传输的格式是XML文本， 可以与其他系统(比如.NET)集成，从某种程度来讲，Burlap是一种不标准的WebService。
4. HttpInvoker：
  HttpInvoker将参数和返回值通过Java的序列化机制进行编组和反编组，它具有RMI的支持所有可序列化对象的优点。 Http Invoker是使用Http协议传输二进制流的，而同时又具有Hessian、Burlap的优点。