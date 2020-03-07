## Synchronize



## 一、概念

synchronized 是 Java 中的关键字，是利用锁的机制来实现同步的。

锁机制有如下两种特性：

- 互斥性：即在同一时间只允许一个线程持有某个对象锁，通过这种特性来实现多线程中的协调机制，这样在同一时间只有一个线程对需同步的代码块(复合操作)进行访问。互斥性我们也往往称为操作的原子性。
- 可见性：必须确保在锁被释放之前，对共享变量所做的修改，对于随后获得该锁的另一个线程是可见的（即在获得锁时应获得最新共享变量的值），否则另一个线程可能是在本地缓存的某个副本上继续操作从而引起不一致。

## 二、对象锁和类锁

#### 1. 对象锁

在 Java 中，**每个对象都会有一个 monitor (监视锁)对象，这个对象其实就是 Java 对象的锁，通常会被称为“内置锁”或“对象锁”。类的对象可以有多个，所以每个对象有其独立的对象锁，互不干扰。**

#### 2. 类锁

在 Java 中，**针对每个类也有一个锁，可以称为“类锁”，类锁实际上是通过对象锁实现的，即类的 Class 对象锁。每个类只有一个 Class 对象，所以每个类只有一个类锁。**



## 三、synchronized 的用法分类



# 三、synchronized 的用法分类

可以用`synchronized`修饰一个方法, 并且也知道对应的由锁保护的代码块就是整个方法体, 但是, **它的锁是什么呢**?
要回答这个问题，首先要区分`synchronized` 所修饰的方法是否是静态方法:

> 如果`synchronized`所修饰的是静态方法, 则其所用的锁为`Class对象`
> 如果`synchronized`所修饰的是非静态方法, 则其所用的锁为`方法调用所在的对象`



**synchronized 的用法可以从两个维度上面分类：**

## 1. 根据修饰对象分类

synchronized 可以修饰方法和代码块

- 修饰代码块
  - synchronized(this|object) {}
  - synchronized(类.class) {}
- 修饰方法
  - 修饰非静态方法
  - 修饰静态方法

## 2. 根据获取的锁分类

- 获取对象锁
  - synchronized(this|object) {}
  - 修饰非静态方法
- 获取类锁
  - synchronized(类.class) {}
  - 修饰静态方法



- `sychronized`关键字添加到`static`静态方法上或`synchonized(Class)`代码块是给Class类上锁
- `sychronized`关键字添加到非`static`静态方法上是给对象上锁

### 同步代码块

，只有获得这把锁的线程才访问，并且同一时刻，只有一个线程能持有这把锁，这样就保证了同一时刻只有一个线程能执行被锁住的代码

同步代码块(Synchronized Block) 是java中最基础的实现线程间的同步与通信的机制之一，本篇我们将对同步代码块以及监视器锁的概念进行讨论





#### 什么是同步代码块

简单来说是将**一段代码**用**一把锁**给锁起来，只有获得这把锁的线程才访问，并且同一时刻，只有一个线程能持有这把锁，这样就保证了同一时刻只有一个线程能执行被锁住的代码

#### 一段代码

一般来说, 由 `synchronized` 锁住的代码都是拿`{}`括起来的代码块:

```java
synchronized(this) {
    //由锁保护的代码
}
```

但值得注意的是, `synchronized` 也可以用来修饰一个方法, 则对应的被锁保护的`一段代码`很自然就是整个方法体.

```java
public class Foo {
    public synchronized void doSomething() {
        // 由锁保护的代码
    }
}
```

#### 锁

其实锁这个东西说起来很抽象, 你可以就把它想象成现实中的锁, 所以它只不过是一块`令牌`, 一把`尚方宝剑`, 它是木头做的还是金属做的并不重要, 你可以拿任何东西当作锁, 重要的是它代表的含义: 谁持有它, 谁就有独立访问临界区(即上面所说的`一段代码`)的权利.

在java中, 我们可以拿一个对象当作锁.

这里引用<<java并发编程实战>>中的一段话:

> 每个java对象都可以用做一个实现同步的锁, 这些锁被称为内置锁(Intrinsic Lock)或者监视器锁(Monitor Lock). 线程在进入同步代码块之前会自动获得锁, 并且在退出同步代码块时自动释放锁.
>
> 获得内置锁的唯一途径就是进入由这个锁保护的同步代码块或方法.

