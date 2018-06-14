spring security 的认证分析，从SecurityContext入手！！！源码分析，受益匪浅

https://blog.csdn.net/kaikai8552/article/details/3930747

1、

 Spring Security在认证成功后，将票据存放在SecurityContext中。SecurityContext是一个接口，
从接口的方法可以看出，用户可以通过SecurityContext存放和读取票据信息(Authentication)。

securityContext,安全上下文，他的作用是保存用户的认证信息，授予了何种权限，写入和提取。



2、

 SecurityContext又存放在SecurityContextHolder。（通过阅读源码了解），有效！！！

SecurityContextHolder定义了SecurityContext
的相关操作，如初始化，清空，读取等。SecurityContextHolder并没有直接实现这些操作，而是使用了策略模式，
由一个SecurityContextHolderStrategy接口，来完成真正的逻辑。

![ecurityContextHolde](C:\Users\zhengwentian\Desktop\SecurityContextHolder.png)

initiaze这个方法是初始化，静态方法执行一次，默认是将stratege设置为ThreadLocalSecurityContextHolderStrategy,这种策略下，每个线程关联一个SecurityContext。每次
都从当前线程读取或存放SecurityContext。一般用于为多个用户服务的服务器，每个用户一个线程。是最常用的模式，也是Spring Security缺省的策略。用户注销时，应该把该用户所在线程的SecurityContext清除掉。(需要保存同一个用户的SecurityContext，Session共享的时候！)，项目使用该方法来在session中保存用户的权限认证信息！）

![trateg](C:\Users\zhengwentian\Desktop\strategy.png)


由一个SecurityContextHolderStrategy接口，来完成真正的逻辑。
   Spring Security提供了3个实现：

GlobalSecurityContextHolderStrategy，在这种策略下，JVM所有的对象共享一个SecurityContext对象。一般在胖客户端使用，一个客户端就是一个用户代理。
      

ThreadLocalSecurityContextHolderStrategy，这种策略下，每个线程关联一个SecurityContext。每次都从当前线程读取或存放SecurityContext。一般用于为多个用户服务的服务器，每个用户一个线程。是最常用的模式，也是Spring Security缺省的策略。用户注销时，应该把该用户所在线程的SecurityContext清除掉。

![hreadLocalStrateg](C:\Users\zhengwentian\Desktop\ThreadLocalStrategy.png)



InheritableThreadLocalSecurityContextHolderStrategy，这种策略是InheritableThreadLocal版本的实现。

​      有两种方法可以设置策略：1）用要使用策略的类名作为系统属性spring.security.strategy的值。2）直接调用SecurityContextHolder的静态方法setStrategyName。





本项目中遇到的问题以及解决问题的办法：

SecurityContext保存用户的Authentication，在openID认证完之后赋予的Authention，使用sesison保存用户的Authentication

```
// 新增Spring Security的Authority（默认已认证通过），账号为corp邮箱，密码为“”
List<GrantedAuthority> updatedAuthorities = new ArrayList<>();
updatedAuthorities.add(new SimpleGrantedAuthority(Role.ROLE_ADMIN));
Authentication newAuth = new UsernamePasswordAuthenticationToken(email, "", updatedAuthorities);
SecurityContextHolder.getContext().setAuthentication(newAuth);
request.getSession().setAttribute(SecurityConst.SECURITY_CONTEXTNAME,
      SecurityContextHolder.getContext());
```

然后用户每次请求的时候开启一个新的线程，对SecurityContextHolder重新赋值，在session中取值，SecurityContextHolder该对象是一个ThreadLocal类型!!!!多机的时候，线程不同，这个需要在session中共享！！！