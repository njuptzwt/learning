java中的代理模式，代理模式的详解！AOP应用

http://www.importnew.com/26116.html，

https://blog.csdn.net/briblue/article/details/73928350

spring AOP的详解

spring AOP的底层实现原理，动态代理---》反射机制



CGLIB动态代理的JDK动态代理

动态代理的核心是InvocationHandler接口和Proxy类，invoke方法

http://www.importnew.com/24305.html

https://www.ibm.com/developerworks/cn/java/j-lo-springaopcglib/

1、项目中的实际用法：**请求参数的验证，和日志的打印**

```
**
 * @author hzcaixinjia
 *
 *         controller拦截器，用于参数验证，返回参数整合，日志输出等
 */
@Aspect
@Component
@Order(2)
public class ReqControllerAspect {

   private static Logger LOG = Logger.getLogger(ReqControllerAspect.class);

   @Around("@annotation(com.netease.mail.vip.juwan.web.annotation.ParamValidate)&&@annotation(paramValidate)")
   public BaseResponseVO doAspect(ProceedingJoinPoint joinPoint, ParamValidate paramValidate) {
      Errors errors = null;
      BaseRequestVO requestVO = null;
      BaseWholeReqInfo baseWholeReqInfo = new BaseWholeReqInfo();
      BaseResponseVO responseVO = null;
      Object[] args = joinPoint.getArgs();
      if (args != null) {
         for (Object arg : args) {
            if (arg instanceof BaseRequestVO) {
               requestVO = (BaseRequestVO) arg;
            } else if (arg instanceof Errors) {
               errors = (Errors) arg;
            }
         }
      }
      baseWholeReqInfo.setRequestInfo(paramValidate.name());
      baseWholeReqInfo.setBaseRequestVO(requestVO);
      if (errors != null && errors.hasErrors()) {
         responseVO = BaseResponseVO.PARAM_LOST;
         baseWholeReqInfo.setBaseResponseVO(responseVO);
         LOG.info(LogUtil.format("[cmd=controller error]log={}", baseWholeReqInfo.printInfo()));
      } else {
         if (paramValidate.needValidate() == true) {
            try {
               responseVO = (BaseResponseVO) joinPoint.proceed();
               System.out.println("out aspect");
               baseWholeReqInfo.setBaseResponseVO(responseVO);
               LOG.info(LogUtil.format("[cmd=controller success]log={}", baseWholeReqInfo.printInfo()));
            } catch (Throwable e) {
               if (e instanceof IllegalArgumentException) {
                  responseVO = BaseResponseVO.PARAM_ERROR;
               } else {
                  responseVO = BaseResponseVO.SYSTEM_ERROR;
               }
               baseWholeReqInfo.setBaseResponseVO(responseVO);
               e.printStackTrace();
               LOG.info(LogUtil.format("[cmd=controller error]error_info={},log={}", e.getMessage(),
                     baseWholeReqInfo.printInfo()));
            }
         }
      }
      return responseVO;
   }
}
```

2、项目中的http请求时间（请求和相应的差值）

**在请求方法before前使用"ThreadLocal"变量来统计请求开始时间**

**在after切面方法中获取请求结束时间，记录总时间**

http://blog.didispace.com/springbootaoplog/

https://blog.csdn.net/DuShiWoDeCuo/article/details/78180803

RequestContextHolder,SecurityContextHolder,上下文的获取类！