[toc]

## ArrayList源码解析

#### 定义

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

ArrayList实际上**是一个动态数组，容量可以动态的增长，其继承了AbstractList，实现了List, RandomAccess, Cloneable, java.io.Serializable这些接口。**



实现了所有可选列表操作，并**允许包括 null 在内的所有元素**。除了实现 List 接口外，此类**还提供一些方法来操作内部用来存储列表的数组的大小**。



ArrayList 实现了**RandmoAccess接口**，即提供了随机访问功能。RandmoAccess是java中用来被List实现，为List提供**快速访问功能**的。在ArrayList中，我们即可以通过元素的序号快速获取元素对象，这就是快速随机访问。

　　ArrayList 实现了**Cloneable接口**，即覆盖了函数clone()，**能被克隆**。

　　ArrayList 实现**java.io.Serializable接口**，这意味着ArrayList**支持序列化**，**能通过序列化去传输**。



> 1.RandomAccess接口，标记接口，表明List提供了随机访问功能，也就是通过下标获取元素对象的功能。之所以是标记接口，是该类本来就具有某项能力，使用接口对其进行标签化，便于其他的类对其进行识别（instanceof）。 
>
> 2.ArrayList数组实现，本身就有通过下标随机访问任意元素的功能。那么需要细节上注意的就是随机下标访问和顺序下标访问（LinkedList）的不同了。也就是为什么LinkedList最好不要使用循环遍历，而是用迭代器遍历的原因。 
>
> 3.实现RandomAccess同时意味着一些算法可以通过类型判断进行一些针对性优化，例子有Collections的shuffle方法，源代码就不粘贴了，简单说就是，如果实现RandomAccess接口就下标遍历，反之迭代器遍历 实现了Cloneable, java.io.Serializable意味着可以被克隆和序列化。


与Java中的数组相比，它的容量能动态增长。实现了所有可选类表操作，并允许包括null在内的所有元素。除了实现List接口外，此类**还提供一些方法来操作内部用来存储列表的数组的大小**。在添加大量元素前，应用程序可以使用ensureCapacity 操作来增加 ArrayList 实例的容量。这可以减少递增式再分配的数量。





​	每一个ArrayList实例都有一个容量，该容量是指用来存储列表元素的数组的大小。总是至少等于列表的大小。随着向ArrayList中不断添加元素，其容量也自动增长。自动增长会带来数据向新数组的重新拷贝，因此，如果可预知数据量的多少，可在构造ArrayList时指定其容量。在添加大量元素前，应用程序也可以使用ensureCapacity操作来增加ArrayList实例的容量，这可以减少递增式再分配的数量。

​	注意，此实现**不是同步**的。如果多个线程同时访问一个ArrayList实例，而其中至少一个线程从结构上修改了列表，那么它必须保持外部同步。这通常是通过同步那些用来封装列表的对象来实现的。但如果没有这样的对象存在，则该列表需要运用{@link Collections#synchronizedList Collections.synchronizedList}来进行“包装”，该方法最好是在创建列表对象时完成，为了避免对列表进行突发的非同步操作。

```java
List list = Collections.synchronizedList(new ArrayList(...));
```

建议在单线程中才使用ArrayList，而在多线程中可以选择Vector或者CopyOnWriteArrayList。

在我们学数据结构的时候就知道了线性表的顺序存储，**插入删除**元素的时间复杂度为**O（n）**,**求表长以及增加元素**，**取第 i 元素**的时间复杂度为**O（1）**

　　ArrayList 继承了AbstractList，实现了List。它是一个数组队列，提供了相关的添加、删除、修改、遍历等功能。

　　和Vector不同，**ArrayList中的操作不是线程安全的**！所以，**建议在单线程中才使用ArrayList，而在多线程中可以选择Vector或者CopyOnWriteArrayList。**





**因其底层数据结构是数组，所以可想而知，它是占据一块连续的内存空间（容量就是数组的length），所以它也有数组的缺点，空间效率不高。**

**由于数组的内存连续，可以根据下标以O1的时间读写(改查)元素，因此时间效率很高。**

**当集合中的元素超出这个容量，便会进行扩容操作。扩容操作也是ArrayList 的一个性能消耗比较大的地方，所以若我们可以提前预知数据的规模，应该通过public ArrayList(int initialCapacity) {}构造方法，指定集合的大小，去构建ArrayList实例，以减少扩容次数，提高效率。**

或者在需要扩容的时候，手动调用public void ensureCapacity(int minCapacity) {}方法扩容。
不过该方法是ArrayList的API，不是List接口里的，所以使用时需要强转:
((ArrayList)list).ensureCapacity(30);

当每次修改结构时，增加导致扩容，或者删，都会修改modCount。


#### 构造方法

 

```java
//如果是无参构造方法创建对象的话，ArrayList的初始化长度为10，这是一个静态常量
private static final int DEFAULT_CAPACITY = 10;

//在这里可以看到我们不解的EMPTY_ELEMENTDATA实际上是一个空的对象数组
    private static final Object[] EMPTY_ELEMENTDATA = {};

//保存ArrayList数据的对象数组缓冲区 空的ArrayList的elementData = EMPTY_ELEMENTDATA 这就是为什么说ArrayList底层是数组实现的了。elementData的初始容量为10，大小会根据ArrayList容量的增长而动态的增长。
//存储集合元素的底层实现：真正存放元素的数组
    private transient Object[] elementData;
//集合的长度
    private int size;


//默认构造方法
public ArrayList() {
    //默认构造方法只是简单的将 空数组赋值给了elementData
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

//空数组
private static final Object[] EMPTY_ELEMENTDATA = {};
//带初始容量的构造方法
public ArrayList(int initialCapacity) {
    //如果初始容量大于0，则新建一个长度为initialCapacity的Object数组.
    //注意这里并没有修改size(对比第三个构造函数)
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {//如果容量为0，直接将EMPTY_ELEMENTDATA赋值给elementData
        this.elementData = EMPTY_ELEMENTDATA;
    } else {//容量小于0，直接抛出异常
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}

//利用别的集合类来构建ArrayList的构造函数
public ArrayList(Collection<? extends E> c) {
    //直接利用Collection.toArray()方法得到一个对象数组，并赋值给elementData 
    elementData = c.toArray();
    //因为size代表的是集合元素数量，所以通过别的集合来构造ArrayList时，要给size赋值
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)//这里是当c.toArray出错，没有返回Object[]时，利用Arrays.copyOf 来复制集合c中的元素到elementData数组中
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        //如果集合c元素数量为0，则将空数组EMPTY_ELEMENTDATA赋值给elementData 
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

小结一下，构造函数走完之后，会构建出数组elementData和数量size。

这里大家要注意一下Collection.toArray()这个方法，在Collection子类各大集合的源码中，高频使用了这个方法去获得某Collection的所有元素。

关于方法：Arrays.copyOf(elementData, size, Object[].class)，就是根据class的类型来决定是new 还是反射去构造一个泛型数组，同时利用native函数，批量赋值元素至新数组中。
如下：

```java
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
    @SuppressWarnings("unchecked")
    //根据class的类型来决定是new 还是反射去构造一个泛型数组
    T[] copy = ((Object)newType == (Object)Object[].class)
        ? (T[]) new Object[newLength]
        : (T[]) Array.newInstance(newType.getComponentType(), newLength);
    //利用native函数，批量赋值元素至新数组中。
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));
    return copy;
}
```


值得注意的是，System.arraycopy也是一个很高频的函数，大家要留意一下。

```java
public static native void arraycopy(Object src,  int  srcPos,
                                    Object dest, int destPos,
                                    int length);
