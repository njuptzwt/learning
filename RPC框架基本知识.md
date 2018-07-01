RPC框架和Spring-Cloud微服务治理：



一、何为RPC

远程服务调用，也就是说两个服务在不同的服务器上，互相访问的时候涉及到网络的连接，http请求，tcp,socket比较复杂。RPC调用的时候就会涉及到两个问题：

第一个问题，两台服务器通信，是直接通过ip+port访问还是根据服务名字访问，

第二个问题：通信的时候对于调用的函数，函数的参数如何传递参数，传递参数涉及到序列化和反序列化。

开源的RPC框架帮我们解决了这种服务调用的麻烦，主流的RPC框架：

1、阿里的Dubbo

2、spring cloud netflix (Eureka的服务治理)

​    a、spring cloud ribbon 结合 RestTemplate实现远程rpc调用

​    b、spring cloud Feign 接口模式的rpc调用

3、OSP（唯品会的rpc框架)





二、RPC和Spring-Cloud的区别联系

spring cloud netflix 使用eureka进行服务治理，eureka server服务注册中心，其他的服务为eureka client。各个服务注册的时候，会把service instance 发送到注册中心注册，service-instance带有服务的IP+PORT,服务发现和负载均衡的时候会使用。

1、spring cloud ribbon+loadBalanceClient+RestTemplate,实现按照服务名来进行RPC调用，ribbon实现负载均衡，负载均衡策略由轮询，权重，随机等

2、spring cloud feign,feign使用ribbon做负载均衡，使用了hystrix来做服务熔断，使用接口形式来进行远程RPC调用

三、阿里的Dubbo和Spring-Cloud的区别联系

1、dubbo使用zookeeper来进行服务治理，使用服务名进行rpc访问，spring-cloud使用eureka来进行服务治理

2、dubbo没有成熟的熔断机制，没有服务网关。spring-cloud有熔断机制和，spring-cloud-zuul来进行服务网关。

