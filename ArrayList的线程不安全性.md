ArrayList的线程不安全性，以及其他iterator的操作不安全性的原因：



modCount==expectModCount，不相等抛出的ConcurrentModificationException

为什么会有这种问题？

多线程的时候：

modCount是一个全局的变量。在使用iterator的迭代的时候，会出现因为多线程并发改变modcount,进一步导致线程安全的问题。



使用CopyOnArrayList是线程安全的！源码里面有解析！



