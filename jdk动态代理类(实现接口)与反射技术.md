动态代理类的实现：动态代理的作用，增强作用，底层的实现原理是反射技术。

动态代理是一个新的类，它实现的时候是在运行过程中生成的



动态代理的实现步骤归纳总结如下：



第一个：需要一个接口，该接口实现的是所有主类的方法



第二个：被代理类实现自己的相应的方法



第三个需要实现一个InvocationHandler类，该类中包含被代理类对象，结构如下：



public class HelloWorldHandler implments InvocationHandler{

public Object object;

public HelloWorldHandler(Object obj) {

​        super();

​        this.obj = obj;

 }



// 代理类，使用反射技巧实现动态代理的关键地方

public Object invoke(Object proxy, Method method, Object[] args) throws Throwable

// 动态增强



}



第四个：生成一个代理类

 HelloWorld helloWorld=new HelloWorldImpl();// 生成被代理对象

  InvocationHandler handler=new HelloWorldHandler(helloWorld);// 生成代理类实际执行的控制器

  //创建动态代理对象

 HelloWorld proxy=(HelloWorld)Proxy.newProxyInstance(helloWorld.getClass().getClassLoader(),helloWorld.getClass().getInterfaces(), handler);

// 动态代理调用对象

proxy.sayHelloWorld();

