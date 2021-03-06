

#### 1、java中==和equals 和hashCode的区别

1）==若是基本数据类型比较，是比较值，若是引用类型，则比较的是他们在内存中的存
放地址。对象是存放在堆中，栈中存放的对象的引用，所以==是对栈中的值进行比较，若
返回 true 代表变量的内存地址相等；
2）equals 是Object 类中的方法，Object 类的equals 方法用于判断对象的内存地址引用
是不是同一个地址（是不是同一个对象）。若是类中覆盖了 equals 方法，就要根据具体代
码来确定，一般覆盖后都是通过对象的内容是否相等来判断对象是否相等。
3）hashCode()计算出对象实例的哈希码，在对象进行散列时作为 key 存入。之所以有
hashCode 方法，因为在批量的对象比较中，hashCode 比较要比 equals 快。在添加新元
素时，先调用这个元素的 hashCode 方法，一下子能定位到它应该旋转的物理位置，若该
位置没有元素，可直接存储；若该位置有元素，就调用它的 equals 方法与新元素进行比较，
若相同则不存，不相同，就放到该位置的链表末端。
4）equals 与 hashCode 方法关系：
hashCode()是一个本地方法，实现是根据本地机器上关的。equals()相等的对象，
hashCode()也一定相等； hashCode()不等， equals()一定也不等； hashCode()相等， equals()
可能相等，也可能不等。
所以在重写 equals(Object obj)方法，有必要重写 hashCode()方法，确保通过
equals(Object obj)方法判断结果为 true 的两个对象具备相等的 hashCode()返回值。
5）equals 与==的关系：
Integer b1 = 127;在 java 编译时被编译成 Integer b1 = Integer.valueOf(127);对于-128
到127 之间的 Integer 值，用的是原生数据类型int，会在内存里供重用，也就是这之间的
Integer 值进行==比较时，只是进行 int原生数据类型的数值进行比较。而超出-128〜127
的范围，进行==比较时是进行地址及数值比较。

#### 2、int、char、long 各占多少字节数

int\float 占用 4个字节，short\char 占用2个字节，long 占用8 个字节，byte/boolean
占用 1个字节
基本数据类型存放在栈里，包装类栈里存放的是对象的引用，即值的地址，而值存放在堆里。

#### 3、int与integer 的区别

Integer 是 int的包装类， int 则是java的一种基本数据类型， Integer 变量必须实例化才能
使用，当 new 一个 Integer 时，实际是生成一个指向此对象的引用，而 int 是直接存储数
据的值，Integer 默认值是 null，而 int 默认值是 0

#### 4、谈谈对java多态的理解

同一个消息可以根据发送对象的不同而采用多种不同的行为方式，在执行期间判断所引用的
对象的实际类型，根据其实际的类型调用其相应的方法。
作用：消除类型之间的耦合关系。实现多态的必要条件：继承、重写（因为必须调用父类中
存在的方法）、父类引用指向子类对象

#### 5、String、StringBuffer、StringBuilder 区别

都是字符串类，String类中使用字符数组保存字符串，因有 final 修饰符，String对象是不
可变的，每次对 String操作都会生成新的 String对象，这样效率低，且浪费内存空间。但
线程安全。
StringBuilder 和StringBuffer 也是使用字符数组保存字符，但这两种对象都是可变的，即
对字符串进行 append 操作，不会产生新的对象。它们的区别是：StringBuffer 对方法加
了同步锁，是线程安全的，StringBuilder 非线程安全。

#### 6、什么是内部类？内部类的作用

内部类指在类的内部再定义另一个类。
内部类的作用：1）实现多重继承，因为java中类的继承只能单继承，使用内部类可达到多
重继承；2）内部类可以很好的实现隐藏，一般非内部类，不允许有 private 或 protected
权限的，但内部类可以；3）减少了类文件编译后产生的字节码文件大小；
内部类在编译完后也会产生.class 文件，但文件名称是：外部类名称$内部类名称.class。

