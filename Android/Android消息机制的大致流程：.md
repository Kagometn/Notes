[TOC]


&emsp;&ensp;Handler是Android消息机制的上层接口，这使得在开发过程中，只需要和Handler交互即可，通过它很方便的将线程切换到Handler所在的线程中。

&emsp;&emsp;Android的消息机制主要指的是Handler的运行机制，handler的运行需要底层的MessageQueue以及Looper的支撑。

> - MessageQueue内部存储结构并不是真正的队列，而是采用的单链表的数据结构来存储消息队列的。
> - Looper会以一种无限循环的方式来检测是否有新的消息，如果有的话就进行处理，反之就会继续检测，是通俗的ThreadLocal。

注：ThreadLocal不是线程，他的作用就是可以在每个线程中存储数据。Handler创建的时候采用当前的Looper来构造消息循环系统，那么Handler是如何获取Looper的呢：

> 线程默认没有Looper，如果使用Handler的时候就必须为线程创建Looper,而一般说的主线程，即UI线程，ActivityThread,在创建的时候就会初始化looper，所以可以可以在主线程默认可以使用Handler的原因。

从根本上而言，Android系统和Window一样，也属于消息驱动型系统，而消息驱动型系统会有一下四大要素：




- 接收消息的"消息队列"
- 阻塞式的从消息队列中接收消息并进行处理的"线程"
- 可发送的"消息的格式"
- 消息发送函数



Android系统与之对应的实现的就是(MessageQueue,Looper,Handler)

- 接收消息的"消息队列",——[MessageQueue]主要功能是投递消息(MeaasgeQueue.enqueueMessage)和取走消息池的消息(MessageQueue.next);
- 阻塞式的从消息队列中接收消息并进行处理的"线程"——(Thread+Looper)
- 可发送的"消息格式"——Meaasge
- 消息发送函数——(Handler的post和sendMessage)

&ensp;&emsp;一个Looper相当于一个消息泵，他本身是一个死循环，不断地从MessageQueue中提取Message或者是Runnable。而Handler 可以看做是一个Looper的暴露接口，向外部暴露一些事件，以及sendMessage()和post()函数。



