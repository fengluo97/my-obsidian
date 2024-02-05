# Tomcat
Tomcat 是一款中量级的开源 Web 服务器，其包含了 HTTP 服务器 + Servlet 容器。

一个 Tomcat 实例有一个 Server，一个 Server 可以有多个 Service，一个 Service 可以包含多个 Connector 和 一个 Container。一般单个 Service 即可向外提供 Web 服务。

## Servlet 基础
Servlet 生命周期：init() -> service() -> destroy()
![[Pasted image 20240110003638.png]]
1、请求到达 server 端，server 根据 url 映射到相应的 Servlet。
2、判断 Servlet 实例是否存在，不存在则加载和实例化 Servlet 并调用 init 方法。
3、Server 分别创建 Request 和 Response 对象，调用 Servlet 实例的 service 方法（service 方法内部会根据 http 请求方法类型调用相应的 doXXX 方法）。
4、doXXX 方法内为业务逻辑实现，从 Request 对象获取请求参数，处理完毕之后将结果通过 response 对象返回给调用方。
5、当 Server 不再需要 Servlet 时（一般当 Server 关闭时），Server 调用 Servlet 的 destroy() 方法。

## Tomcat 架构设计
![[Pasted image 20240110223703.png]]
### HTTP 服务器
HTTP（请求行、请求头、请求体）是超文本传输协议。
在 Coyote 包下，由 Connector 连接器组件实现，负责接收请求和返回响应，监听端口接收外界请求，并将请求传递给容器进行业务处理，最后将结果返回给客户端。
1、监听网络端口，接收和响应网络请求。
2、网络字节流处理。将二进制流解析为 Request 对象再转换为标准的 ServletRequest 对象给容器，同时将容器传来的 ServletResponse 对象转成 Response 再转成网络字节流返回客户端。
在一个 Service 中可以有多个 Connector 来适用不同的协议的IO请求。
#### EndPoint
端点，处理 Socket 接收和发送的逻辑，其内部由 Acceptor 监听请求并由 Poller 交给 Executor 处理，Handler 处理数据、AsyncTimeout 检查请求超时。并提供字节流给 Processor。 具体实现有 NioEndPoint、AprEndpoint 等。
#### Processor
处理器，负责解析 HTTP 请求字节流，构建 Tomcat Request，具体实现有 Http11Processor等。
#### ProtocolHandler
协议处理器，将不同的协议和通讯方式组合封装成对应的协议处理器，如 Http11NioProtocol 封装的是 Http1.1 和 NIO。也就是说，ProtocolHandler 包含了 EndPoint 和 Processor。
#### Adaptor
适配器，负责将 Request 对象与 ServletRequest 对象、Response 对象与 ServletResponse 对象之间的转换。先调用 Mapper 类的 map 方法来匹配对应的Host、Context、Wrapper，并调用 StandardEngine 的 pipeline 开始处理 ServletRequest、ServletResponse。
通过 Adaptor 实现了连接器与Servlet容器的解耦。

### Servlet 容器
在 catalina 包下，由 Container 容器组件实现，包含了四层容器。负责管理、调用、执行业务逻辑。四层容器的划分体现了单一职责的设计原则。
#### Engine
一个 Service 最多有一个容器引擎，可指定默认 Host，可包含多个 Host，目前很少用多个 Host。它的 Vavle 阀门处理所有到达当前 Tomcat 的请求，StandardEngineVavle 阀门负责找到 Host 中的第一个 Vavle 阀门。
#### Host
虚拟主机，通过域名区分，可包含多个 Context。它的 Vavle 阀门处理所有同一个域名的请求，StandardHostVavle 阀门负责找到 Context 中的第一个 Vavle 阀门。
#### Context
应用级上下文，context-path 即应用顶级路径名，可包含多个 Wrapper。它的 Vavle 阀门处理所有同一个应用的请求，StandardContextVavle 阀门负责找到 Wrapper 中的第一个 Vavle 阀门。
#### Wrapper
所有请求共用一个 Servlet 实例，若 Servlet 有状态则线程不安全。包装了同一个 Servlet 类型的所有实例。它的 Vavle 阀门处理所有同一个 Servlet 的请求，StandardWrapperVavle 阀门负责找到对应的 Servlet 实例，在 FilterChain 过滤链处理之后，调用执行 service 方法。
#### 容器处理过程
每一层容器都有一个 pipeline 流水线管道，每个 pipeline 内部都有一个 StandardValve，相当于一个个阀门来处理所有的 Request 和 Response，也就是责任链模式。

