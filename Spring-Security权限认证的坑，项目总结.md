spring-security用于用户认证和权限分配。所做的项目服务都是多机部署的，这样就产生了一系列Session相关的问题，原来存储在Session中的信息都需要进行共享才不会发生错误。



所做的项目中，认证过程使用了网易openId认证，使用openId认证完成之后，会有一个回调，在该回调函数中，进行用户的权限分配。

阅读源码学到的两个主要知识点，都是项目中踩到的坑。



**第一个坑：Spring Security源码分析十三：Spring Security 基于表达式的权限控制**

在HttpConfigure中赋予权限：.antMatchers("/upload/**")<u>.hasAnyRole("Role_ADMIN")</u>;



在认证完毕之后赋予权限：

```
List<GrantedAuthority> updatedAuthorities = new ArrayList<>();
// 赋予用户对应的角色信息
updatedAuthorities.add(new SimpleGrantedAuthority(user.getRole().getRole()));
Authentication newAuth = new UsernamePasswordAuthenticationToken(email, "", updatedAuthorities);
SecurityContextHolder.getContext().setAuthentication(newAuth);
```

这样认证会有问题？认证不通过？原因在哪里？

http://niocoder.com/2018/01/30/Spring-Security%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E5%8D%81%E4%B8%89-Spring-Security%E6%9D%83%E9%99%90%E6%8E%A7%E5%88%B6/#%E5%B8%B8%E8%A7%81%E7%9A%84%E8%A1%A8%E8%BE%BE%E5%BC%8F

原因：

​     hasAnyRole("ROLE_ADMIN")， hasRole("ROLE_ADMIN")，这些赋值的时候会在字符串中自动加     入ROLE_,所以实际对象会变成ROLE_ROLE_ADMIN,所以认证不通过。

解决方案：

a、 hasAnyRole("ADMIN")， hasRole("ADMIN")变成这种

