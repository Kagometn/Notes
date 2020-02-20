# Java基础

# 一，操作系统相关

#### 1，什么是操作系统

- 一组能有效地组织和管理计算机硬件和软件资源，合理的对各类作业进行调度以及方便用户使用的程序的集合。 

-  管理着计算机的内存和进程，以及所有的软件和硬件。它还允许你在不知道如何说出计算机语言的情况下与计算机进行交流。没有操作系统，计算机就毫无用处。 

-  **OS****它是一个计算机系统资源的管理者，并实现了对计算机资源的抽象**，连接了用户与计算机硬件系统，**作为用户与计算机硬件系统之间的接口**（软件接口）。 

  - OS的主要功能就是：

    （1）、处理机管理：分配和控制处理机

    （2）、存储器管理：内存分配与回收

    （3）、I/O设备管理：I/O设备分配与操作

    （4）、文件管理：文件的存取、共享和保护

#### 2，什么是线程，什么是进程

**概述：**两者都是一个时间段的描述，是CPU工作时间端的描述

**进程：进程是资源分配的最小单位**：

​		 是具有一定独立功能的程序关于某个数据集合上的一次运行活动,进程是系统进行资源分配和调度的一个独立单位. 

**线程：CPU调度的最小单位**

​		 线程是进程的一个实体,是CPU调度和分派的基本单位,它是比进程更小的能独立运行的基本单位.线程自己基本上不拥有系统资源,只拥有一点在运行中必不可少的资源(如程序计数器,一组寄存器和栈),但是它可与同属一个进程的其他的线程共享进程所拥有的全部资源. 

 **进程=火车，线程=车厢** 

**注意：**

-  一个线程可以创建和撤销另一个线程;同一个进程中的多个线程之间可以并发执行. 
-  相对进程而言，线程是一个更加接近于执行体的概念，它可以与同进程中的其他线程共享数据，但拥有自己的栈空间，拥有独立的执行序列。 
-  在串行程序基础上引入线程和进程是为了提高程序的并发度，从而提高程序运行效率和响应时间。 

**区别：**

 进程有独立的地址空间，一个进程崩溃后，在保护模式下不会对其它进程产生影响，而线程只是一个进程中的不同执行路径。线程有自己的堆栈和局部变量，但线程之间没有单独的地址空间，一个线程死掉就等于整个进程死掉，所以多进程的程序要比多线程的程序健壮，但在进程切换时，耗费资源较大，效率要差一些。**但对于一些要求同时进行并且又要共享某些变量的并发操作，只能用线程，不能用进程。** 

-  **简而言之,一个程序至少有一个进程,一个进程至少有一个线程.**
-  线程的划分尺度小于进程，使得多线程程序的并发性高。
-  另外，进程在执行过程中拥有独立的内存单元，而多个线程共享内存，从而极大地提高了程序的运行效率。
-  线程在执行过程中与进程还是有区别的。每个独立的线程有一个程序运行的入口、顺序执行序列和程序的出口。**但是线程不能够独立执行，**必须依存在应用程序中，由应用程序提供多个线程执行控制。
- 从逻辑角度来看，多线程的意义在于一个应用程序中，有多个执行部分可以同时执行。但操作系统并没有将多个线程看做多个独立的应用，来实现进程的调度和管理以及资源分配。**这就是进程和线程的重要区别。**

**优缺点：**

-  线程和进程在使用上各有优缺点：线程执行开销小，但不利于资源的管理和保护；而进程正相反。同时，线程适合于在SMP机器上运行，而进程则可以跨机器迁移。 

**多线程与多进程：**

进程就是一个程序在运行的时候被CPU抽象出来的，一个程序运行后被抽象为一个进程，但是线程是从一个进程中分割出来的，由于CPU处理进程的时候采用的是时间片轮转的方式，所以要把一个大的进程给分割为多个线程 ，例如：网际快车中文件分成100部分 10个线程 文件就被分成了10份来同时下载 1-10 占一个线程 11-20占一个线程,依次类推,线程越多,文件就被分的越多,同时下载 当然速度也就越快 。

- **进程是程序在计算机上的一次执行活动。**当你运行一个程序，你就启动了一个进程。显然，程序只是一组指令的有序集合，它本身没有任何运行的含义，只是一个静态实体。而进程则不同，它是程序在某个数据集上的执行，是一个动态实体。它因创建而产生，因调度而运行，因等待资源或事件而被处于等待状态，因完成任务而被撤消，反映了一个程序在一定的数据集上运行的全部动态过程。
- **进程是操作系统分配资源的单位。**在Windows下，进程又被细化为线程，也就是一个进程下有多个能独立运行的更小的单位。
- **线程(Thread)是进程的一个实体，是CPU调度和分派的基本单位。**线程不能够独立执行，必须依存在应用程序中，由应用程序提供多个线程执行控制。

**线程和进程的关系是：**线程是属于进程的，线程运行在进程空间内，同一进程所产生的线程共享同一内存空间，当进程退出时该进程所产生的线程都会被强制退出并清除。线程可与属于同一进程的其它线程共享进程所拥有的全部资源，但是其本身基本上不拥有系统资源，只拥有一点在运行中必不可少的信息(如程序计数器、一组寄存器和栈)。

在同一个时间里，同一个计算机系统中如果允许两个或两个以上的进程处于运行状态，这便是多任务。现代的操作系统几乎都是多任务操作系统，能够同时管理多个进程的运行。 多任务带来的好处是明显的，比如你可以边听mp3边上网，与此同时甚至可以将下载的文档打印出来，而这些任务之间丝毫不会相互干扰。那么这里就涉及到并行的问题，俗话说，一心不能二用，这对计算机也一样，原则上一个CPU只能分配给一个进程，以便运行这个进程。我们通常使用的计算机中只有一个CPU，也就是说只有一颗心，要让它一心多用，同时运行多个进程，就必须使用并发技术。实现并发技术相当复杂，**最容易理解的是“时间片轮转进程调度算法”，**它的思想简单介绍如下：在操作系统的管理下，所有正在运行的进程轮流使用CPU，每个进程允许占用CPU的时间非常短(比如10毫秒)，这样用户根本感觉不出来CPU是在轮流为多个进程服务，就好象所有的进程都在不间断地运行一样。但实际上在任何一个时间内有且仅有一个进程占有CPU。

如果一台计算机有多个CPU，情况就不同了，如果进程数小于CPU数，则不同的进程可以分配给不同的CPU来运行，这样，多个进程就是真正同时运行的，这便是并行。但如果进程数大于CPU数，则仍然需要使用并发技术。

**在Windows中，进行CPU分配是以线程为单位的，**一个进程可能由多个线程组成，这时情况更加复杂，但简单地说，有如下关系：

总线程数<= CPU数量：并行运行

总线程数> CPU数量：并发运行

并行运行的效率显然高于并发运行，所以在多CPU的计算机中，多任务的效率比较高。但是，如果在多CPU计算机中只运行一个进程(线程)，就不能发挥多CPU的优势。

 多任务操作系统(如Windows)的基本原理是:操作系统将CPU的时间片分配给多个线程,每个线程在操作系统指定的时间片内完成(注意,这里的多个线程是分属于不同进程的).操作系统不断的从一个线程的执行切换到另一个线程的执行,如此往复,宏观上看来,就好像是多个线程在一起执行.由于这多个线程分属于不同的进程,因此在我们看来,就好像是多个进程在同时执行,这样就实现了多任务.

**举例：**

线程在进程下行进（单纯的车厢无法运行）

一个进程可以包含多个线程（一辆火车可以有多个车厢）

不同进程间数据很难共享（一辆火车上的乘客很难换到另外一辆火车，比如站点换乘）

同一进程下不同线程间数据很易共享（A车厢换到B车厢很容易）

进程要比线程消耗更多的计算机资源（采用多列火车相比多个车厢更耗资源）

进程间不会相互影响，一个线程挂掉将导致整个进程挂掉（一列火车不会影响到另外一列火车，但是如果一列火车上中间的一节车厢着火了，将影响到所有车厢）

进程可以拓展到多机，进程最多适合多核（不同火车可以开在多个轨道上，同一火车的车厢不能在行进的不同的轨道上）

进程使用的内存地址可以上锁，即一个线程使用某些共享内存时，其他线程必须等它结束，才能使用这一块内存。（比如火车上的洗手间）－"互斥锁"

进程使用的内存地址可以限定使用量（比如火车上的餐厅，最多只允许多少人进入，如果满了需要在门口等，等有人出来了才能进去）－“信号量”

# 二，Java基础

## 1.1	面向过程&面向对象

#### 1，什么是面向过程(OPP) & 什么是面向对象(OOP) & 区别？

 面向过程就是**分析出解决问题所需要的步骤**，然后用函数把这些步骤一步一步实现，使用的时候一个一个依次调用就可以了。**面向对象是把构成问题事务分解成各个对象**，建立对象的目的不是为了完成一个步骤，而是为了描叙某个事物在整个解决问题的步骤中的行为。 

