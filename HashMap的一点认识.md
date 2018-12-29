纠正之前一直错误的认识：

HashMap线程不安全的根本原因是？ put()和get()操作并没有做同步处理

比如说去统计一个接口的访问次数：Map<String,Integer> count=new HashMap<>();

先做count.get();再amount++;再count.put()，会出现线程问题，会使得次数减少，高并发的时候没有做线程安全控制。

而：resize()出现的死锁问题只是一个比较严重的问题，hashmap1.7中没有进行处理，但是1.8中进行了处理

HashMap知识：1.7中HashCode(),每次都要计算hash()值，然后底层是实现数组+链表的方式实现的

有容量因子：0.75

capacity:16 默认

问题1、会出现resize()高并发的时候，链表的环状，这样的话，取key的时候会死循环，程序崩掉

问题2、如果hashmap分配不均匀的话，会出现某个bucket链特别长，那么会出现的是O(N)复杂度

http://www.importnew.com/27043.html

http://www.importnew.com/20386.html

https://blog.csdn.net/wushiwude/article/details/75331926

针对问题1：

1.7中使用分段锁的机制来，控制高并发的问题，排除链表的回环

针对问题2：

1.8中使用红黑树来实现Log(N)的时间复杂度来实现，当某个bucket的长度大于8之后，会将链表转化成红黑树，当红黑树的个数少于6之后会使用红黑树转化为链表。当整个数组的长度大于64的时候才能用链表转化为红黑树，而小于64的时候进行扩容

******1.8中的数组扩容，resize不会出现链表的回环问题，而且不需要重新计算hash值（这都是由每次扩容的的时候是2的幂次方，这样的话可以用相与运算这种方式来做，而且不会出现回环的问题)

看看源码：

```
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```





可以学习一下：CocurrentHashMap的知识：