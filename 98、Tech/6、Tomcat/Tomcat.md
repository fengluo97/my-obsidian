# Tomcat
Tomcat 是一款轻量级的开源 Web 服务器，其包含了 HTTP 服务器 + Servlet 容器。

一个 Tomcat 实例有一个 Server，一个 Server 可以有多个 Service，一个 Service 可以包含多个 Connector 和 一个 Container。一般单个 Service 即可向外提供 Web 服务。

## 架构设计
### HTTP 服务器
HTTP（请求行、请求头、请求体）。
在 Coyote 包下，由 Connector 连接器组件实现，负责接收请求和返回响应，监听端口接收外界请求，并将请求传递给容器进行业务处理，最后将结果返回给客户端。
1、监听网络端口，接收和响应网络请求。
2、网络字节流处理。将二进制流解析为 Request 对象再转换为标准的 ServletRequest 对象给容器，同时将容器传来的 ServletResponse 对象转成 Response 再转成网络字节流返回客户端。
在一个 Service 中可以有多个 Connector 来适用不同的协议的IO请求。
#### EndPoint
端点，处理 Socket 接收和发送的逻辑，其内部由 Acceptor 监听请求、Handler 处理数据、AsyncTimeout 检查请求超时。并提供字节流给 Processor。 具体实现有 NioEndPoint、AprEndpoint 等。
#### Processor
处理器，负责解析 HTTP 请求字节流，构建 Tomcat Request，具体实现有 Http11Processor等。
#### Adaptor
适配器，负责将 Request 对象与 ServletRequest 对象、Response 对象与 ServletResponse 对象之间的转换。
#### ProtocolHandler
协议处理器，将不同的协议和通讯方式组合封装成对应的协议处理器，如 Http11NioProtocol 封装的是 Http1.1 和 NIO。也就是说，ProtocolHandler 包含了 EndPoint 和 Processor。

### Servlet 容器
在 catalina 包下，由 Container 容器组件实现，包含了四层容器。负责管理、调用、执行业务逻辑。四层容器的划分体现了单一职责的设计原则。
#### Engine
一个 Service 最多有一个容器引擎，可指定默认 Host，可包含多个 Host，目前很少用多个 Host。它的 Vavle 阀门处理所有到达当前 tomcat 的请求，StandardEngineVavle 阀门负责找到 Host 中的第一个 Vavle 阀门。
#### Host
虚拟主机，通过域名区分，可包含多个 Context。它的 Vavle 阀门处理所有同一个域名的请求，StandardHostVavle 阀门负责找到 Context 中的第一个 Vavle 阀门。
#### Context
应用级上下文，context-path 即应用顶级路径名，可包含多个 Wrapper。它的 Vavle 阀门处理所有同一个应用的请求，StandardContextVavle 阀门负责找到 Wrapper 中的第一个 Vavle 阀门。
#### Wrapper
所有请求共用一个 Servlet 实例，若 Servlet 有状态则线程不安全。包装了同一个 Servlet 类型的所有实例。它的 Vavle 阀门处理所有同一个 Servlet 的请求，StandardWrapperVavle 阀门负责找到对应的 Servlet 实例，在 FilterChain 过滤链处理之后，调用执行 service 方法。
#### 容器请求处理过程
每一层容器都有一个 pipeline 管理流水线，每个 pipeline 内部都有一个 StandardValve，相当于一个个阀门来处理所有的 Request 和 Response，也就是责任链模式。

## 执行流程
### 启动流程
Tomcat 设计了一个 Lifecycle 接口，各个组件实现这个接口，来执行对应生命周期的操作，包含了 init、start、stop、destroy 方法。
启动流程
![[Pasted image 20240109235850.png]]
1） 启动tomcat ， 需要调用 bin/startup.bat (在linux 目录下 , 需要调用 bin/startup.sh) ， 在startup.bat 脚本中, 调用了catalina.bat。
2） 在catalina.bat 脚本文件中，调用了BootStrap 中的main方法。
3）在BootStrap 的main 方法中调用了 init 方法 ， 来创建Catalina 及 初始化类加载器。
4）在BootStrap 的main 方法中调用了 load 方法 ， 在其中又调用了Catalina的load方法。
5）在Catalina 的load 方法中 , 需要进行一些初始化的工作, 并需要构造Digester 对象, 用于解析 XML。
6） 然后在调用后续组件的初始化操作 。。。

