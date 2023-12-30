# 一、简介
## Spring 是啥？
官网：[Spring | Home](https://spring.io/)
1、Spring 是一个轻量级的开源框架，是Spring生态圈的基础和核心，面向企业级应用。
2、Spring 包含了两个最核心的模块：
- Ioc（Inverse of Control）控制反转
- Aop（Aspect Oriented Programming）面向切面编程

## 特点
- 非侵入式，对系统侵入很低
- 控制反转
- 面向切面编程
- 容器管理
- 一站式生态

# 二、Ioc
Inverse of Control 控制反转，把对象创建和对象之间的调用过程，交给 Spring 进行管理，解耦对象的创建与调用。也就是说 Spring Ioc 会帮我们实现创建对象与依赖注入的操作（可基于xml或者注解方式）。

## Ioc 容器
Spring 中通过 Ioc 容器来管理 Bean，有两个核心接口
- BeanFactory：最顶层的接口，Ioc容器基本实现，是Spring内部使用的接口，一般不提供给外部开发人员使用，懒汉式创建对象
- ApplicationContext：BeanFactory子接口，提供更多的功能，面向开发人员使用，饿汉式创建对象

## Ioc 创建对象
Ioc 创建对象基本原理：xml 配置类属性（比如全限定名） + 工厂模式（也就是对象工厂，创建Bean） + 反射创建对象。

## Ioc 依赖注入
也就是注入属性
1、创建类之后通过 **set 方法**注入属性。Spring xml 配置文件中通过 property 标签注入就是调用了类的 set 方法。
2、**有参构造器**注入属性，xml 中的 constructor-arg 标签
3、
4、

 
 