```
![2018-01-11_110237](https://user-gold-cdn.xitu.io/2018/1/12/160ea59dacef93e0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



可以说ArrayList的作者真的是很贴心，连缓存都处理好了，多次new出来的新对象，都执行同一个引用。

## 常用API

#### 1 增：add

**每次 add之前，都会判断add后的容量，是否需要扩容。**

```java
private static final int MAX_ARRAY_SIZE = 2147483639;
private static final int DEFAULT_CAPACITY = 10;
private static final Object[] EMPTY_ELEMENTDATA = new Object[0];
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = new Object[0];


public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;//在数组末尾追加一个元素，并修改size
    return true;
}

    private static final int DEFAULT_CAPACITY = 10;//默认扩容容量 10

//初始长度是10，size默认值0，假定添加的是第一个元素，
//那么minCapacity=1 这是最小容量 如果小于这个容量就会报错
//如果底层数组就是默认数组，那么选一个大的值，就是10
private void ensureCapacityInternal(int minCapacity) {
        //利用 == 可以判断数组是否是用默认构造函数初始化的
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }


private void ensureExplicitCapacity(int minCapacity) {
    //记录修改的次数
    modCount++;//如果确定要扩容，会修改modCount 

    // overflow-conscious code
    //如果传来的值大于初始长度，执行grow方法(参数为传来的值)扩容
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

//需要扩容的话，默认扩容一半
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    //新的容量是在原有的容量基础上+50% 右移一位就是二分之一
    int newCapacity = oldCapacity + (oldCapacity >> 1);//默认扩容一半
    //如果新容量小于最小容量，按照最小容量进行扩容
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    //重点：调用工具类Arrays的copyOf扩容
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);//拷贝，扩容，构建一个新数组，
}


private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}

public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
    @SuppressWarnings("unchecked")
    //根据class的类型来决定是new 还是反射去构造一个泛型数组
    T[] copy = ((Object)newType == (Object)Object[].class)
        ? (T[]) new Object[newLength]
        : (T[]) Array.newInstance(newType.getComponentType(), newLength);
    //利用native函数，批量赋值元素至新数组中。
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));
    return copy;
}

```



##### add(int index, E element)

添加到指定的位置

```java
public void add(int index, E element) {
  //判断索引是否越界，如果越界就会抛出下标越界异常
  rangeCheckForAdd(index);
//扩容检查
  ensureCapacityInternal(size + 1);  // Increments modCount!!
  //将指定下标空出 具体作法就是index及其后的所有元素后移一位
  System.arraycopy(elementData, index, elementData, index + 1,size - index);
  //将要添加元素赋值到空出来的指定下标处
  elementData[index] = element;
  //长度加1
  size++;
}
//判断是否出现下标是否越界
private void rangeCheckForAdd(int index) {
  //如果下标超过了集合的尺寸 或者 小于0就是越界  
  if (index > size || index < 0)
    throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```

```java
public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount //确认是否需要扩容
    System.arraycopy(a, 0, elementData, size, numNew);// 复制数组完成复制
    size += numNew;
    return numNew != 0;
}
```

```java
public boolean addAll(int index, Collection<? extends E> c) {
    rangeCheckForAdd(index);//越界判断

    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount //确认是否需要扩容

    int numMoved = size - index;
    if (numMoved > 0)
        System.arraycopy(elementData, index, elementData, index + numNew,
                         numMoved);//移动（复制)数组

    System.arraycopy(a, 0, elementData, index, numNew);//复制数组完成批量赋值
    size += numNew;
    return numNew != 0;
}
```

**总结：**
add、addAll。
先判断是否越界，是否需要扩容。
如果扩容， 就复制数组。
然后设置对应下标元素值。

值得注意的是：
1 **如果需要扩容的话，默认扩容一半。如果扩容一半不够，就用目标的size作为扩容后的容量。**
2 **在扩容成功后，会修改modCount**

#### 2 删:remove(int index)

ArrayList支持两种删除元素的方式：

- remove(int index) 按照下标删除 常用
- remove(Object o) 按照元素删除 会删除和参数匹配的第一个元素

```java
public E remove(int index) {
    rangeCheck(index);//判断是否越界
    modCount++;//修改次数统计
    E oldValue = elementData(index);//获取该下标的值(准备删除)
    //计算出需要移动的元素个数 （因为需要将后续元素左移一位 此处计算出来的是后续元素的位数）
    int numMoved = size - index - 1;
    //如果这个值大于0 说明后续有元素需要左移  是0说明被移除的对象就是最后一位元素
    if (numMoved > 0)
        //索引index只有的所有元素左移一位  覆盖掉index位置上的元素
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);//用复制 覆盖数组数据
    elementData[--size] = null; // clear to let GC do its work  
    //置空原尾部数据 不再强引用， 可以GC掉
    //返回index位置上的元素
    return oldValue;
}
    //根据下标从数组取值 并强转
    E elementData(int index) {
        return (E) elementData[index];
    }

