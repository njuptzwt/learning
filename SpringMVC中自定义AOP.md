SpringMVC中自定义AOP在请求前后进行数据的处理：



```
@Aspect
@Component
@Order(2)
```

Aspect注解引入切面类,切面一定要和切入点对应。

Component组件的注解，将切面类注入spring容器中

Order用来定义切面的执行注入的顺序

```
@Around("@annotation(com.netease.mail.vip.juwan.web.annotation.ParamValidate)&&@annotation(paramValidate)")
public BaseResponseVO doAspect(ProceedingJoinPoint joinPoint, ParamValidate paramValidate) {
   System.out.println("enter aspect");
   Errors errors = null;
   BaseRequestVO requestVO = null;
   BaseWholeReqInfo baseWholeReqInfo = new BaseWholeReqInfo();
   BaseResponseVO responseVO = null;
   Object[] args = joinPoint.getArgs();//获取函数中的参数
   if (args != null) {
      for (Object arg : args) {
         if (arg instanceof BaseRequestVO) {
            requestVO = (BaseRequestVO) arg;//判断参数中是否有需要的参数继承BaseRequestVO
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
            responseVO = (BaseResponseVO) joinPoint.proceed();//执行切面函数
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
            LOG.info(LogUtil.format("[cmd=controller error]error_info={},log={}", e.getMessage(),
                  baseWholeReqInfo.printInfo()));
         }
      }
   }
   return responseVO;
}
```

通过@Around里面直接定义切入点（也可以使用@Pointcut来实现切入点的定义),最后通过参数：

ProceedingJoinPoint joinPoint，连接点获取一系列的参数和执行所要进行的函数。



joinPoint.getArgs();来获取所要执行函数的参数



joinPoint.proceed()用来执行要执行的函数。退出切面







