数据库mysql中的sql注入的定义，statement语句，prestatement语句的用法详解：



sql注入的定义：通过字符串的注入达到SQL语句的拼接，获取数据库的信息



Statement是一个完整的sql语句，数据库JDBC启动连接的时候，执行sql语句的时候，需要把完整的sql语句发送给DBMS进行编译检查在进行数据库的操作。



preStatement是一个预处理语句，他会进行sql语句的检查，有效的提高数据库的安全性，降低sql注入的风险。他会把sql语句提前发送给DBMS进行预处理，然后在数据库中进行语句的缓存，下次可以直接使用编译好的语句进行sql查询等操作。



语句缓存的好处
• ORACLE执行SQL语句时，先将SQL语句的字串通过一个哈希算法得出一个哈希值，然后检查共享池中是否已存在这个哈希值，若有就用已缓存的执行计划来执行这个语句（CACHE HIT 缓存命中），若没有（CACHE MISS 缓存缺失）则需进行解析，解析需要完成下面的工作：

Ø 语法检查；
Ø 语义检查，看参考对象是否存在，类型是否正确；
Ø （如果是CBO优化模式）收集参考对象的统计；
Ø 检查用户的权限是否足够；
Ø 从许多可能的执行路径中选择一条作为执行计划；
Ø 生成语句的编译版本（P-CODE）。

• 解析是一个昂贵的操作，因为过程中需要消耗许多资源；
• 最大化CACHE HIT是调整共享池的目标





顺便引出的是Mybatis的数据库操作语句：

mybatis是使用默认的开启预编译处理的语句。两个过程:

1、会对 sql 进行动态解析，解析为一个 BoundSql 对象，也是在此处对动态 SQL 进行处理的。

mybatis的动态解析包括了：

1、<if test=" "> <then> 等动态语句的解析

2、第二是#{}和${}的解析

会将#{}解析成预编译处理语句中的 select * from user where name = ?;问号

但是${}直接把相应的位置换成了字符串，这一步就会引起sql注入的危险。



2、动态解析完成之后才会进行sql的预编译处理，所以之前的动态解析中有可能会发生sql注入的危险



$符号使用的场景:数据库的分页中使用：

```
SELECT * FROM tb_juwan_item AS t1
JOIN (SELECT id FROM tb_juwan_item ORDER BY id desc LIMIT ${(pagenum-1)*pagesize}, 1) AS t2
WHERE t1.id <![CDATA[ <= ]]> t2.id  ORDER BY t1.id desc LIMIT ${pagesize};
```

相对高效的数据库分页语句。分页查询，并且从相应的id开始查询，效率高！！！