//移除特定的元素
//删除该元素在数组中第一次出现的位置上的数据。 如果有该元素返回true，如果false。
public boolean remove(Object o) {
    //如果元素为null，遍历数组移除第一个null
    if (o == null) {
        for (int index = 0; index < size; index++)
            //遍历找到第一个null元素的下标  调用下标移除元素的方法
            if (elementData[index] == null) {
                fastRemove(index);//根据index删除元素
                return true;
            }
    } else {
        //找到元素对应的下标  调用下标移除元素的方法
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}
//按照下标移除元素
//不会越界 不用判断 ，也不需要取出该元素。
private void fastRemove(int index) {
    modCount++;//修改modCount
    int numMoved = size - index - 1;//计算要移动的元素数量
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);//以复制覆盖元素 完成删除
    elementData[--size] = null; // clear to let GC do its work  //置空 不再强引用
}

//批量删除
public boolean removeAll(Collection<?> c) {
    Objects.requireNonNull(c);//判空
    return batchRemove(c, false);
}
//批量移动
private boolean batchRemove(Collection<?> c, boolean complement) {
    final Object[] elementData = this.elementData;
    int r = 0, w = 0;//w 代表批量删除后 数组还剩多少元素
    boolean modified = false;
    try {
        //高效的保存两个集合公有元素的算法
        for (; r < size; r++)
            if (c.contains(elementData[r]) == complement) // 如果 c里不包含当前下标元素， 
                elementData[w++] = elementData[r];//则保留
    } finally {
        // Preserve behavioral compatibility with AbstractCollection,
        // even if c.contains() throws.
        if (r != size) { //出现异常会导致 r !=size , 则将出现异常处后面的数据全部复制覆盖到数组里。
            System.arraycopy(elementData, r,
                             elementData, w,
                             size - r);
            w += size - r;//修改 w数量
        }
        if (w != size) {//置空数组后面的元素
            // clear to let GC do its work
            for (int i = w; i < size; i++)
                elementData[i] = null;
            modCount += size - w;//修改modCount
            size = w;// 修改size
            modified = true;
        }
    }
    return modified;
}
```




从这里我们也可以看出，当用来作为删除元素的集合里的元素多余被删除集合时，也没事，只会删除它们共同拥有的元素。

小结：
1 删除操作一定会修改modCount，且可能涉及到数组的复制，相对低效。
2 批量删除中，涉及高效的保存两个集合公有元素的算法，可以留意一下。

#### 3 改  set

不会修改modCount，相对增删是高效的操作。

```JAVA
public E set(int index, E element) {
    rangeCheck(index);//越界检查
    E oldValue = elementData(index); //取出元素 
    elementData[index] = element;//覆盖元素
    return oldValue;//返回元素
}

```

#### 4 查  get

不会修改modCount，相对增删是高效的操作。

```JAVA
public E get(int index) {
    rangeCheck(index);//越界检查
    return elementData(index); //下标取数据
}
E elementData(int index) {
    return (E) elementData[index];
}
```



#### 5 清空：clear

会修改modCount。

```JAVA
public void clear() {
    modCount++;//修改modCount
    // clear to let GC do its work
    for (int i = 0; i < size; i++)  //将所有元素置null
        elementData[i] = null;

    size = 0; //修改size 

}
```

#### 6 包含 contain

```JAVA
//普通的for循环寻找值，只不过会根据目标对象是否为null分别循环查找。
public boolean contains(Object o) {
    return indexOf(o) >= 0;
}
//普通的for循环寻找值，只不过会根据目标对象是否为null分别循环查找。
public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```



#### 7 判空 isEmpty()

```java
public boolean isEmpty() {
    return size == 0;
}
```



#### 8 迭代器 Iterator.

```java
public Iterator<E> iterator() {
    return new Itr();
}
/**

 * An optimized version of AbstractList.Itr
   */
   private class Itr implements Iterator<E> {
   int cursor;       // index of next element to return //默认是0
   int lastRet = -1; // index of last element returned; -1 if no such  //上一次返回的元素 (删除的标志位)
   int expectedModCount = modCount; //用于判断集合是否修改过结构的 标志

   public boolean hasNext() {
       return cursor != size;//游标是否移动至尾部
   }

   @SuppressWarnings("unchecked")
   public E next() {
       checkForComodification();
       int i = cursor;
       if (i >= size)//判断是否越界
           throw new NoSuchElementException();
       Object[] elementData = ArrayList.this.elementData;
       if (i >= elementData.length)//再次判断是否越界，在 我们这里的操作时，有异步线程修改了List
           throw new ConcurrentModificationException();
       cursor = i + 1;//游标+1
       return (E) elementData[lastRet = i];//返回元素 ，并设置上一次返回的元素的下标
   }

   public void remove() {//remove 掉 上一次next的元素
       if (lastRet < 0)//先判断是否next过
           throw new IllegalStateException();
       checkForComodification();//判断是否修改过

       try {
           ArrayList.this.remove(lastRet);//删除元素 remove方法内会修改 modCount 所以后面要更新Iterator里的这个标志值
           cursor = lastRet; //要删除的游标
           lastRet = -1; //不能重复删除 所以修改删除的标志位
           expectedModCount = modCount;//更新 判断集合是否修改的标志，
       } catch (IndexOutOfBoundsException ex) {
           throw new ConcurrentModificationException();
       }

   }
   //判断是否修改过了List的结构，如果有修改，抛出异常
   final void checkForComodification() {
       if (modCount != expectedModCount)
           throw new ConcurrentModificationException();
   }
   }
   
