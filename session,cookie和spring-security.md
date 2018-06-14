Java中的session和cookie机制，session的实现机制，和cookie的使用方法！



https://www.cnblogs.com/roy-blog/p/8250519.html



1、Session共享：

然后是,spring mvc中使用@EnableRedisHttpSession来实现sessionId的共享方案，核心是使用redis来缓存Sesssion，然后根据SessionId来找对应的Session，一个SessionId对应的是唯一的Session，



2、服务器中request.getSession()判断过程：

（1）当我们首次在某个请求中通过调用request.getSession去获取session的时候（这个调用不一定是显式的，很多框架把session封装成map等其他的类型，名称也不一定是session，但是本质都是在调用session），首先servlet通过getCookie的到本次请求的cookie信息，然后去寻找cookie数组中是否有否有一个名为JSESSIONID的cookie，没有的话就创建一个session，并且把sessionId做为JSESSIONID这个cookie的值，然后调用addCookie（）方法把该cookie放入cookie数组。



（2）如果上一步中从cookie数组中取到的cookie数组已经包含了JSESSIONID这个cookie，这时候我我们取出JSESSIONID的值，然后去内存中的session（内存中有很多session）去寻找对应的sessionId为JSESSIONID的值的session，如果找得到的话，就使用这个session，找不到的话，就新建一个session，并且同样的调用addCookie（）方法覆盖掉原来的JSESSIONID这个cookie的值。



session能够保存用户的信息，克服浏览器的无状态特点，存取信息！！





3、spring-security的表达式：实现认证和权限控制！后台管理系统使用OpenID来实现登陆流程，以及遇到的问题和解决办法！！

添加spring-security的注解

```
@EnableWebSecurity

```

```
protected void configure(HttpSecurity http) throws Exception {
   http.csrf().disable().authorizeRequests().antMatchers("/front/**").permitAll().antMatchers("/item/**")
         .hasAnyAuthority(Role.ROLE_ADMIN).antMatchers("/order/**").hasAnyAuthority(Role.ROLE_ADMIN)
         .antMatchers("/upload/**").hasAnyAuthority(Role.ROLE_ADMIN).antMatchers("/advertising/**")
         .hasAnyAuthority(Role.ROLE_ADMIN).antMatchers("/admin/**").hasAnyAuthority(Role.ROLE_ADMIN);
   http.addFilterBefore(new SecurityContextFilter(), FilterSecurityInterceptor.class);
}
这个部分是对不同角色授予url的权限，授予角色的表达式！！！！！！

第一个坑，不熟悉spring-security的表达式，导致每一次使用hasRole(“admin”)变成hasRole(“ROLE_admin”)，权限认证一直失败！！！

解决的办法：使用hasAuthority来分配权限

hasRole
判定是否有指定的角色,若指定的角色中没有以”ROLE_”开头,则会自动加上,如hasRole(“admin”) 则表示判断是否有”ROLE_admin”角色,而非判断是否有”admin”角色.”ROLE_”可以在DefaultWebSecurityExpressionHandler类中修改默认设置.

hasAnyRole
判断是否有指定角色中的任意一个.

hasAuthority
判断是否有指定的角色

hasAnyAuthority
判断是否有指定角色中的任意一个

principal
Allows direct access to the principal object representing the current user (没看懂是什么意思)

authentication
Allows direct access to the current Authentication object obtained from the SecurityContext (没看懂是什么意思)

permitAll
不管控,所有的人都可以访问

denyAll
禁止所有的人访问

isAnonymous
判断是否是匿名用户,即用户未登录则返回true

isRememberMe
从字面讲理解用户记住登录的则返回true(但是不太理解具体是什么意思)

isAuthenticated
用户已经登录的则返回true

isFullyAuthenticated
用户已经登录了,且不是RememberMe状态的.

hasPermission(Object target, Object permission)
检验是否有某对象的指定权限.

hasPermission(Object targetId,String targetType, Object permission)
检验是否有某对象的指定权限(不太明白怎么使用前两个参数来确定具体的对象).


为登陆的用户动态的分配角色：

// 新增Spring Security的Authority（默认已认证通过），账号为corp邮箱，密码为“”
				List<GrantedAuthority> updatedAuthorities = new ArrayList<>();
				updatedAuthorities.add(new SimpleGrantedAuthority(Role.ROLE_ADMIN));
				Authentication newAuth = new UsernamePasswordAuthenticationToken(email, "", updatedAuthorities);
				SecurityContextHolder.getContext().setAuthentication(newAuth);
				request.getSession().setAttribute(SecurityConst.SECURITY_CONTEXTNAME,
						SecurityContextHolder.getContext());//解决多机情况下的问题！

```



2、使用openId认证的过程，以及这个过程出现的一些问题！！



1. Consumer 向 OpenID Server 发起关联请求

   从openID中获取assoiciation然后用于权限认证！！！这个值放在session中，然后供给openID回调的时候进行权限的认证。

2.  Consumer 通知 User Agent 跳转

   openId回调给定的url地址openid.return_to（指定的地址），在回调接口中通过session获取association来进行认证，然后在这个过程中动态的赋予用户角色权限！



这个过程中的问题是：因为后台的服务部署在两台服务器上，是一个多机的形式，所以需要实现session的共享方案！！（每一个浏览器请求带相同cookie，带的sessionID是一样的，把session放在redis中去实现session的共享）！！！！！！，然后就是权限角色的赋予，详细可以看admin的设计过程！！！！！！！！！