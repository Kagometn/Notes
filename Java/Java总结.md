### Java总结



#### 2，Hash的作用

###### 	见Java**Hash函数**

#### 3，Java的封装，继承和多态

##### 	封装：

> 1.定义：隐藏对象的属性和实现细节，仅对外公开接口,控制在程序中属性的读和修改的访问级别。

2.目的：增强安全性和简化编程，使用者不必了解具体的实现细节，而只是要通过外部接口，一特定的访问权限来使用类的成员。

3.基本要求:把所有的属性私有化，对每个属性提供getter和setter方法，如果有一个带参的构造函数的话，那一定要写一个不带参的构造函数。在开发的时候经常要对已经编写的类进行测试，所以在有的时候还有重写toString方法，但这不是必须的。

##### 继承：

1.目的：实现代码的复用。

2.介绍：当两个类具有相同的特征（属性）和行为（方法）时，可以将相同的部分抽取出来放到一个类中作为父类，其它两个类继承这个父类。继承后子类自动拥有了父类的属性和方法，但特别注意的是，父类的私有属性和构造方法并不能被继承。另外子类可以写自己特有的属性和方法，目的是实现功能的扩展，子类也可以复写父类的方法即方法的重写。子类不能继承父类中访问权限为private的成员变量和方法。子类可以重写父类的方法，及命名与父类同名的成员变量。有时候我们需要这样的需求：我们需要将某些事物尽可能地对这个世界隐藏，但是仍然允许子类的成员来访问它们。这个时候就需要使用到protected。

##### 多态：

多态：

1.概念：相同的事物，调用其相同的方法，参数也相同时，但表现的行为却不同。

2. Java实现多态有三个必要条件：继承、重写、向上转型。

         继承：在多态中必须存在有继承关系的子类和父类。
            
         重写：子类对父类中某些方法进行重新定义，在调用这些方法时就会调用子类的方法。
            
         向上转型：在多态中需要将子类的引用赋给父类对象，只有这样该引用才能够具备技能调用父类的方法和子类的方法。

      只有满足了上述三个条件，我们才能够在同一个继承结构中使用统一的逻辑实现代码处理不同的对象，从而达到执行不同的行为。

3.多态的实现方式：

###### （1）基于继承实现的多态

      基于继承的实现机制主要表现在父类和继承该父类的一个或多个子类对某些方法的重写，多个子类对同一方法的重写可以表现出不同的行为。

###### （2）基于接口实现的多态

      继承是通过重写父类的同一方法的几个不同子类来体现的，那么就可就是通过实现接口并覆盖接口中同一方法的几不同的类体现的。
    
      在接口的多态中，指向接口的引用必须是指定这实现了该接口的一个类的实例程序，在运行时，根据对象引用的实际类型来执行对应的方法。
    
      继承都是单继承，只能为一组相关的类提供一致的服务接口。但是接口可以是多继承多实现，它能够利用一组相关或者不相关的接口进行组合与扩充，能够对外提供一致的服务接口。所以它相对于继承来说有更好的灵活性。