```



#### 总结



##### **1，System.arraycopy()和Arrays.copyOf()方法**

　　通过上面源码我们发现这两个实现数组复制的方法被广泛使用而且很多地方都特别巧妙。比如下面add(int index, E element)方法就很巧妙的用到了arraycopy()方法让数组自己复制自己实现让index开始之后的所有成员后移一个位置：

![img](ArrayList%E2%80%94%E2%80%94LinkedList.assets/v2-4a459c0a464d763c31229da63788e486_hd.jpg)

又如toArray()方法中用到了copyOf()方法

![img](ArrayList%E2%80%94%E2%80%94LinkedList.assets/v2-f28c0e3395a45360d332adbdf568e2a2_hd.jpg)

**两者联系与区别：**

**联系：**

看两者源代码可以发现copyOf()内部调用了System.arraycopy()方法
**区别：**

1，arraycopy()需要目标数组，将原数组拷贝到你自己定义的数组里，而且可以选择拷贝的起点和长度以及放入新数组中的位置

2，copyOf()是系统自动在内部新建一个数组，并返回该数组。



##### 2.0

1. 增删改查中， 增导致扩容，则会修改modCount，删一定会修改。 改和查一定不会修改modCount。

2. 扩容操作会导致数组复制，批量删除会导致 找出两个集合的交集，以及数组复制操作，因此，增、删都相对低效。 而 改、查都是很高效的操作。

   >
   >
   >因此，结合特点，在使用中，以Android中最常用的展示列表为例，列表滑动时需要展示每一个Item（element）的数组，所以 查 操作是最高频的。相对来说，增操作 只有在列表加载更多时才会用到 ，而且是在列表尾部插入，所以也不需要移动数据的操作。而删操作则更低频。 故选用ArrayList作为保存数据的结构。

3. 实现了RandomAccess接口，底层又是数组，get读取元素性能很好

4. **线程不安全，所有的方法均不是同步方法也没有加锁，因此多线程下慎用**

5. 顺序添加很方便

6. **删除和插入需要复制数组 性能很差（可以使用LinkindList）**

7. 在面试中还有可能会问到和Vector的区别，我大致看了一下Vector的源码，内部也是数组做的，区别在于Vector在API上都加了synchronized所以它是线程安全的，以及Vector扩容时，是翻倍size，而ArrayList是扩容50%。



##### 2，扩容具体指的是什么？



可以看到，`ArrayList`里面有两个概念，**一个是`capacity`，它表示的就是“容量”，其实质是数组`elementData`的长度。而`size`则表示的“存放的元素的个数”**。

因为Java中，数组操作不能越界，所以我们必须要保证在插入操作的时候，不会抛出数组越界异常。



##### 3，为什么ArrayList的elementData是用transient修饰的？

**transient修饰的属性意味着不会被序列化，也就是说在序列化ArrayList的时候，不序列化elementData。**

为什么要这么做呢？

1. elementData不总是满的，每次都序列化，会浪费时间和空间
2. 重写了writeObject  保证序列化的时候虽然不序列化全部 但是有的元素都序列化

所以说不是不序列化 而是不全部序列化。

```java
private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
// Write out element count, and any hidden stuff
int expectedModCount = modCount;
s.defaultWriteObject();
        // Write out array length
       s.writeInt(elementData.length);
    // Write out all elements in the proper order.
for (int i=0; i<size; i++)
           s.writeObject(elementData[i]);
    if (modCount != expectedModCount) {
           throw new ConcurrentModificationException();
    }
}
```

#### 4，ArrayList和Vector的区别

##### 标准答案

1. ArrayList是线程不安全的，Vector是线程安全的
2. 扩容时候ArrayList扩0.5倍，Vector扩1倍

那么问题来了

**ArrayList有没有办法线程安全？**

Collections工具类有一个synchronizedList方法

可以把list变为线程安全的集合，但是意义不大，因为可以使用Vector

**Vector为什么是线程安全的？**

老实讲，抛开多线程	它俩区别没多大，但是多线程下就不一样了，那么Vector是如何实现线程安全的，我们来看几个关键方法

```java
public synchronized boolean add(E e) {
  modCount++;
  ensureCapacityHelper(elementCount + 1);
  elementData[elementCount++] = e;
  return true;
}

public synchronized E remove(int index) {
  modCount++;
  if (index >= elementCount)
    throw new ArrayIndexOutOfBoundsException(index);
  E oldValue = elementData(index);

  int numMoved = elementCount - index - 1;
  if (numMoved > 0)
    System.arraycopy(elementData, index+1, elementData, index,
                     numMoved);
  elementData[--elementCount] = null; // Let gc do its work

  return oldValue;
}
```

就代码实现上来说，和ArrayList并没有很多逻辑上的区别，但是在Vector的关键方法都使用了synchronized修饰。




## LinkedList源码解析

![img](ArrayList%E2%80%94%E2%80%94LinkedList.assets/image-20200304000956592.png)

#### 定义：

**LinkedList 是线程不安全的，允许元素为null的双向链表**。

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable

```

- 继承了AbstractSequentialList

- 实现List，Deque，Cloneable, Serializable接口

  - List 接口，能对它进行队列操作

  - Deque 接口，即能将LinkedList当作双端队列使用。
  - Cloneable接口，即覆盖了函数clone()，能克隆。
  - java.io.Serializable接口，这意味着LinkedList支持序列化，能通过序列化去传输。
  - LinkedList 是非同步的。
    

**和ArrayList比，没有实现RandomAccess所以其以下标，随机访问元素速度较慢。**



因其底层数据结构是链表，所以可想而知，它的增删只需要移动指针即可，故时间效率较高。不需要批量扩容，也不需要预留空间，所以空间效率比ArrayList高。

缺点就是**需要随机访问元素时，时间效率很低**，**虽然底层在根据下标查询Node的时候，会根据index判断目标Node在前半段还是后半段，然后决定是顺序还是逆序查询，以提升时间效率。不过随着n的增大，总体时间效率依然很低。**

当每次增、删时，都会修改modCount。



 LinkedList相对于ArrayList来说，是可以快速添加，删除元素，ArrayList添加删除元素的话需移动数组元素，可能还需要考虑到扩容数组长度。 

- AbstractSequentialList相较于AbstractList`(ArrayList的父类)`,**只支持次序访问，而不支持随机访问**，因为它的 `get(int index)` ,`set(int index, E element)`, `add(int index, E element)`, `remove(int index)` 都是基于迭代器实现的。**所以在LinkedList使用迭代器遍历更快，而ArrayList使用get (i)更快。**
-  接口方面，LinkedList多继承了一个Deque接口，所以实现了双端队列的一系列方法。

#### 基本数据结构

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable{   
	//当前有多少个节点
    transient int size = 0;
    //第一个节点
    transient Node<E> first;
    //最后一个节点
    transient Node<E> last;
	//省略内部类和方法。。
}
```

LinkedList中主要定义了头节点指针，尾节点指针，以及size用于计数链表中节点个数。那么每一个Node的结构如何呢？

```java
private static class Node<E> {
    //元素值    
    E item;
    //后置节点
    Node<E> next;
    //前置节点
    Node<E> prev;
    
    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;  //当前节点值
        this.next = next; //后继节点
        this.prev = prev;//前驱节点
    }
}
```

可以看出，这是一个典型的双向链表的节点。



![Node节点](https://user-gold-cdn.xitu.io/2017/12/13/1604dbfc2215852c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)





#### 构造函数

```java
public LinkedList() {
}

