zipkin分布式服务链路追踪，其中zipkin的作用和实习过程中的难点！

最大难点：数据埋点！即trceID和spanID的传递，span的记录，http无状态



zipkin跟踪一个request的完整流程是一个trceId,一个完整的请求过程中会有多个span，一个span是分布式服务链路追踪的基本服务单元。

一个span的基本结构如下：

traceId: 一个请求链路跟踪的完整过程，唯一值

id: spanId，唯一值

parentId: 上一个span

annotations:(数据埋点的位置，所谓的数据埋点就是需要记录和传递traceId和spanId的信息)

（用途：用于定位一个request的开始和结束，cs/sr/ss/cr含有额外的信息，比如说时间点）
                   cs：Client Start,表示客户端发起请求
                          一个span的开始
                   sr：Server Receive,表示服务端收到请求
                   ss：Server Send,表示服务端完成处理，并将结果发送给客户端
                   cr：Client Received,表示客户端获取到服务端返回信息
                         一个span的结束
                         当这个annotation被记录了，这个RPC也被认为完成了
 BinaryAnnotation：（用途：提供一些额外信息，一般已key-value对出现）
  Span：一个请求（包含一组Annotation和BinaryAnnotation）；它是基本工作单元，一次链路调用(可以是RPC，DB等没有特定的限制)创建一个span，通过一个64位ID标识它。 span通常还有其他的数据，例如描述信息，时间戳，key-value对的(Annotation)tag信息，parent-id等,其中parent-id可以表示span调用链路来源，通俗的理解span就是一次请求信息。
 Trace：类似于树结构的Span集合，表示一条调用链路，存在唯一标识
                  Traces are built by collecting all Spans that share a traceId
                  通过traceId、spanId和parentId，被收集到的span会汇聚成一个tree，从而提供出一个request的整体流程。（这也是zipkin的工作原理）

```
{
    "traceId": "ec2b9331680312af",
    "id": "0a3fba79fceef5d1",
    "name": "http:/juwan-sync/yxpay/regneworder",
    "parentId": "358ab74ec7d99d4c",
    "timestamp": 1532414139511000,
    "duration": 20000,
    "annotations": [
      {
        "timestamp": 1532414139511000,
        "value": "cs",
        "endpoint": {
          "serviceName": "juwan-zuul",
          "ipv4": "10.200.179.131",
          "port": 8971
        }
      },
      {
        "timestamp": 1532414139511500,
        "value": "sr",
        "endpoint": {
          "serviceName": "juwan-sync",
          "ipv4": "10.200.179.134",
          "port": 8871
        }
      },
      {
        "timestamp": 1532414139530500,
        "value": "ss",
        "endpoint": {
          "serviceName": "juwan-sync",
          "ipv4": "10.200.179.134",
          "port": 8871
        }
      },
      {
        "timestamp": 1532414139531000,
        "value": "cr",
        "endpoint": {
          "serviceName": "juwan-zuul",
          "ipv4": "10.200.179.131",
          "port": 8971
        }
      }
    ],
    "binaryAnnotations": [
      {
        "key": "http.method",
        "value": "GET",
        "endpoint": {
          "serviceName": "juwan-zuul",
          "ipv4": "10.200.179.131",
          "port": 8971
        }
      },
      {
        "key": "http.path",
        "value": "/yxPay/regNewOrder",
        "endpoint": {
          "serviceName": "juwan-zuul",
          "ipv4": "10.200.179.131",
          "port": 8971
        }
      },
      {
        "key": "http.status_code",
        "value": "200",
        "endpoint": {
          "serviceName": "juwan-zuul",
          "ipv4": "10.200.179.131",
          "port": 8971
        }
      },
      {
        "key": "http.url",
        "value": "/yxPay/regNewOrder",
        "endpoint": {
          "serviceName": "juwan-zuul",
          "ipv4": "10.200.179.131",
          "port": 8971
        }
      },
      {
        "key": "lc",
        "value": "zuul",
        "endpoint": {
          "serviceName": "juwan-zuul",
          "ipv4": "10.200.179.131",
          "port": 8971
        }
      },
      {
        "key": "mvc.controller.class",
        "value": "PayController",
        "endpoint": {
          "serviceName": "juwan-sync",
          "ipv4": "10.200.179.134",
          "port": 8871
        }
      },
      {
        "key": "mvc.controller.method",
        "value": "regNewOrder",
        "endpoint": {
          "serviceName": "juwan-sync",
          "ipv4": "10.200.179.134",
          "port": 8871
        }
      },
      {
        "key": "spring.instance_id",
        "value": "juwan-dg179-134.yx.hz.infra.mail:juwan-sync:8871",
        "endpoint": {
          "serviceName": "juwan-sync",
          "ipv4": "10.200.179.134",
          "port": 8871
        }
      },
      {
        "key": "spring.instance_id",
        "value": "juwan-dg179-131.yx.hz.infra.mail:juwan-zuul:8971",
        "endpoint": {
          "serviceName": "juwan-zuul",
          "ipv4": "10.200.179.131",
          "port": 8971
        }
      }
    ]
  }
```

