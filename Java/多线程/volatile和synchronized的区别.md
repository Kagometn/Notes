# volatile和synchronized的区别

#### volatile和synchronized特点

首先需要理解**线程安全的两个方面：执行控制和内存可见。**

**执行控制的目的是控制代码执行（顺序）及是否可以并发执行。**

**内存可见控制的是线程执行结果在内存中对其它线程的可见性。**根据Java内存模型的实现，线程在具体执行时，**会先拷贝主存数据到线程本地（CPU缓存），操作完成后再把结果从线程本地刷到主存**。

**synchronized关键字解决的是执行控制的问题，它会阻止其它线程获取当前对象的监控锁，这样就使得当前对象中被synchronized关键字保护的代码块无法被其它线程访问，也就无法并发执行**。更重要的是，**synchronized还会创建一个内存屏障，内存屏障指令保证了所有CPU操作结果都会直接刷到主存中，从而保证了操作的内存可见性，同时也使得先获得这个锁的线程的所有操作，都happens-before于随后获得这个锁的线程的操作。**

**volatile关键字解决的是内存可见性的问题，会使得所有对volatile变量的读写都会直接刷到主存，即保证了变量的可见性。这样就能满足一些对变量可见性有要求而对读取顺序没有要求的需求。**

**使用volatile关键字仅能实现对原始变量(如boolen、 short 、int 、long等)操作的原子性，但需要特别注意， volatile不能保证复合操作的原子性，即使只是i++，实际上也是由多个原子操作组成：read i; inc; write i，假如多个线程同时执行i++，volatile只能保证他们操作的i是同一块内存，但依然可能出现写入脏数据的情况。**

在Java 5提供了原子数据类型atomic wrapper classes，对它们的increase之类的操作都是原子操作，不需要使用sychronized关键字。

**对于volatile关键字，当且仅当满足以下所有条件时可使用：**

1. **对变量的写入操作不依赖变量的当前值，或者你能确保只有单个线程更新变量的值。**
2. **该变量没有包含在具有其他变量的不变式中。**




#### volatile和synchronized的区别

- **volatile本质是在告诉jvm当前变量在寄存器（工作内存）中的值是不确定的，需要从主存中读取；synchronized则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住。**
- **volatile仅能使用在变量级别；synchronized则可以使用在变量、方法、和类级别的**
- **volatile仅能实现变量的修改可见性，不能保证原子性；而synchronized则可以保证变量的修改可见性和原子性**
- **volatile不会造成线程的阻塞；synchronized可能会造成线程的阻塞。**
- **volatile标记的变量不会被编译器优化；synchronized标记的变量可以被编译器优化**





##### volatile修饰的变量具有可见性

**volatile是变量修饰符，其修饰的变量具有可见性。**

可见性也就是说一旦某个线程修改了该被volatile修饰的变量，它会保证修改的值会立即被更新到主存，当有其他线程需要读取时，可以立即获取修改之后的值。

在Java中为了加快程序的运行效率，对一些变量的操作通常是在该线程的寄存器或是CPU缓存上进行的，之后才会同步到主存中，而加了volatile修饰符的变量则是直接读写主存。

例子请查看下面的3.1，帮助理解。



##### volatile禁止指令重排 

volatile可以禁止进行指令重排。

**指令重排是指处理器为了提高程序运行效率，可能会对输入代码进行优化，它不保证各个语句的执行顺序同代码中的顺序一致，但是它会保证程序最终执行结果和代码顺序执行的结果是一致的。指令重排序不会影响单个线程的执行，但是会影响到线程并发执行的正确性。**

程序执行到volatile修饰变量的读操作或者写操作时，在其前面的操作肯定已经完成，且结果已经对后面的操作可见，在其后面的操作肯定还没有进行。

例子请查看下面3.2，帮助理解。



3. ##### synchronized 

**synchronized可作用于一段代码或方法，既可以保证可见性，又能够保证原子性。**

可见性体现在：**通过synchronized或者Lock能保证同一时刻只有一个线程获取锁然后执行同步代码，并且在释放锁之前会将对变量的修改刷新到主存中。**