## Tomca 执行流程
### 启动流程
Tomcat 设计了一个 Lifecycle 接口，各个组件实现这个接口，来执行对应生命周期的操作，包含了 init、start、stop、destroy 方法。
启动流程
![[Pasted image 20240109235850.png]]
1） 启动 tomcat， 需要调用 bin/startup.bat (在linux 目录下 , 需要调用 bin/startup.sh) ， 在 startup.bat 脚本中, 调用了 catalina.bat。
2） 在 catalina.bat 脚本文件中，调用了 BootStrap 类中的 main 方法。
3）在 BootStrap 的 main 方法中调用了 init 方法 ， 来创建 Catalina 及 初始化类加载器。
4）在 BootStrap 的 main 方法中调用了 load 方法 ， 在其中又调用了Catalina的load方法。
5）在 Catalina 的 load 方法中 , 进行了一些初始化的工作, 并需要构造Digester 对象, 用于解析 XML。
6） 然后在调用后续组件的初始化操作 。。。

加载 Tomcat 的配置文件，初始化容器组件 ，监听对应的端口号， 准备接受客户端请求 。
也就是进行各组件逐级执行 init() 和 start() 方法。

### 执行流程
当一个请求进入 Tomcat 时，执行情况如下( 一般情况下一个 Tomcat 只有一个 Service）
![[Pasted image 20240110000251.png]]
查找 Servlet 类主要通过 Mapper 组件来实现，其本质上就是 K、V 键值对。在解析时首先会将请求网址进行解析，将其中的 Host 部分在 Mapper 类中的 hosts属性（MappedHost数组，保存所有的 Host 信息）中进行查找，找到后再解析 Context 部分，在该 MapperHost 中又有 contextList 属性（保存所有的 context 信息），然后再向下找，最终得到对应的 Servlet，执行。
![[Pasted image 20240110000417.png]]
1）Connector 组件 Endpoint 中的 Acceptor 监听客户端套接字连接并接收 Socket。
2）将连接交给线程池 Executor 处理，开始执行请求响应任务。
3）Processor 组件读取消息报文，解析请求行、请求头、请求体，封装成 Request 对象。
4）CoyoteAdaptor 组件负责将 Connector 组件和 Engine 容器关联起来，把生成的 Request 对象和响应对象 Response 转换成标准的 ServletRequset 与 ServletResponse 对象再传递到 Engine 容器中，调用 Pipeline。
5）Mapper 组件根据请求行的 URL 值和请求头的 Host 值匹配由哪个 Host 容器、Context 容器、Wrapper 容器处理请求。
6）Engine 容器的管道开始处理，管道中包含若干个 Valve 阀门、每个 Valve 负责部分处理逻辑。执行完 Valve 后会执行基础的 Valve--StandardEngineValve，负责调用 Host 容器的Pipeline。
7）Host 容器的管道开始处理，流程类似，最后执行 Context 容器的 Pipeline。
8）Context 容器的管道开始处理，流程类似，最后执行 Wrapper 容器的 Pipeline。
9）Wrapper 容器的管道开始处理，流程类似，在调用完 FilterChain 过滤器链之后，执行对应的 Servlet 对象的处理方法。

### Server.xml
核心配置文件，包含了 Connector 和 Servlet 容器的相关配置，Executor 配置。

## Tomcat 网络模型
|支持的IO模型|特点|
|---|---|
|BIO|同步阻塞式IO，每一个请求都会创建一个线程，对性能开销大，不适合高并发场景。|
|NIO|同步非阻塞式IO，基于多路复用Selector监测连接状态通知线程处理，达到非阻塞的目的，比传统的BIO效率高。Tomcat8.0的默认方式。|
|AIO|异步IO，基于各种回调处理。|
|APR|Apache Http服务器的支持库，通过JNI技术调用本地库实现非阻塞IO。|

大部分、默认情况下Tomcat选择的就是NIO模型，大部分场景的网络业务都是扛得住的，到了瓶颈的时候一般也会考虑集群、分布式架构了。

## Tomcat 中的设计模式
Tomcat 在架构设计上相当精巧，使用了相当多的设计模式以及基本原则。比如：
在整体的架构上，连接器与容器之间各司其职，体现了单一职责原则。
在具体的实现上，抽取了非常多的接口与抽象类，体现了依赖倒转原则。
工厂模式、模板方法、适配器模式、责任链模式、外观模式、观察者模式等。

### 模板方法模式
在 Tomcat 的 Lifecycle 接口中，定义了统一的生命周期，包含了 init()、start()、stop()、destroy() 方法，以及对应的生命周期事件监听器方法。Tomcat 使组件实现 Lifecycle 接口来管理所有的组件的生命周期。
LifecycleBase 抽象类实现了 Lifecycle，实现了其中的方法，每个方法实现了统一的逻辑，定义了每个生命周期事件的逻辑框架，同时也提供了对应的 internal() 抽象方法来让组件实现具体的生命周期操作。
LifecycleBase 抽象类实现了 Lifecycle 接口
```java
public abstract class LifecycleBase implements Lifecycle
```
LifecycleBase 实现 start() 方法
```java
@Override
public final synchronized void start() throws LifecycleException {

	if (LifecycleState.STARTING_PREP.equals(state) || LifecycleState.STARTING.equals(state) ||
			LifecycleState.STARTED.equals(state)) {

		if (log.isDebugEnabled()) {
			Exception e = new LifecycleException();
			log.debug(sm.getString("lifecycleBase.alreadyStarted", toString()), e);
		} else if (log.isInfoEnabled()) {
			log.info(sm.getString("lifecycleBase.alreadyStarted", toString()));
		}

		return;
	}

	if (state.equals(LifecycleState.NEW)) {
		init();
	} else if (state.equals(LifecycleState.FAILED)) {
		stop();
	} else if (!state.equals(LifecycleState.INITIALIZED) &&
			!state.equals(LifecycleState.STOPPED)) {
		invalidTransition(BEFORE_START_EVENT);
	}

	try {
		setStateInternal(LifecycleState.STARTING_PREP, null, false);
		startInternal();
		if (state.equals(LifecycleState.FAILED)) {
			// This is a 'controlled' failure. The component put itself into the
			// FAILED state so call stop() to complete the clean-up.
			stop();
		} else if (!state.equals(LifecycleState.STARTING)) {
			// Shouldn't be necessary but acts as a check that sub-classes are
			// doing what they are supposed to.
			invalidTransition(AFTER_START_EVENT);
		} else {
			setStateInternal(LifecycleState.STARTED, null, false);
		}
	} catch (Throwable t) {
		// This is an 'uncontrolled' failure so put the component into the
		// FAILED state and throw an exception.
		handleSubClassException(t, "lifecycleBase.startFail", toString());
	}
}
```
提供了 startInternal() 抽象方法，供各个子类组件实现各自的生命周期逻辑。
```java
protected abstract void startInternal() throws LifecycleException;
```
总结一下，Tomcat 中的模板方法模式把公共的逻辑提取到父类中，将特殊逻辑抽象为抽象方法供子类实现，从而将公共逻辑与特殊逻辑解耦，提升了代码复用与并规范流程。

### 责任链模式
Tomcat 中的 Container 容器中对请求的处理使用了责任链模式。
对于一个 HTTP 请求：
1、请求首先被 Connector 组件接收，Connector 负责将请求转换成 ServletRequest 和 ServletResponse，并传递给 Container 中的 Engine 的 pipeline 组件。
2、从 Engine 的 pipeline（管道流水线） 中的第一个 valve（阀门）开始，每个内部的 valve 都会对请求做自己的逻辑处理，并最终传递给 StandardEngineValve，由 StandardEngineValve 传递给 Host 的 pipeline。
3、请求流转到 Host 的 pipeline 组件中，并且经过内部Valve的过滤。
4、请求流转到 Context 的 pipeline 组件中，并且经过内部的 Valve 的过滤。
5、请求流转到 Wrapper 的 pipeline 组件中，并且经过内部的 Valve 的过滤。
6、Wrapper 内部的 WrapperValve 创建 FilterChain 实例，调用指定的 Servlet 实例处理请求。
7、逐层返回。

### 观察者模式
在Tomcat的组件生命周期状态只要一变化，Tomcat就会通知改组件的所有的观察者，把状态变化通知到所有的观察者，看是否有观察者对相关组件的状态变化感兴趣。

### 适配器模式
一个请求分为 建立连接、按照协议解析请求、处理请求三个部分，因此`Tomcat`将其分成了两个模块，分别为`Connector`和`Engine`，为了方便两个模块之间彼此解耦，通过实现**适配器模式**（CoyoteAdaptor类）达到两个模块的解耦。

## Tomcat 为何打破双亲委派机制
为了防止内存中存在多份同样的字节码 .class 文件，使用了双亲委派机制（它不会自己去尝试加载类，而是把请求委托给父加载器去完成，依次向上）。JDK 中的本地方法类一般由根加载器（Bootstrp loader）装载，JDK 中内部实现的扩展类一般由扩展加载器（ExtClassLoader ）实现装载，而程序中的类文件则由系统加载器（AppClassLoader ）实现装载。
![[Pasted image 20240114163427.png]]
JDK中默认类加载器有三个: AppClassLoader、ExtClassLoader、BootStrapClassLoader。AppClassLoader 的父加载器为 ExtClassLoader，ExtClassLoader 的父加载器为 BootStrapClassLoader。这里的父子关系并不是通过继承实现的，而是组合。
什么是双亲委派机制：加载器在加载过程中，先把类交由父类加载器进行加载父类加载器没找到才由自身加载
双亲委派机制目的：为了防止内存中存在多份同样的字节码 (安全)
类加载规则：如果一个类由类加载器 A 加载，那么这个类的依赖类也是由相同的类加载器加载。
如何打破双亲委派机制：自定义 ClassLoader，重写 loadClass 方法 (只要不依次往上交给父加载器进行加载，就算是打破双亲委派机制)
Tomcat 为何打破双亲委派机制：
![[Pasted image 20240114165342.png]]
- 为了 Web 应用程序类之间隔离，为每个应用程序创建WebAppClassLoader 类加载器
- 为了 Web 应用程序类之间共享，把 ShareClassLoader 作为 WebAppClassLoader 的父类加载器，如果 WebAppClassLoader 加载器找不到，则尝试用 ShareClassLoader 进行加载
- 为了 Tomcat 本身与 Web 应用程序类隔离，用 CatalinaClassLoader 类加载器进行隔离 CatalinaClassLoader 加载 Tomcat 本身的类
- 为了 Tomcat 与 Web 应用程序类共享，用CommonClassLoader 作为 CatalinaClassLoader、ShareClassLoader 的父类加载器
ShareClassLoader、CatalinaClassLoader、CommonClassLoader 的目录可以在 Tomcat 的catalina.properties 进行配置。

线程上下文加载器：由于类加载的规则，很可能导致父加载器加载时依赖子加载器的类，导致无法加载成功 (BootStrapClassLoader 无法加载第三方库的类)，所以存在线程上下文加载器来进行加载。有了线程上下文加载器，JNDI服务使用这个线程上下文加载器去加载所需要的SPI代码，也就是父类加载器请求子类加载器去完成类加载的动作，这种行为实际上就是打通了双亲委派模型的层次结构来逆向使用类加载器，实际上已经违背了双亲委派模型的一般性原则。但这无可奈何，Java中所有涉及SPI的加载动作基本胜都采用这种方式。例如JNDI，JDBC，JCE，JAXB，JBI等。

## Tomcat 热加载与热部署
热部署就是在服务器运行时重新部署项目，热加载即在在运行时重新加载class，从而升级应用。
通常情况下在开发环境中我们使用的是热加载，因为热加载的实现的方式在Web容器中启动一个后台线程，定期检测相关文件的变化，如果有变化就重新加载类，这个过程不会清空Session。而在生产环境我们一般应用的是热部署，热部署也是在Web应用后台线程定期检测，发现有变化就会重新加载整个Web应用，这种方式更加彻底会清空Session。

## SpringBoot 整合 Tomcat
SpringBoot 默认内嵌了 Tomcat，当然也可以替换为其他的 Web 服务器如 Jetty、Undertow。在容器启动时，在 refresh 方法中，会启动内置的 Tomcat。
Tomcat 并发度决定了一个 SpringBoot 微服务节点对处理 HTTP 请求的并发度。
这里需要注意，最大连接数（maxConnections 默认 10000），最大线程数（maxThreads 默认 200），pending 队列数（acceptCount 默认 100）
情况1：接受一个请求，此时 tomcat 启动的线程数没有到达 maxThreads，tomcat 会起动一个线程来处理此请求。
情况2：接受一个请求，此时 tomcat 启动的线程数已经到达 maxThreads，tomcat 会把此请求放入等待队列，等待空闲线程。
情况3：接受一个请求，此时 tomcat 启动的线程数已经到达maxThreads，等待队列中的请求个数也达到了acceptCount，此时 tomcat 会直接拒绝此次请求，返回connection refused。

## Tomcat 调优
### JVM 层面
因为 Tomcat 是一个 Java 应用，所以可以考虑 JVM 相关的调优。
增大堆内存，一般可以指定物理内存的80%，在配置文件中加入配置信息。
### Tomcat 连接器
连接器部分的相关配置其实是 Tomcat 性能的关键。
1、优化线程池，增加线程数。
2、设置 enableLookups="false" 取消 DNS 检查。
3、设置 compression=”on” 对http相应数据启用Gzip压缩。如果开启压缩，意味着较少的网络传输量，但是将消耗一定的CPU进行压缩处理。如果你的应用有较高的CPU性能结余，且响应数据均是一些文本字符串，那么开启压缩，会有较大的收益。（并不是所有的浏览器都能够合理的支持gzip压缩，特别是低版本）
4、把 useURIValidationHack 设成”false”，减少对一些 url 的不必要的检查从而减省开销。
### 异步 Servlet
通过使用异步 Servlet，在 Connector 接收请求之后，将请求抛给业务线程池来处理，从而提前归还 Connector 线程池中的线程，提高了 Tomcat 的对请求处理的吞吐量。

## 参考文章
1、[Tomcat基础知识总结 | Java学习&面试指南-程序员大彬 (topjavaer.cn)](https://topjavaer.cn/web/tomcat.html#%E6%9E%B6%E6%9E%84)
2、[Tomcat组成与工作原理 - 掘金 (juejin.cn)](https://juejin.cn/post/6844903473482317837)
3、[阿里二面：你知道 Tomcat 的工作原理么？ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/379107635)
4、[「面试必背」Tomcat面试题（收藏）-CSDN博客](https://blog.csdn.net/q66562636/article/details/124455010)
5、[Tomcat中涉及哪些设计模式_tomcat 设计模式-CSDN博客](https://blog.csdn.net/u014653854/article/details/82217173#:~:text=Tomcat%E4%B8%AD%E6%B6%89%E5%8F%8A%E5%93%AA%E4%BA%9B%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%201%201.%20%E5%B7%A5%E5%8E%82%E6%A8%A1%E5%BC%8F%20%E5%9C%A8Tomcat%E5%90%AF%E5%8A%A8%E7%9A%84%E8%BF%87%E7%A8%8B%E4%B8%AD%EF%BC%8C%E8%B0%83%E7%94%A8Connector%E7%BB%84%E4%BB%B6%E7%9A%84startInternal%20%28%29%E6%96%B9%E6%B3%95%EF%BC%9A%20...%202,2.%20%E6%A8%A1%E7%89%88%E6%A8%A1%E5%BC%8F%20%E5%9C%A8%E8%AE%B2%E8%A7%A3Tomcat%E4%B8%AD%E7%BB%84%E4%BB%B6%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E7%AE%A1%E7%90%86%E7%9A%84%E7%9B%B8%E5%85%B3%E6%96%87%E7%AB%A0%E4%B8%AD%EF%BC%8C%E6%88%91%E4%BB%AC%E4%BA%86%E8%A7%A3%E5%88%B0Lifecycle%E6%8E%A5%E5%8F%A3%E7%9A%84%E7%9B%B8%E5%85%B3%E7%B1%BB%E5%9B%BE%E5%A6%82%E4%B8%8B%EF%BC%9A%20...%203%203.%20%E8%A7%82%E5%AF%9F%E8%80%85%E6%A8%A1%E5%BC%8F%20)
6、[(20 封私信 / 14 条消息) Tomcat为什么要JAVA破坏双亲委派机制？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/466696410)
7、[Tomcat——调优指南 - 掘金 (juejin.cn)](https://juejin.cn/post/7246264754141921341?from=search-suggest)
8、[Tomcat的Connector配置 - 简书 (jianshu.com)](https://www.jianshu.com/p/b254c1c04770)
