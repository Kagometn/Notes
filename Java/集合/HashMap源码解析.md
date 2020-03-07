 

### 一，HashMap源码：

#### 什么是HashMap

HashMap**实现了Map接口**，**Map接口对键值对进行映射**。Map中**不允许重复的键**。Map接口有两个基本的实现，**HashMap和TreeMap。TreeMap保存了对象的排列次序，而HashMap则不能。HashMap允许键和值为null。HashMap是非synchronized的，但collection框架提供方法能保证HashMap synchronized，这样多个线程同时访问HashMap时，能保证只有一个线程更改Map。**

public Object **put(Object Key,Object value)方法用来将元素添加到map中**。

#### 概述：

使用**数组**和**单向链表**的数据结构，但是是**线程不安全的**，而且**允许key和value为null**。

其底层数据结构是**数组**称之为**哈希桶**，每个桶里放的是**链表**，链表的每个节点，就是哈希表中的**每个元素。**



#### HashMap的工作原理：

 HashMap**基于hashing原理**，**我们通过put()和get()方法储存和获取对象**。当我们将键值对传递给put()方法时，**它调用键对象的hashCode()方法来计算hashcode，让后找到bucket位置来储存值对象**。**当获取对象时，通过键对象的equals()方法找到正确的键值对，然后返回值对象。HashMap使用链表来解决碰撞问题，当发生碰撞了，对象将会储存在链表的下一个节点中。 HashMap在每个链表节点中储存键值对对象。** 

#### 继承关系：

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable 
```

#### 常用的常量：

```java
public class HashMap<K, V> extends AbstractMap<K, V> implements Map<K, V>, Cloneable, Serializable {
    private static final long serialVersionUID = 362498820763181265L;
    //HashMap容量的初始大小，默认为16
    static final int DEFAULT_INITIAL_CAPACITY = 16;
    //1<<30,HashMap容量极限，最大为2的30次幂
    static final int MAXIMUM_CAPACITY = 1073741824;
    //负载因子默认大小为0.75，决定了是否扩容
    static final float DEFAULT_LOAD_FACTOR = 0.75F;
    //链表的最大长度，当超过8且桶的数量大于等于64的时候转化为红黑树
    static final int TREEIFY_THRESHOLD = 8;
    //hash表某个节点的数量小于6的时候进行树拆分
    static final int UNTREEIFY_THRESHOLD = 6;
    //树化时的最小桶的数量
    static final int MIN_TREEIFY_CAPACITY = 64;
    //Node时Map.Entry接口的实现类
    //再次存储数据的Node数组容量时2次幂
    //每一个Node本质都是一个单项链表
    transient HashMap.Node<K, V>[] table;
    transient Set<Entry<K, V>> entrySet;
    //HashMap大小，代表了HashMap保存的键值对的多少
    transient int size;
    //HashMap被改变的次数
    transient int modCount;
    //下次进行HashMap扩容的大小
    int threshold;
    //存储负载因子的常量
    final float loadFactor;
```



- initialCapacity = 16表示的是数组的长度

  >initailCapacity要设置成2的n次幂，为了减少碰撞的几率，提高查询的效率。
  >
  >![1574325231590](C:\Users\Lenovo\Desktop\java\1574325231590.png)
  >
  >当数组的长度为2的n次幂的时候，不同的key得到的index相同的概率就会很小，那么数据在数组上分布就比较均匀，也就是碰撞的几率就会变小，相应的查询的时候就不会遍历某个位置上的来年表就会提高查找效率

- loadFactor = 0.75

  >
  >
  >而负载因子表示一个散列表的空间的使用程度，有这样一个公式：initailCapacity*loadFactor=HashMap的容量。
  >
  >- 负载因子越大则散列表的装填程度越高，也就是能容纳更多的元素
  >- 负载因子越小，则链表的数据量就会越稀疏，此时会造成空间浪费，到那时索引的效率高
  >- hash表每次扩容长度为之前的2倍
  >- HashMap是线程不安全的，在JDK1.7的时候进行多线程的put操作的时候，之后遍历，就会陷入死循环，而在JDK1.8的时候由于对resize进行了优化，所以不会再成**链表闭环**

#### hash:根据key求他的hash值（判断下标位置）

```java
 //key.hashCode()根据key求他的hash值
    // h >>> 16， >>> 无符号右移，高位补零
    //为计算hashMap的下标有关：
    //n = table.length;
	//index = （n-1） & hash;
    //table的长度为2的幂，因此index只与hash值的低n位有关
    static final int hash(Object key) { 
        int h;
        return key == null ? 0 : (h = key.hashCode()) ^ h >>> 16;
    }
```

#### comparableClassFor：

```java
    //对象x的类为C,如果C实现了Comparable<C>接口，那么返回c，否则返回null
    static Class<?> comparableClassFor(Object x) {
        if (x instanceof Comparable) {
            Class c;
            //如果x是一个字符串对象
            if ((c = x.getClass()) == String.class) { 
                //返回String.class
                return c;
            } 
            Type[] ts;
            //
            if ((ts = c.getGenericInterfaces()) != null) {
                Type[] var5 = ts; 
                int var6 = ts.length;    
                for(int var7 = 0; var7 < var6; ++var7) {   
                    Type t = var5[var7]; 
                    Type[] as;    
                    ParameterizedType p;     
                    if (t instanceof ParameterizedType && (p = (ParameterizedType)t).getRawType() == Comparable.class && (as = p.getActualTypeArguments()) != null && as.length == 1 && as[0] == c) {   
                        return c;    
                    }         
                }       
            }    
        }        
        return null;  
    } 
  