public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);  //把集合中所有节点添加到list中
}
```





#### 增

1 addAll
接上文，先说addAll



```java
//addAll ,在尾部批量增加
public boolean addAll(Collection<? extends E> c) {
    return addAll(size, c);//以size为插入下标，插入集合c中所有元素
}
//以index为插入下标，插入集合c中所有元素
public boolean addAll(int index, Collection<? extends E> c) {
    checkPositionIndex(index);//检查越界 [0,size] 闭区间

Object[] a = c.toArray();//拿到目标集合数组
int numNew = a.length;//新增元素的数量
if (numNew == 0)
    return false;

Node<E> pred, succ;  //index节点的前置节点，后置节点
if (index == size) { //在链表尾部追加数据
    succ = null;  //size节点（队尾）的后置节点一定是null
    pred = last;//前置节点是队尾
} else {
    succ = node(index);//取出index节点，作为后置节点
    pred = succ.prev; //前置节点是，index节点的前一个节点
}
//链表批量增加，是靠for循环遍历原数组，依次执行插入节点操作。对比ArrayList是通过System.arraycopy完成批量增加的
for (Object o : a) {//循环插入节点
    @SuppressWarnings("unchecked") E e = (E) o;
    Node<E> newNode = new Node<>(pred, e, null);//以前置节点 和 元素值e，构建new一个新节点，
    if (pred == null) //如果前置节点是空，说明是头结点
        first = newNode;
    else//否则 前置节点的后置节点设置问新节点
        pred.next = newNode;
    pred = newNode;//步进，当前的节点为前置节点了，为下次添加节点做准备
}
//如果插入最后面，则last节点是最后一个插入的节点
if (succ == null) {//循环结束后，判断，如果后置节点是null。 说明此时是在队尾append的。
    last = pred; //则设置尾节点
} else {
    pred.next = succ; // 否则是在队中插入的节点 ，更新前置节点 后置节点
    succ.prev = pred; //更新后置节点的前置节点
}

size += numNew;  // 修改数量size
modCount++;  //修改modCount
return true;
}
//根据index 查询出Node，
Node<E> node(int index) {
    // assert isElementIndex(index);
//通过下标获取某个node 的时候，（增、查 ），会根据index处于前半段还是后半段 进行一个折半，以提升查询效率
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}

private void checkPositionIndex(int index) {
    if (!isPositionIndex(index))
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
private boolean isPositionIndex(int index) {
    return index >= 0 && index <= size;  //插入时的检查，下标可以是size [0,size]
}
```


小结：

链表批量增加，是靠for循环遍历原数组，依次执行插入节点操作。对比ArrayList是通过System.arraycopy完成批量增加的
通过下标获取某个node 的时候，（add select），会根据index处于前半段还是后半段 进行一个折半，以提升查询效率

#### 2 插入单个节点add

```java
//在尾部插入一个节点： add
public boolean add(E e) {
    linkLast(e);
    return true;
}

//生成新节点 并插入到 链表尾部， 更新 last/first 节点。
void linkLast(E e) { 
    final Node<E> l = last; //记录原尾部节点
    final Node<E> newNode = new Node<>(l, e, null);//以原尾部节点为新节点的前置节点
    last = newNode;//更新尾部节点
    if (l == null)//若原链表为空链表，需要额外更新头结点first
        first = newNode;
    else//否则更新原尾节点的后置节点为现在的尾节点（新节点）,完成新增节点操作
        l.next = newNode;
    size++;//修改size
    modCount++;//修改modCount
}
```






```java
   //在指定下标，index处，插入一个节点
    public void add(int index, E element) {
        checkPositionIndex(index);//检查下标是否越界[0,size]
        if (index == size)//在尾节点后插入
            linkLast(element);
        else//在中间插入
            linkBefore(element, node(index));
    }
    //在succ节点前，插入一个新节点e
    void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        //保存后置节点的前置节点
        final Node<E> pred = succ.prev;
        //以前置和后置节点和元素值e 构建一个新节点
        final Node<E> newNode = new Node<>(pred, e, succ);
        //新节点new是原节点succ的前置节点
        succ.prev = newNode;
        if (pred == null)//如果之前的前置节点是空，说明succ是原头结点。所以新节点是现在的头结点
            first = newNode;
        else//否则构建前置节点的后置节点为new
            pred.next = newNode;
        size++;//修改数量
        modCount++;//修改modCount
    
```



小结：

* 增一定会修改modCount

#### 删 remove

remove方法 

- remove()`删除第一个元素，直接调用的removeFirst()方法`
- remove(int index)`删除下标index上的元素，如果下边越界，会抛IndexOutOfBoundsException异常`
- remove(Object o)`删除第一个匹配到的元素o`
- removeFirst()`删除第一个元素（如果list为空，会抛出NoSuchElementException异常）`
- removeFirstOccurrence(Object o)`直接调用remove(o)`
- removeLast()`删除最后一个元素（如果list为空，会抛出NoSuchElementException异常）`
- removeLastOccurrence(Object o)`删除最后一个匹配到的元素`




```java
//删：remove目标节点
public E remove(int index) {
    checkElementIndex(index);//检查是否越界 下标[0,size)
    return unlink(node(index));//从链表上删除某节点
}
//从链表上删除x节点
E unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item; //当前节点的元素值
    final Node<E> next = x.next; //当前节点的后置节点
    final Node<E> prev = x.prev;//当前节点的前置节点
if (prev == null) { //如果前置节点为空(说明当前节点原本是头结点)
    first = next;  //则头结点等于后置节点 
} else { 
    prev.next = next;
    x.prev = null; //将当前节点的 前置节点置空
}

if (next == null) {//如果后置节点为空（说明当前节点原本是尾节点）
    last = prev; //则 尾节点为前置节点
} else {
    next.prev = prev;
    x.next = null;//将当前节点的 后置节点置空
}

x.item = null; //将当前元素值置空
size--; //修改数量
modCount++;  //修改modCount
return element; //返回取出的元素值
}
    private void checkElementIndex(int index) {
        if (!isElementIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
    //下标[0,size)
    private boolean isElementIndex(int index) {
        return index >= 0 && index < size;
    }

```

删除链表中的指定节点：

```java
//因为要考虑 null元素，也是分情况遍历
public boolean remove(Object o) {
    if (o == null) {//如果要删除的是null节点(从remove和add 里 可以看出，允许元素为null)
        //遍历每个节点 对比
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
//将节点x，从链表中删除
E unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item;//继续元素值，供返回
    final Node<E> next = x.next;//保存当前节点的后置节点
    final Node<E> prev = x.prev;//前置节点

    if (prev == null) {//前置节点为null，
        first = next;//则首节点为next
    } else {//否则 更新前置节点的后置节点
        prev.next = next;
        x.prev = null;//记得将要删除节点的前置节点置null
    }
    //如果后置节点为null，说明是尾节点
    if (next == null) {
        last = prev;
    } else {//否则更新 后置节点的前置节点
        next.prev = prev;
        x.next = null;//记得删除节点的后置节点为null
    }
    //将删除节点的元素值置null，以便GC
    x.item = null;
    size--;//修改size
    modCount++;//修改modCount
    return element;//返回删除的元素值
}
```


```JAVA
    public E remove() {
        return removeFirst();
    }
    public E removeFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return unlinkFirst(f);
    }
    private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.item;
        final Node<E> next = f.next;
        f.item = null;
        f.next = null; // 断开删除节点的连接，帮助垃圾回收
        first = next;
        if (next == null)
            last = null;
        else
            next.prev = null;
        size--;
        modCount++;
        return element;
    }
    
    public E remove(int index) {
        checkElementIndex(index);
        return unlink(node(index)); //删除下标index上的节点
    }
    E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        if (prev == null) { //如果是头节点，first指向next节点，即删除当前节点
            first = next;
        } else {
            prev.next = next;   //否则，前一个节点的next指向下一个节点
            x.prev = null;
        }

        if (next == null) { //如果是尾节点，last指向前一个节点，即删除当前节点
            last = prev;
        } else {
            next.prev = prev;   //否则，下一个节点的prev指向前一个节点
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }
    
    /**
    * 删除list中最前面的元素o（如果存在），从first节点往后循环查找
    * null的equals方法总是返回false，所以需要分开判断
    */
    public boolean remove(Object o) {
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {    //从first节点往后循环，删除第一个匹配的元素，结束循环
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
    
    public boolean removeFirstOccurrence(Object o) {
        return remove(o);
    }
    
    public E removeLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return unlinkLast(l);
    }
    
    public boolean removeLastOccurrence(Object o) { //删除list中最后面的元素o（如果存在），从last节点往前循环查找即可
        if (o == null) {
            for (Node<E> x = last; x != null; x = x.prev) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = last; x != null; x = x.prev) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }

```



删也一定会修改modCount。 按下标删，也是先根据index找到Node，然后去链表上unlink掉这个Node。 按元素删，会先去遍历链表寻找是否有该Node，考虑到允许null值，所以会遍历两遍，然后再去unlink它。

#### 改set

```java
public E set(int index, E element) {
    checkElementIndex(index); //检查越界[0,size)
    Node<E> x = node(index);//取出对应的Node
    E oldVal = x.item;//保存旧值 供返回
    x.item = element;//用新值覆盖旧值
    return oldVal;//返回旧值
}
```





改也是先根据index找到Node，然后替换值，改不修改modCount

#### 查get

- get(int index) //获取下标index上的元素
- getFirst() //获取第一个元素（first节点的值）
- getLast() //获取最后一个元素（last节点的值）

```java
//根据index查询节点

public E get(int index) {
    checkElementIndex(index);//判断是否越界 [0,size)
    return node(index).item; //调用node()方法 取出 Node节点，
}
//根据节点对象，查询下标

public int indexOf(Object o) {
    int index = 0;
    if (o == null) {//如果目标对象是null
    //遍历链表
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null)
                return index;
            index++;
        }
    } else {////遍历链表
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item))
                return index;
            index++;
        }
    }
    return -1;
}
//从尾至头遍历链表，找到目标元素值为o的节点

    public int lastIndexOf(Object o) {
        int index = size;
        if (o == null) {
            for (Node<E> x = last; x != null; x = x.prev) {
                index--;
                if (x.item == null)
                    return index;
            }
        } else {
            for (Node<E> x = last; x != null; x = x.prev) {
                index--;
                if (o.equals(x.item))
                    return index;
            }
        }
        return -1;
    }