分为以下几种：
1）**成员内部类**，作为外部类的一个成员存在，与外部类的属性、方法并列，成员内部类持
有外部类的引用，成员内部类不能定义 static 变量和方法。应用场合：每一个外部类都需要
一个内部类实例，内部类离不开外部类存在。
2）**静态内部类**，内部类以static 声明，其他类可通过外部类.内部类来访问。特点：不会持
有外部类的引用，可以访问外部类的静态变量，若要访问成员变量须通过外部类的实例访问。
应用场合：内部类不需要外部类的实例，仅为外部类提供或逻辑上属于外部类，逻辑上可单
独存在。设计的意义：加强了类的封装性（静态内部类是外部类的子行为或子属性，两者保
持着一定关系），提高了代码的可读性（相关联的代码放在一起）。
3）**匿名内部类**，在整个操作中只使用一次，没有名字，使用 new创建，没有具体位置。
4）**局部内部类**，在方法内或是代码块中定义类，

#### 7、抽象类和接口区别

抽象类在类前面须用 **abstract 关键字修饰**，**一般至少包含一个抽象方法**，抽象方法指只有
声明，用关键字 abstract 修饰，没有具体的实现的方法。因抽象类中含有无具体实现的方
法，固不能用抽象类创建对象。当然如果只是用 abstract 修饰类而无具体实现，也是抽象
类。**抽象类也可以有成员变量和普通的成员方法**。**抽象方法必须为public或protected**（若
为private，不能被子类继承，子类无法实现该方法）。**若一个类继承一个抽象类，则必须**
**实现父类中所有的抽象方法，若子类没有实现父类的抽象方法**，则也应该定义为抽象类。



接口用**关键字interface 修饰**，接口也可以含有变量和方法，接口中的**变量会被隐式指定为**
**public static final 变量**。**方法会被隐式的指定为public abstract**，接口中的所有方法均不
能有具体的实现，即接口中的方法都必须为抽象方法。**若一个非抽象类实现某个接口，必须**
**实现该接口中所有的方法。**
区别：
1）抽象类可以提供成员方法实现的细节，而接口只能存在抽象方法；
2）抽象类的成员变量可以是各种类型，而接口中成员变量只能是 publicstaticfinal 类型；
3）**接口中不能含有静态方法及静态代码块，而抽象类可以有静态方法和静态代码块；**

```JAVA
/*
接口中若定义常量需要初始化常量。(接口中并且没有变量，只有静态的常量，需要在最开始的时候赋初始值。)

静态的变量和方法在内存种分配了空间， 而接口只是类的表现形式，是没有分配空间的。
接口中的方法为抽象方法。全局静态方法。没有方法主体。
默认public static final修饰。
private 、 protected 、 static 均不能用来修饰interface

但是需要注意的是Java8引入了一种新特性，为了使接口具有更大的灵活性，将接口静态方法来一个默认实现，当然子类可以重写，也可以不重写。如下：
*/
public interface Service{

    static void method(){

            System.out.println("111");

        }

}
/*

则这种情况是允许的，在实现类中可以进行覆盖method(),也可以不进行覆盖。

接口中的一切抽象都会在子类中被强迫重写。
*/
```



4）一个类只能继承一个抽象类，用extends 来继承，却可以实现多个接口，用 implements
来实现接口。
7.1、抽象类的意义
**抽象类是用来提供子类的通用性，用来创建继承层级里子类的模板，减少代码编写，有利于**
**代码规范化。**
7.2、**抽象类与接口的应用场景**
**抽象类的应用场景**：1）规范了一组公共的方法，与状态无关，可以共享的，无需子类分别
实现；而另一些方法却需要各个子类根据自己特定状态来实现特定功能；
2）定义一组接口，但不强迫每个实现类都必须实现所有的方法，可用抽象类定义一组方法
体可以是空方法体，由子类选择自己感兴趣的方法来覆盖；
7.3、抽象类是否可以没有方法和属性？
可以
7.4、**接口的意义**
1）有利于代码的规范，对于大型项目，对一些接口进行定义，可以给开发人员一个清晰的
指示，防止开发人员随意命名和代码混乱，影响开发效率。
2）有利于代码维护和扩展，当前类不能满足要求时，不需要重新设计类，只需要重新写了
个类实现对应的方法。
3）解耦作用，全局变量的定义，当发生需求变化时，只需改变接口中的值即可。
4）直接看接口，就可以清楚知道具体实现类间的关系，代码交给别人看，别人也能立马明
白。

#### 8、泛型中extends 和super 的区别

