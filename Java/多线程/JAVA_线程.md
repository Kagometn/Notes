### 1.9 线程

#### 1.什么是线程？能解决什么问题。

**一** **基本概念**

一个程序就是一个进程，而一个程序中的多个任务就是线程。

进程就是表示资源分配的基本单位，而线程是进程中执行运算的最小的单位，亦或是最小的调度单位

举个例子：

打开你的计算机上的任务管理器，会显示出当前机器的所有进程，QQ，360 等，当 QQ 运行时，就有很多子任务在同时运行。比如，当你边打字发送表情，边好友视频时这些不同的功能都可以同时运行，其中每一项任务都可以理解成“线程”在工作。



Java编写程序都运行在在Java虚拟机（JVM）中，在JVM的内部，程序的多任务是通过线程来实现的。每用java命令启动一个java应用程序，就会启动一个JVM进程。在同一个JVM进程中，有且只有一个进程，就是它自己。在这个JVM环境中，所有程序代码的运行都是以线程来运行。

　　一般常见的方法，这样就产生了一个线程，这个线程称之为主线程。当main方法结束后，主线程运行完成。JVM进程也随即退出。

　　对于一个进程中的多个线程来说，多个线程共享进程的内存块，当有新的线程产生的时候，操作系统不分配新的内存，而是让新线程共享原有的进程块的内存。因此，线程间的通信很容易，速度也很快。不同的进程因为处于不同的内存块，因此进程之间的通信相对困难。

　　进程是指一个内存中运行的应用程序，**每个进程都有自己独立的一块内存空间，一个进程中可以启动多个线程**。比如在Windows系统中，一个运行的exe就是一个进程。

　　线程是指进程中的一个执行流程，一个进程可以运行多个线程。比如java.exe进程可以运行很多线程。**线程总是输入某个进程，进程中的多个线程共享进程的内存。**

Java中线程是指java.lang.Thread类的一个实例或线程的执行。使用java.lang.Thread或java.lang.Runnable接口编写代码定义、实例化、启动新线程。

方法运行在一个线程内，称为主线程。一旦创建一个新的线程，就产生一个新的调用栈。

　　线程分为两类：**用户线程和守候线程**。当所有用户线程执行完毕后，JVM自动关闭。但是守候线程却不独立与JVM，守候线程一般是有操作系统或用户自己创建的。



#### 1.1 线程的调度

**1、调整线程优先级**：Java线程有优先级，优先级高的线程会获得较多的运行机会。

Java线程的优先级用整数表示，取值范围是1~10，Thread类有以下三个静态常量：
static int MAX_PRIORITY
          线程可以具有的最高优先级，取值为10。
static int MIN_PRIORITY
          线程可以具有的最低优先级，取值为1。
static int NORM_PRIORITY
          分配给线程的默认优先级，取值为5。

Thread类的setPriority()和getPriority()方法分别用来设置和获取线程的优先级。
 每个线程都有默认的优先级。主线程的默认优先级为Thread.NORM_PRIORITY。
线程的优先级有继承关系，比如A线程中创建了B线程，那么B将和A具有相同的优先级。
JVM提供了10个线程优先级，但与常见的操作系统都不能很好的映射。如果希望程序能移植到各个操作系统中，应该仅仅使用Thread类有以下三个静态常量作为优先级，这样能保证同样的优先级采用了同样的调度方式。

**2、线程睡眠**：**Thread.sleep(long millis)方法**，使线程转到阻塞状态。millis参数设定睡眠的时间，以毫秒为单位。当睡眠结束后，就转为就绪（Runnable）状态。sleep()平台移植性好。

**3、线程等待**：**Object类中的wait()方法**，导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 唤醒方法。这个两个唤醒方法也是Object类中的方法，行为等价于调用 wait(0) 一样。

**4、线程让步**：**Thread.yield()** 方法，暂停当前正在执行的线程对象，把执行机会让给相同或者更高优先级的线程。

**5、线程加入：join()方法**，等待其他线程终止。在当前线程中调用另一个线程的join()方法，则当前线程转入阻塞状态，直到另一个进程运行结束，当前线程再由阻塞转为就绪状态。

**6、线程唤醒**：**Object类中的notify()方法**，唤醒在此对象监视器上等待的单个线程。如果所有线程都在此对象上等待，则会选择唤醒其中一个线程。选择是任意性的，并在对实现做出决定时发生。线程通过调用其中一个 wait 方法，在对象的监视器上等待。 直到当前的线程放弃此对象上的锁定，才能继续执行被唤醒的线程。被唤醒的线程将以常规方式与在该对象上主动同步的其他所有线程进行竞争；例如，唤醒的线程在作为锁定此对象的下一个线程方面没有可靠的特权或劣势。类似的方法还有一个notifyAll()，唤醒在此对象监视器上等待的所有线程。
 注意：Thread中suspend()和resume()两个方法在JDK1.5中已经废除，不再介绍。因为有死锁倾向。

#### 1.2常用函数说明：sleep,join,yield,setPriority,interrupt,wait

 **①sleep(long millis): 在指定的毫秒数内让当前正在执行的线程休眠（暂停执行）** 

注意：

**在使用sleep方法时有两点需要注意：**

\1. sleep方法有两个重载形式，其中一个重载形式不仅可以设毫秒，而且还可以设纳秒(1,000,000纳秒等于1毫秒)。但大多数操作系统平台上的Java虚拟机都无法精确到纳秒，因此，如果对sleep设置了纳秒，Java虚拟机将取最接近这个值的毫秒。

\2. 在使用sleep方法时必须使用throws或try{…}catch{…}。因为run方法无法使用throws，所以只能使用try{…}catch{…}。当在线程休眠的过程中，使用interrupt方法中断线程时sleep会抛出一个InterruptedException异常。

**②join():指等待t线程终止。**
使用方式。
join是Thread类的一个方法，启动线程后直接调用，即join()的作用是：“等待该线程终止”，这里需要理解的就是该线程是指的主线程等待子线程的终止。也就是在子线程调用了join()方法后面的代码，只有等到子线程结束了才能执行。

Thread t = new AThread(); t.start(); t.join();
为什么要用join()方法
在很多情况下，主线程生成并起动了子线程，如果子线程里要进行大量的耗时的运算，主线程往往将于子线程之前结束，但是如果主线程处理完其他的事务后，需要用到子线程的处理结果，也就是主线程需要等待子线程执行完成之后再结束，这个时候就要用到join()方法了。

**③yield:暂停当前正在执行的线程对象，并执行其他线程。**
        Thread.yield()方法作用是：暂停当前正在执行的线程对象，并执行其他线程。
         yield()应该做的是让当前运行线程回到可运行状态，以允许具有相同优先级的其他线程获得运行机会。因此，使用yield()的目的是让相同优先级的线程之间能适当的轮转执行。但是，实际中无法保证yield()达到让步目的，因为让步的线程还有可能被线程调度程序再次选中。

结论：yield()从未导致线程转到等待/睡眠/阻塞状态。在大多数情况下，yield()将导致线程从运行状态转到可运行状态，但有可能没有效果。可看上面的图。
**sleep()和yield()的区别**
        sleep()和yield()的区别):sleep()使当前线程进入停滞状态，所以执行sleep()的线程在指定的时间内肯定不会被执行；yield()只是使当前线程重新回到可执行状态，所以执行yield()的线程有可能在进入到可执行状态后马上又被执行。
        sleep 方法使当前运行中的线程睡眼一段时间，进入不可运行状态，这段时间的长短是由程序设定的，yield 方法使当前线程让出 CPU 占有权，但让出的时间是不可设定的。实际上，yield()方法对应了如下操作：先检测当前是否有相同优先级的线程处于同可运行状态，如有，则把 CPU  的占有权交给此线程，否则，继续运行原来的线程。所以yield()方法称为“退让”，它把运行机会让给了同等优先级的其他线程
       另外，sleep 方法允许较低优先级的线程获得运行机会，但 yield()  方法执行时，当前线程仍处在可运行状态，所以，不可能让出较低优先级的线程些时获得 CPU 占有权。在一个运行系统中，如果较高优先级的线程没有调用 sleep 方法，又没有受到 I\O 阻塞，那么，较低优先级线程只能等待所有较高优先级的线程运行结束，才有机会运行。

**④setPriority()**: 更改线程的优先级。
　　　　MIN_PRIORITY = 1
  　　   NORM_PRIORITY = 5
           MAX_PRIORITY = 10

用法：
Thread4 t1 = new Thread4("t1");
Thread4 t2 = new Thread4("t2");
t1.setPriority(Thread.MAX_PRIORITY);
t2.setPriority(Thread.MIN_PRIORITY);



**⑤interrupt():**不要以为它是中断某个线程！它只是线线程发送一个中断信号，让线程在无限等待时（如死锁时）能抛出抛出，从而结束线程，但是如果你吃掉了这个异常，那么这个线程还是不会中断的！
**⑥wait()**

Obj.wait()，与Obj.notify()必须要与synchronized(Obj)一起使用，也就是wait,与notify是针对已经获取了Obj锁进行操作，从语法角度来说就是Obj.wait(),Obj.notify必须在synchronized(Obj){...}语句块内。从功能上来说wait就是说线程在获取对象锁后，主动释放对象锁，同时本线程休眠。直到有其它线程调用对象的notify()唤醒该线程，才能继续获取对象锁，并继续执行。相应的notify()就是对对象锁的唤醒操作。但有一点需要注意的是notify()调用后，并不是马上就释放对象锁的，而是在相应的synchronized(){}语句块执行结束，自动释放锁后，JVM会在wait()对象锁的线程中随机选取一线程，赋予其对象锁，唤醒线程，继续执行。这样就提供了在线程间同步、唤醒的操作。Thread.sleep()与Object.wait()二者都可以暂停当前线程，释放CPU控制权，主要的区别在于Object.wait()在释放CPU同时，释放了对象锁的控制。



⑤interrupt():不要以为它是中断某个线程！它只是线线程发送一个中断信号，让线程在无限等待时（如死锁时）能抛出抛出，从而结束线程，但是如果你吃掉了这个异常，那么这个线程还是不会中断的！
⑥wait()

Obj.wait()，与Obj.notify()必须要与synchronized(Obj)一起使用，也就是wait,与notify是针对已经获取了Obj锁进行操作，从语法角度来说就是Obj.wait(),Obj.notify必须在synchronized(Obj){...}语句块内。从功能上来说wait就是说线程在获取对象锁后，主动释放对象锁，同时本线程休眠。直到有其它线程调用对象的notify()唤醒该线程，才能继续获取对象锁，并继续执行。相应的notify()就是对对象锁的唤醒操作。但有一点需要注意的是notify()调用后，并不是马上就释放对象锁的，而是在相应的synchronized(){}语句块执行结束，自动释放锁后，JVM会在wait()对象锁的线程中随机选取一线程，赋予其对象锁，唤醒线程，继续执行。这样就提供了在线程间同步、唤醒的操作。Thread.sleep()与Object.wait()二者都可以暂停当前线程，释放CPU控制权，主要的区别在于Object.wait()在释放CPU同时，释放了对象锁的控制。



```JAVA
public class MyThreadPrinter2 implements Runnable {   
	  
    private String name;   
    private Object prev;   
    private Object self;   
  
    private MyThreadPrinter2(String name, Object prev, Object self) {   
        this.name = name;   
        this.prev = prev;   
        this.self = self;   
    }   
  
    @Override  
    public void run() {   
        int count = 10;   
        while (count > 0) {   
            synchronized (prev) {   
                synchronized (self) {   
                    System.out.print(name);   
                    count--;  
                    
                    self.notify();   
                }   
                try {   
                    prev.wait();   
                } catch (InterruptedException e) {   
                    e.printStackTrace();   
                }   
            }   
  
        }   
    }   
  
    public static void main(String[] args) throws Exception {   
        Object a = new Object();   
        Object b = new Object();   
        Object c = new Object();   
        MyThreadPrinter2 pa = new MyThreadPrinter2("A", c, a);   
        MyThreadPrinter2 pb = new MyThreadPrinter2("B", a, b);   
        MyThreadPrinter2 pc = new MyThreadPrinter2("C", b, c);   
           
           
        new Thread(pa).start();
        Thread.sleep(100);  //确保按顺序A、B、C执行
        new Thread(pb).start();
        Thread.sleep(100);  
        new Thread(pc).start();   
        Thread.sleep(100);  
        }   
}  

/*
输出结果：
ABCABCABCABCABCABCABCABCABCABC

     先来解释一下其整体思路，从大的方向上来讲，该问题为三线程间的同步唤醒操作，主要的目的就是ThreadA->ThreadB->ThreadC->ThreadA循环执行三个线程。为了控制线程执行的顺序，那么就必须要确定唤醒、等待的顺序，所以每一个线程必须同时持有两个对象锁，才能继续执行。一个对象锁是prev，就是前一个线程所持有的对象锁。还有一个就是自身对象锁。主要的思想就是，为了控制执行的顺序，必须要先持有prev锁，也就前一个线程要释放自身对象锁，再去申请自身对象锁，两者兼备时打印，之后首先调用self.notify()释放自身对象锁，唤醒下一个等待线程，再调用prev.wait()释放prev对象锁，终止当前线程，等待循环结束后再次被唤醒。运行上述代码，可以发现三个线程循环打印ABC，共10次。程序运行的主要过程就是A线程最先运行，持有C,A对象锁，后释放A,C锁，唤醒B。线程B等待A锁，再申请B锁，后打印B，再释放B，A锁，唤醒C，线程C等待B锁，再申请C锁，后打印C，再释放C,B锁，唤醒A。看起来似乎没什么问题，但如果你仔细想一下，就会发现有问题，就是初始条件，三个线程按照A,B,C的顺序来启动，按照前面的思考，A唤醒B，B唤醒C，C再唤醒A。但是这种假设依赖于JVM中线程调度、执行的顺序。
*/
```



线程类的一些常用方法： 

　　sleep(): 强迫一个线程睡眠Ｎ毫秒。 
　　isAlive(): 判断一个线程是否存活。 
　　join(): 等待线程终止。 
　　activeCount(): 程序中活跃的线程数。 
　　enumerate(): 枚举程序中的线程。 
    currentThread(): 得到当前线程。 
　　isDaemon(): 一个线程是否为守护线程。 
　　setDaemon(): 设置一个线程为守护线程。(用户线程和守护线程的区别在于，是否等待主线程依赖于主线程结束而结束) 
　　setName(): 为线程设置一个名称。 
　　wait(): 强迫一个线程等待。 
　　notify(): 通知一个线程继续运行。 
　　setPriority(): 设置一个线程的优先级。





#### 2.Java中创建线程的2种方式 & 区别？

Java中实现创建多线程的方式有两种，一个是继承Thread，另一个是实现Runnable接口。继承Thread类的最大实现就是只能单继承，所以为了实现多继承就可以实现Runnable接口。需要说明的是，这两种方式在工作时的性质都是一样的，没有本质的区别：



###### 继承Thread类创建线程

- 定义Thread类的子类，并重写该类的run方法，该run方法的方法体就代表了线程要完成的任务。因此把run()方法称为执行体

- 创建Thread子类的实例，即创建线程对象

- 调用线程对象的start()方法来启动线程

  ```java
  public class FirstThreadTest extends Thread{
  	int i = 0;
  	//重写run方法，run方法的方法体就是现场执行体
  	public void run()
  	{
  		for(;i<100;i++){
  		System.out.println(getName()+"  "+i);
  		
  		}
  	}
  	public static void main(String[] args)
  	{
  		for(int i = 0;i< 100;i++)
  		{
  			System.out.println(Thread.currentThread().getName()+"  : "+i);
  			if(i==20)
  			{
  				new FirstThreadTest().start();
  				new FirstThreadTest().start();
  			}
  		}
  	}
   
  }
  ```

  上述代码中，Thread.currentThread()方法返回当前正在执行的线程对象，GetName()方法返回调用该方法的线程名字

  

  常见的Thread方法：

  ![image-20200207201518016](C:\Users\Lenovo\Desktop\image-20200207201518016.png)

  

  测试线程是否处于活动状态，上述方法是被Thread对象调用的，而下面是Thead的静态方法：

  ![image-20200207201703498](C:\Users\Lenovo\Desktop\image-20200207201703498.png)

  ###### 二、通过Runnable接口创建线程类

  （1）定义runnable接口的实现类，并重写该接口的run()方法，该run()方法的方法体同样是该线程的线程执行体。

  （2）创建 Runnable实现类的实例，并依此实例作为Thread的target来创建Thread对象，该Thread对象才是真正的线程对象。

  （3）调用线程对象的start()方法来启动该线程。

  示例代码为：

  ```java
  public class RunnableThreadTest implements Runnable{
  	private int i;
  	public void run(){
  		for(i = 0;i <100;i++){
  			System.out.println(Thread.currentThread().getName()+" "+i);
  		}
  	}
  	public static void main(String[] args){
  		for(int i = 0;i < 100;i++){
  			System.out.println(Thread.currentThread().getName()+" "+i);
  			if(i==20){
  				RunnableThreadTest rtt = new RunnableThreadTest();
  				new Thread(rtt,"新线程1").start();
  				new Thread(rtt,"新线程2").start();
  			}
  		}
  	 
  	}
  }
  
  ```

  ###### 三、通过Callable和Future创建线程

  （1）创建Callable接口的实现类，并实现call()方法，该call()方法将作为线程执行体，并且有返回值。

  （2）创建Callable实现类的实例，使用FutureTask类来包装Callable对象，该FutureTask对象封装了该Callable对象的call()方法的返回值。

  （3）使用FutureTask对象作为Thread对象的target创建并启动新线程。

  （4）调用FutureTask对象的get()方法来获得子线程执行结束后的返回值

  实例代码：

  ```java
  import java.util.concurrent.Callable;
  import java.util.concurrent.ExecutionException;
  import java.util.concurrent.FutureTask;
  
  public class CallableThreadTest implements Callable<Integer>{
      public static void main(String[] args){
          CallableThreadTest ctt = new CallableThreadTest();
          FutureTask<Integer> ft = new FutureTask<>(ctt);
          for(int i = 0;i < 100;i++){
              System.out.println(Thread.currentThread().getName()+" 的循环变量i的值"+i);
              if(i==20){
                  new Thread(ft,"有返回值的线程").start();
              }
          }
          try{
              System.out.println("子线程的返回值："+ft.get());
          } catch (InterruptedException e){
              e.printStackTrace();
          } catch (ExecutionException e){
              e.printStackTrace();
          }
      }
  
      @Override
      public Integer call() throws Exception{
          int i = 0;
          for(;i<100;i++){
              System.out.println(Thread.currentThread().getName()+" "+i);
          }
          return i;
      }
  }
  ```

  ###### 二、创建线程的三种方式的对比

  采用实现Runnable、Callable接口的方式创见多线程时

  - 优势是

    - 线程类只是实现了Runnable接口或Callable接口，还可以继承其他类。

    - 在这种方式下，多个线程可以共享同一个target对象，所以**非常适合多个相同线程来处理同一份资源的情况，从而可以将CPU、代码和数据分开，形成清晰的模型，较好地体现了面向对象的思想**。

  - 劣势是：
    - 编程稍微复杂，如果要访问当前线程，则必须使用Thread.currentThread()方法。

  

  使用继承Thread类的方式创建多线程时优势是：

  - **编写简单，如果需要访问当前线程**，则无需使用Thread.currentThread()方法，直接使用this即可获得当前线程。
  - **劣势是：**线程类已经继承了Thread类，所以不能再继承其他父类。



