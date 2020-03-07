[toc]

### LinkedHashMap

#### 概述：

 HashMap 是 Java Collection Framework 的重要成员，也是Map族(如下图所示)中我们最为常用的一种。不过遗憾的是，HashMap是无序的，也就是说，迭代HashMap所得到的元素顺序并不是它们最初放置到HashMap的顺序。HashMap的这一缺点往往会造成诸多不便，因为在有些场景中，我们确需要用到一个可以保持插入顺序的Map。庆幸的是，JDK为我们解决了这个问题，它为HashMap提供了一个子类 —— LinkedHashMap。虽然LinkedHashMap增加了时间和空间上的开销，但是它通过维护一个额外的双向链表保证了迭代顺序。特别地，该迭代顺序可以是插入顺序，也可以是访问顺序。因此，根据链表中元素的顺序可以将LinkedHashMap分为：保持插入顺序的LinkedHashMap 和 保持访问顺序的LinkedHashMap，其中LinkedHashMap的默认实现是按插入顺序排序的。

 ![1](C:\Users\Lenovo\Desktop\java\1.png)



 

本质上，HashMap和双向链表合二为一即是LinkedHashMap。所谓LinkedHashMap，其落脚点在HashMap，因此更准确地说，它是一个将所有Entry节点链入一个双向链表双向链表的HashMap。在LinkedHashMapMap中，所有put进来的Entry都保存在如下面第一个图所示的哈希表中，但由于它又额外定义了一个以head为头结点的双向链表(如下面第二个图所示)，因此对于每次put进来Entry，除了将其保存到哈希表中对应的位置上之外，还会将其插入到双向链表的尾部。



![2](C:\Users\Lenovo\Desktop\java\2.png)

![3](C:\Users\Lenovo\Desktop\java\3.jpeg)

