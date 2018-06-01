redis如何实现缓存一致性原理：

有一种方法是这样实现的：



当你的缓存数据发生更改的时候，先把对应的缓存清理，之后数据库查询的时候，会重新创建redis的key来实现数据的重新缓存。



@Cacheable

@CacheEvict

@Cacheput

几个注解结合起来使用，通过SpringBoot的Redis来实现缓存，没有使用Java的Jedis来实现。