所以, `synchronized` 同步代码块的标准写法应该是:

```java
synchronized(reference-to-lock) {
    //临界区
}
```

其中, 括号里面的reference-to-lock就是锁的引用, 它只要指向一个Java对象就行, 你可以自己随便new一个不相关的对象, 将它作为锁放进去, 也可以像之前的例子一样, 直接使用`this`, 代表使用当前对象作为锁.

有的同学就要问了, 我们前面说

#### 注：

1，不同的锁一个是类锁（methodA），一个是对象锁（methodC）。更深层次的说明了类锁和对象锁是两个不一样的锁，控制着不同的区域，它们是互不干扰的。同样，线程获得对象锁的同时，也可以获得该类锁，即同时获得两个锁，这是允许的.

2，当使用`synchronized` 修饰非静态方法时, 以下两种写法是等价的:

```java
//写法1
public synchronized void doSomething() {
    // 由锁保护的代码
}

//写法2
public void doSomething() {
    synchronized(this) {
        // 由锁保护的代码
    }
}
```

## 四、synchronized 的用法详解

这里根据获取的锁分类来分析 synchronized 的用法





##### 对于同一个对象

**`this` 和 `方法调用所在的对象`**

这两个其实是一个意思, 我们需要特别注意的是, 一个Class可以有多个实例(Instance), 每一个Instance都可以作为锁, 不同Instance就是不同的锁, **同一个Instance就是同一个锁, `this` 和 `方法调用所在的对象` 指代的都是调用这个同步代码块的对象.**