*Dapper*论文中对zipkin的论文：![ipki](C:\Users\zhengwentian\Desktop\zipkin.png)

该图片是zipkin的设计原理图。

其中zipkin是严格按照来Dapper论文来设计的，他提供了完整的跟踪记录收集、存储功能，以及查询API与界面。其存储支持多种数据库：MySql、ElasticSearch、Cassandra、Redis等等，**收集API支持HTTP和Thrift**。一个span的完整过程：

![pan 结](C:\Users\zhengwentian\Desktop\Span 结构.png)

一个完整的trace:  cs-sr-ss-cr,过程这其中的计算！

![PM跟踪过程与数据传](C:\Users\zhengwentian\Desktop\APM跟踪过程与数据传输.png)

一个服务追踪的例子，和实际的解析图：

![pan 流](C:\Users\zhengwentian\Desktop\span 流转.png)

设计中的难点问题：



1、traceID是一个64位的bit位的标志，需要特点实现！

http://sharecore.net/2017/03/09/RPC%E6%9C%8D%E5%8A%A1%E8%BF%BD%E8%B8%AA%E7%9A%84%E5%8E%9F%E7%90%86%E4%B8%8E%E5%AE%9E%E8%B7%B5/



2、traceId和spanId的数据埋点，每个服务中如何保存这些信息，如何保存到下一个rpc服务span中！

zipkin中是这样实现的，使用线程上下文保存traceId和spanId，ThreadLocal变量，保存，但是如果是异步线程启用RPC服务，那么线程信息不能复制，所以使用了InheritThreadLocal变量来实现这个。

缺点：不能用线程池执行，线程上下文信息会丢失，只能去修改线程池的run方法进一步修改

如何实现traceId和spanId在服务调用之间的传递：

HTTP方式：使用request的header设计

Thrift：zipkin没有默认实现，通过研究Thrift代码，发现在Thrift的传输协议实现里，服务端读取数据反序列化协议的入口方法是：
`public abstract TMessage readMessageBegin() throws TException;`

http://sharecore.net/2017/03/09/RPC%E6%9C%8D%E5%8A%A1%E8%BF%BD%E8%B8%AA%E7%9A%84%E5%8E%9F%E7%90%86%E4%B8%8E%E5%AE%9E%E8%B7%B5/



参考资料和文献：

https://cloud.tencent.com/developer/article/1025966

http://sharecore.net/2017/03/09/RPC%E6%9C%8D%E5%8A%A1%E8%BF%BD%E8%B8%AA%E7%9A%84%E5%8E%9F%E7%90%86%E4%B8%8E%E5%AE%9E%E8%B7%B5/

https://www.google.co.jp/search?q=zipkin%E4%BC%9A%E6%B6%88%E8%80%97%E5%86%85%E5%AD%98%E5%98%9B%EF%BC%9F&oq=zipkin%E4%BC%9A%E6%B6%88%E8%80%97%E5%86%85%E5%AD%98%E5%98%9B%EF%BC%9F&aqs=chrome..69i57.12712j0j7&sourceid=chrome&ie=UTF-8