[toc]

## LinkedHashMap源码解析

#### 概述

由于HashMap在多线程情况下使用不当会有线程安全问题。大多数情况下，只要不涉及线程安全问题，Map基本都可以使用HashMap，不过HashMap有一个问题，就是迭代HashMap的顺序并不是HashMap放置的顺序，也就是无序的。HashMap的这个缺点往往会带来一些困扰，因此LinkedHashMap也会氤氲而生。

虽然说增加了时间和空间上的而开销，但是通过维护一个运行于所有条目的双向链表，LinkedHashMap保证了元素迭代的顺序



**四个关注点在LinkedHashMap中的答案**

| 关注点                          | 结论                         |
| ------------------------------- | ---------------------------- |
| LinkedHashMap是否允许键值对为空 | key和value都允许空           |
| LinkedHashMap是否允许重复数据   | key重复会覆盖，Value允许重复 |
| LinkedHashMap是否有序           | 有序                         |
| LinkedHashMap是否线程安全       | 非线程安全                   |
|                                 |                              |



#### LinkedHashMap基本数据类型:

**首先两点：**

- LinkedHashMap可以认为是HashMap+LinkedList，即他即使用了HashMap操作数据结构，有使用LinkedList维护插入元素的先后顺序

-  LinkedHashMap的基本实现思想就是----**多态**。可以说，理解多态，再去理解LinkedHashMap原理会事半功倍；反之也是，对于LinkedHashMap原理的学习，也可以促进和加深对于多态的理解。 

  ```java
  public class LinkedHashMap<K,V> extends	HashMap<K,V> implements	Map<K,V>{
  	...
  }
  ```

  由于LinkedHashMap是HashMap的子类，自然LinkedHashMap也就继承了HashMap的所有非私有的方法。再看看LInkedHashMap中本身的方法：

  

 ![img](https://images2015.cnblogs.com/blog/801753/201512/801753-20151216205748224-1556969372.png) 

由此可见LinkedHashMap中并没有什么操作数据结构的方法，也就是及说LinkedHashMap操作数据结构，和HashMap操作数据结构的方法完全一样，无非就是一些实现细节的不同



LinkedHashMap的基本数据结构，也就是Entry:

```JAVA
private static class Entry<K,V> extends HashMap.Entry<K,V> {
    // These fields comprise the doubly linked list used for iteration.
    Entry<K,V> before, after;

Entry(int hash, K key, V value, HashMap.Entry<K,V> next) {
        super(hash, key, value, next);
    }
    ...
}
```

Entry里面的一些属性：

- **K key**
- **V value**
- **Entry next**
- **int hash**
- **Entry before**
- **Entry after**

其中前四个是从HashMap.Entry中继承而来的，后面两个是LInkedHashMap独有的。不要搞错了next和before，After，**next是用于维护HashMap指定table位置上连接的Entry的顺序的，before,After是用来维护Entry插入的先后顺序的**

 ![img](https://images2015.cnblogs.com/blog/801753/201512/801753-20151216211941693-590937713.png) 