    ......
      
```



#### tableSizeFor：

```java
 // 1073741824 = 2^30
      static final int tableSizeFor(int cap) {
        //numberOfLeadingZeros的返回值是最高位前面的0的个数。负数返回0，0返回32
        int n = -1 >>> Integer.numberOfLeadingZeros(cap - 1);
        return n < 0 ? 1 : (n >= 1073741824 ? 1073741824 : n + 1);
    }
    
    
   
```

#### HashMap的四个构造函数:



```java

//有参构造器，可以很具业务需求自定义数组长度以及负载因子
     public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0) {
            throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
        } else {
            //数组最大长度
            if (initialCapacity > 1073741824) {
                initialCapacity = 1073741824;
            }
            
            //
            if (loadFactor > 0.0F && !Float.isNaN(loadFactor)) {
                this.loadFactor = loadFactor;
                //threshold是hashMap判断size是否需要扩容的阈值，通过调用tableSizeFor(initialCapacity)来设置threshold
                this.threshold = tableSizeFor(initialCapacity);
            } else {
                throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
            }
        }
    }
    
```

```java
	//创建一个空的哈希表，只当容量，使用默认的加载因子
	public HashMap(int initialCapacity) {
        this(initialCapacity, 0.75F);
    }
//创建空的哈希表，使用默认的容量以及加载因子
    public HashMap() {
        this.loadFactor = 0.75F;
    }
//创建一个内容为参数m的内容的哈希表
    public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = 0.75F;
        this.putMapEntries(m, false);
    }
```



```java
//
    final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
        int s = m.size();
        if (s > 0) {
            //判断table是否存在，如果不存在的话就一次输入map的大小定义存储空间大小，否则的话判断时候需要扩容，
            if (this.table == null) {
                float ft = (float)s / this.loadFactor + 1.0F;
                int t = ft < 1.07374182E9F ? (int)ft : 1073741824;
                if (t > this.threshold) {
                    this.threshold = tableSizeFor(t);
                }
                //m.size大于当前容量的话则进行扩容(利用resize方法)
            } else if (s > this.threshold) {
                this.resize();
            }

            Iterator var8 = m.entrySet().iterator();

            while(var8.hasNext()) {
                Entry<? extends K, ? extends V> e = (Entry)var8.next();
                K key = e.getKey();
                V value = e.getValue();
                this.putVal(hash(key), key, value, false, evict);
            }
        }
```

#### put():

```java
public V put(K key, V value) {
        return this.putVal(hash(key), key, value, false, true);
    }
//Hash()
 static final int hash(Object key) {
        int h;
        return key == null ? 0 : (h = key.hashCode()) ^ h >>> 16;
    }
```

#### putVal():

```java
//onlyIfAbsent:如果是true的话，不覆盖已经存在的值
//evict:如果是false的话，则表示在创建模式
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
        HashMap.Node[] tab;
        int n;
    //HashMap实现的是懒加载，新建HashMap时不会对table进行赋值, 而是到第一次插入时, 进行resize时构建table;
    //如果桶(数组)为空或者是长度为0的话，就对其进行初始化扩容
    if ((tab = this.table) == null || (n = tab.length) == 0) {
            n = (tab = this.resize()).length;
        }

        Object p;
        int i;
    /**
    * 根据key的hash值计算需要存储的位置，如果该位置没有数据，则直接存储
    */
        if ((p = tab[i = n - 1 & hash]) == null) {
            tab[i] = this.newNode(hash, key, value, (HashMap.Node)null);
        } else {
             //该位置已有数据存在，则判断该位置链表中有没有目标key,查看自己的key与hash值是否与该位置的是否一致，如果一致的话，就将记录下该元素以下为e，如果有一个和我插入元素的key一样的话，后续可能会被覆盖
            Object e;
            Object k;
            if (((HashMap.Node)p).hash == hash && ((k = ((HashMap.Node)p).key) == key || key != null && key.equals(k))) {
                e = p;
                //如果只是 hash值相等而key不相等的话：指的是hash碰撞，此时一般采用链地址法，，将碰撞的元素连成一个链表，这里由于连掉太长的话就会进行树化，
                //该语句即使判断是否是p里面放的是不是个红黑树
            } else if (p instanceof HashMap.TreeNode) {
                //如果是红黑树的话，就将结点放入到红黑树，这里也不一定是插入到树中
                //如果插入的元素和红黑树中的某个结点的key值相同的话，也会考虑新值换旧值的问题
                e = ((HashMap.TreeNode)p).putTreeVal(this, tab, hash, key, value);
            } else {
           //如果执行到此时，说明是链表结构，而binCount用来记录链表中元素的个数，从而判断链表是否需要树化成红黑树
                int binCount = 0;
                while(true) {
                    //e的后一个结点为空，则直接获取需要插入的元素
                    if ((e = ((HashMap.Node)p).next) == null) {
                        ((HashMap.Node)p).next = this.newNode(hash, key, value, (HashMap.Node)null);
                        //TREEIFY_THRESHOLD 是树化的阈值且其值为8
                        //如果binCount长度大于等于7，则就可以树化了，其实真正树化的条件是
                        //链表中的元素的个数大于等于8
                        if (binCount >= 7) {
                            this.treeifyBin(tab, hash);
                        }
                        break;
                    }
                    //待插入的key在链表中找到了，记录下来然后退出
                    if (((HashMap.Node)e).hash == hash && ((k = ((HashMap.Node)e).key) == key || key != null && key.equals(k))) {
                        break;
                    }

                    p = e;
                    ++binCount;
                }
            }
            //说明已经找到了key相同的元素
            if (e != null) {
                V oldValue = ((HashMap.Node)e).value;
                //判断是否需要旧值换新值，默认下是允许更换的
                if (!onlyIfAbsent || oldValue == null) {
                    ((HashMap.Node)e).value = value;
                }
                this.afterNodeAccess((HashMap.Node)e);
                return oldValue;
            }
        }
    	//修改次数+1
        ++this.modCount;
    	//判断是否达到扩容的阈值
        if (++this.size > this.threshold) {
            //当HashMap.size 大于 threshold时, 会进行resize;threshold的值我们在上一次分享中提到过: 当第一次构建时, 如果没有指定HashMap.table的初始长度, 就用默认值16, 否则就是指定的值; 然后不管是第一次构建还是后续扩容, threshold = table.length * loadFactor;
            //扩容的方法，在本方法的前面需要初始化的时候也出现过
            this.resize();
        }
    //这个方法也是服务与LinkedHashMap的
        this.afterNodeInsertion(evict);
    //没有找到元素，返回空
        return null;
    }