<? extends T>**限定参数类型的上界**，参数类型必须是 T 或 T 的子类型，但对于 List<?
extends T>，不能通过add()来加入元素，因为不知道<? extends T>是 T的哪一种子类；
<? super T>**限定参数类型的下界，参数类型必须是 T 或T 的父类型**，不能能过 get()获取
元素，因为不知道哪个超类；

#### 9、父类的静态方法能否被子类重写？静态属性和静态方法是否可以被继承？

**父类的静态方法和属性不能被子类重写，但子类可以继承父类静态方法和属性**，如父类和子
类都有同名同参同返回值的静态方法 show()，声明的实例 Father father = new Son();
(Son extends Father)，会调用 father 对象的静态方法。静态是指在编译时就会分配内存
且一直存在，跟对象实例无关。

#### 10、进程和线程的区别

**进程：具有一定独立功能的程序，是系统进行资源分配和调度运行的基本单位。**
**线程：进程的一个实体，是CPU 调度的苯单位，也是进程中执行运算的最小单位**，即执行
处理机调度的基本单位，如果把进程理解为逻辑上操作系统所完成的任务，线程则表示完成
该任务的许多可能的子任务之一。
关系：**一个进程可有多个线程，至少一个；一个线程只能属于一个进程**。同一进程的所有线
程共享该进程的所有资源。不同进程的线程间要利用消息通信方式实现同步。
区别：**进程有独立的地址空间，而多个线程共享内存；进程具有一个独立功能的程序，线程**
**不能独立运行，必须依存于应用程序中；**

#### 11、final，finally，finalize的区别

final：变量、类、方法的修饰符，被 final 修饰的类不能被继承，变量或方法被 final 修饰
则不能被修改和重写。
finally：异常处理时提供 finally 块来执行清除操作，不管有没有异常抛出，此处代码都会
被执行。如果 try 语句块中包含 return语句，finally 语句块是在return 之后运行；
finalize：Object 类中定义的方法，若子类覆盖了 finalize()方法，在在垃圾收集器将对象
从内存中清除前，会执行该方法，确定对象是否会被回收。

#### 12、序列化Serializable 和Parcelable 的区别

**序列化**：**将一个对象转换成可存储或可传输的状态，序列化后的对象可以在网络上传输，也**
**可以存储到本地，或实现跨进程传输；**
为什么要进行序列化：开发过程中，我们**需要将对象的引用传给其他 activity 或fragment**
**使用时，需要将这些对象放到一个 Intent 或Bundle 中，再进行传递，而 Intent 或Bundle**
**只能识别基本数据类型和被序列化的类型。**
Serializable：**表示将一个对象转换成可存储或可传输的状态**。
Parcelable：**与 Serializable 实现的效果相同，也是将一个对象转换成可传输的状态，但它**
**的实现原理是将一个完整的对象进行分解，分解后的每一部分都是 Intent 所支持的数据类**
**型，这样实现传递对象的功能**。
Parcelable实现序列化的重要方法：序列化功能是由 writeToParcel完成，通过 Parcel 中
的write 方法来完成；反序列化由CREATOR完成，内部标明了如何创建序列化对象及数级，
通过Parcel 的read 方法完成；内容描述功能由 describeContents 方法完成，一般直接返
回0。
**区别**：**Serializable 在序列化时会产生大量临时变量，引起频繁GC。Serializable 本质上使**
**用了反射，序列化过程慢。Parcelable 不能将数据存储在磁盘上，在外界变化时，它不能**
**很好的保证数据的持续性。**
选择原则：**若仅在内存中使用，如 activity\service 间传递对象，优先使用 Parcelable，它**
**性能高。若是持久化操作，优先使用 Serializable**
注意：

- **静态成员变量属于类，不属于对象，固不会参与序列化的过程**；
- 用 **transient 关键字编辑的成员变量不会参与序列化过程**；
- 可以通过重写writeObject()和 readObject()方法来重写系统默认的序列化和反序列化。

#### 13、谈谈对kotlin的理解

特点：1）代码量少且代码末尾没有分号；2）空类型安全（编译期处理了各种 null 情况，
避免执行时异常）；3）函数式的，可使用 lambda 表达式；4）可扩展方法（可扩展任意
类的的属性）；5）互操作性强，可以在一个项目中使用 kotlin和 java两种语言混合开发；