#### 3.给我说说线程的生命周期。

线程在创建并启动以后，他既不是一启动就进入了执行状态，也不是一直处于执行状态。在线程的生命周期中，要经过**新建(new)**，**就绪(Runnable)**，**运行(Running)**，**阻塞(Blocked)**和**死亡(Dead)**5种状态。尤其是当线程启动以后，就不可以一直“霸占”着CPU独自运行，所以CPU需要在多条线程之间切换，于是线程状态也会多次在运行，阻塞之间切换



###### 3.1	Java 中的线程生命周期

- 1. 线程状态图：

     ![img](https://www.runoob.com/wp-content/uploads/2014/09/716271-20170320112245721-1831918220.jpg)

     线程共包括以下 5 种状态：

     - **新建 (New)** ：当程序使用 new 关键字创建了一个线程后。此时由 JVM 为其分配内存，并初始化其成员变量的值。此时的线程对象并没有表现出任何线程的动态特征，程序也不会执行线程的线程执行体例如，Thread thread = new Thread()。

     - **可运行/就绪状态(Runnable):** 也被称为“可执行状态”。线程对象被创建后，其它线程调用了该对象的start()方法，从而来启动该线程。Java虚拟机会为其创建方法调用栈可程序计数器，等到调度运行。至于该线程何时开始运行，取决于 JVM 里线程调度器的调度。  例如，thread.start()。处于就绪状态的线程，随时可能被CPU调度执行。

       > 注意：启动线程使用 start() 方法，而不是 run() 方法。调用 `start()`方法启动线程，系统会把该 run方法当成方法执行体处理。需要切记的是：调用了线程的 `run()`方法之后，该线程就不在处于新建状态，不要再次调用 `start()`方法，只能对新建状态的线程调用`start()`方法，否则会引发 IllegaIThreadStateExccption异常。

       

     - **运行状态(Running):** 线程获取CPU分片后，进行执行run方法。需要注意的是，线程只能从就绪状态进入到运行状态。

     - **阻塞状态(Blocked):** 阻塞状态是线程因为某种原因失去了CPU使用权，暂时停止运行。直到线程进入就绪状态，才有机会转到运行状态。阻塞的情况分三种：

       -  (01) 等待阻塞 -- 通过调用线程的wait()方法，让线程等待某工作的完成。
       -  (02) 同步阻塞 -- 线程在获取synchronized同步锁失败(因为锁被其它线程所占用)，它会进入同步阻塞状态。
       -  (03) 其他阻塞 -- 通过调用线程的sleep()或join()或发出了I/O请求时，线程会进入到阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入就绪状态。

     - **死亡状态(Dead):** 程序正常执行完毕之后或者因异常退出了run()方法，或者是线程抛出了一个未捕获的ExceptionhuoError，或者调用线程的中断方法，该线程结束生命周期。

**不要试图对一个已经死亡的线程调用  start() 方法让它重新启动，死亡就是死亡了，该线程不可再次作为线程执行。**

 ![状态转换](https://user-gold-cdn.xitu.io/2019/9/16/16d382b41e53bb4b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)**** 

- **RUNNABLE 与 BLOCKED 的状态转换**

线程等待 synchronized 的隐式锁，synchronized 修饰的方法、代码块同一时刻只允许一个线程执行，其他线程只能等待，一个茅坑多个人想拉屎，只允许一个人在里面其他人只能在门外等着。这时候 等待的线程就会从 RUNNABLE 转换到 BLOCKED 状态。当某个等待的线程获得 synchronized 隐式锁时，就会从 BLOCKED 转换到 RUNNABLE 状态。因为拿到去茅坑的小票了。

- RUNNABLE 与 WAITING 的状态转换

  一共有三种场景触发这种状态转换。

  - 获得 synchronized 锁的线程调用 `Object.wait()`方法。会释放锁，并且放弃 CPU 分片调度。
  - 调用 `Thread.join()`方法。假如一个线程对象 thread A, 调用 `A.join()`，执行这条语句的线程会等待  thread A 执行完，而作为等待的这个线程就会从 运行中变成 WAITING。比如主线程等待 thread A执行完成，那么主线程就会变成等待状态。
  - 调用 `LockSupport.park()`方法。Java 并发包的锁都是基于此实现，调用 `LockSupport.park()`方法，释放锁。当前线程会阻塞，线程的状态会从 RUNNABLE 转换到 WAITING。调用 `LockSupport.unpark(Thread thread)`可唤醒目标线程，目标线程竞争到锁后状态又会从 WAITING 状态转换到 RUNNABLE。

- RUNNABLE 与 TIMED_WAITING 的状态转换

  触发场景如下所示：

  - 调用带超时参数的 `Thread.sleep(long millis)`，并不会释放锁。

  - 获得 synchronized 隐式锁的线程，调用**带超时参数**的 `Object.wait(long timeout)` 方法；会释放锁。

  - 调用**带超时参数**的 `Thread.join(long millis)`方法；

  - 调用**带超时参数**的`LockSupport.parkNanos(Object blocker, long deadline)`方法；会释放锁。

  - 调用**带超时参数**的`LockSupport.parkUntil(long deadline)`方法。会释放锁。

    注：

    其实更多的都是多了超时参数。



- **从运行中到终止状态**

  线程执行完run()方法之后，会自动转换到终止状态，但是当运行run()方法异常的时候，也会导致线程终止，有时候我们需要中断 `run()`方法的执行，比如有的人占着茅坑很久不拉屎，我们要轰出去，后面的人等不了了，快憋死了。在 Java 中 Thread 类里面倒是有个 `stop()`方法，不过已经标记为 @Deprecated，所以不建议使用了。正确的姿势其实是调用 `interrupt()`方法。

**那 stop() 和 interrupt() 方法的主要区别是什么呢？**

stop() 方法会真的杀死线程，不给线程喘息的机会，如果线程持有 synchronized 隐式锁，也不会释放，那其他线程就再也没机会获得 synchronized 隐式锁，这实在是太危险了。所以该方法就不建议使用了，类似的方法还有 suspend() 和 resume() 方法，这两个方法同样也都不建议使用了，所以这里也就不多介绍了。

而 interrupt() 方法就温柔多了，interrupt() 方法仅仅是通知线程，线程有机会执行一些后续操作，同时也可以无视这个通知。被 interrupt 的线程，是怎么收到通知的呢？一种是异常，另一种是主动检测。



****

#### 4.线程死锁的原因 & 举个栗子 & 如何避免死锁。

 死锁问题是**多线程**特有的问题，它可以被认为是线程间切换消耗系统性能的一种极端情况。在死锁时，***\*线程间相互等待资源，而又不释放自身的资源，导致无穷无尽的等待\****，其结果是系统任务永远无法执行完成。死锁问题是在多线程开发中应该坚决避免和杜绝的问题。 

**二、死锁产生的原因**
**1) 系统资源的竞争**
通常系统中拥有的不可剥夺资源，其数量不足以满足多个进程运行的需要，使得进程在 运行过程中，会因争夺资源而陷入僵局，如磁带机、打印机等。只有对不可剥夺资源的竞争 才可能产生死锁，对可剥夺资源的竞争是不会引起死锁的。
**2) 进程推进顺序非法**
进程在运行过程中，请求和释放资源的顺序不当，也同样会导致死锁。例如，并发进程 P1、P2分别保持了资源R1、R2，而进程P1申请资源R2，进程P2申请资源R1时，两者都 会因为所需资源被占用而阻塞。

信号量使用不当也会造成死锁。进程间彼此相互等待对方发来的消息，结果也会使得这 些进程间无法继续向前推进。例如，进程A等待进程B发的消息，进程B又在等待进程A 发的消息，**可以看出进程A和B不是因为竞争同一资源，而是在等待对方的资源导致死锁。**
**3) 死锁产生的必要条件**
产生死锁必须同时满足以下四个条件，只要其中任一条件不成立，死锁就不会发生。
**互斥条件**：进程要求对所分配的资源（如打印机）进行排他性控制，即在一段时间内某 资源仅为一个进程所占有。此时若有其他进程请求该资源，则请求进程只能等待。
**不剥夺条件**：进程所获得的资源在未使用完毕之前，不能被其他进程强行夺走，即只能 由获得该资源的进程自己来释放（只能是主动释放)。
**请求和保持条件**：进程已经保持了至少一个资源，但又提出了新的资源请求，而该资源 已被其他进程占有，此时请求进程被阻塞，但对自己已获得的资源保持不放。
**循环等待条件**：存在一种进程资源的循环等待链，链中每一个进程已获得的资源同时被 链中下一个进程所请求。即存在一个处于等待状态的进程集合{Pl, P2, ..., pn}，其中Pi等 待的资源被P(i+1)占有（i=0, 1, ..., n-1)，Pn等待的资源被P0占有，如图2-15所示。

直观上看，循环等待条件似乎和死锁的定义一样，其实不然。按死锁定义构成等待环所 要求的条件更严，它要求Pi等待的资源必须由P(i+1)来满足，而循环等待条件则无此限制。 例如，系统中有两台输出设备，P0占有一台，PK占有另一台，且K不属于集合{0, 1, ..., n}。