```



往表中批量增加数据

```java
    public void putAll(Map<? extends K, ? extends V> m) {
        //这个函数上一节也已经分析过。//将另一个Map的所有元素加入表中，参数evict初始化时为false，其他情况为true
        putMapEntries(m, true);
    }

```









 在resize的过程，**简单的说就是把bucket扩充为2倍，之后重新计算index，把节点再放到新的bucket中**。resize的注释是这样描述的： 

####  resize():

在resize的过程，简单的说就是把bucket扩充为2倍，之后重新计算index，把节点再放到新的bucket中。resize的注释是这样描述的： 



大致意思就是说，当超过限制的时候会resize，然而又因为我们使用的是`2次幂`的扩展(指长度扩

为原来2倍)，所以，**元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置**。 



 ***经过rehash之后，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置*。对应的就是下方的resize的注释。** 

看下图可以明白这句话的意思，n为table的长度，图（a）表示扩容前的key1和key2两种key确定索引位置的示例，图（b）表示扩容后key1和key2两种key确定索引位置的示例，**其中hash1是key1对应的哈希值(也就是根据key1算出来的hashcode值)与高位与运算的结果。**

 ![img](https://img-blog.csdn.net/20170123110634173) 

 元素在重新计算hash之后，因为n变为2倍，那么n-1的mask范围在高位多1bit(红色)，因此新的index就会发生这样的变化： 

![img](https://img-blog.csdn.net/20170123110716285)

因此，**我们在扩充HashMap的时候，不需要像JDK1.7的实现那样重新计算hash，只需要看看原来的hash值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成“原索引+oldCap”**，可以看看下图为16扩充为32的resize示意图：

 ![resize16to32](https://user-gold-cdn.xitu.io/2019/12/21/16f27d5edbb21532?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 



***\*是 没看懂他连个图是怎么前后对应的，谁看懂了，交流哈赛。\****

***\*当时上面这个图，没看懂，是因为，他就没说每个节点的hashcode是啥，他怎么确定是保留在原来的位置，还是说在原来位置的基础上再加个原来数组的长度呢。所以，上面那个图仅仅具有丁点儿参考价值。\****

这个设计确实非常的巧妙，既省去了重新计算hash值的时间，而且同时，由于新增的1bit是0还是1可以认为是随机的，因此resize的过程，均匀的把之前的冲突的节点分散到新的bucket了。这一块就是JDK1.8新增的优化点。有一点注意区别，JDK1.7中rehash的时候，旧链表迁移新链表的时候，如果在新表的数组索引位置相同，则链表元素会倒置，但是从上图可以看出，JDK1.8不会倒置。有兴趣的同学可以研究下JDK1.8的resize源码，写的很赞，如下:

```java
 