这么说可能比较抽象, 我们直接上例子: (以下例子转载自博客[Java中Synchronized的用法](https://blog.csdn.net/luoweifu/article/details/46613015))

```java
class SyncThread implements Runnable {
    private static int count;

    public SyncThread() {
        count = 0;
    }

    public  void run() {
        synchronized(this) {
            for (int i = 0; i < 5; i++) {
                try {
                    System.out.println(Thread.currentThread().getName() + ":" + (count++));
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    
    public static void main(String[] args) {
        SyncThread syncThread = new SyncThread();
        //线程1和线程2使用了SyncThread类的同一个对象实例
        //因此, 这两个线程中的synchronized(this), 持有的是同一把锁
        Thread thread1 = new Thread(syncThread, "SyncThread1");
        Thread thread2 = new Thread(syncThread, "SyncThread2");
        thread1.start();
        thread2.start();
    }
}
```

运行结果:

> SyncThread1:0
> SyncThread1:1
> SyncThread1:2
> SyncThread1:3
> SyncThread1:4
> SyncThread2:5
> SyncThread2:6
> SyncThread2:7
> SyncThread2:8
> SyncThread2:9

这里两个线程`SyncThread1` 和 `SyncThread2` 持有同一个对象`syncThread`的锁, **因此同一时刻, 只有一个线程能访问同步代码块, 线程`SyncThread2` 只有等 `SyncThread1` 执行完同步代码块后, `SyncThread1`线程自动释放了锁, 随后 `SyncThread2`才能获取同一把锁, 进入同步代码块.**



##### 对于不用对象

我们也可以修改一下main函数, 让两个线程持有同一Class对象的不同实例的锁:

```java
public static void main(String[] args) {
    Thread thread1 = new Thread(new SyncThread(), "SyncThread1");
    Thread thread2 = new Thread(new SyncThread(), "SyncThread2");
    thread1.start();
    thread2.start();
}
```

上面这段等价于:

```java
public static void main(String[] args) {
    SyncThread syncThread1 = new SyncThread();
    SyncThread syncThread2 = new SyncThread();
    Thread thread1 = new Thread(syncThread1, "SyncThread1");
    Thread thread2 = new Thread(syncThread2, "SyncThread2");
    thread1.start();
    thread2.start();
}
```

运行结果:

> SyncThread1:0
> SyncThread2:1
> SyncThread1:2
> SyncThread2:3
> SyncThread1:4
> SyncThread2:5
> SyncThread1:6
> SyncThread2:7
> SyncThread1:8
> SyncThread2:9

可见, **两个线程这次都能访问同步代码块, 这是因为线程1执行的是`syncThread1`对象的同步代码块, 线程2执行的是`syncThread2`的同步代码块, 虽然这两个同步代码块一样, 但是他们在不同的对象实例里面, 即虽然它们都用`this`作为锁,** 但是`this`指代的对象在这两个线程中不是同一个对象, 两个线程各自都能获得锁, 因此各自都能执行这一段同步代码块.

这告诉我们, 当一段代码用同步代码块包起来的时候, 并不绝对意味着这段代码同一时刻只能由一个线程访问, 这种情况只发生在多个线程访问的是同一个Instance, 也就是说, 多个线程请求的是同一把锁.

再回顾我们上面两个例子, 第一个例子中, 两个线程使用的是同一个对象实例, 他们需要同一把对象锁 `syncThread`,
第二个例子中, 两个线程分别使用了一个对象实例, 他们分别请求的是自己访问的对象实例的锁`syncThread1`, `syncThread2`, 因此都能访问同步代码块.

**导致不同线程可以同时访问同步代码块的最根本原因就是我们使用的是`当前实例对象锁(this)`, 因为类的实例可以有多个, 这导致了同步代码块`散布在`类的多个实例中, 虽然同一个实例中的同步代码块只能由持有锁的单个线程访问(this对象锁保护), 但是我们可以每个线程访问自己的对象实例, 而每一个对象实例的同步代码块都是一致的, 这就间接导致了多个线程同时访问了"同一个"同步代码块.**

上面这种情况在某些条件下是没有问题的, 例如同步代码块中不存在对静态变量(共享的状态量)的修改.

但是, 对于上面的例子, 这样的情况明显违背了我们加同步代码块的初衷.

要解决上面的情况, 一种可行的办法就是像第一个例子一样, 多个线程使用同一个对象实例, 例如在单例模式下, 本身就只有一个对象实例, 所以多个线程必将请求同一把锁, 从而实现同步访问.

另一种方法就是我们下面要讲的: 使用Class锁.

#### 使用Class级别锁

前面我们提到:

> 如果`synchronized`所修饰的是静态方法, 则其所用的锁为`Class对象`

这是因为静态方法是属于类的而不属于对象的, 因此synchronized修饰的静态方法锁定的是这个类的所有对象。我们来看下面一个例子(以下例子同样转载自博客[Java中Synchronized的用法](https://blog.csdn.net/luoweifu/article/details/46613015)):

```java
class SyncThread implements Runnable {
    private static int count;

    public SyncThread() {
        count = 0;
    }

    // synchronized 关键字加在一个静态方法上
    public synchronized static void staticMethod() {
        for (int i = 0; i < 5; i ++) {
            try {
                System.out.println(Thread.currentThread().getName() + ":" + (count++));
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public void run() {
        staticMethod();
    }

    public static void main(String[] args) {
        SyncThread syncThread1 = new SyncThread();
        SyncThread syncThread2 = new SyncThread();
        Thread thread1 = new Thread(syncThread1, "SyncThread1");
        Thread thread2 = new Thread(syncThread2, "SyncThread2");
        thread1.start();
        thread2.start();
    }
}
```

运行结果:

> SyncThread1:0
> SyncThread1:1
> SyncThread1:2
> SyncThread1:3
> SyncThread1:4
> SyncThread2:5
> SyncThread2:6
> SyncThread2:7
> SyncThread2:8
> SyncThread2:9

可见, 静态方法锁定了类的所有对象, 用我们之前的话来说, **如果说"因为类的实例可以有多个, 这导致了同步代码块`散布在`类的多个实例中", 那么类的静态方法就是阻止同步代码块`散布在`类的实例中, 因为类的静态方法只属于类本身.**

其实, 上面的例子的本质就是拿Class对象作为锁, 我们前面也提到了, 可以拿任何对象作为锁, 如果我们直接拿类的Class对象作为锁, 同样可以保证所以线程请求的都是同一把锁, 因为Class对象只有一个.

> 类锁实际上是通过对象锁实现的，即类的 Class 对象锁。每个类只有一个 Class 对象，所以每个类只有一个类锁。

```java
class SyncThread implements Runnable {
    private static int count;

    public SyncThread() {
        count = 0;
    }

    public void run() {
        // 这里直接拿Class对象作为锁
        synchronized(SyncThread.class) {
            for (int i = 0; i < 5; i++) {
                try {
                    System.out.println(Thread.currentThread().getName() + ":" + (count++));
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) {
        SyncThread syncThread1 = new SyncThread();
        SyncThread syncThread2 = new SyncThread();
        Thread thread1 = new Thread(syncThread1, "SyncThread1");
        Thread thread2 = new Thread(syncThread2, "SyncThread2");
        thread1.start();
        thread2.start();
    }
}
```

这样所得到的结果与上面的类的静态方法加锁是一致的。



2.当一个线程访问对象的一个synchronized(this)同步代码块时，另一个线程仍然可以访问该对象中的非synchronized(this)同步代码块。

##### 多个线程访问同步和非同步代码块

```java
class Counter implements Runnable{
   private int count;

   public Counter() {
      count = 0;
   }

   public void countAdd() {
      synchronized(this) {
         for (int i = 0; i < 5; i ++) {
            try {
               System.out.println(Thread.currentThread().getName() + ":" + (count++));
               Thread.sleep(100);
            } catch (InterruptedException e) {
               e.printStackTrace();
            }
         }
      }
   }

   //非synchronized代码块，未对count进行读写操作，所以可以不用synchronized
   public void printCount() {
      for (int i = 0; i < 5; i ++) {
         try {
            System.out.println(Thread.currentThread().getName() + " count:" + count);
            Thread.sleep(100);
         } catch (InterruptedException e) {
            e.printStackTrace();
         }
      }
   }

   public void run() {
      String threadName = Thread.currentThread().getName();
      if (threadName.equals("A")) {
         countAdd();
      } else if (threadName.equals("B")) {
         printCount();
      }
   }
}


```



调用代码:

```java
Counter counter = new Counter();
Thread thread1 = new Thread(counter, "A");
Thread thread2 = new Thread(counter, "B");
thread1.start();
thread2.start();
```

结果如下：

> A:0
> B count:1
> A:1
> B count:2
> A:2
> B count:3
> A:3
> B count:4
> A:4
> B count:5

上面代码中countAdd是一个synchronized的，printCount是非synchronized的。从上面的结果中可以看出一个线程访问一个对象的synchronized代码块时，别的线程可以访问该对象的非synchronized代码块而不受阻塞。




#### 指定要给某个对象加锁

```java
/**
 * 银行账户类
 */
class Account {
   String name;
   float amount;

   public Account(String name, float amount) {
      this.name = name;
      this.amount = amount;
   }
   //存钱
   public  void deposit(float amt) {
      amount += amt;
      try {
         Thread.sleep(100);
      } catch (InterruptedException e) {
         e.printStackTrace();
      }
   }
   //取钱
   public  void withdraw(float amt) {
      amount -= amt;
      try {
         Thread.sleep(100);
      } catch (InterruptedException e) {
         e.printStackTrace();
      }
   }

   public float getBalance() {
      return amount;
   }
}

/**
 * 账户操作类
 */
class AccountOperator implements Runnable{
   private Account account;
   public AccountOperator(Account account) {
      this.account = account;
   }

   public void run() {
      synchronized (account) {
         account.deposit(500);
         account.withdraw(500);
         System.out.println(Thread.currentThread().getName() + ":" + account.getBalance());
      }
   }
}
```



在AccountOperator 类中的run方法里，我们用synchronized 给account对象加了锁。这时，当一个线程访问account对象时，其他试图访问account对象的线程将会阻塞，直到该线程访问account对象结束。也就是说谁拿到那个锁谁就可以运行它所控制的那段代码。
当有一个明确的对象作为锁时，就可以用类似下面这样的方式写程序。

```java
public void method3(SomeObject obj)
{
   //obj 锁定的对象
   synchronized(obj)
   {
      // todo
   }
}
```



当没有明确的对象作为锁，只是想让一段代码同步时，可以创建一个特殊的对象来充当锁：

```java
class Test implements Runnable
{
   private byte[] lock = new byte[0];  // 特殊的instance变量
   public void method()
   {
      synchronized(lock) {
         // todo 同步代码块
      }
   }

   public void run() {

   }
}
```



说明：零长度的byte数组对象创建起来将比任何对象都经济――查看编译后的字节码：生成零长度的byte[]对象只需3条操作码，而Object lock = new Object()则需要7行操作码。



修饰一个方法
Synchronized修饰一个方法很简单，就是在方法的前面加synchronized，public synchronized void method(){//todo}; synchronized修饰方法和修饰一个代码块类似，只是作用范围不一样，修饰代码块是大括号括起来的范围，而修饰方法范围是整个函数。如将【Demo1】中的run方法改成如下的方式，实现的效果一样。

*【Demo4】：synchronized修饰一个方法

```java
public synchronized void run() {
   for (int i = 0; i < 5; i ++) {
      try {
         System.out.println(Thread.currentThread().getName() + ":" + (count++));
         Thread.sleep(100);
      } catch (InterruptedException e) {
         e.printStackTrace();
      }
   }
}

```



Synchronized作用于整个方法的写法。
写法一：

```java
public synchronized void method()
{
   // todo
}

```



写法二：

```java
public void method()
{
   synchronized(this) {
      // todo
   }
}

```

写法一修饰的是一个方法，写法二修饰的是一个代码块，但写法一与写法二是等价的，都是锁定了整个方法时的内容。

在用synchronized修饰方法时要注意以下几点：
1. synchronized关键字不能继承。
虽然可以使用synchronized来定义方法，但synchronized并不属于方法定义的一部分，因此，synchronized关键字不能被继承。如果在父类中的某个方法使用了synchronized关键字，而在子类中覆盖了这个方法，在子类中的这个方法默认情况下并不是同步的，而必须显式地在子类的这个方法中加上synchronized关键字才可以。当然，还可以在子类方法中调用父类中相应的方法，这样虽然子类中的方法不是同步的，但子类调用了父类的同步方法，因此，子类的方法也就相当于同步了。这两种方式的例子代码如下：
在子类方法中加上synchronized关键字



```java
class Parent {
   public synchronized void method() { }
}
class Child extends Parent {
   public synchronized void method() { }
}
```



在子类方法中调用父类的同步方法

```java
class Parent {
   public synchronized void method() {   }
}
class Child extends Parent {
   public void method() { super.method();   }
} 

```



​	

在定义接口方法时不能使用synchronized关键字。
构造方法不能使用synchronized关键字，但可以使用synchronized代码块来进行同步。
修饰一个静态的方法
Synchronized也可修饰一个静态方法，用法如下：

```java
public synchronized static void method() {
   // todo
}

```


我们知道静态方法是属于类的而不属于对象的。同样的，synchronized修饰的静态方法锁定的是这个类的所有对象。我们对Demo1进行一些修改如下：

【Demo5】：synchronized修饰静态方法

```java
/**

 * 同步线程
   */
   class SyncThread implements Runnable {
   private static int count;

   public SyncThread() {
      count = 0;
   }

   public synchronized static void method() {
      for (int i = 0; i < 5; i ++) {
         try {
            System.out.println(Thread.currentThread().getName() + ":" + (count++));
            Thread.sleep(100);
         } catch (InterruptedException e) {
            e.printStackTrace();
         }
      }
   }

   public synchronized void run() {
      method();
   }
   }
   
```



调用代码:

```java
SyncThread syncThread1 = new SyncThread();
SyncThread syncThread2 = new SyncThread();
Thread thread1 = new Thread(syncThread1, "SyncThread1");
Thread thread2 = new Thread(syncThread2, "SyncThread2");
thread1.start();
thread2.start();

```



结果如下：

> SyncThread1:0
> SyncThread1:1
> SyncThread1:2
> SyncThread1:3
> SyncThread1:4
> SyncThread2:5
> SyncThread2:6
> SyncThread2:7
> SyncThread2:8
> SyncThread2:9

syncThread1和syncThread2是SyncThread的两个对象，但在thread1和thread2并发执行时却保持了线程同步。这是因为run中调用了静态方法method，而静态方法是属于类的，所以syncThread1和syncThread2相当于用了同一把锁。这与Demo1是不同的。

修饰一个类
Synchronized还可作用于一个类，用法如下：

```java
class ClassName {
   public void method() {
      synchronized(ClassName.class) {
         // todo
      }
   }
}
```




我们把Demo5再作一些修改。
【Demo6】:修饰一个类

```java
/**

 * 同步线程
   */
   class SyncThread implements Runnable {
   private static int count;

   public SyncThread() {
      count = 0;
   }

   public static void method() {
      synchronized(SyncThread.class) {
         for (int i = 0; i < 5; i ++) {
            try {
               System.out.println(Thread.currentThread().getName() + ":" + (count++));
               Thread.sleep(100);
            } catch (InterruptedException e) {
               e.printStackTrace();
            }
         }
      }
   }

   public synchronized void run() {
      method();
   }
   }
```



其效果和【Demo5】是一样的，synchronized作用于一个类T时，是给这个类T加锁，T的所有对象用的是同一把锁。

总结：
A. 无论synchronized关键字加在方法上还是对象上，如果它作用的对象是非静态的，则它取得的锁是对象；如果synchronized作用的对象是一个静态方法或一个类，则它取得的锁是对类，该类所有的对象同一把锁。
B. 每个对象只有一个锁（lock）与之相关联，谁拿到这个锁谁就可以运行它所控制的那段代码。
C. 实现同步是要很大的系统开销作为代价的，甚至可能造成死锁，所以尽量避免无谓的同步控制。






## 几点补充

其实到这里, 重要的部分已经讲完了, 下面补充说明几点:

(1) 当一个线程访问对象的一个synchronized(this)同步代码块时，另一个线程仍然可以访问该对象中的非synchronized(this)同步代码块。

这个结论是显而易见的, 在没有加锁的情况下, 所有的线程都可以自由地访问对象中的代码, 而synchronized关键字只是限制了线程对于已经加锁的同步代码块的访问, 并不会对其他代码做限制.
这里也提示我们:

> 同步代码块应该越短小越好

(2) 当一个线程访问object的一个synchronized(this)同步代码块时，其他线程对object中所有其它synchronized(this)同步代码块的访问将被阻塞。

这个结论也是显而易见的, 因为synchronized(this)拿的都是当前对象的锁, 如果一个线程已经进入了一个同步代码块, 说明它已经拿到了锁, 而访问同一个object中的其他同步代码块同样需要当前对象的锁, 所以它们会被阻塞.

(3) synchronized关键字不能继承。

对于父类中用synchronized 修饰的方法，子类在覆盖该方法时，默认情况下不是同步的，必须显式的使用 synchronized 关键字修饰才行, 当然子类也可以直接调用父类的方法, 这样就间接实现了同步.

(4) 在定义接口方法时不能使用synchronized关键字。
(5) 构造方法不能使用synchronized关键字，但可以使用synchronized代码块来进行同步。
(6) 离开同步代码块后，所获得的锁会被自动释放。

## 总结

- `synchronized`关键字通过一把`锁`锁住`一段代码`, 使得线程只有在持有锁的时候才能访问这段代码
- 任何java对象都可以作为这把锁
- 可以在`synchronized`后面用`()`显式的指定锁. 也可以直接作用在方法上
  - 作用于普通方法时, 相当于以`this`对象作为锁, 此时同步代码块`散布于`类的所有实例中, 每一个实例的同步代码块的锁 为该实例对象自身。
  - 作用于静态方法时, 相当于以Class对象作为锁, 此时对象的所有实例只能争抢同一把锁。
- 内置锁的一个重要的特性是当离开同步代码块之后， 会自动释放锁，而其他的高级锁（如ReentrantLock)需要显式释放锁。

## 思考题

前面我们说明了`synchronized` 的使用方法，但对一些底层的细节并不了解，如:

1. 前面说“获得内置锁的唯一途径就是进入由这个锁保护的同步代码块或方法.”， 这句话看上去很有道理，其实是废话，同步代码块究竟是怎么获得锁的？
2. 我们说，JAVA中任何对象都可以作为锁，那么锁信息是怎么被记录和存储的?
3. 为什么代码离开了同步代码块锁就被释放了，谁释放了锁，怎样叫释放了锁？

这些问题，我们后续的文章再研究。