```
```JAVA
public E getFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return f.item;
    }


public E getLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return l.item;
    }

```





查不修改modCount

#### toArray()

把list中的元素复制到新数组中并返回该数组

```java
public Object[] toArray() {
    //new 一个新数组 然后遍历链表，将每个元素存在数组里，返回
    Object[] result = new Object[size];
    int i = 0;
    for (Node<E> x = first; x != null; x = x.next)
        result[i++] = x.item;
    return result;
}

//toArray(T[] a)方法，把list中的元素复制到指定的数组中
  public <T> T[] toArray(T[] a) {
        if (a.length < size)
            a = (T[])java.lang.reflect.Array.newInstance(
                                a.getClass().getComponentType(), size);
        int i = 0;
        Object[] result = a;
        for (Node<E> x = first; x != null; x = x.next)
            result[i++] = x.item;

        if (a.length > size)
            a[size] = null;

        return a;
    }
```


#### peek方法

- peekFirst()，同peek()

- peekLast()，返回最后一个元素，不删除

返回list中的第一个元素，不删除，如果list为空，返回null

```java
    public E peek() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }
```

#### poll方法

- pollFirst()，同poll()

- pollLast()，返回最后一个元素，并删除

返回list中的第一个元素，并删除，**如果list为空，返回null**

```java
    public E poll() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }
```



#### push(E e)方法，添加元素，

```java
    public void push(E e) {
        addFirst(e);
    }
```

#### pop()方法，

删除list中的第一个元素，**如果list为空，抛出NoSuchElementException异常**

```java
    public E pop() {
        return removeFirst();
    }
```

#### clear()方法，删除list中的所有元素

```java
    public void clear() {
        for (Node<E> x = first; x != null; ) {  //循环list，把item、next、prev赋值为null，帮助垃圾回收
            Node<E> next = x.next;
            x.item = null;
            x.next = null;
            x.prev = null;
            x = next;
        }
        first = last = null;
        size = 0;
        modCount++;
    }
```

#### clone()方法，复制一个相同的list

```java
    public Object clone() {
        LinkedList<E> clone = superClone();

        // 初始化list的状态
        clone.first = clone.last = null;
        clone.size = 0;
        clone.modCount = 0;

        // 把原list中的元素复制到新list中
        for (Node<E> x = first; x != null; x = x.next)
            clone.add(x.item);

        return clone;
    }
    
    private LinkedList<E> superClone() {
        try {
            return (LinkedList<E>) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new InternalError(e);
        }
    }
```

#### contains(Object o)方法，判断list中是否包含元素o

```java
    public boolean contains(Object o) {
        return indexOf(o) != -1;
    }