#### 14、string 转换成 integer的方式及原理

1）parseInt(String s)内部调用parseInt(s, 10)默认为10进制 。
2）正常判断null\进制范围，length等。
3）判断第一个字符是否是符号位。
4）循环遍历确定每个字符的十进制值。
5）通过*=和-=进行计算拼接。
6）判断是否为负值返回结果。

#### 1，内存模型以及分区，需要详细到每个区放什么。

 JVM 分为堆区和栈区，还有方法区，**初始化的对象放在堆里面，引用放在栈里面， class 类信息常量池（static 常量和 static 变量）等放在方法区 new**: 

 方法区：**主要是存储类信息，常量池（static 常量和 static 变量）**，编译后的代码（字 节码）等数据 

 堆：**初始化的对象，成员变量 （那种非 static 的变量）**，所有的对象实例和数组都要 在堆上分配 

 栈：**栈的结构是栈帧组成的，调用一个方法就压入一帧**，帧上面存储局部变量表，操作数栈，方法出口等信息，局部变量表存放的是 8 大基础类型加上一个应用类型，所 以还是一个指向地址的指针 

 本地方法栈：**主要为 Native 方法服务** 

 程序计数器：**记录当前线程执行的行号** 

2. 堆里面的分区：**Eden，survival （from+ to），老年代**，各自的特点。
   - **堆里面分为新生代和老生代（java8 取消了永久代，采用了 Metaspace），新生代包 含 Eden+Survivor 区，survivor 区里面分为 from 和 to 区**，内存回收时，如果用的是**复 制算法，从 from 复制到 to，当经过一次或者多次 GC 之后，存活下来的对象会被移动 到老年区**，当 JVM 内存不够用的时候，会触发 **Full GC**，清理 JVM 老年区 
   - 当**新生区满了之后会触发 YGC**,先把存活的对象放到其中一个 Survice 区，然后进行垃圾清理。**因为如果仅仅清理需要删除的对象，这样会导致内存碎 片，因此一般会把 Eden 进行完全的清理，然后整理内存**。那么下次 GC 的时候， 就会使用下一个 Survive，这样循环使用。
   - 如果有特别大的对象，新生代放不下， 就会使用老年代的担保，直接放到老年代里面。因为 JVM 认为，一般大对象的存 活时间一般比较久远。 
3. 对象创建方法，对象的内存分配，对象的访问定位。 new 一个对象 
4. GC 的两种判定方法： 
   - **引用计数法**：指的是如果某个地方引用了这个对象就+1，如果失效了就-1，当为 0 就 会回收但是 JVM 没有用这种方式，因为无法判定相互循环引用（A 引用 B,B 引用 A） 的情况 
   - **引用链法**： 通过一种 GC ROOT 的对象（方法区中静态变量引用的对象等-static 变 量）来判断，如果有一条链能够到达 GC ROOT 就说明，不能到达 GC ROOT 就说明 可以回收 
5. SafePoint 是什么 比如 GC 的时候必须要等到 Java 线程都进入到 safepoint 的时候 VMThread 才能开始 执行 GC，

  - 循环的末尾 (防止大循环的时候一直不进入 safepoint，而其他线程在等待它进入 safepoint) 
  - 方法返回前 
  -  调用方法的 call 之后
  -  抛出异常的位置 

6.  GC 的三种收集方法：**标记清除、标记整理、复制算法**的原理与特点，分别用 在什么地方，如果让你优化收集方法，有什么思路？ 

  **标记清除**：先标记，标记完毕之后再清除，效率不高，会产生碎片
  **复制算法：**分为 8：1 的 Eden 区和 survivor 区，就是上面谈到的 YGC 