b、hasAnyAuthority(”ROLE_ADMIN“），hasAuthority(”ROLE_ADMIN“）

| 表达式                            | 描述                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| `hasRole([role]`)                 | 用户拥有制定的角色时返回true （`Spring security`默认会带有`ROLE_`前缀）,去除参考[Remove the ROLE_](https://github.com/spring-projects/spring-security/issues/4134) |
| `hasAnyRole([role1,role2])`       | 用户拥有任意一个制定的角色时返回true                         |
| `hasAuthority([authority])`       | 等同于`hasRole`,但不会带有`ROLE_`前缀                        |
| `hasAnyAuthority([auth1,auth2])`  | 等同于`hasAnyRole`                                           |
| `permitAll`                       | 永远返回true                                                 |
| `denyAll`                         | 永远返回false                                                |
| `anonymous`                       | 当前用户是`anonymous`时返回true                              |
| `rememberMe`                      | 当前勇士是`rememberMe`用户返回true                           |
| `authentication`                  | 当前登录用户的`authentication`对象                           |
| `fullAuthenticated`               | 当前用户既不是`anonymous`也不是`rememberMe`用户时返回true    |
| `hasIpAddress('192.168.1.0/24'))` | 请求发送的IP匹配时返回true                                   |





第二个坑：Authentication,对象存储在SecurityContextHolder中的SecurityContext。同一个用户的多个请求（不同线程）是通过Session来获取SecurityContext的，进一步获取Authentication，进一步获取权限的认证。(多机需要实现Session共享，同时把一台机子上的Context设置到另一台机子中！)

有一点很重要：每次请求结束之后，都会在Session中保持最新的Authentication,同一个用户的请求都会保持相同的权限，Thread中SecurityContext都是Session中的拷贝！！结合项目，来理解！

知识点：Spring-Security的过滤器和实现原理机制，参考文献如下

https://juejin.im/post/5a6aef4f6fb9a01cbb3957b7

https://zhuanlan.zhihu.com/p/27644391

主要的几个filter，Filterchain有10几个filter

1、SecurityContextPersistenceFilter（SecurityContext持久化和获取）

2、UserNamePassWordAuthentionFilter（用户认证，用户名和密码）

3、CSRF Filter（CSRF攻击）

4、FilterSecurityInterceptor(权限过滤拦截使用这个过滤器)



详解一个请求来的时候，Spring-Security怎么做权限的认证。详解源码，主要是SecurityContextHolder和SecurityContextPersitenceFilter



SecurityContextHolder：源码

```
public class SecurityContextHolder {
    public static final String MODE_THREADLOCAL = "MODE_THREADLOCAL";
    public static final String MODE_INHERITABLETHREADLOCAL = "MODE_INHERITABLETHREADLOCAL";
    public static final String MODE_GLOBAL = "MODE_GLOBAL";
    public static final String SYSTEM_PROPERTY = "spring.security.strategy";
    private static String strategyName = System.getProperty("spring.security.strategy");
    private static SecurityContextHolderStrategy strategy;
    private static int initializeCount = 0;

    public SecurityContextHolder() {
    }

    public static void clearContext() {
        strategy.clearContext();
    }

    public static SecurityContext getContext() {
        return strategy.getContext();
    }

    public static int getInitializeCount() {
        return initializeCount;
    }

    private static void initialize() {
    //默认策略，ThreadLocal
        if (!StringUtils.hasText(strategyName)) {
            strategyName = "MODE_THREADLOCAL";
        }

        if (strategyName.equals("MODE_THREADLOCAL")) {
            strategy = new ThreadLocalSecurityContextHolderStrategy();
        } else if (strategyName.equals("MODE_INHERITABLETHREADLOCAL")) {
            strategy = new InheritableThreadLocalSecurityContextHolderStrategy();
        } else if (strategyName.equals("MODE_GLOBAL")) {
            strategy = new GlobalSecurityContextHolderStrategy();
        } else {
            try {
                Class<?> clazz = Class.forName(strategyName);
                Constructor<?> customStrategy = clazz.getConstructor();
                strategy = (SecurityContextHolderStrategy)customStrategy.newInstance();
            } catch (Exception var2) {
                ReflectionUtils.handleReflectionException(var2);
            }
        }

        ++initializeCount;
    }

    public static void setContext(SecurityContext context) {
        strategy.setContext(context);
    }

    public static void setStrategyName(String strategyName) {
        strategyName = strategyName;
        initialize();
    }

    public static SecurityContextHolderStrategy getContextHolderStrategy() {
        return strategy;
    }

    public static SecurityContext createEmptyContext() {
        return strategy.createEmptyContext();
    }

    public String toString() {
        return "SecurityContextHolder[strategy='" + strategyName + "'; initializeCount=" + initializeCount + "]";
    }

   // 默认启动策略
    static {
        initialize();
    }
}
```

SecurityContextPersistenceFilter源码关键部分：

```
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {
    HttpServletRequest request = (HttpServletRequest)req;
    HttpServletResponse response = (HttpServletResponse)res;
    if (request.getAttribute("__spring_security_scpf_applied") != null) {
        chain.doFilter(request, response);
    } else {
        boolean debug = this.logger.isDebugEnabled();
        request.setAttribute("__spring_security_scpf_applied", Boolean.TRUE);
        if (this.forceEagerSessionCreation) {
            HttpSession session = request.getSession();
            if (debug && session.isNew()) {
                this.logger.debug("Eagerly created session: " + session.getId());
            }
        }

        HttpRequestResponseHolder holder = new HttpRequestResponseHolder(request, response);
        // 从Sesion中获取SecurityContext
        SecurityContext contextBeforeChainExecution = this.repo.loadContext(holder);
        boolean var13 = false;

        try {
            var13 = true;
            
            // 当前线程获取SecurityContext
            SecurityContextHolder.setContext(contextBeforeChainExecution);
            chain.doFilter(holder.getRequest(), holder.getResponse());
            var13 = false;
        } finally {
            if (var13) {
                SecurityContext contextAfterChainExecution = SecurityContextHolder.getContext();
                // 结束，清除当前线程的SecurityContext
                SecurityContextHolder.clearContext();
                // Session重新保存SecurityContext
                this.repo.saveContext(contextAfterChainExecution, holder.getRequest(), holder.getResponse());
                request.removeAttribute("__spring_security_scpf_applied");
                if (debug) {
                    this.logger.debug("SecurityContextHolder now cleared, as request processing completed");
                }

            }
        }

        SecurityContext contextAfterChainExecution = SecurityContextHolder.getContext();
        SecurityContextHolder.clearContext();
        this.repo.saveContext(contextAfterChainExecution, holder.getRequest(), holder.getResponse());
        request.removeAttribute("__spring_security_scpf_applied");
        if (debug) {
            this.logger.debug("SecurityContextHolder now cleared, as request processing completed");
        }

    }
}
```

总结：

浏览器发起一个Http请求到达Tomcat，Tomcat将其封装成一个Request，先经过Filter，其中经过spring security的SecurityContextPersistenceFilter，从session中取出SecurityContext（如果没有就创建新的）存入当前线程的ThreadLocalMap中，因为是当前线程，所以不同的线程之间根本互不影响。之后完成Servlet调用，执行finally语句块，清除当前线程中ThreadLocalMap对应的SecurityContext，再将其覆盖session中之前的部分。作者：FonLin链接：https://juejin.im/post/5a6aef4f6fb9a01cbb3957b7来源：掘金著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

理解图：

![pring-Securit](C:\Users\zhengwentian\Desktop\踩坑之路\Spring-Security.jpg)