![https://user-gold-cdn.xitu.io/2019/4/3/169dee974ea39507?imageslim](https://user-gold-cdn.xitu.io/2019/4/3/169dee974ea39507?imageslim)


从开发角度而言，Handler是Android消息机制的上层接口，使得在开发过程中只需要和handler交互即可。通过Handler可以轻松地将一个任务切换到Handler所在的线程中去执行。

> 更新UI布局是Handler的一个特殊的使用场景。<font color=red> 一般在主线程中更新UI,在子线程中执行耗时操作</font> 子线程中一般执行的耗时操作：读取文件或者访问网络等,由于Android开发规范的限制，一般不能再主线程中访问UI组件

&nbsp;

- 为什么系统不允许在子线程访问UI?

&nbsp;
&emsp;由于UI控件不是线程安全的，如果在多线程中并发访问可能导致UI控件处于不可预期的状态
- 那为什么系统不对UI控件的访问采用锁机制？

&nbsp;
&emsp;缺点有两个：

&nbsp;
 ①加上锁机制会让UI访问的逻辑变得复杂
 &nbsp;

 &ensp;②锁机制会降低UI访问的效率，因为锁机制会阻塞某些线程的执行

 &nbsp;
 鉴于以上两点，最简单且高效的方法就是采用单线程模型来处理UI操作，对于开发者而言不是很麻烦，只需要使用Handler切换线程即可。

# Android消息机制的大致流程：
&nbsp;
由于消息模型贯穿了整个App的生命周期，所以主线程的消息模型在App启动时就会被创建.

&nbsp;
Android App的入口类是ActivityThread，这个类的main方法就是整个App的入口。


```java
//创建一个消息循环
public static void main(String[] args) {
    ......
    Looper.prepareMainLooper(); 
    //主线程的Looper(),也是app应用主主要的消息机制的消息管理者
    ......
    Looper.loop();
    throw new RuntimeException("Main thread loop unexpectedly exited");
}
```


&emsp;&emsp;上述代码最后一句就是个异常，为了是程序不执行到最后一句，Looper通过死循环的方式。
```java
public static void loop() {
    ......
    for(;;){
      ......
    }
    ......
}
```

Looper不停的去轮询消息队列，当有消息的时候进行更新，没有消息的时候就休眠，那么Looper是如何控制该状态的呢？
> 因为Android系统基于Linux系统，自然会汲取一些Linux的成熟的一些设计，而消息机制就是借助Linux的epoll机制，消息循环中有两个重要的native方法：nativePollOnce以及nativeWake消息唤醒。此时主线程会释放CPU资源进入休眠状态，直到下个消息到达或者有事务发生，通过往pipe管道写端写入数据来唤醒主线程工作。这里采用的epoll机制，是一种IO多路复用机制，可以同时监控多个描述符，当某个描述符就绪(读或写就绪)，则立刻通知相应程序进行读或写操作，本质同步I/O，即读写是阻塞的。 所以说，主线程大多数时候都是处于休眠状态，并不会消耗大量CPU资源。

# Android的消息循环机制的主要流程：


主线程：    
- App进程创建
- 消息队列初始化
- 主线程死循环读取消息队列
- 消息处理完之后调用nativePollOnce方法


其他线程：
- 引用主线程的消息队列，添加到消息队列中(由于实现了线程同步问题，这时队列中的数据已经和子线程一致了)
- 调用nativeWake方法唤醒进程


主线程：
- 主线程nativePollOnce挂起取消
- 处理消息
- 处理消息之后，nativePollOnce再次挂起

## Android 消息机制的主要流程
### Handler发送Message之后，Message如何进入到MessageQueue中

实际开发中一般使用Handler的sendMessage,或者是sendMessageDelayed，此时就会调用Message的sendToTarget方法(handler.obtainMessage().sendToTag()),通过对源码的解析，发现上述的几种方法最终都会调用Handler的sendMessageAtTime方法：
```java
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    return enqueueMessage(queue, msg, uptimeMillis);
}

```
不难发现消息进入到队列的入口——enqueueMessage()
```java
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
    msg.target = this;
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis);
}

```
Message是通过该方法进入到MessageQueue的<font color=red> msg.target = this;</font>
```java
Message prev;
for (;;) {
    prev = p;
    p = p.next;
    if (p == null || when < p.when) {
        break;
    }
    if (needWake && p.isAsynchronous()) {
        needWake = false;
    }
}
msg.next = p; // invariant: p == prev.next
prev.next = msg;
```

&emsp;&emsp;P代表的是MessageQueue的队列头部，而prev指的是p指向的Message的前一条。在for循环中，prev和p从队列的头部一直索引到队列的尾部，当跳出for循环时，如果p指向的是null，则说明最后一条消息就是prev所指向的Message，当两个表达式执行完毕之后，msg被添加到队列的尾部，至此Message向MessageQueue中添加消息过程完毕。


### Looper是如何循环获取Message的

MessageQueue只是Message的容器，并不具本选择Message派发给相应Handler的功能，此时就要使用Looper了：
```java
for (;;) {
    Message msg = queue.next(); // might block
    if (msg == null) {
        // No message indicates that the message queue is quitting.
        return;
    }

    // This must be in a local variable, in case a UI event sets the logger
    final Printer logging = me.mLogging;
    if (logging != null) {
        logging.println(">>>>> Dispatching to " + msg.target + " " +
                msg.callback + ": " + msg.what);
    }

    final long slowDispatchThresholdMs = me.mSlowDispatchThresholdMs;

    final long traceTag = me.mTraceTag;
    if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
        Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
    }
    final long start = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
    final long end;
    try {
        msg.target.dispatchMessage(msg);
        end = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
    } finally {
        if (traceTag != 0) {
            Trace.traceEnd(traceTag);
        }
    }
```
特此拿出第二行以及倒数第七行：
```
 Message msg = queue.next(); // might block
 msg.target.dispatchMessage(msg);
```

Looper一直在不停的从MessageQueue中获取队列头部的Message，然后在获取这个Message的target并且执行dispatchMessage()方法。通过查看Handler的enqueueMessage的源码不难发现Message的target的值就是一个Handler，也就是发送Message的这个Handler，至此Message已经交给了Handler。
### Message是如何被Handler处理的

&emsp;&emsp;之前说Message已经交给了Handler了，并且执行了Handler的dispatchMessage()方法
```
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
        handleCallback(msg);
    } else {
        if (mCallback != null) {
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        handleMessage(msg);
    }
}
```
&emsp;&emsp;该方法逻辑较为简单。首先判断msg是否被设置了回调方法，如果有就执行他的回调方法(handleCallback方法)，也就是执行Message消息的callback对象的run方法，否则如何Handler的MCallback对象存在，则会执行mCallback的handleMessage方法，当两者都不存在或者mCallbackd的handleMessage方法返回为false时，就执行Handler里的handleMessage相关方法。通常并没有为Message设置回调函数，并且一直都是直接new一个不带回调函数的Handler，所以一般通过Handler的handleMessage方法来执行Message的，在handleMessage方法中通过what匹配，然后执行不同的逻辑。


由于Handler发送小徐实际上是王该Handler持有的线程的消息队列中添加消息，当该消息被执行的时候，就已经喊到了Handler所在的线程了。当Handler持有的线程为主线程更得时候消息执行肯定运行在了主线程，这样就达到了子线程向主线程的切换效果。

#### 为什么平常使用的时候不使用Looper.prepare和Looper.loop方法？

> 因为一同已经完成了这个任务，在ActivityThread中(Activity的真正入口)的main方法中已经实现了这个功能，所以在主线程使用Handler的时候可以不需要你使用上述的两个方法。而如果在子线程中使用Handler的时候如果在构造Handler之前没有调用上述的两个方法的时候就会抛出运行时异常。

## 通过源码细说消息机制的流程：

&emsp;&emsp;典型的关于Handler/Looper的线程：
    
```
class LooperThread extends Thread {
    public Handler mHandler;

    public void run() {
        Looper.prepare();  

        mHandler = new Handler() {  
            public void handleMessage(Message msg) {
                //TODO 定义消息处理逻辑. 
            }
        };

        Looper.loop();  
    }
}
//只有在子线程需要  Looper.prepare()和 Looper.loop();  
//因为主线程在初始化的时候已经创建了一个Looper对象了
```

### Looper:
#### prepare:

无参数的前提下默认调用的prepare(true)的方法，表示的就许这个loope允许退出,如果参数是false的话则表示当前的Looper不允许退出，一般主线程中的Looper都不允许退出。
```java
private static void prepare(boolean quitAllowed) {
    //每个线程只允许执行一次该方法，第二次执行时线程的TLS已有数据，则会抛出异常。
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    //创建Looper对象，并保存到当前线程的TLS区域
    sThreadLocal.set(new Looper(quitAllowed));
}

    //这个方法是主线程Looper初始化的代码，入口在ActivityThread中,即App初始化时就会被调用
    public static void prepareMainLooper() {
    prepare(false); //设置不允许退出的Looper
    synchronized (Looper.class) {
        //将当前的Looper保存为主Looper，每个线程只允许执行一次。
        if (sMainLooper != null) {
            throw new IllegalStateException("The main Looper has already been prepared.");
        }
        sMainLooper = myLooper();
    }
}
```

- 每个线程只能有一个Looper对象，并且与线程(Thread)绑定，在prepare方法中Looper新建了MessageQueue对象和Looper对象
- Looper中有一个ThreadLocal类型的sThreadLocal静态字段，Looper通过他的set()和get()方法来赋值和取值
- 由于TheadLocal与线程是绑定的，所以我们只要把Looper和ThreadLocal绑定了那么Looper和Thread也就绑定了。

#### 注：
> Looper除了prepare方法之外，还提供了prepareMainLooper方法，这个方法主要给ActivityThread创建Looper使用的，其本质也是通过prepare方法来实现。由于主线程的Looper比较特殊，所以Looper提供了一个getMainLooper方法，通过它可以再任何地方得到主线程的Looper。

#### Looper是如何保证线程单例的？
通过调用Looper的prepare方法，在本线程的TSL中查询是否有Looper实例，如果有的话就抛出异常提示用户一个线程只能有一个Looper。

Looper提供了两种方法来退出一个Looper:
&nbsp;

①quit可以直接退出Looper
②quitSafely只是设定一个退出标记，然后把消息队列中的已有的消息处理完毕之后才安全的退出

 ②Looper退出之后，通过Handler发送的消息会失败，这时候Handler的send方法就会返回false.

 ③在子线程中，如果手动为期创建了Looper，那么在所有的事件完成之后应该调用quit方法返回false来终止消息循环，否则这个线程就会一直处于的等待状态，而如果退出Looper之后，这个线程就会立即终止，因此建议在不需要的时候终止Looper。

#### loop():

```
public static void loop() {
    final Looper me = myLooper();  //获取TLS存储的Looper对象 
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    final MessageQueue queue = me.mQueue;  //获取Looper对象中的消息队列

    Binder.clearCallingIdentity();
    final long ident = Binder.clearCallingIdentity();

    for (;;) { //死循环
        Message msg = queue.next(); //可能会阻塞 
        if (msg == null) { //没有消息，则退出循环
            return;
        }
        //默认为null，可通过setMessageLogging()方法来指定输出，用于debug功能
        Printer logging = me.mLogging;  
        if (logging != null) {
            logging.println(">>>>> Dispatching to " + msg.target + " " +
                    msg.callback + ": " + msg.what);
        }
        //msg.target是一个Handler对象,回调Handler.dispatchMessage(msg)
        msg.target.dispatchMessage(msg); 
        ...
        msg.recycleUnchecked();  //将Message放入消息池,用以复用
    }
}
```

- 读取MessageQueue中的下一条Message
- 把Message分发给相应的Handler
- 将Message回收到到消息池中，以便重复利用


#### quit()
```
public void quit() {
    mQueue.quit(false); //消息移除
}

public void quitSafely() {
    mQueue.quit(true); //安全地消息移除
}

----------------------------------------MessageQueue#quit();-----------------------------

void quit(boolean safe) {
        // 主线程的MessageQueue.quit()行为会抛出异常,
        if (!mQuitAllowed) {
            throw new IllegalStateException("Main thread not allowed to quit.");
        }
        synchronized (this) {
            if (mQuitting) { //防止多次执行退出操作
                return;
            }
            mQuitting = true;
            if (safe) {
                removeAllFutureMessagesLocked(); //移除尚未触发的所有消息
            } else {
                removeAllMessagesLocked(); //移除所有的消息
            }
             // We can assume mPtr != 0 because mQuitting was previously false.
            nativeWake(mPtr);
        }
    }
```

#### 其他方法
```
Looper.myLooper()  //获取当前线程
public static @Nullable Looper myLooper() {
    return sThreadLocal.get();
}

Looper.getMainLooper() //获取主线程
    
//判断当前线程是不是主线程
Looper.myLooper() == Looper.getMainLooper()
Looper.getMainLooper().getThread() == Thread.currentThread()
Looper.getMainLooper().isCurrentThread()
```
## Handler创建过程：


主要作用：
- 跨进程通信
- 跨线程更新UI
- 与Looper，MessageQueue一起构建了Android的消息循环模型

&emsp;&emsp;Handler的工作主要包含消息的发送和接收过程。消息的发送就是通过post的一系列的方法以及send的一系列方法来实现，而post的一系列方法就是通过send的一系列方法来实现的


### Handler的创建

#### 无参构造
```
// 最简单的创建方式
public Handler() {
    this(null, false);
}

// ....还有很多种方式，但这些方式最终都执行这个构造方法
public Handler(Callback callback, boolean async) {
  //匿名类、内部类或本地类都必须申明为static，否则会警告可能出现内存泄露，
  //这个警告Android studio会提示给开发者
  // 1.检查内存泄漏
  if (FIND_POTENTIAL_LEAKS) {
      final Class<? extends Handler> klass = getClass();
      if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
              (klass.getModifiers() & Modifier.STATIC) == 0) {
          Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
              klass.getCanonicalName());
      }
  }
  //Looper与Handler的绑定过程，如果绑定之前没有调用Looper.prepare方法就会报错
  // 2.通过Looper.myLooper()//从当前线程的TLS中获取当前线程的Looper对象
  mLooper = Looper.myLooper();//获取Looper对象
  // 3.如果Looper为空，抛出异常
  if (mLooper == null) {
      throw new RuntimeException(
          "Can't create handler inside thread " + Thread.currentThread()
                  + " that has not called Looper.prepare()");
  }
  mQueue = mLooper.mQueue;//消息队列，来自Looper对象
  mCallback = callback;//回调方法
  mAsynchronous = async;//设置消息是否为异步处理方式
}

```

&emsp;&emsp;对于Looper中的无参构造方法，默认采用当前线程中的TSL中的Looper对象，并且callBack对象为null,且消息为同步处理方式。只要执行Looper.prepare方法就可以获取有效的Looper对象。而上述代码也揭示了为什么在没有Looper子线程中使用Handler会引发程序异常的原因

#### 有参构造：

```
//Handler与传入的Looper进行绑定
public Handler(Looper looper) {
    this(looper, null, false);
}

public Handler(Looper looper, Callback callback, boolean async) {
    mLooper = looper;
    mQueue = looper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
```

Handler可以在有参构造函数中指定要绑定的Looper,callback回调方法以及消息的处理方式(同步或者是异步)

#### Handler消息分发：
&emsp;&emsp;  Handler发送消息的过程仅仅是想队列中插入了一条消息，MessageQueue的next方法就会返回这条消息给Looper,Looper收到消息之后就会开始处理了，最终消息由Looper交由Handler处理，此时Handler的dispatchMessage方法就会被调用，这时Handler就进入了处理消息的阶段。
```
public void dispatchMessage(Message msg) {
    //调用Handler.post系列方法时，会给Message的callback赋值,
    //所以在分发消息的时候会调用handleCallback(msg)
    if (msg.callback != null) {
        handleCallback(msg);
    } else {
        if (mCallback != null) {
            //当Handler设置了回调对象Callback变量时，回调方法handleMessage()；
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        //Handler自身的回调方法handleMessage() 需子类实现相应逻辑
        handleMessage(msg);
    }
}
```

- 首先会先检查callback是否为null，不为null就通过handleCallback来处理消息，Message的callback是一个Runnable对象，实际上就是Handler的post方法所传递的Runnable参数。handleCallback的逻辑也十分简单：
```
private static void handleCallback(Message message){
    message.callback.run();
}
```
- 检查callback是否为null，不为null就调用mCallback的handleMessage方法来处理消息。Callback是一个接口
```
/**
* Callback interface you can use when instantiating a Handler to avoid having to implement your own subclass of Handler.

@param msg A {@link android.os.Message Message}object
@return True if no further handling is desired
*/

public interface Callback(){
    public boolean handleMessage(Message msg)
}
```

> 通过Callback可以采用以下方式来创建Handler对象：Handler handler = new Handler(callback)。Callback的意义在于：可以用来创建一个Handler的实例但是不需要派生Handler的子类。日常开发中最常见的就是派生一个Handler的子类并且重写handleMessage方法来处理具体的消息，而Callback有提供了另一种使用Handler的方式，当我们不想派生子类的时候，就可以通过Callback实现。Handler处理消息的过程如下图所示：


​    
​    
#### 消息发送

查看Handler的所有的消息发送的入口，包括post()系列和send()系列的方法，会发现最终都会到MessageQueue.enqueueMessage()方法
```
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
    //设置Message的消费Handler为当前Handler
    msg.target = this;
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis); 
}
//uptimeMillis = SystemClock.uptimeMillis() + delayMillis
//SystemClock.uptimeMillis() 代表的是自系统启动开始从0开始的到调用该方法时相差的毫秒数
//比起System.currentTimeMillis()更加可靠,因为System.currentTimeMillis()是跟系统时间强关联的,
//修改了系统时间,System.currentTimeMillis()就会发生变化.
```

#### 消息回收

##### removeMessages
```
public final void removeMessage(int what){
    mQueue.removeMessage(this,what,null);
}
```

##### removeCallbacksAndMessages
```
public final void removeCallbacksAndMessages(Object token) {
     mQueue.removeCallbacksAndMessages(this, token);
 }
```

实际上都是通过调用MessageQueue来达到目的


#### 如何避免因Handler引起的内存泄漏：

&ensp;&ensp;如上述代码，在创建Handler之处就先判断是否有内存泄漏。如果继承Handler的是匿名内部类或者是非静态内部类或者是本地内部类，则会提示有可能出现内存泄漏。一般Handler是在Activity或者是Fragment中，使用，匿名内部类，非静态内部类，本地内部类都会支持Activity对象，当Handler的生命周期比Activity长的时候，就会导致Activity的对象无法释放，从而导致内存泄漏。

### MessageQueue
MessageQueue主要包括两个主要的操作，就是插入和删除。而读取操作的本身就伴随着删除的操作，插入和读入对应的方法分别为enqueueMessage和next,其中enqueueMessage的作用是往消息队列中插入一条消息，而next的作用是从消息队列中取出一条消息并将其从消息队列中移除。


> <font color = red> 虽然MessageQueue叫做消息队列但是其实际是由单链表的数据结构实现的，因为单链表在插入何如删除上比较有优势。</font>
> &nbsp;

&emsp;是消息机制的Java层和C++层的连接纽带，大部分核心方法都是交由native层来处理的，其中有两个比较重要的方法：
```
private native void nativePollOnce(long ptr, int timeoutMillis);  //阻塞loop循环
private native static void nativeWake(long ptr);                  //唤醒loop循环
```

- next() //如果队列中还有消息，则提取下一条消息
- enqueueMessage()//按照参数when来添加消息到队列中
- removeMessages()//移除消息
```
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }
```

主要将Message消息添加到MessageQueue这个队列中，而MessageQueue消息队列就是上面Handler构造函数new出来的，由于Looper是ThreadLocal持有的，而Looper又持有MessageQueue，所以说ThreadLocal持有MessageQueue，也就是说每一个线程对应着一个MessageQueue对象


### 消息池

recycler()方法，用于把消息添加到消息池中。
&nbsp;

&emsp;好处：
&nbsp;

&emsp;&emsp;当线程池不为空时，可以直接从消息池中直接获取Message对象，而不是直接创建，提高效率



# 总结


## 一些要点：
### App进程启动和ActivityThread的关系
- Android中一个App就是一个进程
- 在普通的Java程序中一个程序的入口就是main方法，其实就是启动一个java进程，然后这个Java进程就是一个主线程
- Android系统会键有很多进程，AMS，PMS等都是在系统进程里面，app都要和这些进程通信

同理，在Android中也是启动一个进程，然后执行main方法，这个启动类就是ActivityThread.java,所以首先2执行的应该是ActivityThread的main方法
```
public static void main(String[] args) {
        SamplingProfilerIntegration.start();
        ........
        // 初始化一个 Loop
        Looper.prepareMainLooper();

        // 创建 当前进程中最主要的对象
        ActivityThread thread = new ActivityThread();

        // 初始化 application， Instrumentation
        thread.attach(false);
        ......
        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }
        // 主线程 循环； 应用第一个线程就不会停止了
        Looper.loop();
}
```

应用的进程启动的时候，首先调用的是ActivityThread的main方法，在main方法中初始化一个ActivityThread;然后执行ActivityThread实例的attach()方法，attach中把ApplicationThread和IActivityManager关联起来，并且和ViewRootImpl做了关联；<font color=red>ActivityThread中有个内部ApplicationThread类；这和主要用于Android系统的AMS进行系统进程通信。</font>

#### Activity被ActivityThread启动的过程
查看方法名  android.app.ActivityThread#performLaunchActivity

```
private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
        // System.out.println("##### [" + System.currentTimeMillis() + "] ActivityThread.performLaunchActivity(" + r + ")");

        // 创建 activity
        activity = mInstrumentation.newActivity(
                    cl, component.getClassName(), r.intent);

        try {
            // 获得 Application
            Application app = r.packageInfo.makeApplication(false, mInstrumentation);

                // 创建 context
                Context appContext = createBaseContextForActivity(r, activity);
                CharSequence title = r.activityInfo.loadLabel(appContext.getPackageManager());
                Configuration config = new Configuration(mCompatConfiguration);
                if (DEBUG_CONFIGURATION) Slog.v(TAG, "Launching activity "
                        + r.activityInfo.name + " with config " + config);
                // activity 中把 context 放进去，把 appliaction也放进去, 且在 attach 的时候，把activity 的windown 创建好了
                activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent, r.activityInfo, title, r.parent,
                        r.embeddedID, r.lastNonConfigurationInstances, config);
                // 把一些样式放进去

                activity.mCalled = false;
                // 调用 activity 的 onCreate方法
                mInstrumentation.callActivityOnCreate(activity, r.state);


            mActivities.put(r.token, r);

        //....................
        return activity;
    }
```

从上述代码可以看出，activity的创建和context，application的复制过程都是在ActivityThread中进行的且流程是
> new Activity ————> 设置application ————> 设置ContextImpl————> attach() ————> attachBaseContext() ————> onCreate()

以上所有都是正常使用java创建类的思路去理解，而且相对应的Activity的一些生命周期的管理也是从这个类中开始的。


## 其他补充
- 一个线程中可以有多个handler，但是只能够有一个Looper
- MessageQueue是有序的
- 子线程可以创建Handler对象
- 不可以在子线程中直接调用Handler的无参构造方法，因为Handler在创建的时候必须要绑定一个Looper对象，而有两种方法绑定：

> - 先调用Looper.prepare()在当前的线程中初始化一个Looper

```
Looper.prepare();
Handler handler = new Handler();
//...
//这一步必须
Looper.loop();
```
> - 通过构造方法传入一个Looper

```
Looper looper =......;//这里可以传入主线程的Looper
Handler handler = new Handler(looper);
```

## ThreadLocal

&emsp;&emsp;是一个线程内部的数据存储类(Thread Local Storagy,简称为TLS)，通过它可以在线程中存储数据，数据存储之后，只有在指定的线程中获取到存储的数据，对于其他的线程来说则无法获取到数据。日常开发的时候，很少有使用ThreadLocal的地方，但是系统在Handler机制中使用ThreadLocal来保证每一个Handler中只有一个独立的Looper对象。 **一般存储的当前进程的Looper对象**

### ThreadLocal是什么？

&emsp;&emsp;位于java.lang包下

&emsp;&emsp;是一个关于创建线程局部变量的类，什么是线程的局部变量？其实就是这个变量的作用域是线程，而其他的线程无法访问。而通常情况下我们所创建的变量是可以被任何一个线程访问的，而使用TheadLocal创建的对象只能被当前的线程所访问。

### 使用实例

```
public class ThreadLocalTest extends AppCompatActivity {
    private static final String TAG = "ThreadLocalTest";
    private ThreadLocal<String> stringThreadLocal = new ThreadLocal<>();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        booleanThreadLocal.set("MainThread");
        Log.d(TAG, "MainThread's stringThreadLocal=" + stringThreadLocal.get());
        new Thread("Thread#1") {
            @Override
            public void run() {
                Log.d(TAG, "Thread#1's stringThreadLocal=" + stringThreadLocal.get());
            }
        }.start();
    }
}
```
首先创建了一个泛型为String类型的ThreadLocal对象，并初始化，用以保存String类型的ThreadLocal对象。接着在主线程和子线程分别操作该对象，并且使用set()方法对其进行赋值，然后使用get()方法取值，
```
D/ThreadLocalTest: MainThread's stringThreadLocal = MainThread
D/ThreadLocalTest: Thread#1's stringThreadLocal = null
```

从打印结果可以看出ThreadLocal创建的对象的作用域是当前线程。
而虽然在不同的线程中访问同一个ThreadLocal对象，但是他们通过ThreadLocal获取到的值却不一样。主要输因为当不同的线程访问同一个ThreadLocal的get方法，ThreadLocal内部会从各自的线程中获一个人数组，然后在从数组中根据当前的ThreadLocal的索引去查找出相应的value值。<font color=DarkCyan >由于不同的线程中的数组各不相同，这就是为什么通过ThreadLocal可以在不同的线程中维护一套数据的副本并且彼此之间互不干扰。</font>

### ThreadLocal的内部实现：

#### ThreadLocal的set方法：
```
public void set(T value){
    Thread currentThread = Thread.currentThread();
    Values values = values(currentThread);
    if(values == null){
        values - initializeValues(currentThread);
    }
    values.put(this,value);
}
```

首先会通过value方法来获取当前线程中ThreadLocal的数据，即在Thread类的内部有一个成员专门用于存储线程的ThreadLocal的数据：ThreadLocal.Values localValues,因此获取当前线程的ThreadLocal的值就十分的简单了，如果ThreadLocal的值为null,就需要对其进行初始化，初始化之后再将ThreadLocal的值进行存储。

- **ThreadLocal的值如何在localValues中进行存储的**
  

localValues内部有一个table数组：private Object[] table,ThreadLocal中的值就存在这个table数组。
```
void put(ThreadLocal<?> key Object value){
    cleanUp();
    
    //Keep track of first tombstone.That's where we want to go back
    //and add an entry in necessary
    int firstTomstone = -1;
    
    for(int index=key.hash && mask;;index = next(index)){
        Object k = table[index];
        
        if(k == key.reference){
            //Replace existing entry
            table[index+1] = value;
            return;
        }
        
        if(key==null){
            if(firstTombStone == -1){
                //Fill in null slot.
                table[index] = key.reference;
                table[index+1] = value;
                size++;
                return;
            }
            
            //Go back and replace first tombstone
            table[firstTombstone] = key.reference;
            table[firstTombstone + 1] = value;
            tombstones--;
            size++;
            return;
        }
        //Remeber first tombstone
        if(firstTombstone == -1 && k == TOMBSTONE){
            firsttombstone = index; 
        }
    }
}
```

上述代码实现了数据的存储过程，可以的带一个存储规则，那就是ThreadLocal的值在table数组中存储的位置总是为ThreadLocal的reference字段所标识的对象的下一个位置，比如Thread的reference对象在table数组中的索引为index，那么ThreadLocal的值在table中的索引就是index+1。最终ThreadLocal的值将会被存储在table数组中：table[index+1]=value.


#### **ThreadLocal的get方法**

```
public T get(){
    //Optimized for the fast path
    Thread currentThread = Thread.currentThread();
    values values = values(currentThread);
    if(values !=null){
        Onject[] table = values.table;
        int index = hash && values.mask;
        if(this.reference == table[index]){
            return (T) table[index+1];
        }
    }else{
        values = initializeValues(currentThread);
    }
    
    return (T) values.getAfterMiss(this);
}
```

ThreadLocal的get同样是取出当前线程的localValues对象，如果这个对象为null那么就返回初始值(由ThreadLocal的initialValue方法来描述，默认情况下为null。当然也可以重写这个方法，默认实现如下：)，
```
/**
* Providers the initial values of this variable for the current thread.
*The default implementation returns{@code null}
*@ return the initial value of the variable.
*/

protected T initialValue(){
    return null;
}
```

&emsp;&emsp;如果localValues的对象不为null，就取出他的table数组并找出ThreadLocal的reference对象在table数组中的位置，然后table数组中的下一个位置所存储的数据就是ThreadLocal的值。
&emsp;&emsp;从ThreadLocal的get和set方法中可以看出，他们所操作的对象都是当前线程的localValues对象的table数组，因此在不同的线程中访问同一个ThreadLocal的set和get方法，他们对TheadLocal所做的读/写操作仅限于各自线程的内部，这就是为什么ThreadLocal可以在多个线程中互不干扰的存储以及修改数据。
&nbsp;

ThreadLocal在Android源码中的表现主要在Looper，ActivityThread以及AMS。具体到ThreadLocal的使用场景，一般来水，当某些数据是以线程为作用域并且不同线程具有不同的数据副本的时候，就可以考虑采用ThreadLocal。比如：
- 对于Handler来说，他需要获取当前线程的Looper,很显然Looper的作用域就是线程并且不同的线程具有不同的Looper，此时就可以通过ThreadLocal轻松地实现Looper在线程中的存取。如果不采用ThreadLocal的话，系统就必须
提供一个全局的哈希表共Handler查找指定线程的Looper，这样一来就必须提供一个类似于LooperManager的类了，但是系统并没有这么做，而是使用了ThreadLocal。
- ThreadLocal的另一个使用场景就是复杂逻辑下的对象传递，比如监听器发的传递，有时候一个线程中的任务过于复杂，这可能表现为函数调用栈比较深以及代码入口的多样性，在这种情况下，我们有需要监听器能够贯穿整个线程的执行过程，这是就可以采用ThreadLocal，可以让监听器作为县城内的全局变量而存在，在线程内部只要通过get方法就可以获取到监听器。
> 如果不采用ThreadLocal的话，有如下两种方法：
- 将监听器通过参数的形式在韩式调用栈中传递
- 将监听器作为静态变量供线程访问

<font color=red>上述两种方法都是有局限性的：

①第一种方法时当函数栈很深的时候，通过函数参数传递监听器对象几乎是不可接受的，

②第二种方法是可以接受的，但是这种状态是不具有可扩充性的，比如同时与两个线程在执行的时候，就需要提供两个静态的监听对象。而当线程个数过多时就会十分的麻烦。

而如果直接使用ThreadLocal的话，每个监听器对象都在自己的线程内部存储，根部不需要考虑上述的两个问题
</font>
```
public class ThreadLocalTest extends AppCompatActivity {
    private static final String TAG = "ThreadLocalTest";
    private ThreadLocal<Boolean> mBooleanThreadLocal = new ThreadLocal<Boolean>();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        mBooleanThreadLocal.set(true);
        Log.d(TAG, "[Thread#main]mBooleanThreadLocal=" + mBooleanThreadLocal.get());
 
        new Thread("Thread#1") {
            @Override
            public void run() {
                mBooleanThreadLocal.set(false);
                Log.d(TAG, "[Thread#1]mBooleanThreadLocal=" + mBooleanThreadLocal.get());
            };
        }.start();
 
        new Thread("Thread#2") {
            @Override
            public void run() {
                Log.d(TAG, "[Thread#2]mBooleanThreadLocal=" + mBooleanThreadLocal.get());
            };
        }.start();
    }
}

//输出结果
//[Thread#main]mBooleanThreadLocal=true
//[Thread#1]mBooleanThreadLocal=false
//[Thread#2]mBooleanThreadLocal=null
```

&emsp;&emsp;从输出结果来看，虽然调用的都是mBooleanThreadLocal.get()
方法，但是输出的结果却不同，这是由于ThreadLocal会在每一个线程中创建一个数据存储空间(ThreadLocalMap),分别保存各个线程中的数据。

### ThreadLocal的实现

&emsp;&emsp;ThreadLocal有三个public方法：set，get，remove

> <font size=3 color=CadetBlue> ThreadLocal#set</font>
```
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

- 获取当前的线程
- 获取当前线程对应的ThreeadLocalMap,这个map是跟线程绑定的，所以存在这个map的数据也适合线程绑定的
- 往ThreadLocalMap中添加数据，或者重新创建一个ThreadLocalMap对象


> <font size=3 color=CadetBlue> ThreadLocal#get</font>
```
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null)
            return (T)e.value;
    }
    return setInitialValue();
}
```

将之前通过set方法保存到ThreadLocal中的数据取出来


&nbsp;

&emsp;&emsp;通过分析Handler的源码可知，他主要是通过ThreadLocal原理和不断从对应线程的消息队列中取消息的机制，将消息的执行过程抛到对UI你给的线程中去执行，从而达到线程切换的效果。而在主线程更新UI只是Handler的一小部分功能的实现。


### ThreadLocal中的H作用？

<font color = red >**进程是分配资源的单位，而进程内的线程共享这些资源(包括内存)**</font>

```
 private class H extends Handler {
        public static final int LAUNCH_ACTIVITY         = 100;
        public static final int PAUSE_ACTIVITY          = 101;
        ...
        // 100 - 157 顺序排列的标志位：包括四大组件的状态等
        public static final int ATTACH_AGENT = 155;
        public static final int APPLICATION_INFO_CHANGED = 156;
        public static final int ACTIVITY_MOVED_TO_DISPLAY = 157;

        String codeToString(int code) {
            if (DEBUG_MESSAGES) {
                switch (code) {
                    case LAUNCH_ACTIVITY: return "LAUNCH_ACTIVITY";
                    case PAUSE_ACTIVITY: return "PAUSE_ACTIVITY";
                    ...
                    // 将上面定义的标志位转换成对应的字符串
                    case LOCAL_VOICE_INTERACTION_STARTED: return "LOCAL_VOICE_INTERACTION_STARTED";
                    case ATTACH_AGENT: return "ATTACH_AGENT";
                    case APPLICATION_INFO_CHANGED: return "APPLICATION_INFO_CHANGED";
                }
            }
            return Integer.toString(code);
        }
        public void handleMessage(Message msg) {
            if (DEBUG_MESSAGES) Slog.v(TAG, ">>> handling: " + codeToString(msg.what));
            switch (msg.what) {
                case LAUNCH_ACTIVITY: {
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityStart");
                    final ActivityClientRecord r = (ActivityClientRecord) msg.obj;

                    r.packageInfo = getPackageInfoNoCheck(
                            r.activityInfo.applicationInfo, r.compatInfo);
                    handleLaunchActivity(r, null, "LAUNCH_ACTIVITY");
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                } break;
                case RELAUNCH_ACTIVITY: {
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityRestart");
                    ActivityClientRecord r = (ActivityClientRecord)msg.obj;
                    handleRelaunchActivity(r);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                } break;
                ...
                // 按着处理58种信息
                case ATTACH_AGENT:
                    handleAttachAgent((String) msg.obj);
                    break;
                case APPLICATION_INFO_CHANGED:
                    mUpdatingSystemConfig = true;
                    try {
                        handleApplicationInfoChanged((ApplicationInfo) msg.obj);
                    } finally {
                        mUpdatingSystemConfig = false;
                    }
                    break;
            }
            Object obj = msg.obj;
            if (obj instanceof SomeArgs) {
                ((SomeArgs) obj).recycle();
            }
            if (DEBUG_MESSAGES) Slog.v(TAG, "<<< done: " + codeToString(msg.what));
        }
    }
```

通过上述的源码可知，H是ActivityThread的内部类，主要作用如下：

- 定义了58中状态，包括了四大组件(主要)，Application，Windows等的状态
- 将状态值转化为flag对应的字符串，主要用于之后的Log的信息输出。
- 最主要的是根据状态值进行相应的处理，比如重写handlerMessage方法。
比如，在App中创建一个Activity，然后使用Binder调用远程服务，然后调用第一个case里面的handlerLaunchActivity(r,null,"Launch_Activity")通过类加载器创建一个新的Activity。



然后H在执行Main函数的时候进行了实例化。

```
  public static void main(String[] args) {
        ....

        Looper.prepareMainLooper();

        ActivityThread thread = new ActivityThread();
        thread.attach(false);

        ...
        Looper.loop();
        ...
    }
```