**标记整理**：标记完毕之后，让所有存活的对象向一端移动 

  7. **GC 收集器有哪些**？

     **CMS 收集器与 G1 收集器的特点**。 **并行收集器**：串行收集器使用一个单独的线程进行收集，GC 时服务有停顿时间 **串行收集器**：次要回收中使用多线程来执行 CMS 收集器是基于“标记—清除”算法实现的，经过多次标记才会被清除 G1 从整体来看是基于“标记—整理”算法实现的收集器，从局部（两个 Region 之间） 上来看是基于“复制”算法实现的

  8. Minor GC 与 Full GC 分别在什么时候发生？ 新生代内存不够用时候发生 MGC 也叫 YGC，JVM 内存不够的时候发生 FGC 

  9.  几种常用的内存调试工具：jmap、jstack、jconsole、jhat jstack 可以看当前栈的情况，jmap 查看内存，jhat 进行 dump 堆的信息 mat（eclipse 的也要了解一下） 

  10. 类加载的几个过程： **加载、验证、准备、解析、初始化**。然后是使用和卸载了 通过全限定名来加载生成 class 对象到内存中，然后进行验证这个 class 文件，包括文 件格式校验、元数据验证，字节码校验等。准备是对这个对象分配内存。解析是将符 号引用转化为直接引用（指针引用），初始化就是开始执行构造器的代码 11.JVM内存分哪几个区，每个区的作用是什么? 
      java虚拟机主要分为以下一个区: 
      方法区： 

1. 有时候也成为永久代，在该区内很少发生垃圾回收，但是并不代表不发生GC，在这里
进行的GC主要是对方法区里的常量池和对类型的卸载 
2. 方法区主要用来存储已被虚拟机加载的类的信息、常量、静态变量和即时编译器编译后
的代码等数据。 
3. 该区域是被线程共享的。 
4. 方法区里有一个运行时常量池，用于存放静态编译产生的字面量和符号引用。该常量池
具有动态性，也就是说常量并不一定是编译时确定，运行时生成的常量也会存在这个常量
池中。 
虚拟机栈: 
5. 虚拟机栈也就是我们平常所称的栈内存,它为java方法服务，每个方法在执行的时候都会创建一个栈帧，用于存储局部变量表、操作数栈、动态链接和方法出口等信息。 
2. 虚拟机栈是线程私有的，它的生命周期与线程相同。 
3. 局部变量表里存储的是基本数据类型、returnAddress类型（指向一条字节码指令的地
址）和对象引用，这个对象引用有可能是指向对象起始地址的一个指针，也有可能是代表
对象的句柄或者与对象相关联的位置。局部变量所需的内存空间在编译器间确定 
4.操作数栈的作用主要用来存储运算结果以及运算的操作数，它不同于局部变量表通过索
引来访问，而是压栈和出栈的方式 
5.每个栈帧都包含一个指向运行时常量池中该栈帧所属方法的引用，持有这个引用是为了
支持方法调用过程中的动态连接.动态链接就是将常量池中的符号引用在运行期转化为直接
引用。 
**本地方法栈** 
本地方法栈和虚拟机栈类似，只不过本地方法栈为Native方法服务。 
**堆** 
java堆是所有线程所共享的一块内存，在虚拟机启动时创建，几乎所有的对象实例都在这
里创建，因此该区域经常发生垃圾回收操作。 
**程序计数器** 
内存空间小，字节码解释器工作时通过改变这个计数值可以选取下一条需要执行的字节码
指令，分支、循环、跳转、异常处理和线程恢复等功能都需要依赖这个计数器完成。该内
存区域是唯一一个java虚拟机规范没有规定任何OOM情况的区域

#### 2.如和判断一个对象是否存活?(或者GC对象的判定方 法) 

判断一个对象是否存活有两种方法: 

1. **引用计数法** 
所谓引用计数法就是**给每一个对象设置一个引用计数器**，每当有一个地方引用这个对象
时，就将计数器加一，引用失效时，计数器就减一。当一个对象的引用计数器为零时，说
明此对象没有被引用，也就是“死对象”,将会被垃圾回收. 
引用计数法**有一个缺陷就是无法解决循环引用问题**，也就是说当对象A引用对象B，对象
B又引用者对象A，那么此时A,B对象的引用计数器都不为零，也就造成无法完成垃圾回
收，所以主流的虚拟机都没有采用这种算法。 
**2.可达性算法(引用链法)** 
该算法的思想是：**从一个被称为GC Roots的对象开始向下搜索，如果一个对象到GC** 
**Roots没有任何引用链相连时，则说明此对象不可用**。 
在java中可以作为GC Roots的对象有以下几种: 
 虚拟机栈中引用的对象 
 方法区类静态属性引用的对象 
 方法区常量池引用的对象 
 本地方法栈JNI引用的对象 