加载Tomcat的配置文件，初始化容器组件 ，监听对应的端口号， 准备接受客户端请求 。
也就是进行各组件逐级执行 init() 和 start() 方法。

### 执行流程
当一个请求进入 Tomcat 时，执行情况如下( 一般情况下一个 Tomcat 只有一个 Service）
![[Pasted image 20240110000251.png]]
查找 Servlet 类主要通过 Mapper 组件来实现，其本质上就是 K、V 键值对。在解析时首先会将请求网址进行解析，将其中的 Host 部分在 Mapper 类中的 hosts属性（MappedHost数组，保存所有的 Host 信息）中进行查找，找到后再解析 Context 部分，在该 MapperHost 中又有 contextList 属性（保存所有的 context 信息），然后再向下找，最终得到对应的 Servlet，执行。
![[Pasted image 20240110000417.png]]
1）Connector组件Endpoint中的Acceptor监听客户端套接字连接并接收Socket。
2）将连接交给线程池Executor处理，开始执行请求响应任务。
3）Processor组件读取消息报文，解析请求行、请求头、请求体，封装成Request对象。
4）CoyoteAdaptor组件负责将Connector组件和Engine容器关联起来，把生成的Request对象和响应对象Response转换成标准的ServletRequset 与 ServletResponse对象再传递到Engine容器中，调用 Pipeline。
5）Mapper组件根据请求行的URL值和请求头的Host值匹配由哪个Host容器、Context容器、Wrapper容器处理请求。
6）Engine容器的管道开始处理，管道中包含若干个Valve、每个Valve负责部分处理逻辑。执行完Valve后会执行基础的 Valve--StandardEngineValve，负责调用Host容器的Pipeline。
7）Host容器的管道开始处理，流程类似，最后执行 Context容器的Pipeline。
8）Context容器的管道开始处理，流程类似，最后执行 Wrapper容器的Pipeline。
9）Wrapper容器的管道开始处理，流程类似，在调用完 FilterChain 过滤器链之后，执行对应的Servlet对象的处理方法。

### Server.xml
核心配置文件，包含了 Connector 和 Servlet 容器的相关配置，Executor 配置。

## 网络模型


## Servlet
Servlet 生命周期
![[Pasted image 20240110003638.png]]
1、请求到达 server 端，server 根据 url 映射到相应的 Servlet
2、判断 Servlet 实例是否存在，不存在则加载和实例化 Servlet 并调用 init 方法
3、Server 分别创建 Request 和 Response 对象，调用 Servlet 实例的 service 方法（service 方法内部会根据 http 请求方法类型调用相应的 doXXX 方法）
4、doXXX 方法内为业务逻辑实现，从 Request 对象获取请求参数，处理完毕之后将结果通过 response 对象返回给调用方
5、当 Server 不再需要 Servlet 时（一般当 Server 关闭时），Server 调用 Servlet 的 destroy() 方法。

## 设计模式
Tomcat 在架构设计上相当精巧，利用了很多设计模式以及基本原则。
在整体的架构上，
### 责任链模式

### 模板方法模式

### 观察者模式

### 装饰器模式

### 组合模式

## 破坏双亲委派机制

## 热加载与热部署

## 调优
### JVM 层面
因为 Tomcat 是一个 Java 应用，所以可以考虑 JVM 的调优。
增大堆内存，一般可以指定物理内存的80%，在配置文件中加入配置信息。
### Tomcat 连接器
连接器的配置其实是 Tomcat 性能的关键。
优化线程池，增加线程数。
在connector中设置enableLookups="false"取消DNS检查。
设置compression=”on”进行压缩。
把useURIValidationHack设成”false”，减少对一些url的不必要的检查从而减省开销。

## 最佳实践
[Socket 核心原理分享 - ITDragon龙 - 博客园 (cnblogs.com)](https://www.cnblogs.com/itdragon/p/13700939.html)

参考文章
[Tomcat基础知识总结 | Java学习&面试指南-程序员大彬 (topjavaer.cn)](https://topjavaer.cn/web/tomcat.html#%E6%9E%B6%E6%9E%84)
[Tomcat组成与工作原理 - 掘金 (juejin.cn)](https://juejin.cn/post/6844903473482317837)
[阿里二面：你知道 Tomcat 的工作原理么？ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/379107635)
[「面试必背」Tomcat面试题（收藏）-CSDN博客](https://blog.csdn.net/q66562636/article/details/124455010)[Tomcat面试题（10道含答案），由浅入深！ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/673679599)