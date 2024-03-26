1、实例化：Spring 容器启动时，读取配置信息或者注解信息，拿到类信息定义（BeanDefinition），再反射 getDeclaredConstructor 通过构造器 newInstance 创建 bean 实例，其实在实例化前后也有回调方法，在实例化前后做处理，比如修改 BeanDefinition 等。
2、设置对象属性（依赖注入），注入相关依赖（通过@Autowried与@Resource），以及检查 Aware 相关接口，来 set 对应的 aware 的属性（比如 beanName，ApplicationContext，BeanFactory）。
3、如果有实现 BeanPostProcessor 接口则执行`postProcessBeforeInitialization()`方法，进行前置处理。
4、初始化，如果有实现 InitializingBean 接口则调用 afterPropertiesSet 方法或者存在被 @PostConstruct 注解标记的方法，则进行 bean 的初始化。
5、如果有实现 BeanPostProcessor 接口则执行`postProcessAfterInitialization()`方法，进行后置处理。
6、使用中，bean 已被放入 Ioc 容器中。
7、容器关闭时，如果有实现 DisposableBean 接口，则调用 bean 的 destroy 方法，或者是 @PreDestroy 注解标记的方法。