虽然这些算法**可以判定一个对象是否能被回收，但是当满足上述条件时，一个对象比不一定会被回收。当一个对象不可达GC Root时，这个对象并不会立马被回收，而是出于一个死缓的阶段，若要被真正的回收需要经历两次标记** 
- **如果对象在可达性分析中没有与GC Root的引用链**，那么此时就会被第一次标记并且进行一次筛选，筛选的条件是是否有必要执行finalize()方法。当对象没有覆盖finalize()方法或者已被虚拟机调用过，那么就认为是没必要的。 
- **如果该对象有必要执行finalize()方法**，那么这个对象将会放在一个称为F-Queue的对队列中，虚拟机会触发一个Finalize()线程去执行，此线程是低优先级的，并且虚拟机不会承诺一直等待它运行完，这是因为如果finalize()执行缓慢或者发生了死锁，那么就会造成FQueue队列一直等待，造成了内存回收系统的崩溃。GC对处于F-Queue中的对象进行第二次被标记，这时，该对象将被移除”即将回收”集合，等待回收

#### 3.简述java垃圾回收机制? 

在java中，程序员是不需要显示的去释放一个对象的内存的，而是由虚拟机自行执行。在
**JVM中，有一个垃圾回收线程，它是低优先级的，在正常情况下是不会执行的，只有在虚**
**拟机空闲或者当前堆内存不足时，才会触发执行，扫面那些没有被任何引用的对象，并将**
**它们添加到要回收的集合中，进行回收**

#### 4.java中垃圾收集的方法有哪些? 

1. **标记-清除:** 
  这是垃圾收集算法中最基础的，根据名字就可以知道，**它的思想就是标记哪些要被**
  **回收的对象，然后统一回收**。

  这种方法很简单，但是会有两个主要问题

  - **效率不高，标记和清除的效率都很低**；
  - 会产生大量不连续的内存碎片，导致以后程序在分配较大的对象时，由于没有充足的连续内存而提前触发一次GC动作。 

2. **复制算法**: 
  为了解决效率问题，**复制算法将可用内存按容量划分为相等的两部分，然后每次只**
  **使用其中的一块，当一块内存用完时，就将还存活的对象复制到第二块内存上，然**
  **后一次性清楚完第一块内存，再将第二块上的对象复制到第一块**。

  **但是这种方式，内存的代价太高，每次基本上都要浪费一般的内存。 **
  于是将该算法进行了改进，内存区域不再是按照1：1去划分，**而是将内存划分为**
  **8:1:1三部分，较大那份内存交Eden区，其余是两块较小的内存区叫Survior区**。
  每次都**会优先使用Eden区，若Eden区满，就将对象复制到第二块内存区上，然**
  **后清除Eden区，如果此时存活的对象太多**，以至于Survivor不够时，会将这些对
  象通过分配担保机制复制到老年代中。(java堆又分为新生代和老年代) 

3. **标记-整理** 
  该算法主要是**为了解决标记-清除，产生大量内存碎片的问题**；当对象存活率较高
  时，也解决了复制算法的效率问题。它的不同之处就是**在清除对象的时候现将可回**
  **收对象移动到一端，然后清除掉端边界以外的对象，这样就不会产生内存碎片了**。 

4. **分代收集**  
  **现在的虚拟机垃圾收集大多采用这种方式，它根据对象的生存周期，将堆分为新生**
  **代和老年代。在新生代中，由于对象生存期短，每次回收都会有大量对象死去，那**
  **么这时就采用复制算法。老年代里的对象存活率较高，没有额外的空间进行分配担**
  **保，所以可以使用标记-整理 或者 标记-清**

#### 5.java内存模型 

1. **java内存模型(JMM)是线程间通信的控制机制.**JMM**定义了主内存和线程之间抽象关系**。
   **线程之间的共享变量存储在主内存（main memory）中，每个线程都有一个私有的本地**
   **内存（local memory），本地内存中存储了该线程以读/写共享变量的副本**。本地内存是
   JMM的一个抽象概念，并不真实存在。它涵盖了缓存，写缓冲区，寄存器以及其他的硬
   件和编译器优化。Java内存模型的抽象示意图如下：


   从上图来看，线程A与线程B之间如要通信的话，必须要经历下面2个步骤： 

   1. 首先，线程A把本地内存A中更新过的共享变量刷新到主内存中去。 
   2. 然后，线程B到主内存中去读取线程A之前已更新过的共享变量