**面向对象的特点：万物皆为对象**

 ![这里写图片描述](https://img-blog.csdn.net/20180110192411924?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamVycnkxMTExMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

 **属性**用来描述具体某个对象的特征。比如小志身高180M，体重70KG，这里身高、体重都是属性。
面向对象的思想就是把一切都看成对象，而对象一般都由属性+方法组成！

属性属于对象静态的一面，用来形容对象的一些特性，方法属于对象动态的一面，咱们举一个例子，小明会跑，会说话，跑、说话这些行为就是对象的方法！所以为动态的一面， 我们把属性和方法称为这个对象的成员！

类：具有同种属性的对象称为类，是个抽象的概念。比如“人”就是一类，期中有一些人名，比如小明、小红、小玲等等这些都是对象，类就相当于一个模具，他定义了它所包含的全体对象的公共特征和功能，对象就是类的一个实例化，小明就是人的一个实例化！我们在做程序的时候，经常要将一个变量实例化，就是这个原理！我们一般在做程序的时候一般都不用类名的，比如我们在叫小明的时候，不会喊“人，你干嘛呢！”而是说的是“小明，你在干嘛呢！”

**将面向对象类比盖浇饭，面向过程就是蛋炒饭**

盖浇饭用软件工程的专业术语就是”**可维护性**“比较好，”饭” 和”菜”的耦合度比较低。蛋炒饭将”蛋”“饭”搅和在一起，想换”蛋”“饭”中任何一种都很困难，耦合度很高，以至于”可维护性”比较差。软件工程追求的目标之一就是可维护性，可维护性主要表现在3个方面：**可理解性、可测试性和可修改性**。面向对象的好处之一就是显著的改善了软件系统的可维护性。

OPP:

> 优点：性能比面向对象高，因为类调用时需要实例化，开销比较大，比较消耗资源;比如单片机、嵌入式开发、 Linux/Unix等一般采用面向过程开发，性能是最重要的因素。
> 缺点：没有面向对象易维护、易复用、易扩展

OOP:



> 优点：易维护、易复用、易扩展，由于面向对象有封装、继承、多态性的特性，可以设计出低耦合的系统，使系统 更加灵活、更加易于维护
>缺点：性能比面向过程低 



#### 2，给我说说Java面向对象的特征以及讲讲你代码中凸显这些特征的经验

#### 3，什么是重载 & 什么是重写 & 区别

**一，重写（Override）**

重写是子类对父类的允许访问的方法的实现过程进行重新编写, 返回值和形参都不能改变。**即外壳不变，核心重写！**

**重写规则：**

- 参数列表必须完全与被重写的方法相同，否则不能称其为重写而是重载。
- 返回的类型必须一直与被重写的方法的返回类型相同，否则不能称其为重写而是重载。
- 访问修饰符的限制一定要大于被重写方法的访问修饰符（public>protected>default>private）
- 重写方法一定不能抛出新的检查异常或者比被重写方法申明更加宽泛的检查型异常。例如
- 父类的一个方法申明了一个检查异常IOException，在重写这个方法是就不能抛出Exception,只能抛出IOException的子类异常，可以抛出非检查异常。

```java
public class Father{

   public void speak(){

       System.out.println(Father);

    }

}

public class Son extends Father{

    public void speak(){

        System.out.println("son");

   }

}
```

**二、重载（Overload）**

重载(overloading) 是在一个类里面，方法名字相同，而参数不同。返回类型可以相同也可以不同。每个重载的方法（或者构造函数）都必须有一个独一无二的参数类型列表。

**重载规则：**

- 必须具有不同的参数列表；
- 可以有不责骂的返回类型，只要参数列表不同就可以了；
- 可以有不同的访问修饰符；
- 可以抛出不同的异常；

```java
public class Base
    {
        void test(int i)
        {
            System.out.print(i);
        }
        void test(byte b)
        {
            System.out.print(b);
        }
    }
    public class TestOverriding extends Base
    {
        void test(int i)
        {
            i++;
            System.out.println(i);
        }
          public static void main(String[]agrs)
        {
            Base b=new TestOverriding();
            b.test(0)
            b.test((byte)0)
        }
    }


    这时的输出结果是1     0，这是运行时动态绑定的结果。
```

**三、重写与重载的区别**



![img](https://user-gold-cdn.xitu.io/2019/6/1/16b10fe9671627da?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)





**区别：**

重写多态性起作用，对调用被重载过的方法可以大大减少代码的输入量，同一个方法名只要往里面传递不同的参数就可以拥有不同的功能或返回值。

用好重写和重载可以设计一个结构清晰而简洁的类，可以说重写和重载在编写代码过程中的作用非同一般.

**四、总结**

在 Java 中重载是由**静态类型**确定的，在类加载的时候即可确定，属于**静态分派**；而重写是由**动态类型**确定，是**在运行时确定**的，属于**动态分派**，**动态分派是由虚方法表实现的**，虚方法表中存在着各个方法的实际入口地址，如若父类中某个方法子类没有重写，则父类与子类的方法表中的方法地址相同，如若重写了，则子类方法表的地址指向重写后的地址；

一般**重写针对于子类继承父类**，**重写父类的方法，通过动态绑定实现的；**而重载时同一个方法名，但是参数类型或者个数不同，重载可以理解为是一个类中的多态。

若子类中的方法与父类的某一方法具有相同的方法名、返回类型和参数表，则新方法将覆盖原有的方法，如需使用父类中原有的方法，可以使用 super 关键字，该关键字引用了当前类的父类。子**类函数的访问修饰权限不能少于父类的。**



#### 4，谈谈你对this和super的认识

**this**

this 是自身的一个对象，代表对象本身，可以理解为：**指向对象本身的一个指针**。

**this 的用法在 Java 中大体可以分为3种：**

- **普通的直接引用**

这种就不用讲了，this 相当于是指向当前对象本身。

- **形参与成员名字重名，用 this 来区分：**

  ```java
  class Person {
      private int age = 10;
      public Person(){
      System.out.println("初始化年龄："+age);
  }
      public int GetAge(int age){
          this.age = age;
          return this.age;
      }
  }
   
  public class test1 {
      public static void main(String[] args) {
          Person Harry = new Person();
          System.out.println("Harry's age is "+Harry.GetAge(12));
      }
  }
  
  //运行结果：
  
  //初始化年龄：10
  //Harry's age is 12
  //这里 age 是 GetAge 成员方法的形参，this.age 是 Person 类的成员变量。
  ```

  

-  **引用构造函数** 

   这个和 super 放在一起讲，见下面。 

**super**

super 可以理解为是指向自己超（父）类对象的一个指针，而这个超类指的是离自己最近的一个父类。

super 也有三种用法：

**1.普通的直接引用**

与 this 类似，super 相当于是指向当前对象的父类，这样就可以用 **super.xxx** 来引用父类的成员。

**2.子类中的成员变量或方法与父类中的成员变量或方法同名**

```java
class Country {
    String name;
    void value() {
       name = "China";
    }
}
  
class City extends Country {
    String name;
    void value() {
    name = "Shanghai";
    super.value();      //调用父类的方法
    System.out.println(name);
    System.out.println(super.name);
    }
  
    public static void main(String[] args) {
       City c=new City();
       c.value();
       }
}
/**运行结果：

*Shanghai
*China
*可以看到，这里既调用了父类的方法，也调用了父类的变量。若不调用父类方法 value()，只调用父类变量 name 的话，则父类 name 值为默认值 null。
*/

```

**3.引用构造函数**

- **super(参数)**：调用父类中的某一个构造函数（应该为构造函数中的第一条语句）。
- **this(参数)**：调用本类中另一种形式的构造函数（应该为构造函数中的第一条语句）。

```java
//实例
class Person { 
    public static void prt(String s) { 
       System.out.println(s); 
    } 
   
    Person() { 
       prt("父类·无参数构造方法： "+"A Person."); 
    }//构造方法(1) 
    
    Person(String name) { 
       prt("父类·含一个参数的构造方法： "+"A person's name is " + name); 
    }//构造方法(2) 
} 
    
public class Chinese extends Person { 
    Chinese() { 
       super(); // 调用父类构造方法（1） 
       prt("子类·调用父类"无参数构造方法"： "+"A chinese coder."); 
    } 
    
    Chinese(String name) { 
       super(name);// 调用父类具有相同形参的构造方法（2） 
       prt("子类·调用父类"含一个参数的构造方法"： "+"his name is " + name); 
    } 
    
    Chinese(String name, int age) { 
       this(name);// 调用具有相同形参的构造方法（3） 
       prt("子类：调用子类具有相同形参的构造方法：his age is " + age); 
    } 
    
    public static void main(String[] args) { 
       Chinese cn = new Chinese(); 
       cn = new Chinese("codersai"); 
       cn = new Chinese("codersai", 18); 
    } 
}	
```



运行结果：

```
父类·无参数构造方法： A Person.
子类·调用父类”无参数构造方法“： A chinese coder.
父类·含一个参数的构造方法： A person's name is codersai
子类·调用父类”含一个参数的构造方法“： his name is codersai
父类·含一个参数的构造方法： A person's name is codersai
子类·调用父类”含一个参数的构造方法“： his name is codersai
子类：调用子类具有相同形参的构造方法：his age is 18
```

从本例可以看到，可以用 super 和 this 分别调用父类的构造方法和本类中其他形式的构造方法。

例子中 Chinese 类第三种构造方法调用的是本类中第二种构造方法，而第二种构造方法是调用父类的，因此也要先调用父类的构造方法，再调用本类中第二种，最后是重写第三种构造方法。



## super 和 this的异同

- super(参数)：调用基类中的某一个构造函数（应该为构造函数中的第一条语句）
- this(参数)：调用本类中另一种形成的构造函数（应该为构造函数中的第一条语句）
- super:　它引用当前对象的直接父类中的成员（用来访问直接父类中被隐藏的父类中成员数据或函数，基类与派生类中有相同成员定义时如：super.变量名 super.成员函数据名（实参） this：它代表当前对象名（在程序中易产生二义性之处，应使用 this 来指明当前对象；如果函数的形参与类中的成员数据同名，这时需用 this 来指明成员变量名）
- 调用super()必须写在子类构造方法的第一行，否则编译不通过。每个子类构造方法的第一条语句，都是隐含地调用 super()，如果父类没有这种形式的构造函数，那么在编译的时候就会报错。
- super() 和 this() 类似,区别是，super() 从子类中调用父类的构造方法，this() 在同一类内调用其它方法。
- super() 和 this() 均需放在构造方法内第一行。
- 尽管可以用this调用一个构造器，但却不能调用两个。
- this 和 super 不能同时出现在一个构造函数里面，因为this必然会调用其它的构造函数，其它的构造函数必然也会有 super 语句的存在，所以在同一个构造函数里面有相同的语句，就失去了语句的意义，编译器也不会通过。
- this() 和 super() 都指的是对象，所以，均不可以在 static 环境中使用。包括：static 变量,static 方法，static 语句块。
- 从本质上讲，this 是一个指向本对象的指针, 然而 super 是一个 Java 关键字。

#### 5，接口和抽象类的区别

 **从设计层面来说，抽象是对类的抽象，是一种模板设计，接口是行为的抽象，是一种行为的规范** 

**抽象类：**

为何存在抽象类：

当编写一个类的时候，会为该类定义一些方法，这些方法会用一描述该类的行为方式，则这些方法会有具体的方法体。但是在某些情况下，某个父类只知道七子类应该包含怎样的方法，但是无法准确的只到子类应该如何实现这些方法。例如定义了一个Shape类，该类提供了计算周长的方法calPerimeter()方法，但是不同Shape的子类对周长的计算不同，即Shape()无法准确的只到七子类计算周长的方法。

​	如果在Shape()类中，Shape即不知道如何实现，也不管。则假设：有一个Shape()类的引用变量，该变量实际上引用带Shape的子类的实例，那么这个Shape则无法调用calPerimeter()方法，必须将其强制转化为七子类类型，才可以调用calPerimeter()方法，就降低了程序的灵活性

抽象方法：只有方法签名，没有方法实现的方法

**抽想类和抽象方法：**

 使用abstract修饰符修饰的类。官方点的定义就是：如果一个类没有包含足够多的信息来描述一个具体的对象，这样的类就是抽象类。 由抽象方法的类只能被定义为抽象类，抽象类可以没有抽象方法

规则如下：

- 抽象类必须使用abstract修饰，抽象方法也必须使用该修饰符修饰，抽象方法不能有方法体
- 抽象类不能被实例化，无法使用new关键字来调用抽象类的构造器创建抽象类的实例。即使抽象类李不包含抽象方法，这个抽象类也不能创建实例
- 抽象类可以包含成员变量，方法(普通方法和抽象方法均可)，构造器，初始化块，内部类(接口，枚举)5种成分。抽象类的构造器不能用于创建实例，主要用于被子类调用
- 含有抽象方法的类(包括直接定义了一个抽象方法；或继承了一个抽象父类，但是没有完全实现父类包含的抽象方法；或实现了一个接口，但是没有完全实现接口包含的抽象方法三种情况)只能被定义为抽象类

- 被abstract修饰的方法，只有方法名没有方法实现，具体的实现要由子类实现。方法名后面直接跟一个分号，而不是花括号。例如：public abstract int A(); 
-  抽象类就是为了继承而存在的， 如果一个类继承于一个抽象类，则子类必须实现父类的抽象方法。如果子类没有实现父类的抽象方法，则必须将子类也定义为为abstract类。 
-  抽象方法必须为public或者protected（因为如果为private，则不能被子类继承，子类便无法实现该方法），缺省情况下默认为public。 



**接口： 接口是对动作的抽象** 

  是抽象类的一种特殊形式，使用 `interface` 修饰。 Java 为了保证数据安全性是不能多继承的，也就是一个类只有一个父类。

但是接口不同，一个类可以同时实现多个接口，不管这些接口之间有没有关系，所以接口弥补了抽象类不能多继承的缺陷

```java
public interface OnClickListener{
	void onCLick(View v);
}
```

- 修饰符可以是public或者是省略，如果省略的话则默认采用包权限访问控制符，即只有在相同报结构下才可以访问该接口
- 接口名应采用与类名相同的命名规则
- 一个接口可以有多个直接父接口，但接口只能继承接口，不能继承类

**总结：**

**同：**

- 接口和抽象类都不能被实例化，他们都位于继承树的顶端，用于被其他类实现和继承
- 接口和抽象类都可以包含抽象方法，实现接口或继承抽象类的普通子类都必须是西安这些抽象方法

**异：**

- 两者的设计目的不同：

  - 接口作为系统与外界交互的窗口，体现的是一种规范。对于接口的实现者而言，接口规定了是极限这必须向外界提供那些服务(以方法的形式来提供)；对于接口的调用者而言，接口规定了调用者可以调用那些服务，以及如何调用这些服务。当一个程序中使用接口时，接口是多个模块间的耦合标准；当多个程序之间使用接口时，接口时多个程序之间的通信标准。类似于整个系统的"总纲"，她制定了系统各个模块应该遵循的标准，因此一个系统中的接口不应该经常改变。一旦接口改变，对于整个系统甚至其他系统的影响时辐射式的
  - 抽象类作为兄同种多个子类共同的父类，她所体现的是一种模板式设计。抽象类作为多个子类的抽象父类，可以被当作是系统实现过程中的中间产物，这个中间产品已经实现了系统的部分功能，但这个产品依然不能当成最终产品，必须有更进一步的完善

- 接口中只能包含抽象方法和默认方法，不能为普通方法提供方法是实现；抽象类则完全可以包含普通方法

- 接口中不能有静态方法；抽象类中可以定义静态方法

- 接口中只能定义静态常量，不能定义普通成员变量；抽象类里则可以定义普通成员变量，也可以定义静态常量

- 接口中不包括构造器；抽象类里可以包含构造器，抽象类里的构造器并不是用于创建对象，而是让其子类调用这些构造器来完成属于抽象类的初始化操作

- 接口中不能有初始化块；但是抽象类中完全可以包含初始化块

- 一个类中只能有一个直接父类，包括抽想类；但是一个类可以直接实现多个接口，通过实现多个接口可以弥补Java单继承的不足

  

  

 ![这里写图片描述](https://user-gold-cdn.xitu.io/2017/11/2/e7d88fe9c400ba6c6ad2d80f67956987?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

 了解到抽象类和接口的这些不同：

- 抽象层次不同
  - 抽象类是对类抽象，而接口是对行为的抽象
  - 抽象类是对整个类整体进行抽象，包括属性、行为，但是接口却是对类局部行为进行抽象
- 跨域不同
  - 抽象类所跨域的是具有相似特点的类，而接口却可以跨域不同的类
  - 抽象类所体现的是一种继承关系，考虑的是子类与父类本质**“是不是”**同一类的关系
  - 而接口并不要求实现的类与接口是同一本质，它们之间只存在**“有没有这个能力”**的关系
- 设计层次不同
  - 抽象类是自下而上的设计，在子类中重复出现的工作，抽象到抽象类中
  - 接口是自上而下，定义行为和规范

在进行选择时，可以参考以下几点：

- 若使用接口，我们可以同时获得抽象类以及接口的好处
- 所以假如想创建的基类没有任何方法定义或者成员变量，那么无论如何都愿意使用接口，而不要选择抽象类
- 如果事先知道某种东西会成为基础类，那么第一个选择就是把它变成一个接口
- 只有在必须使用方法定义或者成员变量的时候，才应考虑采用抽象类

此外使用接口最重要的一个原因：**实现接口可以使一个类向上转型至多个基础类。**

比如 `Serializable` 和 `Cloneable` 这样常见的接口，一个类实现后就表示有这些能力，它可以被当做 `Serializable` 和 `Cloneable` 进行处理。

> 推荐接口和抽象类同时使用，这样既保证了数据的安全性又可以实现多继承

#### 6，静态属性和静态方法能被继承吗？静态方法又是否能被重写呢？

静态属性和静态方法可以被继承，但是不可以被重写

```java
public calss A{
    public static String staticStr = "A的静态属性"；
public String nonStaticStr = "A非静态属性";
	public static void staticMethod(){
		System.out.println("A静态方法");
	}
	public void nonStaticMethod(){
		System.out.println("A非静态方法");
	}
}

public class B extends A{//子类B
	public static String staticStr = "B改写后的静态属性";
	public String nonStaticStr = "B改写后的非静态属性";
	public static void staticMethod(){
		System.out.println("B改写后的静态方法");
	}
}


```

```java
public class One {
	//静态属性和静态方法是否可以被重写？以及原因？
	public static String one_1 = "one";
	public static void oneFn() {
		System.out.println("oneFn");
	}
}

public class Two extends One {

	public static String one_1 = "two";

	public static void oneFn() {

		System.out.println("TwoFn");
	}
}
public class MyTest {
	//静态属性和静态方法是否可以被继承？是否可以被重写？以及原因？
	public static void main(String[] args) {
		One one = new Two();
		one.oneFn();
		String one_1 = One.one_1;
		System.out.println("One.one_1>>>>>>>"+one_1);
		String one_12 = one.one_1;
		System.out.println("one.one_1>>>>>>>"+one_12);
	}
}
//打印结果如下
//oneFn
//One.one_1>>>>>>>one
//one.one_1>>>>>>>one

```



**原因：**

- 静态方法和属性是属于类的，调用的时候直接通过类名.方法名完成对，不需要继承机制及可以调用。如果子类里面定义了静态方法和属性，那么这时候父类的静态方法或属性称之为"隐藏"。如果你想要调用父类的静态方法和属性，直接通过父类名.方法或变量名完成，至于是否继承一说，子类是有继承静态方法和属性，但是跟实例方法和属性不太一样，存在"隐藏"的这种情况。
- 多态之所以能够实现依赖于继承、接口和重写、重载（继承和重写最为关键）。有了继承和重写就可以实现父类的引用指向不同子类的对象。重写的功能是："重写"后子类的优先级要高于父类的优先级，但是“隐藏”是没有这个优先级之分的。
- 静态属性、静态方法和非静态的属性都可以被继承和隐藏而不能被重写，因此不能实现多态，不能实现父类的引用可以指向不同子类的对象。非静态方法可以被继承和重写，因此可以实现多态。
  

#### 7，给我说说权限修饰符特性

为什么会有包访问权限修饰:

在写代码的时候都会用到一些类库，比如 Java JDK 中自带的公共类库，或者一些第三方的库。使用这些库都可以极大的方便我们的开发，少写很多代码。
类库的开发者将类库提供给其他开发者使用的时候，肯定都不希望自己库的代码可以被人随意的调用（特别是第三方的商业SDK），这些类库一般都只会向外提供一些少量的方法供使用者去调用，而把自己的核心代码藏起来，不让使用者看到和调用。
同时，类库也会面临不断修改升级的问题，如果不加以控制，类库方做任何修改都会直接影响到使用者的程序，这样的话，类库要想修改的话就会非常麻烦。
鉴于以上的问题，Java 在设计的时候就提供了访问权限修饰符这个东西，通过权限来控制类，方法，变量是否能被访问。

**类修饰符：**

**public：**（访问控制符）将一个类声明为公共类，它可以被任何对象访问，一个程序的主类必须是公共类

**abstract：**将一个类声明为抽象类，没有实现的方法，需要子类提供方法的实现

**final：**将一个类声明为最终类(即非继承类)，表示他不能被其他类继承

**static：**将一个类声明为讲台的，进内部类可以使用此修饰符

**成员变量修饰符：**

**public：**(公共访问控制符)指定该变量为公共的，可以被任何对象的方法访问

**protected：**(保护访问控制符)指定该变量可以制备自己的类和子类访问。在子类中可以覆盖此变量

**private：**(私有访问控制符)指定该变量只允许自己的类的方法访问，其他任何类(包括子类)中的方法军部能访问

**final：**最终修饰符，指定此变量的值不能被修改

**static：**(静态修饰符)指定变量被所有对象共享，即所有实例都可以使用该变量。变量属于这个类

**方法修饰符：**

**public：**(公共控制符)指定该方法为公共的，可以被任何对象的方法访问

**protected：**(保护访问控制符)指定该方法可以被他的类和子类访问

**private：**(私有控制符)指定此方法只能有自己类等方法访问，其他类不能访问(包括子类)

**final：**指定该方法不能被重载

**static：**不需要实例化就可以激活一个方法、

![image-20191216170510058](C:\Users\Lenovo\Desktop\image-20191216170510058.png)

如果你想让这个类，方法或变量能够被其他的任何类访问到，那么就给它加上 public 修饰符。
protected 修饰符，只能用来修饰变量和方法。如果你不想这个变量或方法被其他包里面的类访问到，同时又需要被继承此类的子孙类访问，那么你应该使用 protected 修饰符。
而如果不使用任何修饰符的话，也就是默认状态，那么它的访问范围就进一步缩小了，只能是当前类和相同包名下面的类才能访问了，子孙类都是访问不到的。
最后一个 private ，它的范围最小，同样是只适用于变量和方法。被它修饰了的话，就只能是同一个类下面的才能访问了，其他任何地方都是没法访问的。



#### 8，给我谈谈Java中的内部类。

#### 9，闭包和内部类的区别？

#### 10，Java多态的实现机制是什么？

#### 11，谈谈你对对象生命周期的认识？

**java对象生命周期**

对象的整个生命周期大致可以分为**7个阶段**：**创建**阶段（Creation）、**应用**阶段（Using）、**不可视**阶段（Invisible）、不可到达阶段（Unreachable）、**可收集**阶段（Collected）、**终结**阶段（Finalized）与**释放**阶段（Free）。

#### 12，static关键字的作用

**用途：**

"static方法是没有this的方法。在sttatic方法的北部不能调用非静态方法，反过来是可以的。而且可以在没有 创建任何对象的前提下，仅仅通过类本身来调用static方法。这实际上正是static方法的主要用途"

基于此，static关键字的主要作用：

**<font style='color:red'>方便在没有创建对象的情况下来进行调用(方法/变量)</font>**



被static关键字修饰的方法或者变量不需要依赖于对象来进行访问，只要类被加载了，就可以通过类名取进行访问。static关键字可以用来修饰类的成员方法，类的成员变量，另外可以编写static代码块来优化程序程序性能

**static方法**

static方法一般称作静态方法，由于静态方法不依赖于任何对象就可以进行访问，因此对于静态方法来说，是没有this的，因为它不依附于任何对象，既然都没有对象，就谈不上this了。并且由于这个特性，在静态方法中不能访问类的非静态成员变量和非静态成员方法，因为非静态成员方法/变量都是必须依赖具体的对象才能够被调用。

　　但是要注意的是，虽然在静态方法中不能访问非静态成员方法和非静态成员变量，但是在非静态成员方法中是可以访问静态成员方法/变量的。举个简单的例子：

 ![img](https://images0.cnblogs.com/i/288799/201406/201612439117678.jpg)

 在上面的代码中，由于print2方法是独立于对象存在的，可以直接用过类名调用。假如说可以在静态方法中访问非静态方法/变量的话，那么如果在main方法中有下面一条语句：

　　MyObject.print2();

　　此时对象都没有，str2根本就不存在，所以就会产生矛盾了。同样对于方法也是一样，由于你无法预知在print1方法中是否访问了非静态成员变量，所以也禁止在静态成员方法中访问非静态成员方法。

　　而对于非静态成员方法，它访问静态成员方法/变量显然是毫无限制的。

　　因此，如果说想在不创建对象的情况下调用某个方法，就可以将这个方法设置为static。我们最常见的static方法就是main方法，至于为什么main方法必须是static的，现在就很清楚了。因为程序在执行main方法的时候没有创建任何对象，因此只有通过类名来访问。

　　另外记住，关于构造器是否是static方法可参考：http://blog.csdn.net/qq_17864929/article/details/48006835

2）static变量

　　static变量也称作静态变量，静态变量和非静态变量的区别是：静态变量被所有的对象所共享，在内存中只有一个副本，它当且仅当在类初次加载时会被初始化。而非静态变量是对象所拥有的，在创建对象的时候被初始化，存在多个副本，各个对象拥有的副本互不影响。

　　static成员变量的初始化顺序按照定义的顺序进行初始化。

3）static代码块

　　static关键字还有一个比较关键的作用就是 用来形成静态代码块以优化程序性能。static块可以置于类中的任何地方，类中可以有多个static块。在类初次被加载的时候，会按照static块的顺序来执行每个static块，并且只会执行一次。

　　为什么说static块可以用来优化程序性能，是因为它的特性:只会在类加载的时候执行一次。下面看个例子:

```java
`class` `Person{``  ``private` `Date birthDate;``  ` `  ``public` `Person(Date birthDate) {``    ``this``.birthDate = birthDate;``  ``}``  ` `  ``boolean` `isBornBoomer() {``    ``Date startDate = Date.valueOf(``"1946"``);``    ``Date endDate = Date.valueOf(``"1964"``);``    ``return` `birthDate.compareTo(startDate)>=``0` `&& birthDate.compareTo(endDate) < ``0``;``  ``}``}`
```

　　isBornBoomer是用来这个人是否是1946-1964年出生的，而每次isBornBoomer被调用的时候，都会生成startDate和birthDate两个对象，造成了空间浪费，如果改成这样效率会更好：

```java
`class` `Person{``  ``private` `Date birthDate;``  ``private` `static` `Date startDate,endDate;``  ``static``{``    ``startDate = Date.valueOf(``"1946"``);``    ``endDate = Date.valueOf(``"1964"``);``  ``}``  ` `  ``public` `Person(Date birthDate) {``    ``this``.birthDate = birthDate;``  ``}``  ` `  ``boolean` `isBornBoomer() {``    ``return` `birthDate.compareTo(startDate)>=``0` `&& birthDate.compareTo(endDate) < ``0``;``  ``}``}`
```

　　因此，很多时候会将一些只需要进行一次的初始化操作都放在static代码块中进行。

**误区：**

- static关键字会改变类中成员的访问权限？

  与C/C++中的static不同，Java中的static关键字不会影响到变量或者方法的作用域。在Java中能够影响到访问权限的只有private，protected，public这几个关键字。

   ![img](https://images0.cnblogs.com/i/288799/201406/201658474265949.jpg) 

  Person.age不可视，说明static关键字并不会改变变量和方法的访问权限

- 能通过this访问静态成员变量嘛

  虽然对于静态方法来说没有this，那么在非静态方法中能够通过this访问静态成员变量嘛?

  ```java
  public class Main {　　
      static int value = 33;
   
      public static void main(String[] args) throws Exception{
          new Main().printValue();
      }
   
      private void printValue(){
          int value = 3;
          System.out.println(this.value);
      }
  }
  //33
  ```

  this代表当前对象，那么通过new Main() 来调用printValue的话，当前对象就是通过new Main()生成的对象。而static变量是被对象所享有的，因此在printValue中的this.value的值毫无疑问是33。在printValue方法内部的value是局部变量，根本不可能与this关联，所以输出结果是33。在这里永远要记住一点：静态成员变量虽然独立于对象，但是不代表不可以通过对象去访问，所有的静态方法和静态变量都可以通过对象访问（只要访问权限足够）

- 虽然对于静态方法来说没有this，那么在非静态方法中能够通过this访问静态成员变量吗？

  

- static能作用与局部变量嘛

  在C/C++中static是可以作用域局部变量的，但是在Java中切记：static是不允许用来修饰局部变量。不要问为什么，这是Java语法的规定。

  　　具体原因可以参考这篇博文的讨论：http://www.debugease.com/j2se/178932.html

**常见面试题：**

**下面代码的输出结果：**

```java
public class Test extends Base{
 
    static{
        System.out.println("test static");
    }
     
    public Test(){
        System.out.println("test constructor");
    }
     
    public static void main(String[] args) {
        new Test();
    }
}
 
class Base{
     
    static{
        System.out.println("base static");
    }
     
    public Base(){
        System.out.println("base constructor");
    }
}

/**
*base static
test static
base constructor
test constructor
*/
```

 至于为什么是这个结果，我们先不讨论，先来想一下这段代码具体的执行过程，在执行开始，先要寻找到main方法，因为main方法是程序的入口，但是在执行main方法之前，必须先加载Test类，而在加载Test类的时候发现Test类继承自Base类，因此会转去先加载Base类，在加载Base类的时候，发现有static块，便执行了static块。在Base类加载完成之后，便继续加载Test类，然后发现Test类中也有static块，便执行static块。在加载完所需的类之后，便开始执行main方法。在main方法中执行new Test()的时候会先调用父类的构造器，然后再调用自身的构造器。因此，便出现了上面的输出结果。 

2，代码的输出结果

```java
public class Test {
    Person person = new Person("Test");
    static{
        System.out.println("test static");
    }
     
    public Test() {
        System.out.println("test constructor");
    }
     
    public static void main(String[] args) {
        new MyClass();
    }
}
 
class Person{
    static{
        System.out.println("person static");
    }
    public Person(String str) {
        System.out.println("person "+str);
    }
}
 
 
class MyClass extends Test {
    Person person = new Person("MyClass");
    static{
        System.out.println("myclass static");
    }
     
    public MyClass() {
        System.out.println("myclass constructor");
    }
}

/**
test static
myclass static
person static
person Test
test constructor
person MyClass
myclass constructo
*/
```

类似地，我们还是来想一下这段代码的具体执行过程。首先加载Test类，因此会执行Test类中的static块。接着执行new MyClass()，而MyClass类还没有被加载，因此需要加载MyClass类。在加载MyClass类的时候，发现MyClass类继承自Test类，但是由于Test类已经被加载了，所以只需要加载MyClass类，那么就会执行MyClass类的中的static块。在加载完之后，就通过构造器来生成对象。而在生成对象的时候，必须先初始化父类的成员变量，因此会执行Test中的Person person = new Person()，而Person类还没有被加载过，因此会先加载Person类并执行Person类中的static块，接着执行父类的构造器，完成了父类的初始化，然后就来初始化自身了，因此会接着执行MyClass中的Person person = new Person()，最后执行MyClass的构造器。

3.这段代码的输出结果是什么？

```java
public class Test {
     
    static{
        System.out.println("test static 1");
    }
    public static void main(String[] args) {
         
    }
     
    static{
        System.out.println("test static 2");
    }
}

/**

test static 1
test static 2
**/
```

 　虽然在main方法中没有任何语句，但是还是会输出，原因上面已经讲述过了。另外，static块可以出现类中的任何地方（只要不是方法内部，记住，任何方法内部都不行），并且执行是按照static块的顺序执行的。 

#### 13，final关键字的作用。

**基本用法：**

- **修饰类**

  ​	当final修饰类的时候，表明这个类不能被继承。final类中的成员变量可以根据需要设为final，注意：final类中的所有成员方法会被饮食的指定为final方法

  > 自己一般将util包和helper包下的类设为final

- **修饰方法**

  ​	第一个原因是把方法锁定，以防任何继承类修改它的含义；第二个原因是效率。在早期的Java实现版本中，会将final方法转为内嵌调用。但是如果方法过于庞大，可能看不到内嵌调用带来的任何性能提升。在最近的Java版本中，不需要使用final方法进行这些优化了。“

  因此，如果只有再向明确禁止该方法在子类中被副高的情况下才将方法设置为final的

  注：类的private方法会隐式的设置为final

- **修饰变量**

  - **修饰基本数据类型**

    这类常量必须时基本数据库类型，用final修饰，在定义的时候就必须赋初值，对于编译期常量，编译器

  可以将该常量值代入任何用到他的计算式中华，可以在编译期执行计算式，减轻运行时负担。

  ```java
  public final int i = 10; //定义时就赋值，基本数据类型
  public final int[] is = {1,2,3}; //即 new int[]{1,2,3} 运行时才被确定，有 new 关键字，数组不是基本数据类型
  public final String str = "str"; // String 不是基本数据类型
  
  
  ```

  - **修饰引用数据类型**

    final修饰引用数据类型时，final关键字时被修饰的变量的引用不能改变，然而，对象自身确实可以修改的

    ```java
    public class Test {
        public final Te te = new Te();// te 为引用数据类型，且被 final 修饰
        public void fun(){
    //        te = new Te(); // 试图给 te 修改引用，指向一个新的对象，此时编译会报错
            te.i = 10; // te 的内容可以修改（对象自身可以修改）
        }
        class Te{
            int i = 3;
        }
    }
    
    
    ```

    当变量被定义为final，同时也被static修饰，表示该变量被初始化之后就占据者一段不能改变的存储空间，而且只有一份

    final使变量不能改变引用(基本数据类型时为值不能被修改)，而static强调只有一部分

    ```java
    public static final String = "str" // 所在类加载时会被初始化
    
    ```

    - 空白final

      所谓空白 final 指被声明为 final，但又未给定初值的域。但编译器又需要确保在使用前变量被赋值，因而 java 要求空白 final 必须在构造器中赋值。静态空白 final 可在静态代码块中赋值，非静态空白  final 要在构造代码块或是每一个构造函数中赋值。

      > 空白 final 为类使用者提供了更大的灵活性，在保证变量只被赋值一次的同时，可以根据对象而有所不同。

      如：

      ```java
      public class Test {
      
          private static final String STR = "str";
          private static final int A;
          private static final String STR1;
      
          static {  // 静态空白 final 只能在定义时赋值或在静态代码块中赋值
              A = 3;
              STR1 = "str1";
          }
      
          private final int b = 1;
          private final int c;
          private final String str;
          
          // 非静态空白 final 可在构造代码块中赋值
          {
              c = 1;
          }
          // 选择构造代码块赋值时只需在其中一个构造代码块中赋值即可
          {
          }
      
      	// 选择构造函数赋值时必须在每一个构造函数中都进行赋值，因为编译器无法知道类使用者会选择哪一个构造函数构造对象
          public Test() {
              str = "str";
          }
      
          public Test(String s) {
              str = s;
          }
      }
      ```

      

  对于一个final变量，如果时基本数据类型的变量，则数值一般被初始化之后便不能被改变；如果时引用类型的变量，则在其初始化之后便不能在让其指向另一个对象

  

**Q&A**

1，类的final变量和普通变量的区别

当用final作用于类的成员变量的时候，成员变量( 注意是类的成员变量，局部变量只需要保证在使用之前被初始化赋值即可 ) 必须在定义时或者构造器中进行初始化赋值，而且final变量一旦被初始化赋值之后，就不能再被赋值了。 

，被final修饰的引用变量指向的对象内容可变吗? 
引用变量被final修饰之后，虽然不能再指向其他对象，但是它指向的对象的内容是可变的。 

3，final和static 
 static作用于成员变量用来表示只保存一份副本，而final的作用是用来保证变量不可变。



```csharp
public class Test {
    public static void main(String[] args)  {
        MyClass myClass1 = new MyClass();
        MyClass myClass2 = new MyClass();
        System.out.println(myClass1.i);
        System.out.println(myClass1.j);
        System.out.println(myClass2.i);
        System.out.println(myClass2.j);
 
    }
}
 
class MyClass {
    public final double i = Math.random();
    public static double j = Math.random();
}
```

每次打印的两个j值都是一样的，而i的值却是不同的。从这里就可以知道final和static变量的区别了。

 \5. final参数 **

使用final修饰方法参数的目的是防止修改这个参数的值，同时也是一种声明和约定，强调这个参数是不可变的。

但是实际情况中，无论参数是基本数据类型的变量还是引用类型的变量，使用final声明都不会达到上面所说的效果。



```java
public class FinalTest {
  public static void main(String[] args) {
    A a = new A();
    int i = 0;
    a.increase(i);
  } 
}

class A {
  void final increase(final int i) {
    i++;
  }
}
```

i++会报错。不过，在方法内部改变i的值并不会影响方法外的i值，因为java采用值传递。



```csharp
public class Test {
    public static void main(String[] args)  {
        MyClass myClass = new MyClass();
        StringBuffer buffer = new StringBuffer("hello");
        myClass.changeValue(buffer);
        System.out.println(buffer.toString());
    }
}
 
class MyClass {
 
    void changeValue(final StringBuffer buffer) {
        buffer.append("world");
    }
}
```

运行这段代码就会发现输出结果为 helloworld。很显然，用final进行修饰并没有阻止在changeValue中改变buffer指向的对象的内容。如把final去掉了，然后在changeValue中让buffer指向了其他对象，也不会影响到main方法中的buffer，**原因在于java采用的是值传递，对于引用变量，传递的是引用的值**，也就是说让实参和形参同时指向了同一个对象，因此让形参重新指向另一个对象对实参并没有任何影响。





## 1.2 八大基本数据类型&引用类型

#### 1.说说Java中的8大基本类型 & 内存中占有的字节 & 什么是引用类型？

**基本数据类型：**

java中一共分为8种基本数据类型：byte、short、int、long、float、double、char、boolean,其中byte、short、int、long是整型。float、double是浮点型,char是字符型,boolean是布尔型。

**引用类型：**

java为每种基本类型都提供了对应的封装类型，分别为：Byte、Short、Integer、Long、Float、Double、Character、Boolean。引用类型是一种对象类型,它的值是指向内存空间的引用，就是地址。

**基本类型于引用类型的区别**

**1，默认值**



| byte、short、int、long | 0     |
| ---------------------- | ----- |
| float、double          | 0.0   |
| boolean                | false |
| char                   | 空    |
| 对应的包装类型         | null  |
| **2，内存分配：**      |       |

基本数据类型的变量是存储在栈内存中，而引用类型变量存储在栈内存中，保存的是实际对象在堆内存中的地址，实际对象中保存这内容。

#### 2.什么是拆箱 & 装箱，能给我举栗子吗？

Java从jdk1.5开始引入自动装箱和拆箱,使得基本数据类型与引用类型之间相互转换变得简单。

**自动装箱:** java自动将原始类型转化为引用类型的过程，自动装箱时编译器会调用valueOf方法,将原始类型转化为对象类型。

**自动拆箱:** java自动将引用类型转化为原始类型的过程，自动拆箱时编译器会调用intValue(),doubleValue()这类的方法将对象转换成原始类型值。

自动装箱主要发生在两种情况：一种是赋值时，一种是方法调用时。
**a.赋值**

```java
Integer a = 3; //自动装箱
int b = a; //自动拆箱
```

**b.方法调用**

```java
public Integer query(Integer a){
   return a;
}
query(3); //自动装箱
int result = query(3); //自动拆箱
```



**3，自动装箱，拆箱带来的问题**

**1，程序的性能**

由于装箱会隐式地创建对象创建，因此千万不要在一个循环中进行自动装箱的操作，下面就是一个循环中进行自动装箱的例子，会额外创建多余的对象,增加GC的压力,影响程序的性能：

```java
Integer sum = 0;
 for(int i=0; i<1000; i++){
   sum+=i;
}
```



**2，空指针异常**

拆箱的时候会产生空指针异常：

```java
Object obj = null;
int i = (Integer)obj;
```



**3，对象相等比较时**

常见例子：

```java
Integer a = 120;
int b= 120;
Integer c = 120;
Integer d = new Integer(120);
System.out.println(a == b);   //true    t1
System.out.println(a == c);   //true    t2
System.out.println(a == d);   //false   t3

Integer e = 128;
Integer f = 128;
System.out.println(e == f);   //false    t4
```

返回结果是不是出乎大家的意料,解释一下每种结果的原因：
我们先反编译一下生成字节码：

```java
Integer a = Integer.valueOf(120);
int b = 120;
Integer c = Integer.valueOf(120);
Integer d = new Integer(120);
System.out.println(a.intValue() == b);
System.out.println(a == c);
System.out.println(a == d);

Integer e = Integer.valueOf(127);
Integer f = Integer.valueOf(127);
System.out.println(e == f);

Integer e1 = Integer.valueOf(128);
Integer f1 = Integer.valueOf(128);
System.out.println(e1 == f1);
```

可以看到变量a、c在初始化的时候编译器调用了valueOf进行自动装箱，在a==b时对变量a调用了intValue()方法进行了自动拆箱操作，这就很好解释t1~t4的结果了。

t1产生的原因是编译器编译时会调用intValue()自动的将a进行了拆箱，结果肯定是true;
t2跟t4的结果比较难理解：这是因为初始化时，编译器会调用装箱类的valueOf()方法,查看jdk的源码：

```java
public static Integer valueOf(int i) {
    assert IntegerCache.high >= 127;
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```



发现jdk对-128~127之间的值做了缓存，对于-128~127之间的值会取缓存中的引用,通过缓存经常请求的值而显著提高空间和时间性能。
这就能解释t2结果返回true，而t4由于128不在缓存区间内，编译器调用valueOf方法会重新创建新的对象，两个不同的对象返回false。

t3结果无论如何都不会相等的，因为new Integer(120)构造器会创建新的对象。

Byte、Short、Integer、Long、Char这几个装箱类的valueOf()方法都会做缓存，而Float、Double则不会，原因也很简单，因为byte、Short、integer、long、char在某个范围内的整数个数是有限的，但是float、double这两个浮点数却不是。



## 1.3数组

#### 1.能说说多维数组在内存上是怎么存储的吗？

#### 2.你对数组二次封装过吗？说说封装了什么



## 1.4 Java异常

#### 1.说说Java异常体系主要用来干什么的 & 异常体系？

**异常的定义：**组织当前方法或作用域继续执行的问题。虽然Java中有异常处理机制，但是要明确一点，绝不应该用"正常"的态度来看待异常。绝对一点说异常就是某种意义上的错误，就是问题，他可能会导致程序失败。之所以Java要提出一场梳理机制，就是要告诉开发人员，你的程序出现了不正常的情况，请注意。

​	异常的用途：

```java
public class Calculator {
    public int devide(int num1, int num2) {
        //判断除数是否为0
        if(num2 == 0) {
            throw new IllegalArgumentException("除数不能为零");
        }
         
        return num1/num2;
    }
}

```

看一下这个类中关于除运算的方法，如果你是新手你可能会直接返回计算结果，根本不去考虑什么参数是否正确，是否合法（当然可以原谅，谁都是这样过来的）。但是我们应尽可能的考虑周全，把可能导致程序失败的"苗头"扼杀在摇篮中，所以进行参数的合法性检查就很有必要了。其中执行参数检查抛出来的那个参数非法异常，这就属于这个方法的不正常情况。正常情况下我们会正确的使用计算器，但是不排除粗心大意把除数赋值为0。如果你之前没有考虑到这种情况，并且恰巧用户数学基础不好，那么你完了。但是如果你之前考虑到了这种情况，那么很显然错误已在你的掌控之中。

**java的异常体系：**

![img](https://img2018.cnblogs.com/blog/1294391/201809/1294391-20180919152605174-2400592.png)





#### 2.Error和Exception的区别？

`Throwable` 类是 Java 语言中所有错误或异常的超类（这就是一切皆可抛的东西）。它有两个子类：**Error**`和`**Exception**

**1、Error与Exception**
    **Error**是**程序无法处理的错误**，它是由JVM产生和抛出的，比如OutOfMemoryError、ThreadDeath等。这些异常发生时，Java虚拟机（JVM）一般会选择线程终止。
    **Exception**是**程序本身可以处理的异常**，这种异常分两大类运行时异常和非运行时异常。程序中应当尽可能去处理这些异常。

#### **3.说说运行时异常和非运行时异常的区别**？

​    **运行时异常**都是**RuntimeException类及其子类异常**，如NullPointerException、IndexOutOfBoundsException ClassNotFoundException ， NoSuchMetodException 等，这些异常是不检查异常，程序中可以选择捕获处理，也可以不处理。这些异常一般是由程序逻辑错误引起的，程序应该从逻辑角度尽可能避免这类异常的发生。
​    **非运行时异常**是**RuntimeException以外的异常**，类型上都属于Exception类及其子类。从程序语法角度讲是必须进行处理的异常，如果不处理，程序就不能编译通过。如IOException、SQLException等以及用户自定义的Exception异常，一般情况下不自定义检查异常。



**区别：**

​	 error 表示恢复不是不可能但很困难的情况下的一种严重问题。比如说内存溢出。不可能指望程序能处理这样的情况。 exception 表示一种设计或实现问题。也就是说，它表示如果程序运行正常，从不会发生的情况。 

#### 4.**如何自定义一个异常**？

在Java中要想创建自定义异常，需要继承Throwable或者他的子类Exception。

语法：

```java
class  自定义异常类 extends 异常类型(Exception){

 // 因为父类已经把异常信息的操作都完成了，所在子类只要在构造时，将异常信息传递给父类通过super 语句即可。
  // 重写 有参 和 无参  构造方法
}
```

例如：

```java
public class CustomException extends Exception {

    //无参构造方法
    public CustomException(){

        super();
    }

    //有参的构造方法
    public CustomException(String message){
        super(message);

    }

    // 用指定的详细信息和原因构造一个新的异常
    public CustomException(String message, Throwable cause){

        super(message,cause);
    }

    //用指定原因构造一个新的异常
     public CustomException(Throwable cause) {

         super(cause);
     }

}

// 备注： 这些方法怎么来的？ 重写父类Exception的方法，那么如何查看Exception具有哪些API，快捷键：选中Exception, command+单击。windows系统 ：选中Exception, control+单击。
```

自定义使用例子：

​	自定义test1()方法，抛出 “我喝酒了”的异常信息，test2()方法调用test1()方法，并将异常包装成RuntimeException类型的异常，继续抛出，在main方法中调用test2()方法，并尝试捕获异常

```java
public void test2() {

        try{

            test1();

        }catch (CustomException e){

           RuntimeException exception  =  new RuntimeException(e);
           //exception.initCause(cause)
           throw  exception;
        }

    }

    public void test1() throws CustomException{

        throw new CustomException("我喝酒了");
    }
// main方法
    public static void main(String[] args) {

        CustomExceptionInital object =  new  CustomExceptionInital();

        //try{

            object.test2();

        //}catch(Exception e){

        //e.printStackTrace();

        //}

    }
```

输出结果：

![这里写图片描述](https://img-blog.csdn.net/20170614141757661?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMTg1MDU3MTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

常见的RuntimeException

```java

```



#### 5.throw和**throws** 的区别？

1，Throw用于方法内部，Throws用于方法声明上

2、Throw后跟异常对象，Throws后跟异常类型

3、Throw后只能跟一个异常对象，Throws后可以一次声明多种异常类型

4，Throw是具体向外抛异常的，抛出的是一个异常实例，Throws声明了是哪一种异常，是他的调用者可以捕获这个异常

```java
//throw
throw new NumberFormatException();
//throws
public void test throws Exception1,Exception2(){}
```

#### 6.try{}catch{}finally{}可以没有finally吗？

finally块主要用于资源回收的时候，由于try-catch在资源回收的时候需要考虑到每一个需要考虑资源回收的地方，较为繁琐，故而需要使用

可以 。最适合处理我们无法控制的错误，如I/O操作等，后端nodejs或Java读取I/O擦皮做比较多比如读取数据库，所以用try catch比较多。前端可以用在上传图片，使用别人的JS库报错，async await同步调接口等地方适用

```java
async function f() {
  try {
    await Promise.reject('出错了');
  } catch(e) {
  }
  return await Promise.resolve('hello world');
}

```

try的意思就是，试着执行里面的语句，所以ruguotry内部抛出异常Exception，那么将执行catch部分，以及try外面的后面的语句

如果try内部痴线了Error，表示出错，后面的语句就不执行，catch也抓不住

但是如果有finally块的恶化，finally几乎是必定执行的，但是有一个先后顺序的问题，应该是finally块合并打印异常堆栈将会在另一个线程中执行

如果try-catch之中，执行System.exit(0)；那么finally不会被执行，此外的情况，不管是Error还是return，finally块都会执行到



#### 7.finally语块有什么特点？

- 只有finally相对应的try语句块得到执行的情况下，finally语句块才会执行。以上两种情况都是在try语句块之前返回或者是抛出异常，所以try对应的finally语句块没有执行
- 当finally对应的try语句块得到执行的情况下，finally块也并不一定会执行：当try语句块中执行了System.exit（0）语句，终止了Java虚拟机的运行。但是一般不会调用这个语句
- 如果不调用System.exit（0）语句，finally语句块也并不一定会执行：当一个线程在执行 try 语句块或者 catch 语句块时被打断（interrupted）或者被终止（killed），与其相对应的 finally 语句块可能不会执行。还有更极端的情况，就是在线程运行 try 语句块或者 catch 语句块时，突然死机或者断电，finally 语句块肯定不会执行了。可能有人认为死机、断电这些理由有些强词夺理，没有关系，我们只是为了说明这个问题。
- **不管 try 语句块正常结束还是异常结束，finally 语句块是保证要执行的**。如果 try 语句块正常结束，那么在 try 语句块中的语句都执行完之后，再执行 finally 语句块。如果 try 中有控制转移语句（return、break、continue）呢？那 finally 语句块是在控制转移语句之前执行，还是之后执行呢？似乎从上面的描述中我们还看不出任何端倪，不要着急，后面的讲解中我们会分析这个问题。如果 try 语句块异常结束，应该先去相应的 catch 块做异常处理，然后执行 finally 语句块。同样的问题，如果 catch 语句块中包含控制转移语句呢？ finally 语句块是在这些控制转移语句之前，还是之后执行呢？我们也会在后续讨论中提到。
- 其实 finally 语句块是在 try 或者 catch 中的 return 语句之前执行的。更加一般的说法是，finally 语句块应该是在控制转移语句之前执行，控制转移语句除了 return 外，还有 break 和 continue。另外，throw 语句也属于控制转移语句。虽然 return、throw、break 和 continue 都是控制转移语句，但是它们之间是有区别的。其中 return 和 throw 把程序控制权转交给它们的调用者（invoker），而 break 和 continue 的控制权是在当前方法内转移。请大家先记住它们的区别，在后续的分析中我们还会谈到。



```java
public class Test { 
    public static void main(String[] args) {  
    	System.out.println("reture value of test() : " + test()); 
	} 
    
	public static int test(){ 
		int i = 1; 
        
        try {  
            System.out.println("try block");  
                    i = 1 / 0; 
            return 1;  
        }catch (Exception e){ 
            System.out.println("exception block"); 
            return 2; 
        }finally {  
            System.out.println("finally block");  
        } 
  	} 
}
/*
try block 
exception block 
finally block 
reture value of test() : 2
*/
```

- Java 虚拟机会把 finally 语句块作为 subroutine（对于这个 subroutine 不知该如何翻译为好，干脆就不翻译了，免得产生歧义和误解。）直接插入到 try 语句块或者 catch 语句块的控制转移语句之前。但是，还有另外一个不可忽视的因素，那就是在执行 subroutine（也就是 finally 语句块）之前，try 或者 catch 语句块会保留其返回值到本地变量表（Local Variable Table）中。待 subroutine 执行完毕之后，再恢复保留的返回值到操作数栈中，然后通过 return 或者 throw 语句将其返回给该方法的调用者（invoker）。请注意，前文中我们曾经提到过 return、throw 和 break、continue 的区别，对于这条规则（保留返回值），只适用于 return 和 throw 语句，不适用于 break 和 continue 语句，因为它们根本就没有返回值。

#### 8.return在try{}catch{}finally{}中执行具有哪些规则？

**至少有两种情况下finally语句是不会被执行的：**

**（1）try语句没有被执行到，如在try语句之前就返回了，这样finally语句就不会执行，这也说明了finally语句被执行的必要而非充分条件是：相应的try语句一定被执行到。**

**（2）在try块中有System.exit(0);这样的语句，System.exit(0);是终止Java虚拟机JVM的，连JVM都停止了，所有都结束了，当然finally语句也不会被执行到。**

- finally语句在return语句执行之后return返回之前执行的。

-  finally块中的return语句会覆盖try块中的return返回。

  ```java
  public class FinallyTest2 {
  
      public static void main(String[] args) {
  
          System.out.println(test2());
      }
  
      public static int test2() {
          int b = 20;
  
          try {
              System.out.println("try block");
  
              return b += 80;
          } catch (Exception e) {
  
              System.out.println("catch block");
          } finally {
  
              System.out.println("finally block");
  
              if (b > 25) {
                  System.out.println("b>25, b = " + b);
              }
  
              return 200;
          }
  
          // return b;
      }
  
  }
  /*
  try block
  finally block
  b>25, b = 100
  200
  */
  ```

  这说明finally里的return直接返回了，就不管try中是否还有返回语句，这里还有个小细节需要注意，finally里加上return过后，finally外面的return b就变成不可到达语句了，也就是永远不能被执行到，所以需要注释掉否则编译器报错。

-  如果finally语句中没有return语句覆盖返回值，那么原来的返回值可能因为finally里的修改而改变也可能不变。

  ```java
  public class FinallyTest3 {
  
      public static void main(String[] args) {
  
          System.out.println(test3());
      }
  
      public static int test3() {
          int b = 20;
  
          try {
              System.out.println("try block");
  
              return b += 80;
          } catch (Exception e) {
  
              System.out.println("catch block");
          } finally {
  
              System.out.println("finally block");
  
              if (b > 25) {
                  System.out.println("b>25, b = " + b);
              }
  
              b = 150;
          }
  
          return 2000;
      }
  
  }
  /*
  try block
  finally block
  b>25, b = 100
  100
  */
  
  import java.util.*;
  
  public class FinallyTest6
  {
      public static void main(String[] args) {
          System.out.println(getMap().get("KEY").toString());
      }
       
      public static Map<String, String> getMap() {
          Map<String, String> map = new HashMap<String, String>();
          map.put("KEY", "INIT");
           
          try {
              map.put("KEY", "TRY");
              return map;
          }
          catch (Exception e) {
              map.put("KEY", "CATCH");
          }
          finally {
              map.put("KEY", "FINALLY");
              map = null;
          }
           
          return map;
      }
  }
  /*
  FINALLY
  */
  ```

  为什么测试用例1中finally里的b = 150;并没有起到作用而测试用例2中finally的map.put("KEY", "FINALLY");起了作用而map = null;却没起作用呢？这就是Java到底是传值还是传址的问题了，具体请看[精选30道Java笔试题解答](http://www.cnblogs.com/lanxuezaipiao/p/3371224.html)，里面有详细的解答，简单来说就是：Java中只有传值没有传址，这也是为什么map = null这句不起作用。这同时也说明了返回语句是try中的return语句而不是 finally外面的return b;这句，不相信的话可以试下，将return b;改为return 294，对原来的结果没有一点影响。

  - try块里的return语句在异常的情况下不会被执行，这样具体返回哪个看情况。

    ```java
    public class FinallyTest4 {
    
        public static void main(String[] args) {
    
            System.out.println(test4());
        }
    
        public static int test4() {
            int b = 20;
    
            try {
                System.out.println("try block");
    
                b = b / 0;
    
                return b += 80;
            } catch (Exception e) {
    
                b += 15;
                System.out.println("catch block");
            } finally {
    
                System.out.println("finally block");
    
                if (b > 25) {
                    System.out.println("b>25, b = " + b);
                }
    
                b += 50;
            }
    	`	//返回204
           // return 204;
        }
    
    }
    /*
    try block
    catch block
    finally block
    b>25, b = 35
    85
    */
    ```

    

  - 当发生异常后，catch中的return执行情况与未发生异常时try中return的执行情况完全一样。

    ```java
    public class FinallyTest5 {
    
        public static void main(String[] args) {
    
            System.out.println(test5());
        }
    
        public static int test5() {
            int b = 20;
    
            try {
                System.out.println("try block");
                
                b = b /0;
    
                return b += 80;
            } catch (Exception e) {
    
                System.out.println("catch block");
                return b += 15;
            } finally {
    
                System.out.println("finally block");
    
                if (b > 25) {
                    System.out.println("b>25, b = " + b);
                }
    
                b += 50;
            }
    
            //return b;
        }
    
    }
    /*
    try block
    catch block
    finally block
    b>25, b = 35
    35
    */
    ```

    说明了发生异常后，catch中的return语句先执行，确定了返回值后再去执行finally块，执行完了catch再返回，finally里对b的改变对返回值无影响，原因同前面一样，也就是说情况与try中的return语句执行完全一样。

    

    **最后总结：finally块的语句在try或catch中的return语句执行之后返回之前执行且finally里的修改语句可能影响也可能不影响try或catch中 return已经确定的返回值，若finally里也有return语句则覆盖try或catch中的return语句直接返回。**

#### 9.给我例举至少5个常见的运行时异常。

常见的RuntimeException：

```java


ArithmeticException——由于除数为0引起的异常； 
ArrayStoreException——由于数组存储空间不够引起的异常； 
ClassCastException—一当把一个对象归为某个类，但实际上此对象并不是由这个类 创建的，也不是其子类创建的，则会引起异常； 
IllegalMonitorStateException——监控器状态出错引起的异常； 
NegativeArraySizeException—一数组长度是负数，则产生异常； 
NullPointerException—一程序试图访问一个空的数组中的元素或访问空的对象中的 方法或变量时产生异常； 
OutofMemoryException——用new语句创建对象时，如系统无法为其分配内存空 间则产生异常； 
SecurityException——由于访问了不应访问的指针，使安全性出问题而引起异常； 
IndexOutOfBoundsExcention——由于数组下标越界或字符串访问越界引起异常； 
IOException——由于文件未找到、未打开或者I/O操作不能进行而引起异常； 
ClassNotFoundException——未找到指定名字的类或接口引起异常； 
CloneNotSupportedException——一程序中的一个对象引用Object类的clone方法，但 此对象并没有连接Cloneable接口，从而引起异常； 
InterruptedException—一当一个线程处于等待状态时，另一个线程中断此线程，从 而引起异常，有关线程的内容，将在下一章讲述； 
NoSuchMethodException一所调用的方法未找到，引起异常； 
Illega1AccessExcePtion—一试图访问一个非public方法； 
StringIndexOutOfBoundsException——访问字符串序号越界，引起异常； 
ArrayIdexOutOfBoundsException—一访问数组元素下标越界，引起异常； 
NumberFormatException——字符的UTF代码数据格式有错引起异常； 
IllegalThreadException—一线程调用某个方法而所处状态不适当，引起异常； 
FileNotFoundException——未找到指定文件引起异常； 
EOFException——未完成输入操作即遇文件结束引起异常。
```



## 1.5 NIO/BIO/AIO

#### 1.NIO是什么 & BIO是什么 & AIO是什么 & 它们之间的区别？



#### 2.IO按照方向和数据类型划分能划分为哪些数据流？



#### 3.能给我说说NIO有什么特点？平常开发中使用过吗？

## 1.6 集合(容器)

#### 1.说说Java中集合的框架？

##### 概述：

在Java 2之前，Java是没有完整的集合框架的。它只有一些简单的可以自扩展的容器类，比如Vector，Stack，Hashtable等。这些容器类在使用的过程中由于效率问题饱受诟病，因此在Java 2中，Java设计者们进行了大刀阔斧的整改，重新设计，于是就有了现在的集合框架。需要注意的是，之前的那些容器类库并没有被弃用而是进行了保留，主要是为了向下兼容的目的，但我们在平时使用中还是应该尽量少用。





 集合可以看作是一种容器，用来存储对象信息。所有集合类都位于java.util包下，但支持多线程的集合类位于**java.util.concurrent**包下。 



**数组与集合的区别如下**：

1）数组长度不可变化而且无法保存具有映射关系的数据；集合类用于保存数量不确定的数据，以及保存具有映射关系的数据。

2）数组元素既可以是基本类型的值，也可以是对象；集合只能保存对象。

Java集合类主要由两个根接口**Collection**和**Map**派生出来的，Collection派生出了三个子接口：List、Set、Queue（Java5新增的队列），因此Java集合大致也可分成List、Set、Queue、Map四种接口体系，（**注意：Map不是Collection的子接口**）。

Java集合框架图如下：



![img](https://user-gold-cdn.xitu.io/2019/10/14/16dc8ee74157e0ab?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)





![img](https://user-gold-cdn.xitu.io/2019/10/14/16dc8ee741612be7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



其中List代表了有序可重复集合，可直接根据元素的索引来访问；Set代表无序不可重复集合，只能根据元素本身来访问；Queue是队列集合；Map代表的是存储key-value对的集合，可根据元素的key来访问value。

  上图中淡绿色背景覆盖的是集合体系中常用的实现类，分别是ArrayList、LinkedList、ArrayQueue、HashSet、TreeSet、HashMap、TreeMap等实现类。


 https://juejin.im/post/5da41417f265da5ba532b21c 



## 1.14 高级Java知识点

1.AOP是什么 & 和OOP区别？实现的方式有哪些？Android中如何实现？

2.APT是什么？例举一些基于它实现的轮子 & 自己有玩过它吗 & 做了些什么？

3.字节码篡改技术了解吗？

## 1.15 其它Java部分有关面试题

1.为什么局部内部类访问局部变量需要final?(校招&实习)

2.String、StringBuffer、StringBuilder、CharSequence的区别。(校招&实习)

3.equals和==的区别？(校招&实习)

4.关于字符串的拼接你在项目中常常怎么操作的？为什么不能用“+”的方式进行拼接呢？(校招&实习)

5.什么是Callback,讲讲你项目中使用的一些有关Callback的栗子。(校招&实习)

6.retrun & break & continue 区别？(校招&实习)

7.如何判断一个字符串是回文字符串？(校招&实习)

8.final,finally,finalize的区别？(校招&实习)

9.什么是动态代理 & 什么是静态代理？

10.String为什么会加final？

11.OOM可以try{}catch{}吗？

12.给我谈谈正则表达式。(校招&实习)

13.如何将String转成int?(校招&实习)

14.谈谈你对String的理解。

15.你如何理解序列化？有哪些方式序列化？

16.谈谈你对依赖注入的理解。

17.给我谈谈你对分派的理解

# 数据结构及算法

## 数据结构

栈和队列

数组和链表，自定义一个动态数组

Hash表，及Hash冲突的解决

二叉树

B+ B-树

基础排序算法：重点 快排、归并排序、堆排序（大根堆、小根堆）

快排的优化

二分查找与变种二分查找

哈夫曼树、红黑树

字符串操作，字符串查找，KMP算法

图的BFS、DFS、prim、Dijkstra算法（高阶技能）

经典问题：海量数据的处理 （10亿个数中找出最大的10000个数 TOP K问题）

## 算法

分治算法

动态规划

贪心算法

分支限界法



