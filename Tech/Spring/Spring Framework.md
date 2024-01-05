# 一、简介
## Spring 是啥？
官网：[Spring | Home](https://spring.io/)
1、Spring 是一个轻量级的开源框架，是Spring生态圈的基础和核心，面向企业级应用。
2、Spring 包含了两个最核心的模块：
- Ioc（Inverse of Control）控制反转
- Aop（Aspect Oriented Programming）面向切面编程
![[Pasted image 20231230150829.png]]
## 特点
- 非侵入式，对系统侵入很低
- 控制反转
- 面向切面编程
- 容器管理
- 一站式生态

# 二、Ioc
Inverse of Control 控制反转，把对象创建和对象之间的调用过程，交给 Spring 进行管理，解耦对象的创建与调用。也就是说 Spring Ioc 会帮我们实现创建对象与依赖注入的操作（可基于xml或者注解方式）。
IoC 容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的；当我们需要依赖一个对象的时候，只需要使用 Ioc 容器获取 Bean 或者使用注解将需要的 Bean 注入即可。
在传统的Java开发中，对象的创建和依赖关系通常在类的内部或通过工厂模式等方式进行管理。而在IoC容器中，这些责任被反转了，对象的创建和依赖关系的管理由容器负责，而应用程序则从容器中获取所需的对象。

## Ioc 容器管理
Spring 中通过 Ioc 容器来管理 Bean，所有的 Bean 都在 Ioc 容器中进行管理，有两个核心接口：
- BeanFactory：最顶层的接口，Ioc容器基本实现，是Spring内部使用的接口，一般不提供给外部开发人员使用，懒汉式创建对象
- ApplicationContext：BeanFactory子接口，提供更多的功能，面向开发人员使用，饿汉式创建对象

## Ioc 创建对象
基本原理：xml 配置类属性定义或者注解方式获取类的信息（扫描包路径） + 反射或者工厂方法创建对象。

## Ioc 依赖注入
基本原理：对容器中的 bean 进行依赖注入，通过反射获取需要的字段属性，并为其注入值。有三种方式注入属性或者对象。
1、Setter 方法注入，创建类之后通过 **set 方法**注入属性。Spring xml 配置文件中通过 property 标签注入就是调用了类的 set 方法。
2、构造器注入，**有参构造器**注入属性，xml 中的 constructor-arg 标签。
3、Field 属性注入，属性直接注入方式，当前最流行的方式。目前Idea会对Field注入标黄，表示不建议使用属性注入，原因可能在于导致某个类容易引入过多的外部依赖。
这三种方式都可以结合注入注解使用。

## Spring Ioc Bean
简单来说，Bean 代指的就是那些被 IoC 容器所管理的对象。

### 注册 Bean 的注解
- `@Component`：通用的注解，可标注任意类为 `Spring` 组件。如果一个 Bean 不知道属于哪个层，可以使用`@Component` 注解标注。
- `@Repository`: 对应持久层即 Dao 层，主要用于数据库相关操作。
- `@Service` : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao 层。
- `@Controller`: 对应 Spring MVC 控制层，主要用于接受用户请求并调用 `Service` 层返回数据给前端页面。
-  `@Configuration`: 用来声明配置类。
- `@Bean`：唯一一个作用在方法上的注解，通常和 `@Configuration`注解配合使用。@Bean 注解用于告诉 Ioc，该方法会产生一个 Bean 对象，然后将这个 Bean 对象交给 Ioc 管理。产生这个 Bean 对象的方法 Spring 只会调用一次，随后 Spring 会将这个 Bean 对象放在自己的 Ioc 容器中；

### 注入 Bean 的注解
Spring 内置的 `@Autowired` 以及 JDK 内置的 `@Resource`。
- `Autowired` 属于 Spring 内置的注解，默认的注入方式为`byType`，也就是说会优先根据接口类型去匹配并注入 Bean（接口的实现类），Autowired 遇到多个实现类时，会匹配属性名对应的类名，不过一般建议使用@Qualifier 注解显式指定（首字母小写，根据 byName 的方式自动装配）。
- `@Resource`属于 JDK 提供的注解，默认注入方式为 `byName`。Resource 遇到多个实现类时，会先匹配属性名对应的类名。如果无法通过名称匹配到对应的 Bean 的话，注入方式会变为`byType`。有两个比较重要且日常开发常用的属性：`name`（名称）、`type`（类型）。
当一个接口存在多个实现类的情况下，`@Autowired` 和`@Resource`都需要通过名称才能正确匹配到对应的 Bean。`Autowired` 可以通过 `@Qualifier` 注解来显式指定名称，`@Resource`可以通过 `name` 属性来显式指定名称。
`@Autowired` 支持在构造函数、方法、字段和参数上使用。`@Resource` 主要用于字段和方法上的注入，不支持在构造函数或参数上使用。

### Bean 的作用域
通过设置 @Scope 注解来设置 Bean 的作用域，默认是单例。
- singleton：默认值，单例
- prototype：每次都是一个新的实例对象
- request：每次http请求都是一个新的实例对象
- session：每次新的session的http请求都会产生一个新的实例对象
- application：每个Web应用启动时创建一个实例对象
一般项目中使用单例的作用域。

### Bean 是否线程安全？
陷阱问题，需要结合 Bean 的作用域以及是否有状态来考虑。
如果是 prototype，那么每次获取都是一个新的实例，不会有线程安全问题。
如果是 singleton，那么就要看这个 Bean 内部是否是有状态的，也就是说是否有共享的变量，如果这个 Bean 是有状态的，那么就会存在线程安全问题。

### Bean 的生命周期
1、实例化：Spring 容器启动时，读取配置信息或者注解信息，拿到类信息，再通过构造器或者反射或者工厂方法创建 bean 实例
2、设置对象属性，以及检查 Aware 相关接口，并注入相关依赖（通过@Autowried与@Resource）
3、如果有实现 BeanPostProcessor 接口则执行`postProcessBeforeInitialization()`方法，进行前置处理
4、如果有实现 InitializingBean 接口则调用 afterPropertiesSet 方法或者存在被 @PostConstruct 注解标记的方法，则进行 bean 的初始化
5、如果有实现 BeanPostProcessor 接口则执行`postProcessAfterInitialization()`方法，进行后置处理
6、使用中，bean 已被放入 Ioc 容器中
7、容器关闭时，如果有实现 DisposableBean 接口，则调用 bean 的 destroy 方法，或者是 @PreDestroy 注解标记的方法

### Spring 如何解决 Bean 的循环依赖？

# 三、Spring Aop
底层实现：动态代理

1、**切面（Aspect）**
切面是横切关注点的模块化体现，包含了一组通知（Advice）和切点（Pointcut）。在 Spring 中，切面通常由一个 Java 类来表示，其中包含了横切关注点的定义和相应的增强通知逻辑。
2、**连接点（Join Point）**
连接点是在程序执行过程中能够插入切面的点。典型的连接点包括方法调用、方法执行、对象的创建等。
3、**通知（Advice）**
通知定义了在连接点上执行的操作。Spring AOP 提供了不同类型的通知，包括前置通知（Before）、后置通知（After）、返回通知（AfterReturning）、异常通知（AfterThrowing）、环绕通知（Around，在方法执行前后进行通知）等。
4、**切点（Pointcut）**
切点是一组连接点的集合。它定义了在何处应用通知。切点可以使用表达式来定义，以匹配程序中的特定方法调用或类。
5、**引入（Introduction）**
引入允许向现有的类添加新的方法和属性，以实现对类的扩展。这使得可以在不修改原有代码的情况下，给类添加新的功能。
6、**目标对象（Target Object）**
目标对象是被通知（增强）的对象。它是原始的业务逻辑对象。
7、**代理对象（Proxy Object）**
代理对象是包含通知的对象，它代替了目标对象被注入到程序中。Spring AOP 默认使用动态代理（JDK 动态代理或 CGLIB）生成代理对象。


1、动态代理分类
JDK 动态代理类（有接口使用）
- 代理对象和目标对象实现同样的接口
cglib 动态代理类（无接口使用）
- 代理对象继承被代理的目标类

## AspectJ
AspectJ是一个强大的面向切面编程（AOP）框架，它在 Java 编程语言的基础上扩展了 AOP 的功能。AspectJ 提供了更丰富和灵活的切面编程能力，相较于 Spring AOP，AspectJ更直接地支持 AOP 的各种概念和功能。


切面优先级

@Pointcut() 来声明切点方法。


# Spring 事务








 
 


