springbootapplication注解是一个组合的注解：



@configuration,配置类和(@Bean注解搭配实现<Beans><Bean></Bean></Beans>)

@EnableAutoConfiguration,自动注入的类,通过import导入类

@ComponentScan,自动扫描项目下的注解，注入Bean

通过开启EnableAutoConfiguration和@Conditional条件来根据当前类路径有当前的class之后实现依赖注入，依赖注入的时候同时通过@properties来实现属性的注入，同时可以在application.properties中实现默认属性的覆盖。注册自动配置的类。。。了解springboot的启动原理。



接管springboot的web配置(比如拦截器，消息传递器等，文件上传器，HttpMessageConvert等)

```
@Configuration
@EnableWebMvc
@ComponentScan("com.netease.mail.vip.juwan.web.controller")
public class MyMvcConfig extends WebMvcConfigurerAdapter {
   @Override
   public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
      converters.add(converter());
   }
```

通过@EnableWebMVC实现，继承WebMvcConfigurerAdapter同时需要覆盖其中的相应的方法：

HttpMessageConvert实现extendMessageConverters的重写，这个例子可以参见SpringBoot实战。



springboot常见注解,@controller,@Service，@Param.@Pathvariable,@Autowired等注解，springboot的配置属性文件，不同开发环境的环境配置等等知识点。



spring boot经典的开发实战，有详细解释。。





!!!!!!!

@Configuration

@Bean

等于:

<beans><bean></bean></beans>往spring容器中注入bean

同时spring-boot启动中的时候会自动扫描

@Component,@Service,@Controller,@RestController等注解，将对象注入到Spring容器中

所谓的控制反转：IOC



原来我们使用一个对象的时候，自己手动创建一个对象使用，Date date=new Date().....例如这样，自己管理对象，但是有了Spring之后，将对象注入到容器中(单例模式)，之后我们需要调用的时候只要跟容器说一声，容器就会把对象建好给我们用，即我们只是发个命令，但是创建管理的工作由容器负责，即所谓的控制反转。

依赖注入：需要使用Bean的时候，使用注解@Autowired获取





所谓的面向切面编程：AOP见AOP详解