final HashMap.Node<K, V>[] resize() { 
        HashMap.Node<K, V>[] oldTab = this.table;
        int oldCap = oldTab == null ? 0 : oldTab.length;
        int oldThr = this.threshold;
        int newThr = 0;
        int newCap;
     //如果旧的hash桶不为空的话
        if (oldCap > 0) {
            //判断时候超出数组长度的最大值
            //不在扩容，直接返回
            if (oldCap >= 1073741824) {
                //Integer.MAX_VALUE=214743674
                this.threshold = 2147483647;
                return oldTab;
            }
            //判断新的hash桶的长度被扩容后有没有超过桶的最大长度
            if ((newCap = oldCap << 1) < 1073741824 && oldCap >= 16) {
                //将新的阈值扩容为之前的2倍
                newThr = oldThr << 1;
            }
         // 在构造函数一节中我们知道
   		// 如果没有指定initialCapacity, 则不会给threshold赋值, 该值被初始化为0
    	// 如果指定了initialCapacity, 该值被初始化成大于initialCapacity的最小的2的次幂
    
    	// 这里是指, 如果构造时指定了initialCapacity, 则用threshold作为table的实际大小
         //hash表的阈值已经初始化过了
        } else if (oldThr > 0) {
            newCap = oldThr;
        //如果构造的时候没有指定initialCapacity，则使用默认值
        } else {
            //需要初始化新的hash桶的容量以及新容量的阈值
            newCap = 16;
            newThr = 12;
        }
     	//新的局部变量阈值赋值
    	// 计算指定了initialCapacity情况下的新的 threshold
        if (newThr == 0) {
            float ft = (float)newCap * this.loadFactor;
            newThr = newCap < 1073741824 && ft < 1.07374182E9F ? (int)ft : 2147483647;
        }
     //为当前容量阈值赋值
        this.threshold = newThr;
       
    //从以上操作我们知道, 初始化HashMap时, 
    //如果构造函数没有指定initialCapacity, 则table大小为16
    //如果构造函数指定了initialCapacity, 则table大小为threshold, 即大于指定initialCapacity的最小的2的整数次幂
    HashMap.Node<K, V>[] newTab = new HashMap.Node[newCap];
    this.table = newTab;
    //如果旧的hash桶不为空，需要将旧的hash表里面的键值对重新进行映射到新的hash桶中
        if (oldTab != null) {
            for(int j = 0; j < oldCap; ++j) { 
                HashMap.Node e;
                if ((e = oldTab[j]) != null) {
                    // 这里注意, table中存放的只是Node的引用, 这里将oldTab[j]=null只是清除旧表的引用, 但是真正的node节点还在, 只是现在由e指向它
                    oldTab[j] = null;
                    //如果只有一个结点，就直接将其放到新表的目标位置
                    if (e.next == null) {
                        newTab[e.hash & newCap - 1] = e;
                        //如果是红黑树的，需要进行树拆分然后映射
                    } else if (e instanceof HashMap.TreeNode) {
                        ((HashMap.TreeNode)e).split(this, newTab, j, oldCap);
                        //如果是多个节点的链表，将原来的链表拆分为两个链表，两个链表的索引】
                        //一个是原索引，一个为原索引你加上就的hash桶长度的偏移量
                    } else {
                        HashMap.Node<K, V> loHead = null;
                        HashMap.Node<K, V> loTail = null;
                        HashMap.Node<K, V> hiHead = null;
                        HashMap.Node hiTail = null;

                        HashMap.Node next;
                        do {
                            next = e.next;
                            //链表1;原索引
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null) {
                                    loHead = e;
                                } else {
                                    loTail.next = e;
                                }

                                loTail = e;
                                //链表2;原索引+oldCap
                            } else {
                                if (hiTail == null) {
                                    hiHead = e;
                                } else {
                                    hiTail.next = e;
                                }

                                hiTail = e;
                            }

                            e = next;
                        } while(next != null);
                        //链表1存于原索引
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        //链表二存于原索引加上原hash桶长度的偏移量
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



##### 索引变化：

```java
HashMap.Node<K, V> loHead = null;
HashMap.Node<K, V> loTail = null;
HashMap.Node<K, V> hiHead = null;
HashMap.Node hiTail = null;

HashMap.Node next;
do {
    next = e.next;
    //链表1;原索引
    if ((e.hash & oldCap) == 0) {
        if (loTail == null) {
            loHead = e;
        } else {
            loTail.next = e;
        }
        loTail = e;
        //链表2;原索引+ol
    } else {
        if (hiTail == null) {
            hiHead = e;
        } else {
            hiTail.next = e;
        }
        hiTail = e;
    }
    e = next;
} while(next != null);
//链表1存于原索引
if (loTail != null) {
    loTail.next = null;
    newTab[j] = loHead;
}
//链表二存于原索引加上原hash桶长度的偏移量
if (hiTail != null) {
    hiTail.next = null;
    newTab[j + oldCap] = hiHead;
}
```



###### 第一段

```java
Node<K,V> loHead = null, loTail = null;
Node<K,V> hiHead = null, hiTail = null;
```

上面这段定义了四个Node的引用, 从变量命名上,我们初步猜测, 这里定义了两个链表, 我们把它称为 `lo链表` 和 `hi链表`, `loHead` 和 `loTail` 分别指向 `lo链表`的头节点和尾节点, `hiHead` 和 `hiTail`以此类推.

###### 第二段

```java
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
```

上面这段是一个do-while循环, 我们先从中提取出主要框架:

```java
do {
    next = e.next;
    ...
} while ((e = next) != null);
```

从上面的框架上来看, 就是在按顺序遍历该存储桶位置上的链表中的节点.

我们再看`if-else` 语句的内容:

```java
// 插入lo链表
if (loTail == null)
    loHead = e;
else
    loTail.next = e;
loTail = e;

// 插入hi链表
if (hiTail == null)
    hiHead = e;
else
    hiTail.next = e;
hiTail = e;
```

上面结构类似的两段看上去就是一个将节点e插入链表的动作.

最后再加上 `if` 块, 则上面这段的目的就很清晰了:

> 我们首先准备了两个链表 `lo` 和 `hi`, 然后我们顺序遍历该存储桶上的链表的每个节点, **如果 `(e.hash & oldCap) == 0`, 我们就将节点放入`lo`链表, 否则, 放入`hi`链表**.

###### 第三段

第二段弄明白之后, 我们再来看第三段:

```java 
if (loTail != null) {
    loTail.next = null;
    newTab[j] = loHead;
}
if (hiTail != null) {
    hiTail.next = null;
    newTab[j + oldCap] = hiHead;
}
```

这一段看上去就很简单了:

> 如果lo链表非空, 我们就把整个lo链表放到新table的`j`位置上
> 如果hi链表非空, 我们就把整个hi链表放到新table的`j+oldCap`位置上

综上我们知道, 这段代码的意义就是将原来的链表拆分成两个链表, 并将这两个链表分别放到新的table的 `j` 位置和 `j+oldCap` 上, `j`位置就是原链表在原table中的位置, 拆分的标准就是:

```java 
(e.hash & oldCap) == 0
```

为了帮助大家理解，我画了个示意图：
![rezise](HashMap%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/2076773863-5b5e7d6916752_articlex.png)
*（ps: 画个图真的好累啊， 大家有什么好的画图工具推荐吗？）*

###### 关于 `(e.hash & oldCap) == 0` `j` 以及 `j+oldCap`

上面我们已经弄懂了链表拆分的代码, 但是这个拆分条件看上去很奇怪, 这里我们来稍微解释一下:

首先我们要明确三点:

1. **oldCap一定是2的整数次幂, 这里假设是2^m**
2. **newCap是oldCap的两倍, 则会是2^(m+1)**
3. **hash对数组大小取模`(n - 1) & hash` 其实就是取hash的低`m`位**

例如:
我们假设 oldCap = 16, 即 2^4,
16 - 1 = 15, 二进制表示为 `0000 0000 0000 0000 0000 0000 0000 1111`
可见除了低4位, 其他位置都是0（简洁起见，高位的0后面就不写了）, 则 `(16-1) & hash` 自然就是取hash值的低4位,我们假设它为 `abcd`.

以此类推, 当我们将oldCap扩大两倍后, 新的index的位置就变成了 `(32-1) & hash`, 其实就是取 hash值的低5位. 那么对于同一个Node, 低5位的值无外乎下面两种情况:

```java
0abcd
1abcd
```

其中, `0abcd`与原来的index值一致, 而`1abcd` = `0abcd + 10000` = `0abcd + oldCap`

故**虽然数组大小扩大了一倍，但是同一个`key`在新旧table中对应的index却存在一定联系： 要么一致，要么相差一个 `oldCap`。**

而新旧index是否一致就体现在hash值的第4位(我们把最低为称作第0位), 怎么拿到这一位的值呢, 只要:

```java
hash & 0000 0000 0000 0000 0000 0000 0001 0000
```

上式就等效于

```java
hash & oldCap
```

故得出结论:

> **如果 `(e.hash & oldCap) == 0` 则该节点在新表的下标位置与旧表一致都为 `j`**
> **如果 `(e.hash & oldCap) == 1` 则该节点在新表的下标位置 `j + oldCap`**

根据这个条件, 我们将原位置的链表拆分成两个链表, 然后一次性将整个链表放到新的Table对应的位置上.

newNode如下：构建一个链表节点

```java
// Create a regular (non-tree) node
Node<K,V> newNode(int hash, K key, V value, Node<K,V> next) {
    return new Node<>(hash, key, value, next);
}
```

​		

```java
    // Callbacks to allow LinkedHashMap post-actions
    void afterNodeAccess(Node<K,V> p) { }
    void afterNodeInsertion(boolean evict) { }
```





#### 

##### 小结：

* 运算尽量都用位运算代替，更高效。
* 对于扩容导致需要新建数组存放更多元素时，除了要将老数组中的元素迁移过来，也记得将老数组中的引用置null，以便GC
* 取下标 是用 哈希值 与运算 （桶的长度-1） i = (n - 1) & hash。 由于桶的长度是2的n次方，这么做其实是等于 一个模运算。但是效率更高
* 扩容时，如果发生过哈希碰撞，节点数小于8个。则要根据链表上每个节点的哈希值，依次放入新哈希桶对应下标位置。
* 因为扩容是容量翻倍，所以原链表上的每个节点，现在可能存放在原来的下标，即low位， 或者扩容后的下标，即high位。 high位= low位+原哈希桶容量
* 利用哈希值 与运算 旧的容量 ，if ((e.hash & oldCap) == 0),可以得到哈希值去模后，是大于等于oldCap还是小于oldCap，等于0代表小于oldCap，应该存放在低位，否则存放在高位。这里又是一个利用位运算 代替常规运算的高效点
* 如果追加节点后，链表数量》=8，则转化为红黑树
* 插入节点操作时，有一些空实现的函数，用作LinkedHashMap重写使用】 



1. resize发生在table初始化, 或者table中的节点数超过`threshold`值的时候, `threshold`的值一般为负载因子乘以容量大小.
2. **每次扩容都会新建一个table, 新建的table的大小为原大小的2倍.**
3. 扩容时,会将原table中的节点re-hash到新的table中, 但节点在新旧table中的位置存在一定联系: 要么下标相同, 要么相差一个`oldCap`(原table的大小).

- 新初始化哈希表时，容量为默认容量，与职位容量*加载因子

- 已有哈希表扩容时，容量和阈值均翻倍

- 如果之前的结点为树结点的话，需要将新哈希桶里当前同也变成树形结构

- 复制给新哈希表中需要重新索引(rehash)，这里采用的计算方法是：

  ​	e.hash&(newCap-1),等价于 e.hash%newCap

**结合扩容源码可以发现扩容的确开销很大，需要迭代所有的元素，rehash、赋值，还得保留原来的数据结构。**

**所以在使用的时候，最好在初始化的时候就指定好 HashMap 的长度，尽量避免频繁 resize()。**

#### treeifyBin:

```java
//将桶的所有的链表节点替换成红黑树结点
final void treeifyBin(HashMap.Node<K, V>[] tab, int hash) {
        int n;
        //如果当前的哈希表不为空并且表的长度大于等于64  进行初始化的阈值(默认为64)，就去新建		//以及扩容
        if (tab != null && (n = tab.length) >= 64) {
            int index;
            HashMap.Node e;
            if ((e = tab[index = n - 1 & hash]) != null) {
                HashMap.TreeNode<K, V> hd = null;
                HashMap.TreeNode tl = null;

                do {
                    HashMap.TreeNode<K, V> p = this.replacementTreeNode(e, (HashMap.Node)null);
                    if (tl == null) {
                        hd = p;
                    } else {
                        p.prev = tl;
                        tl.next = p;
                    }

                    tl = p;
                } while((e = e.next) != null);
                //让桶的第一个元素只想新建的红黑树树头结点，以后这个同i你功力的元素就是红黑树而不是链表
                if ((tab[index] = hd) != null) {
                    hd.treeify(tab);
                }
            }
        } else {
            this.resize();
        }

    }
```

#### remove:

```java
 public V remove(Object key) {
        HashMap.Node e;
        return (e = this.removeNode(hash(key), key, (Object)null, false, true)) == null ? null : e.value;
    }
```

//从哈希表中删除某个节点， 如果参数matchValue是true，则必须key 、value都相等才删除。
//如果movable参数是false，在删除节点时，不移动其他节点

```java
final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
    // p 是待删除节点的前置节点
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    //如果哈希表不为空，则根据hash值算出的index下 有节点的话。
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        //node是待删除节点
        Node<K,V> node = null, e; K k; V v;
        //如果链表头的就是需要删除的节点
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;//将待删除节点引用赋给node
        else if ((e = p.next) != null) {//否则循环遍历 找到待删除节点，赋值给node
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        //如果有待删除节点node，  且 matchValue为false，或者值也相等
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            else if (node == p)//如果node ==  p，说明是链表头是待删除节点
                tab[index] = node.next;
            else//否则待删除节点在表中间
                p.next = node.next;
            ++modCount;//修改modCount
            --size;//修改size
            afterNodeRemoval(node);//LinkedHashMap回调函数
            return node;
        }
    }
    return null;
}
```
```java
    void afterNodeRemoval(Node<K,V> p) { }

```


以key value 为条件删除

```java
    @Override
    public boolean remove(Object key, Object value) {
        //这里传入了value 同时matchValue为true
        return removeNode(hash(key), key, value, true, true) != null;
    }

```

#### get：查

以key为条件，找到返回value，未找到返回null

```JAVA
 public V get(Object key) {
        HashMap.Node e;
     //传入扰动后的哈希值 和 key 找到目标节点Node
        return (e = this.getNode(hash(key), key)) == null ? null : e.value;
    }
```

```JAVA
//获取当前的结点
final HashMap.Node<K, V> getNode(int hash, Object key) {
        HashMap.Node[] tab;
        HashMap.Node first;
        int n;
    //tab指向哈希表，并且获取哈希表的长度，first为第一个结点的长度
        if ((tab = this.table) != null && (n = tab.length) > 0 && (first = tab[n - 1 & hash]) != null) {
            Object k;
            //如果桶中的第一个元素就相等，就返回
            if (first.hash == hash && ((k = first.key) == key || key != null && key.equals(k))) {
                return first;
            }

            HashMap.Node e;
            //否则的话就要继续查找
            if ((e = first.next) != null) {
                //如果第一个结点是树结点的话，就调用树形节点的get方法
                if (first instanceof HashMap.TreeNode) {
                    return ((HashMap.TreeNode)first).getTreeNode(hash, key);
                }

                do {
                    //使用do-while遍历链表中的所有的节点
                    if (e.hash == hash && ((k = e.key) == key || key != null && key.equals(k))) {
                        return e;
                    }
                } while((e = e.next) != null);
            }
        }

        return null;
    }
```

##### 注：

- 先计算哈希值

- 然后再用(n-1)&hash计算出桶的位置

- 在桶里的链表中遍历查找

- 时间复杂度一般和链表的长度有关，因此hash算法越好，元素分布就越均匀，get()方法就越快，不然遍历一条长链表，就及其耗时

  

  

##### 优缺点：

##### 优点：





##### 缺点：

- HashMap有很多优点，但是**不是同步的**

  > 当多线程并发访问一个哈希表时，需要在外部进行同步操作，否则会引起数据不同步的为问题
  >
  > 

#### 判断是否包含该key

```java
    public boolean containsKey(Object key) {
        return getNode(hash(key), key) != null;
    }
```



#### 判断是否包含value

   ```java
public boolean containsValue(Object value) {
        Node<K,V>[] tab; V v;
        //遍历哈希桶上的每一个链表
        if ((tab = table) != null && size > 0) {
            for (int i = 0; i < tab.length; ++i) {
                for (Node<K,V> e = tab[i]; e != null; e = e.next) {
                    //如果找到value一致的返回true
                    if ((v = e.value) == value ||
                        (value != null && value.equals(v)))
                        return true;
                }
            }
        }
        return false;
    }
   ```



replace方法
replace(K key, V oldValue, V newValue)
根据key和value定位到Node，然后将Node中的value用新value 替换，返回旧的value，否则返回空。

```java
public boolean replace(K key, V oldValue, V newValue) {
    Node<K,V> e; V v;
    if ((e = getNode(hash(key), key)) != null &&
        ((v = e.value) == oldValue || (v != null && v.equals(oldValue)))) {
        e.value = newValue;
        afterNodeAccess(e);
        return true;
    }
    return false;
}
```



#### replace(K key, V value)

根据key定位到Node，然后将Node中的value 替换，返回旧的value，否则返回空

```java
public V replace(K key, V value) {
    Node<K,V> e;
    if ((e = getNode(hash(key), key)) != null) {
        V oldValue = e.value;
        e.value = value;
        afterNodeAccess(e);
        return oldValue;
    }
    return null;
}
```

#### clear方法

clear 方法将每个数组元素置空

 ```java
public void clear() {
        Node<K,V>[] tab;
        modCount++;
        if ((tab = table) != null && size > 0) {
            size = 0;
            for (int i = 0; i < tab.length; ++i)
                tab[i] = null;
        }
    }
 ```



#### 备注：

###### 1，>>>和>>

##### 2，如何让HashMap实现同步

1.Collections.synchronizeMap(hashMap);

2.使用ConcurrentHashMap

## 二，HashMap与Hashtable的区别与联系

#### 1，继承的父类不同

HashMap继承自AbstractMap类，而HashTable是继承自Dictionary类。不过都同时实现了map，Cloneable(可复制)，Serializable(可序列化)这三个接口

 ![img](https://pic3.zhimg.com/80/v2-12c49eba132902bbea990a4d77ecce37_hd.jpg)

 ![img](https://pic3.zhimg.com/80/v2-d7d5449c9638ec4955288a3aa2ba9f13_hd.jpg)

   Dictionary类是一个已经被废弃的类（见其源码中的注释）。父类都被废弃，自然而然也没人用它的子类Hashtable了。 



#### 2，对外提供的接口不同

Hashtable比HashMap多提供了elments()和contains()方法

elments()方法继承自Hashtable的父类Dictionnary。elements()方法用于返回此Hashtable中的value的枚举

contains()方法判断该Hashtable是否包含传入的value。他的作用与containsValue()一致。事实上，containsValue()就是调用了一下contains()方法

 ![img](https://pic2.zhimg.com/80/v2-7964d6ae928711f9c25440b915375964_hd.jpg) 

#### 3，对空的键以及值的支持不同

**HashMap允许null key和null value，但是HashTable不允许**。

 ![img](https://pic4.zhimg.com/80/v2-ae3530e843efc2293e12f3babc415439_hd.jpg)

当key为null的时候，调用put()方法，运用 到下一步的时候就会返回一个空指针异常。因为拿一个null值去调用方法了

 ![img](https://pic3.zhimg.com/80/v2-5d6a22430b498d4b645ca09a43dcbcf1_hd.jpg) 

HashMap中，null可以作为键，这样的键只有一个；可以有一个或多个间对应的值为null。**因此，在HashMap中不能由get()方法来判断HashMap中是否存在某个值，而应该用containsKey()方法来判断**

#### 4，线程安全性不同

**Hashtable是线程安全的，它的每个方法中都加入了Synchronize方法。在多线程并发的环境下，可以直接使用Hashtable，不需要自己为它的方法实现同步**



**HashMap不是线程安全的，在多线程并发的环境下，可能会产生死锁等问题**。具体的原因在下一篇文章中会详细进行分析。使用HashMap时就必须要自己增加同步处理，



虽然HashMap不是线程安全的，但是它的效率会比Hashtable要好很多。这样设计是合理的。在我们的日常使用当中，大部分时间是单线程操作的。HashMap把这部分操作解放出来了。当需要多线程操作的时候可以使用线程安全的ConcurrentHashMap。**ConcurrentHashMap虽然也是线程安全的，但是它的效率比Hashtable要高好多倍。因为ConcurrentHashMap使用了分段锁，并不对整个数据进行锁定。**

#### 5，遍历方式的内部实现不同

**Hashtable、HashMap都使用了 Iterator。而由于历史原因，Hashtable还使用了Enumeration的方式 。**



**HashMap的Iterator是fail-fast迭代器。当有其它线程改变了HashMap的结构（增加，删除，修改元素），将会抛出ConcurrentModificationException。不过，通过Iterator的remove()方法移除元素则不会抛出ConcurrentModificationException异常。但这并不是一个一定发生的行为，要看JVM。**



JDK8之前的版本中，Hashtable是没有fast-fail机制的。在JDK8及以后的版本中 ，HashTable也是使用fast-fail的， 源码如下：

![img](https://pic1.zhimg.com/50/v2-e659500a1d3f8f6a38a3dff57cedbdb5_hd.jpg)![img](https://pic1.zhimg.com/80/v2-e659500a1d3f8f6a38a3dff57cedbdb5_hd.jpg)



modCount的使用类似于并发编程中的CAS（Compare and Swap）技术。**我们可以看到这个方法中，每次在发生增删改的时候都会出现modCount++的动作。而modcount可以理解为是当前hashtable的状态。每发生一次操作，状态就向前走一步。设置这个状态，主要是由于hashtable等容器类在迭代时，判断数据是否过时时使用的。尽管hashtable采用了原生的同步锁来保护数据安全。但是在出现迭代数据的时候，则无法保证边迭代，边正确操作。于是使用这个值来标记状态。一旦在迭代的过程中状态发生了改变，则会快速抛出一个异常，终止迭代行为。**



#### 6， 初始容量大小和每次扩充容量大小的不同

**Hashtable默认的初始大小为11，之后每次扩充，容量变为原来的2n+1。HashMap默认的初始化大小为16。之后每次扩充，容量变为原来的2倍。**



**创建时，如果给定了容量初始值，那么Hashtable会直接使用你给定的大小，而HashMap会将其扩充为2的幂次方大小。也就是说Hashtable会尽量使用素数、奇数。而HashMap则总是使用2的幂作为哈希表的大小。**



之所以会有这样的不同，是因为Hashtable和HashMap设计时的侧重点不同。**Hashtable的侧重点是哈希的结果更加均匀，使得哈希冲突减少。当哈希表的大小为素数时，简单的取模哈希的结果会更加均匀。而HashMap则更加关注hash的计算效率问题。在取模计算时，如果模数是2的幂，那么我们可以直接使用位运算来得到结果，效率要大大高于做除法。HashMap为了加快hash的速度，将哈希表的大小固定为了2的幂。当然这引入了哈希分布不均匀的问题，所以HashMap为解决这问题，又对hash算法做了一些改动。这从而导致了Hashtable和HashMap的计算hash值的方法不同**

#### 7， 计算hash值的方法不同

为了得到元素的位置，首先需要根据元素的 KEY计算出一个hash值，然后再用这个hash值来计算得到最终的位置。



**Hashtable直接使用对象的hashCode。hashCode是JDK根据对象的地址或者字符串或者数字算出来的int类型的数值。然后再使用除留余数发来获得最终的位置。**

![img](https://pic4.zhimg.com/50/v2-df830c3a8054ee1a9f482dc3ccd66bf3_hd.jpg)![img](https://pic4.zhimg.com/80/v2-df830c3a8054ee1a9f482dc3ccd66bf3_hd.jpg)



**Hashtable在计算元素的位置时需要进行一次除法运算，而除法运算是比较耗时的。**

**HashMap为了提高计算效率，将哈希表的大小固定为了2的幂，这样在取模预算时，不需要做除法，只需要做位运算。位运算比除法的效率要高很多。**



HashMap的效率虽然提高了，但是hash冲突却也增加了。因为它得出的hash值的低位相同的概率比较高，而计算位运算



为了解决这个问题，HashMap重新根据hashcode计算hash值后，又对hash值做了一些运算来打散数据。使得取得的位置更加分散，从而减少了hash冲突。**当然了，为了高效，HashMap只做了一些简单的位处理。从而不至于把使用2 的幂次方带来的效率提升给抵消掉**。



![img](https://pic1.zhimg.com/50/v2-95305c0eddb50900fe62b4ac879c005a_hd.jpg)![img](https://pic1.zhimg.com/80/v2-95305c0eddb50900fe62b4ac879c005a_hd.jpg)

## 三，HashMap与HashSet的区别

**HashMap和HashSet的区别**

HashMap实现了Map接口HashSet实现了Set接口

HashMap储存键值对HashSet仅仅存储对象

使用put()方法将元素放入map中使用add()方法将元素放入set中

**HashMap中使用键对象来计算hashcode值HashSet使用成员对象来计算hashcode值**，对于两个对象来说hashcode可能相同，所以equals()方法用来判断对象的相等性，如果两个对象不同的话，那么返回false

**HashMap比较快，因为是使用唯一的键来获取对象HashSet较HashMap来说比较慢**



## 四，面试相关

#### 1，“你用过HashMap吗？” “什么是HashMap？你为什么用到它？”**

几乎每个人都会回答“是的”，然后回答HashMap的一些特性，譬如HashMap可以接受null键值和值，而Hashtable则不能；HashMap是非synchronized;HashMap很快；以及HashMap储存的是键值对等等。这显示出你已经用过HashMap，而且对它相当的熟悉。但是面试官来个急转直下，从此刻开始问出一些刁钻的问题，关于HashMap的更多基础的细节。面试官可能会问出下面的问题：

#### **2，“你知道HashMap的工作原理吗？” “你知道HashMap的get()方法的工作原理吗？”**

**HashMap是基于hashing的原理，我们使用put(key, value)存储对象到HashMap中，使用get(key)从HashMap中获取对象。当我们给put()方法传递键和值时，我们先对键调用hashCode()方法，返回的hashCode用于找到bucket位置来储存Entry对象**。”这里关键点在于指出，**HashMap是在bucket中储存键对象和值对象，作为Map.Entry**。这一点有助于理解获取对象的逻辑。如果你没有意识到这一点，或者错误的认为仅仅只在bucket中存储值的话，你将不会回答如何从HashMap中获取对象的逻辑。这个答案相当的正确，也显示出面试者确实知道hashing以及HashMap的工作原理。但是这仅仅是故事的开始，当面试官加入一些Java程序员每天要碰到的实际场景的时候，错误的答案频现。下个问题可能是关于HashMap中的碰撞探测(collision detection)以及碰撞的解决方法：

#### **3，“当两个对象的hashcode相同会发生什么？”**

**“因为hashcode相同，所以它们的bucket位置相同，‘碰撞’会发生。因为HashMap使用链表存储对象，这个Entry(包含有键值对的Map.Entry对象)会存储在链表中**。”这个答案非常的合理，虽然有很多种处理碰撞的方法，这种方法是最简单的，也正是HashMap的处理方法。但故事还没有完结，面试官会继续问：

#### **4，“如果两个键的hashcode相同，你如何获取值对象？”**

面试者会回答：当我们调用get()方法，HashMap会使用键对象的hashcode找到bucket位置，然后获取值对象。面试官提醒他如果有两个值对象储存在同一个bucket，他给出答案:将会遍历链表直到找到值对象。面试官会问因为你并没有值对象去比较，你是如何确定确定找到值对象的？除非面试者直到HashMap在链表中存储的是键值对，否则他们不可能回答出这一题。

其中一些记得这个重要知识点的面试者会说，**找到bucket位置之后，会调用keys.equals()方法去找到链表中正确的节点，最终找到要找的值对象**。完美的答案！

许多情况下，面试者会在这个环节中出错，因为他们混淆了hashCode()和equals()方法。因为在此之前hashCode()屡屡出现，而**equals()方法仅仅在获取值对象的时候才出现**。一些优秀的开发者会指出使用不可变的、声明作final的对象，并且采用合适的equals()和hashCode()方法的话，将会减少碰撞的发生，提高效率。不可变性使得能够缓存不同键的hashcode，这将提高整个获取对象的速度，使用String，Interger这样的wrapper类作为键是非常好的选择。

#### 5，**“如果HashMap的大小超过了负载因子(load factor)定义的容量，怎么办？”**

**默认的负载因子大小为0.75，也就是说，当一个map填满了75%的bucket时候，和其它集合类(如ArrayList等)一样，将会创建原来HashMap大小的两倍的bucket数组，来重新调整map的大小，并将原来的对象放入新的bucket数组中。这个过程叫作rehashing，因为它调用hash方法找到新的bucket位置**。

#### **6，“你了解重新调整HashMap大小存在什么问题吗？”**

**当多线程的情况下，可能产生条件竞争(race condition)。**

当重新调整HashMap大小的时候，确实存在条件竞争，因为如果两个线程都发现HashMap需要重新调整大小了，它们会同时试着调整大小。**在调整大小的过程中，存储在链表中的元素的次序会反过来，因为移动到新的bucket位置的时候，HashMap并不会将元素放在链表的尾部，而是放在头部，这是为了避免尾部遍历(tail traversing)。如果条件竞争发生了，那么就死循环了**。这个时候，你可以质问面试官，为什么这么奇怪，要在多线程的环境下使用HashMap呢？

#### **7，我们可以使用CocurrentHashMap来代替Hashtable吗？**

这是另外一个很热门的面试题，因为ConcurrentHashMap越来越多人用了。我们知道Hashtable是synchronized的，但是ConcurrentHashMap同步性能更好，因为**它仅仅根据同步级别对map的一部分进行上锁。ConcurrentHashMap当然可以代替HashTable，但是HashTable提供更强的线程安全性**。