　　　 更直观地，下图很好地还原了LinkedHashMap的原貌：HashMap和双向链表的密切配合和分工合作造就了LinkedHashMap。特别需要注意的是，**next用于维护HashMap各个桶中的Entry链**，**before、after用于维护LinkedHashMap的双向链表，虽然它们的作用对象都是Entry，但是各自分离，是两码事儿**。 　　　　　　　

 

 ![彻头彻尾理解 LinkedHashMap](https://www.javazhiyin.com/wp-content/uploads/2019/03/java6-1553058540.jpg) 



其中，HashMap与LinkedHashMap的Entry结构示意图如下图所示：

　　　　![彻头彻尾理解 LinkedHashMap](https://www.javazhiyin.com/wp-content/uploads/2019/03/java10-1553058540.png)

 

 特别地，由于LinkedHashMap是HashMap的子类，所以LinkedHashMap自然会拥有HashMap的所有特性。比如，LinkedHashMap也最多只允许一条Entry的键为Null(多条会覆盖)，但允许多条Entry的值为Null。此外，LinkedHashMap 也是 Map 的一个非同步的实现。此外，LinkedHashMap还可以用来实现LRU (Least recently used, 最近最少使用)算法，这个问题会在下文的特别谈到。 

#### 二. LinkedHashMap 在 JDK 中的定义

##### 1、类结构定义

LinkedHashMap继承于HashMap，其在JDK中的定义为：

| `1 2 3 4 5 6 ` | `public class LinkedHashMap extends HashMap implements Map {     ... }` |
| -------------- | ------------------------------------------------------------ |
|                |                                                              |

##### 2、成员变量定义

与HashMap相比，LinkedHashMap增加了两个属性用于保证迭代顺序，分别是 双向链表头结点header 和 标志位accessOrder (值为true时，表示按照访问顺序迭代；值为false时，表示按照插入顺序迭代)。

| `1 2 3 4 5 6 7 8 9 10 11 12 ` | `/**     * The head of the doubly linked list.     */ private transient Entry header;  // 双向链表的表头元素  /**     * The iteration ordering method for this linked hash map: true     * for access-order, false for insertion-order.     *     * @serial     */ private final boolean accessOrder;  //true表示按照访问顺序迭代，false时表示按照插入顺序` |
| ----------------------------- | ------------------------------------------------------------ |
|                               |                                                              |

##### 3、成员方法定义

从下图我们可以看出，LinkedHashMap中并增加没有额外方法。也就是说，LinkedHashMap与HashMap在操作上大致相同，只是在实现细节上略有不同罢了。

　　　　　　　　　　　　　　![彻头彻尾理解 LinkedHashMap](https://www.javazhiyin.com/wp-content/uploads/2019/03/java0-1553058540.png)

 

##### 4、基本元素 Entry

LinkedHashMap采用的hash算法和HashMap相同，但是它重新定义了Entry。LinkedHashMap中的Entry增加了两个指针 before 和 after，它们分别用于维护双向链接列表。特别需要注意的是，next用于维护HashMap各个桶中Entry的连接顺序，before、after用于维护Entry插入的先后顺序的，源代码如下：

| `1 2 3 4 5 6 7 8 9 10 ` | `private static class Entry extends HashMap.Entry {  // These fields comprise the doubly linked list used for iteration.    Entry before, after;     Entry(int hash, K key, V value, HashMap.Entry next) { super(hash, key, value, next);    }    ... }` |
| ----------------------- | ------------------------------------------------------------ |
|                         |                                                              |

形象地，HashMap与LinkedHashMap的Entry结构示意图如下图所示：

**![彻头彻尾理解 LinkedHashMap](https://www.javazhiyin.com/wp-content/uploads/2019/03/java7-1553058540.png)**

 

#### 三. LinkedHashMap 的构造函数

LinkedHashMap 一共提供了五个构造函数，它们都是在HashMap的构造函数的基础上实现的，分别如下：

 

1、LinkedHashMap()

该构造函数意在构造一个具有 默认初始容量 (16)和默认负载因子(0.75)的空 LinkedHashMap，是 Java Collection Framework 规范推荐提供的，其源码如下：

```JAVA
/**
     * Constructs an empty insertion-ordered <tt>LinkedHashMap</tt> instance
     * with the default initial capacity (16) and load factor (0.75).
     */
public LinkedHashMap() {
        super();  // 调用HashMap对应的构造函数
        accessOrder = false;           // 迭代顺序的默认值
    }
```



2、LinkedHashMap(int initialCapacity, float loadFactor)

该构造函数意在构造一个指定初始容量和指定负载因子的空 LinkedHashMap，其源码如下：

```java
/**
     * Constructs an empty insertion-ordered <tt>LinkedHashMap</tt> instance
     * with the specified initial capacity and load factor.
     *
     * @param  initialCapacity the initial capacity
     * @param  loadFactor      the load factor
     * @throws IllegalArgumentException if the initial capacity is negative
     *         or the load factor is nonpositive
     */

public LinkedHashMap(int initialCapacity, float loadFactor) {
super(initialCapacity, loadFactor);      // 调用HashMap对应的构造函数
        accessOrder = false;            // 迭代顺序的默认值
    }
```



3、LinkedHashMap(int initialCapacity)

该构造函数意在构造一个指定初始容量和默认负载因子 (0.75)的空 LinkedHashMap，其源码如下：

```java
/**
     * Constructs an empty insertion-ordered <tt>LinkedHashMap</tt> instance
     * with the specified initial capacity and a default load factor (0.75).
     *
     * @param  initialCapacity the initial capacity
     * @throws IllegalArgumentException if the initial capacity is negative
     */
public LinkedHashMap(int initialCapacity) {
super(initialCapacity);  // 调用HashMap对应的构造函数
        accessOrder = false;     // 迭代顺序的默认值
    }
```



4、LinkedHashMap(Map<? extends K, ? extends V> m)

该构造函数意在构造一个与指定 Map 具有相同映射的 LinkedHashMap，其 初始容量不小于 16 (具体依赖于指定Map的大小)，负载因子是 0.75，是 Java Collection Framework 规范推荐提供的，其源码如下：

```java
/**
     * Constructs an insertion-ordered <tt>LinkedHashMap</tt> instance with
     * the same mappings as the specified map.  The <tt>LinkedHashMap</tt>
     * instance is created with a default load factor (0.75) and an initial
     * capacity sufficient to hold the mappings in the specified map.
     *
     * @param  m the map whose mappings are to be placed in this map
     * @throws NullPointerException if the specified map is null
     */
public LinkedHashMap(Map<? extends K, ? extends V> m) {
super(m);       // 调用HashMap对应的构造函数
        accessOrder = false;    // 迭代顺序的默认值
    }
```



5、LinkedHashMap(int initialCapacity, float loadFactor, boolean accessOrder)

该构造函数意在构造一个指定初始容量和指定负载因子的具有指定迭代顺序的LinkedHashMap，其源码如下：

```java
/**
     * Constructs an empty <tt>LinkedHashMap</tt> instance with the
     * specified initial capacity, load factor and ordering mode.
     *
     * @param  initialCapacity the initial capacity
     * @param  loadFactor      the load factor
     * @param  accessOrder     the ordering mode - <tt>true</tt> for
     *         access-order, <tt>false</tt> for insertion-order
     * @throws IllegalArgumentException if the initial capacity is negative
     *         or the load factor is nonpositive
     */
public LinkedHashMap(int initialCapacity,
float loadFactor,
boolean accessOrder) {
super(initialCapacity, loadFactor);   // 调用HashMap对应的构造函数
this.accessOrder = accessOrder;    // 迭代顺序的默认值
    }
```



正如我们在《Map 综述（一）：彻头彻尾理解 HashMap》一文中提到的那样，初始容量 和负载因子是影响HashMap性能的两个重要参数。同样地，它们也是影响LinkedHashMap性能的两个重要参数。此外，LinkedHashMap 增加了双向链表头结点 header和标志位 accessOrder两个属性用于保证迭代顺序。

 

##### 6、init 方法

从上面的五种构造函数我们可以看出，无论采用何种方式创建LinkedHashMap，其都会调用HashMap相应的构造函数。事实上，不管调用HashMap的哪个构造函数，HashMap的构造函数都会在最后调用一个init()方法进行初始化，只不过这个方法在HashMap中是一个空实现，而**在LinkedHashMap中重写了它用于初始化它所维护的双向链表**。例如，HashMap的参数为空的构造函数以及init方法的源码如下：

```java
/**
     * Constructs an empty <tt>HashMap</tt> with the default initial capacity
     * (16) and the default load factor (0.75).
     */
public HashMap() {
this.loadFactor = DEFAULT_LOAD_FACTOR;
        threshold = (int)(DEFAULT_INITIAL_CAPACITY * DEFAULT_LOAD_FACTOR);
        table = new Entry[DEFAULT_INITIAL_CAPACITY];
        init();
    }
 
/**
     * Initialization hook for subclasses. This method is called
     * in all constructors and pseudo-constructors (clone, readObject)
     * after HashMap has been initialized but before any entries have
     * been inserted.  (In the absence of this method, readObject would
     * require explicit knowledge of subclasses.)
     */
void init() {
    }
```



**在LinkedHashMap中，它重写了init方法以便初始化双向列表**，源码如下：

```JAVA
/**
     * Called by superclass constructors and pseudoconstructors (clone,
     * readObject) before any entries are inserted into the map.  Initializes
     * the chain.
     */
void init() {
        header = new Entry<K,V>(-1, null, null, null);
        header.before = header.after = header;
    }
```



因此，我们在创建LinkedHashMap的同时就会不知不觉地对双向链表进行初始化。

 

#### 四. LinkedHashMap 的数据结构

本质上，**LinkedHashMap = HashMap + 双向链表**，也就是说，HashMap和双向链表合二为一即是LinkedHashMap。也可以这样理解，LinkedHashMap 在不对HashMap做任何改变的基础上，给HashMap的任意两个节点间加了两条连线(before指针和after指针)，使这些节点形成一个双向链表。在LinkedHashMapMap中，所有put进来的Entry都保存在HashMap中，但由于它又额外定义了一个以head为头结点的空的双向链表，因此对于每次put进来Entry还会将其插入到双向链表的尾部。

 

![彻头彻尾理解 LinkedHashMap](https://www.javazhiyin.com/wp-content/uploads/2019/03/java8-1553058541.jpg)

　　　　　　

#### 五. LinkedHashMap 的快速存取

我们知道，在HashMap中最常用的两个操作就是：put(Key,Value) 和 get(Key)。同样地，在 LinkedHashMap 中最常用的也是这两个操作。对于put(Key,Value)方法而言，LinkedHashMap完全继承了HashMap的 put(Key,Value) 方法，只是对put(Key,Value)方法所调用的recordAccess方法和addEntry方法进行了重写；对于get(Key)方法而言，LinkedHashMap则直接对它进行了重写。下面我们结合JDK源码看 LinkedHashMap 的存取实现。

 

##### 1、存储实现 : put(key, vlaue)

上面谈到，LinkedHashMap没有对 put(key,vlaue) 方法进行任何直接的修改，完全继承了HashMap的 put(Key,Value) 方法，其源码如下：

```JAVA
/**
     * Associates the specified value with the specified key in this map.
     * If the map previously contained a mapping for the key, the old
     * value is replaced.
     *
     * @param key key with which the specified value is to be associated
     * @param value value to be associated with the specified key
     * @return the previous value associated with key, or null if there was no mapping for key.
     *  Note that a null return can also indicate that the map previously associated null with key.
     */
public V put(K key, V value) {
 
//当key为null时，调用putForNullKey方法，并将该键值对保存到table的第一个位置 
if (key == null)
return putForNullKey(value); 
 
//根据key的hashCode计算hash值
int hash = hash(key.hashCode());           
 
//计算该键值对在数组中的存储位置（哪个桶）
int i = indexFor(hash, table.length);              
 
//在table的第i个桶上进行迭代，寻找 key 保存的位置
for (Entry<K,V> e = table[i]; e != null; e = e.next) {      
            Object k;
//判断该条链上是否存在hash值相同且key值相等的映射，若存在，则直接覆盖 value，并返回旧value
if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this); // LinkedHashMap重写了Entry中的recordAccess方法--- (1)    
return oldValue;    // 返回旧值
            }
        }
 
        modCount++; //修改次数增加1，快速失败机制
 
//原Map中无该映射，将该添加至该链的链头
        addEntry(hash, key, value, i);  // LinkedHashMap重写了HashMap中的createEntry方法 ---- (2)    
return null;
    }
```



上述源码反映了LinkedHashMap与HashMap保存数据的过程。特别地，在LinkedHashMap中，它对addEntry方法和Entry的recordAccess方法进行了重写。下面我们对比地看一下LinkedHashMap 和HashMap的addEntry方法的具体实现：

```JAVA
/**
     * This override alters behavior of superclass put method. It causes newly
     * allocated entry to get inserted at the end of the linked list and
     * removes the eldest entry if appropriate.
     *
     * LinkedHashMap中的addEntry方法
     */
void addEntry(int hash, K key, V value, int bucketIndex) {   
 
//创建新的Entry，并插入到LinkedHashMap中  
        createEntry(hash, key, value, bucketIndex);  // 重写了HashMap中的createEntry方法
 
//双向链表的第一个有效节点（header后的那个节点）为最近最少使用的节点，这是用来支持LRU算法的
        Entry<K,V> eldest = header.after;  
//如果有必要，则删除掉该近期最少使用的节点，  
//这要看对removeEldestEntry的覆写,由于默认为false，因此默认是不做任何处理的。  
if (removeEldestEntry(eldest)) {  
            removeEntryForKey(eldest.key);  
        } else {  
//扩容到原来的2倍  
if (size >= threshold)  
                resize(2 * table.length);  
        }  
    } 
 
    -------------------------------我是分割线------------------------------------
 
/**
     * Adds a new entry with the specified key, value and hash code to
     * the specified bucket.  It is the responsibility of this
     * method to resize the table if appropriate.
     *
     * Subclass overrides this to alter the behavior of put method.
     * 
     * HashMap中的addEntry方法
     */
void addEntry(int hash, K key, V value, int bucketIndex) {
//获取bucketIndex处的Entry
        Entry<K,V> e = table[bucketIndex];
 
//将新创建的 Entry 放入 bucketIndex 索引处，并让新的 Entry 指向原来的 Entry 
        table[bucketIndex] = new Entry<K,V>(hash, key, value, e);
 
//若HashMap中元素的个数超过极限了，则容量扩大两倍
if (size++ >= threshold)
            resize(2 * table.length);
    }
```



由于LinkedHash**Map本身维护了插入的先后顺序，因此其可以用来做缓存，14~19行的操作就是用来支持LRU算法的，这里暂时不用去关心它**。**此外，在LinkedHashMap的addEntry方法中，它重写了HashMap中的createEntry方法，我们接着看一下createEntry方法**：

```JAVA
oid createEntry(int hash, K key, V value, int bucketIndex) { 
// 向哈希表中插入Entry，这点与HashMap中相同 
//创建新的Entry并将其链入到数组对应桶的链表的头结点处， 
        HashMap.Entry<K,V> old = table[bucketIndex];  
        Entry<K,V> e = new Entry<K,V>(hash, key, value, old);  
        table[bucketIndex] = e;     
 
//在每次向哈希表插入Entry的同时，都会将其插入到双向链表的尾部，  
//这样就按照Entry插入LinkedHashMap的先后顺序来迭代元素(LinkedHashMap根据双向链表重写了迭代器)
//同时，新put进来的Entry是最近访问的Entry，把其放在链表末尾 ，也符合LRU算法的实现  
        e.addBefore(header);  
        size++;  
    }
```



由以上源码我们可以知道，在LinkedHashMap中向哈希表中插入新Entry的同时，还会通过Entry的addBefore方法将其链入到双向链表中。其中，addBefore方法本质上是一个双向链表的插入操作，其源码如下：

```JAVA
//在双向链表中，将当前的Entry插入到existingEntry(header)的前面  
private void addBefore(Entry<K,V> existingEntry) {  
        after  = existingEntry;  
        before = existingEntry.before;  
        before.after = this;  
        after.before = this;  
    }
```



到此为止，我们分析了在LinkedHashMap中put一条键值对的完整过程。总的来说，相比HashMap而言，LinkedHashMap在向哈希表添加一个键值对的同时，也会将其链入到它所维护的双向链表中，以便设定迭代顺序。

 

##### 2、扩容操作 : resize()

在HashMap中，我们知道随着HashMap中元素的数量越来越多，发生碰撞的概率将越来越大，所产生的子链长度就会越来越长，这样势必会影响HashMap的存取速度。为了保证HashMap的效率，系统必须要在某个临界点进行扩容处理，该临界点就是HashMap中元素的数量在数值上等于threshold（table数组长度*加载因子）。但是，不得不说，扩容是一个非常耗时的过程，因为它需要重新计算这些元素在新table数组中的位置并进行复制处理。所以，如果我们能够提前预知HashMap 中元素的个数，那么在构造HashMap时预设元素的个数能够有效的提高HashMap的性能。

　　

同样的问题也存在于LinkedHashMap中，因为LinkedHashMap本来就是一个HashMap，只是它还将所有Entry节点链入到了一个双向链表中。**LinkedHashMap完全继承了HashMap的resize()方法，只是对它所调用的transfer方法进行了重写**。我们先看resize()方法源码：

```JAVA
/**
     * Rehashes the contents of this map into a new array with a
     * larger capacity.  This method is called automatically when the
     * number of keys in this map reaches its threshold.
     *
     * If current capacity is MAXIMUM_CAPACITY, this method does not
     * resize the map, but sets threshold to Integer.MAX_VALUE.
     * This has the effect of preventing future calls.
     *
     * @param newCapacity the new capacity, MUST be a power of two;
     *        must be greater than current capacity unless current
     *        capacity is MAXIMUM_CAPACITY (in which case value
     *        is irrelevant).
     */
void resize(int newCapacity) {
        Entry[] oldTable = table;
int oldCapacity = oldTable.length;
 
// 若 oldCapacity 已达到最大值，直接将 threshold 设为 Integer.MAX_VALUE
if (oldCapacity == MAXIMUM_CAPACITY) {  
            threshold = Integer.MAX_VALUE;
return;             // 直接返回
        }
 
// 否则，创建一个更大的数组
        Entry[] newTable = new Entry[newCapacity];
 
//将每条Entry重新哈希到新的数组中
        transfer(newTable);  //LinkedHashMap对它所调用的transfer方法进行了重写
 
        table = newTable;
        threshold = (int)(newCapacity * loadFactor);  // 重新设定 threshold
    }
```



从上面代码中我们可以看出，Map扩容操作的核心在于重哈希。所谓重哈希是指重新计算原HashMap中的元素在新table数组中的位置并进行复制处理的过程。鉴于性能和LinkedHashMap自身特点的考量，LinkedHashMap对重哈希过程(transfer方法)进行了重写，源码如下：

```JAVA
/**
     * Transfers all entries to new table array.  This method is called
     * by superclass resize.  It is overridden for performance, as it is
     * faster to iterate using our linked list.
     */
void transfer(HashMap.Entry[] newTable) {
int newCapacity = newTable.length;
// 与HashMap相比，借助于双向链表的特点进行重哈希使得代码更加简洁
for (Entry<K,V> e = header.after; e != header; e = e.after) {
int index = indexFor(e.hash, newCapacity);   // 计算每个Entry所在的桶
// 将其链入桶中的链表
            e.next = newTable[index];
            newTable[index] = e;   
        }
    }
```



如上述源码所示，LinkedHashMap借助于自身维护的双向链表轻松地实现了重哈希操作。

 

##### 3、读取实现 ：get(Object key)

相对于LinkedHashMap的存储而言，读取就显得比较简单了。LinkedHashMap中重写了HashMap中的get方法，源码如下：

```JAVA
/**
     * Returns the value to which the specified key is mapped,
     * or {@code null} if this map contains no mapping for the key.
     *
     * <p>More formally, if this map contains a mapping from a key
     * {@code k} to a value {@code v} such that {@code (key==null ? k==null :
     * key.equals(k))}, then this method returns {@code v}; otherwise
     * it returns {@code null}.  (There can be at most one such mapping.)
     *
     * <p>A return value of {@code null} does not <i>necessarily</i>
     * indicate that the map contains no mapping for the key; it's also
     * possible that the map explicitly maps the key to {@code null}.
     * The {@link #containsKey containsKey} operation may be used to
     * distinguish these two cases.
     */
public V get(Object key) {
// 根据key获取对应的Entry，若没有这样的Entry，则返回null
        Entry<K,V> e = (Entry<K,V>)getEntry(key); 
if (e == null)      // 若不存在这样的Entry，直接返回
return null;
        e.recordAccess(this);
return e.value;
    }
 
/**
     * Returns the entry associated with the specified key in the
     * HashMap.  Returns null if the HashMap contains no mapping
     * for the key.
     * 
     * HashMap 中的方法
     *     
     */
final Entry<K,V> getEntry(Object key) {
if (size == 0) {
return null;
        }
 
        int hash = (key == null) ? 0 : hash(key);
for (Entry<K,V> e = table[indexFor(hash, table.length)];
             e != null;
             e = e.next) {
            Object k;
if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k))))
return e;
        }
return null;
    }
```



在LinkedHashMap的get方法中，通过HashMap中的getEntry方法获取Entry对象。注意这里的recordAccess方法，如果链表中元素的排序规则是按照插入的先后顺序排序的话，该方法什么也不做；如果链表中元素的排序规则是按照访问的先后顺序排序的话，则将e移到链表的末尾处，笔者会在后文专门阐述这个问题。

 

另外，同样地，调用LinkedHashMap的get(Object key)方法后，若返回值是 NULL，则也存在如下两种可能：

 

 该 key 对应的值就是 null;

 HashMap 中不存在该 key。

 

##### 4、LinkedHashMap 存取小结

LinkedHashMap 的存取过程基本与HashMap基本类似，只是在细节实现上稍有不同，这是由LinkedHashMap本身的特性所决定的，因为它要额外维护一个双向链表用于保持迭代顺序。在put操作上，虽然LinkedHashMap完全继承了HashMap的put操作，但是在细节上还是做了一定的调整，比如，在LinkedHashMap中向哈希表中插入新Entry的同时，还会通过Entry的addBefore方法将其链入到双向链表中。在扩容操作上，虽然LinkedHashMap完全继承了HashMap的resize操作，但是鉴于性能和LinkedHashMap自身特点的考量，LinkedHashMap对其中的重哈希过程(transfer方法)进行了重写。在读取操作上，LinkedHashMap中重写了HashMap中的get方法，通过HashMap中的getEntry方法获取Entry对象。在此基础上，进一步获取指定键对应的值。









#### LinkedHashMap和HashMap区别

大多数情况下，**只要不涉及线程安全问题，Map基本都可以使用HashMap**，不过HashMap有一个问题，就是迭代HashMap的顺序并不是HashMap放置的顺序，也就是无序。HashMap的这一缺点往往会带来困扰，因为有些场景，我们期待一个有序的Map.这就是我们的LinkedHashMap,看个小Demo:

```JAVA

public static void main(String[] args) {
    Map<String, String> map = new LinkedHashMap<String, String>();
    map.put("apple", "苹果");
    map.put("watermelon", "西瓜");
    map.put("banana", "香蕉");
    map.put("peach", "桃子");

    Iterator iter = map.entrySet().iterator();
    while (iter.hasNext()) {
        Map.Entry entry = (Map.Entry) iter.next();
        System.out.println(entry.getKey() + "=" + entry.getValue());
    }
}

/*
输出为：
apple=苹果
watermelon=西瓜
banana=香蕉
peach=桃子
*/
```

**可以看到，在使用上，LinkedHashMap和HashMap的区别就是LinkedHashMap是有序的。** 上面这个例子是根据插入顺序排序，此外，LinkedHashMap还有一个参数决定**是否在此基础上再根据访问顺序(get,put)排序**,记住，是在插入顺序的基础上再排序，后面看了源码就知道为什么了。看下例子:

```JAVA
public static void main(String[] args) {
    Map<String, String> map = new LinkedHashMap<String, String>(16,0.75f,true);
    map.put("apple", "苹果");
    map.put("watermelon", "西瓜");
    map.put("banana", "香蕉");
    map.put("peach", "桃子");

    map.get("banana");
    map.get("apple");

    Iterator iter = map.entrySet().iterator();
    while (iter.hasNext()) {
        Map.Entry entry = (Map.Entry) iter.next();
        System.out.println(entry.getKey() + "=" + entry.getValue());
    }
}

/*
输出为：
watermelon=西瓜
peach=桃子
banana=香蕉
apple=苹果
*/
```

**可以看到香蕉和苹果在原来排序的基础上又排后了。**





### 2. LinkedHashMap底层实现

我先说结论，然后再慢慢跟代码。

- **LinkedHashMap继承自HashMap**,**它的新增(put)和获取(get)方法都是复用父类的HashMap的代码**，**只是自己重写了put给get内部的某些接口来搞事情**，这个特性在C++中叫**钩子技术**，**在Java里面大家喜欢叫多态**，其实多态这个词并不能很好的形容这种现象。

- **LinkedHashMap的数据存储和HashMap的结构一样采用(数组+单向链表)的形式**，**只是在每次节点Entry中增加了用于维护顺序的before和after变量维护了一个双向链表来保存LinkedHashMap的存储顺序**，当调用迭代器的时候不再使用HashMap的的迭代器，而是自己写迭代器来遍历这个双向链表即可。

- HashMap和LinkedHashMap内部逻辑图如下: 

  ![img](https://user-gold-cdn.xitu.io/2018/1/3/160b9c2b90ffdb53?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

  ![img](https://user-gold-cdn.xitu.io/2018/1/3/160b9d2d8e63660f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

好了，大家肯定会觉得很神奇，如图所示，本来HashMap的数据是0-7这样的无须的，而LinkedHashMap却把它变成了如图所示的1.6.5.3.。。2这样的有顺序了。到底是如何做到的了？其实说白了，就一句话，**钩子技术**，在put和get的时候维护好了这个双向链表，遍历的时候就有序了。好了，一步一步的跟。 先看一下LinkedHashMap中的Entry(也就是每个元素):

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

可以看到继承自HashMap的Entry，并且多了两个指针before和after，这两个指针说白了，就是为了维护双向链表新加的两个指针。 列一下新Entry的所有成员变量吧:

- K key
- V value
- Entry<K, V> next
- int hash
- **Entry before**
- **Entry after**

其中**前面四个，是从HashMap.Entry中继承过来的；后面两个，是是LinkedHashMap独有的**。不要搞错了next和before、**After，next是用于维护HashMap指定table位置上连接的Entry的顺序的，before、After是用于维护Entry插入的先后顺序的(为了维护双向链表)**。



#### 2.0 构造方法

 ![img](https://upload-images.jianshu.io/upload_images/4843132-3ec0494c6b87c52f.png?imageMogr2/auto-orient/strip|imageView2/2/w/330/format/webp) 

LinkedHashMap提供了多个构造方法，我们先看空参的构造方法。



```java
    public LinkedHashMap() {
        // 调用HashMap的构造方法，其实就是初始化Entry[] table
        super();
        // 这里是指是否基于访问排序，默认为false
        accessOrder = false;
    }
```

首先使用super调用了父类HashMap的构造方法，其实就是根据初始容量、负载因子去初始化Entry[] table，详细的看上一篇[HashMap解析](https://www.jianshu.com/p/dde9b12343c1)。

然后把accessOrder设置为false，这就跟存储的顺序有关了，**LinkedHashMap存储数据是有序的，而且分为两种：插入顺序和访问顺序。**

这里**accessOrder设置为false，表示不是访问顺序而是插入顺序存储的**，**这也是默认值**，表示LinkedHashMap中存储的顺序是按照调用put方法插入的顺序进行排序的。LinkedHashMap也提供了可以设置accessOrder的构造方法，我们来看看这种模式下，它的顺序有什么特点？

```dart
        // 第三个参数用于指定accessOrder值
        Map<String, String> linkedHashMap = new LinkedHashMap<>(16, 0.75f, true);
        linkedHashMap.put("name1", "josan1");
        linkedHashMap.put("name2", "josan2");
        linkedHashMap.put("name3", "josan3");
        System.out.println("开始时顺序：");
        Set<Entry<String, String>> set = linkedHashMap.entrySet();
        Iterator<Entry<String, String>> iterator = set.iterator();
        while(iterator.hasNext()) {
            Entry entry = iterator.next();
            String key = (String) entry.getKey();
            String value = (String) entry.getValue();
            System.out.println("key:" + key + ",value:" + value);
        }
        System.out.println("通过get方法，导致key为name1对应的Entry到表尾");
        linkedHashMap.get("name1");
        Set<Entry<String, String>> set2 = linkedHashMap.entrySet();
        Iterator<Entry<String, String>> iterator2 = set2.iterator();
        while(iterator2.hasNext()) {
            Entry entry = iterator2.next();
            String key = (String) entry.getKey();
            String value = (String) entry.getValue();
            System.out.println("key:" + key + ",value:" + value);
        }
```

![img](https:////upload-images.jianshu.io/upload_images/4843132-442ce33c7124092c.png?imageMogr2/auto-orient/strip|imageView2/2/w/342/format/webp)

因为调用了get("name1")导致了name1对应的Entry移动到了最后，这里只要知道LinkedHashMap有插入顺序和访问顺序两种就可以，后面会详细讲原理。

还记得，上一篇HashMap解析中提到，在HashMap的构造函数中，调用了init方法，而在HashMap中init方法是空实现，但LinkedHashMap重写了该方法，所以在LinkedHashMap的构造方法里，调用了自身的init方法，init的重写实现如下：



```dart
    /**
     * Called by superclass constructors and pseudoconstructors (clone,
     * readObject) before any entries are inserted into the map.  Initializes
     * the chain.
     */
    @Override
    void init() {
        // 创建了一个hash=-1，key、value、next都为null的Entry
        header = new Entry<>(-1, null, null, null);
        // 让创建的Entry的before和after都指向自身，注意after不是之前提到的next
        // 其实就是创建了一个只有头部节点的双向链表
        header.before = header.after = header;
    }
```

这好像跟我们上一篇HashMap提到的Entry有些不一样，HashMap中静态内部类Entry是这样定义的：



```dart
    static class Entry<K,V> implements Map.Entry<K,V> {
        final K key;
        V value;
        Entry<K,V> next;
        int hash;
```

没有before和after属性啊！原来，LinkedHashMap有自己的静态内部类Entry，它继承了HashMap.Entry，定义如下:



```cpp
    /**
     * LinkedHashMap entry.
     */
    private static class Entry<K,V> extends HashMap.Entry<K,V> {
        // These fields comprise the doubly linked list used for iteration.
        Entry<K,V> before, after;

        Entry(int hash, K key, V value, HashMap.Entry<K,V> next) {
            super(hash, key, value, next);
        }
```

所以**LinkedHashMap构造函数，主要就是调用HashMap构造函数初始化了一个Entry[] table，然后调用自身的init初始化了一个只有头结点的双向链表**。完成了如下操作：



![img](https:////upload-images.jianshu.io/upload_images/4843132-cac0b65e4d23b6bd.png?imageMogr2/auto-orient/strip|imageView2/2/w/524/format/webp)

LinkedHashMap构造函数.png



#### 2.1 初始化

```java
public LinkedHashMap() {
	super();
	accessOrder = false;
}
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    threshold = (int)(DEFAULT_INITIAL_CAPACITY * DEFAULT_LOAD_FACTOR);
    table = new Entry[DEFAULT_INITIAL_CAPACITY];
    init();
}

void init() {
   header = new Entry<K,V>(-1, null, null, null);
   header.before = header.after = header;
}
```

我们已经知道 LinkedHashMap 的 Entry 元素继承 HashMap 的 Entry，提供了双向链表的功能。在上述 HashMap 的构造器中，最后会调用 init() 方法，进行相关的初始化，这个方法在 HashMap 的实现中并无意义，只是提供给子类实现相关的初始化调用。

但在 LinkedHashMap 重写了 init() 方法，在调用父类的构造方法完成构造后，进一步实现了对其元素 Entry 的初始化操作。

#### 2.2 LinkedHashMap添加元素



 **LinkedHashMap 并未重写父类 HashMap 的 put 方法，而是重写了父类 HashMap 的 put 方法调用的子方法void recordAccess(HashMap m) ，void addEntry(**int hash, K key, V value, int bucketIndex) 和void createEntry(int hash, K key, V value, int bucketIndex)，**提供了自己特有的双向链接列表的实现**。我们在之前的文章中已经讲解了HashMap的put方法，我们在这里重新贴一下 HashMap 的 put 方法的源代码： 

##### 仍旧是HashMap中的put方法:

```java
public V put(K key, V value) {
    //key为null的处理
    if (key == null)
       return putForNullKey(value);
    //计算Hash
    int hash = hash(key.hashCode());
    //得到在table中的index
    int i = indexFor(hash, table.length);
    //遍历table[index],是否key已经存在，如果存在则替换，并返回旧值
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
         Object k;
         if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
             V oldValue = e.value;
           e.value = value;
11             e.recordAccess(this);
12             return oldValue;
13         }
14     }
15 
16     modCount++;
    //如果key之前在table中不存在，则调用addEntry,LinkedHashMap重写了该方法
17     addEntry(hash, key, value, i);
18     return null;
19 }
```

LinkedHashMap中的addEntry(又是一个钩子技术):

```JAVA
void recordAccess(HashMap<K,V> m) {
    LinkedHashMap<K,V> lm = (LinkedHashMap<K,V>)m;
    if (lm.accessOrder) {
        lm.modCount++;
        remove();
        addBefore(lm.header);
        }
}

void addEntry(int hash, K key, V value, int bucketIndex) {
    // 调用create方法，将新元素以双向链表的的形式加入到映射中。
    createEntry(hash, key, value, bucketIndex);

    // 删除最近最少使用元素的策略定义
    Entry<K,V> eldest = header.after;
    if (removeEldestEntry(eldest)) {
        removeEntryForKey(eldest.key);
    } else {
        if (size >= threshold)
            resize(2 * table.length);
    }
}

void createEntry(int hash, K key, V value, int bucketIndex) {
    HashMap.Entry<K,V> old = table[bucketIndex];
    Entry<K,V> e = new Entry<K,V>(hash, key, value, old);
    table[bucketIndex] = e;
    // 调用元素的addBrefore方法，将元素加入到哈希、双向链接列表。  
    e.addBefore(header);
    size++;
}

private void addBefore(Entry<K,V> existingEntry) {
    after  = existingEntry;
    before = existingEntry.before;
    before.after = this;
    after.before = this;
}
```

好了，addEntry先把数据加到HashMap中的结构中(数组+单向链表),然后调用addBefore，这个我就不和大家画图了，**其实就是挪动自己和Header的Before与After成员变量指针把自己加到双向链表的尾巴上。** 同样的，无论put多少次，都会把当前元素加到队列尾巴上。这下大家知道怎么维护这个双向队列的了吧。

上面说了LinkedHashMap在新增数据的时候自动维护了双向列表，这要还要提一下的是LinkedHashMap的另外一个属性，**根据查询顺序排序**,**说白了，就是在get的时候或者put(更新时)把元素丢到双向队列的尾巴上。这样不就排序了吗**？这里涉及到LinkedHashMap的另外一个构造方法:

```JAVA
public LinkedHashMap(int initialCapacity,
         float loadFactor,
                     boolean accessOrder) {
    super(initialCapacity, loadFactor);
    this.accessOrder = accessOrder;
}
```

第三个参数，accessOrder为是否开启**查询排序功能的开关**，默认为False。如果想开启那么必须调用这个构造方法。 然后看下get和put(更新操作)时是如何维护这个队列的。



##### GET

```JAVA
public V get(Object key) {
    // 调用父类HashMap的getEntry()方法，取得要查找的元素。
    Entry<K,V> e = (Entry<K,V>)getEntry(key);
    if (e == null)
        return null;
    // 记录访问顺序。
    e.recordAccess(this);
    return e.value;
}
```

此外，在put的时候，代码11行(见上面的代码)，也是调用了e.recordAccess(this);我们来看下这个方法:

```JAVA
void recordAccess(HashMap<K,V> m) {
    LinkedHashMap<K,V> lm = (LinkedHashMap<K,V>)m;
    // 如果定义了LinkedHashMap的迭代顺序为访问顺序，
    // 则删除以前位置上的元素，并将最新访问的元素添加到链表表头。  
    if (lm.accessOrder) {
        lm.modCount++;
        remove();
        addBefore(lm.header);
    }
}

/**
* Removes this entry from the linked list.
*/
private void remove() {
    before.after = after;
    after.before = before;
}

/**clear链表，设置header为初始状态*/
public void clear() {
 super.clear();
 header.before = header.after = header;
}


private void addBefore(Entry<K,V> existingEntry) {
    after  = existingEntry;
    before = existingEntry.before;
    before.after = this;
    after.before = this;
}
```

看到每次recordAccess的时候做了两件事情：

1. **把待移动的Entry的前后Entry相连**
2. **把待移动的Entry移动到尾部**

### 排序模式

LinkedHashMap 定义了排序模式 accessOrder，该属性为 boolean 型变量，对于访问顺序，为 true；对于插入顺序，则为 false。一般情况下，不必指定排序模式，其迭代顺序即为默认为插入顺序。

这些构造方法都会默认指定排序模式为插入顺序。如果你想构造一个 LinkedHashMap，并打算按从近期访问最少到近期访问最多的顺序（即访问顺序）来保存元素，那么请使用下面的构造方法构造 LinkedHashMap：public LinkedHashMap(int initialCapacity, float loadFactor, boolean accessOrder)

该哈希映射的迭代顺序就是最后访问其条目的顺序，这种映射很适合构建 LRU 缓存。LinkedHashMap 提供了 removeEldestEntry(Map.Entry<K,V> eldest) 方法。该方法可以提供在每次添加新条目时移除最旧条目的实现程序，默认返回 false，这样，此映射的行为将类似于正常映射，即永远不能移除最旧的元素。

我们会在后面的文章中详细介绍关于如何用 LinkedHashMap 构建 LRU 缓存。





当然，这一切都是基于accessOrder=true的情况下。 假设现在我们开启了accessOrder，然后调用get("111");看下是如何操作的: 

![img](https://user-gold-cdn.xitu.io/2018/1/3/160b9f6522608282?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)





### 3. 利用LinkedHashMap实现LRU缓存

**LRU即Least Recently Used，最近最少使用，也就是说，当缓存满了，会优先淘汰那些最近最不常访问的数据**。我们的LinkedHashMap正好满足这个特性，为什么呢？当我们开启accessOrder为true时，**最新访问(get或者put(更新操作))的数据会被丢到队列的尾巴处，那么双向队列的头就是最不经常使用的数据了**。比如:

如果有1 2 3这3个Entry，那么访问了1，就把1移到尾部去，即2 3 1。每次访问都把访问的那个数据移到双向队列的尾部去，那么每次要淘汰数据的时候，双向队列最头的那个数据不就是最不常访问的那个数据了吗？换句话说，双向链表最头的那个数据就是要淘汰的数据。

此外，LinkedHashMap还提供了一个方法，这个方法就是为了我们实现LRU缓存而提供的，**removeEldestEntry(Map.Entry eldest) 方法。该方法可以提供在每次添加新条目时移除最旧条目的实现程序，默认返回 false**。

来，给大家一个简陋的LRU缓存:

```
public class LRUCache extends LinkedHashMap
{
    public LRUCache(int maxSize)
    {
        super(maxSize, 0.75F, true);
        maxElements = maxSize;
    }

    protected boolean removeEldestEntry(java.util.Map.Entry eldest)
    {
        //逻辑很简单，当大小超出了Map的容量，就移除掉双向队列头部的元素，给其他元素腾出点地来。
        return size() > maxElements;
    }

    private static final long serialVersionUID = 1L;
    protected int maxElements;
}
复制代码
```

是不是很简单。。

### 结语

其实 LinkedHashMap 几乎和 HashMap 一样：从技术上来说，不同的是它定义了一个 Entry<K,V> header，这个 header 不是放在 Table 里，它是额外独立出来的。LinkedHashMap 通过继承 hashMap 中的 Entry<K,V>,并添加两个属性 Entry<K,V> before,after,和 header 结合起来组成一个双向链表，来实现按插入顺序或访问顺序排序。**如何维护这个双向链表了，就是在get和put的时候用了钩子技术(多态)调用LinkedHashMap重写的方法来维护这个双向链表，然后迭代的时候直接迭代这个双向链表即可**,好了LinkedHashMap算是给大家分享完了.