Pn等待一台输出设备，它可以从P0获得，也可能从PK获得。因此，虽然Pn、P0和其他 一些进程形成了循环等待圈，但PK不在圈内，若PK释放了输出设备，则可打破循环等待, 如图2-16所示。因此循环等待只是死锁的必要条件。

 ![img](http://c.biancheng.net/cpp/uploads/allimg/140630/1-140630152010348.jpg) 

资源分配图含圈而系统又不一定有死锁的原因是同类资源数大于1。但若系统中每类资 源都只有一个资源，则资源分配图含圈就变成了系统出现死锁的充分必要条件。

**三、如何避免死锁**
在有些情况下死锁是可以避免的。三种用于避免死锁的技术：

**加锁顺序（线程按照一定的顺序加锁）**
**加锁时限（线程尝试获取锁的时候加上一定的时限，超过时限则放弃对该锁的请求，并释放自己占有的锁）**
**死锁检测**

**加锁顺序**
当多个线程需要相同的一些锁，但是按照不同的顺序加锁，死锁就很容易发生。

如果能确保所有的线程都是按照相同的顺序获得锁，那么死锁就不会发生。看下面这个例子：

```java
Thread 1:
  lock A 
  lock B

Thread 2:
   wait for A
   lock C (when A locked)

Thread 3:
   wait for A
   wait for B
   wait for C
```




如果一个线程（比如线程3）需要一些锁，那么它必须按照确定的顺序获取锁。它只有获得了从顺序上排在前面的锁之后，才能获取后面的锁。

例如，线程2和线程3只有在获取了锁A之后才能尝试获取锁C(译者注：获取锁A是获取锁C的必要条件)。因为线程1已经拥有了锁A，所以线程2和3需要一直等到锁A被释放。然后在它们尝试对B或C加锁之前，必须成功地对A加了锁。

按照顺序加锁是一种有效的死锁预防机制。但是，这种方式需要你事先知道所有可能会用到的锁(译者注：并对这些锁做适当的排序)，但总有些时候是无法预知的。

**加锁时限**
另外一个可以避免死锁的方法是在尝试获取锁的时候加一个超时时间，这也就意味着在尝试获取锁的过程中若超过了这个时限该线程则放弃对该锁请求。若一个线程没有在给定的时限内成功获得所有需要的锁，则会进行回退并释放所有已经获得的锁，然后等待一段随机的时间再重试。这段随机的等待时间让其它线程有机会尝试获取相同的这些锁，并且让该应用在没有获得锁的时候可以继续运行(译者注：加锁超时后可以先继续运行干点其它事情，再回头来重复之前加锁的逻辑)。

以下是一个例子，展示了两个线程以不同的顺序尝试获取相同的两个锁，在发生超时后回退并重试的场景：

```java
Thread 1 locks A
Thread 2 locks B

Thread 1 attempts to lock B but is blocked
Thread 2 attempts to lock A but is blocked

Thread 1's lock attempt on B times out
Thread 1 backs up and releases A as well
Thread 1 waits randomly (e.g. 257 millis) before retrying.

Thread 2's lock attempt on A times out
Thread 2 backs up and releases B as well
Thread 2 waits randomly (e.g. 43 millis) before retrying.

```



在上面的例子中，线程2比线程1早200毫秒进行重试加锁，因此它可以先成功地获取到两个锁。这时，线程1尝试获取锁A并且处于等待状态。当线程2结束时，线程1也可以顺利的获得这两个锁（除非线程2或者其它线程在线程1成功获得两个锁之前又获得其中的一些锁）。

需要注意的是，由于存在锁的超时，所以我们不能认为这种场景就一定是出现了死锁。也可能是因为获得了锁的线程（导致其它线程超时）需要很长的时间去完成它的任务。

此外，如果有非常多的线程同一时间去竞争同一批资源，就算有超时和回退机制，还是可能会导致这些线程重复地尝试但却始终得不到锁。如果只有两个线程，并且重试的超时时间设定为0到500毫秒之间，这种现象可能不会发生，但是如果是10个或20个线程情况就不同了。因为这些线程等待相等的重试时间的概率就高的多（或者非常接近以至于会出现问题）。
(译者注：超时和重试机制是为了避免在同一时间出现的竞争，但是当线程很多时，其中两个或多个线程的超时时间一样或者接近的可能性就会很大，因此就算出现竞争而导致超时后，由于超时时间一样，它们又会同时开始重试，导致新一轮的竞争，带来了新的问题。)

这种机制存在一个问题，在Java中不能对synchronized同步块设置超时时间。你需要创建一个自定义锁，或使用Java5中java.util.concurrent包下的工具。写一个自定义锁类不复杂，但超出了本文的内容。后续的Java并发系列会涵盖自定义锁的内容。

**死锁检测**
死锁检测是一个更好的死锁预防机制，它主要是针对那些不可能实现按序加锁并且锁超时也不可行的场景。

每当一个线程获得了锁，会在线程和锁相关的数据结构中（map、graph等等）将其记下。除此之外，每当有线程请求锁，也需要记录在这个数据结构中。

当一个线程请求锁失败时，这个线程可以遍历锁的关系图看看是否有死锁发生。例如，线程A请求锁7，但是锁7这个时候被线程B持有，这时线程A就可以检查一下线程B是否已经请求了线程A当前所持有的锁。如果线程B确实有这样的请求，那么就是发生了死锁（线程A拥有锁1，请求锁7；线程B拥有锁7，请求锁1）。

当然，死锁一般要比两个线程互相持有对方的锁这种情况要复杂的多。线程A等待线程B，线程B等待线程C，线程C等待线程D，线程D又在等待线程A。线程A为了检测死锁，它需要递进地检测所有被B请求的锁。从线程B所请求的锁开始，线程A找到了线程C，然后又找到了线程D，发现线程D请求的锁被线程A自己持有着。这是它就知道发生了死锁。

下面是一幅关于四个线程（A,B,C和D）之间锁占有和请求的关系图。像这样的数据结构就可以被用来检测死锁。

 ![img](http://ifeve.com/wp-content/uploads/2013/03/deadlock-detection-graph.png) 

那么当检测出死锁时，这些线程该做些什么呢？

一个可行的做法是释放所有锁，回退，并且等待一段随机的时间后重试。这个和简单的加锁超时类似，不一样的是只有死锁已经发生了才回退，而不会是因为加锁的请求超时了。虽然有回退和等待，但是如果有大量的线程竞争同一批锁，它们还是会重复地死锁（编者注：原因同超时类似，不能从根本上减轻竞争）。

一个更好的方案是给这些线程设置优先级，让一个（或几个）线程回退，剩下的线程就像没发生死锁一样继续保持着它们需要的锁。如果赋予这些线程的优先级是固定不变的，同一批线程总是会拥有更高的优先级。为避免这个问题，可以在死锁发生的时候设置随机的优先级。


#### 5.0	线程同步

**为什么使用同步：**

  java允许多线程并发控制，当多个线程同时操作一个可共享的资源变量时（如数据的增删改查），
  将会导致数据不准确，相互之间产生冲突，因此加入同步锁以避免在该线程没有完成操作之前，被其他线程的调用，
  从而保证了该变量的**唯一性和准确性**。 

假设银行里某一用户账户有1000元，线程A读取到1000，并想取出这1000元，并且在栈中修改成了0但还没有刷新到堆中，线程B也读取到1000，此时账户刷新到银行系统中，则账户的钱变成了0，这个时候也想去除1000，再次刷新到行系统中，账号的钱变成0，这个时候A，B都取出1000元，但是账户只有1000，显然出现了问题。针对上述问题，假设我们添加了同步机制，那么就可以很容易的解决。

怎样解决这种问题呢，在线程使用一个资源时为其加锁即可。**访问资源的第一个线程为其加上锁以后，其他线程便不能再使用那个资源，除非被解锁。**


```JAVA
package com.java.test;

/**
 * Created by xiaofandiy03 on 2018/4/14.
 */
public class     DrawMoneyTest {
    public static void main(String[] args)
    {
        Bank bank = new Bank();

        Thread t1 = new MoneyThread(bank);// 从银行取钱
        Thread t2 = new MoneyThread(bank);// 从取款机取钱

        t1.start();
        t2.start();

    }
}
class Bank{
    private int money =1000;
    public int getMoney(int number)
    {
        if(number <0)
        {
            return -1;
        }else if(number >money) {
            return  -2;
        }else if(money <0)
        {
            return -3;
        }else {
            try {
                Thread.sleep(1000);
            }catch (InterruptedException e)
            {
                e.printStackTrace();
            }
        }
            money-=number;
        System.out.println("Left Money :" +money);
        return  number;
    }
}
class MoneyThread extends Thread
{
    private Bank bank;

    public MoneyThread(Bank bank)
    {
        this.bank = bank;
    }

    @Override
    public void run()
    {
        System.out.println(bank.getMoney(1000));
    }
}
```



怎么解决这种问题呢，解决的方案是**加锁**。

你想要进行对一组加锁的代码进行操作吗？想的话就先拿到锁，拿到锁之后就可以操作被加锁的代码，倘若拿不到锁的话就只能等着，因为等的线程太多了，这就是线程的阻塞。

###### **竞态条件和内存可见性**

线程和线程之间是共享内存的，当多线程对共享内存进行操作的时候有几个问题是难以避免的，竞态条件和内存可见性。

**竞态条件**

当多线程访问和操作同一对象的时候计算的正确性取决于多个线程的交替执行时序时，就会发生竞态条件

最常见的竞态条件为：

1. 先检测后执行。执行依赖于检测的结果，而检测结果依赖于多个线程的执行时序，而多个线程的执行时序通常情况下是不固定不可判断的，从而导致执行结果出现各种问题。
2. 延迟初始化（最典型即为单例）

上文中说到的加锁就是为了解决这个问题，常见的解决方案有：

- 使用synchronized关键字
- 使用显式锁（Lock）
- 使用原子变量

**内存可见性**

 

关于内存可见性问题要先从内存和cpu的配合谈起，内存是一个硬件，执行速度比CPU慢几百倍，所以在计算机中，CPU在执行运算的时候，不会每次运算都和内存进行数据交互，而是先把一些数据写入CPU中的缓存区（寄存器和各级缓存），在结束之后写入内存。这个过程是及其快的，单线程下并没有任何问题。

但是在多线程下就出现了问题，一个线程对内存中的一个数据做出了修改，但是并没有及时写入内存（暂时存放在缓存中）；这时候另一个线程对同样的数据进行修改的时候拿到的就是内存中还没有被修改的数据，也就是说一个线程对一个共享变量的修改，另一个线程不能马上看到，甚至永远看不到。

这就是内存的可见性问题。

解决这个问题的常见方法是：

- 使用volatile关键字
- 使用synchronized关键字或显式锁同步

##### **线程同步方法**

###### **同步方法：**

即有synchronized关键字修饰方法。悠悠java每个对象都有一个内置锁，放用关键字修饰方法时，内置所会保护整个方法。在调用该方法钱，获得内置锁，否则就处于阻塞状态。

```
class Bank{
    private int money =1000;
    public synchronized int getMoney(int number)
    {
        if(number <0)
        {
            return -1;
        }else if(number >money) {
            return  -2;
        }else if(money <0)
        {
            return -3;
        }else {
            try {
                Thread.sleep(1000);
            }catch (InterruptedException e)
            {
                e.printStackTrace();
            }
        }
            money-=number;
        System.out.println("Left Money :" +money);
        return  number;
    }
}复制代码
```

 synchronized关键字也可以修饰静态方法，此时如果调用该静态方法，将会锁住整个类

###### **同步代码块**

即有synchronized关键字修饰的语句块。被该关键字修饰的语句块会自动被加上内置锁，从而实现同步

```
class Bank{
    private int money =1000;
    public  int getMoney(int number)
    {
        synchronized (this) {
            if (number < 0) {
                return -1;
            } else if (number > money) {
                return -2;
            } else if (money < 0) {
                return -3;
            } else {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            money -= number;
        }
        System.out.println("Left Money :" +money);
        return  number;
    }
}
复制代码
```

同步是一种高开销的操作，因此应该尽量减少同步的内容。通常没有必要同步整个方法，使用synchronized代码块同步关键代码即可。

###### **使用重入锁实现线程同步**

在JavaSE5.0中新增了一个java.util.concurrent包来支持同步。ReentrantLock类是可重入、互斥、实现了Lock接口的锁， 它与使用synchronized方法和快具有相同的基本行为和语义，并且扩展了其能力。
     ReenreantLock类的常用方法有：
         ReentrantLock() : 创建一个ReentrantLock实例 
         lock() : 获得锁 
         unlock() : 释放锁 
    注：ReentrantLock()还有一个可以创建公平锁的构造方法，但由于能大幅度降低程序运行效率，不推荐使用 

eentrantLock具有和synchronized相似的作用，但是更加的灵活和强大。

它是一个重入锁（synchronized也是），所谓重入就是可以重复进入同一个函数，这有什么用呢？

假设一种场景，一个递归函数，如果一个函数的锁只允许进入一次，那么线程在需要递归调用函数的时候，应该怎么办？退无可退，有不能重复进入加锁的函数，也就形成了一种新的死锁。

重入锁的出现就解决了这个问题，实现重入的方法也很简单，就是给锁添加一个计数器，一个线程拿到锁之后，每次拿锁都会计数器加1，每次释放减1，如果等于0那么就是真正的释放了锁。

###### **volatile 关键字**

当一个共享变量被volatile修饰的时候，他会保证变量被修改之后立马在内存中更新，另一线程在取值时候需要去内存中读取新的值。

volatile可以保证变量的内存可见性，但是不能保证原子性，对于b++这个操作来说，并不是一步到位的，而是分好几步的，读取白那两，定义常量1，变量b加1，结果同步到内存。虽然在每一步中获取的都是变量的最新值，但是没有保证b++的原子性，自然无法做到线程安全。

###### **使用局部变量实现线程同步**

如果使用ThreadLocal管理变量，则每一个使用该变量的线程都获得该变量的副本，副本之间相互独立，这样每一个线程都可以随意修改自己的变量副本，而不会对其他线程产生影响。现在明白了吧，原来每个线程运行的都是一个副本，也就是说存钱和取钱是两个账户，知识名字相同而已。所以就会发生上面的效果。



**ThreadLocal与同步机制** 
a.ThreadLocal与同步机制都是为了解决多线程中相同变量的访问冲突问题

b.前者采用以"空间换时间"的方法，后者采用以"时间换空间"的方式 



#### 5.Synchronized放在静态方法和非静态方法上的锁对象分别是什么？(校招&实习)

1.Synchronized修饰非静态方法，实际上是对调用该方法的对象加锁，俗称“对象锁”。

       Java中每个对象都有一个锁，并且是唯一的。假设分配的一个对象空间，里面有多个方法，相当于空间里面有多个小房间，如果我们把所有的小房间都加锁，因为这个对象只有一把钥匙，因此同一时间只能有一个人打开一个小房间，然后用完了还回去，再由JVM 去分配下一个获得钥匙的人。



情况1：同一个对象在两个线程中分别访问该对象的两个同步方法

结果：会产生互斥。

解释：因为锁针对的是对象，当对象调用一个synchronized方法时，其他同步方法需要等待其执行结束并释放锁后才能执行。



情况2：不同对象在两个线程中调用同一个同步方法

结果：不会产生互斥。

解释：因为是两个对象，锁针对的是对象，并不是方法，所以可以并发执行，不会互斥。形象的来说就是因为我们每个线程在调用方法的时候都是new 一个对象，那么就会出现两个空间，两把钥匙，



2.Synchronized修饰静态方法，实际上是对该类对象加锁，俗称“类锁”。

情况1：用类直接在两个线程中调用两个不同的同步方法

结果：会产生互斥。

解释：因为对静态对象加锁实际上对类（.class）加锁，类对象只有一个，可以理解为任何时候都只有一个空间，里面有N个房间，一把锁，因此房间（同步方法）之间一定是互斥的。

注：上述情况和用单例模式声明一个对象来调用非静态方法的情况是一样的，因为永远就只有这一个对象。所以访问同步方法之间一定是互斥的。



情况2：用一个类的静态对象在两个线程中调用静态方法或非静态方法

结果：会产生互斥。

解释：因为是一个对象调用，同上。



情况3：一个对象在两个线程中分别调用一个静态同步方法和一个非静态同步方法

结果：不会产生互斥。

解释：因为虽然是一个对象调用，但是两个方法的锁类型不同，调用的静态方法实际上是类对象在调用，即这两个方法产生的并不是同一个对象锁，因此不会互斥，会并发执行。



 测试代码：

同步方法类：SynchronizedTest.java 



```JAVA
public class SynchronizedTest {  
    /*private SynchronizedTest(){} 
    private static SynchronizedTest st;           //懒汉式单例模式，线程不安全，需要加synchronized同步 
    public static SynchronizedTest getInstance(){ 
        if(st == null){ 
            st = new SynchronizedTest(); 
        } 
        return st; 
    }*/  
    /*private SynchronizedTest(){} 
    private static final SynchronizedTest st = new SynchronizedTest();  //饿汉式单利模式，天生线程安全 
    public static SynchronizedTest getInstance(){ 
        return st; 
    }*/  
      
    public static SynchronizedTest staticIn = new SynchronizedTest();   //静态对象  
      
     public synchronized void method1(){                                      //非静态方法1  
         for(int i = 0;i < 10;i++){    
             System.out.println("method1 is running!");  
             try {  
                Thread.sleep(1000);  
            } catch (InterruptedException e) {  
                // TODO Auto-generated catch block  
                e.printStackTrace();  
            }  
         }  
     }  
     public synchronized void method2(){                                   //非静态方法2  
         for( int i = 0; i < 10 ; i++){  
             System.out.println("method2 is running!");  
             try {  
                Thread.sleep(1000);  
            } catch (InterruptedException e) {  
                // TODO Auto-generated catch block  
                e.printStackTrace();  
            }  
         }  
     }  
    public synchronized static void staticMethod1(){                     //静态方法1  
        for( int i = 0; i < 10 ; i++){  
         System.out.println("static method1 is running!");  
         try {  
                Thread.sleep(1000);  
            } catch (InterruptedException e) {  
                // TODO Auto-generated catch block  
                e.printStackTrace();  
            }  
     }  
    }  
    public synchronized static void staticMethod2(){                      //静态方法2  
        for( int i = 0; i < 10 ; i++){  
         System.out.println("static method2 is running!");  
         try {  
                Thread.sleep(1000);  
            } catch (InterruptedException e) {  
                // TODO Auto-generated catch block  
                e.printStackTrace();  
            }  
     }  
    }  
}  
```



 线程类1：Thread1.java（释放不同的注释可以测试不同的情况） 



```JAVA
public class Thread1 implements Runnable{  
  
    @Override  
    public void run() {  
//      SynchronizedTest s = SynchronizedTest.getInstance();  
//      s.method1();  
//      SynchronizedTest s1 = new SynchronizedTest();  
//      s1.method1();  
        SynchronizedTest.staticIn.method1();  
  
//      SynchronizedTest.staticMethod1();  
//      SynchronizedTest.staticMethod2();  
    }  
}  
```




线程类2：Thread2.Java



```JAVA
public class Thread2 implements Runnable{  
  
    @Override  
    public void run() {  
        // TODO Auto-generated method stub  
//      SynchronizedTest s = SynchronizedTest.getInstance();  
//      SynchronizedTest s2 = new SynchronizedTest();  
//      s2.method1();  
//      s.method2();  
//      SynchronizedTest.staticMethod1();  
//      SynchronizedTest.staticMethod2();  
//      SynchronizedTest.staticIn.method2();  
        SynchronizedTest.staticIn.staticMethod1();  
    }  
}  
```



 


主类：ThreadMain.java

 



```JAVA
import java.util.concurrent.ExecutorService;  
import java.util.concurrent.Executors;  
  
public class ThreadMain {  
    public static void main(String[] args) {  
    Thread t1 = new Thread(new Thread1());  
        Thread t2 = new Thread(new Thread2());  
        ExecutorService exec = Executors.newCachedThreadPool();  
        exec.execute(t1);  
        exec.execute(t2);  
        exec.shutdown();  
    }  
  
} 
```



 总结： 

  1.对象锁钥匙只能有一把才能互斥，才能保证共享变量的唯一性

  2.在静态方法上的锁，和 实例方法上的锁，默认不是同样的，如果同步需要制定两把锁一样。

  3.关于同一个类的方法上的锁，来自于调用该方法的对象，如果调用该方法的对象是相同的，那么锁必然相同，否则就不相同。比如 new A().x() 和 new A().x(),对象不同，锁不同，如果A的单利的，就能互斥。

  4.静态方法加锁，能和所有其他静态方法加锁的 进行互斥

  5.静态方法加锁，和xx.class 锁效果一样，直接属于类的

 https://www.jianshu.com/p/8327c5c15cb8 

#### 6.如何停止掉一个线程？

停止线程是在多线程开发是很重要的技能点之一，并不像break那样干脆，而Java中有一下**3**种方式可以终止正在运行的线程：

- 使用退出标志，使线程正常退出，也就是当run()方法结束之后停止线程

- 使用stop()方法强制停止线程，但是不推荐使用这个方法，因为该方法已经作废过期，使用时会出现一些不可预料的后果

- 使用interrupt()方法中断线程：

  - 暴力法停止线程

    调用stop()方法是会跑出java.lang.ThreadDeath异常，但是筒仓情况下，磁异常不需要显式的捕捉

    ```java
          try {
                myThread.stop();
            } catch (ThreadDeath e) {
                e.printStackTrace();
            }
    
    ```

     方法 stop() 已经被作废，因为如果强制让线程停止线程则有可能使一些清理性的工作得不到完成。另外一个情况就是对锁定的对象进行了“解锁”，导致数据得不到同步的处理，出现数据不一致的情况。示例如下： 

    ```JAVA
    public class UserPass {
        private String username = "aa";
        private String password = "AA";
    
        public String getUsername() {
            return username;
        }
    
        public void setUsername(String username) {
            this.username = username;
        }
    
        public String getPassword() {
            return password;
        }
    
        public void setPassword(String password) {
            this.password = password;
        }
    
        synchronized public void println(String username, String password){
            this.username = username;
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            this.password = password;
        }
    
        public static void main(String[] args) throws InterruptedException {
            UserPass userPass = new UserPass();
            Thread thread = new Thread(new Runnable() {
                @Override
                public void run() {
                    userPass.println("bb","BB");
                }
            });
            thread.start();
            Thread.sleep(500);
            thread.stop();
            System.out.println(userPass.getUsername()+" "+userPass.getPassword());
        }
    
    }
    //运行结果
    //bb
    //AA
    ```

  - 异常法停止线程

    使用interrupt()方法并不会真正的停止线程，而是调用interrupt()方法仅仅是在当前线程中打下了一个停止标记，并不是真正的停止线程

    而如果判断该线程是否被打下了停止标记，Thread类提供了两种方法：

    ```java
    interrupted()//测试当前线程是否中断
    isInterrupted()//测试线程是否已经中断
        
    /**
    interrupted：不知可以判断当前线程是否已经被中断，而且可以回清除该状态的中断状态
    isInterrupted:只会判断当前线程是否已经中断，不会清除线程中的中断状态
    */
    ```

    但是仅靠上述的两种方法可以通过while(!this.isInterrupted()){}对代码进行控制，但是如果循环外还有语句，程序还是会继续运行的。这是可以抛出异常从而使线程彻底终止：

    ```java
    public class MyThread extends Thread {
        @Override
        public void run() {
            try {
                for (int i=0; i<50000; i++){
                    if (this.isInterrupted()) {
                        System.out.println(" 已经是停止状态了！");
                        throw new InterruptedException();
                    }
                    System.out.println(i);
                }
                System.out.println(" 不抛出异常，我会被执行的哦！");
            } catch (Exception e) {
    //            e.printStackTrace();
            }
        }
    
        public static void main(String[] args) throws InterruptedException {
            MyThread myThread =new MyThread();
            myThread.start();
            Thread.sleep(100);
            myThread.interrupt();
        }
    
    }
    
    /**
    打印结果：
    
    …
    2490
    2491
    2492
    2493
    已经是停止状态了！
    */
    ```

    注意：

    如果线程在sleep()状态下被停止，也就是线程对象的run()方法含有sleep()方法，在此期间又执行thread.interrupt()方法，会跑出java》lang.InterruptedException：sleep interrupted异常，提前中断

  - return法停止线程

    润方法很简单，只需要把异常法中的额抛出异常更改为return即可，代码如下：

    ```java
    public class MyThread extends Thread {
        @Override
        public void run() {
    
            for (int i=0; i<50000; i++){
                if (this.isInterrupted()) {
                    System.out.println(" 已经是停止状态了！");
                    return;// 替换此处
                }
                System.out.println(i);
            }
            System.out.println(" 不进行 return，我会被执行的哦！");
    
        }
    }
    
    ```

     不过还是建议使用“抛异常”来实现线程的停止，因为在 catch 块中可以对异常的信息进行相关的处理，而且使用异常能更好、更方便的控制程序的运行流程，不至于代码中出现多个 return，造成污染。 

    

#### 6.1 如何暂停线程



**暂停线程的四种方法的区别**

**1、Final void join()**

**调用该方法的线程**强制执行，其它线程处于阻塞状态，该线程执行完毕后，其它线程再执行

**案例：**线程的联合-join()

线程A在运行期间，可以调用线程B的join()方法，让线程B和线程A联合。这样，线程A就必须等待线程B执行完毕后，才能继续执行。如下面示例中，“爸爸线程”要抽烟，于是联合了“儿子线程”去买烟，必须等待“儿子线程”买烟完毕，“爸爸线程”才能继续抽烟。

```text
public class TestThreadState {
    public static void main(String[] args) {
        System.out.println("爸爸和儿子买烟故事");
        Thread father = new Thread(new FatherThread());
        father.start();
    }
}
 
class FatherThread implements Runnable {
    public void run() {
        System.out.println("爸爸想抽烟，发现烟抽完了");
        System.out.println("爸爸让儿子去买包红塔山");
        Thread son = new Thread(new SonThread());
        son.start();
        System.out.println("爸爸等儿子买烟回来");
        try {
            son.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
            System.out.println("爸爸出门去找儿子跑哪去了");
            // 结束JVM。如果是0则表示正常结束；如果是非0则表示非正常结束
            System.exit(1);
        }
        System.out.println("爸爸高兴的接过烟开始抽，并把零钱给了儿子");
    }
}
 
class SonThread implements Runnable {
    public void run() {
        System.out.println("儿子出门去买烟");
        System.out.println("儿子买烟需要10分钟");
        try {
            for (int i = 1; i <= 10; i++) {
                System.out.println("第" + i + "分钟");
                Thread.sleep(1000);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("儿子买烟回来了");
    }
}
```

执行结果如下图所示：



![img](https://pic3.zhimg.com/80/v2-0b0fec6c8b75c0b3233bf52d43e4d95e_hd.jpg)



**2、Static void sleep(long millis )**

使用当前正在执行的线程休眠millis秒，线程处于阻塞状态

**案例：**暂停线程的方法-sleep()

```text
//使用继承方式实现多线程
class StateThread extends Thread {
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println(this.getName() + ":" + i);
            try {
                Thread.sleep(2000);//调用线程的sleep()方法；
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    public static void main(String[] args) {
        StateThread thread1 = new StateThread();
        thread1.start();
        StateThread thread2 = new StateThread();
        thread2.start();
    }
}
```

执行结果如下图所示(注：以下图示只是部分结果，运行时可以感受到每条结果输出之前的延迟，是Thread.sleep(2000)语句在起作用)：



![img](https://pic3.zhimg.com/80/v2-492477eda49990c4ed5c298da9deb8ae_hd.jpg)



**3、Static void yield（）**

**当前正在执行的线程**暂停一次，允许其他线程执行，不阻塞，线程进入就绪状态，如果没有其他等待执行的线程，这个时候当前线程就会马上恢复执行。

**案例：**暂停线程的方法-yield()

```text
public class TestThreadState {
    public static void main(String[] args) {
        StateThread thread1 = new StateThread();
        thread1.start();
        StateThread thread2 = new StateThread();
        thread2.start();
    }
}
//使用继承方式实现多线程
class StateThread extends Thread {
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println(this.getName() + ":" + i);
            Thread.yield();//调用线程的yield()方法；
        }
    }
}
```



执行结果如下图所示(注：以下图示只是部分结果，可以引起线程切换，但运行时没有明显延迟)：



![img](https://pic4.zhimg.com/80/v2-ab286da5c98b4475d76bcb341f3891e7_hd.jpg)



**4、Final void stop（）**

强迫线程停止执行，**已过时**。不推荐使用。



**5，wait()与notify()方法：**

wait()方法同样可以使线程进行挂起操作，调用了wait()方法的线程进入了“非可执行”状态，使用wait()方法有两种方式，例如：
thread.wait(1000);
或：
thread.wait();
thread.notify();
其中第一种方式给定线程挂起时间，基本上与sleep()方法用法相同。第二种方式是wait()与notify()方法配合使用，这种方式让wait()方法无限等下去，直到线程接收到notify()或notifyAll()消息为止。
     wait()、notify()、notifyAll()不同于其他线程方法，这3个方法是java.lang.Object类的一部分，所以在定义自己类时会继承下来。wait()、notify()、notifyAll()都被声明为final类，所以无法重新定义。



**6、suspend()与resume()方法**
       有时更好地挂起方法是强制挂起线程，而不是为线程指定休眠时间，这种情况下由其他线程负责唤醒其继续执行，除了wait()与notify()方法之外，线程中还有一对方法用于完成此功能，这就是suspend()与resume()方法。thread.suspend();thread.resume()，线程thread在运行到suspend()之后被强制挂起，暂停运行，直到主线程调用thread.resume()方法时才被重新唤醒。
      Java2中已经废弃了suspend()和resume()方法，因为使用这两个方法可能会产生死锁，所以应该使用同步对象调用wait()和notify()的机制来代替suspend()和resume()进行线程控制。



***注意事项：***

**1、sleep：**不会释放锁，Sleep时别的线程也不可以访问锁定对象，即**当我在一个同步方法或者同步代码块中，我调用了sleep，此时线程进入了阻塞状态，但是此时锁定的对象并没有因为线程进入阻塞状态而释放他，也就是说此时的对象依然是被锁定，其他线程并不能对次对象进行访问。**

**2、Yield：**让出CPU的使用权，从运行态直接进入就绪状态。让CPU重新挑选哪一个线程进行运行状态。

**3、Join：**当某个线程等待另一个线程执行结束后，才继续执行时，使调用该方法的线程在此之前执行完毕，也就是等待调用该方法的线程执行完毕后再往下继续执行。**调用它的线程**才会被阻塞，而其他线程并不会。





***(2) sleep,yield,join方法将会导致线程进入什么状态？***



Sleep，会导致**正在执行的线程**进入**阻塞状态**

Yield，会导致**正在执行的线程**进入**就绪状态**

Join，会导致**调用它的线程**进入**阻塞状态**，而**调用此方法的线程**强制执行，也就是变成**运行状态**






#### 6.2 守护线程

Java中的线程有两种，一种是**用户线程**，另一种是守护线程

 用户线程即运行在前台的线程，而守护线程是运行在后台的线程 .

守护线程作用是为其他前台线程的运行提供便利服务，而且仅在普通、非守护线程仍然运行时才需要，比如垃圾回收线程就是一个守护线程。当 VM 检测仅剩一个守护线程，而用户线程都已经退出运行时，VM就会退出，因为没有如果没有了被守护这，也就没有继续运行程序的必要了。如果有非守护线程仍然存活，VM 就不会退出。

守护线程并非只有虚拟机内部提供，用户在编写程序时也可以自己设置守护线程。用户可以用 Thread 的 setDaemon（true）方法设置当前线程为守护线程。

虽然守护线程可能非常有用，但必须小心确保其他所有非守护线程消亡时，不会由于它的终止而产生任何危害。因为你不可能知道在所有的用户线程退出运行前，守护线程是否已经完成了预期的服务任务。一旦所有的用户线程退出了，虚拟机也就退出运行了。 因此，不要在守护线程中执行业务逻辑操作（比如对数据的读写等）。

另外有几点需要注意：

- setDaemon(true)必须在调用线程的 start()方法之前设置，否则会跑出 IllegalThreadStateException 异常。
- 在守护线程中产生的新线程也是守护线程。
- 不要认为所有的应用都可以分配给守护线程来进行服务，比如读写操作或者计算逻辑。

**守护线程**：是一种特殊的线程，当进程中不存在非守护线程了，则守护线程自动销毁。典型的守护线程就是垃圾回收线程，当进程中没有非守护线程了，则垃圾回收线程也就是没有了存在的必要，就会自动销毁。简单来说：任何一个守护线程都是非守护线程的保姆



如何设置守护线程？通过Thread.setDamon(false)设置为用户线程，通过Thread.setDaemon(true)设置为守护线程。如果不设置属性，默认为用户线程

```java
thread.setDaemon(true);

```

示例如下：

```java
public class MyThread extends Thread {
    private int i = 0;
    @Override
    public void run() {
        try {
            while (true){
                i++;
                System.out.println("i="+i);
                Thread.sleep(1000);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        MyThread thread = new MyThread();
        thread.setDaemon(true);
        thread.start();
        Thread.sleep(5000);
        System.out.println(" 我离开后 thread 对象也就不再打印了 ");
    }
}

/*
打印结果：
i=1;
i=2;
i=3;
i=4;
i=5;
我离开后thread对象也就不再打印了
*/
```



#### 7.**给我说说线程池的种类** & 特点 & 内部原理 & 平时当中使用案例。

线程池是一种多线程处理形式，处理过程中将任务添加队列，然后在创建线程后自动启动这些任务，每个线程都使用默认的堆栈大小，以默认的优先级运行，并处在多线程单元中，如果某个线程在托管代码中空闲，则线程池将插入另一个辅助线程来使所有处理器保持繁忙。如果所有线程池都始终保持繁忙，但队列中包含挂起的工作，则线程池将在一段时间后辅助线程的数目永远不会超过最大值。超过最大值的线程可以排队，但他们要等到其他线程完成后才能启动。

java里面的线程池的顶级接口是Executor，Executor并不是一个线程池，而只是一个执行线程的工具，而真正的线程池是ExecutorService。

  **在什么情况下使用线程池？** 

  1.单个任务处理的时间比较短 
  2.将需处理的任务的数量大 

  **使用线程池的好处:** 

  1.减少在创建和销毁线程上所花的时间以及系统资源的开销 
  2.如不使用线程池，有可能造成系统创建大量线程而导致消耗完系统内存以及”过度切换”。 

线程池工作原理：http://www.ibm.com/developerworks/cn/java/j-jtp0730/ 
该文章里有个例子，简单的描述了线程池的内部实现，建议根据里面的例子来了解JAVA 线程池的原理。同时，里面还详细描述了使用线程池存在的优点和弊端，大家可以研究下，我觉得是篇非常好的文章。  

 **JDK自带线程池总类介绍介绍：** 

  1、**newFixedThreadPool**创建一个指定工作线程数量的线程池。每当提交一个任务就创建一个工作线程，如果工作线程数量达到线程池初始的最大数，则将提交的任务存入到池队列中。 

  2、**newCachedThreadPool**创建一个可缓存的线程池。这种类型的线程池特点是： 
  1).工作线程的创建数量几乎没有限制(其实也有限制的,数目为Interger. MAX_VALUE), 这样可灵活的往线程池中添加线程。 
  2).如果长时间没有往线程池中提交任务，即如果工作线程空闲了指定的时间(默认为1分钟)，则该工作线程将自动终止。终止后，如果你又提交了新的任务，则线程池重新创建一个工作线程。 

  3、**newSingleThreadExecutor**创建一个单线程化的Executor，即只创建唯一的工作者线程来执行任务，如果这个线程异常结束，会有另一个取代它，保证顺序执行(我觉得这点是它的特色)。单工作线程最大的特点是可保证顺序地执行各个任务，并且在任意给定的时间不会有多个线程是活动的 。 

  4、**newScheduleThreadPool**创建一个定长的线程池，而且支持定时的以及周期性的任务执行，类似于Timer。(这种线程池原理暂还没完全了解透彻) 

- 一.FixedThreadPool是一个典型且优秀的线程池，它具有线程池提高程序效率和节省创建线程时所耗的开销的优点。但是，在线程池空闲时，即线程池中没有可运行任务时，它不会释放工作线程，还会占用一定的系统资源。 
-  二．CachedThreadPool的特点就是在线程池空闲时，即线程池中没有可运行任务时，它会释放工作线程，从而释放工作线程所占用的资源。但是，但当出现新任务时，又要创建一新的工作线程，又要一定的系统开销。并且，在使用CachedThreadPool时，一定要注意控制任务的数量，否则，由于大量线程同时运行，很有会造成系统瘫痪。 

#### 8.给我谈谈你是如何保证线程数据安全问题的？

1，多线程数据安全问题引入

      多线程可以提高程序的使用率，也可以提高cpu的使用使用率，同时也会带来一些弊端，比如：

数据安全问题
      今天我们就先说一下多线程会产生什么样的数据安全问题；在讲之前，我们需要明确：

cpu执行的操作是原子性的，是不可拆分的
这里说的数据安全针对的数据是多个线程共享的数据，所以我们会使用在第一部分中说的方式2来实现多线程
      由此，我们可以总结出会造成多线程数据安全的必要条件：

多线程环境
多个线程操作共享数据
操作共享数据的语句不是原子性的（多条）



用于解决多线程安全问题的方式：

- 同步代码块 (隐式锁) 

    synchronized(被加锁的对象){ 代码 } ==》 解决线程安全问题 

  ```JAVA
  package com.lxj.juc;
   
  public class TestSync {
   
  	public static void main(String[] args) {
  		Ticket ticket = new Ticket();
  		new Thread(ticket).start();
  		new Thread(ticket).start();
  		new Thread(ticket).start();
  	}
  	
  }
  class Ticket implements Runnable{
   
  	private int ticket = 100;
  	
  	@Override
  	public void run() {
  		while(true) {
              /*
              改代码块中在一个时间点只能被一个线程访问，其他线程需要排队
              */
  			synchronized(this) {  //this代表对当前对象上锁
  				if(ticket > 0) {
  					try {
  						Thread.sleep(200);
  					} catch (InterruptedException e) {
  						e.printStackTrace();
  					}
  					System.out.println(Thread.currentThread().getName()+" : 购票成功，余票为：" +  --ticket );
  				}else {
  					break;
  				}
  			}
  		}
  	}
  	
  }
  /*
  运行结果:
  Thread-0 : 购票成功，余票为：99
  Thread-0 : 购票成功，余票为：98
  Thread-2 : 购票成功，余票为：97
  .....
  Thread-2 : 购票成功，余票为：7
  Thread-0 : 购票成功，余票为：6
  Thread-0 : 购票成功，余票为：5
  Thread-1 : 购票成功，余票为：4
  Thread-1 : 购票成功，余票为：3
  Thread-0 : 购票成功，余票为：2
  Thread-0 : 购票成功，余票为：1
  Thread-2 : 购票成功，余票为：0
  */
  ```

  

- 同步方法(隐式锁) 

  

  ```JAVA
  
  package com.lxj.juc;
   
  public class TestSync {
   
  	public static void main(String[] args) {
  		Ticket ticket = new Ticket();
  		new Thread(ticket).start();
  		new Thread(ticket).start();
  		new Thread(ticket).start();
  	}
   
  }
   
  class Ticket implements Runnable {
   
  	private int ticket = 100;
   
  	@Override
  	public void run() {
  		while (true) {
  			if (ticket > 0) {
  				int i = purchase();
  				if(i == 0) {
  					break;
  				}
  			}else {
  				break;
  			}
  		}
  	}
   /*
  	 * 在方法定义时，添加一个synchronized修饰符
  	 * 效果： 
   	 * 使用synchronized修饰的方法就是同步方法
  	 * 	特点： 1.效率会变低
  	 * 		  2.没有线程安全问题了
  	 *  原理：在方法添加synchronized之后，就相等于给当前对象(当前场景是a)加了一把锁.
  	 *  锁的作用：当一个线程进入该方法时，先看锁还在不在，锁在：把锁取走
  	 *  在第一个线程还没有结束时，第二个线程访问该方法，先看锁还在不在？       锁不在==》等待锁回来
  	 *  第一个线程执行结束，把锁还回去，第二个线程发现锁回来了，把锁取走，开始执行方法
  	 */
  	
  	private synchronized int purchase() {
  		if (ticket > 0) {
  			try {
  				Thread.sleep(200);
  			} catch (InterruptedException e) {
  				e.printStackTrace();
  			}
  			System.out.println(Thread.currentThread().getName() + " : 购票成功，余票为：" + --ticket);
  		}else {
  			return 0;
  		}
  		return 1;
  	}
   
  }
  /*
  package com.lxj.juc;
   
  public class TestSync {
   
  	public static void main(String[] args) {
  		Ticket ticket = new Ticket();
  		new Thread(ticket).start();
  		new Thread(ticket).start();
  		new Thread(ticket).start();
  	}
   
  }
   
  class Ticket implements Runnable {
   
  	private int ticket = 100;
   
  	@Override
  	public void run() {
  		while (true) {
  			if (ticket > 0) {
  				int i = purchase();
  				if(i == 0) {
  					break;
  				}
  			}else {
  				break;
  			}
  		}
  	}
   
  	private synchronized int purchase() {
  		if (ticket > 0) {
  			try {
  				Thread.sleep(200);
  			} catch (InterruptedException e) {
  				e.printStackTrace();
  			}
  			System.out.println(Thread.currentThread().getName() + " : 购票成功，余票为：" + --ticket);
  		}else {
  			return 0;
  		}
  		return 1;
  	}
   
  }
  */
  
  ```

  

- 同步锁 Lock( jdk 1.5 后)  

  ```JAVA
  /*
  使用Lock的步骤：
   * 1.创建Lock实现类的对象
   * 2.使用Lock对象的lock方法加锁
   * 3.使用Lock对象的unlock方法解锁
   * 注意:可把unlock方法的调用放在finally代码块中，保证一定能解锁
  
  */
  
  package com.lxj.juc;
   
  import java.util.concurrent.locks.Lock;
  import java.util.concurrent.locks.ReentrantLock;
   
  public class TestSync {
   
  	public static void main(String[] args) {
  		Ticket ticket = new Ticket();
  		new Thread(ticket).start();
  		new Thread(ticket).start();
  		new Thread(ticket).start();
  	}
   
  }
   
  class Ticket implements Runnable {
   
  	private int ticket = 100;
   
      private Lock lock = new ReentrantLock();
  	
  	@Override
  	public void run() {
  		while (true) {
  			lock.lock(); //上锁
  			try {
  				if (ticket > 0) {
  					try {
  						Thread.sleep(200);
  					} catch (InterruptedException e) {
  						e.printStackTrace();
  					}
  					System.out.println(Thread.currentThread().getName() + " : 购票成功，余票为：" + --ticket);
  				}else {
  					break;
  				}
  			} finally {
  				lock.unlock(); //解锁
  			}
  		}
  	}
   
   
  }
  
  /*
  运行结果：
  Thread-0 : 购票成功，余票为：99
  Thread-1 : 购票成功，余票为：98
  Thread-1 : 购票成功，余票为：97
  Thread-1 : 购票成功，余票为：96
  Thread-1 : 购票成功，余票为：95
  Thread-2 : 购票成功，余票为：94
  ......
  Thread-0 : 购票成功，余票为：6
  Thread-0 : 购票成功，余票为：5
  Thread-1 : 购票成功，余票为：4
  Thread-2 : 购票成功，余票为：3
  Thread-2 : 购票成功，余票为：2
  Thread-2 : 购票成功，余票为：1
  Thread-2 : 购票成功，余票为：0
  */
  ```

  **注意使用同步锁一定要记得关闭锁，放在try{}finally{}中。**

- 注：是一个显示锁，需要通过 lock() 方法上锁，必须通过 unlock() 方法进行释放锁

  

#### 9.wait()和sleep()的区别？



sleep 是线程的方法， wait / notify / notifyAll 是 Object 类的方法；
sleep 不会释放当前线程持有的锁，到时间后程序会继续执行，wait 会释放线程持有的锁并挂起，直到通过 notify 或者 notifyAll 重新获得锁。 

wait和sleep区别
共同点：

1. 他们都是在多线程的环境下，都可以在程序的调用处阻塞指定的毫秒数，并返回。
2. wait()和sleep()都可以通过interrupt()方法 打断线程的暂停状态 ，从而使线程立刻抛出InterruptedException。
   如果线程A希望立即结束线程B，则可以对线程B对应的Thread实例调用interrupt方法。如果此刻线程B正在wait/sleep /join，则线程B会立刻抛出InterruptedException，在catch() {} 中直接return即可安全地结束线程。
   需要注意的是，InterruptedException是线程自己从内部抛出的，并不是interrupt()方法抛出的。对某一线程调用 interrupt()时，如果该线程正在执行普通的代码，那么该线程根本就不会抛出InterruptedException。但是，一旦该线程进入到 wait()/sleep()/join()后，就会立刻抛出InterruptedException 。
   不同点：
3. Thread类的方法：sleep(),yield()等
   Object的方法：wait()和notify()等
4. 每个对象都有一个锁来控制同步访问。Synchronized关键字可以和对象的锁交互，来实现线程的同步。
   sleep方法没有释放锁，而wait方法释放了锁，使得其他线程可以使用同步控制块或者方法。
5. wait，notify和notifyAll只能在同步控制方法或者同步控制块里面使用，而sleep可以在任何地方使用
   所以sleep()和wait()方法的最大区别是：
   　　　　sleep()睡眠时，保持对象锁，仍然占有该锁；
   　　　　而wait()睡眠时，释放对象锁。
   　　但是wait()和sleep()都可以通过interrupt()方法打断线程的暂停状态，从而使线程立刻抛出InterruptedException（但不建议使用该方法）。
   sleep（）方法
   sleep()使当前线程进入停滞状态（阻塞当前线程），让出CUP的使用、目的是不让当前线程独自霸占该进程所获的CPU资源，以留一定时间给其他线程执行的机会;
   　　 sleep()是Thread类的Static(静态)的方法；因此他不能改变对象的机锁，所以当在一个Synchronized块中调用Sleep()方法是，线程虽然休眠了，但是对象的机锁并木有被释放，其他线程无法访问这个对象（即使睡着也持有对象锁）。
   　　在sleep()休眠时间期满后，该线程不一定会立即执行，这是因为其它线程可能正在运行而且没有被调度为放弃执行，除非此线程具有更高的优先级。
   wait（）方法
   wait()方法是Object类里的方法；当一个线程执行到wait()方法时，它就进入到一个和该对象相关的等待池中，同时失去（释放）了对象的机锁（暂时失去机锁，wait(long timeout)超时时间到后还需要返还对象锁）；其他线程可以访问；
   　　wait()使用notify或者notifyAlll或者指定睡眠时间来唤醒当前等待池中的线程。
   　　wiat()必须放在synchronized block中，否则会在program runtime时扔出”java.lang.IllegalMonitorStateException“异常。
   
   

#### 10.什么是公平锁&非公平锁&区别？



[超详细的图解](https://www.jianshu.com/p/f584799f1c77)

公平锁即尽量以请求锁的顺序来获取锁。同时有多个线程在等待一个锁，当这个锁被释放时，等待时间最久的线程（最先请求的线程）会获得该锁，这种就是公平锁。

非公平锁即无法保证锁的获取是按照请求锁的顺序进行的，这样就可能导致某个或者一些线程永远获取不到锁。

`synchronized`是非公平锁，它无法保证等待的线程获取锁的顺序。对于`ReentrantLock`和`ReentrantReadWriteLock`，默认情况下是非公平锁，但是可以设置为公平锁。


 ReentrantLock 提供了公平锁和非公平锁，只需要在构造方法中使用一个 `boolean` 参数即可。默认非公平锁。 

**1. 类 UML 图**



![image.png](https://user-gold-cdn.xitu.io/2018/5/1/16317a5c6a7dd181?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



`ReentrantLock` 内部有一个抽象类 `Sync`，继承了 AQS。

而公平锁的实现就是 `FairSync`，非公平锁的实现就是 `NodFairSync`。

两把锁的区别在于`lock` 方法的实现。

**2. 公平锁 lock 方法实现**

```
final void lock() {
    acquire(1);
}
复制代码
```

调用的是 AQS 的`acquire`方法，熟悉 AQS 的同学都知道，AQS 会回调子类的 `tryAcquire` 方法，看看公平锁的`tryAcquire`实现。

```
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
复制代码
```

说下逻辑：

1. 获取 state 变量，如果是 0，说明锁可以获取。
2. **判断 AQS 队列中是否有等待的线程**，如果没有，就使用 CAS 尝试获取。获取成功后，将 CLH 的持有线程修改为当前线程。
3. 重入锁逻辑。
4. 如果失败，返回 false， AQS 会将这个线程放进队列，并挂起。

注意上面的第二步：**判断 AQS 队列中是否有等待的线程**。

> 这就是公平的体现。

再看看非公平锁的区别。

**3. 非公平锁 lock 方法实现**

```
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}

protected final boolean tryAcquire(int acquires) {
    return nonfairTryAcquire(acquires);
}
复制代码
```

lock  方法就不同了，很冲动的一个方法，直接使用 CAS 获取锁，如果成功了，就设置锁的持有线程为自己。很快速。所以默认使用了非公平锁。

如果失败了，就调用 AQS 的 `acquire` 方法。当然，我们看的还是`tryAcquire`方法，在上面的代码中，`tryAcquire`方法调用了父类`Sync` 的 `nonfairTryAcquire`，为什么在父类中呢？

在`ReentrantLock` 的 `tryLock`方法中，也调用了该方法。因为这个方法是快速返回的。该方法不会让等待时间久的线程获取锁。符合 `tryLock` 的设计。

方法实现如下：

```
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
复制代码
```

该方法相比较公平锁的 `tryAcquire`方法，少了一步**判断 AQS 队列中是否有等待的线程**的操作。

他要做的就是直接抢锁，不让给队列里那些等待时间长的。

抢不到再进入队列。等待他的前置节点唤醒他。这个过程是公平的。

**4. 总结**

`ReentrantLock`中的公平锁和非公平锁的区别就在于：调用 `lock` 方法获取锁的时候 `要不要判断 AQS 队列中是否有等待的线程`，公平锁为了让每一个线程都均衡的使用锁，就需要判断，如果有，让给他，非公平锁很霸道，不让不让就不让。

但如果失败了，进入队列了，进会按照 AQS 的逻辑来，整体顺序就是公平的。

还有个注意的地方就是：`ReentrantLock` 的 `tryLock（无超时机制）` 方法使用的非公平策略。符合他的设计。

而 `tryLock(long timeout, TimeUnit unit)` 方法则会根据 Sync 的具体实现来调用。不会冲动的调用 `nonfairTryAcquire` 方法。



**优缺点：**

非公平锁性能高于公平锁性能。首先，在恢复一个被挂起的线程与该线程真正运行之间存在着严重的延迟。而且，非公平锁能更充分的利用cpu的时间片，尽量的减少cpu空闲的状态时间。

**使用场景**

使用场景的话呢，其实还是和他们的属性一一相关，举个栗子：如果业务中线程占用(处理)时间要远长于线程等待，那用非公平锁其实效率并不明显，但是用公平锁会给业务增强很多的可控制性。



#### 11.给我讲讲线程间通信

###### java中常用两种：

1. 通过访问共享变量的方式**(注:需要处理同步问题)**

   **其中方法一有两种实现方法,即
   方法一a)通过内部类实现线程的共享变量
   代码如下:**

   ```JAVA
   public class Innersharethread {     
       public static void main(String[] args) {     
           Mythread mythread = new Mythread();     
           mythread.getThread().start();     
           mythread.getThread().start();     
           mythread.getThread().start();     
           mythread.getThread().start();     
       }     
   }     
   class Mythread {     
       int index = 0;     
       
       private class InnerThread extends Thread {     
           public synchronized void run() {     
               while (true) {     
                   System.out.println(Thread.currentThread().getName()     
                           + "is running and index is " + index++);     
               }     
           }     
       }     
       
       public Thread getThread() {     
           return new InnerThread();     
       }     
   }    
     
   /** 
    * 通过内部类实现线程的共享变量 
    * 
    */  
   public class Innersharethread {  
       public static void main(String[] args) {  
           Mythread mythread = new Mythread();  
           mythread.getThread().start();  
           mythread.getThread().start();  
           mythread.getThread().start();  
           mythread.getThread().start();  
       }  
   }  
   class Mythread {  
       int index = 0;  
     
       private class InnerThread extends Thread {  
           public synchronized void run() {  
               while (true) {  
                   System.out.println(Thread.currentThread().getName()  
                           + "is running and index is " + index++);  
               }  
           }  
       }  
     
       public Thread getThread() {  
           return new InnerThread();  
       }  
   }
   ```

   **b)通过实现Runnable接口实现线程的共享变量
   代码如下：**

   ```JAVA
   public class Interfacaesharethread {
   	public static void main(String[] args) {
   		Mythread mythread = new Mythread();
   		new Thread(mythread).start();
   		new Thread(mythread).start();
   		new Thread(mythread).start();
   		new Thread(mythread).start();
   	}
   }
   
   /* 实现Runnable接口 */
   class Mythread implements Runnable {
   	int index = 0;
   
   	public synchronized void run() {
   		while (true)
   			System.out.println(Thread.currentThread().getName() + "is running and
                           the index is " + index++);
   	}
   }
   
   /**
    * 通过实现Runnable接口实现线程的共享变量
    
    */
   public class Interfacaesharethread {
   	public static void main(String[] args) {
   		Mythread mythread = new Mythread();
   		new Thread(mythread).start();
   		new Thread(mythread).start();
   		new Thread(mythread).start();
   		new Thread(mythread).start();
   	}
   }
   
   /* 实现Runnable接口 */
   class Mythread implements Runnable {
   	int index = 0;
   
   	public synchronized void run() {
   		while (true)
   			System.out.println(Thread.currentThread().getName() + "is running and
                           the index is " + index++);
   	}
   }
   ```

2. 通过管道流

**代码如下：**

```JAVA
public class CommunicateWhitPiping {
	public static void main(String[] args) {
		/**
		 * 创建管道输出流
		 */
		PipedOutputStream pos = new PipedOutputStream();
		/**
		 * 创建管道输入流
		 */
		PipedInputStream pis = new PipedInputStream();
		try {
			/**
			 * 将管道输入流与输出流连接 此过程也可通过重载的构造函数来实现
			 */
			pos.connect(pis);
		} catch (IOException e) {
			e.printStackTrace();
		}
		/**
		 * 创建生产者线程
		 */
		Producer p = new Producer(pos);
		/**
		 * 创建消费者线程
		 */
		Consumer c = new Consumer(pis);
		/**
		 * 启动线程
		 */
		p.start();
		c.start();
	}
}

/**
 * 生产者线程(与一个管道输入流相关联)
 * 
 */
class Producer extends Thread {
	private PipedOutputStream pos;

	public Producer(PipedOutputStream pos) {
		this.pos = pos;
	}

	public void run() {
		int i = 8;
		try {
			pos.write(i);
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
}

/**
 * 消费者线程(与一个管道输入流相关联)
 * 
 */
class Consumer extends Thread {
	private PipedInputStream pis;

	public Consumer(PipedInputStream pis) {
		this.pis = pis;
	}

	public void run() {
		try {
			System.out.println(pis.read());
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
}
```

###### Android 中线程通信与进程通信

线程通信： `Handler`消息队列
进程通信： `binder` 机制，底层用的还是**共享内存**的方式。

#### 12.volatile关键字是如何使用的？原理是什么



 https://juejin.im/post/5a2b53b7f265da432a7b821c#heading-7 

就我理解的而言，被volatile修饰的共享变量，就具有了以下两点特性：

1 . 保证了不同线程对该变量操作的内存可见性;

2 . 禁止指令重排序



volatile 为实例域的同步访问提供了免锁机制，如果声明一个域为 volatile，那么
编译器和虚拟机就直到该域可能被另一个线程并发更新

#### 13.说说使用5个线程去计算一个数组之和的思路



 五个线程交替累加计算数组之和，这种方法其实不如单线程直接累加快，因为交替累加需要前一个线程计算的结果。 

```JAVA
package test;
 
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
 
public class FiveThreadCount {
	private int count=0;
	private int[] arr={1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28};
	private int j=0;
	//定义一个任务，关键点所在
	private class MyThread extends Thread{
		@Override
		public void run() {
			super.run();
				while(j<arr.length)
				{
					synchronized (MyThread.class) {
						if(j>=arr.length){
							return;
						}
						count+=arr[j++];
						try {
							Thread.sleep(100);
						} catch (InterruptedException e) {
							// TODO Auto-generated catch block
							e.printStackTrace();
						}
						System.out.println(Thread.currentThread().getName());
					}
				}
		}
	}
 
	//方法一
	public void test1(){
		for(int i=0;i<5;i++){
			new MyThread().start();
		}
        try {
			Thread.sleep(10000);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
        System.out.println(count);
	}
	//方法二
	public void test2(){
		Thread myThread=new MyThread();
		for(int i=0;i<5;i++){
			new Thread(myThread).start();
		}
        try {
			Thread.sleep(10000);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
        System.out.println(count);
	}
	//方法一的线程池实现版
	public void test3(){
		ExecutorService service=Executors.newCachedThreadPool();
		for(int i=0;i<5;i++){
			service.execute(new MyThread());
		}
        try {
			Thread.sleep(10000);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
        System.out.println(count);
	}
	//方法二的线程池实现版
	public void test4(){
		ExecutorService service=Executors.newCachedThreadPool();
		Thread myThread=new MyThread();
		for(int i=0;i<5;i++){
			service.execute(myThread);
		}
        try {
			Thread.sleep(10000);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
        System.out.println(count);
	}
 
}
```

 上边代码中，用到了sleep方法的原因，sleep(100)是为了让其他线程有时间执行任务，如果不sleep的话，有可能一个线程就全部执行完了。 最后的sleep（10000）是为了等所有线程执行完后，打印最后的计算结果。 

#### 14.谈谈线程阻塞的原因有哪些？

 线程在运行的过程中因为某些原因而发生阻塞，阻塞状态的线程的特点是：该线程放弃CPU的使用，暂停运行，只有等到导致阻塞的原因消除之后才回复运行。或者是被其他的线程中断，该线程也会退出阻塞状态，同时抛出InterruptedException。

        导致阻塞的原因有很多种，大致分为三种来讨论，分别是一般线程中的阻塞，Socket客户端的阻塞，Socket服务器端的阻塞。



一般线程中的阻塞：

        A、线程执行了Thread.sleep(int millsecond);方法，当前线程放弃CPU，睡眠一段时间，然后再恢复执行
    
        B、线程执行一段同步代码，但是尚且无法获得相关的同步锁，只能进入阻塞状态，等到获取了同步锁，才能回复执行。
    
        C、线程执行了一个对象的wait()方法，直接进入阻塞状态，等待其他线程执行notify()或者notifyAll()方法,才可以将其唤醒。
    
        D、线程执行某些IO操作，因为等待相关的资源而进入了阻塞状态。比如说监听system.in，read()方法时，但是尚且没有收到键盘的输入数据才会从read()方法返回回来。进程在远程通信时，在客户程序中，则进入阻塞状态。



Socket客户端的阻塞：

       A、请求与服务器建立连接时，即当线程执行Socket的带参数的构造方法，或执行Socket的connect()方法时，会进入阻塞状态，直到连接成功，此线程才从Socket的构造方法或connect()方法返回
        B、当从Socket输入流读取数据时，在读取足够的数据之前或者到达了输入流的末尾，或者出现了异常，才会从输入流的read()方法返回或者异常中断，会进入阻塞状态。比如说通过BufferedReader类使用readLine()方法时，在没有读出一行数据之前，数据量就不算是足够，会处在阻塞状态下。
    
        C、调用Socket的setSoLinger()方法关闭了Socket延迟，当执行Socket的close方法时，会进入阻塞状态，直到底层Socket发送完所有剩余数据，或者超过了setSoLinger()方法设置的延迟时间，才从close()方法返回。



Socket服务器的阻塞：

        A、线程执行ServerSocket的accept()方法，等待客户的连接，知道接收到客户的连接，才从accept方法中返回一个Socket对象
    
        B、从Socket输入流读取数据时，如果输入流没有足够的数据，就会进入阻塞状态
    
        C、线程向Socket的输出流写入一批数据，可能进入阻塞状态当程序阻塞时，会降低程序的效率，于是人们就希望能引入非阻塞的操作方法。    
    
        所谓非阻塞方法，就是指当线程执行这些方法时，如果操作还没有就绪，就立即返回，不会阻塞着等待操作就绪。Java.nio 提供了这些支持非阻塞通信的类。



      服务器程序用多线程来处理阻塞I/O，尽管能满足同时响应多个客户的需求，但是有以下局限：
    
    (1)Java虚拟机会为每个线程分配独立的堆栈空间，工作线程数目越多，系统开销就越大，而且增加了Java虚拟机调度线程的负担，增加了线程之间同步的复杂性，提高了线程死锁的可能性；
    
    (2)工作线程的许多时间都浪费在阻塞I/O操作上，Java虚拟机需要频繁地转让CPU的使用权，使进入阻塞状态的线程放弃CPU,再把CPU分配给处于可运行状态的线程。  



#### 15.谈谈你对notify,wait的理解以及使用？

1.对于wait()和notify()的理解

对于wait()和notify()的理解，还是要从jdk官方文档中开始，在Object类方法中有：

void notify()
Wakes up a single thread that is waiting on this object’s monitor.
译：唤醒在此对象监视器上等待的单个线程

void notifyAll()
Wakes up all threads that are waiting on this object’s monitor.
译：唤醒在此对象监视器上等待的所有线程

void wait( )
Causes the current thread to wait until another thread invokes the notify() method or the notifyAll( ) method for this object.
译：导致当前的线程等待，直到其他线程调用此对象的notify( ) 方法或 notifyAll( ) 方法

void wait(long timeout)
Causes the current thread to wait until either another thread invokes the notify( ) method or the notifyAll( ) method for this object, or a specified amount of time has elapsed.
译：导致当前的线程等待，直到其他线程调用此对象的notify() 方法或 notifyAll() 方法，或者指定的时间过完。

void wait(long timeout, int nanos)
Causes the current thread to wait until another thread invokes the notify( ) method or the notifyAll( ) method for this object, or some other thread interrupts the current thread, or a certain amount of real time has elapsed.
译：导致当前的线程等待，直到其他线程调用此对象的notify( ) 方法或 notifyAll( ) 方法，或者其他线程打断了当前线程，或者指定的时间过完。

上面是官方文档的简介，下面我们根据官方文档总结一下：

wait( )，notify( )，notifyAll( )都不属于Thread类，而是属于Object基础类，也就是每个对象都有wait( )，notify( )，notifyAll( ) 的功能，因为每个对象都有锁，锁是每个对象的基础，当然操作锁的方法也是最基础了。

当需要调用以上的方法的时候，一定要对竞争资源进行加锁，如果不加锁的话，则会报 IllegalMonitorStateException 异常

当想要调用wait( )进行线程等待时，必须要取得这个锁对象的控制权（对象监视器），一般是放到synchronized(obj)代码中。

在while循环里而不是if语句下使用wait，这样，会在线程暂停恢复后都检查wait的条件，并在条件实际上并未改变的情况下处理唤醒通知

调用obj.wait( )释放了obj的锁，否则其他线程也无法获得obj的锁，也就无法在synchronized(obj){ obj.notify() } 代码段内唤醒A。

notify( )方法只会通知等待队列中的第一个相关线程（不会通知优先级比较高的线程）

notifyAll( )通知所有等待该竞争资源的线程（也不会按照线程的优先级来执行）

假设有三个线程执行了obj.wait( )，那么obj.notifyAll( )则能全部唤醒tread1，thread2，thread3，但是要继续执行obj.wait（）的下一条语句，必须获得obj锁，因此，tread1，thread2，thread3只有一个有机会获得锁继续执行，例如tread1，其余的需要等待thread1释放obj锁之后才能继续执行。

当调用obj.notify/notifyAll后，调用线程依旧持有obj锁，因此，thread1，thread2，thread3虽被唤醒，但是仍无法获得obj锁。直到调用线程退出synchronized块，释放obj锁后，thread1，thread2，thread3中的一个才有机会获得锁继续执行。
 **2.wait和notify简单使用示例** 

```JAVA
public class WaitNotifyTest {

    // 在多线程间共享的对象上使用wait
    private String[] shareObj = { "true" };

    public static void main(String[] args) {
        WaitNotifyTest test = new WaitNotifyTest();
        ThreadWait threadWait1 = test.new ThreadWait("wait thread1");
        threadWait1.setPriority(2);
        ThreadWait threadWait2 = test.new ThreadWait("wait thread2");
        threadWait2.setPriority(3);
        ThreadWait threadWait3 = test.new ThreadWait("wait thread3");
        threadWait3.setPriority(4);

        ThreadNotify threadNotify = test.new ThreadNotify("notify thread");

        threadNotify.start();
        threadWait1.start();
        threadWait2.start();
        threadWait3.start();
    }

    class ThreadWait extends Thread {

        public ThreadWait(String name){
            super(name);
        }

        public void run() {
            synchronized (shareObj) {
                while ("true".equals(shareObj[0])) {
                    System.out.println("线程"+ this.getName() + "开始等待");
                    long startTime = System.currentTimeMillis();
                    try {
                        shareObj.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    long endTime = System.currentTimeMillis();
                    System.out.println("线程" + this.getName() 
                            + "等待时间为：" + (endTime - startTime));
                }
            }
            System.out.println("线程" + getName() + "等待结束");
        }
    }

    class ThreadNotify extends Thread {

        public ThreadNotify(String name){
            super(name);
        }


        public void run() {
            try {
                // 给等待线程等待时间
                sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized (shareObj) {
                System.out.println("线程" + this.getName() + "开始准备通知");
                shareObj[0] = "false";
                shareObj.notifyAll();
                System.out.println("线程" + this.getName() + "通知结束");
            }
            System.out.println("线程" + this.getName() + "运行结束");
        }
    }
}

/*
线程wait thread1开始等待
线程wait thread3开始等待
线程wait thread2开始等待
线程notify thread开始准备通知
线程notify thread通知结束
线程notify thread运行结束
线程wait thread2等待时间为：2998
线程wait thread2等待结束
线程wait thread3等待时间为：2998
线程wait thread3等待结束
线程wait thread1等待时间为：3000
线程wait thread1等待结束

*/
```

​                                              





#### 15.2 notify与notifyAll的联系与区别

**先说两个概念：锁池和等待池**

- **锁池**:假设线程A已经拥有了某个对象(注意:不是类)的锁，而其它的线程想要调用这个对象的某个synchronized方法(或者synchronized块)，由于这些线程在进入对象的synchronized方法之前必须先获得该对象的锁的拥有权，但是该对象的锁目前正被线程A拥有，所以这些线程就进入了该对象的锁池中。
- **等待池**:假设一个线程A调用了某个对象的wait()方法，线程A就会释放该对象的锁后，进入到了该对象的等待池中

> Reference：[java中的锁池和等待池 ](https://link.zhihu.com/?target=http%3A//blog.csdn.net/emailed/article/details/4689220)

**然后再来说notify和notifyAll的区别**

-  如果线程调用了对象的 wait()方法，那么线程便会处于该对象的**等待池**中，等待池中的线程**不会去竞争该对象的锁**。
- 当有线程调用了对象的 **notifyAll**()方法（唤醒所有 wait 线程）或 **notify**()方法（只随机唤醒一个 wait 线程），被唤醒的的线程便会进入该对象的锁池中，锁池中的线程会去竞争该对象锁。也就是说，调用了notify后只要一个线程会由等待池进入锁池，而notifyAll会将该对象等待池内的所有线程移动到锁池中，等待锁竞争
- 优先级高的线程竞争到对象锁的概率大，假若某线程没有竞争到该对象锁，它**还会留在锁池中**，唯有线程再次调用 wait()方法，它才会重新回到等待池中。而竞争到对象锁的线程则继续往下执行，直到执行完了 synchronized 代码块，它会释放掉该对象锁，这时锁池中的线程会继续竞争该对象锁。

> Reference：[线程间协作：wait、notify、notifyAll ](https://link.zhihu.com/?target=http%3A//wiki.jikexueyuan.com/project/java-concurrency/collaboration-between-threads.html)

综上，所谓唤醒线程，另一种解释可以说是将线程由等待池移动到锁池，notifyAll调用后，会将全部线程由等待池移到锁池，然后参与锁的竞争，竞争成功则继续执行，如果不成功则留在锁池等待锁被释放后再次参与竞争。而notify只会唤醒一个线程。

有了这些理论基础，后面的notify可能会导致死锁，而notifyAll则不会的例子也就好解释了



 https://blog.csdn.net/djzhao/article/details/79410229 

**生产者和消费者：**

 https://juejin.im/post/5c930141e51d450ad30e43d3#heading-1 

#### 16.你觉得Lock和Synchronized的区别是什么？

**总的来说，Lock和synchronized有以下几点不同：**

(1) Lock是一个接口，是JDK层面的实现；而synchronized是Java中的关键字，是Java的内置特性，是JVM层面的实现；

(2) synchronized 在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生；而Lock在发生异常时，如果没有主动通过unLock()去释放锁，则很可能造成死锁现象，因此使用Lock时需要在finally块中释放锁；

(3) Lock 可以让等待锁的线程响应中断，而使用synchronized时，等待的线程会一直等待下去，不能够响应中断；

(4) 通过Lock可以知道有没有成功获取锁，而synchronized却无法办到；

(5) Lock可以提高多个线程进行读操作的效率。

在性能上来说，如果竞争资源不激烈，两者的性能是差不多的。而当竞争资源非常激烈时（即有大量线程同时竞争），此时Lock的性能要远远优于synchronized。所以说，在具体使用时要根据适当情况选择。

**性能对比：**

JDK1.5中，synchronized是性能低效的。因为这是一个重量级操作，它对性能最大的影响是阻塞的是实现，挂起线程和恢复线程的操作都需要转入内核态中完成，这些操作给系统的并发性带来了很大的压力。相比之下使用Java提供的Lock对象，性能更高一些。多线程环境下，synchronized的吞吐量下降的非常严重，而ReentrankLock则能基本保持在同一个比较稳定的水平上。

到了JDK1.6，发生了变化，对synchronize加入了很多优化措施，有自适应自旋，锁消除，锁粗化，轻量级锁，偏向锁等等。导致在JDK1.6上synchronize的性能并不比Lock差。官方也表示，他们也更支持synchronize，在未来的版本中还有优化余地，所以还是提倡在synchronized能实现需求的情况下，优先考虑使用synchronized来进行同步。



还是想提一下这2中机制的具体区别。据我所知，synchronized原始采用的是CPU悲观锁机制，即线程获得的是独占锁。独占锁意味着其他线程只能依靠阻塞来等待线程释放锁。而在CPU转换线程阻塞时会引起线程上下文切换，当有很多线程竞争锁的时候，会引起CPU频繁的上下文切换导致效率很低。



而Lock用的是乐观锁方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。乐观锁实现的机制就是CAS操作（Compare and Swap）。我们可以进一步研究ReentrantLock的源代码，会发现其中比较重要的获得锁的一个方法是compareAndSetState。这里其实就是调用的CPU提供的特殊指令。



现代的CPU提供了指令，可以自动更新共享数据，而且能够检测到其他线程的干扰，而 compareAndSet() 就用这些代替了锁定。这个算法称作非阻塞算法，意思是一个线程的失败或者挂起不应该影响其他线程的失败或挂起的算法。



我也只是了解到这一步，具体到CPU的算法如果感兴趣的读者还可以在查阅下，如果有更好的解释也可以给我留言，我也学习下。



三、synchronized和lock用途区别



synchronized原语和ReentrantLock在一般情况下没有什么区别，但是在非常复杂的同步应用中，请考虑使用ReentrantLock，特别是遇到下面2种需求的时候。



1.某个线程在等待一个锁的控制权的这段时间需要中断

2.需要分开处理一些wait-notify，ReentrantLock里面的Condition应用，能够控制notify哪个线程

3.具有公平锁功能，每个到来的线程都将排队等候






![image-20200208162315542](C:\Users\Lenovo\Desktop\image-20200208162315542.png)



对于Lock的详细讲解

 https://blog.csdn.net/u012403290/article/details/64910926 

#### 17.谈谈你对ReentrantLock的认识





实现了Lock接口，可重入锁，内部定义了公平锁与非公平锁。默认为非公平锁：

```
public ReentrantLock() {  
  sync = new NonfairSync();  
} 
复制代码
```

可以手动设置为公平锁：

```
public ReentrantLock(boolean fair) {  
  sync = fair ? new FairSync() : new NonfairSync();  
}  
```

由于ReentrantLock是java.util.concurrent包下提供的一套互斥锁，相比Synchronized，ReentrantLock类提供了一些高级功能，主要有以下3项：

        1.等待可中断，持有锁的线程长期不释放的时候，正在等待的线程可以选择放弃等待，这相当于Synchronized来说可以避免出现死锁的情况。通过lock.lockInterruptibly()来实现这个机制。
    
        2.公平锁，多个线程等待同一个锁时，必须按照申请锁的时间顺序获得锁，Synchronized锁非公平锁，ReentrantLock默认的构造函数是创建的非公平锁，可以通过参数true设为公平锁，但公平锁表现的性能不是很好。

公平锁、非公平锁的创建方式：

//创建一个非公平锁，默认是非公平锁
Lock lock = new ReentrantLock();
Lock lock = new ReentrantLock(false);

//创建一个公平锁，构造传参true
Lock lock = new ReentrantLock(true);
        3.锁绑定多个条件，一个ReentrantLock对象可以同时绑定对个对象。ReenTrantLock提供了一个Condition（条件）类，用来实现分组唤醒需要唤醒的线程们，而不是像synchronized要么随机唤醒一个线程要么唤醒全部线程。

ReenTrantLock实现的原理：
之后还会总结一篇ReenTrantLock相关的原理底层原理分析，简单来说，ReenTrantLock的实现是一种自旋锁，通过循环调用CAS操作来实现加锁。它的性能比较好也是因为避免了使线程进入内核态的阻塞状态。想尽办法避免线程进入内核的阻塞状态是我们去分析和理解锁设计的关键钥匙。

什么情况下使用ReenTrantLock：
答案是，如果你需要实现ReenTrantLock的三个独有功能时。

ReentrantLock的用法如下：



```JAVA
public class SynDemo{
    public static void main(String[] arg){
        Runnable t1=new MyThread();
        new Thread(t1,"t1").start();
        new Thread(t1,"t2").start();
    }
}
```




```JAVA
class MyThread implements Runnable {
    private Lock lock=new ReentrantLock();
    public void run() {
            lock.lock();
            try{
                for(int i=0;i<5;i++)
                    System.out.println(Thread.currentThread().getName()+":"+i);
            }finally{
                lock.unlock();
            }
    }
}
    
```


对ReentrantLock的源码分析这有一篇很好的文章

http://www.blogjava.net/zhanglongsr/articles/356782.html

 https://juejin.im/post/5aeb0a8b518825673a2066f0 

#### 18.调用run()和start()的区别？

##### start() :

它的作用是**启动一个新线程。**
 通过start()方法来启动的新线程，处于就绪（可运行）状态，并没有运行，一旦得到cpu时间片，就开始执行相应线程的run()方法，这里方法**run()称为线程体，它包含了要执行的这个线程的内容**，run方法运行结束，此线程随即终止。start()不能被重复调用。**用start方法来启动线程**，真正实现了多线程运行，即无需等待某个线程的run方法体代码执行完毕就直接继续执行下面的代码。这里无需等待run方法执行完毕，即可继续执行下面的代码，即进行了线程切换。



故而start()方法是Thread类的方法用来异步启动一个线程，然后主线程立即返回。盖其东的线程不会马上运行，会放到等待队列中等待CPU调度，只有线程真正被CPU调度的时候才会执行

所以start方法只是一个表示线程为就绪状态的一个附加方法，start源码：



```java
//start0()是一个本地的native方法
public synchronized void start() {
    if (threadStatus != 0)
        throw new IllegalThreadStateException();

    group.add(this);

    boolean started = false;
    try {
        start0();
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {
            /* do nothing. If start0 threw a Throwable then
              it will be passed up the call stack */
        }
    }
}
//start()方法被标志为synchronized，即为了防止多次启动一个同步操作
```



##### run()   :

run()就和普通的成员方法一样，**可以被重复调用。**程序还是要顺序执行，
 如果直接调用run方法，并不会启动新线程！程序中依然只有主线程这一个线程，其程序执行路径还是只有一条，还是要顺序执行，还是要等待run方法体执行完毕后才可继续执行下面的代码，这样就没有达到多线程的目的。
 总结：**调用start方法方可启动线程，而run方法只是thread的一个普通方法调用，还是在主线程里执行。**



```csharp
public class Test {
    static void pong(){
        System.out.print("pong");
    }
    public static void main(String[] args) {
        Thread t=new Thread(){
            public void run(){
                pong();
            }
        };
        t.run();
        System.out.print("ping");       
    }
}
运行结果：
pongping
```



**从线程状态而言：**

- 第一是创建状态。在生成线程对象，并没有调用该对象的start方法，这是线程处于创建状态。
- 第二是就绪状态。当调用了线程对象的start方法之后，该线程就进入了就绪状态，但是此时线程调度程序还没有把该线程设置为当前线程，此时处于就绪状态。在线程运行之后，从等待或者睡眠中回来之后，也会处于就绪状态。
- 第三是运行状态。线程调度程序将处于就绪状态的线程设置为当前线程，此时线程就进入了运行状态，开始运行run函数当中的代码。
- 第四是阻塞状态。线程正在运行的时候，被暂停，通常是为了等待某个时间的发生(比如说某项资源就绪)之后再继续运行。sleep,suspend，wait等方法都可以导致线程阻塞。
- 第五是死亡状态。如果一个线程的run方法执行结束或者调用stop方法后，该线程就会死亡。对于已经死亡的线程，无法再使用start方法令其进入就绪。

**从实现并启动线程的方法：**

- 1、写一个类继承自Thread类，重写run方法。用start方法启动线程
  2、写一个类实现Runnable接口，实现run方法。用new Thread(Runnable target).start()方法来启动

  **多线程原理**：相当于玩游戏机，只有一个游戏机（cpu），可是有很多人要玩，于是，start是排队！等CPU选中你就是轮到你，你就run（），当CPU的运行的时间片执行完，这个线程就继续排队，等待下一次的run（）。

  调用start（）后，线程会被放到等待队列，等待CPU调度，并不一定要马上开始执行，只是将这个线程置于可动行状态。然后通过JVM，线程Thread会调用run（）方法，执行本线程的线程体。先调用start后调用run，这么麻烦，为了不直接调用run？就是为了实现多线程的优点，没这个start不行。

  - **1.start（）方法来启动线程，真正实现了多线程运行**。这时无需等待run方法体代码执行完毕，可以直接继续执行下面的代码；通过调用Thread类的start()方法来启动一个线程， 这时此线程是处于就绪状态， 并没有运行。 然后通过此Thread类调用方法run()来完成其运行操作的， 这里方法run()称为线程体，它包含了要执行的这个线程的内容， Run方法运行结束， 此线程终止。然后CPU再调度其它线程。

  - **2.run（）方法当作普通方法的方式调用。**程序还是要顺序执行，要等待run方法体执行完毕后，才可继续执行下面的代码； 程序中只有主线程——这一个线程， 其程序执行路径还是只有一条， 这样就没有达到写线程的目的。
    记住：**多线程就是分时利用CPU，宏观上让所有线程一起执行 ，也叫并发**

    ```JAVA
    public class Test {
    	public static void main(String[] args) {
    		Runner1 runner1 = new Runner1();
    		Runner2 runner2 = new Runner2();
    //		Thread(Runnable target) 分配新的 Thread 对象。
    		Thread thread1 = new Thread(runner1);
    		Thread thread2 = new Thread(runner2);
    //		thread1.start();
    //		thread2.start();
    		thread1.run();
    		thread2.run();
    	}
    }
     
    class Runner1 implements Runnable { // 实现了Runnable接口，jdk就知道这个类是一个线程
    	public void run() {
    		for (int i = 0; i < 100; i++) {
    			System.out.println("进入Runner1运行状态——————————" + i);
    		}
    	}
    }
     
    class Runner2 implements Runnable { // 实现了Runnable接口，jdk就知道这个类是一个线程
    	public void run() {
    		for (int i = 0; i < 100; i++) {
    			System.out.println("进入Runner2运行状态==========" + i);
    		}
    	}
    }
    ```

    

总结一下：

1. start() 可以启动一个新线程，run()不能
2. start()不能被重复调用，run()可以
3. start()中的run代码可以不执行完就继续执行下面的代码，即进行了线程切换。直接调用run方法必须等待其代码全部执行完才能继续执行下面的代码。
4. start() 实现了多线程，run()没有实现多线程。

5，如果只是用一个run()方法，就等于调用了一个普通的同步方法，打不动多线程运行的异步执行

#### 19.transient关键字的用法 & 作用 & 原理。

**一、初识transient关键字**

其实这个关键字的作用很好理解，就是简单的一句话：将不需要序列化的属性前添加关键字transient，序列化对象的时候，这个属性就不会被序列化。

概念也很好理解，下面使用代码去验证一下：

```java
public class User implements Serializable {
    private static final long serialVersionUID = 123456L;
    private transient int age;
    private String name;    
    
    //getter和setter方法
    //toString方法
}
```

然后我们在Test中去验证一下：

```java
public class Test {
    public static void main(String[] args) throws Exception, IOException {
        SerializeUser();
        DeSerializeUser();
    }
    //序列化
    private static void SerializeUser() throws FileNotFoundException, IOException, ClassNotFoundException {
        User user = new User();
        user.setName("Java的架构师技术栈");
        user.setAge(24);
        ObjectOutputStream oos = 
        new ObjectOutputStream(new FileOutputStream("G://Test/template"));
        oos.writeObject(user);
        oos.close();
        System.out.println("添加了transient关键字序列化：age=  "+user.getAge());
    }
    //反序列化
    private static void DeSerializeUser() throws IOException, ClassNotFoundException {
        File file = new File("G://Test/template");
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file));
        User newUser = (User)ois.readObject();
        System.out.println("添加了transient关键字反序列化：age=  "+newUser.getAge());
    }
}

```

从上面可以看出，在序列化SerializeUser方法中，首先创建一个序列化user类，然后将其写入到G://Test/template路径中。在反序列化DeSerializeUser方法中，首先创建一个File，然后读取G://Test/template路径中的数据。

这就是序列化和反序列化的基本实现，而且我们看一下结果，也就是被transient关键字修饰的age属性是否被序列化。

![img](https://pic1.zhimg.com/80/v2-73ca66950a6a57a516201d3a315c7da8_hd.jpg)

从上面的这张图可以看出，age属性变为了0，说明被transient关键字修饰之后没有被序列化。

**二、深入分析transient关键字**

为了更加深入的去分析transient关键字，我们需要带着几个问题去解读：

（1）**transient底层实现的原理是什么？**

（2）**被transient关键字修饰过得变量真的不能被序列化嘛？**

（3）**静态变量能被序列化吗？被transient关键字修饰之后呢？**

带着这些问题一个一个来解决：

**1、transient底层实现原理是什么？**

java的serialization提供了一个非常棒的存储对象状态的机制，说白了serialization就是把对象的状态存储到硬盘上 去，等需要的时候就可以再把它读出来使用。有些时候像银行卡号这些字段是不希望在网络上传输的，transient的作用就是把这个字段的生命周期**仅存于调用者的内存中而不会写到磁盘里持久化**，意思是transient修饰的age字段，他的生命周期仅仅在内存中，不会被写到磁盘中。

**2、被transient关键字修饰过得变量真的不能被序列化嘛？**

想要解决这个问题，首先还要再重提一下对象的序列化方式：

Java序列化提供两种方式。

- 一种是实现Serializable接口
- 另一种是实现Exteranlizable接口。 需要重写writeExternal和readExternal方法，它的效率比Serializable高一些，并且可以决定哪些属性需要序列化（即使是transient修饰的），但是对大量对象，或者重复对象，则效率低。

从上面的这两种序列化方式，我想你已经看到了，使用**Exteranlizable**接口实现序列化时，我们自己指定那些属性是需要序列化的，即使是transient修饰的。下面就验证一下

首先我们定义User1类：这个类是被Externalizable接口修饰的

```java
public class User1 implements Externalizable{   
    private transient String name;
    //getter、setter、toString方法
    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        out.writeObject(name);
    }
    @Override
    public void readExternal(ObjectInput in) throws IOException {
        name = (String) in.readObject();
    }
}
```

然后我们就可以测试了

```java
public class Test {
    public static void main(String[] args) throws Exception, IOException {
        SerializeUser();
        DeSerializeUser();
    }
    //序列化
    private static void SerializeUser() throws FileNotFoundException, IOException {
        User1 user = new User1();
        user.setName("Java的架构师技术栈");
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("G://template"));
        oos.writeObject(user);
        oos.close();
        System.out.println("使用Externalizable接口，添加了transient关键字序列化之前："+user.toString());
    }
    //反序列化
    private static void DeSerializeUser() throws IOException, ClassNotFoundException {
        File file = new File("G://template");
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file));
        User1 newUser = (User1)ois.readObject();
        System.out.println("使用Externalizable接口，添加了transient关键字序列化之后："+newUser.toString());
    }
}
```

上面，代码分了两个方法，一个是序列化，一个是反序列化。里面的代码和一开始给出的差不多，只不过，User1里面少了age这个属性。

然后看一下结果：

![img](https://pic2.zhimg.com/80/v2-505b83af11a0714e89243703ac1838f5_hd.jpg)

结果基本上验证了我们的猜想，也就是说，实现了Externalizable接口，哪一个属性被序列化使我们手动去指定的，即使是transient关键字修饰也不起作用。

**3、静态变量能被序列化吗？没被transient关键字修饰之后呢？**

这个我可以提前先告诉结果，静态变量是不会被序列化的，即使没有transient关键字修饰。下面去验证一下，然后再解释原因。

首先，在User类中对age属性添加transient关键字和static关键字修饰。

然后，在Test类中去测试

```java
        //序列化
        private static void SerializeUser() throws 
        FileNotFoundException, IOException, ClassNotFoundException {
            User user = new User();
            user.setName("Java的架构师技术栈");
            //序列化之前静态变量age年龄是24.
            user.setAge(24);
            ObjectOutputStream oos = 
            new ObjectOutputStream(new FileOutputStream("G://template"));
            oos.writeObject(user);
            oos.close();
            //再读取，通过modifyUser.getAge()打印新的值
            System.out.println("static、transient关键字修饰age之前："+user.getAge());
            
            //现在把年龄改成18
            user.setAge(18);
            ObjectInputStream oin = 
            new ObjectInputStream(new FileInputStream( "G://template"));
            User modifyUser = (User) oin.readObject();
            oin.close();

            //再读取，通过modifyUser.getAge()打印新的值
            System.out.println("改变age之后："+modifyUser.getAge());
        }
```

最后，测试一下，看看结果

![img](https://pic3.zhimg.com/80/v2-0ea7a4f099baafe4356046ee58d1029a_hd.jpg)

结果已经很明显了。现在解释一下，为什么会是这样，其实在前面已经提到过了。因为静态变量在全局区,本来流里面就没有写入静态变量,我打印静态变量当然会去全局区查找,而我们的序列化是写到磁盘上的，所以JVM查找这个静态变量的值，是从全局区查找的，而不是磁盘上。user.setAge(18);年龄改成18之后，被写到了全局区，其实就是方法区，只不过被所有的线程共享的一块空间。因此可以总结一句话：

**静态变量不管是不是transient关键字修饰，都不会被序列化**

**三、transient关键字总结**

java 的transient关键字为我们提供了便利，你只需要实现Serilizable接口，将不需要序列化的属性前添加关键字transient，序列化对象的时候，这个属性就不会序列化到指定的目的地中。像银行卡、密码等等这些数据。这个需要根据业务情况了。

#### 20.ThreadPoolExecutor的工作策略有哪些？

线程池的工作原理：线程池可以减少创建和销毁线程的次数，从而减少系统资源
的消耗，当一个任务提交到线程池时
a. 首先判断核心线程池中的线程是否已经满了，如果没满，则创建一个核心线
程执行任务，否则进入下一步 b. 判断工作队列是否已满，没有满则加入工作队列，否则执行下一步 c. 判断线程数是否达到了最大值，如果不是，则创建非核心线程执行任务，否
则执行饱和策略，默认抛出异常

**ThreadPoolExecutor执行任务时会遵循如下规则 **

-  如果线程池中的线程数量未达到核心线程的数量，那么会直接启动一个核心线程来执行任务。 
-  如果线程池中的线程数量已经达到或则超过核心线程的数量，那么任务会被插入任务队列中排队等待执行。 
- 如果在第2点无法将任务插入到任务队列中，这往往是由于任
  务队列已满，这个时候如果在线程数量未达到线程池规定的最
  大值，那么会立刻启动一个非核心线程来执行任务。 
-  如果第3点中线程数量已经达到线程池规定的最大值，那么就
  拒绝执行此任务，ThreadPoolExecutor会调用RejectedExecutionHandler的rejectedExecution方法来通知调用者。

#### 21.ThreadLocal了解吗？说说原理。

ThreadLocal是一个本地线程副本变量工具类，各个线程都拥有一份线程私有的数据，线程之间的变量互不干扰，在高并发场景下，可以实现无状态的调用。

ThreadLocal提供了线程安全的另一种思路，我们平常说的线程安全主要是保证共享数据的并发访问问题，通过sychronized锁或者CAS无锁策略来保证数据的一致性。

**ThreadLocal结构图：**

 ![640?wx_fmt=jpeg](https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_jpg/LnN03EBxu3yvLtHPwnjIq95KZqFLaicbxfOuVicibWzv4Tqmy5gaYD2RAGGgRjNIAmk0mPQke0QibICLxt3ZibaQ4KA/640?wx_fmt=jpeg) 

ThreadLocal是一个关于创建线程局部变量的类。使用场景如
下所示： 

- 实现单个线程单例以及单个线程上下文信息存储，比如
  交易id等。 
- 实现线程安全，非线程安全的对象使用ThreadLocal之
  后就会变得线程安全，因为每个线程都会有一个对应的
  实例。 承载一些线程相关的数据，避免在方法中来回传
  递参数。 
-  当需要使用多线程时，有个变量恰巧不需要共享，此时就不必使用synchronized这么麻烦的关键字来锁住，每个线程都相当于在堆内存中开辟一个空间，线程中带有对共享变量的缓冲
  区，通过缓冲区将堆内存中的共享变量进行读取和操作，
  ThreadLocal相当于线程内的内存，一个局部变量。每次可以
  对线程自身的数据读取和操作，并不需要通过缓冲区与 主内存中的变量进行交互。并不会像synchronized那样修改主内存的数据，再将主内存的数据复制到线程内的工作内存。ThreadLocal可以让线程独占资源，存储于线程内部，避免线
  程堵塞造成CPU吞吐下降。 
-  在每个Thread中包含一个ThreadLocalMap，ThreadLocalMap的key是ThreadLocal的对象，value是独享数据。 

#### 22.权衡多线程的性能

**多线程的优点：** 

- 方便高效的内存共享 - 多进程下内存共享比较不便，且
  会抵消掉多进程编程的好处 
- 较轻的上下文切换开销 - 不用切换地址空间，不用更改
  CR3寄存器，不用清空TLB 
- 线程上的任务执行完后自动销毁 
  
  

**多线程的缺点： **

-  开启线程需要占用一定的内存空间(默认情况下,每一个
  线程都占512KB) 

- 如果开启大量的线程,会占用大量的内存空间,降低程序的
  性能 

-  线程越多,cpu在调用线程上的开销就越大 

-  程序设计更加复杂,比如线程间的通信、多线程的数据共
  享 

  

 综上得出，多线程不一定能提高效率，在内存空间紧张的情况
下反而是一种负担，因此在日常开发中，应尽量 

- 不要频繁创建，销毁线程，使用线程池 
- 减少线程间同步和通信（最为关键） 
-  避免需要频繁共享写的数据 
-  合理安排共享数据结构，避免伪共享（false sharing） 
-  使用非阻塞数据结构/算法 
-  避免可能产生可伸缩性问题的系统调用（比如mmap） 
-  避免产生大量缺页异常，尽量使用Huge Page 
-  可以的话使用用户态轻量级线程代替内核线程 

#### 23.如何理解同步和异步，阻塞和非阻塞

作者：严肃





“阻塞”与"非阻塞"与"同步"与“异步"不能简单的从字面理解，提供一个从分布式系统角度的回答。
**1.同步与异步**
同步和异步关注的是**消息通信机制** (synchronous communication/ asynchronous communication)
所谓同步，就是在发出一个*调用*时，在没有得到结果之前，该*调用*就不返回。但是一旦调用返回，就得到返回值了。
换句话说，就是由*调用者*主动等待这个*调用*的结果。

而异步则是相反，***调用\*在发出之后，这个调用就直接返回了，所以没有返回结果**。换句话说，当一个异步过程调用发出后，调用者不会立刻得到结果。而是在*调用*发出后，*被调用者*通过状态、通知来通知调用者，或通过回调函数处理这个调用。

典型的异步编程模型比如Node.js

举个通俗的例子：
你打电话问书店老板有没有《分布式系统》这本书，如果是同步通信机制，书店老板会说，你稍等，”我查一下"，然后开始查啊查，等查好了（可能是5秒，也可能是一天）告诉你结果（返回结果）。
而异步通信机制，书店老板直接告诉你我查一下啊，查好了打电话给你，然后直接挂电话了（不返回结果）。然后查好了，他会主动打电话给你。在这里老板通过“回电”这种方式来回调。

\2. 阻塞与非阻塞
阻塞和非阻塞关注的是**程序在等待调用结果（消息，返回值）时的状态.**

阻塞调用是指调用结果返回之前，当前线程会被挂起。调用线程只有在得到结果之后才会返回。
非阻塞调用指在不能立刻得到结果之前，该调用不会阻塞当前线程。

还是上面的例子，
你打电话问书店老板有没有《分布式系统》这本书，你如果是阻塞式调用，你会一直把自己“挂起”，直到得到这本书有没有的结果，如果是非阻塞式调用，你不管老板有没有告诉你，你自己先一边去玩了， 当然你也要偶尔过几分钟check一下老板有没有返回结果。
在这里阻塞与非阻塞与是否同步异步无关。跟老板通过什么方式回答你结果无关。作者：严肃





“阻塞”与"非阻塞"与"同步"与“异步"不能简单的从字面理解，提供一个从分布式系统角度的回答。
**1.同步与异步**
同步和异步关注的是**消息通信机制** (synchronous communication/ asynchronous communication)
所谓同步，就是在发出一个*调用*时，在没有得到结果之前，该*调用*就不返回。但是一旦调用返回，就得到返回值了。
换句话说，就是由*调用者*主动等待这个*调用*的结果。

而异步则是相反，***调用\*在发出之后，这个调用就直接返回了，所以没有返回结果**。换句话说，当一个异步过程调用发出后，调用者不会立刻得到结果。而是在*调用*发出后，*被调用者*通过状态、通知来通知调用者，或通过回调函数处理这个调用。

典型的异步编程模型比如Node.js

举个通俗的例子：
你打电话问书店老板有没有《分布式系统》这本书，如果是同步通信机制，书店老板会说，你稍等，”我查一下"，然后开始查啊查，等查好了（可能是5秒，也可能是一天）告诉你结果（返回结果）。
而异步通信机制，书店老板直接告诉你我查一下啊，查好了打电话给你，然后直接挂电话了（不返回结果）。然后查好了，他会主动打电话给你。在这里老板通过“回电”这种方式来回调。

\2. 阻塞与非阻塞
阻塞和非阻塞关注的是**程序在等待调用结果（消息，返回值）时的状态.**

阻塞调用是指调用结果返回之前，当前线程会被挂起。调用线程只有在得到结果之后才会返回。
非阻塞调用指在不能立刻得到结果之前，该调用不会阻塞当前线程。

还是上面的例子，
你打电话问书店老板有没有《分布式系统》这本书，你如果是阻塞式调用，你会一直把自己“挂起”，直到得到这本书有没有的结果，如果是非阻塞式调用，你不管老板有没有告诉你，你自己先一边去玩了， 当然你也要偶尔过几分钟check一下老板有没有返回结果。
在这里阻塞与非阻塞与是否同步异步无关。跟老板通过什么方式回答你结果无关。

#### 25.比较一下线程和协程

1、进程

进程是具有一定独立功能的程序关于某个数据集合上的一次运行活动,进程是系统进行资源分配和调度的一个独立单位。每个进程都有自己的独立内存空间，不同进程通过进程间通信来通信。由于进程比较重量，占据独立的内存，所以上下文进程间的切换开销（栈、寄存器、虚拟内存、文件句柄等）比较大，但相对比较稳定安全。

　　2、线程

线程是进程的一个实体,是CPU调度和分派的基本单位,它是比进程更小的能独立运行的基本单位.线程自己基本上不拥有系统资源,只拥有一点在运行中必不可少的资源(如程序计数器,一组寄存器和栈),但是它可与同属一个进程的其他的线程共享进程所拥有的全部资源。线程间通信主要通过共享内存，上下文切换很快，资源开销较少，但相比进程不够稳定容易丢失数据。

　　3、协程

**协程是一种用户态的轻量级线程，**协程的调度完全由用户控制。协程拥有自己的寄存器上下文和栈。协程调度切换时，将寄存器上下文和栈保存到其他地方，在切回来的时候，恢复先前保存的寄存器上下文和栈，直接操作栈则基本没有内核切换的开销，可以不加锁的访问全局变量，所以上下文的切换非常快。

 

二、区别：

　　1、进程多与线程比较

线程是指进程内的一个执行单元,也是进程内的可调度实体。线程与进程的区别:
1) 地址空间:线程是进程内的一个执行单元，进程内至少有一个线程，它们共享进程的地址空间，而进程有自己独立的地址空间
2) 资源拥有:进程是资源分配和拥有的单位,同一个进程内的线程共享进程的资源
3) 线程是处理器调度的基本单位,但进程不是
4) 二者均可并发执行

5) 每个独立的线程有一个程序运行的入口、顺序执行序列和程序的出口，但是线程不能够独立执行，必须依存在应用程序中，由应用程序提供多个线程执行控制

　　2、协程多与线程进行比较

 ***协程与线程主要区别是它将不再被内核调度，而是交给了程序自己而线程是将自己交给内核调度，所以也不难理解golang中调度器的存在***。所以我们可以看出，协程的概念并不是与线程对应的，应该说和函数调用 call/return对应（也不难理解为什么会把golang中的goruntine当作一个以函数为单位的执行单元）。它们的区别在于协程允许一个函数有多个入口、出口（逻辑上的），并且在切换到另一个函数执行时，允许使用一个新的context(包括调用栈）。正是有了这个机制基础，再加上CPU支持了保护模式，操作系统就可以接着实现进程、线程了。 

1) 一个线程可以多个协程，一个进程也可以单独拥有多个协程，这样python中则能使用多核CPU。

2) 线程进程都是同步机制，而协程则是异步

3) 协程能保留上一次调用时的状态，每次过程重入时，就相当于进入上一次调用的状态

#### 26.从源码角度讲讲你对Thread类中run方法的理解

#### 启动线程

```
public synchronized void start() {
        //如果不是刚创建的线程，抛出异常
        if (threadStatus != 0)
            throw new IllegalThreadStateException();

        //通知线程组，当前线程即将启动，线程组当前启动线程数+1，未启动线程数-1
        group.add(this);
        
        //启动标识
        boolean started = false;
        try {
            //直接调用本地方法启动线程
            start0();
            //设置启动标识为启动成功
            started = true;
        } finally {
            try {
                //如果启动呢失败
                if (!started) {
                    //线程组内部移除当前启动的线程数量-1，同时启动失败的线程数量+1
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                /* do nothing. If start0 threw a Throwable then
                  it will be passed up the call stack */
            }
        }
    }
```

我们正常的启动线程都是调用`Thread`的`start()`方法，然后`Java`虚拟机内部会去调用`Thred`的`run`方法，可以看到`Thread`类也是实现`Runnable`接口，重写了`run`方法的：

```JAVA
 @Override
    public void run() {
        //当前执行任务的Runnable对象不为空，则调用其run方法
        if (target != null) {
            target.run();
        }
    }
```

#### 27.谈谈Java内存模型

java内存模型(JMM)是线程间通信的控制机制.JMM定义了主内存和线程之间抽象关系。
线程之间的共享变量存储在主内存（main memory）中，每个线程都有一个私有的本地
内存（local memory），本地内存中存储了该线程以读/写共享变量的副本。本地内存是
JMM的一个抽象概念，并不真实存在。它涵盖了缓存，写缓冲区，寄存器以及其他的硬
件和编译器优化。Java内存模型的抽象示意图如下：


从上图来看，线程A与线程B之间如要通信的话，必须要经历下面2个步骤： 

1. 首先，线程A把本地内存A中更新过的共享变量刷新到主内存中去。 
2. 然后，线程B到主内存中去读取线程A之前已更新过的共享变量

#### 28.两次调用Thread对象的start方法会发生什么？为什么？



Java的线程是不允许启动两次的，第二次调用必然会抛出IllegalThreadStateException，这是一种运行时异常，多次调用start被认为是编程错误。

关于线程生命周期的不同状态，在Java 5以后，线程状态被明确定义在其公共内部枚举类型java.lang.Thread.State中，分别是：

- **新建（NEW），表示线程被创建出来还没真正启动的状态，可以认为它是个Java内部状态。**

- **就绪（RUNNABLE），表示该线程已经在JVM中执行，当然由于执行需要计算资源，它可能是正在运行，也可能还在等待系统分配给它CPU片段，在就绪队列里面排队。**

- **在其他一些分析中，会额外区分一种状态RUNNING，但是从Java API的角度，并不能表示出来。**

- **阻塞（BLOCKED），这个状态和我们前面两讲介绍的同步非常相关，阻塞表示线程在等待Monitor lock。比如，线程试图通过synchronized去获取某个锁，但是其他线程已经独占了，那么当前线程就会处于阻塞状态。**

- **等待（WAITING），表示正在等待其他线程采取某些操作。一个常见的场景是类似生产者消费者模式，发现任务条件尚未满足，就让当前消费者线程等待（wait），另外的生产者线程去准备任务数据，然后通过类似notify等动作，通知消费线程可以继续工作了。Thread.join()也会令线程进入等待状态。**

- **计时等待（TIMED_WAIT），其进入条件和等待状态类似，但是调用的是存在超时条件的方法，比如wait或join等方法的指定超时版本，如下面示例：**

public final native void wait(long timeout) throws InterruptedException;

- **终止（TERMINATED），不管是意外退出还是正常执行结束，线程已经完成使命，终止运行，也有人把这个状态叫作死亡。**

在第二次调用start()方法的时候，线程可能处于终止或者其他（非NEW）状态，但是不论如何，都是不可以再次启动的。



Java的线程是不允许启动两次的，第二次调用必然会抛出IllegalThreadStateException，这是一种运行时异常，多次调用start被认为是编程错误。

关于线程生命周期的不同状态，在Java 5以后，线程状态被明确定义在其公共内部枚举类型java.lang.Thread.State中，分别是：

- **新建（NEW），表示线程被创建出来还没真正启动的状态，可以认为它是个Java内部状态。**

- **就绪（RUNNABLE），表示该线程已经在JVM中执行，当然由于执行需要计算资源，它可能是正在运行，也可能还在等待系统分配给它CPU片段，在就绪队列里面排队。**

- **在其他一些分析中，会额外区分一种状态RUNNING，但是从Java API的角度，并不能表示出来。**

- **阻塞（BLOCKED），这个状态和我们前面两讲介绍的同步非常相关，阻塞表示线程在等待Monitor lock。比如，线程试图通过synchronized去获取某个锁，但是其他线程已经独占了，那么当前线程就会处于阻塞状态。**

- **等待（WAITING），表示正在等待其他线程采取某些操作。一个常见的场景是类似生产者消费者模式，发现任务条件尚未满足，就让当前消费者线程等待（wait），另外的生产者线程去准备任务数据，然后通过类似notify等动作，通知消费线程可以继续工作了。Thread.join()也会令线程进入等待状态。**

- **计时等待（TIMED_WAIT），其进入条件和等待状态类似，但是调用的是存在超时条件的方法，比如wait或join等方法的指定超时版本，如下面示例：**

public final native void wait(long timeout) throws InterruptedException;

- **终止（TERMINATED），不管是意外退出还是正常执行结束，线程已经完成使命，终止运行，也有人把这个状态叫作死亡。**

在第二次调用start()方法的时候，线程可能处于终止或者其他（非NEW）状态，但是不论如何，都是不可以再次启动的。

 在第二次调用start() 方法的时候，已经被start的线程已经不再是（NEW）状态了，所以无论如何是不能再次启动的。 



 https://blog.csdn.net/zl1zl2zl3/article/details/80776112 





#### 29.Thread的sleep方法会清除中断的状态吗？



中断其实有 2 种状态，运行时中断，阻塞时中断。

顾名思义，运行时中断指的是线程运行时，我们中断他，当然，这个对线程毫无影响，只能通过标志位来判断。

而阻塞时中断就分为 2 种。一种是在等待锁的时候中断，一种是进入锁的时候，wait 的时候中断。

例如一个 synchronized 同步块，当多线程访问同步块时，同步块外的就是等待锁的状态。进入锁了，执行 wait 方法，也是阻塞的状态。

虽然都是阻塞的状态，但这两种阻塞状态是不同的。

基于前言中的阻塞方式，我们一个个来分析。



![img](https://user-gold-cdn.xitu.io/2018/5/19/1637854ee8663fa5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



**总结一下**




非 Lock 接口，即 synchronized 和 Object 还有 Thread 的相关方法，除了 synchronized 不会响应中断，其他的都会响应中断并抛出异常。

Lock 接口，无论是使用 lock 系列方法，还是 Condition 的 await 系列方法，都可随心所欲，想使用什么模式，就使用什么模式。在日常的开发，这个特性还是非常有用的。

相比而言，Lock 相关的接口更加的灵活，对于线程中断的响应和处理可自行设置，而非 Lock 接口则需要了解他们的中断特性.




**例如 sleep 方法和 wait 方法则会清除中断状态。**

**Lock 系列方法抛出异常后，也是会清除中断状态的。**

Lock 清除中断状态的手段则是 Thread.interrupted 方法。

为什么要清除中断状态呢？如果下次有线程再次中断，此时便可以判断。




#### 30.为什么线程通信的方法wait,notify,notifyAll被定义于Object中，而sleep方法被定义在Thread类中？



 sleep()是让某个线程暂停运行一段时间,其控制范围是由当前线程决定,也就是说,在线程里面决定.好比如说,我要做的事情是 "点火->烧水->煮面",而当我点完火之后我不立即烧水,我要休息一段时间再烧.对于运行的主动权是由我的流程来控制.  



 wait 和 notify 不仅仅是普通方法或同步工具，更重要的是它们是 Java 中两个线程之间的通信机制。对语言设计者而言, 如果不能通过 Java 关键字(例如 synchronized)实现通信此机制，同时又要确保这个机制对每个对象可用, 那么 Object 类则是的正确声明位置。记住同步和等待通知是两个不同的领域，不要把它们看成是相同的或相关的。同步是提供互斥并确保 Java 类的线程安全，而 wait 和 notify 是两个线程之间的通信机制。 





 ***\*其实两者都可以让线程暂停一段时间,但是本质的区别是一个线程的运行状态控制,一个是线程之间的通讯的问题\****

 在java.lang.Thread类中，提供了sleep()，
而java.lang.Object类中提供了wait()， notify()和notifyAll()方法来操作线程
sleep()可以将一个线程睡眠，参数可以指定一个时间。
而wait()可以将一个线程挂起，直到超时或者该线程被唤醒。
  wait有两种形式wait()和wait(milliseconds).
sleep和wait的区别有：
 1，这两个方法来自不同的类分别是Thread和Object
 2，最主要是sleep方法没有释放锁，而wait方法释放了锁，使得其他线程可以使用同步控制块或者方法。
 3，wait，notify和notifyAll只能在同步控制方法或者同步控制块里面使用，而sleep可以在
  任何地方使用

```java
  synchronized(x){
   x.notify()
   //或者wait()
  }
```

  4,sleep必须捕获异常，而wait，notify和notifyAll不需要捕获异常 



#### 31.说说Thread类中提供的getState()方法作用，然后说说线程的状态有哪些以及转换过程



 getState(): 返回该线程状态 



Thread类有几个常用的实例方法，分别是：

```JAVA
getId(): 返回该线程的id
getName(): 返回该线程的名字
getPriority(): 返回该线程的优先级
getState(): 返回该线程状态
interrupt(): 使该线程中断
isInterrupted(): 返回该线程是否被中断
isAlive(): 返回该线程是否处于活动状态
isDaemon(): 返回该线程是否是守护线程
join(): 等待该线程终止
join(long millis): 等待该线程终止,至多等待多少毫秒数
join(long millis, int nanos):  等待该线程终止,至多等待多少毫秒数+纳秒数
start(): 使该线程开始执行
toString(): 返回该线程的信息——名字、优先级和所属线程组
setDaemon(boolean on): 将该线程标记为守护线程或用户线程，如果不标记默认是非守护线程
setName(String name): 设置该线程的名字
setPriority(int newPriority): 改变该线程的优先级
```


其中interrupt()和isInterrupted()方法的使用，可以参考： [多线程——interrupt(),interrupted()和isInterrupted() ](https://blog.csdn.net/gxx_csdn/article/details/79220690) 

其中start()方法，可以参考： [start()方法和run()方法]( https://blog.csdn.net/gxx_csdn/article/details/78983305 )

其中join()方法，可以参考： [多线程——join()]( https://blog.csdn.net/gxx_csdn/article/details/79241476 )

```JAVA
/*
 * 测试Thread类的常用方法
 */
public class Test_Thread_Method implements Runnable {

    @Override
    public void run() {

        //用于输出当前执行线程的信息
        System.out.println("名字: "+Thread.currentThread().getName());
        System.out.println("id："+Thread.currentThread().getId());
        System.out.println("优先级："+Thread.currentThread().getPriority());
        System.out.println("状态："+Thread.currentThread().getState());
        System.out.println("是否存活："+Thread.currentThread().isAlive());
        System.out.println("所属线程组："+Thread.currentThread().getThreadGroup());
        System.out.println("该线程组活动线程数目："+Thread.activeCount());
        System.out.println("线程信息String: " + Thread.currentThread().toString());  
        System.out.println("是否持有锁: " + Thread.holdsLock(this));  
        synchronized (this) {
            System.out.println("是否持有锁: " + Thread.holdsLock(this));
        }


    }

    public static void main(String[] args) {

        Test_Thread_Method r = new Test_Thread_Method();
        Thread t = new Thread(r, "test");  //在创建线程时指定线程名字为"test"
        t.start();


        try {
            Thread.sleep(1000);  //休眠1s
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        //用于输出线程t停止后当前线程的信息
        System.out.println();
        System.out.println("名字: "+Thread.currentThread().getName());
        System.out.println("id："+Thread.currentThread().getId());
        System.out.println("优先级："+Thread.currentThread().getPriority());
        System.out.println("状态："+Thread.currentThread().getState());
        System.out.println("是否存活："+Thread.currentThread().isAlive());
        System.out.println("所属线程组："+Thread.currentThread().getThreadGroup());
        System.out.println("该线程组活动线程数目："+Thread.activeCount());

        //用于输出线程t停止后的状态
        System.out.println();
        System.out.println("名字: "+t.getName());
        System.out.println("状态："+t.getState());
        System.out.println("是否存活："+t.isAlive());
    }

}


/*
名字: test
id：10
优先级：5
状态：RUNNABLE
是否存活：true
所属线程组：java.lang.ThreadGroup[name=main,maxpri=10]
该线程组活动线程数目：2
线程信息String: Thread[test,5,main]
是否持有锁: false
是否持有锁: true

名字: main
id：1
优先级：5
状态：RUNNABLE
是否存活：true
所属线程组：java.lang.ThreadGroup[name=main,maxpri=10]
该线程组活动线程数目：1

名字: test
状态：TERMINATED
是否存活：false

*/
```



#### 32.用至少2种方式手写生产者消费者模式代码

 https://segmentfault.com/a/1190000016260650 

#### 33.interrupted和isInterrupted方法的区别？

```java
interrupted()//测试当前线程是否中断
isInterrupted()//测试线程是否已经中断
    
/**
interrupted：不知可以判断当前线程是否已经被中断，而且可以回清除该状态的中断状态
isInterrupted:只会判断当前线程是否已经中断，不会清除线程中的中断状态
*/
```



#### 34.分别讲讲JVM内存结构,Java内存模型,Java对象模型。

###### java内存模型：

java内存模型(JMM)是线程间通信的控制机制.JMM定义了主内存和线程之间抽象关系。
线程之间的共享变量存储在主内存（main memory）中，每个线程都有一个私有的本地
内存（local memory），本地内存中存储了该线程以读/写共享变量的副本。本地内存是
JMM的一个抽象概念，并不真实存在。它涵盖了缓存，写缓冲区，寄存器以及其他的硬
件和编译器优化。Java内存模型的抽象示意图如下：


从上图来看，线程A与线程B之间如要通信的话，必须要经历下面2个步骤： 

1. 首先，线程A把本地内存A中更新过的共享变量刷新到主内存中去。 

2. 然后，线程B到主内存中去读取线程A之前已更新过的共享变量

   

###### JVM内存结构：



###### Java对象模型：

 https://juejin.im/post/5b7625aa6fb9a009910e641d 

 https://blog.csdn.net/w372426096/article/details/81167669 

#### 35.什么是happens-before?它的规则有哪些？

倘若在程序开发中，仅靠sychronized和volatile关键字来保证原子性、可见性以及有序性，那么编写并发程序可能会显得十分麻烦，幸运的是，在Java内存模型中，还提供了happens-before 原则来辅助保证程序执行的原子性、可见性以及有序性的问题，它是判断数据是否存在竞争、线程是否安全的依据，happens-before 原则内容如下



- 程序顺序原则，即在一个线程内必须保证语义串行性，也就是说按照代码顺序执行。

- 锁规则 解锁(unlock)操作必然发生在后续的同一个锁的加锁(lock)之前，也就是说，如果对于一个锁解锁后，再加锁，那么加锁的动作必须在解锁动作之后(同一个锁)。

- volatile规则 volatile变量的写，先发生于读，这保证了volatile变量的可见性，简单的理解就是，volatile变量在每次被线程访问时，都强迫从主内存中读该变量的值，而当该变量发生变化时，又会强迫将最新的值刷新到主内存，任何时刻，不同的线程总是能够看到该变量的最新值。

- 线程启动规则 线程的start()方法先于它的每一个动作，即如果线程A在执行线程B的start方法之前修改了共享变量的值，那么当线程B执行start方法时，线程A对共享变量的修改对线程B可见

- 传递性 A先于B ，B先于C 那么A必然先于C

- 线程终止规则 线程的所有操作先于线程的终结，Thread.join()方法的作用是等待当前执行的线程终止。假设在线程B终止之前，修改了共享变量，线程A从线程B的join方法成功返回后，线程B对共享变量的修改将对线程A可见。

- 线程中断规则 对线程 interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过Thread.interrupted()方法检测线程是否中断。

- 对象终结规则 对象的构造函数执行，结束先于finalize()方法

上述8条原则无需手动添加任何同步手段(synchronized|volatile)即可达到效果，下面我们结合前面的案例演示这8条原则如何判断线程是否安全，如下：



```java
class MixedOrder{
    int a = 0;
    boolean flag = false;
    public void writer(){
        a = 1;
        flag = true;
    }
	public void read(){
 	   if(flag){
    	    int i = a + 1；
  	  }
	}
}
```


同样的道理，存在两条线程A和B，线程A调用实例对象的writer()方法，而线程B调用实例对象的read()方法，线程A先启动而线程B后启动，那么线程B读取到的i值是多少呢？现在依据8条原则，由于存在两条线程同时调用，因此程序次序原则不合适。writer()方法和read()方法都没有使用同步手段，锁规则也不合适。没有使用volatile关键字，volatile变量原则不适应。线程启动规则、线程终止规则、线程中断规则、对象终结规则、传递性和本次测试案例也不合适。线程A和线程B的启动时间虽然有先后，但线程B执行结果却是不确定，也是说上述代码没有适合8条原则中的任意一条，也没有使用任何同步手段，所以上述的操作是线程不安全的，因此线程B读取的值自然也是不确定的。修复这个问题的方式很简单，要么给writer()方法和read()方法添加同步手段，如synchronized或者给变量flag添加volatile关键字，确保线程A修改的值对线程B总是可见。

#### 36.什么是JMM?谈谈工作内存和主内存的关系。

Java内存模型即JMM（Java  Memory  Model），JMM定义了一个Java虚拟机(JVM)再计算机内存中的工作方式，JVM是整个计算机虚拟模型，所以JMM隶属于JVM的



关于JMM中的主内存和工作内存说明如下

- **主内存**

主要存储的是Java实例对象，所有线程创建的实例对象都存放在主内存中，不管该实例对象是成员变量还是方法中的本地变量(也称局部变量)，当然也包括了共享的类信息、常量、静态变量。由于是共享数据区域，多条线程对同一个变量进行访问可能会发现线程安全问题。

- **工作内存**

主要存储当前方法的所有本地变量信息(工作内存中存储着主内存中的变量副本拷贝)，每个线程只能访问自己的工作内存，即线程中的本地变量对其它线程是不可见的，就算是两个线程执行的是同一段代码，它们也会各自在自己的工作内存中创建属于当前线程的本地变量，当然也包括了字节码行号指示器、相关Native方法的信息。注意由于工作内存是每个线程的私有数据，线程间无法相互访问工作内存，因此存储在工作内存的数据不存在线程安全问题。

弄清楚主内存和工作内存后，接了解一下主内存与工作内存的数据存储类型以及操作方式，根据虚拟机规范，对于一个实例对象中的成员方法而言，如果方法中包含本地变量是基本数据类型（boolean,byte,short,char,int,long,float,double），将直接存储在工作内存的帧栈结构中，但倘若本地变量是引用类型，那么该变量的引用会存储在功能内存的帧栈中，而对象实例将存储在主内存(共享数据区域，堆)中。但对于实例对象的成员变量，不管它是基本数据类型或者包装类型(Integer、Double等)还是引用类型，都会被存储到堆区。至于static变量以及类本身相关信息将会存储在主内存中。需要注意的是，在主内存中的实例对象可以被多线程共享，倘若两个线程同时调用了同一个对象的同一个方法，那么两条线程会将要操作的数据拷贝一份到自己的工作内存中，执行完成操作后才刷新到主内存，简单示意图如下所示：

 ![img](https://img-blog.csdn.net/20170609093435508?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamF2YXplamlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#### 37.Java重排序了解吗？谈谈重排序的3种情况。

#### 38.什么是可见性？为什么存在可见性问题？怎样解决可见性带来的问题？

#### 39.管程是什么？谈谈它的重要性。

