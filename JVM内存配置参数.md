servlet在多线程下是线程不安全的！

https://www.cnblogs.com/chanshuyi/p/5052426.html

核心：servlet是单例模式



# JVM内存配置参数

-Xmx10240m -Xms10240m -Xmn5120m -XXSurvivorRatio=3

解析：

-Xmx：最大堆大小

-Xms：初始堆大小

-Xmn:年轻代大小

-XXSurvivorRatio：年轻代中Eden区与Survivor区的大小比值

年轻代5120m， Eden：Survivor=3，Survivor区大小=1024m（Survivor区有两个，即将年轻代分为5份，每个Survivor区占一份），总大小为2048m。

-Xms初始堆大小即最小内存值为10240m





java 程序的优化，优化GC垃圾回收机制！！！获取最优的性能！





配置tomcat调用的虚拟机内存大小，和配置java内存参数是一样的！