```

#### indexOf(Object o)方法，返回元素o的下标

```JAVA
    public int indexOf(Object o) {
        int index = 0;
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        return -1;
    }
```



#### size方法：返回list大小

```JAVA
    public int size() {
        return size;
    }
```

#### ListIterator迭代器

- listIterator(int index)

```JAVA
    public ListIterator<E> listIterator(int index) {
        checkPositionIndex(index);
        return new ListItr(index);
    }
```

LinkedList中的迭代器，比较简单，使用next、nextIndex、lastReturned变量做个标记。

```JAVA
    private class ListItr implements ListIterator<E> {
        private Node<E> lastReturned;
        private Node<E> next;
        private int nextIndex;
        private int expectedModCount = modCount;

        ListItr(int index) {
            // assert isPositionIndex(index);
            next = (index == size) ? null : node(index);
            nextIndex = index;
        }

        public boolean hasNext() {
            return nextIndex < size;
        }

        public E next() {
            checkForComodification();
            if (!hasNext())
                throw new NoSuchElementException();

            lastReturned = next;
            next = next.next;
            nextIndex++;
            return lastReturned.item;
        }

        public boolean hasPrevious() {
            return nextIndex > 0;
        }

        public E previous() {
            checkForComodification();
            if (!hasPrevious())
                throw new NoSuchElementException();

            lastReturned = next = (next == null) ? last : next.prev;
            nextIndex--;
            return lastReturned.item;
        }

        public int nextIndex() {
            return nextIndex;
        }

        public int previousIndex() {
            return nextIndex - 1;
        }

        public void remove() {
            checkForComodification();
            if (lastReturned == null)
                throw new IllegalStateException();

            Node<E> lastNext = lastReturned.next;
            unlink(lastReturned);
            if (next == lastReturned)
                next = lastNext;
            else
                nextIndex--;
            lastReturned = null;
            expectedModCount++;
        }

        public void set(E e) {
            if (lastReturned == null)
                throw new IllegalStateException();
            checkForComodification();
            lastReturned.item = e;
        }

        public void add(E e) {
            checkForComodification();
            lastReturned = null;
            if (next == null)
                linkLast(e);
            else
                linkBefore(e, next);
            nextIndex++;
            expectedModCount++;
        }

        public void forEachRemaining(Consumer<? super E> action) {
            Objects.requireNonNull(action);
            while (modCount == expectedModCount && nextIndex < size) {
                action.accept(next.item);
                lastReturned = next;
                next = next.next;
                nextIndex++;
            }
            checkForComodification();
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }

```



#### 总结：

LinkedList 是双向列表。

链表批量增加，是靠for循环遍历原数组，依次执行插入节点操作。对比ArrayList是通过System.arraycopy完成批量增加的。增加一定会修改modCount。
通过下标获取某个node 的时候，（add select），会根据index处于前半段还是后半段 进行一个折半，以提升查询效率

删也一定会修改modCount。 按下标删，也是先根据index找到Node，然后去链表上unlink掉这个Node。 按元素删，会先去遍历链表寻找是否有该Node，如果有，去链表上unlink掉这个Node。

改也是先根据index找到Node，然后替换值。改不修改modCount。
查本身就是根据index找到Node。
所以它的CRUD操作里，都涉及到根据index去找到Node的操作。







## ArrayList和LinkedList的区别与联系

##### 各自效率问题：

![img](ArrayList%E2%80%94%E2%80%94LinkedList.assets/20180724111654310.png)

#### ArrayList和LinkedList的区别：

1,ArrayList是实现了基于**动态数组**的数据结构，而LinkedList是**基于链表的**数据结构

2，**对于随机访问的get和set，ArrayList要由于LinkedList，因为LinkedList要移动指针**

3，**对于添加和删除链表add和remove,一般大家都会说LinkedList要比ArrayList快，因为ArrayList要移动数据。但是实际情况并非这样，对于添加和删除，LinkedList和ArrayList并不能说明谁快谁**慢，

#### 数据的更新和查找

**ArrayList的所有数据是在同一个地址上,而LinkedList的每个数据都拥有自己的地址.所以在对数据进行查找的时候，由于LinkedList的每个数据地址不一样，get数据的时候ArrayList的速度会优于LinkedList，而更新数据的时候，虽然都是通过循环循环到指定节点修改数据，但LinkedList的查询速度已经是慢的，而且对于LinkedList而言，更新数据时不像ArrayList只需要找到对应下标更新就好，LinkedList需要修改指针，速率不言而喻**

#### 数据的增加和删除

**对于数据的增加元素，ArrayList是通过移动该元素之后的元素位置，其后元素位置全部+1，所以耗时较长，而LinkedList只需要将该元素前的后续指针指向该元素并将该元素的后续指针指向之后的元素即可。与增加相同，删除元素时ArrayList需要将被删除元素之后的元素位置-1，而LinkedList只需要将之后的元素前置指针指向前一元素，前一元素的指针指向后一元素即可。当然，事实上，若是单一元素的增删，尤其是在List末端增删一个元素，二者效率不相上下。**



ArrayList中的随机访问，添加和删除源码如下：

```java
//获取index位置的元素值
public E get(int index) {
    rangeCheck(index); //首先判断index的范围是否合法
 
    return elementData(index);
}
 
//将index位置的值设为element，并返回原来的值
public E set(int index, E element) {
    rangeCheck(index);
 
    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
 
//将element添加到ArrayList的指定位置
public void add(int index, E element) {
    rangeCheckForAdd(index);
 
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    //将index以及index之后的数据复制到index+1的位置往后，即从index开始向后挪了一位
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index); 
    elementData[index] = element; //然后在index处插入element
    size++;
}
 
//删除ArrayList指定位置的元素
public E remove(int index) {
    rangeCheck(index);
 
    modCount++;
    E oldValue = elementData(index);
 
    int numMoved = size - index - 1;
    if (numMoved > 0)
        //向左挪一位，index位置原来的数据已经被覆盖了
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    //多出来的最后一位删掉
    elementData[--size] = null; // clear to let GC do its work
 
    return oldValue;
}
 
```

   LinkedList中的随机访问、添加和删除部分源码如下：

```java
    LinkedList中的随机访问、添加和删除部分源码如下：
//获得第index个节点的值
public E get(int index) {
	checkElementIndex(index);
	return node(index).item;
}
 
//设置第index元素的值
public E set(int index, E element) {
	checkElementIndex(index);
	Node<E> x = node(index);
	E oldVal = x.item;
	x.item = element;
	return oldVal;
}
 
//在index个节点之前添加新的节点
public void add(int index, E element) {
	checkPositionIndex(index);
 
	if (index == size)
		linkLast(element);
	else
		linkBefore(element, node(index));
}
 
//删除第index个节点
public E remove(int index) {
	checkElementIndex(index);
	return unlink(node(index));
}
 
//定位index处的节点
Node<E> node(int index) {
	// assert isElementIndex(index);
	//index<size/2时，从头开始找
	if (index < (size >> 1)) {
		Node<E> x = first;
		for (int i = 0; i < index; i++)
			x = x.next;
		return x;
	} else { //index>=size/2时，从尾开始找
		Node<E> x = last;
		for (int i = size - 1; i > index; i--)
			x = x.prev;
		return x;
	}
}
```

 从源码可以看出，**ArrayList想要get(int index)元素时，直接返回index位置上的元素，而LinkedList需要通过for循环进行查找，虽然LinkedList已经在查找方法上做了优化，比如index < size / 2，则从左边开始查找，反之从右边开始查找，但是还是比ArrayList要慢**。这点是毋庸置疑的。
      

​	**当数据量较小时，测试程序中，大约小于30的时候，两者效率差不多，没有显著区别**；当数据量较大时，大约在容量的1/10处开始，LinkedList的效率就开始没有ArrayList效率高了，特别到一半以及后半的位置插入时，LinkedList效率明显要低于ArrayList，而且数据量越大，越明显。比如我测试了一种情况，在index=1000的位置(容量的1/10)插入10000条数据和在index=5000的位置以及在index=9000的位置插入10000条数据的运行时间如下：



```java
//在index=1000出插入结果：
array time:4
linked time:240
array insert time:20
linked insert time:18

//在index=5000处插入结果：
array time:4
linked time:229
array insert time:13
linked insert time:90

//在index=9000处插入结果：
array time:4
linked time:237
array insert time:7
linked insert time:92
```




​        从运行结果看，LinkedList的效率是越来越差。
​        所以当插入的数据量很小时，两者区别不太大，当插入的数据量大时，大约在容量的1/10之前，LinkedList会优于ArrayList，在其后就劣与ArrayList，且越靠近后面越差。所以个人觉得，一般首选用ArrayList，由于LinkedList可以实现栈、队列以及双端队列等数据结构，所以当特定需要时候，使用LinkedList，当然咯，数据量小的时候，两者差不多，视具体情况去选择使用；当数据量大的时候，如果只需要在靠前的部分插入或删除数据，那也可以选用LinkedList，反之选择ArrayList反而效率更高。



#### 性能上的有缺点：

1.对ArrayList和LinkedList而言，**在列表末尾增加一个元素所花的开销都是固定的**。对 ArrayList而言，主要是**在内部数组中增加一项，指向所添加的元素，偶尔可能会导致对数组重新进行分配**；而对LinkedList而言，**这个开销是 统一的，分配一个内部Entry对象**。
2.在ArrayList集合中**添加或者删除一个元素时，当前的列表所所有的元素都会被移动**。而LinkedList集合中**添加或者删除一个元素的开销是固定的**。
3.**LinkedList集合不支持 高效的随机随机访问（RandomAccess），因为可能产生二次项的行为**。
4.**ArrayList的空间浪费主要体现在在list列表的结尾预留一定的容量空间，而LinkedList的空间花费则体现在它的每一个元素都需要消耗相当的空间**



#### 数组和链表各自的特性

数组和链表的特性差异，本质是：**连续空间存储和非连续空间存储的差异**，主要有下面两点：

1. ArrayList：底层是Object数组实现的：由于数组的地址是连续的，**数组支持O(1)随机访问**；**数组在初始化时需要指定容量；数组不支持动态扩容，像ArrayList、Vector和Stack使用的时候看似不用考虑容量问题（因为可以一直往里面存放数据）；但是它们的底层实际做了扩容；数组扩容代价比较大，需要开辟一个新数组将数据拷贝进去，数组扩容效率低；适合读数据较多的场合**
2. LinkedList：**底层使用一个Node数据结构，有前后两个指针，双向链表实现的**。**相对数组，链表插入效率较高，只需要更改前后两个指针即可；另外链表不存在扩容问题，因为链表不要求存储空间连续，每次插入数据都只是改变last指针；另外，链表所需要的内存比数组要多，因为他要维护前后两个指针；它适合删除，插入较多的场景。**



## 面试有关

## ArrayList有缩容吗？

`ArrayList`没有缩容。无论是`remove`方法还是`clear`方法，它们都不会改变现有数组`elementData`的长度。但是它们都会把相应位置的元素设置为`null`，以便垃圾收集器回收掉不使用的元素，节省内存。







##### ArrayList是线程安全的么？

当然不是，线程安全版本的数组容器是Vector。Vector的实现很简单，就是把所有的方法统统加上synchronized就完事了。你也可以不使用Vector，用Collections.synchronizedList把一个普通ArrayList包装成一个线程安全版本的数组容器也可以，原理同Vector是一样的，就是给所有的方法套上一层synchronized。

##### **数组用来做队列合适么？**

队列一般是FIFO的，如果用ArrayList做队列，就需要在数组尾部追加数据，数组头部删除数组，反过来也可以。但是无论如何总会有一个操作会涉及到数组的数据搬迁，这个是比较耗费性能的。

这个回答是错误的！

ArrayList固然不适合做队列，**但是数组是非常合适的**。比如ArrayBlockingQueue内部实现就是一个环形队列，它是一个定长队列，内部是用一个定长数组来实现的。另外著名的Disruptor开源Library也是用环形数组来实现的超高性能队列，具体原理不做解释，比较复杂。简单点说就是使用两个偏移量来标记数组的读位置和写位置，如果超过长度就折回到数组开头，前提是它们是定长数组。

##### ArrayList插入删除一定慢么？

取决于你删除的元素离数组末端有多远，ArrayList拿来作为堆栈来用还是挺合适的，push和pop操作完全不涉及数据移动操作。

###### 那他的删除怎么实现的呢？

删除其实跟新增是一样的，不过叫是叫删除，但是在代码里面我们发现，他还是在copy一个数组。

#### ArrayList的遍历和LinkedList遍历性能比较如何？

论遍历ArrayList要比LinkedList快得多，ArrayList遍历最大的优势在于内存的连续性，CPU的内部缓存结构会缓存连续的内存片段，可以大幅降低读取内存的性能开销。