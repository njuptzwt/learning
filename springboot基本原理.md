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