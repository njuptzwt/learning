了解一下，什么叫多路复用：

https://www.zhihu.com/question/28594409

自己的理解：多路I/O复用的另一个名字叫做事件驱动模型，这种形式可以大大降低线程的阻塞等开销。

特点：单线程处理问题，需要非阻塞的I/O,高效不需要线程之间的大量切换。



传统的多路请求，SOCKET的通信过程演化：

1、最老土的办法

为每一个用户的请求分配线程进行socket的监听，相应客户端的请求，非常耗开销，只有一个处理器单核的话，每个线程都在抢处理器，还要兼顾线程上下文的切换。

2、单线程，依次轮询每一个请求的socket，这样的效率很慢，而且很有可能，因为一个地方卡住了，影响全部socket连接，很不靠谱

3、I/O多路复用

一条单线程，监听所有的socket是否有连接，如果有socket可以进行处理了，发布事件，线程进行相应的处理，效率很高。



Redis复习的精讲篇：网址

http://zhongmin2012.iteye.com/blog/2424069



redis是单线程，单进程运行的，redis为什么这么快速？

1、redis是内存存储的

2、单线程操作，避免频繁的线程切换

3、非阻塞I/O多路复用，提高了速度



redis的缺点：

1、缓存一致性

2、缓存雪崩，所有缓存在同一个时间失效，然后高并发访问的数据直接访问爆掉数据库

3、缓存击穿，黑客针对无效缓存的数据，直接访问数据库，搞爆数据库

4、key值竞争



redis的删除策略：

redis只能存储5g的数据，但是来了10g的数据，怎么删数据？？？

几个原则：

1、删除最近没使用的key的键值

2、删除标注了过期时间，最近最少使用的key

3、其他方法........





redis学习，redis集群和高可用性，主从复制，哨兵，redis的五种常见使用类型，redis的使用场景！

https://blog.csdn.net/u010648555/article/details/79430105

https://blog.csdn.net/u010648555/article/details/79427608

https://blog.csdn.net/u010648555/article/details/79427606

https://blog.csdn.net/u010648555/article/details/73423560

https://blog.csdn.net/plei_yue/article/details/79362372



redis的用处

https://my.oschina.net/yeevan/blog/609174

redis的五种常见的数据格式

https://blog.csdn.net/xiangwanpeng/article/details/54607181

https://blog.csdn.net/winwill2012/article/details/71717456

redis的常见使用场景：

https://www.jianshu.com/p/d645cebff386



redis的session共享方案：https://blog.csdn.net/u010648555/article/details/79491988

redis的共享session原理，SessionReposityFilter,SessionReposity，过滤链条，数据库存取sessionId