#### 4，LinkedList和ArrayList的异同

 ![img](https://img-blog.csdn.net/20160413184734236?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

###### 基于性能分析： 

- ArrayList基于动态数组的数据结构，而LinkedList是基于链表的数据结构；

- 对于随机访问的get和set,ArrayList要由于LinkedList，因为LinkedList要由于指针
- 对于添加和删除操作的add和remove，一般大家都会说LinkedList要比ArrayList快，因为ArrayList要移动数据。但实际情况就是比较复杂，对于添加或者删除，LinkedList和ArrayList并不能说明谁快谁慢，

###### ArrayList中的随机访问，添加和删除部分源码如下：

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

###### LinkedList中的随机访问，添加和删除部分源码如下：

```java
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

###### 基于源码分析：

  从源码可以看出，ArrayList想要get(int index)元素时，直接返回index位置上的元素，而LinkedList需要通过for循环进行查找，虽然LinkedList**已经在查找方法上做了优化**，比如index < size / 2，则从左边开始查找，反之从右边开始查找，但是还是比ArrayList要慢。这点是毋庸置疑的。
        **ArrayList想要在指定位置插入或删除元素时，主要耗时的是System.arraycopy动作，会移动index后面所有的元素；LinkedList主耗时的是要先通过for循环找到index，然后直接插入或删除。这就导致了两者并非一定谁快谁慢**，下面通过一个测试程序来测试一下两者插入的速度：

```java
import java.util.ArrayList;  
import java.util.Collections;  
import java.util.LinkedList;  
import java.util.List;  
/*
 * @description 测试ArrayList和LinkedList插入的效率
 * @eson_15     
 */
public class ArrayOrLinked {  
    static List<Integer> array=new ArrayList<Integer>();  
    static List<Integer> linked=new LinkedList<Integer>();  
  
    public static void main(String[] args) {  
  
    	//首先分别给两者插入10000条数据
        for(int i=0;i<10000;i++){  
            array.add(i);  
            linked.add(i);  
        }  
        //获得两者随机访问的时间
        System.out.println("array time:"+getTime(array));  
        System.out.println("linked time:"+getTime(linked));  
        //获得两者插入数据的时间
        System.out.println("array insert time:"+insertTime(array));  
        System.out.println("linked insert time:"+insertTime(linked));  
  
    }  
    public static long getTime(List<Integer> list){  
        long time=System.currentTimeMillis();  
        for(int i = 0; i < 10000; i++){  
            int index = Collections.binarySearch(list, list.get(i));  
            if(index != i){  
                System.out.println("ERROR!");  
            }  
        }  
        return System.currentTimeMillis()-time;  
    }  
    
    //插入数据
    public static long insertTime(List<Integer> list){ 
    	/*
    	 * 插入的数据量和插入的位置是决定两者性能的主要方面，
    	 * 我们可以通过修改这两个数据，来测试两者的性能
    	 */
    	long num = 10000; //表示要插入的数据量
    	int index = 1000; //表示从哪个位置插入
        long time=System.currentTimeMillis();  
        for(int i = 1; i < num; i++){  
            list.add(index, i);     
        }  
        return System.currentTimeMillis()-time;  
          
    }  
  
}  
```

**主要有两个因素决定他们的效率，插入的数据量和插入的位置。我们可以在程序里改变这两个因素来测试它们的效率。**
        **当数据量较小时，测试程序中，大约小于30的时候，两者效率差不多，没有显著区别；当数据量较大时，大约在容量的1/10处开始，LinkedList的效率就开始没有ArrayList效率高了**，特别到一半以及后半的位置插入时，LinkedList效率明显要低于ArrayList，而且数据量越大，越明显。比如我测试了一种情况，在index=1000的位置(容量的1/10)插入10000条数据和在index=5000的位置以及在index=9000的位置插入10000条数据的运行时间如下：

```java
在index=1000出插入结果：
array time:4
linked time:240
array insert time:20
linked insert time:18
 
在index=5000处插入结果：
array time:4
linked time:229
array insert time:13
linked insert time:90
 
在index=9000处插入结果：
array time:4
linked time:237
array insert time:7
linked insert time:92
```

 从运行结果看，LinkedList的效率是越来越差。

###### 总结：

​        **所以当插入的数据量很小时，两者区别不太大，当插入的数据量大时，大约在容量的1/10之前，LinkedList会优于ArrayList，在其后就劣与ArrayList，且越靠近后面越差**。所以个人觉得，一般首选用ArrayList，由于LinkedList可以实现栈、队列以及双端队列等数据结构，所以当特定需要时候，使用LinkedList，当然咯，数据量小的时候，两者差不多，视具体情况去选择使用；当数据量大的时候，**如果只需要在靠前的部分插入或删除数据，那也可以选用LinkedList，反之选择ArrayList反而效率更高**。

##### 5，HashMap源码分析：

###### get()：

###### put():

##### 6，Hashmap，HashSet以及HashTable之间的区别：

##### 7，final修饰的类的使用方式：

Java关键字final有“这是无法改变的”或者“终态的”含义，它可以修饰非抽象类、非抽象类成员方法和变量。

**final类不能被继承，没有子类，final类中的方法默认是final的。** 

**final方法不能被子类的方法覆盖，但可以被继承。** 

**final成员变量表示常量，只能被赋值一次，赋值后值不再改变。** 

**final不能用于修饰构造方法。** 

注意：父类的private成员方法是不能被子类方法覆盖的，因此private类型的方法默认是final类型的。

###### 1、final类

**final类不能被继承**，因此final类的成员方法没有机会被覆盖，默认都是final的。在设计类时候，如果这个类不需要有子类，类的实现细节不允许改变，并且确信这个类不会载被扩展，那么就设计为final类。

###### 2、final方法 

如果**一个类不允许其子类覆盖某个方法**，则可以把这个方法声明为final方法。使用final方法的原因有二：
①把方法锁定，防止任何继承类修改它的意义和实现。 

②高效，编译器在遇到调用final方法时候会转入内嵌机制，大大提高执行效率。 

###### 3、final变量（常量）

用**final修饰的成员变量表示常量，值一旦给定就无法改变**；final修饰的变量有三种：静态变量、实例变量和局部变量，分别表示三种类型的常量。从下面的例子中可以看出，一旦给final变量初值后，值就不能再改变了。

另外，final变量定义的时候，可以先声明，而不给初值，这中变量也称为final空白，无论什么情况，编译器都确保空白final在使用之前必须被初始化。但是，final空白在final关键字final的使用上提供了更大的灵活性，为此，一个类中的final数据成员就可以实现依对象而有所不同，却有保持其恒定不变的特征。

###### 4、final参数

当函数参数为final类型时，你可以读取使用该参数，但是无法改变该参数的值。

