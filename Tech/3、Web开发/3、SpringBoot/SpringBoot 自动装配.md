一句话解释自动装配：SpringBoot 会根据用户的配置来自动进行条件化的 Bean 的注入，从而获得对应的能力，比如我们引入了。

SpringBoot 启动类核心注解：`SpringBootApplication`
```java
@Target({ElementType.TYPE})  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
@Inherited  
@SpringBootConfiguration  
@EnableAutoConfiguration  
@ComponentScan(  
excludeFilters = {@Filter(  
type = FilterType.CUSTOM,  
classes = {TypeExcludeFilter.class}  
), @Filter(  
type = FilterType.CUSTOM,  
classes = {AutoConfigurationExcludeFilter.class}  
)}  
)  
public @interface SpringBootApplication {
	// ...
}
```

`@EnableAutoConfiguration`：启用 SpringBoot 的自动配置机制的核心注解
`@Configuration`：允许在上下文中注册额外的 bean 或导入其他配置类
`@ComponentScan`：扫描被`@Component` (`@Service`,`@Controller`)注解的 bean，注解默认会扫描启动类所在的包下所有的类 ，可以自定义不扫描某些 bean。

![[Pasted image 20240323233853.png]]

# @EnableAutoConfiguration