#### 6.java类加载过程? 

java类加载需要经历一下7个过程： 
**加载** 
加载时类加载的第一个过程，在这个阶段，将完成一下三件事情： 

1. 通过一个类的全限定名获取该类的二进制流。 
2. 将该二进制流中的静态存储结构转化为方法去运行时数据结构。  
3. 在内存中生成该类的Class对象，作为该类的数据访问入口。 

**验证** 
验证的目的是为了确保Class文件的字节流中的信息不回危害到虚拟机.在该阶段主要完成
以下四钟验证: 

1. 文件格式验证：验证字节流是否符合Class文件的规范，如主次版本号是否在当前虚拟
机范围内，常量池中的常量是否有不被支持的类型. 
2. 元数据验证:对字节码描述的信息进行语义分析，如这个类是否有父类，是否集成了不
被继承的类等。 
3. 字节码验证：是整个验证过程中最复杂的一个阶段，通过验证数据流和控制流的分析，
确定程序语义是否正确，主要针对方法体的验证。如：方法中的类型转换是否正确，跳转指令是否正确等
4. 符号引用验证：这个动作在后面的解析过程中发生，主要是为了确保解析动作能正确执
    行。 
    **准备** 
    准备阶段是为类的静态变量分配内存并将其初始化为默认值，这些内存都将在方法区中进
    行分配。准备阶段不分配类中的实例变量的内存，实例变量将会在对象实例化时随着对象
    一起分配在Java堆中。 

```JAVA
public static int value=123; // 在准备阶段 value 初始值为 0 。在初始化阶段才会变
   为 123 

```



**解析** 
该阶段主要完成符号引用到直接引用的转换动作。解析动作并不一定在初始化动作完成之
前，也有可能在初始化之后。 
**初始化** 
初始化时类加载的最后一步，前面的类加载过程，除了在加载阶段用户应用程序可以通过
自定义类加载器参与之外，其余动作完全由虚拟机主导和控制。到了初始化阶段，才真正
开始执行类中定义的Java程序代码

#### 7.简述java类加载机制? 

**虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校验，解析和初始化，最**
**终形成可以被虚拟机直接使用的java类型。**

#### 8.类加载器双亲委派模型机制？ 

**当一个类收到了类加载请求时，不会自己先去加载这个类，而是将其委派给父类，由父类**
**去加载，如果此时父类不能加载，反馈给子类，由子类去完成类的加载**

#### 9.什么是类加载器，类加载器有哪些? 

**实现通过类的权限定名获取该类的二进制字节流的代码块叫做类加载器。** 
主要有一下四种类加载器: 

1. **启动类加载器**(Bootstrap ClassLoader)**用来加载java核心类库**，无法被java程序直接
引用。
2. **扩展类加载器**(extensions class loader):**它用来加载 Java 的扩展库**。Java 虚拟机的
实现会提供一个扩展库目录。该类加载器在此目录里面查找并加载 Java 类。 
3. **系统类加载器**（system class loader）：**它根据 Java 应用的类路径（CLASSPATH）**
**来加载 Java 类**。一般来说，Java 应用的类都是由它来完成加载的。可以通过 
ClassLoader.getSystemClassLoader()来获取它。 
4. **用户自定义类加载器**，**通过继承 java.lang.ClassLoader类的方式实现**。 

#### 10.简述java内存分配与回收策率以及Minor GC和 Major GC 

1. 对象优先在堆的Eden区分配。 
2. 大对象直接进入老年代. 

3. 长期存活的对象将直接进入老年代. 
当Eden区没有足够的空间进行分配时，虚拟机会执行一次Minor GC.Minor Gc通
常发生在新生代的Eden区，在这个区的对象生存期短，往往发生Gc的频率较高，
回收速度比较快;Full Gc/Major GC 发生在老年代，一般情况下，触发老年代GC
的时候不会触发Minor GC,但是通过配置，可以在Full GC之前进行一次Minor 
GC这样可以加快老年代的回收速度