**原子性表现在：要么不执行，要么执行到底。**

例子请查看下面3.3，帮助理解。



#### 总结

（1）从而我们可以看出volatile虽然具有可见性但是并不能保证原子性。

（2）**性能方面，synchronized关键字是防止多个线程同时执行一段代码，就会影响程序执行效率，而volatile关键字在某些情况下性能要优于synchronized。**

但是要注意volatile关键字是无法替代synchronized关键字的，因为volatile关键字无法保证操作的原子性。



## 总结

> 1.volatile仅能使用在变量级别；  synchronized则可以使用在变量、方法、和类级别的
>
> 2.volatile仅能实现变量的修改可见性，并不能保证原子性；synchronized则可以保证变量的修改可见性和原子性
>
> 3.volatile不会造成线程的阻塞； synchronized可能会造成线程的阻塞。
>
> 4.volatile标记的变量不会被编译器优化；synchronized标记的变量可以被编译器优化



### volatile与synchronized的使用场景举例（结合第1部分进行理解学习）

##### 3.1 volatile的使用举例

```java
class MyThread extends Thread {           
    private volatile boolean isStop = false;        
    public void run() {    
        while (!isStop) {    
            System.out.println("do something");    
        }    
    }    
    public void setStop() {    
        isStop = true;    
    }          
}  
```



线程执行run()的时候我们需要在线程中不停的做一些事情，比如while循环，那么这时候该如何停止线程呢？如果线程做的事情不是耗时的，那么只需要使用一个标志即可。如果需要退出时，调用setStop()即可。这里就使用了关键字volatile，这个关键字的目的是如果修改了isStop的值，那么在while循环中可以立即读取到修改后的值。

如果线程做的事情是耗时的，那么可以使用interrupt方法终止线程 。如果在子线程“睡觉”时被interrupt，那么子线程可以catch到InterruptExpection异常，处理异常后继续往下执行。

 

##### 3.2 volatile的使用举例

```java
//线程1:
context = loadContext();   //语句1  context初始化操作
inited = true;             //语句2

//线程2:
while(!inited ){
  sleep()
}
doSomethingwithconfig(context);
```



因为指令重排序，有可能语句2会在语句1之前执行，可能导致context还没被初始化，而线程2中就使用未初始化的context去进行操作，导致程序出错。

这里如果用volatile关键字对inited变量进行修饰，就不会出现这种问题了。

 

##### 3.3 必须使用synchronized而不能使用volatile的场景

​     
```java
public class Test {
    public volatile int inc = 0;
    public void increase() {
        inc++;
    }
    
    public static void main(String[] args) {
        final Test test = new Test();
        for(int i=0;i<10;i++){
            new Thread(){
                public void run() {
                    for(int j=0;j<1000;j++)
                        test.increase();
                };
            }.start();
        }

        while(Thread.activeCount()>1)  //保证前面的线程都执行完
            Thread.yield();
        System.out.println(test.inc);
    }
}
```

例子中用new了10个线程，分别去调用1000次increase()方法，每次运行结果都不一致，都是一个小于10000的数字。自增操作不是原子操作，volatile 是不能保证原子性的。回到文章一开始的例子，使用volatile修饰int型变量i，多个线程同时进行i++操作。比如有两个线程A和B对volatile修饰的i进行i++操作，i的初始值是0，A线程执行i++时刚读取了i的值0，就切换到B线程了，B线程（从内存中）读取i的值也为0，然后就切换到A线程继续执行i++操作，完成后i就为1了，接着切换到B线程，因为之前已经读取过了，所以继续执行i++操作，最后的结果i就为1了。同理可以解释为什么每次运行结果都是小于10000的数字。
但是使用synchronized对部分代码进行如下修改，就能保证同一时刻只有一个线程获取锁然后执行同步代码。运行结果必然是10000。

```java
public  int inc = 0;
public synchronized void increase() {
        inc++;
}
```

https://blog.csdn.net/suifeng3051/article/details/52611310